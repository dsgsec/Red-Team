Énumération des services et internes Linux
==========================================

* * * * *

Maintenant que nous avons creusé dans l'environnement et obtenu la configuration du terrain et découvert autant que possible nos autorisations d'utilisateur et de groupe en ce qui concerne les fichiers, les scripts, les fichiers binaires, les répertoires, etc., nous allons faire un pas en avant plus loin et examinez plus en profondeur les composants internes du système d'exploitation hôte. Dans cette phase, nous énumérerons les éléments suivants qui aideront à éclairer bon nombre des attaques abordées dans les sections ultérieures de ce module.

-   Quels services et applications sont installés ?

-   Quels services fonctionnent ?

-   Quelles prises sont utilisées ?

-   Quels utilisateurs, administrateurs et groupes existent sur le système ?

-   Qui est actuellement connecté ? Quels utilisateurs se sont récemment connectés ?

-   Quelles politiques de mot de passe, le cas échéant, sont appliquées sur l'hôte ?

-   L'hôte est-il joint à un domaine Active Directory ?

-   Quels types d'informations intéressantes pouvons-nous trouver dans les fichiers d'historique, de journal et de sauvegarde ?

-   Quels fichiers ont été modifiés récemment et à quelle fréquence ? Existe-t-il des modèles intéressants dans la modification de fichiers qui pourraient indiquer une tâche cron en cours d'utilisation que nous pourrions être en mesure de détourner ?

-   Informations d'adressage IP actuelles

-   Rien d'intéressant dans le `/etc/hosts`dossier ?

-   Existe-t-il des connexions réseau intéressantes vers d'autres systèmes dans le réseau interne ou même en dehors du réseau ?

-   Quels sont les outils installés sur le système dont nous pouvons tirer parti ? (Netcat, Perl, Python, Ruby, Nmap, tcpdump, gcc, etc.)

-   Pouvons-nous accéder au `bash_history`fichier pour tous les utilisateurs et pouvons-nous découvrir quelque chose d'intéressant à partir de leur historique de ligne de commande enregistré, comme les mots de passe ?

-   Y a-t-il des tâches Cron en cours d'exécution sur le système que nous pourrions être en mesure de détourner ?

À ce stade, nous souhaitons également collecter autant d'informations que possible sur le réseau. Quelle est notre adresse IP actuelle ? Le système a-t-il d'autres interfaces et, par conséquent, pourrait-il être utilisé pour pivoter vers un autre sous-réseau qui était auparavant inaccessible depuis notre hôte d'attaque ? Nous le faisons avec la `ip a`commande ou `ifconfig`, mais cette commande ne fonctionnera parfois pas sur certains systèmes si le paquet [net-tools](https://packages.ubuntu.com/search?keywords=net-tools) n'est pas présent.

* * * * *

Internes
--------

Lorsque nous parlons de `internals`, nous entendons la configuration interne et la façon de travailler, y compris les processus intégrés conçus pour accomplir des tâches spécifiques. Nous commençons donc par les interfaces à travers lesquelles notre système cible peut communiquer.

#### Interfaces réseau

  Interfaces réseau

```
dsgsec@htb[/htb]$ ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:50:56:b9:ed:2a brd ff:ff:ff:ff:ff:ff
    inet 10.129.203.168/16 brd 10.129.255.255 scope global dynamic ens192
       valid_lft 3092sec preferred_lft 3092sec
    inet6 dead:beef::250:56ff:feb9:ed2a/64 scope global dynamic mngtmpaddr
       valid_lft 86400sec preferred_lft 14400sec
    inet6 fe80::250:56ff:feb9:ed2a/64 scope link
       valid_lft forever preferred_lft forever

```

Y a-t-il quelque chose d'intéressant dans le `/etc/hosts`dossier ?

#### Hôtes

  Hôtes

```
dsgsec@htb[/htb]$ cat /etc/hosts

127.0.0.1 localhost
127.0.1.1 nixlpe02
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

Il peut également être utile de vérifier l'heure de la dernière connexion de chaque utilisateur pour essayer de voir quand les utilisateurs se connectent généralement au système et à quelle fréquence. Cela peut nous donner une idée de la large utilisation de ce système, ce qui peut ouvrir la possibilité à davantage de mauvaises configurations ou de répertoires ou d'historiques de commandes "désordonnés".

#### Dernière connexion de l'utilisateur

  Dernière connexion de l'utilisateur

```
dsgsec@htb[/htb]$ lastlog

