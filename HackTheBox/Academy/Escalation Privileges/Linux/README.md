Introduction à l'escalade de privilèges Linux
=========================================

* * * * *

Le compte racine sur les systèmes Linux fournit un accès de niveau administratif complet au système d'exploitation. Au cours d'une évaluation, vous pouvez obtenir un shell à faibles privilèges sur un hôte Linux et avoir besoin d'effectuer une élévation des privilèges vers le compte root. Compromettre complètement l'hôte nous permettrait de capturer le trafic et d'accéder aux fichiers sensibles, qui peuvent être utilisés pour accéder davantage à l'environnement. De plus, si la machine Linux est jointe à un domaine, nous pouvons obtenir le hachage NTLM et commencer à énumérer et attaquer Active Directory.

* * * * *

Énumération
-----------

L'énumération est la clé de l'escalade de privilèges. Plusieurs scripts d'assistance (tels que [LinEnum](https://github.com/rebootuser/LinEnum)) existent pour faciliter l'énumération. Néanmoins, il est également important de comprendre quelles informations rechercher et de pouvoir effectuer votre dénombrement manuellement. Lorsque vous obtenez un accès shell initial à l'hôte, il est important de vérifier plusieurs détails clés.

`Version du système d'exploitation` : Connaître la distribution (Ubuntu, Debian, FreeBSD, Fedora, SUSE, Red Hat, CentOS, etc.) vous donnera une idée des types d'outils qui peuvent être disponibles. Cela identifierait également la version du système d'exploitation, pour laquelle des exploits publics peuvent être disponibles.

« Version du noyau » : comme pour la version du système d'exploitation, il peut y avoir des exploits publics qui ciblent une vulnérabilité dans une version spécifique du noyau. Les exploits du noyau peuvent provoquer une instabilité du système ou même un crash complet. Soyez prudent lorsque vous les exécutez sur n'importe quel système de production et assurez-vous de bien comprendre l'exploit et les ramifications possibles avant d'en exécuter un.

"Services en cours d'exécution": il est important de savoir quels services s'exécutent sur l'hôte, en particulier ceux qui s'exécutent en tant que root. Un service mal configuré ou vulnérable exécuté en tant que root peut être une victoire facile pour l'élévation des privilèges. Des failles ont été découvertes dans de nombreux services courants tels que Nagios, Exim, Samba, ProFTPd, etc. Des PoC d'exploits publics existent pour beaucoup d'entre eux, comme CVE-2016-9566, une faille locale d'élévation de privilèges dans Nagios Core < 4.2.4.

#### Liste des processus actuels

Lister les processus actuels

```
dsgsec@htb[/htb]$ ps aux | grep racine

racine 1 1.3 0.1 37656 5664 ? Ss 23:26 0:01 /sbin/init
racine 2 0.0 0.0 0 0 ? S 23:26 0:00 [kthread]
racine 3 0.0 0.0 0 0 ? S 23:26 0:00 [ksoftirqd/0]
racine 4 0.0 0.0 0 0 ? S 23:26 0:00 [kworker/0:0]
racine 5 0.0 0.0 0 0 ? S< 23:26 0:00 [kworker/0:0H]
racine 6 0.0 0.0 0 0 ? S 23:26 0:00 [kworker/u8:0]
racine 7 0.0 0.0 0 0 ? S 23:26 0:00 [rcu_sched]
racine 8 0.0 0.0 0 0 ? S 23:26 0:00 [rcu_bh]
racine 9 0.0 0.0 0 0 ? S 23:26 0:00 [migration/0]

<SNIP>

```

« Packages et versions installés » : comme pour les services en cours d'exécution, il est important de vérifier les packages obsolètes ou vulnérables qui peuvent être facilement exploités pour l'élévation des privilèges. Un exemple est Screen, qui est un multiplexeur de terminal commun (similaire à tmux). Il vous permet de démarrer une session et d'ouvrir de nombreuses fenêtres ou terminaux virtuels au lieu d'ouvrir plusieurs sessions de terminal. La version 4.05.00 de Screen souffre d'une vulnérabilité d'escalade de privilèges qui peut être facilement exploitée pour élever les privilèges.

« Utilisateurs connectés » : savoir quels autres utilisateurs sont connectés au système et ce qu'ils font peut donner plus d'informations sur les éventuels mouvements latéraux locaux et les chemins d'escalade des privilèges.

