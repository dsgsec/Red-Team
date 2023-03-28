Débordement de tampon basé sur la pile
===========================

* * * * *

Les exceptions mémoire sont la réaction du système d'exploitation à une erreur dans un logiciel existant ou lors de l'exécution de ceux-ci. Ceci est responsable de la plupart des vulnérabilités de sécurité dans les flux de programmes au cours de la dernière décennie. Des erreurs de programmation se produisent souvent, entraînant des débordements de mémoire tampon dus à l'inattention lors de la programmation avec des langages peu abstraits tels que `C` ou `C++`.

Ces langages sont compilés presque directement en code machine et, contrairement aux langages hautement abstraits tels que Java ou Python, fonctionnent avec peu ou pas de système d'exploitation à structure de contrôle. Les débordements de tampon sont des erreurs qui permettent à des données trop volumineuses de tenir dans un tampon de la mémoire du système d'exploitation qui n'est pas assez grand, faisant ainsi déborder ce tampon. Suite à cette mauvaise manipulation, la mémoire d'autres fonctions du programme exécuté est écrasée, créant potentiellement une faille de sécurité.

Un tel programme (fichier binaire), est un fichier exécutable général stocké sur un support de stockage de données. Il existe plusieurs formats de fichiers différents pour ces fichiers binaires exécutables. Par exemple, le `Portable Executable Format` (`PE`) est utilisé sur les plates-formes Microsoft.

Un autre format pour les fichiers exécutables est le `Executable and Linking Format` (`ELF`), pris en charge par presque toutes les variantes `UNIX` modernes. Si l'éditeur de liens charge un tel fichier binaire exécutable et que le programme sera exécuté, le code de programme correspondant sera chargé dans la mémoire principale puis exécuté par le CPU.

Les programmes stockent des données et des instructions en mémoire pendant l'initialisation et l'exécution. Ce sont des données qui sont affichées dans le logiciel exécuté ou saisies par l'utilisateur. En particulier pour les entrées utilisateur attendues, un tampon doit être créé au préalable en enregistrant l'entrée.

Les instructions sont utilisées pour modéliser le déroulement du programme. Entre autres choses, les adresses de retour sont stockées dans la mémoire, qui se réfère à d'autres adresses mémoire et définit ainsi le flux de contrôle du programme. Si une telle adresse de retour est délibérément écrasée en utilisant un débordement de tampon, un attaquant peut manipuler le déroulement du programme en faisant référence à l'adresse de retour à une autre fonction ou sous-routine. De plus, il serait possible de revenir à un code précédemment introduit par l'entrée de l'utilisateur.

Pour comprendre comment cela fonctionne sur le plan technique, nous devons nous familiariser avec comment :

- la mémoire est divisée et utilisée
- le débogueur affiche et nomme les instructions individuelles
- le débogueur peut être utilisé pour détecter de telles vulnérabilités
- nous pouvons manipuler la mémoire

Un autre point critique est que les exploits ne fonctionnent généralement que pour une version spécifique du logiciel et du système d'exploitation. Par conséquent, nous devons reconstruire et reconfigurer le système cible pour le ramener au même état. Après cela, le programme que nous étudions est installé et analysé. La plupart du temps, nous n'aurons qu'une seule tentative d'exploitation du programme si nous manquons l'occasion de le redémarrer avec des privilèges élevés.

* * * * *

La mémoire
----------

Lorsque le programme est appelé, les sections sont mappées sur les segments du processus, et les segments sont chargés en mémoire comme décrit par le fichier `ELF`.

#### Amortir

