Énumération de l'environnement
==============================

* * * * *

L'énumération est la clé de l'escalade de privilèges. Plusieurs scripts d'assistance (tels que [LinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) et [LinEnum](https://github.com/rebootuser/LinEnum) existent pour aider à l'énumération. Néanmoins, il est également important de comprendre les informations à rechercher et de pouvoir effectuer votre énumération manuellement. Lorsque vous obtenez un accès initial au shell de l'hôte, il est important de vérifier plusieurs détails clés.

`OS Version`: Connaître la distribution (Ubuntu, Debian, FreeBSD, Fedora, SUSE, Red Hat, CentOS, etc.) vous donnera une idée des types d'outils qui peuvent être disponibles. Cela identifierait également la version du système d'exploitation, pour laquelle des exploits publics peuvent être disponibles.

`Kernel Version`: Comme pour la version du système d'exploitation, il peut y avoir des exploits publics ciblant une vulnérabilité dans une version spécifique du noyau. Les exploits du noyau peuvent provoquer une instabilité du système ou même un crash complet. Soyez prudent lorsque vous les exécutez sur n'importe quel système de production et assurez-vous de bien comprendre l'exploit et les ramifications possibles avant d'en exécuter un.

`Running Services`: Il est important de savoir quels services s'exécutent sur l'hôte, en particulier ceux qui s'exécutent en tant que root. Un service mal configuré ou vulnérable exécuté en tant que root peut être une victoire facile pour l'élévation des privilèges. Des failles ont été découvertes dans de nombreux services courants tels que Nagios, Exim, Samba, ProFTPd, etc. Des PoC d'exploits publics existent pour beaucoup d'entre eux, comme CVE-2016-9566, une faille locale d'élévation de privilèges dans Nagios Core < 4.2.4.

* * * * *

Acquérir une connaissance de la situation
-----------------------------------------

Disons que nous venons d'accéder à un hôte Linux en exploitant une vulnérabilité de téléchargement de fichiers sans restriction lors d'un test de pénétration externe. Après avoir établi notre shell inversé (et idéalement une sorte de persistance), nous devrions commencer par rassembler quelques bases sur le système avec lequel nous travaillons.

Dans un premier temps, nous répondrons à la question fondamentale : à quel système d'exploitation avons-nous affaire ? Si nous atterrissions sur un hôte CentOS ou un hôte Red Hat Enterprise Linux, notre énumération serait probablement légèrement différente de celle si nous atterrissions sur un hôte basé sur Debian tel qu'Ubuntu. Si nous atterrissons sur un hôte tel que FreeBSD, Solaris ou quelque chose de plus obscur tel que le système d'exploitation propriétaire HP HP-UX ou IBM OS AIX, les commandes avec lesquelles nous travaillerons seront probablement différentes. Bien que les commandes puissent être différentes et que nous ayons même besoin de rechercher une référence de commande dans certains cas, les principes sont les mêmes. Pour nos besoins, nous commencerons par une cible Ubuntu pour couvrir les tactiques et techniques générales. Une fois que nous avons appris les bases et que nous les combinons avec une nouvelle façon de penser et les étapes du processus de test d'intrusion, cela devrait

Il existe de nombreuses feuilles de triche pour aider à énumérer les systèmes Linux et certaines informations qui nous intéressent auront deux ou plusieurs façons de les obtenir. Dans ce module, nous couvrirons une méthodologie qui peut probablement être utilisée pour la majorité des systèmes Linux que nous rencontrons dans la nature. Cela étant dit, assurez-vous de comprendre ce que font les commandes et comment les modifier ou trouver les informations dont vous avez besoin d'une manière différente si une commande particulière ne fonctionne pas. Relevez le défi au cours de ce module pour essayer différentes façons de mettre en pratique votre méthodologie et ce qui vous convient le mieux. N'importe qui peut retaper les commandes d'une feuille de triche, mais une compréhension approfondie de ce que vous recherchez et de la manière de l'obtenir nous aidera à réussir dans n'importe quel environnement.

En règle générale, nous voudrons exécuter quelques commandes de base pour nous orienter :

