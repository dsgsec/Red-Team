Détournement de session
=======================

* * * * *

Dans les attaques de détournement de session, l'attaquant profite d'identifiants de session non sécurisés, trouve un moyen de les obtenir et les utilise pour s'authentifier auprès du serveur et se faire passer pour la victime.

Un attaquant peut obtenir l'identifiant de session d'une victime en utilisant plusieurs méthodes, la plus courante étant :

-   Reniflage passif du trafic
-   Script intersite (XSS)
-   Historique du navigateur ou log-diving
-   Accès en lecture à une base de données contenant des informations de session

Comme mentionné dans la section précédente, si le niveau de sécurité d'un identifiant de session est faible, un attaquant peut également être en mesure de le forcer brutalement ou même de le prédire.

* * * * *

Exemple de détournement de session
----------------------------------

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`. Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `xss.htb.net`) spécifié pour accéder à l'application.

Un moyen rapide de spécifier ce vhost (et tout autre) dans votre système d'attaque est le suivant :

```
dsgsec@htb[/htb]$ IP=ENTER SPAWNED TARGET IP HERE
dsgsec@htb[/htb]$ printf "%s\t%s\n\n" "$IP" "xss.htb.net csrf.htb.net oredirect.htb.net minilab.htb.net" | sudo tee -a /etc/hosts

```

Partie 1 : Identifier l'identifiant de session

Accédez `http://xss.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : heavycat106
-   Mot de passe : rocknrol

Il s'agit d'un compte que nous avons créé pour examiner l'application !

Vous devriez maintenant être connecté en tant que "Julie Rogers".

À l'aide des outils de développement Web (Maj+Ctrl+I dans le cas de Firefox), notez que l'application utilise un cookie nommé `auth-session`très probablement comme identifiant de session. Double-cliquez sur la valeur de ce cookie et copiez-le ! ![image](https://academy.hackthebox.com/storage/modules/153/17.png)

* * * * *

Partie 2 : Simuler un attaquant

Maintenant, supposons que vous soyez l'attaquant et que vous ayez accès d'une manière ou d'une autre à la `auth-session`valeur du cookie pour l'utilisateur "Julie Rogers".

Ouvrez un `New Private Window`et naviguez à `http://xss.htb.net`nouveau vers. À l'aide des outils de développement Web (Maj+Ctrl+I dans le cas de Firefox), remplacez la `auth-session`valeur actuelle du cookie par celle que vous avez copiée dans la partie 1. Rechargez la page actuelle et vous remarquerez que vous êtes connecté en tant que "Julie Rogers". sans utiliser d'identifiants !

![image](https://academy.hackthebox.com/storage/modules/153/16.png)

Toutes nos félicitations! Vous venez de vous entraîner à votre première attaque de piratage de session !

Veuillez noter que vous pourriez rencontrer des applications Web qui utilisent plus d'un cookie à des fins de suivi de session.

Dans les sections suivantes, nous expliquerons en détail comment vous pouvez monter les attaques de session les plus courantes.
