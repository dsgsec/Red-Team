Présentation de WPScan
======================

* * * * *

Utiliser WPScan
---------------

[WPScan](https://github.com/wpscanteam/wpscan) est un outil automatisé de scanner et d'énumération WordPress. Il détermine si les différents thèmes et plugins utilisés par un site WordPress sont obsolètes ou vulnérables. Il est installé par défaut sur Parrot OS mais peut également être installé manuellement avec `gem`.

```
dsgsec@htb[/htb]$ gem install wpscan

```

Une fois l'installation terminée, nous pouvons émettre une commande telle que `wpscan --hh`pour vérifier l'installation. Cette commande nous montrera le menu d'utilisation avec tous les commutateurs de ligne de commande disponibles.

```
dsgsec@htb[/htb]$ wpscan --hh
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_\
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.1

       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

Usage: wpscan [options]
        --url URL                                 The URL of the blog to scan
                                                  Allowed Protocols: http, https
                                                  Default Protocol if none provided: http
                                                  This option is mandatory unless update or help or hh or version is/are supplied
    -h, --help                                    Display the simple help and exit
        --hh                                      Display the full help and exit
        --version                                 Display the version and exit
        --ignore-main-redirect                    Ignore the main redirect (if any) and scan the target url
    -v, --verbose                                 Verbose mode
        --[no-]banner                             Whether or not to display the banner
                                                  Default: true
        --max-scan-duration SECONDS               Abort the scan if it exceeds the time provided in seconds
    -o, --output FILE                             Output to FILE
    -f, --format FORMAT                           Output results in the format supplied
                                                  Available choices: cli-no-colour, cli-no-color, json, cli
		<SNIP>

```

Il existe diverses options d'énumération qui peuvent être spécifiées, telles que les plugins vulnérables, tous les plugins, l'énumération des utilisateurs, etc. Il est important de comprendre toutes les options qui s'offrent à nous et d'affiner le scanner en fonction de l'objectif (c'est-à-dire, sommes-nous simplement intéressés de voir si le site WordPress utilise des plugins vulnérables, devons-nous effectuer un audit complet de tous les aspects du site ou sommes-nous simplement intéressés par la création d'une liste d'utilisateurs à utiliser dans une attaque de recherche de mot de passe par force brute ?).

WPScan peut extraire des informations de vulnérabilité provenant de sources externes pour améliorer nos analyses. Nous pouvons obtenir un jeton API auprès de [WPVulnDB](https://wpvulndb.com/) , qui est utilisé par WPScan pour rechercher les vulnérabilités et exploiter les preuves de concept (POC) et les rapports. Le forfait gratuit permet jusqu'à 50 requêtes par jour. Pour utiliser la base de données WPVulnDB, créez simplement un compte et copiez le jeton API depuis la page des utilisateurs. Ce jeton peut ensuite être fourni à WPScan à l'aide du `--api-token`paramètre.

Passez en revue les différentes options de WPScan à l'aide de l'instance Parrot ci-dessous en ouvrant un shell et en exécutant la commande `wpscan --hh`.
