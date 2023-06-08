Identifier les Bad Characters
==========================

* * * * *

Avant de commencer à utiliser le fait que nous pouvons contrôler `EIP` et renverser le flux d'exécution du programme, nous devons déterminer les caractères que nous devons éviter d'utiliser dans notre charge utile.

Comme nous attaquons un paramètre d'entrée (un fichier ouvert dans ce cas), le programme est censé traiter notre entrée. Ainsi, selon le traitement que chaque programme exécute sur notre entrée, certains caractères peuvent indiquer au programme qu'il a atteint la fin de l'entrée. Cela peut se produire même s'il n'a pas encore atteint la fin de l'entrée.

Par exemple, un mauvais caractère très courant est un octet nul `0x00`, utilisé dans Assembly comme un terminateur de chaîne, qui indique au processeur que la chaîne est terminée. Ainsi, si notre charge utile incluait un octet nul, le programme peut arrêter de traiter notre shellcode, pensant qu'il en a atteint la fin. Cela empêcherait notre charge utile de s'exécuter correctement et notre attaque échouerait. D'autres exemples sont `0x0a` et `0x0d`, qui sont respectivement la nouvelle ligne `\n` et le retour chariot `\r`. Si nous exploitions un débordement de mémoire tampon dans une entrée de chaîne censée être une seule ligne (comme une clé de licence), ces caractères mettraient probablement fin à notre entrée prématurément, ce qui entraînerait également l'échec de notre charge utile.

* * * * *

Génération de tous les caractères
-------------------------

Pour identifier les caractères incorrects, nous devons envoyer tous les caractères après avoir renseigné `EIP` adresse, qui est après `4112` + `4` octets. Nous vérifions ensuite si l'un des caractères a été supprimé par le programme ou si notre entrée a été tronquée prématurément après un caractère spécifique.

Pour ce faire, nous aurions besoin de deux fichiers :

1. Un fichier `.wav` avec tous les caractères à charger dans le programme
2. Un fichier `.bin` à comparer avec notre entrée en mémoire

