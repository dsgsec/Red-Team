Recon-ng
========


[Recon-ng](https://github.com/lanmaster53/recon-ng) est un framework qui aide à automatiser le travail OSINT. Il utilise des modules de différents auteurs et offre une multitude de fonctionnalités. Certains modules nécessitent des clés pour fonctionner ; la clé permet au module d'interroger l'API en ligne associée. Dans cette tâche, nous allons démontrer l'utilisation de Recon-ng dans le terminal.

Du point de vue des tests d'intrusion et de l'équipe rouge, Recon-ng peut être utilisé pour trouver divers éléments d'information pouvant aider à une opération ou à une tâche OSINT. Toutes les données collectées sont automatiquement enregistrées dans la base de données liée à votre espace de travail. Par exemple, vous pouvez découvrir des adresses d'hôte pour effectuer une analyse de port ultérieure ou collecter des adresses e-mail de contact pour des attaques de phishing.

Vous pouvez démarrer Recon-ng en exécutant la commande `recon-ng`. Le démarrage de Recon-ng vous donnera une invite comme `[recon-ng][default] >`. À ce stade, vous devez sélectionner le module installé que vous souhaitez utiliser. Cependant, si c'est la première fois que vous exécutez `recon-ng`, vous devrez installer le(s) module(s) dont vous avez besoin.

Dans cette tâche, nous suivrons le flux de travail suivant :

1.  Créez un espace de travail pour votre projet
2.  Insérer les informations de départ dans la base de données
3.  Recherchez un module sur le marché et renseignez-vous avant de l'installer
4.  Listez les modules installés et chargez-en un
5.  Exécutez le module chargé

### Création d'un espace de travail

Exécutez `workspaces create WORKSPACE_NAME`pour créer un nouvel espace de travail pour votre enquête. Par exemple, `workspaces create thmredteam`créera un espace de travail nommé `thmredteam`.

`recon-ng -w WORKSPACE_NAME`commence la reconnaissance avec l'espace de travail spécifique.

### Amorçage de la base de données

En reconnaissance, vous commencez avec une information et vous la transformez en de nouvelles informations. Par exemple, vous pouvez commencer votre recherche avec un nom d'entreprise et l'utiliser pour découvrir le(s) nom(s) de domaine, les contacts et les profils. Ensuite, vous utiliserez les nouvelles informations que vous avez obtenues pour les transformer davantage et en savoir plus sur votre cible.

Considérons le cas où nous connaissons le nom de domaine de la cible, `thmredteam.com`, et nous aimerions l'alimenter dans la base de données Recon-ng liée à l'espace de travail actif. Si nous voulons vérifier les noms des tables de notre base de données, nous pouvons exécuter `db schema`.

Nous voulons insérer le nom de domaine `thmredteam.com`dans la table des domaines. Nous pouvons le faire en utilisant la commande `db insert domains`.

Terminal Pentesteur

```
pentester@TryHackMe$ recon-ng -w thmredteam
[...]
[recon-ng][thmredteam] > db insert domains
domain (TEXT): thmredteam.com
notes (TEXT):
[*] 1 rows affected.
[recon-ng][thmredteam] > marketplace search
```

### Marché de reconnaissance

Nous avons un nom de domaine, donc une prochaine étape logique serait de rechercher un module qui transforme les domaines en d'autres types d'informations. En supposant que nous partions d'une nouvelle installation de Recon-ng, nous rechercherons des modules appropriés sur le marché.

Avant d'installer des modules à l'aide de la place de marché, voici quelques commandes utiles liées à l'utilisation de la place de marché :

-   `marketplace search KEYWORD`pour rechercher les modules disponibles avec *le mot-clé* .
-   `marketplace info MODULE`pour fournir des informations sur le module en question.
-   `marketplace install MODULE`pour installer le module spécifié dans Recon-ng.
-   `marketplace remove MODULE`pour désinstaller le module spécifié.

Les modules sont regroupés en plusieurs catégories, telles que la découverte, l'importation, la reconnaissance et la création de rapports. De plus, la reconnaissance est également divisée en plusieurs sous-catégories en fonction du type de transformation. Exécutez `marketplace search`pour obtenir une liste de tous les modules disponibles.

Dans le terminal ci-dessous, nous recherchons les modules contenant `domains-`.

Terminal Pentesteur

```
pentester@TryHackMe$ recon-ng -w thmredteam
[...]
[recon-ng][thmredteam] > marketplace search domains-
[*] Searching module index for 'domains-'...

  +---------------------------------------------------------------------------------------------------+
  |                        Path                        | Version |     Status    |  Updated   | D | K |
  +---------------------------------------------------------------------------------------------------+
  | recon/domains-companies/censys_companies           | 2.0     | not installed | 2021-05-10 | * | * |
  | recon/domains-companies/pen                        | 1.1     | not installed | 2019-10-15 |   |   |
  | recon/domains-companies/whoxy_whois                | 1.1     | not installed | 2020-06-24 |   | * |
  | recon/domains-contacts/hunter_io                   | 1.3     | not installed | 2020-04-14 |   | * |
  | recon/domains-contacts/metacrawler                 | 1.1     | not installed | 2019-06-24 | * |   |
  | recon/domains-contacts/pen                         | 1.1     | not installed | 2019-10-15 |   |   |
  | recon/domains-contacts/pgp_search                  | 1.4     | not installed | 2019-10-16 |   |   |
  | recon/domains-contacts/whois_pocs                  | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-contacts/wikileaker                  | 1.0     | not installed | 2020-04-08 |   |   |
  | recon/domains-credentials/pwnedlist/account_creds  | 1.0     | not installed | 2019-06-24 | * | * |
  | recon/domains-credentials/pwnedlist/api_usage      | 1.0     | not installed | 2019-06-24 |   | * |
  | recon/domains-credentials/pwnedlist/domain_creds   | 1.0     | not installed | 2019-06-24 | * | * |
  | recon/domains-credentials/pwnedlist/domain_ispwned | 1.0     | not installed | 2019-06-24 |   | * |
  | recon/domains-credentials/pwnedlist/leak_lookup    | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-credentials/pwnedlist/leaks_dump     | 1.0     | not installed | 2019-06-24 |   | * |
  | recon/domains-domains/brute_suffix                 | 1.1     | not installed | 2020-05-17 |   |   |
  | recon/domains-hosts/binaryedge                     | 1.2     | not installed | 2020-06-18 |   | * |
  | recon/domains-hosts/bing_domain_api                | 1.0     | not installed | 2019-06-24 |   | * |
  | recon/domains-hosts/bing_domain_web                | 1.1     | not installed | 2019-07-04 |   |   |
  | recon/domains-hosts/brute_hosts                    | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-hosts/builtwith                      | 1.1     | not installed | 2021-08-24 |   | * |
  | recon/domains-hosts/censys_domain                  | 2.0     | not installed | 2021-05-10 | * | * |
  | recon/domains-hosts/certificate_transparency       | 1.2     | not installed | 2019-09-16 |   |   |
  | recon/domains-hosts/google_site_web                | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-hosts/hackertarget                   | 1.1     | not installed | 2020-05-17 |   |   |
  | recon/domains-hosts/mx_spf_ip                      | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-hosts/netcraft                       | 1.1     | not installed | 2020-02-05 |   |   |
  | recon/domains-hosts/shodan_hostname                | 1.1     | not installed | 2020-07-01 | * | * |
  | recon/domains-hosts/spyse_subdomains               | 1.1     | not installed | 2021-08-24 |   | * |
  | recon/domains-hosts/ssl_san                        | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-hosts/threatcrowd                    | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-hosts/threatminer                    | 1.0     | not installed | 2019-06-24 |   |   |
  | recon/domains-vulnerabilities/ghdb                 | 1.1     | not installed | 2019-06-26 |   |   |
  | recon/domains-vulnerabilities/xssed                | 1.1     | not installed | 2020-10-18 |   |   |
  +---------------------------------------------------------------------------------------------------+

  D = Has dependencies. See info for details.
  K = Requires keys. See info for details.

[recon-ng][thmredteam] >
```

Nous remarquons de nombreuses sous-catégories sous `recon`, telles que `domains-companies`, `domains-contacts`et `domains-hosts`. Cette dénomination nous indique le type de nouvelles informations que nous obtiendrons de cette transformation. Par exemple, `domains-hosts`signifie que le module trouvera des hôtes liés au domaine fourni.

Certains modules, comme `whoxy_whois`, nécessitent une clé, comme nous pouvons le voir `*`sous la `K`colonne. Cette exigence indique que ce module n'est utilisable que si nous disposons d'une clé pour utiliser le service associé.

D'autres modules ont des dépendances, indiquées par un `*`sous la `D`colonne. Les dépendances montrent que des bibliothèques Python tierces peuvent être nécessaires pour utiliser le module associé.

Disons que cela vous intéresse `recon/domains-hosts/google_site_web`. Pour en savoir plus sur un module particulier, vous pouvez utiliser la commande `marketplace info MODULE`; c'est une commande essentielle qui explique ce que fait le module. Par exemple, `marketplace info google_site_web`fournit la description suivante : "Récolte les hôtes de Google.com en utilisant l'opérateur de recherche 'site'. Met à jour la table 'hosts' avec les résultats." En d'autres termes, ce module utilisera le moteur de recherche Google et l'opérateur « site ».

Nous pouvons installer le module que nous voulons avec la commande `marketplace install MODULE`, par exemple, `marketplace install google_site_web`.

### Travailler avec les modules installés

Nous pouvons travailler avec des modules utilisant :

-   `modules search`pour obtenir une liste de tous les modules installés
-   `modules load MODULE`pour charger un module spécifique en mémoire

Chargeons le module que nous avons installé précédemment à partir du marché, `modules load viewdns_reverse_whois`. Pour `run`cela, nous devons définir les options requises.

-   `options list`pour lister les options que nous pouvons définir pour le module chargé.
-   `options set <option> <value>`pour définir la valeur de l'option.

Dans une étape précédente, nous avons installé le module `google_site_web`, chargeons-le en utilisant `load google_site_web`et exécutons-le avec `run`. Nous avons déjà ajouté le domaine `thmredteam.com`à la base de données. Ainsi, lorsque le module sera exécuté, il lira cette valeur dans la base de données, obtiendra de nouveaux types d'informations et les ajoutera à la base de données à son tour. Les commandes et les résultats sont affichés dans la sortie du terminal ci-dessous.

Terminal Pentesteur

```
pentester@TryHackMe$ recon-ng -w thmredteam
[...]
[recon-ng][thmredteam] > load google_site_web
[recon-ng][thmredteam][google_site_web] > run

--------------
THMREDTEAM.COM
--------------
[*] Searching Google for: site:thmredteam.com
[*] Country: None
[*] Host: cafe.thmredteam.com
[*] Ip_Address: None
[*] Latitude: None
[*] Longitude: None
[*] Notes: None
[*] Region: None
[*] --------------------------------------------------
[*] Country: None
[*] Host: clinic.thmredteam.com
[*] Ip_Address: None
[*] Latitude: None
[*] Longitude: None
[*] Notes: None
[*] Region: None
[*] --------------------------------------------------
[...]
[*] 2 total (2 new) hosts found.
[recon-ng][thmredteam][google_site_web] >
```

Ce module a interrogé Google et découvert deux hôtes, `cafe.thmredteam.com`et `clinic.thmredteam.com`. Il est possible qu'au moment où vous exécutez ces étapes, de nouveaux hôtes apparaissent également.

### Clés

Certains modules ne peuvent pas être utilisés sans clé pour l'API de service respective. `K`indique que vous devez fournir la clé de service appropriée pour utiliser le module en question.

-   `keys list`répertorie les clés
-   `keys add KEY_NAME KEY_VALUE`ajoute une clé
-   `keys remove KEY_NAME`supprime une clé

Une fois que vous avez installé l'ensemble de modules, vous pouvez procéder à leur chargement et à leur exécution.

-   `modules load MODULE`charge un module installé
-   `CTRL + C`décharge le module.
-   `info`pour revoir les informations du module chargé.
-   `options list`répertorie les options disponibles pour le module choisi.
-   `options set NAME VALUE`
-   `run`pour exécuter le module chargé.