Username         Port     From             Latest
root                                       **Never logged in**
daemon                                     **Never logged in**
bin                                        **Never logged in**
sys                                        **Never logged in**
sync                                       **Never logged in**
...SNIP...
systemd-coredump                           **Never logged in**
mrb3n            pts/1    10.10.14.15      Tue Aug  2 19:33:16 +0000 2022
lxd                                        **Never logged in**
bjones                                     **Never logged in**
administrator.ilfreight                           **Never logged in**
backupsvc                                  **Never logged in**
cliff.moore      pts/0    127.0.0.1        Tue Aug  2 19:32:29 +0000 2022
logger                                     **Never logged in**
shared                                     **Never logged in**
stacey.jenkins   pts/0    10.10.14.15      Tue Aug  2 18:29:15 +0000 2022
htb-student      pts/0    10.10.14.15      Wed Aug  3 13:37:22 +0000 2022

```

De plus, voyons si quelqu'un d'autre est actuellement sur le système avec nous. Il y a plusieurs façons de le faire, comme la `who`commande. La `finger`commande fonctionnera pour afficher ces informations sur certains systèmes Linux. Nous pouvons voir que l' `cliff.moore`utilisateur est connecté au système avec nous.

#### Utilisateurs connectés

  Utilisateurs connectés

```
dsgsec@htb[/htb]$ w

 12:27:21 up 1 day, 16:55,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
cliff.mo pts/0    10.10.14.16      Tue19   40:54m  0.02s  0.02s -bash

```

Il est également important de vérifier l'historique bash d'un utilisateur, car il peut transmettre des mots de passe en tant qu'argument sur la ligne de commande, travailler avec des référentiels git, configurer des tâches cron, etc. L'examen de ce que l'utilisateur a fait peut vous donner un aperçu considérable du type de serveur sur lequel vous atterrissez et vous donner un indice sur les chemins d'escalade privilégiés.

#### Historique des commandes

  Historique des commandes

```
dsgsec@htb[/htb]$ history

    1  id
    2  cd /home/cliff.moore
    3  exit
    4  touch backup.sh
    5  tail /var/log/apache2/error.log
    6  ssh ec2-user@dmz02.inlanefreight.local
    7  history

```

Parfois, nous pouvons également trouver des fichiers d'historique spéciaux créés par des scripts ou des programmes. Cela peut être trouvé, entre autres, dans des scripts qui surveillent certaines activités des utilisateurs et vérifient les activités suspectes.

#### Recherche de fichiers d'historique

  Recherche de fichiers d'historique

```
dsgsec@htb[/htb]$ find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null

-rw------- 1 htb-student htb-student 387 Nov 27 14:02 /home/htb-student/.bash_history

```

C'est aussi une bonne idée de vérifier les tâches cron sur le système. Les tâches Cron sur les systèmes Linux sont similaires aux tâches planifiées de Windows. Ils sont souvent configurés pour effectuer des tâches de maintenance et de sauvegarde. En conjonction avec d'autres erreurs de configuration telles que des chemins relatifs ou des autorisations faibles, ils peuvent tirer parti pour élever les privilèges lors de l'exécution de la tâche cron planifiée.

#### Cron

  Cron

```
dsgsec@htb[/htb]$ ls -la /etc/cron.daily/

total 48
drwxr-xr-x  2 root root 4096 Aug  2 17:36 .
drwxr-xr-x 96 root root 4096 Aug  2 19:34 ..
-rwxr-xr-x  1 root root  376 Dec  4  2019 apport
-rwxr-xr-x  1 root root 1478 Apr  9  2020 apt-compat
-rwxr-xr-x  1 root root  355 Dec 29  2017 bsdmainutils
-rwxr-xr-x  1 root root 1187 Sep  5  2019 dpkg
-rwxr-xr-x  1 root root  377 Jan 21  2019 logrotate
-rwxr-xr-x  1 root root 1123 Feb 25  2020 man-db
-rw-r--r--  1 root root  102 Feb 13  2020 .placeholder
-rwxr-xr-x  1 root root 4574 Jul 18  2019 popularity-contest
-rwxr-xr-x  1 root root  214 Apr  2  2020 update-notifier-common