Nous pouvons utiliser `ERC` pour générer le fichier `.bin` et générer une liste de tous les caractères pour créer notre fichier `.wav` . Pour ce faire, nous pouvons utiliser la commande `ERC --bytearray` : ![ERC Byte Array](https://academy.hackthebox.com/storage/modules/89/win32bof_erc_bytearry.jpg)

Cela crée également deux fichiers sur notre bureau :

- `ByteArray_1.txt` : qui contient la chaîne de tous les caractères que nous pouvons utiliser dans notre exploit python
- `ByteArray_1.bin` : que nous pouvons utiliser avec `ERC` plus tard pour comparer avec notre entrée en mémoire

* * * * *

Mise à jour de notre exploit
--------------------

L'étape suivante consiste à générer un fichier `.wav` avec la chaîne de caractères générée par `ERC`. Nous allons à nouveau écrire une nouvelle fonction `bad_chars()` et utiliser un code similaire à la fonction `eip_control()` , mais nous utiliserons les caractères sous `C#` dans `ByteArray_1.txt`. Nous allons créer une nouvelle liste d'octets `all_chars = bytes([])` et coller les caractères entre crochets. Nous allons ensuite écrire dans `chars.wav` la même `charge utile` de `eip_control()`, et ajouter après `all_chars`. La fonction finale ressemblerait à ceci :

Code : python

```
def bad_chars():
    all_chars = bytes([
        0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07,
        ...SNIP...
        0xF8, 0xF9, 0xFA, 0xFB, 0xFC, 0xFD, 0xFE, 0xFF
    ])
    
    offset = 4112
    buffer = b"A"*offset
    eip = b"B"*4
    payload = buffer + eip + all_chars
    
    with open('chars.wav', 'wb') as f:
        f.write(payload)

bad_chars()

```

Notez comment nous avons ajouté un `b` avant `A` et `B` pour les transformer en octets

Nous pouvons maintenant exécuter notre exploit avec `F5` pour générer le fichier `chars.wav` .

* * * * *

Comparer notre entrée
-------------------

Nous pouvons maintenant redémarrer notre programme dans `x32dbg` et charger `chars.wav` dedans. Une fois que nous l'avons fait, nous pouvons commencer à comparer notre entrée en mémoire et voir s'il manque des caractères. Pour ce faire, nous pouvons vérifier le volet Stack en bas à droite de `x32dbg`, qui doit être aligné exactement au début de notre entrée : ![Byte Stack](https://academy.hackthebox.com/storage/modules /89/win32bof_bytes_stack.jpg)

Nous pouvons maintenant parcourir manuellement la pile ligne par ligne de droite à gauche et nous assurer que toutes les valeurs hexadécimales sont présentes, de `0x00` à `0xff`. Comme cela peut prendre un certain temps et que nous comptons entièrement sur nos yeux, nous pouvons manquer un personnage ou deux. Nous allons donc à nouveau utiliser `ERC` pour effectuer la comparaison à notre place. Il comparera facilement notre entrée en mémoire à tous les caractères.

Nous devons d'abord copier l'adresse `ESP` car c'est là que se trouve notre entrée. Nous pouvons le faire en cliquant dessus avec le bouton droit de la souris et en sélectionnant `Copier la valeur`, ou en cliquant sur `[Ctrl + C]` : ![Byte ESP](https://academy.hackthebox.com/storage/modules/89/win32bof_bytes_esp .jpg)

Une fois que nous avons la valeur `ESP`, nous pouvons utiliser `ERC --compare` et lui donner l'adresse `ESP` et l'emplacement du fichier `.bin` qui contient tous les caractères, comme suit :

```
ERC --compare 0014F974 C:\Users\htb-student\Desktop\ByteArray_1.bin

```

Ce que cette commande va faire, c'est comparer octet par octet à la fois notre entrée dans `ESP` et tous les caractères que nous avons générés précédemment dans `ByteArray_1.bin` : ![Byte Compare 1](https://academy.hackthebox.com/ stockage/modules/89/win32bof_bytes_compare.jpg)

Comme nous pouvons le voir, cela place chaque octet des deux emplacements à côté de chaqueautre pour repérer rapidement tout problème. La sortie que nous recherchons est celle où tous les octets des deux emplacements sont identiques, sans aucune différence. Cependant, nous constatons qu'après le premier caractère `00`, tous les octets restants sont différents. `Cela indique que 0x00 a tronqué l'entrée restante et qu'il doit donc être considéré comme un mauvais caractère.`

* * * * *

Éliminer les mauvais personnages
--------------------------

Maintenant que nous avons identifié le premier caractère incorrect, nous devons utiliser à nouveau `--bytearray` pour générer une liste de tous les caractères sans les caractères incorrects, que nous pouvons spécifier avec `-bytes 0x00,0x0a,0x0d...etc.` . Ainsi, nous utiliserons la commande suivante :

```
ERC --bytearray -bytes 0x00

```

Maintenant, utilisons à nouveau cette commande avec `ERC` pour générer le nouveau fichier et l'utiliser pour mettre à jour notre exploit :

![ERC Byte Array 2](https://academy.hackthebox.com/storage/modules/89/win32bof_erc_bytearry_2.jpg)

Comme nous pouvons le voir, cette fois, il est indiqué `excluant : 00`, et la table de tableaux n'inclut pas `00` au début. Passons donc au fichier de sortie généré `ByteArray_2.txt`, copions les nouveaux octets sous `C#` et plaçons-les dans notre exploit, qui devrait maintenant ressembler à ceci :

Code : python

```
def bad_chars():
    all_chars = bytes([
        0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08
...SNIP...

```

Remarque : Le fichier `chars.wav` peut toujours être détenu par le débogueur, et notre script python peut ne pas être en mesure de l'écraser. Donc, redémarrez le programme dans `x32dbg` pour libérer le fichier avant d'exécuter l'exploit.

Une fois que nous aurons notre nouveau fichier `chars.wav` , nous le chargerons à nouveau dans notre programme et utiliserons `--compare` avec le nouveau fichier `ByteArray_2.bin` pour voir si les deux entrées correspondent : ![Byte Compare 2] (https://academy.hackthebox.com/storage/modules/89/win32bof_bytes_compare_2.jpg)

Comme nous pouvons le voir, cette fois, les deux lignes correspondent parfaitement jusqu'à `0xFF`, ce qui signifie qu'il n'y a plus de mauvais caractères dans notre entrée. Si nous avions identifié un autre mauvais personnage, nous répéterions le même processus que nous venons de faire pour `Éliminer les mauvais personnages` jusqu'à ce que les deux lignes correspondent parfaitement.

Nous savons donc maintenant que nous devons éviter d'utiliser `0x00` dans l'adresse `EIP` que nous voulons exécuter ou dans notre shellcode.

Astuce : Nous constaterions que les caractères `0x00`, `0x0a`, `0x0d` sont souvent considérés comme des caractères incorrects dans de nombreux programmes et de nombreuses fonctions vulnérables (comme indiqué précédemment). Donc, pour gagner du temps, on peut considérer ceux-ci comme de mauvais personnages dès le départ, et chercher d'autres mauvais personnages.

Remarque : Les mauvais caractères trouvés dans cette section peuvent ne pas refléter les vrais mauvais caractères de notre programme, car il ne s'agissait que d'une démonstration de la façon d'identifier les mauvais caractères. Essayez de répéter le processus pour trouver les vrais mauvais personnages, le cas échéant.
