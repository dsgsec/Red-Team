Registres du processeur
=============

* * * * *

Les registres sont les composants essentiels d'un processeur. Presque tous les registres offrent une petite quantité d'espace de stockage où les données peuvent être stockées temporairement. Cependant, certains d'entre eux ont une fonction particulière.

Ces registres seront divisés en registres généraux, registres de contrôle et registres de segment. Les registres les plus critiques dont nous avons besoin sont les registres généraux. Dans ceux-ci, il existe d'autres subdivisions en registres de données, registres de pointeur et registres d'index.

#### Registres de données

| Registre 32 bits | Registre 64 bits | Descriptif |
| --- | --- | --- |
| `EAX` | `RAX` | L'accumulateur est utilisé en entrée/sortie et pour les opérations arithmétiques |
| `EBX` | `RBX` | Base est utilisée dans l'adressage indexé |
| `ECX` | `RCX` | Le compteur est utilisé pour faire pivoter les instructions et compter les boucles |
| `EDX` | `RDX` | Les données sont utilisées pour les E/S et dans les opérations arithmétiques pour les opérations de multiplication et de division impliquant de grandes valeurs |

* * * * *

#### Registres de pointeur

| Registre 32 bits | Registre 64 bits | Descriptif |
| --- | --- | --- |
| `EIP` | `RIP` | Le pointeur d'instruction stocke l'adresse de décalage de la prochaine instruction à exécuter |
| `ESP` | `RSP` | Stack Pointer pointe vers le haut de la pile |
| `EBP` | `RBP` | Le pointeur de base est également connu sous le nom `Stack Base Pointer` ou `Frame Pointer` qui pointe vers la base de la pile |

* * * * *

Cadres de pile
------------

Étant donné que la pile commence par une adresse élevée et se développe jusqu'à des adresses à faible mémoire au fur et à mesure que des valeurs sont ajoutées, le `Pointeur de base` pointe vers le début (base) de la pile, contrairement au `Pointeur de pile`, qui pointe vers le haut de la pile.

Au fur et à mesure que la pile grandit, elle est logiquement divisée en régions appelées "Stack Frames", qui allouent la mémoire requise dans la pile pour la fonction correspondante. Un cadre de pile définit un cadre de données avec le début (`EBP`) et la fin (`ESP`) qui est poussé sur la pile lorsqu'une fonction est appelée.

Étant donné que la mémoire de la pile est construite sur une structure de données `Last-In-First-Out` (`LIFO`), la première étape consiste à stocker la position `Previous EBP` sur la pile, qui peut être restaurée une fois la fonction terminée. Si nous examinons maintenant la fonction `bowfunc` , elle ressemble à ce qui suit dans GDB :

Registres de pointeur

```
(gdb) disas bowfunc

Vidage du code assembleur pour la fonction bowfunc :
    0x0000054d <+0> : appuyez sur ebp # <---- 1. Stocke l'EBP précédent
    0x0000054e <+1> : mov ebp, esp
    0x00000550 <+3> : appuyez sur ebx
    0x00000551 <+4> : sous esp, 0x404
    <...SNIP...>
    0x00000580 <+51> : quitter
    0x00000581 <+52> : ret

```

Le `EBP` du cadre de pile est défini en premier lorsqu'une fonction est appelée et contient le `EBP` du cadre de pile précédent. Ensuite, la valeur de `ESP` est copiée dans `EBP`, créant un nouveau cadre de pile.

Registres de pointeur

```
(gdb) disas bowfunc

Vidage du code assembleur pour la fonction bowfunc :
    0x0000054d <+0> : appuyez sur ebp # <---- 1. Stocke l'EBP précédent
    0x0000054e <+1> : mov ebp,esp # <---- 2. Crée un nouveau cadre de pile
    0x00000550 <+3> : appuyez sur ebx
    0x00000551 <+4> : sous esp, 0x404
    <...SNIP...>
    0x00000580 <+51> : quitter
    0x00000581 <+52> : ret

```

Ensuite, de l'espace est créé dans la pile, déplaçant `ESP` vers le haut pour les opérations et les variables nécessaires et traitées.

#### Prologue

Prologue

```
(gdb) disas bowfunc

Vidage du code assembleur pour la fonction bowfunc :
    0x0000054d <+0> : appuyez sur ebp # <---- 1. Stocke l'EBP précédent
    0x0000054e <+1> : mov ebp,esp # <---- 2. Crée un nouveau cadre de pile
    0x00000550 <+3> : appuyez sur ebx
    0x00000551 <+4> : sub esp,0x404 # <---- 3. Déplace ESP vers le haut
    <...SNIP...>
    0x00000580 <+51> : quitter
    0x00000581 <+52> : ret

```

Ces trois instructions représentent le soi-disant `Prologue`.

Pour sortir du cadre de la pile, c'est l'inverse qui est fait, l' `Epilogue`. Pendant l'épilogue, `ESP` est remplacé par `EBP` actuel, et sa valeur est réinitialisée à la valeur qu'elle avait auparavant dans le prologue. L'épilogue est relativement court, et à part d'autres possibilités de l'exécuter, dans notre exemple, il est exécuté avec deux instructions :

#### Épilogue

Épilogue

