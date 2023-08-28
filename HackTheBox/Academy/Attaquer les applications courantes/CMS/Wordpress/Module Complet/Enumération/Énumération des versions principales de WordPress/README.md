Énumération des versions principales de WordPress
Il est toujours important de savoir avec quel type d’application nous travaillons. Une partie essentielle de la phase d’énumération consiste à découvrir le numéro de version du logiciel. Ceci est utile lors de la recherche de mauvaises configurations courantes telles que les mots de passe par défaut pouvant être définis pour certaines versions d'une application et lors de la recherche de vulnérabilités connues pour un numéro de version particulier. Nous pouvons utiliser diverses méthodes pour découvrir manuellement le numéro de version. La première étape, et la plus simple, consiste à examiner le code source de la page. Nous pouvons le faire en cliquant avec le bouton droit n'importe où sur la page actuelle et en sélectionnant "Afficher la source de la page" dans le menu ou en utilisant le raccourci clavier [CTRL + U].

Nous pouvons rechercher la meta generatorbalise à l'aide du raccourci [CTRL + F]dans le navigateur ou utiliser cURLavec grepdepuis la ligne de commande pour filtrer ces informations.

Version WP-Code source
Code : html
...SNIP...
<link rel='https://api.w.org/' href='http://blog.inlanefreight.com/index.php/wp-json/' />
<link rel="EditURI" type="application/rsd+xml" title="RSD" href="http://blog.inlanefreight.com/xmlrpc.php?rsd" />
<link rel="wlwmanifest" type="application/wlwmanifest+xml" href="http://blog.inlanefreight.com/wp-includes/wlwmanifest.xml" /> 
<meta name="generator" content="WordPress 5.3.3" />
...SNIP...
  Version WP-Code source
dsgsec@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com | grep '<meta name="generator"'

<meta name="generator" content="WordPress 5.3.3" />
Outre les informations de version, le code source peut également contenir des commentaires pouvant être utiles. Des liens vers CSS (feuilles de style) et JS (JavaScript) peuvent également fournir des indications sur le numéro de version.

Version WP-CSS
Code : html
...SNIP...
<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/bootstrap.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex-style-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/style.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='transportex_color-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/colors/default.css?ver=5.3.3' type='text/css' media='all' />
<link rel='stylesheet' id='smartmenus-css'  href='http://blog.inlanefreight.com/wp-content/themes/ben_theme/css/jquery.smartmenus.bootstrap.css?ver=5.3.3' type='text/css' media='all' />
...SNIP...
Version WP-JS
Code : html
...SNIP...
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-includes/js/jquery/jquery.js?ver=1.12.4-wp'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js?ver=1.4.1'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.3.3'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.3.3'></script>
<script type='text/javascript' src='http://blog.inlanefreight.com/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.3.3'></script>
...SNIP...
Dans les anciennes versions de WordPress, une autre source permettant de découvrir des informations sur la version est le readme.htmlfichier situé dans le répertoire racine de WordPress.