![image](https://academy.hackthebox.com/storage/modules/31/buffer_overflow_1.png)

#### .texte

La section `.text` contient les instructions d'assemblage réelles du programme. Cette zone peut être en lecture seule pour empêcher le processus de modifier accidentellement ses instructions. Toute tentative d'écriture dans cette zone entraînera inévitablement un défaut de segmentation.

* * * * *

#### .données

La section `.data` contient des variables globales et statiques explicitement initialisées par le programme.

* * * * *

#### .bss

Plusieurs compilateurs et éditeurs de liens utilisent la section `.bss` dans le cadre du segment de données, qui contient des variables allouées statiquement représentées exclusivement par des bits 0.

* * * * *

#### Le tas

La `mémoire de tas` est allouée à partir de cette zone. Cette zone commence à la fin du segment ".bss" et s'étend jusqu'aux adresses mémoire supérieures.

* * * * *

#### La pile

La `mémoire de la pile` est une structure de données `Last-In-First-Out` dans laquelle les adresses de retour, les paramètres et, selon les options du compilateur, les pointeurs de trame sont stockés. Les variables locales `C/C++` sont stockées ici, et vous pouvez même copier du code dans la pile. La `pile` est une zone définie dans la `RAM`. L'éditeur de liens réserve cette zone et place généralement la pile dans la zone inférieure de la RAM au-dessus des variables globales et statiques. Le contenu est accessible via le `pointeur de pile`, défini sur l'extrémité supérieure de la pile lors de l'initialisation. Pendant l'exécution, la partie allouée de la pile croît jusqu'aux adresses mémoire inférieures.

Les protections de mémoire modernes (`DEP`/`ASLR`) empêcheraient les dommages causés par les débordements de tampon. DEP (Data Execution Prevention), zones de mémoire marquées "Lecture seule". Les régions de mémoire en lecture seule sont l'endroit où certaines entrées de l'utilisateur sont stockées (exemple : la pile), donc l'idée derrière DEP était d'empêcher les utilisateurs de télécharger le shellcode en mémoire, puis de définir le ipointeur d'instruction vers le shellcode. Les pirates ont commencé à utiliser ROP (Return Oriented Programming) pour contourner ce problème, car cela leur permettait de télécharger le shellcode dans un espace exécutable et d'utiliser les appels existants pour l'exécuter. Avec ROP, l'attaquant a besoin de connaître les adresses mémoire où les choses sont stockées, donc la défense contre cela était d'implémenter ASLR (Address Space Layout Randomization) qui randomise où tout est stocké, ce qui rend ROP plus difficile.

Les utilisateurs peuvent contourner l'ASLR en divulguant des adresses mémoire, mais cela rend les exploits moins fiables et parfois impossibles. Par exemple, le ["Freefloat FTP Server" ](https://www.exploit-db.com/exploits/46763) est facile à exploiter sous Windows XP (avant DEP/ASLR). Cependant, si l'application est exécutée sur un système d'exploitation Windows moderne, le débordement de la mémoire tampon existe, mais il n'est actuellement pas facile à exploiter en raison de DEP/ASLR car il n'existe aucun moyen connu de fuir les adresses mémoire.

* * * * *

Programme vulnérable
------------------

Nous écrivons maintenant un programme C simple appelé `bow.c` avec une fonction vulnérable appelée `strcpy()`.

#### Arc.c

Code : c

```
#include <stdlib.h>
#include <stdio.h>
#include <chaîne.h>

int bowfunc(char *string) {

tampon de caractères [1024] ;
strcpy(tampon, chaîne);
retour 1 ;
}

int main(int argc, char *argv[]) {

fonction arc(argv[1]);
printf("Terminé.\n");
retour 1 ;
}

```

Les systèmes d'exploitation modernes ont des protections intégrées contre de telles vulnérabilités, comme la randomisation de la disposition de l'espace d'adresse (ASLR). Dans le but d'apprendre les bases de l'exploitation du débordement de tampon, nous allons désactiver ces fonctionnalités de protection de la mémoire :

#### Désactiver l'ASLR

Désactiver l'ASLR

```
étudiant@nix-bow :~$ sudo su
root@nix-bow:/home/student# echo 0 > /proc/sys/kernel/randomize_va_space
root@nix-bow:/home/student# cat /proc/sys/kernel/randomize_va_space

0

```

Ensuite, nous compilons le code C dans un binaire ELF 32 bits.

#### Compilation

Compilation

```
étudiant@nix-bow:~$ gcc bow.c -o bow32 -fno-stack-protector -z execstack -m32
étudiant@nix-bow:~$ fichier bow32 | tr "," "\n"

arc : objet partagé ELF 32 bits LSB
  Intel 80386
  version 1 (SYSV)
  lié dynamiquement
  interpréteur /lib/ld-linux.so.2
  pour GNU/Linux 3.2.0
  ID de construction[sha1]=93dda6b77131deecaadf9d207fdd2e70f47e1071
  pas dépouillé

```

* * * * *

Fonctions C vulnérables
----------------------

Il existe plusieurs fonctions vulnérables dans le langage de programmation C qui ne protègent pas indépendamment la mémoire. Voici quelques-unes des fonctions :

- `strcpy`
- `obtient`
- `sprintf`
- `scanf`
- `strcat`
- ...

* * * * *

Présentations GDB
-----------------

GDB, ou GNU Debugger, est le débogueur standard des systèmes Linux développé par le projet GNU. Il a été porté sur de nombreux systèmes et prend en charge les langages de programmation C, C++, Objective-C, FORTRAN, Java et bien d'autres.

GDB nous fournit les fonctionnalités de traçabilité habituelles comme les points d'arrêt ou la sortie de trace de pile et nous permet d'intervenir dans l'exécution des programmes. Cela nous permet aussi, par exemple, de manipuler les variables de l'application ou d'appeler des fonctions indépendamment de l'exécution normale du programme.

Nous utilisons `GNU Debugger` (`GDB`) pour afficher le binaire créé au niveau de l'assembleur. Une fois que nous avons exécuté le binaire avec `GDB`, nous pouvons désassembler la fonction principale du programme.

#### GDB - Syntaxe AT&T

GDB - Syntaxe AT&T

```
étudiant@nix-bow :~$ gdb -q bow32

Lecture des symboles de l'arc... (aucun symbole de débogage trouvé)... fait.
(gdb) démonter principal

Vidage du code assembleur pour la fonction main :
    0x00000582 <+0> : lea 0x4(%esp),%ecx
    0x00000586 <+4> : et $0xfffffff0,%esp
    0x00000589 <+7> : appuyez sur -0x4(%ecx)
    0x0000058c <+10> : poussez %ebp
    0x0000058d <+11> : mouvement %esp,%ebp
    0x0000058f <+13> : poussez %ebx
    0x00000590 <+14> : poussez %ecx
    0x00000591 <+15> : appelez 0x450 <__x86.get_pc_thunk.bx>
    0x00000596 <+20> : ajoutez $0x1a3e,%ebx
    0x0000059c <+26> : mouvement %ecx,%eax
    0x0000059e <+28> : mov 0x4(%eax),%eax
    0x000005a1 <+31> : ajouter $0x4,%eax
    0x000005a4 <+34> : mouvement (%eax),%eax
    0x000005a6 <+36> : sous $0xc,%esp
    0x000005a9 <+39> : appuyez sur %eax
    0x000005aa <+40> : appeler 0x54d <bowfunc>
    0x000005af <+45> : ajoutez $0x10,%esp
    0x000005b2 <+48> : sous $0xc,%esp
    0x000005b5 <+51> : lea -0x1974(%ebx),%eax
    0x000005bb <+57> : appuyez sur %eax
    0x000005bc <+58> : appelez 0x3e0 <puts@plt>
    0x000005c1 <+63> : ajoutez $0x10,%esp
    0x000005c4 <+66> : mov $0x1,%eax
    0x000005c9 <+71> : lea -0x8(%ebp),%esp
    0x000005cc <+74> : pop %ecx
    0x000005cd <+75> : pop %ebx
    0x000005ce <+76> : pop %ebp
    0x000005cf <+77> : lea -0x4(%ecx),%esp
    0x000005d2 <+80> : ret
Fin du dump de l'assembleur.

```

Dans la première colonne, les nombres hexadécimaux représentent les "adresses mémoire". Les nombres avec le signe plus (`+`) indiquent les `sauts d'adresse` dans la mémoire en octets, utilisés pour l'instruction respective. Ensuite, nous pouvons voir les `instructions assembleur` (`mnémoniques`) avec les registres et leurs `suffixes d'opération`. La syntaxe actuelle est `AT&T`, que nous pouvonsreconnaître par les caractères `%` et `$` .

| Adresse mémoire | sauts d'adresse | Instructions d'assemblage | Suffixes d'opération |
| --- | --- | --- | --- |
| 0x00000582 | <+0> : | léa | 0x4(%esp),%ecx |
| 0x00000586 | <+4> : | et | $0xfffffff0,%esp |
| ... | ... | ... | ... |

La syntaxe `Intel` facilite la lecture de la représentation désassemblée, et nous pouvons modifier la syntaxe en saisissant les commandes suivantes dans GDB :

#### GDB - Changer la syntaxe en Intel

GDB - Changer la syntaxe en Intel

```
(gdb) définit les informations de saveur de démontage
(gdb) démonter principal

Vidage du code assembleur pour la fonction main :
    0x00000582 <+0> : lea ecx,[esp+0x4]
    0x00000586 <+4> : et esp,0xfffffff0
    0x00000589 <+7> : appuyez sur DWORD PTR [ecx-0x4]
    0x0000058c <+10> : appuyez sur ebp
    0x0000058d <+11> : mov ebp, esp
    0x0000058f <+13> : appuyez sur ebx
    0x00000590 <+14> : appuyez sur ecx
    0x00000591 <+15> : appelez 0x450 <__x86.get_pc_thunk.bx>
    0x00000596 <+20> : ajouter ebx,0x1a3e
    0x0000059c <+26> : déplacer eax,ecx
    0x0000059e <+28> : déplacer eax,DWORD PTR [eax+0x4]
<SNIP>

```

Nous n'avons pas besoin de changer le mode d'affichage manuellement en permanence. Nous pouvons également définir cela comme syntaxe par défaut avec la commande suivante.

#### Modifier la syntaxe GDB

Modifier la syntaxe GDB

```
étudiant@nix-bow:~$ echo 'set disassembly-flavor intel' > ~/.gdbinit

```

Si nous réexécutons maintenant GDB et démontons la fonction principale, nous voyons la syntaxe Intel.

#### GDB - Syntaxe Intel

GDB - Syntaxe Intel

```
étudiant@nix-bow :~$ gdb ./bow32 -q

Lecture des symboles de l'arc... (aucun symbole de débogage trouvé)... fait.
(gdb) démonter principal

Vidage du code assembleur pour la fonction main :
    0x00000582 <+0> : lea ecx,[esp+0x4]
    0x00000586 <+4> : et esp,0xfffffff0
    0x00000589 <+7> : appuyez sur DWORD PTR [ecx-0x4]
    0x0000058c <+10> : appuyez sur ebp
    0x0000058d <+11> : mov ebp, esp
    0x0000058f <+13> : appuyez sur ebx
    0x00000590 <+14> : appuyez sur ecx
    0x00000591 <+15> : appelez 0x450 <__x86.get_pc_thunk.bx>
    0x00000596 <+20> : ajouter ebx,0x1a3e
    0x0000059c <+26> : déplacer eax,ecx
    0x0000059e <+28> : déplacer eax,DWORD PTR [eax+0x4]
    0x000005a1 <+31> : ajouter eax,0x4
    0x000005a4 <+34> : déplacer eax, DWORD PTR [eax]
    0x000005a6 <+36> : sous-esp,0xc
    0x000005a9 <+39> : appuyez sur eax
    0x000005aa <+40> : appeler 0x54d <bowfunc>
    0x000005af <+45> : ajouter esp,0x10
    0x000005b2 <+48> : sous-esp,0xc
    0x000005b5 <+51> : lea eax,[ebx-0x1974]
    0x000005bb <+57> : appuyez sur eax
    0x000005bc <+58> : appelez 0x3e0 <puts@plt>
    0x000005c1 <+63> : ajouter esp,0x10
    0x000005c4 <+66> : déplacer eax,0x1
    0x000005c9 <+71> : lea esp,[ebp-0x8]
    0x000005cc <+74> : pop ecx
    0x000005cd <+75> : pop ebx
    0x000005ce <+76> : pop ebp
    0x000005cf <+77> : lea esp,[ecx-0x4]
    0x000005d2 <+80> : ret
Fin du dump de l'assembleur.

```

La différence entre la syntaxe `AT&T` et `Intel` ne réside pas seulement dans la présentation des instructions avec leurs symboles, mais également dans l'ordre et la direction dans lesquels les instructions sont exécutées et lues.

Prenons l'instruction suivante comme exemple :

GDB - Syntaxe Intel

```
    0x0000058d <+11> : mov ebp, esp

```

Avec la syntaxe Intel, nous avons l'ordre suivant pour l'instruction de l'exemple :

Syntaxe Intel
------------

| Instruction | `Destination` | source |
| --- | --- | --- |
| mouvement | `ebp` | esp |

Syntaxe AT&T
-----------

| Instruction | source | `Destination` |
| --- | --- | --- |
| mouvement | %esp | `%ebp` |