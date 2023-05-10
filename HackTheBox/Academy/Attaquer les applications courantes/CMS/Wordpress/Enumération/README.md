WordPress - Découverte et énumération
===================================

* * * * *

[WordPress](https://wordpress.org/), lancé en 2003, est un système de gestion de contenu (CMS) open source qui peut être utilisé à des fins multiples. Il est souvent utilisé pour héberger des blogs et des forums. WordPress est hautement personnalisable et convivial pour le référencement, ce qui le rend populaire parmi les entreprises. Cependant, sa personnalisation et sa nature extensible le rendent sujet aux vulnérabilités via des thèmes et des plugins tiers. WordPress est écrit en PHP et fonctionne généralement sur Apache avec MySQL comme backend.

Au moment de la rédaction de cet article, WordPress représente environ 32,5 % de tous les sites sur Internet et est le CMS le plus populaire en termes de part de marché. Voici quelques [faits](https://hostingtribunal.com/blog/wordpress-statistics/) intéressants sur WordPress.

- WordPress propose plus de 50 000 plugins et plus de 4 100 thèmes sous licence GPL
- 317 versions distinctes de WordPress ont été publiées depuis son lancement initial
- Environ 661 nouveaux sites Web WordPress sont créés chaque jour
- Les blogs WordPress sont écrits dans plus de 120 langues
- Une étude a montré qu'environ 8 % des piratages WordPress sont dus à des mots de passe faibles, tandis que 60 % sont dus à une version obsolète de WordPress
- Selon WPScan, sur près de 4 000 vulnérabilités connues, 54 % proviennent de plugins, 31,5 % de WordPress core et 14,5 % de thèmes WordPress.
- Certaines grandes marques qui utilisent WordPress incluent le New York Times, eBay, Sony, Forbes, Disney, Facebook, Mercedes-Benz et bien d'autres

Comme nous pouvons le voir à partir de ces statistiques, WordPress est extrêmement répandu sur Internet et présente une vaste surface d'attaque. Nous sommes assurés de rencontrer WordPress lors de bon nombre de nos évaluations de test de pénétration externe, et nous devons comprendre comment cela fonctionne, comment l'énumérer et les différentes façons dont il peut être attaqué.

Le module [Hacking WordPress](https://academy.hackthebox.com/course/preview/hacking-wordpress) sur HTB Academy va très loin en profondeur sur la structure et la fonction de WordPress et sur les façons dont il peut être abusé.

Imaginons que lors d'un test d'intrusion externe, nous tombions sur une entreprise qui héberge son site web principal basé sur WordPress. Comme beaucoup d'autres applications, WordPress possède des fichiers individuels qui nous permettent d'identifier cette application. En outre, les fichiers, la structure des dossiers, les noms de fichiers et les fonctionnalités de chaque script PHP peuvent être utilisés pour découvrir même la version installée de WordPress. Dans cette application web, par défaut, les métadonnées sont ajoutées par défaut dans le code source HTML de la page web, qui contient même parfois déjà la version. Par conséquent, voyons quelles possibilités nous avons pour trouver des informations plus détaillées sur WordPress.

* * * * *

Découverte/Empreinte
----------------------

Un moyen rapide d'identifier un site WordPress consiste à accéder au fichier `/robots.txt` . Un fichier robots.txt typique sur une installation WordPress peut ressembler à :

```
Agent utilisateur: *
Interdire : /wp-admin/
Autoriser : /wp-admin/admin-ajax.php
Interdire : /wp-content/uploads/wpforms/

Plan du site : https://inlanefreight.local/wp-sitemap.xml

```

Ici, la présence des répertoires `/wp-admin` et `/wp-content` serait un cadeau mort que nous avons affaire à WordPress. En règle générale, une tentative de navigation vers le répertoire `wp-admin` nous redirigera vers la page `wp-login.php` . Il s'agit du portail de connexion au back-end de l'instance WordPress.

![](https://academy.hackthebox.com/storage/modules/113/wp-login2.png)

WordPress stocke ses plugins dans le répertoire `wp-content/plugins` . Ce dossier est utile pour énumérer les plugins vulnérables. Les thèmes sont stockés dans le répertoire `wp-content/themes`. Ces dossiers doivent être soigneusement énumérés car ils peuvent conduire à des RCE.

Il existe cinq types d'utilisateurs sur une installation WordPress standard.

1. Administrateur : cet utilisateur a accès aux fonctions d'administration du site Web. Cela inclut l'ajout et la suppression d'utilisateurs et de publications, ainsi que la modification du code source.
2. Éditeur : un éditeur peut publier et gérer des publications, y compris les publications d'autres utilisateurs.
3. Auteur : Ils peuvent publier et gérer leurs propres publications.
4. Contributeur : ces utilisateurs peuvent rédiger et gérer leurs propres publications, mais ne peuvent pas les publier.
5. Abonné : il s'agit d'utilisateurs standard qui peuvent parcourir les publications et modifier leurs profils.

L'accès à un administrateur est généralement suffisant pour obtenir l'exécution de code sur le serveur. Les éditeurs et les auteurs peuvent avoir accès à certains plugins vulnérables, ce que les utilisateurs normaux n'ont pas.

* * * * *

Énumération
-----------

Un autre moyen rapide d'identifier un site WordPress consiste à consulter la source de la page. L'affichage de la page avec `cURL` et la recherche de `WordPress` peuvent nous aider à confirmer que WordPress est utilisé et à identifier le numéro de version, que nous devons noter pour plus tard. Nous pouvons énumérer WordPress en utilisant une variété de tactiques manuelles et automatisées.

```
dsgsec@htb[/htb]$ curl -s http://blog.inlanefreight.local | grep WordPress

<meta name="générateur" content="WordPress 5.8" /

```

La navigation sur le site et la lecture de la source de la page vous donnerontDonnez-nous des indices sur le thème utilisé, les plugins installés et même les noms d'utilisateur si les noms d'auteurs sont publiés avec les publications. Nous devrions passer un peu de temps à parcourir manuellement le site et à parcourir la source de la page pour chaque page, rechercher le répertoire `wp-content` , `themes` et `plugin`, et commencer à créer une liste de points de données intéressants.

En regardant la source de la page, nous pouvons voir que le thème [Business Gravity](https://wordpress.org/themes/business-gravity/) est utilisé. Nous pouvons aller plus loin et essayer d'identifier le numéro de version du thème et rechercher toutes les vulnérabilités connues qui l'affectent.

```
dsgsec@htb[/htb]$ curl -s http://blog.inlanefreight.local/ | thèmes grep

<link rel='stylesheet' id='bootstrap-css' href='http://blog.inlanefreight.local/wp-content/themes/business-gravity/assets/vendors/bootstrap/css/bootstrap.min.css ' type='text/css' média='tout' />

```

Ensuite, regardons quels plugins nous pouvons découvrir.

```
dsgsec@htb[/htb]$ curl -s http://blog.inlanefreight.local/ | plugins grep

<link rel='stylesheet' id='contact-form-7-css' href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css ?ver=5.4.2' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.8' id='subscriber-js-js '></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.8' id='validation -engine-en-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.8' id='validation-engine -js'></script>
<link rel='stylesheet' id='mm_frontend-css' href='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.8' type ='text/css' media='tous' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/js/index.js?ver=5.4.2' id= 'contact-form-7-js'></script>

```

D'après le résultat ci-dessus, nous savons que le [Formulaire de contact 7](https://wordpress.org/plugins/contact-form-7/) et [mail-masta](https://wordpress.org/plugins/mail -masta/) les plugins sont installés. La prochaine étape consisterait à énumérer les versions.

La navigation sur `http://blog.inlanefreight.local/wp-content/plugins/mail-masta/` nous montre que la liste des répertoires est activée et qu'un fichier `readme.txt` est présent. Ces fichiers sont très souvent utiles pour identifier les numéros de version. D'après le fichier readme, il semble que la version 1.0.0 du plug-in soit installée, qui souffre d'une [Local File Inclusion](https://www.exploit-db.com/exploits/50226) vulnérabilité qui a été publiée en août 2016. 2021.

Creusons un peu plus. En vérifiant la source de la page d'une autre page, nous pouvons voir que le plug-in [wpDiscuz](https://wpdiscuz.com/) est installé et qu'il semble s'agir de la version 7.0.4

```
dsgsec@htb[/htb]$ curl -s http://blog.inlanefreight.local/?p=1 | plugins grep

<link rel='stylesheet' id='contact-form-7-css' href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css ?ver=5.4.2' type='text/css' media='all' />
<link rel='stylesheet' id='wpdiscuz-frontend-css-css' href='http://blog.inlanefreight.local/wp-content/plugins/wpdiscuz/themes/default/style.css?ver=7.0 .4' type='text/css' media='all' />

```

Une recherche rapide de cette version de plug-in montre [cette](https://www.exploit-db.com/exploits/49967) vulnérabilité d'exécution de code à distance non authentifiée à partir de juin 2021. Nous le noterons et passerons à autre chose. Il est important à ce stade de ne pas nous précipiter et de commencer à exploiter la première faille possible que nous voyons, car il existe de nombreuses autres vulnérabilités potentielles et erreurs de configuration possibles dans WordPress que nous ne voulons pas manquer.

* * * * *

Énumération des utilisateurs
-----------------

Nous pouvons également effectuer une énumération manuelle des utilisateurs. Comme mentionné précédemment, la page de connexion WordPress par défaut se trouve à `/wp-login.php`.

Un nom d'utilisateur valide et un mot de passe invalide entraînent le message suivant :

![](https://academy.hackthebox.com/storage/modules/113/valid_user.png)

Cependant, un nom d'utilisateur invalide indique que l'utilisateur n'a pas été trouvé.

![](https://academy.hackthebox.com/storage/modules/113/invalid_user.png)

Cela rend WordPress vulnérable à l'énumération des noms d'utilisateur, qui peut être utilisée pour obtenir une liste de noms d'utilisateur potentiels.

Résumons. A ce stade, nous avons rassemblé les points de données suivants :

- Le site semble exécuter WordPress core version 5.8
- Le thème installé est Business Gravity
- Les plugins suivants sont utilisés : Formulaire de contact 7, mail-masta, wpDiscuz
- La version de wpDiscuz semble être la 7.0.4, qui souffre d'une vulnérabilité d'exécution de code à distance non authentifiée
- La version mail-masta semble être la 1.0.0, qui souffre d'une vulnérabilité Local File Inclusion
- Le site WordPress est vulnérable à l'énumération des utilisateurs, et l'utilisateur `admin` est confirmé comme étant un utilisateur valideutilisateur

Poussons les choses un peu plus loin et validons/ajoutons à certains de nos points de données avec des analyses d'énumération automatisées du site WordPress. Une fois que nous aurons terminé, nous devrions avoir suffisamment d'informations en main pour commencer à planifier et à monter nos attaques.

* * * * *

WPScan
------

[WPScan](https://github.com/wpscanteam/wpscan) est un scanner WordPress automatisé et un outil d'énumération. Il détermine si les différents thèmes et plugins utilisés par un blog sont obsolètes ou vulnérables. Il est installé par défaut sur Parrot OS mais peut également être installé manuellement avec `gem`.

```
dsgsec@htb[/htb]$ sudo gem install wpscan

```

WPScan est également capable d'extraire des informations de vulnérabilité à partir de sources externes. Nous pouvons obtenir un jeton d'API à partir de [WPVulnDB](https://wpvulndb.com/), qui est utilisé par WPScan pour rechercher les PoC et les rapports. Le plan gratuit permet jusqu'à 75 demandes par jour. Pour utiliser la base de données WPVulnDB, créez simplement un compte et copiez le jeton API à partir de la page des utilisateurs. Ce jeton peut ensuite être fourni à wpscan à l'aide du paramètre `--api-token`.

Taper `wpscan -h` fera apparaître le menu d'aide.

```
dsgsec@htb[/htb]$ wpscan -h

_________________________________________________________________
          __ _______ _____
          \ \ / / __ \ / ____|
           \ \ /\ / /| |__) | (___ ___ __ _ _ __ ®
            \ \/ \/ / | ___/ \___ \ / __|/ _` | '_\
             \ /\ / | | ____) | (__| (_| | | | |
              \/ \/ |_| |_____/ \___|\__,_|_| |_|

          Scanner de sécurité WordPress par l'équipe WPScan
                          Version 3.8.7
        Sponsorisé par Automattic - https://automattic.com/
        @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_________________________________________________________________

Utilisation : wpscan [options]
         --url URL L'URL du blog à analyser
                                                   Protocoles autorisés : http, https
                                                   Protocole par défaut si aucun n'est fourni : http
                                                   Cette option est obligatoire sauf si mise à jour ou aide ou hh ou version est/sont fournis
     -h, --help Afficher l'aide simple et quitter
         --hh Afficher l'aide complète et quitter
         --version Afficher la version et quitter
     -v, --verbose mode verbeux
         --[no-]banner Afficher ou non la bannière
                                                   Par défaut : vrai
     -o, --output FICHIER Sortie vers FICHIER
     -f, --format FORMAT Afficher les résultats dans le format fourni
                                                   Choix disponibles : json, cli-no-color, cli-no-color, cli
         --detection-mode MODE Par défaut : mixte
                                                   Choix disponibles : mixte, passif, agressif

<SNIP>

```

L'indicateur `--enumerate` est utilisé pour énumérer divers composants de l'application WordPress, tels que les plugins, les thèmes et les utilisateurs. Par défaut, WPScan énumère les plugins, thèmes, utilisateurs, médias et sauvegardes vulnérables. Cependant, des arguments spécifiques peuvent être fournis pour limiter l'énumération à des composants spécifiques. Par exemple, tous les plugins peuvent être énumérés à l'aide des arguments `--enumerate ap`. Invoquons une analyse d'énumération normale sur un site Web WordPress avec l'indicateur `--enumerate` et transmettons-lui un jeton d'API de WPVulnDB avec l'indicateur `--api-token` .

```
dsgsec@htb[/htb]$ sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>

<SNIP>

[+] URL : http://blog.inlanefreight.local/ [10.129.42.195]
[+] Début : jeu. 16 sept. 23:11:43 2021

Découverte(s) intéressante(s) :

[+] En-têtes
  | Entrée intéressante : Serveur : Apache/2.4.41 (Ubuntu)
  | Trouvé par : en-têtes (détection passive)
  | Confiance : 100 %

[+] XML-RPC semble être activé : http://blog.inlanefreight.local/xmlrpc.php
  | Trouvé par : accès direct (détection agressive)
  | Confiance : 100 %
  | Les références:
  | - http://codex.wordpress.org/XML-RPC_Pingback_API
  | - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
  | - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
  | - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
  | - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] Lisez-moi WordPress trouvé : http://blog.inlanefreight.local/readme.html
  | Trouvé par : accès direct (détection agressive)
  | Confiance : 100 %

[+] Le répertoire de téléchargement a une liste activée : http://blog.inlanefreight.local/wp-content/uploads/
  | Trouvé par : accès direct (détection agressive)
  | Confiance : 100 %

[+] WordPress version 5.8 identifiée (Non sécurisé, publié le 2021-07-20).
  | Trouvé par : Générateur RSS (Détection passive)
  | - http://blog.inlanefreight.local/?feed=rss2, <geopérateur>https://wordpress.org/?v=5.8</generator>
  | - http://blog.inlanefreight.local/?feed=comments-rss2, <generator>https://wordpress.org/?v=5.8</generator>
  |
  | [!] 3 vulnérabilités identifiées :
  |
  | [!] Titre : WordPress 5.4 à 5.8 - Exposition de données via l'API REST
  | Corrigé dans : 5.8.1
  | Les références:
  | - https://wpvulndb.com/vulnerabilities/38dd7e87-9a22-48e2-bab1-dc79448ecdfb
  | - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-39200
  | - https://wordpress.org/news/2021/09/wordpress-5-8-1-security-and-maintenance-release/
  | - https://github.com/WordPress/wordpress-develop/commit/ca4765c62c65acb732b574a6761bf5fd84595706
  | - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-m9hc-7v5q-x8q5
  |
  | [!] Titre : WordPress 5.4 à 5.8 - XSS authentifié dans l'éditeur de blocs
  | Corrigé dans : 5.8.1
  | Les références:
  | - https://wpvulndb.com/vulnerabilities/5b754676-20f5-4478-8fd3-6bc383145811
  | - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-39201
  | - https://wordpress.org/news/2021/09/wordpress-5-8-1-security-and-maintenance-release/
  | - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-wh69-25hr-h94v
  |
  | [!] Titre : WordPress 5.4 à 5.8 - Mise à jour de la bibliothèque Lodash
  | Corrigé dans : 5.8.1
  | Les références:
  | - https://wpvulndb.com/vulnerabilities/5d6789db-e320-494b-81bb-e678674f4199
  | - https://wordpress.org/news/2021/09/wordpress-5-8-1-security-and-maintenance-release/
  | - https://github.com/lodash/lodash/wiki/Changelog
  | - https://github.com/WordPress/wordpress-develop/commit/fb7ecd92acef6c813c1fde6d9d24a21e02340689

[+] Thème WordPress utilisé : transport-gravity
  | Emplacement : http://blog.inlanefreight.local/wp-content/themes/transport-gravity/
  | Dernière version : 1.0.1 (à jour)
  | Dernière mise à jour : 2020-08-02T00:00:00.000Z
  | Lisez-moi : http://blog.inlanefreight.local/wp-content/themes/transport-gravity/readme.txt
  | [!] La liste des répertoires est activée
  | URL du style : http://blog.inlanefreight.local/wp-content/themes/transport-gravity/style.css
  | Style: Transport Gravity
  | Style URI : https://keonthemes.com/downloads/transport-gravity/
  | Description : Transport Gravity est un thème enfant amélioré de Business Gravity. Transport Gravity est fait pour trans...
  | Auteur : Thèmes Keon
  | URI de l'auteur : https://keonthemes.com/
  |
  | Trouvé par: Style CSS dans la page d'accueil (détection passive)
  | Confirmé par : URL de la page d'accueil (détection passive)
  |
  | Version : 1.0.1 (80 % de confiance)
  | Trouvé par : Style (Détection passive)
  | - http://blog.inlanefreight.local/wp-content/themes/transport-gravity/style.css, Correspondance : 'Version : 1.0.1'

[+] Énumération des plugins vulnérables (via des méthodes passives)
[+] Vérification des versions de plugins (via des méthodes passives et agressives)

[i] Plugin(s) identifié(s) :

[+] mail-masta
  | Emplacement : http://blog.inlanefreight.local/wp-content/plugins/mail-masta/
  | Dernière version : 1.0 (à jour)
  | Dernière mise à jour : 2014-09-19T07:52:00.000Z
  |
  | Trouvé par : URL de la page d'accueil (détection passive)
  |
  | [!] 2 vulnérabilités identifiées :
  |
  | [!] Titre : Mail Masta <= 1.0 - Inclusion de fichiers locaux non authentifiés (LFI)

<SNIP>

| [!] Titre : Mail Masta 1.0 - Multiple SQL Injection

  <SNIP

  | Version : 1.0 (100 % de confiance)
  | Trouvé par : Lisez-moi - Balise stable (détection agressive)
  | - http://blog.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt
  | Confirmé par : Lisez-moi - Section ChangeLog (Détection agressive)
  | - http://blog.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt

<SNIP>

[i] Utilisateur(s) identifié(s) :

[+] par :
administrateur
  | Trouvé par : Messages de l'auteur – Nom d'affichage (détection passive)

[+] administrateur
  | Trouvé par : Générateur RSS (Détection passive)
  | Confirmé par:
  | Identifiant d'auteur Brute Forcing - Modèle d'auteur (détection agressive)
  | Messages d'erreur de connexion (détection agressive)

[+] Jean
  | Trouvé par : Identifiant de l'auteur Brute Forcing - Motif de l'auteur (détection agressive)
  | Confirmé par : Messages d'erreur de connexion (détection agressive)

```

WPScan utilise diverses méthodes passives et actives pour déterminer les versions et les vulnérabilités, comme indiqué dans le rapport ci-dessus. Le nombre de threads utilisés par défaut est `5`. Cependant, cette valeur peut être modifiée à l'aide de l'indicateur `-t`.

Cette analyse nous a aidés à confirmer certaines des choses que nous avons découvertes à partir de l'énumération manuelle (WordPress core version 5.8 et liste de répertoires activée), nous a montré que le thème que nous avons identifié n'était pas exactement correct (Transport Gravity est utilisé, qui est un thème enfant de Business Gravity), a découvert un autre nom d'utilisateur (john) et a montré que l'énumération automatisée à elle seule n'est souvent pas suffisante (manqué les plugins wpDiscuz et Contact Form 7). WPScan fournit des informations sur les vulnérabilités connues. La sortie du rapport contient également des URL vers les PoC, ce qui nous permettrait d'exploiter ces vulnérabilités.

L'approche que nous avons adoptée dans cette section, combinant à la fois l'énumération manuelle et automatisée, peut être appliquée à presque toutes les applications que nous découvrons. Les scanners sont superet sont très utiles mais ne peuvent remplacer le contact humain et un esprit curieux. Le perfectionnement de nos compétences en matière de dénombrement peut nous démarquer de la foule en tant qu'excellents testeurs d'intrusion.

* * * * *

Passer à autre chose
---------

D'après les données que nous avons recueillies manuellement et à l'aide de WPScan, nous savons maintenant ce qui suit :

- Le site exécute WordPress core version 5.8, qui souffre de certaines vulnérabilités qui ne semblent pas intéressantes à ce stade
- Le thème installé est Transport Gravity
- Les plugins suivants sont utilisés : Formulaire de contact 7, mail-masta, wpDiscuz
- La version wpDiscuz est 7.0.4, qui souffre d'une vulnérabilité d'exécution de code à distance non authentifiée
- La version mail-masta est 1.0.0, qui souffre d'une vulnérabilité d'inclusion de fichier local ainsi que d'une injection SQL
- Le site WordPress est vulnérable à l'énumération des utilisateurs, et les utilisateurs `admin` et `john` sont confirmés comme étant des utilisateurs valides
- La liste des répertoires est activée sur l'ensemble du site, ce qui peut entraîner l'exposition de données sensibles
- XML-RPC est activé, ce qui peut être exploité pour effectuer une attaque par force brute de mot de passe contre la page de connexion à l'aide de WPScan, [Metasploit](https://www.rapid7.com/db/modules/auxiliary/scanner/http/ wordpress_xmlrpc_login), etc.

Une fois ces informations notées, passons aux choses amusantes : attaquer WordPress !