#### Liste des processus actuels

Lister les processus actuels

```
dsgsec@htb[/htb]$ ps au

USER PID %CPU %MEM VSZ RSS TTY STAT HEURE DE DÉMARRAGE COMMANDE
racine 1256 0.0 0.1 65832 3364 tty1 Ss 23:26 0:00 /bin/login --
cliff.moore 1322 0.0 0.1 22600 5160 tty1 S 23:26 0:00 -bash
partagé 1367 0.0 0.1 22568 5116 pts/0 Ss 23:27 0:00 -bash
racine 1384 0.0 0.1 52700 3812 tty1 S 23:29 0:00 sudo su
racine 1385 0.0 0.1 52284 3448 tty1 S 23:29 0:00 su
racine 1386 0.0 0.1 21224 3764 tty1 S+ 23:29 0:00 bash
partagé 1397 0.0 0.1 37364 3428 pts/0 R+ 23:30 0:00 ps au

```

`Répertoires personnels des utilisateurs` : les répertoires personnels des autres utilisateurs sont-ils accessibles ? Les dossiers d'accueil des utilisateurs peuvent également contenir des clés SSH qui peuvent être utilisées pour accéder à d'autres systèmes ou scripts et fichiers de configuration contenant des informations d'identification. Il n'est pas rare de trouver des fichiers contenant des informations d'identification qui peuvent être exploitées pour accéder à d'autres systèmes ou même accéder à l'environnement Active Directory.

#### Contenu du répertoire d'accueil

Contenu du répertoire d'accueil

```
dsgsec@htb[/htb]$ ls /home

backupsvc bob.jones cliff.moore logger mrb3n partagé stacey.jenkins

```

Nous pouvons vérifier les répertoires d'utilisateurs individuels et vérifier si des fichiers tels que le fichier `.bash_history` sont lisibles et contiennent des commandes intéressantes, regardezpour les fichiers de configuration, et vérifiez si nous pouvons obtenir des copies des clés SSH d'un utilisateur.

#### Contenu du répertoire d'accueil de l'utilisateur

Contenu du répertoire personnel de l'utilisateur

```
dsgsec@htb[/htb]$ ls -la /home/stacey.jenkins/

total 32
drwxr-xr-x 3 stacey.jenkins stacey.jenkins 4096 30 août 23:37 .
drwxr-xr-x 9 racine racine 4096 30 août 23:33 ..
-rw------- 1 stacey.jenkins stacey.jenkins 41 août 30 23:35 .bash_history
-rw-r--r-- 1 stacey.jenkins stacey.jenkins 220 1er septembre 2015 .bash_logout
-rw-r--r-- 1 stacey.jenkins stacey.jenkins 3771 1er septembre 2015 .bashrc
-rw-r--r-- 1 stacey.jenkins stacey.jenkins 97 30 août 23:37 config.json
-rw-r--r-- 1 stacey.jenkins stacey.jenkins 655 16 mai 2017 .profile
drwx------ 2 stacey.jenkins stacey.jenkins 4096 30 août 23:35 .ssh

```

Si vous trouvez une clé SSH pour votre utilisateur actuel, cela pourrait être utilisé pour ouvrir une session SSH sur l'hôte (si SSH est exposé en externe) et obtenir une session stable et entièrement interactive. Les clés SSH pourraient également être utilisées pour accéder à d'autres systèmes du réseau. Au minimum, vérifiez le cache ARP pour voir à quels autres hôtes sont accédés et comparez-les avec toutes les clés privées SSH utilisables.

#### Contenu du répertoire SSH

Contenu du répertoire SSH

```
dsgsec@htb[/htb]$ ls -l ~/.ssh

total 8
-rw------- 1 mrb3n mrb3n 1679 30 août 23:37 id_rsa
-rw-r--r-- 1 mrb3n mrb3n 393 30 août 23:37 id_rsa.pub

```

Il est également important de vérifier l'historique bash d'un utilisateur, car il peut transmettre des mots de passe en tant qu'argument sur la ligne de commande, travailler avec des référentiels git, configurer des tâches cron, etc. L'examen de ce que l'utilisateur a fait peut vous donner un aperçu considérable du type de serveur sur lequel vous atterrissez et vous donner un indice sur les chemins d'escalade privilégiés.