```

Le [système de fichiers proc](https://man7.org/linux/man-pages/man5/proc.5.html) ( `proc`/ `procfs`) est un système de fichiers particulier sous Linux qui contient des informations sur les processus système, le matériel et d'autres informations système. C'est le principal moyen d'accéder aux informations de processus et peut être utilisé pour afficher et modifier les paramètres du noyau. Il est virtuel et n'existe pas en tant que système de fichiers réel mais est généré dynamiquement par le noyau. Il peut être utilisé pour rechercher des informations système telles que l'état des processus en cours d'exécution, les paramètres du noyau, la mémoire système et les périphériques. Il définit également certains paramètres système, tels que la priorité du processus, la planification et l'allocation de mémoire.

#### Proc

  Proc

```
dsgsec@htb[/htb]$ find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"

...SNIP...
startups/usr/lib/packagekit/packagekitd/usr/lib/packagekit/packagekitd/usr/lib/packagekit/packagekitd/usr/lib/packagekit/packagekitdroot@10.129.14.200sshroot@10.129.14.200sshd:
htb-student
[priv]sshd:
htb-student
[priv]/usr/bin/ssh-agent-D-a/run/user/1000/keyring/.ssh/usr/bin/ssh-agent-D-a/run/user/1000/keyring/.sshsshd:
htb-student@pts/2sshd:

```

* * * * *

Prestations de service
----------------------

S'il s'agit d'un système Linux légèrement plus ancien, il est plus probable que nous puissions trouver des packages installés qui peuvent déjà avoir au moins une vulnérabilité. Cependant, les versions actuelles des distributions Linux peuvent également avoir des packages ou des logiciels plus anciens installés qui peuvent avoir de telles vulnérabilités. Par conséquent, nous verrons une méthode pour nous aider à détecter les paquets potentiellement dangereux dans un instant. Pour ce faire, nous devons d'abord créer une liste des packages installés avec lesquels travailler.

#### Paquets installés

  Paquets installés

```
dsgsec@htb[/htb]$ apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list

Listing...
accountsservice-ubuntu-schemas 0.0.7+17.10.20170922-0ubuntu1
accountsservice 0.6.55-0ubuntu12~20.04.5
acl 2.2.53-6
acpi-support 0.143
acpid 2.0.32-1ubuntu1
adduser 3.118ubuntu2
adwaita-icon-theme 3.36.1-2ubuntu0.20.04.2
alsa-base 1.0.25+dfsg-0ubuntu5
alsa-topology-conf 1.2.2-1
alsa-ucm-conf 1.2.2-1ubuntu0.13
alsa-utils 1.2.2-1ubuntu2.1
amd64-microcode 3.20191218.1ubuntu1
anacron 2.3-29
apg 2.2.3.dfsg.1-5
app-install-data-partner 19.04
apparmor 2.13.3-7ubuntu5.1
apport-gtk 2.20.11-0ubuntu27.24
apport-symptoms 0.23
apport 2.20.11-0ubuntu27.24
appstream 0.12.10-2
apt-config-icons-hidpi 0.12.10-2
apt-config-icons 0.12.10-2
apt-utils 2.0.9
...SNIP...

```

C'est aussi une bonne idée de vérifier si la `sudo`version installée sur le système est vulnérable à des exploits hérités ou récents.

#### Version Sudo

  Version Sudo

```
dsgsec@htb[/htb]$ sudo -V

Sudo version 1.8.31
Sudoers policy plugin version 1.8.31
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.31

```

Parfois, il peut également arriver qu'aucun package direct ne soit installé sur le système, mais des programmes compilés sous forme de binaires. Ceux-ci ne nécessitent pas d'installation et peuvent être exécutés directement par le système lui-même.

#### Binaires

  Binaires

```
dsgsec@htb[/htb]$ ls -l /bin /usr/bin/ /usr/sbin/

lrwxrwxrwx 1 root root     7 Oct 27 11:14 /bin -> usr/bin

/usr/bin/:
total 175160
-rwxr-xr-x 1 root root       31248 May 19  2020  aa-enabled
-rwxr-xr-x 1 root root       35344 May 19  2020  aa-exec
-rwxr-xr-x 1 root root       22912 Apr 14  2021  aconnect
-rwxr-xr-x 1 root root       19016 Nov 28  2019  acpi_listen
-rwxr-xr-x 1 root root        7415 Oct 26  2021  add-apt-repository
-rwxr-xr-x 1 root root       30952 Feb  7  2022  addpart
lrwxrwxrwx 1 root root          26 Oct 20  2021  addr2line -> x86_64-linux-gnu-addr2line
...SNIP...

