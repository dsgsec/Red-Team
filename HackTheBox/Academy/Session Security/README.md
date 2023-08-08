Présentation des séances
========================

* * * * *

Une session utilisateur peut être définie comme une séquence de requêtes provenant d'un même client et les réponses associées pendant une période de temps spécifique. Les applications Web modernes doivent maintenir des sessions utilisateur pour suivre les informations et l'état de chaque utilisateur. Les sessions utilisateur facilitent l'attribution des droits d'accès ou d'autorisation, les paramètres de localisation, etc., tandis que les utilisateurs interagissent avec une application, avant et après l'authentification.

HTTP est un protocole de communication sans état, et en tant que tel, toute transaction demande-réponse n'est pas liée aux autres transactions. Cela signifie que chaque demande doit contenir toutes les informations nécessaires pour que le serveur puisse agir de manière appropriée, et l'état de la session réside uniquement du côté du client.

Pour la raison ci-dessus, les applications Web utilisent des cookies, des paramètres d'URL, des arguments d'URL (sur les requêtes GET), des arguments de corps (sur les requêtes POST) et d'autres solutions propriétaires à des fins de suivi et de gestion des sessions.

* * * * *

Sécurité de l'identifiant de session
------------------------------------

Un identifiant de session unique (Session ID) ou jeton est la base sur laquelle les sessions utilisateur sont générées et distinguées.

Nous devons préciser que si un attaquant obtient un identifiant de session, cela peut entraîner un détournement de session, où l'attaquant peut essentiellement se faire passer pour la victime dans l'application Web.

Un attaquant peut obtenir un identifiant de session grâce à une multitude de techniques, qui n'incluent pas toutes d'attaquer activement la victime. Un identifiant de session peut également être :

-   Capturé via le trafic passif/reniflage de paquets
-   Identifié dans les journaux
-   Prédit
-   brutalement forcé

Concentrons-nous maintenant sur la sécurité de l'identifiant de session pendant une minute.

Le niveau de sécurité d'un identifiant de session dépend de :

-   `Validity Scope`(un identifiant de session sécurisée doit être valide pour une seule session)
-   `Randomness`(un identifiant de session sécurisé doit être généré via un algorithme robuste de génération de nombres/chaînes aléatoires afin qu'il ne puisse pas être prédit)
-   `Validity Time`(un identifiant de session sécurisée doit expirer après un certain temps)

Les technologies de programmation établies (PHP, JSP, etc.) génèrent des identifiants de session qui "se conforment" aux exigences ci-dessus concernant la *portée de validité* , *le caractère aléatoire* et *la durée de validité . *Si vous tombez sur une implémentation de génération d'identifiant de session personnalisée, procédez avec une extrême prudence et soyez le plus exhaustif possible dans vos tests.

Le niveau de sécurité d'un identifiant de session dépend également de l'emplacement où il est stocké :

-   `URL`: Si tel est le cas, l'en-tête HTTP *Referer* peut divulguer un identifiant de session à d'autres sites Web. De plus, l'historique du navigateur contiendra également tout identifiant de session stocké dans l'URL.
-   `HTML`: Si tel est le cas, l'identifiant de session peut être identifié à la fois dans la mémoire cache du navigateur et dans tout proxy intermédiaire
-   `sessionStorage`: SessionStorage est une fonctionnalité de stockage de navigateur introduite dans HTML5. Les identifiants de session stockés dans sessionStorage peuvent être récupérés tant que l'onglet ou le navigateur est ouvert. En d'autres termes, les données sessionStorage sont effacées lorsque la *session de la page* se termine. Notez qu'une session de page survit au-delà des rechargements et des restaurations de page.
-   `localStorage`: LocalStorage est une fonctionnalité de stockage de navigateur introduite dans HTML5. Les identifiants de session stockés dans localStorage peuvent être récupérés tant que localStorage n'est pas supprimé par l'utilisateur. En effet, les données stockées dans localStorage ne seront pas supprimées lorsque le processus du navigateur est terminé, à l'exception des sessions de "navigation privée" ou "incognito" où les données stockées dans localStorage sont supprimées au moment où le dernier onglet est fermé.

Les identifiants de session qui sont gérés sans interférence du serveur ou qui ne respectent pas les "caractéristiques" sécurisées ci-dessus doivent être signalés comme faibles.

* * * * *

Attaques de sessions
--------------------

Ce module couvrira différents types d'attaques de session et comment les exploiter. Ceux-ci sont:

-   `Session Hijacking`: Dans les attaques de détournement de session, l'attaquant profite d'identifiants de session non sécurisés, trouve un moyen de les obtenir et les utilise pour s'authentifier auprès du serveur et se faire passer pour la victime.

-   `Session Fixation`: La fixation de session se produit lorsqu'un attaquant peut fixer un identifiant de session (valide). Comme vous pouvez l'imaginer, l'attaquant devra alors tromper la victime pour qu'elle se connecte à l'application en utilisant l'identifiant de session susmentionné. Si la victime le fait, l'attaquant peut procéder à une attaque de Session Hijacking (puisque l'identifiant de session est déjà connu).

