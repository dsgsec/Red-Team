Abus de chemin
==========

* * * * *

[PATH](http://www.linfo.org/path_env_var.html) est une variable d'environnement qui spécifie l'ensemble de répertoires où un exécutable peut se trouver. La variable PATH d'un compte est un ensemble de chemins absolus, permettant à un utilisateur de taper une commande sans spécifier le chemin absolu vers le binaire. Par exemple, un utilisateur peut saisir `cat /tmp/test.txt` au lieu de spécifier le chemin absolu `/bin/cat /tmp/test.txt`. Nous pouvons vérifier le contenu de la variable PATH en tapant `env | grep PATH` ou `echo $PATH`.

```
htb_student@NIX02 :~$ echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games

```

La création d'un script ou d'un programme dans un répertoire spécifié dans le PATH le rendra exécutable à partir de n'importe quel répertoire du système.

```
htb_student@NIX02 :~$ pwd && conncheck

/usr/local/sbin
Connexions Internet actives (serveurs et établis)
Proto Recv-Q Send-Q Adresse locale Adresse étrangère État PID/Nom du programme
tcp 0 0 0.0.0.0:22 0.0.0.0:* ÉCOUTER 1189/sshd
tcp 0 88 10.129.2.12:22 10.10.14.3:43218 ÉTABLI 1614/sshd : mrb3n [p
tcp6 0 0 :::22 :::* ECOUTEZ 1189/sshd
tcp6 0 0 :::80 :::* ECOUTEZ 1304/apache2

```

Comme indiqué ci-dessous, le script `conncheck` créé dans `/usr/local/sbin` s'exécutera toujours dans le répertoire `/tmp` car il a été créé dans un répertoire spécifié dans le PATH.

```
htb_student@NIX02 :~$ pwd && conncheck

/tmp
Connexions Internet actives (serveurs et établis)
Proto Recv-Q Send-Q Adresse locale Adresse étrangère État PID/Nom du programme
tcp 0 0 0.0.0.0:22 0.0.0.0:* ÉCOUTER 1189/sshd
tcp 0 268 10.129.2.12:22 10.10.14.3:43218 ÉTABLI 1614/sshd : mrb3n [p
tcp6 0 0 :::22 :::* ECOUTEZ 1189/sshd
tcp6 0 0 :::80 :::* ECOUTEZ 1304/apache2

```

L'ajout de `.` au PATH d'un utilisateur ajoute son répertoire de travail actuel à la liste. Par exemple, si nous pouvons modifier le chemin d'accès d'un utilisateur, nous pourrions remplacer un binaire commun tel que `ls` par un script malveillant tel qu'un reverse shell. Si nous ajoutons `.` au chemin en exécutant la commande `PATH=.:$PATH` puis `export PATH`, nous pourrons exécuter les fichiers binaires situés dans notre répertoire de travail actuel en tapant simplement le nom du fichier ( c'est-à-dire qu'il suffit de taper `ls` pour appeler le script malveillant nommé `ls` dans le répertoire de travail actuel au lieu du binaire situé dans `/bin/ls`).

```
htb_student@NIX02 :~$ echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games

```

```
htb_student@NIX02 :~$ CHEMIN=. :${CHEMIN}
htb_student@NIX02 :~$ export PATH
htb_student@NIX02 :~$ echo $PATH

.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games

```

Dans cet exemple, nous modifions le chemin pour exécuter une simple commande `echo` lorsque la commande `ls` est tapée.

```
htb_student@NIX02 :~$ toucher ls
htb_student@NIX02:~$ echo 'echo "ABUS DE CHEMIN !!"' > ls
htb_student@NIX02 :~$ chmod +x ls

```

```
htb_student@NIX02 :~$ ls

ABUS DE CHEMIN !!

```