/usr/sbin/:
total 32500
-rwxr-xr-x 1 root root      3068 Mai 19  2020 aa-remove-unknown
-rwxr-xr-x 1 root root      8839 Mai 19  2020 aa-status
-rwxr-xr-x 1 root root       139 Jun 18  2019 aa-teardown
-rwxr-xr-x 1 root root     14728 Feb 25  2020 accessdb
-rwxr-xr-x 1 root root     60432 Nov 28  2019 acpid
-rwxr-xr-x 1 root root      3075 Jul  4 18:20 addgnupghome
lrwxrwxrwx 1 root root         7 Okt 27 11:14 addgroup -> adduser
-rwxr-xr-x 1 root root       860 Dez  7  2019 add-shell
-rwxr-xr-x 1 root root     37785 Apr 16  2020 adduser
-rwxr-xr-x 1 root root     69000 Feb  7  2022 agetty
-rwxr-xr-x 1 root root      5576 Jul 31  2015 alsa
-rwxr-xr-x 1 root root      4136 Apr 14  2021 alsabat-test
-rwxr-xr-x 1 root root    118176 Apr 14  2021 alsactl
-rwxr-xr-x 1 root root     26489 Apr 14  2021 alsa-info
-rwxr-xr-x 1 root root     39088 Jul 16  2019 anacron
...SNIP...

```

[GTFObins](https://gtfobins.github.io/) fournit une excellente plate-forme qui comprend une liste de fichiers binaires pouvant potentiellement être exploités pour élever nos privilèges sur le système cible. Avec le prochain oneliner, nous pouvons comparer les binaires existants avec ceux de GTFObins pour voir quels binaires nous devrions étudier plus tard.

#### GTFObins

  GTFObins

```
dsgsec@htb[/htb]$ for i in $(curl -s https://gtfobins.github.io/ | html2text | cut -d" " -f1 | sed '/^[[:space:]]*$/d');do if grep -q "$i" installed_pkgs.list;then echo "Check GTFO for: $i";fi;done

Check GTFO for: ab
Check GTFO for: apt
Check GTFO for: ar
Check GTFO for: as
Check GTFO for: ash
Check GTFO for: aspell
Check GTFO for: at
Check GTFO for: awk
Check GTFO for: bash
Check GTFO for: bridge
Check GTFO for: busybox
Check GTFO for: bzip2
Check GTFO for: cat
Check GTFO for: comm
Check GTFO for: cp
Check GTFO for: cpio
Check GTFO for: cupsfilter
Check GTFO for: curl
Check GTFO for: dash
Check GTFO for: date
Check GTFO for: dd
Check GTFO for: diff

```

Nous pouvons utiliser l'outil de diagnostic `strace`sur les systèmes d'exploitation basés sur Linux pour suivre et analyser les appels système et le traitement du signal. Il nous permet de suivre le flux d'un programme et de comprendre comment il accède aux ressources système, traite les signaux, reçoit et envoie des données du système d'exploitation. En outre, nous pouvons également utiliser l'outil pour surveiller les activités liées à la sécurité et identifier les vecteurs d'attaque potentiels, tels que les demandes spécifiques aux hôtes distants à l'aide de mots de passe ou de jetons.

La sortie de `strace`peut être écrite dans un fichier pour une analyse ultérieure, et il fournit une multitude d'options qui permettent une surveillance détaillée du comportement du programme.

#### Suivi des appels système

  Suivi des appels système

```
dsgsec@htb[/htb]$ strace ping -c1 10.129.112.20

execve("/usr/bin/ping", ["ping", "-c1", "10.129.112.20"], 0x7ffdc8b96cc0 /* 80 vars */) = 0
access("/etc/suid-debug", F_OK)         = -1 ENOENT (No such file or directory)
brk(NULL)                               = 0x56222584c000
arch_prctl(0x3001 /* ARCH_??? */, 0x7fffb0b2ea00) = -1 EINVAL (Invalid argument)
...SNIP...
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
...SNIP...
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libidn2.so.0", O_RDONLY|O_CLOEXEC) = 3
...SNIP...
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0 \0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0"..., 48, 848) = 48
...SNIP...
socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP) = 3
socket(AF_INET6, SOCK_DGRAM, IPPROTO_ICMPV6) = 4
capget({version=_LINUX_CAPABILITY_VERSION_3, pid=0}, NULL) = 0
capget({version=_LINUX_CAPABILITY_VERSION_3, pid=0}, {effective=0, permitted=0, inheritable=0}) = 0
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 5
...SNIP...
socket(AF_INET, SOCK_DGRAM, IPPROTO_IP) = 5
connect(5, {sa_family=AF_INET, sin_port=htons(1025), sin_addr=inet_addr("10.129.112.20")}, 16) = 0
getsockname(5, {sa_family=AF_INET, sin_port=htons(39885), sin_addr=inet_addr("10.129.112.20")}, [16]) = 0
close(5)                                = 0
...SNIP...
sendto(3, "\10\0\31\303\0\0\0\1eX\327c\0\0\0\0\330\254\n\0\0\0\0\0\20\21\22\23\24\25\26\27"..., 64, 0, {sa_family=AF_INET, sin_port=htons(0), sin_addr=inet_addr("10.129.112.20")}, 16) = 64
...SNIP...
recvmsg(3, {msg_name={sa_family=AF_INET, sin_port=htons(0), sin_addr=inet_addr("10.129.112.20")}, msg_namelen=128 => 16, msg_iov=[{iov_base="\0\0!\300\0\3\0\1eX\327c\0\0\0\0\330\254\n\0\0\0\0\0\20\21\22\23\24\25\26\27"..., iov_len=192}], msg_iovlen=1, msg_control=[{cmsg_len=32, cmsg_level=SOL_SOCKET, cmsg_type=SO_TIMESTAMP_OLD, cmsg_data={tv_sec=1675057253, tv_usec=699895}}, {cmsg_len=20, cmsg_level=SOL_IP, cmsg_type=IP_TTL, cmsg_data=[64]}], msg_controllen=56, msg_flags=0}, 0) = 64
write(1, "64 bytes from 10.129.112.20: icmp_se"..., 57) = 57
write(1, "\n", 1)                       = 1
write(1, "--- 10.129.112.20 ping statistics --"..., 34) = 34
write(1, "1 packets transmitted, 1 receive"..., 60) = 60
write(1, "rtt min/avg/max/mdev = 0.287/0.2"..., 50) = 50
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++

