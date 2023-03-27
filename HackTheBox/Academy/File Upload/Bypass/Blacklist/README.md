Filtres de liste noire
=================

* * * * *

Dans la section précédente, nous avons vu un exemple d'application Web qui appliquait uniquement des contrôles de validation de type sur le front-end (c'est-à-dire côté client), ce qui rendait trivial le contournement de ces contrôles. C'est pourquoi il est toujours recommandé d'implémenter tous les contrôles liés à la sécurité sur le serveur principal, où les attaquants ne peuvent pas le manipuler directement.

Néanmoins, si les contrôles de validation de type sur le serveur principal n'étaient pas codés de manière sécurisée, un attaquant peut utiliser plusieurs techniques pour les contourner et atteindre les téléchargements de fichiers PHP.

L'exercice que nous trouvons dans cette section est similaire à celui que nous avons vu dans la section précédente, mais il contient une liste noire d'extensions non autorisées pour empêcher le téléchargement de scripts Web. Nous verrons pourquoi l'utilisation d'une liste noire d'extensions courantes peut ne pas être suffisante pour empêcher les téléchargements de fichiers arbitraires et discuterons de plusieurs méthodes pour la contourner.

* * * * *

Extensions de liste noire
------------------------

Commençons par essayer l'un des contournements côté client que nous avons appris dans la section précédente pour télécharger un script PHP sur le serveur principal. Nous intercepterons une demande de téléchargement d'image avec Burp, remplacerons le contenu et le nom du fichier par nos scripts PHP et transmettrons la demande :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_disallowed_type.jpg)

Comme nous pouvons le voir, notre attaque n'a pas réussi cette fois-ci, car nous avons obtenu `Extension non autorisée`. Cela indique que l'application Web peut avoir une certaine forme de validation de type de fichier sur le back-end, en plus des validations frontales.

Il existe généralement deux formes courantes de validation d'une extension de fichier sur le back-end :

1. Tester par rapport à une `liste noire` de types
2. Tester par rapport à une `liste blanche` de types

En outre, la validation peut également vérifier le `type de fichier` ou le `contenu du fichier` pour la correspondance de type. La forme de validation la plus faible parmi celles-ci consiste à "tester l'extension de fichier par rapport à une liste noire d'extensions" pour déterminer si la demande de téléchargement doit être bloquée. Par exemple, le morceau de code suivant vérifie si l'extension de fichier importée est `PHP` et supprime la requête si c'est le cas :

Code : php

```
$fileName = basename($_FILES["uploadFile"]["name"]);
$extension = pathinfo($fileName, PATHINFO_EXTENSION);
$listenoire = array('php', 'php7', 'phps');

si (dans_tableau($extension, $liste noire)) {
     echo "Type de fichier non autorisé" ;
     mourir();
}

```

Le code prend l'extension de fichier (`$extension`) du nom de fichier téléchargé (`$fileName`), puis le compare à une liste d'extensions sur liste noire (`$blacklist`). Cependant, cette méthode de validation a un défaut majeur. "Ce n'est pas exhaustif", car de nombreuses autres extensions ne sont pas incluses dans cette liste, qui peuvent toujours être utilisées pour exécuter du code PHP sur le serveur principal si elles sont téléchargées.

Conseil : La comparaison ci-dessus est également sensible à la casse et ne prend en compte que les extensions en minuscules. Dans les serveurs Windows, les noms de fichiers ne sont pas sensibles à la casse, nous pouvons donc essayer de télécharger un `php` avec une casse mixte (par exemple `pHp`), qui peut également contourner la liste noire et devrait toujours s'exécuter en tant que script PHP.

Essayons donc d'exploiter cette faiblesse pour contourner la liste noire et uploader un fichier PHP.

* * * * *

Extensions de fuzz
------------------

Comme l'application Web semble tester l'extension de fichier, notre première étape consiste à fuzzer la fonctionnalité de téléchargement avec une liste d'extensions potentielles et à voir laquelle d'entre elles renvoie le message d'erreur précédent. Toute demande de téléchargement qui ne renvoie pas de message d'erreur, renvoie un message différent ou réussit à télécharger le fichier peut indiquer une extension de fichier autorisée.

Il existe de nombreuses listes d'extensions que nous pouvons utiliser dans notre analyse fuzzing. `PayloadsAllTheThings` fournit des listes d'extensions pour [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) et [.NET](https : //github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) applications Web. Nous pouvons également utiliser `SecLists` la liste des [extensions Web] courantes (https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt).

Nous pouvons utiliser l'une des listes ci-dessus pour notre analyse fuzzing. Comme nous testons une application PHP, nous allons télécharger et utiliser la liste [PHP](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) ci-dessus . Ensuite, à partir de `Burp History`, nous pouvons localiser notre dernière requête vers `/upload.php`, faire un clic droit dessus et sélectionner `Send to Intruder`. Dans l'onglet `Positions` , nous pouvons `Effacer` toutes les positions définies automatiquement, puis sélectionner l'extension `.php` dans `filename="HTB.php"` et cliquer sur le bouton `Ajouter` pour l'ajouter en tant que position fuzzing :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_burp_fuzz_extension.jpg)

Nous conserverons le contenu du fichier pour cette attaque, car nous ne sommes intéressés que par le fuzzing des extensions de fichier. Enfin, nous pouvons `Charger` la liste des extensions PHP d'en haut dans l'onglet `Payloads` sous `Options de charge utile`. Nous allons égalementdécochez l'option `URL Encoding` pour éviter d'encoder le (`.`) avant l'extension de fichier. Une fois cela fait, nous pouvons cliquer sur `Lancer l'attaque` pour commencer à fuzzer les extensions de fichiers qui ne sont pas sur la liste noire :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_burp_intruder_result.jpg)

Nous pouvons trier les résultats par `Length`, et nous verrons que toutes les requêtes avec Content-Length (`193`) ont réussi la validation de l'extension, car elles ont toutes répondu avec `Fichier téléchargé avec succès`. En revanche, les autres ont répondu avec un message d'erreur disant "Extension non autorisée".

* * * * *

Extensions non sur liste noire
--------------------------

Maintenant, nous pouvons essayer de télécharger un fichier en utilisant l'une des `extensions autorisées` ci-dessus, et certaines d'entre elles peuvent nous permettre d'exécuter du code PHP. "Toutes les extensions ne fonctionneront pas avec toutes les configurations de serveur Web", nous devrons donc peut-être essayer plusieurs extensions pour en obtenir une qui exécute avec succès le code PHP.

Utilisons l'extension `.phtml` , que les serveurs Web PHP autorisent souvent pour les droits d'exécution de code. Nous pouvons cliquer avec le bouton droit sur sa demande dans les résultats de l'intrus et sélectionner `Envoyer au répéteur`. Maintenant, tout ce que nous avons à faire est de répéter ce que nous avons fait dans les deux sections précédentes en changeant le nom du fichier pour utiliser l'extension `.phtml` et en changeant le contenu en celui d'un shell Web PHP :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php5_web_shell.jpg)

Comme nous pouvons le voir, notre fichier semble bien avoir été uploadé. La dernière étape consiste à visiter notre fichier de téléchargement, qui devrait se trouver sous le répertoire de téléchargement d'images (`profile_images`), comme nous l'avons vu dans la section précédente. Ensuite, nous pouvons tester l'exécution d'une commande, qui devrait confirmer que nous avons réussi à contourner la liste noire et à télécharger notre shell Web :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell.jpg)