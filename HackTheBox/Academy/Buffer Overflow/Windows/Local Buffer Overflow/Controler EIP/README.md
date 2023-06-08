Contrôle de l'EIP
===============

* * * * *

Jusqu'à présent, nous avons réussi à fuzzer les paramètres et identifié un point d'entrée vulnérable. Notre prochaine étape serait de contrôler précisément quelle adresse est placée dans `EIP`, de sorte qu'elle soit exécutée lorsque le programme revient de la fonction avec l'instruction `ret`. Pour ce faire, nous devons d'abord calculer notre décalage exact de `EIP`, ce qui signifie à quelle distance `EIP` est du début de l'entrée. Une fois que nous connaissons le décalage, nous pouvons remplir le tampon menant à `EIP` avec toutes les données inutiles, puis placer l'adresse de l'instruction que nous voulons exécuter à l'emplacement `EIP`.

* * * * *

Décalage EIP
----------

Il existe de nombreuses astuces que nous pouvons utiliser pour trouver le décalage de `EIP` à partir de notre entrée. Une façon de procéder consiste à envoyer un tampon à moitié rempli de `A` et à moitié rempli de `B`, puis de voir quel caractère remplit `EIP`. S'il est rempli de `0x41`, cela indiquerait qu'il se trouve dans la première moitié, et s'il est rempli de `0x42`, cela signifierait qu'il se trouve dans la seconde moitié. Une fois que nous savons dans quelle moitié `EIP` se trouve, nous pouvons répéter la même procédure avec cette moitié et la diviser en deux quarts, et ainsi de suite, jusqu'à ce que nous trouvions exactement où se trouve `EIP` .

Cependant, comme nous pouvons l'imaginer, ce n'est pas une méthode très rapide ou efficace pour trouver le décalage, et nous ne l'utiliserions que dans les cas où le tampon contient des dizaines de milliers de caractères, car nous ne pourrons pas utiliser notre deuxième méthode dans ces types de cas. Même dans ce cas, nous utiliserions la méthode "moitiés" pour trouver l'emplacement général de `EIP` dans notre tampon, puis nous utiliserions la deuxième méthode pour trouver son emplacement exact.

Une autre méthode pour trouver le décalage de `EIP` consiste à utiliser un modèle unique comme entrée, puis à voir quelles valeurs remplissent `EIP` pour calculer précisément à quelle distance il se trouve du début de notre modèle. Par exemple, nous pouvons envoyer un modèle de nombres séquentiels, 'c'est-à-dire `12345678...`', et voyez quels numéros rempliraient `EIP`. Cependant, ce n'est pas une méthode très pratique, car une fois que les nombres commencent à augmenter, il serait difficile de savoir de quel nombre il s'agit car il peut faire partie d'un nombre et d'une partie d'un autre nombre. De plus, comme les nombres commencent à avoir 2 ou 3 chiffres, ils n'indiqueraient plus le décalage réel puisque chaque nombre remplirait plusieurs octets. Comme nous pouvons le voir, utiliser des nombres comme modèle ne fonctionnerait pas.

La meilleure façon de calculer le décalage exact de `EIP` consiste à envoyer un modèle de caractères unique et non répétitif, de sorte que nous puissions afficher les caractères qui remplissent `EIP` et les rechercher dans notre modèle unique. Comme il s'agit d'un modèle unique non répétitif, nous ne trouverons qu'une seule correspondance, ce qui nous donnera le décalage exact de `EIP`.

Heureusement, nous n'avons pas besoin de coder manuellement un script qui crée ce modèle unique ou d'en coder un autre pour trouver et calculer la distance entre la valeur et le début du modèle unique. En effet, de nombreux outils peuvent le faire, comme `pattern_create` et `pattern_offset`, qui sont également inclus avec le plug-in `ERC` que nous avons installé précédemment.

* * * * *

Créer un motif unique
------------------------

Nous pouvons générer un modèle unique avec `pattern_create` soit dans notre instance `PwnBox` ou directement dans notre débogueur `x32dbg` avec le plug-in `ERC` . Pour ce faire dans `PwnBox`, nous pouvons utiliser la commande suivante :

```
dsgsec@htb[/htb]$ /usr/bin/msf-pattern_create -l 5000

Aa0Aa1Aa2...SNIP...3Gk4Gk5Gk

```

Nous pouvons maintenant envoyer ce tampon à notre programme sous la forme d'un fichier `.wav` . Cependant, il est toujours plus facile de tout faire dans Windows pour éviter de sauter entre deux VM. Voyons donc comment obtenir le même schéma avec `ERC`.

Si nous utilisons la commande `ERC --help` , nous voyons les instructions suivantes :