#### Historique des bashs

Historique des coups

```
dsgsec@htb[/htb]$ historique

     1 pièce d'identité
     2 cd /home/cliff.moore
     3 sortie
     4 touchez backup.sh
     5 queue /var/log/apache2/error.log
     6 ssh ec2-user@dmz02.inlanefreight.local
     7 histoire

```

`Sudo Privileges` : l'utilisateur peut-il exécuter des commandes en tant qu'autre utilisateur ou en tant que root ? Si vous n'avez pas d'informations d'identification pour l'utilisateur, il peut ne pas être possible d'exploiter les autorisations sudo. Cependant, les entrées sudoer incluent souvent `NOPASSWD`, ce qui signifie que l'utilisateur peut exécuter la commande spécifiée sans être invité à entrer un mot de passe. Toutes les commandes, même si nous pouvons les exécuter en tant que root, ne conduiront pas à une élévation des privilèges. Il n'est pas rare d'avoir accès en tant qu'utilisateur avec tous les privilèges sudo, ce qui signifie qu'il peut exécuter n'importe quelle commande en tant que root. L'émission d'une simple commande `sudo su` vous donnera immédiatement une session root.

#### Sudo - Liste des privilèges de l'utilisateur

Sudo - Liste des privilèges de l'utilisateur

```
dsgsec@htb[/htb]$ sudo -l

Entrées par défaut correspondantes pour sysadm sur NIX02 :
     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

L'utilisateur sysadm peut exécuter les commandes suivantes sur NIX02 :
     (racine) NOPASSWD : /usr/sbin/tcpdump

```

« Fichiers de configuration » : les fichiers de configuration peuvent contenir une mine d'informations. Il vaut la peine de rechercher dans tous les fichiers qui se terminent par des extensions telles que `.conf` et `.config`, les noms d'utilisateur, mots de passe et autres secrets.

`Fichier fantôme lisible` : si le fichier fantôme est lisible, vous pourrez collecter les hachages de mot de passe pour tous les utilisateurs qui ont défini un mot de passe. Bien que cela ne garantisse pas un accès ultérieur, ces hachages peuvent être soumis à une attaque par force brute hors ligne pour récupérer le mot de passe en clair.

`Hachages de mot de passe dans /etc/passwd` : De temps en temps, vous verrez des hachages de mot de passe directement dans le fichier /etc/passwd. Ce fichier est lisible par tous les utilisateurs et, comme pour les hachages du fichier `shadow` , ceux-ci peuvent être soumis à une attaque de craquage de mot de passe hors ligne. Cette configuration, bien que peu courante, peut parfois être observée sur les périphériques et routeurs intégrés.

#### Mot de passe

Mot de passe

```
dsgsec@htb[/htb]$ cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
<...SNIP...>
dnsmasq:x:109:65534:dnsmasq,,,:/var/lib/misc:/bin/false
sshd:x:110:65534::/var/run/sshd:/usr/sbin/nologin
mrb3n:x:1000:1000:mrb3n,,,:/home/mrb3n:/bin/bash
colord:x:111:118:colord colour management daemon,,,:/var/lib/colord:/bin/false
backupsvc:x:1001:1001::/home/backupsvc:
bob.jones:x:1002:1002::/home/bob.jones:
cliff.moore:x:1003:1003::/home/cliff.moore:
logger:x:1004:1004::/home/logger:
shared:x:1005:1005::/home/shared:
stacey.jenkins:x:1006:1006::/home/stacey.jenkins:
sysadm:$6$vdH7vuQIv6anIBWg$Ysk.UZzI7WxYUBYt8WRIWF0EzWlksOElDE0HLYinee38QI1A.0HW7WZCrUhZ9wwDz13bPpkTjNuRoUGYhwFE11:1007:1007::/home/sysadm:

```

`Tâches Cron` : les tâches Cron sur les systèmes Linux sont similaires aux tâches planifiées de Windows. Ils sont souventconfiguré pour effectuer des tâches de maintenance et de sauvegarde. En conjonction avec d'autres erreurs de configuration telles que des chemins relatifs ou des autorisations faibles, ils peuvent tirer parti pour élever les privilèges lors de l'exécution de la tâche cron planifiée.

#### Tâches Cron

Tâches Cron