-   `whoami`- sous quel utilisateur courons-nous
-   `id`- à quels groupes notre utilisateur appartient-il ?
-   `hostname`- quel est le nom du serveur. pouvons-nous recueillir quelque chose de la convention de nommage ?
-   `ifconfig`ou `ip -a`- dans quel sous-réseau avons-nous atterri, l'hôte a-t-il des cartes réseau supplémentaires dans d'autres sous-réseaux ?
-   `sudo -l`- notre utilisateur peut-il exécuter quoi que ce soit avec sudo (en tant qu'autre utilisateur en tant que root) sans avoir besoin d'un mot de passe ? Cela peut parfois être la victoire la plus facile et nous pouvons faire quelque chose comme `sudo su`et tomber directement dans un shell root.

J'aime également capturer certaines des informations ci-dessus à utiliser dans le rapport du client pour montrer la preuve d'un RCE réussi et indiquer clairement quel système a été affecté. Passons maintenant à notre énumération plus détaillée, étape par étape.

Nous allons commencer par vérifier à quel système d'exploitation et quelle version nous avons affaire.

```
dsgsec@htb[/htb]$ cat /etc/os-release

NAME="Ubuntu"
VERSION="20.04.4 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.4 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal

```

Nous pouvons voir que la cible exécute [Ubuntu 20.04.4 LTS ("Focal Fossa")](https://releases.ubuntu.com/20.04/) . Quelle que soit la version que nous rencontrons, il est important de voir si nous avons affaire à quelque chose d'obsolète ou de maintenu. Ubuntu publie son [cycle de publication](https://ubuntu.com/about/release-cycle) et à partir de là, nous pouvons voir que "Focal Fossa" n'atteindra sa fin de vie qu'en avril 2030. À partir de ces informations, nous pouvons supposer que nous ne rencontrerons pas de vulnérabilité Kernel bien connue car le client a conservé son l'actif accessible sur Internet a été corrigé, mais nous continuerons de chercher quoi qu'il en soit.

Ensuite, nous voudrons vérifier le PATH de notre utilisateur actuel, qui est l'endroit où le système Linux regarde chaque fois qu'une commande est exécutée pour que tous les exécutables correspondent au nom de ce que nous tapons, c'est-à-dire, qui sur ce système se trouve `id`à `/usr/bin/id`. Comme nous le verrons plus loin dans ce module, si la variable PATH d'un utilisateur cible est mal configurée, nous pourrons peut-être en tirer parti pour élever les privilèges. Pour l'instant, nous allons le noter et l'ajouter à notre outil de prise de notes de choix.

```
dsgsec@htb[/htb]$ echo $PATH

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

```

Nous pouvons également vérifier toutes les variables d'environnement qui sont définies pour notre utilisateur actuel, nous pouvons avoir de la chance et trouver quelque chose de sensible comme un mot de passe. Nous allons noter cela et passer à autre chose.

```
dsgsec@htb[/htb]$ env

SHELL=/bin/bash
PWD=/home/htb-student
LOGNAME=htb-student
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/htb-student
LANG=en_US.UTF-8

<SNIP>

```

Notons ensuite la version du noyau. Nous pouvons effectuer des recherches pour voir si la cible exécute un noyau vulnérable (dont nous tirerons parti plus tard dans le module) qui possède un PoC d'exploit public connu. Nous pouvons le faire de plusieurs façons, une autre serait `cat /proc/version`mais nous utiliserons la `uname -a`commande.

```
dsgsec@htb[/htb]$ uname -a

Linux nixlpe02 5.4.0-122-generic #138-Ubuntu SMP Wed Jun 22 15:00:31 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

```

Nous pouvons ensuite rassembler des informations supplémentaires sur l'hôte lui-même, telles que le type/la version du processeur :

```
dsgsec@htb[/htb]$ lscpu

Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
Address sizes:                   43 bits physical, 48 bits virtual
CPU(s):                          2
On-line CPU(s) list:             0,1
Thread(s) per core:              1
Core(s) per socket:              2
Socket(s):                       1
NUMA node(s):                    1
Vendor ID:                       AuthenticAMD
CPU family:                      23
Model:                           49
Model name:                      AMD EPYC 7302P 16-Core Processor
Stepping:                        0
CPU MHz:                         2994.375
BogoMIPS:                        5988.75
Hypervisor vendor:               VMware

<SNIP>

```

Quels shells de connexion existent sur le serveur ? Notez-les et soulignez que Tmux et Screen sont disponibles pour nous.

```
dsgsec@htb[/htb]$ cat /etc/shells

# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen

```

Nous devons également vérifier si des défenses sont en place et nous pouvons énumérer toute information à leur sujet. Certaines choses à rechercher incluent:

-   [Bouclier exécutif](https://en.wikipedia.org/wiki/Exec_Shield)
-   [iptables](https://linux.die.net/man/8/iptables)
-   [AppArmor](https://apparmor.net/)
-   [SELinux](https://www.redhat.com/en/topics/linux/what-is-selinux)
-   [Fail2ban](https://github.com/fail2ban/fail2ban)
-   [Renifler](https://www.snort.org/faq/what-is-snort)
-   [Pare-feu simple (ufw)](https://wiki.ubuntu.com/UncomplicatedFirewall)

Souvent, nous n'aurons pas les privilèges d'énumérer les configurations de ces protections, mais savoir ce qui, le cas échéant, est en place peut nous aider à ne pas perdre de temps sur certaines tâches.

Ensuite, nous pouvons jeter un œil aux lecteurs et à tous les partages sur le système. Tout d'abord, nous pouvons utiliser la `lsblk`commande pour énumérer les informations sur les périphériques de bloc sur le système (disques durs, clés USB, lecteurs optiques, etc.). Si nous découvrons et pouvons monter un lecteur supplémentaire ou un système de fichiers non monté, nous pouvons trouver des fichiers, des mots de passe ou des sauvegardes sensibles qui peuvent être exploités pour augmenter les privilèges.

```
dsgsec@htb[/htb]$ lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0   55M  1 loop /snap/core18/1705
loop1                       7:1    0   69M  1 loop /snap/lxd/14804
loop2                       7:2    0   47M  1 loop /snap/snapd/16292
loop3                       7:3    0  103M  1 loop /snap/lxd/23339
loop4                       7:4    0   62M  1 loop /snap/core20/1587
loop5                       7:5    0 55.6M  1 loop /snap/core18/2538
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   19G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   18G  0 lvm  /
sr0                        11:0    1  908M  0 rom

```

La commande `lpstat`peut être utilisée pour rechercher des informations sur les imprimantes connectées au système. S'il y a des travaux d'impression actifs ou en file d'attente, pouvons-nous accéder à des informations sensibles ?

Nous devons également vérifier les lecteurs montés et les lecteurs non montés. Pouvons-nous monter un lecteur non monté et accéder à des données sensibles ? Pouvons -nous trouver n'importe quel type d'informations d'identification `fstab`pour les lecteurs montés en recherchant des mots courants tels que mot de passe, nom d'utilisateur, informations d'identification, etc.`/etc/fstab`

```
dsgsec@htb[/htb]$ cat /etc/fstab

# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/ubuntu-vg/ubuntu-lv during curtin installation
/dev/disk/by-id/dm-uuid-LVM-BdLsBLE4CvzJUgtkugkof4S0dZG7gWR8HCNOlRdLWoXVOba2tYUMzHfFQAP9ajul / ext4 defaults 0 0
# /boot was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/20b1770d-a233-4780-900e-7c99bc974346 /boot ext4 defaults 0 0

```

Vérifiez la table de routage en tapant `route`ou `netstat -rn`. Ici, nous pouvons voir quels autres réseaux sont disponibles via quelle interface.

```
dsgsec@htb[/htb]$ route

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 ens192
10.129.0.0      0.0.0.0         255.255.0.0     U     0      0        0 ens192

```

Dans un environnement de domaine, nous voudrons certainement vérifier `/etc/resolv.conf`si l'hôte est configuré pour utiliser le DNS interne, nous pourrons peut-être l'utiliser comme point de départ pour interroger l'environnement Active Directory.

Nous voudrons également vérifier la table arp pour voir avec quels autres hôtes la cible a communiqué.

```
dsgsec@htb[/htb]$ arp -a

_gateway (10.129.0.1) at 00:50:56:b9:b9:fc [ether] on ens192

```

L'énumération de l'environnement inclut également des informations sur les utilisateurs qui existent sur le système cible. En effet, les utilisateurs individuels sont souvent configurés lors de l'installation d'applications et de services pour limiter les privilèges du service. La raison en est de maintenir la sécurité du système lui-même. Parce que si un service s'exécute avec les privilèges les plus élevés ( `root`) et qu'il est contrôlé par un attaquant, l'attaquant a automatiquement les droits les plus élevés sur l'ensemble du système. Tous les utilisateurs du système sont stockés dans le `/etc/passwd`fichier. Le format nous donne quelques informations, telles que :

1.  Nom d'utilisateur
2.  Mot de passe
3.  ID utilisateur (UID)
4.  ID de groupe (GID)
5.  Informations sur l'ID utilisateur
6.  Répertoire d'accueil
7.  Coquille

#### Utilisateurs existants

  Utilisateurs existants

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
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
tcpdump:x:108:115::/nonexistent:/usr/sbin/nologin
mrb3n:x:1000:1000:mrb3n:/home/mrb3n:/bin/bash
bjones:x:1001:1001::/home/bjones:/bin/sh
administrator.ilfreight:x:1002:1002::/home/administrator.ilfreight:/bin/sh
backupsvc:x:1003:1003::/home/backupsvc:/bin/sh
cliff.moore:x:1004:1004::/home/cliff.moore:/bin/bash
logger:x:1005:1005::/home/logger:/bin/sh
shared:x:1006:1006::/home/shared:/bin/sh
stacey.jenkins:x:1007:1007::/home/stacey.jenkins:/bin/bash
htb-student:x:1008:1008::/home/htb-student:/bin/bash
<SNIP>

```

Parfois, nous verrons des hachages de mot de passe directement dans le `/etc/passwd`fichier. Ce fichier est lisible par tous les utilisateurs et, comme pour les hachages dans le `/etc/shadow`fichier, ceux-ci peuvent être soumis à une attaque de craquage de mot de passe hors ligne. Cette configuration, bien que peu courante, peut parfois être observée sur les périphériques et routeurs intégrés.

  Utilisateurs existants

```
dsgsec@htb[/htb]$ cat /etc/passwd | cut -f1 -d:

root
daemon
bin
sys

...SNIP...

mrb3n
lxd
bjones
administrator.ilfreight
backupsvc
cliff.moore
logger
shared
stacey.jenkins
htb-student

```

Avec Linux, plusieurs algorithmes de hachage différents peuvent être utilisés pour rendre les mots de passe méconnaissables. Les identifier dès les premiers blocs de hachage peut nous aider à les utiliser et à travailler avec eux plus tard si nécessaire. Voici une liste des plus utilisés :

| Algorithme | Hacher |
| --- | --- |
| MD5 salé | `$1$`... |
| SHA-256 | `$5$`... |
| SHA-512 | `$6$`... |
| BCrypt | `$2a$`... |
| Scrypter | `$7$`... |
| Argon2 | `$argon2i$`... |

Nous voudrons également vérifier quels utilisateurs ont des shells de connexion. Une fois que nous voyons quels shells sont sur le système, nous pouvons vérifier chaque version pour les vulnérabilités. Parce que les versions obsolètes, telles que Bash version 4.1, sont vulnérables à un `shellshock`exploit.

  Utilisateurs existants

```
dsgsec@htb[/htb]$ grep "*sh$" /etc/passwd

root:x:0:0:root:/root:/bin/bash
mrb3n:x:1000:1000:mrb3n:/home/mrb3n:/bin/bash
bjones:x:1001:1001::/home/bjones:/bin/sh
administrator.ilfreight:x:1002:1002::/home/administrator.ilfreight:/bin/sh
backupsvc:x:1003:1003::/home/backupsvc:/bin/sh
cliff.moore:x:1004:1004::/home/cliff.moore:/bin/bash
logger:x:1005:1005::/home/logger:/bin/sh
shared:x:1006:1006::/home/shared:/bin/sh
stacey.jenkins:x:1007:1007::/home/stacey.jenkins:/bin/bash
htb-student:x:1008:1008::/home/htb-student:/bin/bash

```

Chaque utilisateur des systèmes Linux est affecté à un ou plusieurs groupes spécifiques et reçoit ainsi des privilèges spéciaux. Par exemple, si nous avons un dossier nommé `dev`uniquement pour les développeurs, un utilisateur doit être affecté au groupe approprié pour accéder à ce dossier. Les informations sur les groupes disponibles se trouvent dans le `/etc/group`fichier, qui nous montre à la fois le nom du groupe et les noms d'utilisateur attribués.

#### Groupes existants

  Groupes existants

```
dsgsec@htb[/htb]$ cat /etc/group

root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:syslog,htb-student
tty:x:5:syslog
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:htb-student
floppy:x:25:
tape:x:26:
sudo:x:27:mrb3n,htb-student
audio:x:29:pulse
dip:x:30:htb-student
www-data:x:33:
...SNIP...

```

Le `/etc/group`fichier répertorie tous les groupes du système. Nous pouvons ensuite utiliser la commande [getent](https://man7.org/linux/man-pages/man1/getent.1.html) pour lister les membres de tous les groupes intéressants.

  Groupes existants

```
dsgsec@htb[/htb]$ getent group sudo

sudo:x:27:mrb3n

```

Nous pouvons également vérifier quels utilisateurs ont un dossier sous le `/home`répertoire. Nous voudrons énumérer chacun d'entre eux pour voir si l'un des utilisateurs du système stocke des données sensibles, des fichiers contenant des mots de passe. Nous devrions vérifier si des fichiers tels que le`.bash_history`sont lisibles et contiennent toutes les commandes intéressantes et recherchez les fichiers de configuration. Il n'est pas rare de trouver des fichiers contenant des informations d'identification qui peuvent être exploitées pour accéder à d'autres systèmes ou même accéder à l'environnement Active Directory. Il est également important de vérifier les clés SSH pour tous les utilisateurs, car elles pourraient être utilisées pour assurer la persistance sur le système, potentiellement pour augmenter les privilèges ou pour aider au pivotement et à la redirection de port plus loin dans le réseau interne. Au minimum, vérifiez le cache ARP pour voir à quels autres hôtes sont accédés et comparez-les avec toutes les clés privées SSH utilisables.

  Groupes existants

```
dsgsec@htb[/htb]$ ls /home

administrator.ilfreight  bjones       htb-student  mrb3n   stacey.jenkins
backupsvc                cliff.moore  logger       shared

```

Enfin, nous pouvons rechercher tous les "fruits à portée de main" tels que les fichiers de configuration et d'autres fichiers pouvant contenir des informations sensibles. Les fichiers de configuration peuvent contenir une mine d'informations. Il vaut la peine de rechercher dans tous les fichiers qui se terminent par des extensions telles que .conf et .config, les noms d'utilisateur, les mots de passe et d'autres secrets.

Si nous avons rassemblé des mots de passe, nous devrions les essayer maintenant pour tous les utilisateurs présents sur le système. La réutilisation des mots de passe est courante, nous pourrions donc avoir de la chance !

Sous Linux, il existe de nombreux endroits différents où ces fichiers peuvent être stockés, y compris les systèmes de fichiers montés. Un système de fichiers monté est un système de fichiers attaché à un répertoire particulier sur le système et accessible via ce répertoire. De nombreux systèmes de fichiers, tels que ext4, NTFS et FAT32, peuvent être montés. Chaque type de système de fichiers a ses propres avantages et inconvénients. Par exemple, certains systèmes de fichiers ne peuvent être lus que par le système d'exploitation, tandis que d'autres peuvent être lus et écrits par l'utilisateur. Les systèmes de fichiers qui peuvent être lus et écrits par l'utilisateur sont appelés systèmes de fichiers en lecture/écriture. Le montage d'un système de fichiers permet à l'utilisateur d'accéder aux fichiers et dossiers stockés sur ce système de fichiers. Pour monter un système de fichiers, l'utilisateur doit disposer des privilèges root. Une fois qu'un système de fichiers est monté, il peut être démonté par l'utilisateur disposant des privilèges root.

#### Systèmes de fichiers montés

  Systèmes de fichiers montés

```
dsgsec@htb[/htb]$ df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            1,9G     0  1,9G   0% /dev
tmpfs           389M  1,8M  388M   1% /run
/dev/sda5        20G  7,9G   11G  44% /
tmpfs           1,9G     0  1,9G   0% /dev/shm
tmpfs           5,0M  4,0K  5,0M   1% /run/lock
tmpfs           1,9G     0  1,9G   0% /sys/fs/cgroup
/dev/loop0      128K  128K     0 100% /snap/bare/5
/dev/loop1       62M   62M     0 100% /snap/core20/1611
/dev/loop2       92M   92M     0 100% /snap/gtk-common-themes/1535
/dev/loop4       55M   55M     0 100% /snap/snap-store/558
/dev/loop3      347M  347M     0 100% /snap/gnome-3-38-2004/115
/dev/loop5       47M   47M     0 100% /snap/snapd/16292
/dev/sda1       511M  4,0K  511M   1% /boot/efi
tmpfs           389M   24K  389M   1% /run/user/1000
/dev/sr0        3,6G  3,6G     0 100% /media/htb-student/Ubuntu 20.04.5 LTS amd64
/dev/loop6       50M   50M     0 100% /snap/snapd/17576
/dev/loop7       64M   64M     0 100% /snap/core20/1695
/dev/loop8       46M   46M     0 100% /snap/snap-store/599
/dev/loop9      347M  347M     0 100% /snap/gnome-3-38-2004/119

```

Lorsqu'un système de fichiers est démonté, il n'est plus accessible par le système. Cela peut être fait pour diverses raisons, par exemple lorsqu'un disque est retiré ou qu'un système de fichiers n'est plus nécessaire. Une autre raison peut être que les fichiers, scripts, documents et autres informations importantes ne doivent pas être montés et visualisés par un utilisateur standard. Par conséquent, si nous pouvons étendre nos privilèges à l' `root`utilisateur, nous pourrions monter et lire ces systèmes de fichiers nous-mêmes. Les systèmes de fichiers non montés peuvent être visualisés comme suit :

#### Systèmes de fichiers non montés

  Systèmes de fichiers non montés

```
dsgsec@htb[/htb]$ cat /etc/fstab | grep -v "#" | column -t

UUID=5bf16727-fcdf-4205-906c-0620aa4a058f  /          ext4  errors=remount-ro  0  1
UUID=BE56-AAE0                             /boot/efi  vfat  umask=0077         0  1
/swapfile                                  none       swap  sw                 0  0

```

De nombreux dossiers et fichiers sont cachés sur un système Linux afin qu'ils ne soient pas évidents et que toute modification accidentelle soit évitée. Pourquoi ces fichiers et dossiers sont cachés, il y a beaucoup plus de raisons que celles mentionnées jusqu'à présent. Néanmoins, nous devons être en mesure de localiser tous les fichiers et dossiers cachés car ils peuvent souvent contenir des informations sensibles, même si nous avons des autorisations en lecture seule.

#### Tous les fichiers cachés

  Tous les fichiers cachés

```
dsgsec@htb[/htb]$ find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student

-rw-r--r-- 1 htb-student htb-student 3771 Nov 27 11:16 /home/htb-student/.bashrc
-rw-rw-r-- 1 htb-student htb-student 180 Nov 27 11:36 /home/htb-student/.wget-hsts
-rw------- 1 htb-student htb-student 387 Nov 27 14:02 /home/htb-student/.bash_history
-rw-r--r-- 1 htb-student htb-student 807 Nov 27 11:16 /home/htb-student/.profile
-rw-r--r-- 1 htb-student htb-student 0 Nov 27 11:31 /home/htb-student/.sudo_as_admin_successful
-rw-r--r-- 1 htb-student htb-student 220 Nov 27 11:16 /home/htb-student/.bash_logout
-rw-rw-r-- 1 htb-student htb-student 162 Nov 28 13:26 /home/htb-student/.notes

```

#### Tous les répertoires cachés

  Tous les répertoires cachés

```
dsgsec@htb[/htb]$ find / -type d -name ".*" -ls 2>/dev/null

   684822      4 drwx------   3 htb-student htb-student     4096 Nov 28 12:32 /home/htb-student/.gnupg
   790793      4 drwx------   2 htb-student htb-student     4096 Okt 27 11:31 /home/htb-student/.ssh
   684804      4 drwx------  10 htb-student htb-student     4096 Okt 27 11:30 /home/htb-student/.cache
   790827      4 drwxrwxr-x   8 htb-student htb-student     4096 Okt 27 11:32 /home/htb-student/CVE-2021-3156/.git
   684796      4 drwx------  10 htb-student htb-student     4096 Okt 27 11:30 /home/htb-student/.config
   655426      4 drwxr-xr-x   3 htb-student htb-student     4096 Okt 27 11:19 /home/htb-student/.local
   524808      4 drwxr-xr-x   7 gdm         gdm             4096 Okt 27 11:19 /var/lib/gdm3/.cache
   544027      4 drwxr-xr-x   7 gdm         gdm             4096 Okt 27 11:19 /var/lib/gdm3/.config
   544028      4 drwxr-xr-x   3 gdm         gdm             4096 Aug 31 08:54 /var/lib/gdm3/.local
   524938      4 drwx------   2 colord      colord          4096 Okt 27 11:19 /var/lib/colord/.cache
     1408      2 dr-xr-xr-x   1 htb-student htb-student     2048 Aug 31 09:17 /media/htb-student/Ubuntu\ 20.04.5\ LTS\ amd64/.disk
   280101      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:31 /tmp/.font-unix
   262364      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:32 /tmp/.ICE-unix
   262362      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:32 /tmp/.X11-unix
   280103      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:31 /tmp/.Test-unix
   262830      4 drwxrwxrwt   2 root        root            4096 Nov 28 12:31 /tmp/.XIM-unix
   661820      4 drwxr-xr-x   5 root        root            4096 Aug 31 08:55 /usr/lib/modules/5.15.0-46-generic/vdso/.build-id
   666709      4 drwxr-xr-x   5 root        root            4096 Okt 27 11:18 /usr/lib/modules/5.15.0-52-generic/vdso/.build-id
   657527      4 drwxr-xr-x 170 root        root            4096 Aug 31 08:55 /usr/lib/debug/.build-id

```

De plus, trois dossiers par défaut sont destinés aux fichiers temporaires. Ces dossiers sont visibles par tous les utilisateurs et peuvent être lus. De plus, des journaux temporaires ou des sorties de script peuvent y être trouvés. Les deux `/tmp`et `/var/tmp`sont utilisés pour stocker temporairement des données. Cependant, la principale différence est la durée pendant laquelle les données sont stockées dans ces systèmes de fichiers. La durée de conservation des données `/var/tmp`est beaucoup plus longue que celle de l' `/tmp`annuaire. Par défaut, tous les fichiers et données stockés dans /var/tmp sont conservés jusqu'à 30 jours. Dans /tmp, en revanche, les données sont automatiquement supprimées au bout de dix jours.

De plus, tous les fichiers temporaires stockés dans le `/tmp`répertoire sont immédiatement supprimés au redémarrage du système. Par conséquent, le `/var/tmp`répertoire est utilisé par les programmes pour stocker temporairement les données qui doivent être conservées entre les redémarrages.

#### Fichiers temporaires

  Fichiers temporaires

```
dsgsec@htb[/htb]$ ls -l /tmp /var/tmp /dev/shm

/dev/shm:
total 0

/tmp:
total 52
-rw------- 1 htb-student htb-student    0 Nov 28 12:32 config-err-v8LfEU
drwx------ 3 root        root        4096 Nov 28 12:37 snap.snap-store
drwx------ 2 htb-student htb-student 4096 Nov 28 12:32 ssh-OKlLKjlc98xh
<SNIP>
drwx------ 2 htb-student htb-student 4096 Nov 28 12:37 tracker-extract-files.1000
drwx------ 2 gdm         gdm         4096 Nov 28 12:31 tracker-extract-files.125

/var/tmp:
total 28
drwx------ 3 root root 4096 Nov 28 12:31 systemd-private-7b455e62ec09484b87eff41023c4ca53-colord.service-RrPcyi
drwx------ 3 root root 4096 Nov 28 12:31 systemd-private-7b455e62ec09484b87eff41023c4ca53-ModemManager.service-4Rej9e
...SNIP...

```

* * * * *

Passer à autre chose
--------------------

Nous avons une première vue d'ensemble du terrain et (espérons-le) des points de données sensibles ou utiles qui peuvent nous aider à augmenter nos privilèges ou même à nous déplacer latéralement dans le réseau interne. Ensuite, nous nous concentrerons sur les autorisations et vérifierons quels répertoires, scripts, fichiers binaires, etc. nous pouvons lire et écrire avec nos privilèges d'utilisateur actuels.

Bien que nous nous concentrions sur l'énumération manuelle dans ce module, cela vaut la peine d'exécuter le script [linPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS) à ce stade d'une évaluation réelle afin que nous ayons autant de données à approfondir que possible. Souvent, nous pouvons trouver une solution facile, mais avoir cette sortie à portée de main peut parfois révéler des problèmes nuancés que notre énumération manuelle a manqués. Nous devrions, cependant, pratiquer notre énumération manuelle autant que possible et créer (et continuer à ajouter) notre propre aide-mémoire des commandes clés (et des alternatives pour différents systèmes d'exploitation Linux). Nous allons commencer à développer notre propre style, nos préférences de commande et même voir certains domaines que nous pouvons commencer à scripter nous-mêmes. Les outils sont excellents et ont leur place, mais là où beaucoup échouent, c'est d'être capable d'effectuer une tâche donnée lorsqu'un outil tombe en panne ou que nous ne pouvons pas le faire entrer dans le système.