```
--Modèle
Génère un motif non répétitif. Un modèle de caractères ASCII purs peut être généré jusqu'à 20277 et jusqu'à
66923 si des caractères spéciaux sont utilisés. Le décalage d'une chaîne particulière peut être trouvé à l'intérieur du motif en
fournir une chaîne de recherche (doit comporter au moins 3 caractères).
     Création de modèle : ERC --pattern <create | c> <longueur>
     Décalage du motif : ERC --pattern <décalage | o> <chaîne de recherche>

```

Comme nous pouvons le voir, nous pouvons utiliser `ERC --pattern c 5000` pour obtenir notre modèle. Alors, utilisons cette commande et voyons ce que nous obtenons : ![Pattern Create](https://academy.hackthebox.com/storage/modules/89/win32bof_erc_pattern_create_1.jpg)

Ce modèle est le même que celui que nous avons obtenu avec l'outil `msf-pattern_create` , nous pouvons donc utiliser l'un ou l'autre. Nous pouvons maintenant accéder à notre bureau pour trouver la sortie enregistrée dans un fichier appelé `Pattern_Create_1.txt`. Nous pouvons maintenant enregistrer le modèle dans un fichier `.wav` et le charger dans notre programme. Cependant, pour ce faire, nous allons commencer à construire notre exploit, que nous continuerons à développer et à utiliser pour d'autres parties du processus d'exploitation du débordement de tampon.

* * * * *

Écrire notre exploit
-------------------

Nous allons écrire notre exploit en Python3 car il contient des bibliothèques intégrées pour nous aider dans ce processus, comme `struct` et `requests`. Nous écrirons également chaque partie du processus d'exploitation sous sa propre fonction afin de ne pas avoir à sauter entre différentes expLoit scripts et appelez simplement une fonction différente pour chaque partie du processus.

Nous pouvons commencer par créer une nouvelle fonction avec `def eip_offset():`, puis créer notre variable `payload` en tant qu'objet `bytes` et coller entre parenthèses la `Ascii :` sortie de `Pattern_Create_1.txt`. Ainsi, nous pouvons cliquer sur la barre de recherche Windows en bas et écrire `IDLE`, ce qui ouvrirait l'éditeur Python3, puis cliquer sur `ctrl + N` pour commencer à écrire un nouveau script python où nous pouvons commencer à écrire notre code. Notre code initial devrait ressembler à ceci :

Code : python

```
def eip_offset() :
     charge utile = octets ("Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac"
                     ...COUPER...
                     "Gi3Gi4Gi5Gi6Gi7Gi8Gi9Gj0Gj1Gj2Gj3Gj4Gj5Gj6Gj7Gj8Gj9Gk0Gk1Gk2Gk3Gk4Gk5Gk",
"utf-8")

```

Ensuite, sous la même fonction `eip_offset()` , nous allons écrire `payload` dans un fichier appelé `pattern.wav`, en ajoutant les lignes suivantes :

Code : python

```
     avec open('pattern.wav', 'wb') comme f :
         f.write(charge utile)

```

Notez que nous utilisons toujours des octets pour nos données et utilisez `'wb'` pour écrire notre modèle en octets. En effet, nous devons créer notre charge utile telle qu'elle sera traitée par le programme, qui dans un exercice de dépassement de mémoire tampon est en octets, car elle sera chargée dans la pile en tant que code machine en octets.

Enfin, nous devrions appeler notre fonction `eip_offset()` en ajoutant la ligne suivante à la fin. Sinon, la fonction ne sera pas exécutée :

Code : python3

```
eip_offset()

```

Maintenant, nous pouvons enregistrer cet exploit sur notre bureau sous le nom `win32bof_exploit.py` et l'exécuter. Pour l'exécuter alors qu'il est encore dans notre `IDLE`, nous pouvons cliquer sur `Run > Run Module`, ou sur `F5` : ![Pattern IDLE](https://academy.hackthebox.com/storage/modules/89/ win32bof_python_idle_exploit_2.jpg)

Une fois que nous le ferons, nous verrons le nouveau fichier `pattern.wav` sur notre bureau.

* * * * *

Calcul du décalage EIP
----------------------

Maintenant que notre modèle est enregistré dans un fichier `.wav` , nous pouvons le charger dans notre programme. Nous devons nous assurer que le programme est en cours d'exécution et attaché à `x32dbg`, puis nous pouvons ouvrir notre fichier comme nous l'avons fait dans la section précédente. Nous pouvons cliquer sur le bouton `redémarrer` dans `x32dbg` pour redémarrer notre programme si notre entrée précédente l'avait planté :\
![](https://academy.hackthebox.com/storage/modules/89/win32bof_x32dbg_restart.jpg)

Une fois que nous l'avons fait, nous devrions voir que notre programme se bloque en raison de la longue saisie. Plus important encore, nous devrions voir que le registre `EIP` a été écrasé par une partie de notre modèle unique : ![Pattern EIP](https://academy.hackthebox.com/storage/modules/89/win32bof_pattern_eip.jpg)

Nous pouvons maintenant utiliser la valeur `EIP` pour calculer le décalage. Nous pouvons à nouveau le faire dans notre `PwnBox` avec `msf-pattern_offset` (l'équivalent de `msf-pattern_create`), en utilisant la valeur hexadécimale dans `EIP`, comme suit :

```
dsgsec@htb[/htb]$ /usr/bin/msf-pattern_offset -q 31684630

[*] Correspondance exacte au décalage 4112

```

Comme nous pouvons le voir, cela nous indique que notre décalage `EIP` est de `4112` octets. Nous pouvons également rester dans la VM `Windows` et utiliser `ERC` pour calculer le même décalage. Tout d'abord, nous devons obtenir la valeur ASCII des octets hexadécimaux trouvés dans `EIP`, en cliquant avec le bouton droit sur `EIP` et en sélectionnant `Modifier la valeur`, ou en cliquant sur `EIP` puis en cliquant sur Entrée. Une fois que nous l'aurons fait, nous verrons différentes représentations de la valeur `EIP` , l'ASCII étant la dernière : ![ASCII EIP](https://academy.hackthebox.com/storage/modules/89/win32bof_pattern_eip_ascii.jpg)

La valeur hexadécimale trouvée dans `EIP` représente la chaîne `1hF0`. Maintenant, nous pouvons utiliser `ERC --pattern o 1hF0` pour obtenir le décalage du modèle : ![Pattern Offset](https://academy.hackthebox.com/storage/modules/89/win32bof_pattern_offset.jpg)

Nous obtenons à nouveau `4112` octets comme décalage `EIP` .

Le plug-in `ERC` peut également trouver automatiquement le décalage avec la commande `ERC --findNRP` , mais il convient de noter que cela prend beaucoup plus de temps en fonction de la taille de la RAM :

![Pattern Offset NRP](https://academy.hackthebox.com/storage/modules/89/win32bof_pattern_offset_findnrp.jpg)

Comme nous pouvons le voir, il a trouvé le décalage basé sur des modèles trouvés dans divers registres, chacun pouvant être utile dans des types spécifiques d'exploitation binaire. Pour nous, nous ne nous intéressons qu'au registre `EIP` , qui, selon lui, a un décalage de `4112` octets, comme nous l'avons vu précédemment.

* * * * *

Contrôle de l'EIP
---------------

Notre dernière étape consiste à nous assurer que nous pouvons contrôler la valeur ajoutée à `EIP`. Connaissant le décalage, nous savons exactement à quelle distance se trouve notre `EIP` par rapport au début du tampon. Donc, si nous envoyons `4112` octets, les 4 octets suivants seraient ceux qui remplissent `EIP`.

Ajoutons une autre fonction, `eip_control()`, à notre `win32bof_exploit.py` et créons une variable `offset` avec le décalage que nous avons trouvé. Ensuite, nous allons créer une variable `buffer` avec une chaîne d'octets `A` tant que notre décalage pour remplir l'espace tampon, et une variable `eip` avec la valeur que nous voulons que `EIP` soit, que nous allons utiliser comme `4` octets de `B`. Enfin, nous ajouterons les deux à une variable `payload` et l'écrirons dans `control.wav', comme suit :

Code : python

```
def eip_control() :
     décalage = 4112
     tampon = b"A"*décalage
     eip = b"B"*4
     charge utile = tampon + eip

     avec open('control.wav', 'wb') comme f :
         f.write(charge utile)

eip_control()

```

Diviser notre charge utile en variables nous permet de contrôler précisément chaque partie du tampon et de l'adapter facilement au fur et à mesure que nous travaillons sur notre exploit.

Notez comment la dernière ligne appelle maintenant notre nouvelle fonction `eip_control()`. À l'avenir, nous pourrons simplement modifier cette ligne pour exécuter la fonction dont nous avons besoin. Puisque nous ajoutons des fonctions au fur et à mesure que nous avançons dans ce module, nos fonctions seront triées dans l'ordre dans lequel nous en avons besoin.

Nous pouvons maintenant exécuter notre exploit pour générer `control.wav` et le charger dans notre programme après l'avoir redémarré dans `x32dbg`. Lorsque notre programme plante, nous voyons la valeur hexadécimale `42424242`, qui est la représentation ASCII de `BBBB` :

![Contrôle EIP](https://academy.hackthebox.com/storage/modules/89/win32bof_control_eip.jpg)

Nous voyons que nous pouvons contrôler la valeur exacte qui entre dans `EIP`, et nous pouvons échanger les `B` dans notre exploit avec l'adresse que nous voulons, et le programme doit l'appeler.

Avant de choisir une adresse à appeler, nous devons d'abord vérifier si nous devons éviter des caractères spécifiques dans notre entrée, dont nous parlerons dans la section suivante.