-   `XSS (Cross-Site Scripting)`<-- En mettant l'accent sur les sessions utilisateur

-   `CSRF (Cross-Site Request Forgery)`: Cross-Site Request Forgery (CSRF ou XSRF) est une attaque qui force un utilisateur final à exécuter des actions par inadvertance sur une application Web dans laquelle il est actuellement authentifié. Cette attaque est généralement montée à l'aide de pages Web conçues par l'attaquant que la victime doit visiter ou avec lesquelles interagir. Ces pages Web contiennent des requêtes malveillantes qui héritent essentiellement de l'identité et des privilèges de la victime pour exécuter une fonction indésirable au nom de la victime.

-   `Open Redirects`<-- En mettant l'accent sur les sessions utilisateur : une vulnérabilité de redirection ouverte se produit lorsqu'un attaquant peut rediriger une victime vers un site contrôlé par l'attaquant en abusant de la fonctionnalité de redirection d'une application légitime. Dans de tels cas, tout ce que l'attaquant a à faire est de spécifier un site Web sous son contrôle dans une URL de redirection d'un site Web légitime et de transmettre cette URL à la victime. Comme vous pouvez l'imaginer, cela est possible lorsque la fonctionnalité de redirection de l'application légitime n'effectue aucune sorte de validation concernant les sites Web vers lesquels la redirection pointe.

* * * * *

Cibles des modules
------------------

Nous ferons référence à des URL telles que `http://xss.htb.net`tout au long des sections du module et des exercices. Nous utilisons des hôtes virtuels (vhosts) pour héberger les applications Web afin de simuler un environnement vaste et réaliste avec plusieurs serveurs Web. Étant donné que ces vhosts correspondent tous à un répertoire différent sur le même hôte, nous devons effectuer des entrées manuelles dans notre `/etc/hosts`fichier sur la Pwnbox ou la machine virtuelle d'attaque locale pour interagir avec le laboratoire. Cela doit être fait pour tous les exemples montrant des analyses ou des captures d'écran utilisant un FQDN.

Pour le faire rapidement, nous pourrions exécuter ce qui suit (rappelez-vous que le mot de passe de votre utilisateur se trouve dans le `my_credentials.txt`fichier, qui est placé sur le bureau de la Pwnbox) :

```
dsgsec@htb[/htb]$ IP=ENTER SPAWNED TARGET IP HERE
dsgsec@htb[/htb]$ printf "%s\t%s\n\n" "$IP" "xss.htb.net csrf.htb.net oredirect.htb.net minilab.htb.net" | sudo tee -a /etc/hosts

```

Après cette commande, notre `/etc/hosts`fichier ressemblerait à ce qui suit (sur une Pwnbox nouvellement créée) :

```
dsgsec@htb[/htb]$ cat /etc/hosts

# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
127.0.1.1 htb-9zftpkslke.htb-cloud.com htb-9zftpkslke
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

<TARGET IP>	xss.htb.net csrf.htb.net oredirect.htb.net minilab.htb.net

```

Vous pouvez écrire votre propre script ou éditer le fichier hosts à la main, ce qui est bien.

Si vous générez une cible pendant une section et que vous ne pouvez pas y accéder directement via l'IP, assurez-vous de vérifier votre fichier hosts et de mettre à jour toutes les entrées !

Les exercices de module qui nécessitent des vhosts afficheront une liste que vous pouvez utiliser pour modifier votre fichier hosts après avoir généré la machine virtuelle cible au bas de la section respective.

Plongeons maintenant dans chacune des attaques de session et vulnérabilités mentionnées précédemment en détail.