```
dsgsec@htb[/htb]$ ls -la /etc/cron.daily/

total 60
drwxr-xr-x  2 root root 4096 Aug 30 23:49 .
drwxr-xr-x 93 root root 4096 Aug 30 23:47 ..
-rwxr-xr-x  1 root root  376 Mar 31  2016 apport
-rwxr-xr-x  1 root root 1474 Sep 26  2017 apt-compat
-rwx--x--x  1 root root  379 Aug 30 23:49 backup
-rwxr-xr-x  1 root root  355 May 22  2012 bsdmainutils
-rwxr-xr-x  1 root root 1597 Nov 27  2015 dpkg
-rwxr-xr-x  1 root root  372 May  6  2015 logrotate
-rwxr-xr-x  1 root root 1293 Nov  6  2015 man-db
-rwxr-xr-x  1 root root  539 Jul 16  2014 mdadm
-rwxr-xr-x  1 root root  435 Nov 18  2014 mlocate
-rwxr-xr-x  1 root root  249 Nov 12  2015 passwd
-rw-r--r--  1 root root  102 Apr  5  2016 .placeholder
-rwxr-xr-x  1 root root 3449 Feb 26  2016 popularity-contest
-rwxr-xr-x  1 root root  214 May 24  2016 update-notifier-common

```

« Systèmes de fichiers non montés et lecteurs supplémentaires » : si vous découvrez et pouvez monter un lecteur supplémentaire ou un système de fichiers non monté, vous pouvez trouver des fichiers, des mots de passe ou des sauvegardes sensibles qui peuvent être exploités pour augmenter les privilèges.

#### Systèmes de fichiers et lecteurs supplémentaires

Systèmes de fichiers et lecteurs supplémentaires

```
dsgsec@htb[/htb]$ lsblk

NOM MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
disque sda 8:0 0 30G 0
├─sda1 8:1 0 29G 0 partie /
├─sda2 8:2 0 1K 0 partie
└─sda5 8:5 0 975M 0 partie [SWAP]
sr0 11:0 1 848M 0 rom

```

"Autorisations SETUID et SETGID" : les fichiers binaires sont définis avec ces autorisations pour permettre à un utilisateur d'exécuter une commande en tant que root, sans avoir à accorder un accès de niveau root à l'utilisateur. De nombreux binaires contiennent des fonctionnalités qui peuvent être exploitées pour obtenir un shell root.

`Répertoires inscriptibles` : il est important de découvrir quels répertoires sont accessibles en écriture si vous avez besoin de télécharger des outils sur le système. Vous pouvez découvrir un répertoire accessible en écriture dans lequel une tâche cron place des fichiers, ce qui donne une idée de la fréquence d'exécution de la tâche cron et peut être utilisé pour élever les privilèges si le script exécuté par la tâche cron est également accessible en écriture.

#### Trouver des répertoires inscriptibles

Trouver des répertoires inscriptibles

```
dsgsec@htb[/htb]$ find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null

/dmz-sauvegardes
/tmp
/tmp/VMwareDnD
/tmp/.XIM-unix
/tmp/.Test-unix
/tmp/.X11-unix
/tmp/systemd-private-8a2c51fcbad240d09578916b47b0bb17-systemd-timesyncd.service-TIecv0/tmp
/tmp/.font-unix
/tmp/.ICE-unix
/proc
/dev/mqueue
/dev/shm
/var/tmp
/var/tmp/systemd-private-8a2c51fcbad240d09578916b47b0bb17-systemd-timesyncd.service-hm6Qdl/tmp
/var/accident
/exécuter/verrouiller

```

`Fichiers inscriptibles` : certains scripts ou fichiers de configuration sont-ils accessibles en écriture par tout le monde ? Bien que la modification des fichiers de configuration puisse être extrêmement destructrice, il peut y avoir des cas où une modification mineure peut ouvrir un accès supplémentaire. De plus, tous les scripts exécutés en tant que root à l'aide de tâches cron peuvent être légèrement modifiés pour ajouter une commande.

#### Rechercher des fichiers inscriptibles

Trouver des fichiers inscriptibles

```
dsgsec@htb[/htb]$ find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

/etc/cron.daily/backup
/dmz-backups/backup.sh
/proc
/sys/fs/cgroup/memory/init.scope/cgroup.event_control

<SNIP>

/home/backupsvc/backup.sh

<SNIP>

```