```

Les utilisateurs peuvent lire presque tous les fichiers de configuration sur un système d'exploitation Linux si l'administrateur les a conservés. Ces fichiers de configuration peuvent souvent révéler comment le service est configuré et configuré pour mieux comprendre comment nous pouvons l'utiliser à nos fins. De plus, ces fichiers peuvent contenir des informations sensibles, telles que des clés et des chemins vers des fichiers dans des dossiers que nous ne pouvons pas voir. Cependant, si le fichier a des autorisations de lecture pour tout le monde, nous pouvons toujours lire le fichier même si nous n'avons pas l'autorisation de lire le dossier.

#### Fichiers de configuration

  Fichiers de configuration

```
dsgsec@htb[/htb]$ find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null

-rw-r--r-- 1 root root 448 Nov 28 12:31 /run/tmpfiles.d/static-nodes.conf
-rw-r--r-- 1 root root 71 Nov 28 12:31 /run/NetworkManager/resolv.conf
-rw-r--r-- 1 root root 72 Nov 28 12:31 /run/NetworkManager/no-stub-resolv.conf
-rw-r--r-- 1 root root 0 Nov 28 12:37 /run/NetworkManager/conf.d/10-globally-managed-devices.conf
-rw-r--r-- 1 systemd-resolve systemd-resolve 736 Nov 28 12:31 /run/systemd/resolve/stub-resolv.conf
-rw-r--r-- 1 systemd-resolve systemd-resolve 607 Nov 28 12:31 /run/systemd/resolve/resolv.conf
...SNIP...

```

Les scripts sont similaires aux fichiers de configuration. Souvent, les administrateurs sont paresseux et convaincus de la sécurité du réseau et négligent la sécurité interne de leurs systèmes. Ces scripts, dans certains cas, ont de tels privilèges erronés que nous traiterons plus tard, mais le contenu est d'une grande importance même sans ces privilèges. Parce qu'à travers eux, nous pouvons découvrir des processus internes et individuels qui peuvent nous être d'une grande utilité.

#### Scénarios

  Scénarios

```
dsgsec@htb[/htb]$ find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"

/home/htb-student/automation.sh
/etc/wpa_supplicant/action_wpa.sh
/etc/wpa_supplicant/ifupdown.sh
/etc/wpa_supplicant/functions.sh
/etc/init.d/keyboard-setup.sh
/etc/init.d/console-setup.sh
/etc/init.d/hwclock.sh
...SNIP...

