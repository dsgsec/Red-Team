Filtres de liste blanche
=================

* * * * *

Comme indiqué dans la section précédente, l'autre type de validation d'extension de fichier consiste à utiliser une "liste blanche des extensions de fichier autorisées". Une liste blanche est généralement plus sécurisée qu'une liste noire. Le serveur Web n'autoriserait que les extensions spécifiées, et la liste n'aurait pas besoin d'être exhaustive pour couvrir les extensions peu courantes.

Pourtant, il existe différents cas d'utilisation pour une liste noire et pour une liste blanche. Une liste noire peut être utile dans les cas où la fonctionnalité de téléchargement doit autoriser une grande variété de types de fichiers (par exemple, le gestionnaire de fichiers), tandis qu'une liste blanche n'est généralement utilisée qu'avec les fonctionnalités de téléchargement où seuls quelques types de fichiers sont autorisés. Les deux peuvent également être utilisés en tandem.

* * * * *

Extensions de liste blanche
------------------------

Commençons l'exercice à la fin de cette section et essayons d'importer une extension PHP inhabituelle, comme `.phtml`, et voyons si nous pouvons toujours l'importer comme nous l'avons fait dans la section précédente :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_whitelist_message.jpg)

Nous constatons que nous recevons un message disant "Seules les images sont autorisées", ce qui peut être plus courant dans les applications Web que de voir un type d'extension bloqué. Cependant, les messages d'erreur ne reflètent pas toujours la forme de validation utilisée, alors essayons de rechercher les extensions autorisées comme nous l'avons fait dans la section précédente, en utilisant la même liste de mots que nous avons utilisée précédemment :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_whitelist_fuzz.jpg)

Nous pouvons voir que toutes les variantes des extensions PHP sont bloquées (par exemple `php5`, `php7`, `phtml`). Cependant, la liste de mots que nous avons utilisée contenait également d'autres extensions "malveillantes" qui n'ont pas été bloquées et ont été téléchargées avec succès. Essayons donc de comprendre comment nous avons pu télécharger ces extensions et dans quels cas nous pouvons les utiliser pour exécuter du code PHP sur le serveur principal.

Voici un exemple de test de liste blanche d'extension de fichier :

Code : php

```
$fileName = basename($_FILES["uploadFile"]["name"]);

if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
     echo "Seules les images sont autorisées" ;
     mourir();
}

```

Nous voyons que le script utilise une expression régulière (`regex`) pour tester si le nom de fichier contient des extensions d'image sur liste blanche. Le problème ici réside dans la `regex`, car elle vérifie uniquement si le nom de fichier `contient` l'extension et non s'il se termine réellement par celle-ci. De nombreux développeurs commettent de telles erreurs en raison d'une mauvaise compréhension des modèles de regex.

Voyons donc comment nous pouvons contourner ces tests pour télécharger des scripts PHP.

* * * * *

Rallonges doubles
-----------------

Le code teste uniquement si le nom de fichier contient une extension d'image ; une méthode simple pour réussir le test regex consiste à utiliser `Double Extensions`. Par exemple, si l'extension `.jpg` a été autorisée, nous pouvons l'ajouter au nom de notre fichier téléchargé et toujours terminer notre nom de fichier par `.php` (par exemple `shell.jpg.php`), auquel cas nous devrions pouvoir pour réussir le test de la liste blanche, tout en téléchargeant un script PHP capable d'exécuter du code PHP.

Exercice : Essayez de fuzzer le formulaire de téléchargement avec [Cette liste de mots](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) pour trouver quelles extensions sont ajoutées à la liste blanche par le formulaire de téléchargement.

Interceptons une requête de téléchargement normale et modifions le nom du fichier en (`shell.jpg.php`), et modifions son contenu en celui d'un shell Web :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_double_ext_request.jpg)

Maintenant, si nous visitons le fichier téléchargé et essayons d'envoyer une commande, nous pouvons voir qu'il exécute effectivement avec succès les commandes système, ce qui signifie que le fichier que nous avons téléchargé est un script PHP entièrement fonctionnel :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell.jpg)

Cependant, cela peut ne pas toujours fonctionner, car certaines applications Web peuvent utiliser un modèle "regex" strict, comme mentionné précédemment, comme suit :

Code : php

```
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) { ...SNIP... }

```

Ce modèle ne doit prendre en compte que l'extension de fichier finale, car il utilise (`^.*\.`) pour faire correspondre tout jusqu'au dernier (`.`), puis utilise (`$`) à la fin pour ne faire correspondre que les extensions qui terminent le nom du fichier. Ainsi, `l'attaque ci-dessus ne fonctionnerait pas`. Néanmoins, certaines techniques d'exploitation peuvent nous permettre de contourner ce schéma, mais la plupart reposent sur des mauvaises configurations ou des systèmes obsolètes.

