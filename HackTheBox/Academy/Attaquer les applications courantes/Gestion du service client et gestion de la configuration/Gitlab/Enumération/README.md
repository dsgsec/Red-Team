Gitlab - Découverte et énumération
================================

* * * * *

[GitLab](https://about.gitlab.com/) est un outil d'hébergement de référentiel Git basé sur le Web qui fournit des fonctionnalités de wiki, un suivi des problèmes et une fonctionnalité de pipeline d'intégration et de déploiement continus. Il est open-source et écrit à l'origine en Ruby, mais la pile technologique actuelle comprend Go, Ruby on Rails et Vue.js. Gitlab a été lancé pour la première fois en 2014 et, au fil des ans, est devenu une entreprise de 1 400 personnes avec un chiffre d'affaires de 150 millions de dollars en 2020. Bien que l'application soit gratuite et open source, ils proposent également une version d'entreprise payante. Voici quelques [statistiques](https://about.gitlab.com/company/) sur GitLab :

- Au moment de la rédaction, l'entreprise compte 1 466 employés
- Gitlab compte plus de 30 millions d'utilisateurs enregistrés situés dans 66 pays
- L'entreprise publie la plupart de ses procédures internes et OKR publiquement sur son site Web
- Certaines entreprises qui utilisent GitLab incluent Drupal, Goldman Sachs, Hackerone, Ticketmaster, Nvidia, Siemens et [more](https://about.gitlab.com/customers/)

GitLab est similaire à GitHub et BitBucket, qui sont également des outils de référentiel Git basés sur le Web. Une comparaison entre les trois peut être vue [ici](https://stackshare.io/stackups/bitbucket-vs-github-vs-gitlab).

Lors des tests d'intrusion internes et externes, il est courant de tomber sur des données intéressantes dans le dépôt GitHub d'une entreprise ou dans une instance GitLab ou BitBucket auto-hébergée. Ces référentiels Git peuvent simplement contenir du code accessible au public, tel que des scripts, pour interagir avec une API. Cependant, nous pouvons également trouver des scripts ou des fichiers de configuration qui ont été accidentellement validés contenant des secrets en texte clair tels que des mots de passe que nous pouvons utiliser à notre avantage. Nous pouvons également rencontrer des clés privées SSH. Nous pouvons essayer d'utiliser la fonction de recherche pour rechercher des utilisateurs, des mots de passe, etc. Des applications telles que GitLab permettent des référentiels publics (qui ne nécessitent aucune authentification), des référentiels internes (disponibles pour les utilisateurs authentifiés) et des référentiels privés (restreints à des utilisateurs spécifiques) . Il vaut également la peine de parcourir tous les référentiels publics de données sensibles et, si l'application le permet, enregistrez un compte et regardez si des référentiels internes intéressants sont accessibles. La plupart des entreprises n'autoriseront qu'un utilisateur avec une adresse e-mail d'entreprise à s'inscrire et exigeront qu'un administrateur autorise le compte, mais comme nous le verrons plus tard, une instance GitLab peut être configurée pour permettre à quiconque de s'inscrire puis de se connecter.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_signup_res.png)

Si nous pouvons obtenir les informations d'identification de l'utilisateur à partir de notre OSINT, nous pourrons peut-être nous connecter à une instance GitLab. L'authentification à deux facteurs est désactivée par défaut.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_2fa.png)

* * * * *

Empreinte & Découverte
------------------------

Nous pouvons rapidement déterminer que GitLab est utilisé dans un environnement en naviguant simplement vers l'URL GitLab, et nous serons dirigés vers la page de connexion, qui affiche le logo GitLab.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_login.png)

La seule façon d'identifier le numéro de version de GitLab utilisé est de naviguer sur la page `/help` lorsque vous êtes connecté. Si l'instance GitLab nous permet d'enregistrer un compte, nous pouvons nous connecter et accéder à cette page pour confirmer la version. Si nous ne pouvons pas enregistrer de compte, nous devrons peut-être essayer un exploit à faible risque tel que [this](https://www.exploit-db.com/exploits/49821). Nous ne recommandons pas de lancer divers exploits sur une application, donc si nous n'avons aucun moyen d'énumérer le numéro de version (comme une date sur la page, le premier commit public ou en enregistrant un utilisateur), alors nous devrions nous en tenir à la chasse aux secrets et ne pas tenter aveuglément plusieurs exploits contre lui. Il y a eu quelques exploits sérieux contre GitLab [12.9.0](https://www.exploit-db.com/exploits/48431) et GitLab [11.4.7](https://www.exploit-db.com /exploits/49257) au cours des dernières années ainsi que GitLab Community Edition [13.10.3](https://www.exploit-db.com/exploits/49821), [13.9.3](https://www .exploit-db.com/exploits/49944) et [13.10.2](https://www.exploit-db.com/exploits/49951).

* * * * *

Énumération
-----------

Nous ne pouvons pas faire grand-chose contre GitLab sans connaître le numéro de version ou être connecté. La première chose que nous devrions essayer est de naviguer vers `/explore` et de voir s'il existe des projets publics susceptibles de contenir quelque chose d'intéressant. En naviguant sur cette page, nous voyons un projet appelé `Inlanefreight dev`. Les projets publics peuvent être intéressants car nous pouvons être en mesure de les utiliser pour en savoir plus sur l'infrastructure de l'entreprise, trouver du code de production dans lequel nous pouvons trouver un bogue après une revue de code, des informations d'identification codées en dur, un script ou un fichier de configuration contenant des informations d'identification, ou d'autres secrets tels qu'une clé privée SSH ou une clé API.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_explore.png)

En parcourant le projet, il ressemble à un exemple de projet et peut ne rien contenir d'utile, même s'il vaut toujours la peinecreuser autour.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_example.png)

