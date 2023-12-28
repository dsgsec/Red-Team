![bccf63d717b423189ffff7ec926c408e](https://github.com/dsgsec/Red-Team/assets/82456829/1f0a5db0-babf-45ac-9be5-db4a018dee0f)

Cette tâche se concentre sur l'énumération d'une machine Linux après avoir accédé à un shell, tel que`bash` . Bien que certaines commandes fournissent des informations sur plus d'un domaine, nous avons essayé de regrouper les commandes en quatre catégories en fonction des informations que nous espérons acquérir.

-   Système
-   Utilisateurs
-   La mise en réseau
-   Les services en cours d'exécution

Nous vous recommandons de cliquer sur « Démarrer AttackBox » et « Démarrer la machine » afin de pouvoir expérimenter et répondre aux questions à la fin de cette tâche.

### Système

Sur un système Linux , nous pouvons obtenir plus d'informations sur la distribution Linux et la version en recherchant des fichiers ou des liens se terminant`-release` par `/etc/`. L'exécution `ls /etc/*-release`nous aide à trouver de tels fichiers. Voyons à quoi ressemblent les choses sur un CentOS Linux .

Terminal

```
user@TryHackMe$ ls /etc/*-release
/etc/centos-release  /etc/os-release  /etc/redhat-release  /etc/system-release
$ cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
[...]
```

Essayons un système Fedora.

Terminal

```
user@TryHackMe$ ls /etc/*-release
/etc/fedora-release@  /etc/os-release@  /etc/redhat-release@  /etc/system-release@
$ cat /etc/os-release
NAME="Fedora Linux"
VERSION="36 (Workstation Edition)"
[...]
```

Nous pouvons trouver le nom du système en utilisant la commande `hostname`.

Terminal

```
user@TryHackMe$ hostname
rpm-red-enum.thm
```

Divers fichiers sur un système peuvent fournir de nombreuses informations utiles. En particulier, considérez les éléments suivants `/etc/passwd`, `/etc/group`, et `/etc/shadow`. Tout utilisateur peut lire les fichiers `passwd`et `group`. Cependant, le `shadow`fichier de mots de passe nécessite les privilèges root car il contient les mots de passe hachés. Si vous parvenez à casser les hachages, vous connaîtrez le mot de passe d'origine de l'utilisateur.

Terminal

```
user@TryHackMe$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
[...]
michael:x:1001:1001::/home/michael:/bin/bash
peter:x:1002:1002::/home/peter:/bin/bash
jane:x:1003:1003::/home/jane:/bin/bash
randa:x:1004:1004::/home/randa:/bin/bash

$ cat /etc/group
root:x:0:
[...]
michael:x:1001:
peter:x:1002:
jane:x:1003:
randa:x:1004:

$ sudo cat /etc/shadow
root:$6$pZlRFi09$qqgNBS.00qtcUF9x0yHetjJbXsw0PAwQabpCilmAB47ye3OzmmJVfV6DxBYyUoWBHtTXPU0kQEVUQfPtZPO3C.:19131:0:99999:7:::
[...]
michael:$6$GADCGz6m$g.ROJGcSX/910DEipiPjU6clo6Z6/uBZ9Fvg3IaqsVnMA.UZtebTgGHpRU4NZFXTffjKPvOAgPKbtb2nQrVU70:19130:0:99999:7:::
peter:$6$RN4fdNxf$wvgzdlrIVYBJjKe3s2eqlIQhvMrtwAWBsjuxL5xMVaIw4nL9pCshJlrMu2iyj/NAryBmItFbhYAVznqRcFWIz1:19130:0:99999:7:::
jane:$6$Ees6f7QM$TL8D8yFXVXtIOY9sKjMqJ7BoHK1EHEeqM5dojTaqO52V6CPiGq2W6XjljOGx/08rSo4QXsBtLUC3PmewpeZ/Q0:19130:0:99999:7:::
randa:$6$dYsVoPyy$WR43vaETwoWooZvR03AZGPPKxjrGQ4jTb0uAHDy2GqGEOZyXvrQNH10tGlLIHac7EZGV8hSIfuXP0SnwVmnZn0:19130:0:99999:7:::
```

De même, divers répertoires peuvent révéler des informations sur les utilisateurs et contenir des fichiers sensibles ; l'un est les répertoires de messagerie trouvés sur `/var/mail/`.

Terminal

```
user@TryHackMe$ ls -lh /var/mail/
total 4.0K
-rw-rw----. 1 jane      mail   0 May 18 14:15 jane
-rw-rw----. 1 michael   mail   0 May 18 14:13 michael
-rw-rw----. 1 peter     mail   0 May 18 14:14 peter
-rw-rw----. 1 randa     mail   0 May 18 14:15 randa
-rw-------. 1 root      mail 639 May 19 07:37 root
```

Pour retrouver les applications installées vous pouvez envisager de lister les fichiers dans `/usr/bin/`et `/sbin/`:

-   `ls -lh /usr/bin/`
-   `ls -lh /sbin/`

Sur un système Linux basé sur RPM , vous pouvez obtenir une liste de tous les packages installés à l'aide de`rpm -qa` . Le `-qa`indique que nous voulons *interroger tous* les packages.

Sur un système Linux basé sur Debian , vous pouvez obtenir la liste des packages installés en utilisant`dpkg -l` . La sortie ci-dessous est obtenue à partir d'un serveur Ubuntu.

Terminal

```
user@TryHackMe$ dpkg -l
Desired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                  Version                            Architecture Description
+++-=====================================-==================================-============-===============================================================================
ii  accountsservice                       0.6.55-0ubuntu12~20.04.5           amd64        query and manipulate user account information
ii  adduser                               3.118ubuntu2                       all          add and remove users and groups
ii  alsa-topology-conf                    1.2.2-1                            all          ALSA topology configuration files
ii  alsa-ucm-conf                         1.2.2-1ubuntu0.13                  all          ALSA Use Case Manager configuration files
ii  amd64-microcode                       3.20191218.1ubuntu1                amd64        Processor microcode firmware for AMD CPUs
[...   ]
ii  zlib1g-dev:amd64                      1:1.2.11.dfsg-2ubuntu1.3           amd64        compression library - development
```

### Utilisateurs

Des fichiers tels que `/etc/passwd`révéler les noms d'utilisateur ; cependant, diverses commandes peuvent fournir plus d'informations et d'informations sur les autres utilisateurs du système et leur localisation.

Vous pouvez montrer qui est connecté en utilisant `who`.

Terminal

```
user@TryHackMe$ who
root     tty1         2022-05-18 13:24
jane     pts/0        2022-05-19 07:17 (10.20.30.105)
peter    pts/1        2022-05-19 07:13 (10.20.30.113)
```

Nous pouvons voir que l'utilisateur `root`est connecté directement au système, tandis que les utilisateurs `jane`sont `peter`connectés via le réseau, et nous pouvons voir leurs adresses IP.

Notez que cela `who`ne doit pas être confondu avec `whoami`celui qui imprime votre identifiant utilisateur effectif.

Terminal

```
user@TryHackMe$ whoami
jane
```

Pour passer au niveau supérieur, vous pouvez utiliser `w`, qui montre qui est connecté et ce qu'il fait. Sur la base de la sortie du terminal ci-dessous, `peter`c'est l'édition `notes.txt`et `jane`c'est celui qui s'exécute `w`dans cet exemple.

Terminal

```
user@TryHackMe$ w
 07:18:43 up 18:05,  3 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1                      Wed13   17:52m  0.00s  0.00s less -s
jane     pts/0    10.20.30.105     07:17    3.00s  0.01s  0.00s w
peter    pts/1    10.20.30.113     07:13    5:23   0.00s  0.00s vi notes.txt
```

Pour imprimer les identifiants réels et effectifs des utilisateurs et des groupes , vous pouvez émettre la commande`id` (pour ID).

Terminal

```
user@TryHackMe$ id
uid=1003(jane) gid=1003(jane) groups=1003(jane) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Voulez-vous savoir qui a utilisé le système récemment ? `last`affiche une liste des derniers utilisateurs connectés ; de plus, nous pouvons voir qui s'est déconnecté et combien de temps ils sont restés connectés. Dans le résultat ci-dessous, l'utilisateur `randa`est resté connecté pendant près de 17 heures, tandis qu'il `michael`s'est déconnecté après quatre minutes.

Terminal

```
user@TryHackMe$ last
jane     pts/0        10.20.30.105     Thu May 19 07:17   still logged in
peter    pts/1        10.20.30.113     Thu May 19 07:13   still logged in
michael  pts/0        10.20.30.1       Thu May 19 05:12 - 05:17  (00:04)
randa    pts/1        10.20.30.107     Wed May 18 14:18 - 07:08  (16:49)
root     tty1                          Wed May 18 13:24   still logged in
[...]
```

Enfin, il convient de mentionner que `sudo -l`répertorie les commandes autorisées pour l'utilisateur appelant sur le système actuel.

### La mise en réseau

Les adresses IP peuvent être affichées en utilisant `ip address show`(qui peut être raccourci en `ip a s`) ou avec l'ancienne commande `ifconfig -a`(son package n'est plus maintenu.) La sortie du terminal ci-dessous montre l'interface réseau `ens33`avec l'adresse IP `10.20.30.129`et le masque de sous-réseau `255.255.255.0`tels quels `24`.

Terminal

```
user@TryHackMe$ ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 00:0c:29:a2:0e:7e brd ff:ff:ff:ff:ff:ff
    inet 10.20.30.129/24 brd 10.20.30.255 scope global noprefixroute dynamic ens33
       valid_lft 1580sec preferred_lft 1580sec
    inet6 fe80::761a:b360:78:26cd/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

Les serveurs DNS se trouvent dans le fichier `/etc/resolv.conf`. Considérez la sortie de terminal suivante pour un système qui utilise DHCP pour ses configurations réseau. Le DNS, c'est-à-dire le serveur de noms, est défini sur`10.20.30.2` .

Terminal

```
user@TryHackMe$ cat /etc/resolv.conf
# Generated by NetworkManager
search localdomain thm
nameserver 10.20.30.2
```

`netstat`est une commande utile pour en savoir plus sur les connexions réseau, les tables de routage et les statistiques d'interface. Nous expliquons certaines de ses nombreuses options dans le tableau ci-dessous.

| Option | Description |
| --- | --- |
| `-a` | afficher les sockets d'écoute et de non-écoute |
| `-l` | afficher uniquement les prises d'écoute |
| `-n` | afficher une sortie numérique au lieu de résoudre l'adresse IP et le numéro de port |
| `-t` | TCP |
| `-u` | UDP |
| `-x` | UNIX |
| `-p` | Afficher le PID et le nom du programme auquel appartient le socket |

Vous pouvez utiliser n'importe quelle combinaison adaptée à vos besoins. Par exemple, `netstat -plt`renverra *les programmes en écoute sur les sockets TCP* . Comme nous pouvons le voir dans la sortie du terminal ci-dessous, `sshd`écoute sur le port SSH , tandis `master`qu'il écoute sur le port SMTP sur les adresses IPv4 et IPv6. Notez que pour obtenir tous les PID (ID de processus) et les noms de programmes, vous devez exécuter`netstat` en tant que root ou utiliser `sudo netstat`.

Terminal

```
user@TryHackMe$ sudo netstat -plt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN      978/sshd
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN      1141/master
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN      978/sshd
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN      1141/master
```

`netstat -atupn`affichera *toutes les connexions TCP et UDP* d'écoute et établies ainsi que les noms *des programmes* avec les adresses et les ports au format *numérique* .

Terminal

```
user@TryHackMe$ sudo netstat -atupn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      978/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1141/master
tcp        0      0 10.20.30.129:22         10.20.30.113:38822        ESTABLISHED 5665/sshd: peter [p
tcp        0      0 10.20.30.129:22         10.20.30.105:38826        ESTABLISHED 5723/sshd: jane [pr
tcp6       0      0 :::22                   :::*                    LISTEN      978/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN      1141/master
udp        0      0 127.0.0.1:323           0.0.0.0:*                           640/chronyd
udp        0      0 0.0.0.0:68              0.0.0.0:*                           5638/dhclient
udp6       0      0 ::1:323                 :::*                                640/chronyd
```

On pourrait penser qu'utiliser `nmap`avant d'accéder à la machine cible aurait donné un résultat comparable. Cependant, ce n'est pas entièrement vrai. Nmap doit générer un nombre relativement important de paquets pour vérifier les ports ouverts, ce qui peut déclencher des systèmes de détection et de prévention des intrusions. De plus, les pare-feu sur la route peuvent abandonner certains paquets et gêner l'analyse, ce qui entraîne des résultats Nmap incomplets.

`lsof`signifie Liste des fichiers ouverts. Si nous voulons afficher uniquement les connexions Internet et réseau, nous pouvons utiliser `lsof -i`. La sortie du terminal ci-dessous montre les services d'écoute IPv4 et IPv6 et les connexions en cours. L'utilisateur `peter`est connecté au serveur `rpm-red-enum.thm`sur le `ssh`port. Notez que pour obtenir la liste complète des programmes correspondants, vous devez exécuter `lsof`en tant que root ou utiliser `sudo lsof`.

Terminal

```
user@TryHackMe$ sudo lsof -i
COMMAND   PID      USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
chronyd   640    chrony    5u  IPv4  16945      0t0  UDP localhost:323
chronyd   640    chrony    6u  IPv6  16946      0t0  UDP localhost:323
sshd      978      root    3u  IPv4  20035      0t0  TCP *:ssh (LISTEN)
sshd      978      root    4u  IPv6  20058      0t0  TCP *:ssh (LISTEN)
master   1141      root   13u  IPv4  20665      0t0  TCP localhost:smtp (LISTEN)
master   1141      root   14u  IPv6  20666      0t0  TCP localhost:smtp (LISTEN)
dhclient 5638      root    6u  IPv4  47458      0t0  UDP *:bootpc
sshd     5693     peter    3u  IPv4  47594      0t0  TCP rpm-red-enum.thm:ssh->10.20.30.113:38822 (ESTABLISHED)
[...]
```

La liste pouvant être assez longue, vous pouvez filtrer davantage la sortie en spécifiant les ports qui vous intéressent, tels que le port SMTP 25. En exécutant`lsof -i :25` , nous limitons la sortie à ceux liés au port 25, comme indiqué dans la sortie du terminal ci-dessous. . Le serveur écoute sur le port 25 sur les adresses IPv4 et IPv6.

Terminal

```
user@TryHackMe$ sudo lsof -i :25
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
master  1141 root   13u  IPv4  20665      0t0  TCP localhost:smtp (LISTEN)
master  1141 root   14u  IPv6  20666      0t0  TCP localhost:smtp (LISTEN)
```

### Les services en cours d'exécution

Obtenir un instantané des processus en cours peut fournir de nombreuses informations. `ps`vous permet de découvrir les processus en cours et de nombreuses informations à leur sujet.

Vous pouvez répertorier tous les processus du système à l'aide de `ps -e`, où `-e`sélectionne tous les processus. Pour plus d'informations sur le processus, vous pouvez ajouter `-f`pour le format complet et `-l`pour le format long. Expérimentez avec `ps -e`, `ps -ef`, et `ps -el`.

Vous pouvez obtenir un résultat comparable et voir tous les processus en utilisant la syntaxe BSD : `ps ax`ou `ps aux`. Notez que `a`et `x`sont nécessaires lors de l'utilisation de la syntaxe BSD car ils lèvent les restrictions « uniquement vous-même » et « doit avoir un tty » ; en d'autres termes, il devient possible d'afficher tous les processus. Il `u`s'agit de détails sur l'utilisateur qui dispose du processus.

| Option | Description |
| --- | --- |
| `-e` | tous les processus |
| `-f` | liste en format complet |
| `-j` | format d'emploi |
| `-l` | format long |
| `-u` | format orienté utilisateur |

Pour une sortie plus « visuelle », vous pouvez imprimer `ps axjf`une arborescence de processus. Le `f`signifie « forêt » et crée une hiérarchie de processus artistiques ASCII, comme indiqué dans la sortie du terminal ci-dessous.

Terminal

```
user@TryHackMe$ ps axf
   PID TTY      STAT   TIME COMMAND
     2 ?        S      0:00 [kthreadd]
     4 ?        S<     0:00  \_ [kworker/0:0H]
     5 ?        S      0:01  \_ [kworker/u256:0]
[...]
   978 ?        Ss     0:00 /usr/sbin/sshd -D
  5665 ?        Ss     0:00  \_ sshd: peter [priv]
  5693 ?        S      0:00  |   \_ sshd: peter@pts/1
  5694 pts/1    Ss     0:00  |       \_ -bash
  5713 pts/1    S+     0:00  |           \_ vi notes.txt
  5723 ?        Ss     0:00  \_ sshd: jane [priv]
  5727 ?        S      0:00      \_ sshd: jane@pts/0
  5728 pts/0    Ss     0:00          \_ -bash
  7080 pts/0    R+     0:00              \_ ps axf
   979 ?        Ssl    0:12 /usr/bin/python2 -Es /usr/sbin/tuned -l -P
   981 ?        Ssl    0:07 /usr/sbin/rsyslogd -n
  1141 ?        Ss     0:00 /usr/libexec/postfix/master -w
  1147 ?        S      0:00  \_ qmgr -l -t unix -u
  6991 ?        S      0:00  \_ pickup -l -t unix -u
  1371 ?        Ss     0:00 login -- root
  1376 tty1     Ss     0:00  \_ -bash
  1411 tty1     S+     0:00      \_ man man
  1420 tty1     S+     0:00          \_ less -s
[...]
```

Pour résumer, pensez à utiliser `ps -ef`ou `ps aux`pour obtenir la liste de tous les processus en cours. Pensez à rediriger la sortie via `grep`pour afficher les lignes de sortie avec certains mots. La sortie du terminal ci-dessous montre les lignes qui `peter`les contiennent.

Terminal

```
user@TryHackMe$ ps -ef | grep peter
root       5665    978  0 07:11 ?        00:00:00 sshd: peter [priv]
peter      5693   5665  0 07:13 ?        00:00:00 sshd: peter@pts/1
peter      5694   5693  0 07:13 pts/1    00:00:00 -bash
peter      5713   5694  0 07:13 pts/1    00:00:00 vi notes.txt
```