* * * * *

Double extension inversée
------------------------

Dans certains cas, la fonctionnalité de téléchargement de fichiers elle-même peut ne pas être vulnérable, mais la configuration du serveur Web peut entraîner une vulnérabilité. Par exemple, une organisation peut utiliser une application Web open source, qui dispose d'une fonctionnalité de téléchargement de fichiers. Même si la fonctionnalité de téléchargement de fichiers utilise un modèle regex strict qui ne correspond qu'à l'extension finale du nom de fichier, l'organisation peut utiliser les configurations non sécurisées pour le serveur Web.

Par exemple, le `/etc/apache2/mods-enabled/php7.4.conf` du serveur Web `Apache2` peut êtreincluez la configuration suivante :

Code : xml

```
<CorrespondanceFichiers ".+\.ph(ar|p|tml)">
     Application SetHandler/x-httpd-php
</FilesMatch>

```

La configuration ci-dessus est la façon dont le serveur Web détermine les fichiers pour autoriser l'exécution du code PHP. Il spécifie une liste blanche avec un modèle regex qui correspond `.phar`, `.php` et `.phtml`. Cependant, ce modèle de regex peut avoir la même erreur que nous avons vue plus tôt si nous oublions de le terminer par (`$`). Dans de tels cas, tout fichier contenant les extensions ci-dessus sera autorisé à exécuter du code PHP, même s'il ne se termine pas par l'extension PHP. Par exemple, le nom de fichier (`shell.php.jpg`) devrait réussir le test de liste blanche précédent car il se termine par (`.jpg`), et il serait capable d'exécuter du code PHP en raison de la mauvaise configuration ci-dessus, car il contient (`.php`) dans son nom.

Exercice : L'application Web peut toujours utiliser une liste noire pour refuser les requêtes contenant des extensions "PHP". Essayez de fuzzer le formulaire de téléchargement avec la [liste de mots PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) pour trouver les extensions mises sur liste noire par le formulaire de téléchargement.

Essayons d'intercepter une requête de téléchargement d'image normale et utilisons le nom de fichier ci-dessus pour passer le test strict de la liste blanche :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_reverse_double_ext_request.jpg)

Maintenant, nous pouvons visiter le fichier téléchargé et tenter d'exécuter une commande :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell.jpg)

Comme nous pouvons le constater, nous avons réussi à contourner le test strict de la liste blanche et à exploiter la mauvaise configuration du serveur Web pour exécuter du code PHP et prendre le contrôle du serveur.

Injection de caractères
-------------------

Enfin, discutons d'une autre méthode permettant de contourner un test de validation de liste blanche via `Character Injection`. Nous pouvons injecter plusieurs caractères avant ou après l'extension finale pour que l'application Web interprète mal le nom de fichier et exécute le fichier téléchargé en tant que script PHP.

Voici quelques-uns des caractères que nous pouvons essayer d'injecter :

- `%20`
- `%0a`
- `%00`
- `%0d0a`
- `/`
- `.\`
- `.`
- '...'
- `:`

Chaque caractère a un cas d'utilisation spécifique qui peut inciter l'application Web à mal interpréter l'extension de fichier. Par exemple, (`shell.php%00.jpg`) fonctionne avec les serveurs PHP avec la version `5.X` ou antérieure, car il oblige le serveur Web PHP à terminer le nom de fichier après (`%00`), et stockez-le sous (`shell.php`), tout en passant la liste blanche. La même chose peut être utilisée avec les applications Web hébergées sur un serveur Windows en injectant deux-points (`:`) avant l'extension de fichier autorisée (par exemple `shell.aspx:.jpg`), qui doit également écrire le fichier sous la forme (`shell. aspx`). De même, chacun des autres caractères a un cas d'utilisation qui peut nous permettre de télécharger un script PHP tout en contournant le test de validation de type.

Nous pouvons écrire un petit script bash qui génère toutes les permutations du nom de fichier, où les caractères ci-dessus seraient injectés avant et après les extensions `PHP` et `JPG` , comme suit :

Code : bash

```
pour le caractère dans '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '...' ':'; faire
     pour ext dans '.php' '.phps'; faire
         echo "shell$char$ext.jpg" >> liste de mots.txt
         echo "shell$ext$char.jpg" >> liste de mots.txt
         echo "shell.jpg$char$ext" >> liste de mots.txt
         echo "shell.jpg$ext$char" >> liste de mots.txt
     fait
fait

```

Avec cette liste de mots personnalisée, nous pouvons exécuter une analyse fuzzing avec `Burp Intruder`, similaire à celles que nous avons effectuées précédemment. Si le back-end ou le serveur Web est obsolète ou présente certaines erreurs de configuration, certains des noms de fichiers générés peuvent contourner le test de la liste blanche et exécuter du code PHP.