Contournement charactères sur liste noire
======================================

* * * * *

Outre les opérateurs d'injection et les caractères d'espacement, un caractère très couramment mis sur liste noire est le caractère barre oblique (`/`) ou barre oblique inverse (`\`), car il est nécessaire de spécifier des répertoires sous Linux ou Windows. Nous pouvons utiliser plusieurs techniques pour produire n'importe quel charactères que nous voulons tout en évitant l'utilisation de charactères sur liste noire.

* * * * *

Linux
-----

Il existe de nombreuses techniques que nous pouvons utiliser pour avoir des barres obliques dans notre charge utile. Une de ces techniques que nous pouvons utiliser pour remplacer les barres obliques (`ou tout autre caractère`) consiste à utiliser `Variables d'environnement Linux` comme nous l'avons fait avec `${IFS}`. Alors que `${IFS}` est directement remplacé par un espace, il n'existe pas de telle variable d'environnement pour les barres obliques ou les points-virgules. Cependant, ces caractères peuvent être utilisés dans une variable d'environnement, et nous pouvons spécifier `start` et `length` de notre chaîne pour correspondre exactement à ce caractère.

Par exemple, si nous examinons la variable d'environnement `$PATH` sous Linux, cela peut ressembler à ceci :

```
dsgsec@htb[/htb]$ echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games

```

Ainsi, si nous commençons par le caractère `0` et ne prenons qu'une chaîne de longueur `1`, nous nous retrouverons avec uniquement le caractère `/` , que nous pouvons utiliser dans notre charge utile :

```
dsgsec@htb[/htb]$ echo ${PATH:0:1}

/

```

Remarque : Lorsque nous utilisons la commande ci-dessus dans notre charge utile, nous n'ajoutons pas `echo`, car nous ne l'utilisons dans ce cas que pour afficher le caractère généré.

Nous pouvons faire de même avec les variables d'environnement `$HOME` ou `$PWD` . Nous pouvons également utiliser le même concept pour obtenir un caractère point-virgule, à utiliser comme opérateur d'injection. Par exemple, la commande suivante nous donne un point-virgule :

```
dsgsec@htb[/htb]$ echo ${LS_COLORS:10:1}

;

```

Exercice : essayez de comprendre comment la commande ci-dessus a généré un point-virgule, puis utilisez-la dans la charge utile pour l'utiliser comme opérateur d'injection. Astuce : La commande `printenv` imprime toutes les variables d'environnement sous Linux, afin que vous puissiez rechercher celles qui peuvent contenir des caractères utiles, puis essayer de réduire la chaîne à ce caractère uniquement.

Essayons donc d'utiliser des variables d'environnement pour ajouter un point-virgule et un espace à notre charge utile (`127.0.0.1${LS_COLORS:10:1}${IFS}`) comme charge utile, et voyons si nous pouvons contourner le filtre : ![Opérateur de filtre](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_spaces_5.jpg)

Comme nous pouvons le voir, nous avons également contourné avec succès le filtre de caractères cette fois-ci.

* * * * *

les fenêtres
-------

Le même concept fonctionne également sur Windows. Par exemple, pour produire une barre oblique dans `Windows Command Line (CMD)`, nous pouvons `echo` une variable Windows (`%HOMEPATH%` -> `\Users\htb-student`), puis spécifier une position de départ ( `~6` -> `\htb-student`), et enfin en spécifiant une position de fin négative, qui dans ce cas est la longueur du nom d'utilisateur `htb-student` (`-11` -> `\`) :

```
C:\htb> echo %HOMEPATH:~6,-11%

```

Nous pouvons obtenir la même chose en utilisant les mêmes variables dans `Windows PowerShell`. Avec PowerShell, un mot est considéré comme un tableau, nous devons donc spécifier l'index du caractère dont nous avons besoin. Comme nous n'avons besoin que d'un seul caractère, nous n'avons pas à spécifier les positions de début et de fin :

```
PS C:\htb> $env:HOMEPATH[0]

PS C:\htb> $env:PROGRAMFILES[10]
PS C:\htb>

```

Nous pouvons également utiliser la commande `Get-ChildItem Env:` PowerShell pour imprimer toutes les variables d'environnement, puis en choisir une pour produire le caractère dont nous avons besoin. `Essayez d'être créatif et trouvez différentes commandes pour produire des caractères similaires.`

* * * * *

Changement de charactères
------------------

Il existe d'autres techniques pour produire les caractères requis sans les utiliser, comme le "décalage des caractères". Par exemple, la commande Linux suivante décale le caractère que nous passons de `1`. Donc, tout ce que nous avons à faire est de trouver le caractère dans la table ASCII qui se trouve juste avant le caractère dont nous avons besoin (nous pouvons l'obtenir avec `man ascii`), puis de l'ajouter à la place de `[` dans l'exemple ci-dessous. De cette façon, le dernier caractère imprimé serait celui dont nous avons besoin :

```
dsgsec@htb[/htb]$ man ascii # \ est sur 92, avant d'être [ sur 91
dsgsec@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)

```

Nous pouvons utiliser les commandes PowerShell pour obtenir le même résultat sous Windows, bien qu'elles puissent être plus longues que celles de Linux.