```
(gdb) disas bowfunc

Vidage du code assembleur pour la fonction bowfunc :
    0x0000054d <+0> : appuyez sur ebp
    0x0000054e <+1> : mov ebp, esp
    0x00000550 <+3> : appuyez sur ebx
    0x00000551 <+4> : sous esp, 0x404
    <...SNIP...>
    0x00000580 <+51> : laisser # <----------------------
    0x00000581 <+52> : ret # <--- Quitter le cadre de la pile

```

* * * * *

#### Registres d'index

| Enregistrer 32 bits | Enregistrer 64 bits | Descriptif |
| --- | --- | --- |
| `ESI` | `RSI` | L'index source est utilisé comme pointeur à partir d'une source pour les opérations sur les chaînes |
| "EDI" | `RDI` | La destination est utilisée comme pointeur vers une destination pour les opérations sur les chaînes |

* * * * *

Un autre point important concernant la représentation de l'assembleur est la dénomination des registres. Cela dépend du format dans lequel le binaire a été compilé. Nous avons utilisé GCC pour compiler le code `bow.c` en 32-bit format. Compilons maintenant le même code dans un format "64 bits".

#### Compiler au format 64 bits

Compiler au format 64 bits

```
étudiant@nix-bow:~$ gcc bow.c -o bow64 -fno-stack-protector -z execstack -m64
étudiant@nix-bow:~$ fichier bow64 | tr "," "\n"

bow64 : objet partagé LSB ELF 64 bits
  x86-64
  version 1 (SYSV)
  lié dynamiquement
  interpréteur /lib64/ld-linux-x86-64.so.2
  pour GNU/Linux 3.2.0
  ID de construction[sha1]=9503477016e8604e808215b4babb250ed25a7b99
  pas dépouillé

```

Donc si on regarde maintenant le code assembleur, on voit que les adresses sont deux fois plus grosses, et on a presque la moitié des instructions comme avec un binaire compilé 32 bits.

Compiler au format 64 bits

```
étudiant@nix-bow :~$ gdb -q bow64

Lecture des symboles de bow64... (aucun symbole de débogage trouvé)... fait.
(gdb) disas principal

Vidage du code assembleur pour la fonction main :
    0x00000000000006bc <+0> : appuyez sur rbp
    0x00000000000006bd <+1> : mov rbp,rsp
    0x00000000000006c0 <+4> : sous-rsp, 0x10
    0x00000000000006c4 <+8> : mov DWORD PTR [rbp-0x4], edi
    0x00000000000006c7 <+11> : mov QWORD PTR [rbp-0x10],rsi
    0x00000000000006cb <+15> : mov rax, QWORD PTR [rbp-0x10]
    0x00000000000006cf <+19> : ajouter rax,0x8
    0x00000000000006d3 <+23> : mov rax,QWORD PTR [rax]
    0x00000000000006d6 <+26> : mov rdi,rax
    0x00000000000006d9 <+29> : appeler 0x68a <fonction arc>
    0x00000000000006de <+34> : lea rdi,[rip+0x9f]
    0x00000000000006e5 <+41> : appeler 0x560 <puts@plt>
    0x00000000000006ea <+46> : déplacer eax,0x1
    0x00000000000006ef <+51> : quitter
    0x00000000000006f0 <+52> : ret
Fin du dump de l'assembleur.

```

Cependant, nous allons d'abord jeter un œil à la version 32 bits du binaire vulnérable. L'instruction la plus importante pour nous en ce moment est l'instruction `call` . L'instruction `call` est utilisée pour appeler une fonction et effectue deux opérations :

1. il pousse l'adresse de retour sur la `pile` afin que l'exécution du programme puisse se poursuivre une fois que la fonction a rempli son objectif avec succès,
2. il remplace le `pointeur d'instruction` (`EIP`) par la destination de l'appel et y démarre l'exécution.

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
    0x000005aa <+40> : appel 0x54d <bowfunc> # <--- CALL fonction
<SNIP>

```

* * * * *

Endianité
----------

Lors des opérations de chargement et de sauvegarde dans les registres et les mémoires, les octets sont lus dans un ordre différent. Cet ordre des octets s'appelle `endianness`. L'endianité est distinguée entre le format `little-endian` et le format `big-endian` .

`Big-endian` et `little-endian` concernent l'ordre de valence. Dans `big-endian`, les chiffres avec la valence la plus élevée sont initialement. En `little-endian`, les chiffres avec la valence la plus faible sont au début. Les processeurs mainframe utilisent le format "big-endian", certaines architectures RISC, les mini-ordinateurs et, dans les réseaux TCP/IP, l'ordre des octets est également au format "big-endian".

Maintenant, regardons un exemple avec les valeurs suivantes :

- Adresse : `0xffff0000`
- Mot : `\xAA\xBB\xCC\xDD`

| Adresse mémoire | 0xffff0000 | 0xffff0001 | 0xffff0002 | 0xffff0003 |
| --- | --- | --- | --- | --- |
| Big-Endian | AA | BB | CC | JJ |
| Little-Endian | JJ | CC | BB | AA |

Ceci est très important pour que nous saisissions notre code dans le bon ordre plus tard lorsque nous devrons dire au CPU vers quelle adresse il doit pointer.