Structure WordPress
===================

* * * * *

Structure de fichier WordPress par défaut
-----------------------------------------

WordPress peut être installé sur un hôte Windows, Linux ou Mac OSX. Pour ce module, nous nous concentrerons sur une installation WordPress par défaut sur un serveur web Ubuntu Linux. WordPress nécessite une pile LAMP entièrement installée et configurée (système d'exploitation Linux, serveur HTTP Apache, base de données MySQL et langage de programmation PHP) avant l'installation sur un hôte Linux. Après l'installation, tous les fichiers et répertoires prenant en charge WordPress seront accessibles à la racine Web située à l'adresse `/var/www/html`.

Vous trouverez ci-dessous la structure des répertoires d'une installation WordPress par défaut, affichant les fichiers et sous-répertoires clés nécessaires au bon fonctionnement du site Web.

#### Structure du fichier

  Structure du fichier

```
dsgsec@htb[/htb]$ tree -L 1 /var/www/html
.
├── index.php
├── license.txt
├── readme.html
├── wp-activate.php
├── wp-admin
├── wp-blog-header.php
├── wp-comments-post.php
├── wp-config.php
├── wp-config-sample.php
├── wp-content
├── wp-cron.php
├── wp-includes
├── wp-links-opml.php
├── wp-load.php
├── wp-login.php
├── wp-mail.php
├── wp-settings.php
├── wp-signup.php
├── wp-trackback.php
└── xmlrpc.php

```

* * * * *

Fichiers WordPress clés
-----------------------

Le répertoire racine de WordPress contient les fichiers nécessaires pour configurer WordPress pour qu'il fonctionne correctement.

-   `index.php`est la page d'accueil de WordPress.

-   `license.txt`contient des informations utiles telles que la version WordPress installée.

-   `wp-activate.php`est utilisé pour le processus d'activation du courrier électronique lors de la création d'un nouveau site WordPress.

-   `wp-admin`Le dossier contient la page de connexion pour l'accès administrateur et le tableau de bord backend. Une fois connecté, un utilisateur peut apporter des modifications au site en fonction des autorisations qui lui sont attribuées. La page de connexion peut être située à l'un des chemins suivants :

    -   `/wp-admin/login.php`
    -   `/wp-admin/wp-login.php`
    -   `/login.php`
    -   `/wp-login.php`

Ce fichier peut également être renommé pour rendre plus difficile la recherche de la page de connexion.

-   `xmlrpc.php`est un fichier représentant une fonctionnalité de WordPress qui permet de transmettre des données avec HTTP agissant comme mécanisme de transport et XML comme mécanisme d'encodage. Ce type de communication a été remplacé par l' [API REST](https://developer.wordpress.org/rest-api/reference) WordPress .

* * * * *

Fichier de configuration WordPress
----------------------------------

-   Le `wp-config.php`fichier contient les informations requises par WordPress pour se connecter à la base de données, telles que le nom de la base de données, l'hôte de la base de données, le nom d'utilisateur et le mot de passe, les clés et sels d'authentification et le préfixe de la table de la base de données. Ce fichier de configuration peut également être utilisé pour activer le mode DEBUG, ce qui peut être utile en cas de dépannage.

#### wp-config.php

Code : php

```
<?php
/** <SNIP> */
/** The name of the database for WordPress */
define( 'DB_NAME', 'database_name_here' );

/** MySQL database username */
define( 'DB_USER', 'username_here' );

/** MySQL database password */
define( 'DB_PASSWORD', 'password_here' );

/** MySQL hostname */
define( 'DB_HOST', 'localhost' );

/** Authentication Unique Keys and Salts */
/* <SNIP> */
define( 'AUTH_KEY',         'put your unique phrase here' );
define( 'SECURE_AUTH_KEY',  'put your unique phrase here' );
define( 'LOGGED_IN_KEY',    'put your unique phrase here' );
define( 'NONCE_KEY',        'put your unique phrase here' );
define( 'AUTH_SALT',        'put your unique phrase here' );
define( 'SECURE_AUTH_SALT', 'put your unique phrase here' );
define( 'LOGGED_IN_SALT',   'put your unique phrase here' );
define( 'NONCE_SALT',       'put your unique phrase here' );

/** WordPress Database Table prefix */
$table_prefix = 'wp_';

/** For developers: WordPress debugging mode. */
/** <SNIP> */
define( 'WP_DEBUG', false );

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
	define( 'ABSPATH', __DIR__ . '/' );
}

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';

```

* * * * *

Répertoires WordPress clés
--------------------------

-   Le `wp-content`dossier est le répertoire principal où sont stockés les plugins et les thèmes. Le sous-répertoire `uploads/`est généralement l'endroit où sont stockés tous les fichiers téléchargés sur la plate-forme. Ces répertoires et fichiers doivent être soigneusement énumérés car ils peuvent contenir des données sensibles pouvant conduire à l'exécution de code à distance ou à l'exploitation d'autres vulnérabilités ou à des erreurs de configuration.

#### Contenu WP

  Contenu WP

```
dsgsec@htb[/htb]$ tree -L 1 /var/www/html/wp-content
.
├── index.php
├── plugins
└── themes

```

-   `wp-includes`contient tout sauf les composants administratifs et les thèmes qui appartiennent au site Web. Il s'agit du répertoire dans lequel sont stockés les fichiers principaux, tels que les certificats, les polices, les fichiers JavaScript et les widgets.

#### WP-Includes

  WP-Inclut

```
dsgsec@htb[/htb]$ tree -L 1 /var/www/html/wp-includes
.
├── <SNIP>
├── theme.php
├── update.php
├── user.php
├── vars.php
├── version.php
├── widgets
├── widgets.php
├── wlwmanifest.xml
├── wp-db.php
└── wp-diff.php

```