À partir de là, nous pouvons explorer chacune des pages liées en haut à droite `groups`, `snippets` et `help`. Nous pouvons également utiliser la fonctionnalité de recherche et voir si nous pouvons découvrir d'autres projets. Une fois que nous avons fini de fouiller dans ce qui est disponible en externe, nous devrions vérifier et voir si nous pouvons créer un compte et accéder à des projets supplémentaires. Supposons que l'organisation n'ait pas configuré GitLab uniquement pour permettre aux e-mails de l'entreprise de s'enregistrer ou pour exiger qu'un administrateur approuve un nouveau compte. Dans ce cas, nous pouvons être en mesure d'accéder à des données supplémentaires.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_signup.png)

Nous pouvons également utiliser le formulaire d'inscription pour énumérer les utilisateurs valides (plus à ce sujet dans la section suivante). Si nous pouvons établir une liste d'utilisateurs valides, nous pourrions tenter de deviner des mots de passe faibles ou éventuellement réutiliser les informations d'identification que nous trouvons à partir d'un vidage de mot de passe à l'aide d'un outil tel que `Dehashed` comme indiqué dans la section osTicket. Ici, nous pouvons voir que l'utilisateur `root` est pris. Nous verrons un autre exemple d'énumération de nom d'utilisateur dans la section suivante. Sur cette instance particulière de GitLab (et probablement d'autres), nous pouvons également énumérer les e-mails. Si nous essayons de nous inscrire avec un e-mail qui a déjà été pris, nous obtiendrons l'erreur "1 erreur interdit l'enregistrement de cet utilisateur : l'e-mail a déjà été pris". Au moment de la rédaction, cette technique d'énumération des noms d'utilisateur fonctionne avec la dernière version de GitLab. Même si la case `Sign-up enabled` est décochée dans la page des paramètres sous `Sign-up restrictions`, nous pouvons toujours accéder à la page `/users/sign_up` et énumérer les utilisateurs, mais nous ne pourrons pas enregistrer un utilisateur.

Certaines mesures d'atténuation peuvent être mises en place pour cela, telles que l'application de 2FA sur tous les comptes d'utilisateurs, l'utilisation de "Fail2Ban" pour bloquer les tentatives de connexion infructueuses qui indiquent des attaques par force brute, et même la restriction des adresses IP pouvant accéder à une instance GitLab si elle doit être accessible en dehors du réseau interne de l'entreprise.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_taken2.png)

Allons-y et enregistrons-nous avec les informations d'identification `hacker:Welcome` et connectez-vous et fouinez. Dès que nous avons terminé l'inscription, nous sommes connectés et amenés à la page du tableau de bord des projets. Si nous allons maintenant sur la page `/explore` , nous remarquons qu'un projet interne `Inlanefreight website` est désormais disponible. En fouillant un peu, cela semble être un site Web statique pour l'entreprise. Supposons qu'il s'agisse d'un autre type d'application (comme PHP). Dans ce cas, nous pourrions éventuellement télécharger la source et l'examiner à la recherche de vulnérabilités ou de fonctionnalités cachées ou trouver des informations d'identification ou d'autres données sensibles.

![](https://academy.hackthebox.com/storage/modules/113/gitlab_internal.png)

Dans un scénario réel, nous pourrons peut-être trouver une quantité considérable de données sensibles si nous pouvons nous enregistrer et accéder à l'un de leurs référentiels. Comme l'explique cet [article de blog](https://tillsongalloway.com/finding-sensitive-information-on-github/index.html), il existe une quantité considérable de données que nous pourrions être en mesure de découvrir sur GitLab, GitHub, etc.

* * * * *

À partir de
-------

Cette section nous montre l'importance (et la puissance) de l'énumération et que toutes les applications que nous découvrons ne doivent pas nécessairement être directement exploitables pour s'avérer toujours très intéressantes et utiles pour nous lors d'un engagement. Cela est particulièrement vrai pour les tests de pénétration externes où la surface d'attaque est généralement considérablement plus petite qu'une évaluation interne. Nous pouvons avoir besoin de collecter des données à partir de deux sources ou plus pour monter une attaque réussie.