```

De plus, si nous regardons la liste des processus, cela peut nous donner des informations sur les scripts ou les binaires utilisés et par quel utilisateur. Ainsi, par exemple, s'il s'agit d'un script créé par l'administrateur dans son chemin et dont les droits n'ont pas été restreints, nous pouvons l'exécuter sans entrer dans le `root`répertoire.

#### Exécution des services par utilisateur

  Exécution des services par utilisateur

```
dsgsec@htb[/htb]$ ps aux | grep root

...SNIP...
root           1  2.0  0.2 168196 11364 ?        Ss   12:31   0:01 /sbin/init splash
root         378  0.5  0.4  62648 17212 ?        S<s  12:31   0:00 /lib/systemd/systemd-journald
root         409  0.8  0.1  25208  7832 ?        Ss   12:31   0:00 /lib/systemd/systemd-udevd
root         457  0.0  0.0 150668   284 ?        Ssl  12:31   0:00 vmware-vmblock-fuse /run/vmblock-fuse -o rw,subtype=vmware-vmblock,default_permissions,allow_other,dev,suid
root         752  0.0  0.2  58780 10608 ?        Ss   12:31   0:00 /usr/bin/VGAuthService
root         755  0.0  0.1 248088  7448 ?        Ssl  12:31   0:00 /usr/bin/vmtoolsd
root         772  0.0  0.2 250528  9388 ?        Ssl  12:31   0:00 /usr/lib/accountsservice/accounts-daemon
root         773  0.0  0.0   2548   768 ?        Ss   12:31   0:00 /usr/sbin/acpid
root         774  0.0  0.0  16720   708 ?        Ss   12:31   0:00 /usr/sbin/anacron -d -q -s
root         778  0.0  0.0  18052  2992 ?        Ss   12:31   0:00 /usr/sbin/cron -f
root         779  0.0  0.2  37204  8964 ?        Ss   12:31   0:00 /usr/sbin/cupsd -l
root         784  0.4  0.5 273512 21680 ?        Ssl  12:31   0:00 /usr/sbin/NetworkManager --no-daemon
root         790  0.0  0.0  81932  3648 ?        Ssl  12:31   0:00 /usr/sbin/irqbalance --foreground
root         792  0.1  0.5  48244 20540 ?        Ss   12:31   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root         793  1.3  0.2 239180 11832 ?        Ssl  12:31   0:00 /usr/lib/policykit-1/polkitd --no-debug
root         806  2.1  1.1 1096292 44976 ?       Ssl  12:31   0:01 /usr/lib/snapd/snapd
root         807  0.0  0.1 244352  6516 ?        Ssl  12:31   0:00 /usr/libexec/switcheroo-control
root         811  0.1  0.2  17412  8112 ?        Ss   12:31   0:00 /lib/systemd/systemd-logind
root         817  0.0  0.3 396156 14352 ?        Ssl  12:31   0:00 /usr/lib/udisks2/udisksd
root         818  0.0  0.1  13684  4876 ?        Ss   12:31   0:00 /sbin/wpa_supplicant -u -s -O /run/wpa_supplicant
root         871  0.1  0.3 319236 13828 ?        Ssl  12:31   0:00 /usr/sbin/ModemManager
root         875  0.0  0.3 178392 12748 ?        Ssl  12:31   0:00 /usr/sbin/cups-browsed
root         889  0.1  0.5 126676 22888 ?        Ssl  12:31   0:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root         906  0.0  0.2 248244  8736 ?        Ssl  12:31   0:00 /usr/sbin/gdm3
root        1137  0.0  0.2 252436  9424 ?        Ssl  12:31   0:00 /usr/lib/upower/upowerd
root        1257  0.0  0.4 293736 16316 ?        Ssl  12:31   0:00 /usr/lib/packagekit/packagekitd

```

* * * * *

Cela nous donnerait un assez bon aperçu de notre système cible, afin que nous puissions entrer plus en détail ensuite et déterminer les autorisations individuelles pour les composants que nous avons trouvés.

Serveurs VPN

Attention : A chaque "Switch", vos clés de connexion sont régénérées et vous devez retélécharger votre fichier de connexion VPN.

Toutes les instances de VM associées à l'ancien serveur VPN seront résiliées lors du passage à un nouveau serveur VPN.
