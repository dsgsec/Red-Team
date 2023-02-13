# Enumération des services

## MySQL

MySQL est un système de gestion de base de données relationnelle SQL open source développé et pris en charge par Oracle. Une base de données est simplement une collection structurée de données organisées pour une utilisation et une récupération faciles. Le système de base de données peut traiter rapidement de grandes quantités de données avec des performances élevées. Au sein de la base de données, le stockage des données se fait de manière à occuper le moins d'espace possible. La base de données est contrôlée à l'aide du langage de base de données SQL. MySQL fonctionne selon le principe client-serveur et se compose d'un serveur MySQL et d'un ou plusieurs clients MySQL. Le serveur MySQL est le véritable système de gestion de base de données. Il s'occupe du stockage et de la distribution des données. Les données sont stockées dans des tables avec des colonnes, des lignes et des types de données différents. Ces bases de données sont souvent stockées dans un seul fichier avec l'extension de fichier .sql, par exemple, comme wordpress.sql.

### Client MySQL
Les clients MySQL peuvent récupérer et modifier les données à l'aide de requêtes structurées adressées au moteur de base de données. L'insertion, la suppression, la modification et la récupération de données s'effectuent à l'aide du langage de base de données SQL. Par conséquent, MySQL convient à la gestion de nombreuses bases de données différentes auxquelles les clients peuvent envoyer plusieurs requêtes simultanément. Selon l'utilisation de la base de données, l'accès est possible via un réseau interne ou l'Internet public.

L'un des meilleurs exemples d'utilisation de bases de données est le CMS WordPress. WordPress stocke tous les messages, noms d'utilisateur et mots de passe créés dans sa propre base de données, qui n'est accessible qu'à partir de l'hôte local. Cependant, comme expliqué plus en détail dans le module Introduction aux applications Web, certaines structures de bases de données sont également réparties sur plusieurs serveurs.

### Bases de données MySQL
MySQL est parfaitement adapté aux applications telles que les sites Web dynamiques, où une syntaxe efficace et une vitesse de réponse élevée sont essentielles. Il est souvent combiné avec un système d'exploitation Linux, PHP et un serveur Web Apache et est également connu dans cette combinaison sous le nom de LAMP (Linux, Apache, MySQL, PHP) ou, lors de l'utilisation de Nginx, sous le nom de LEMP. Dans un hébergement Web avec base de données MySQL, cela sert d'instance centrale dans laquelle le contenu requis par les scripts PHP est stocké. Parmi ceux-ci figurent :

+ En-têtes Textes 
+ Balises Meta 
+ Formulaires Clients Noms d'utilisateur Administrateurs Modérateurs Adresses e-mail Informations utilisateur Autorisations Mots de passe
+ Liens externes/internes Liens vers des fichiers Contenus spécifiques Valeurs

### Commandes MySQL
Une base de données MySQL traduit les commandes en interne en code exécutable et exécute les actions demandées. L'application web informe l'utilisateur si une erreur survient lors du traitement, ce que différentes injections SQL peuvent provoquer. Souvent, ces descriptions d'erreurs contiennent des informations importantes et confirment, entre autres, que l'application Web interagit avec la base de données d'une manière différente de celle prévue par les développeurs.

L'application Web renvoie les informations générées au client si les données sont traitées correctement. Ces informations peuvent être des extraits de données d'une table ou des enregistrements nécessaires pour un traitement ultérieur avec des connexions, des fonctions de recherche, etc. Les commandes SQL peuvent afficher, modifier, ajouter ou supprimer des lignes dans les tables. En outre, SQL peut également modifier la structure des tables, créer ou supprimer des relations et des index et gérer les utilisateurs.

MariaDB, qui est souvent connectée à MySQL, est un fork du code MySQL original. En effet, le développeur en chef de MySQL a quitté la société MySQL AB après son acquisition par Oracle et a développé un autre système de gestion de base de données SQL open source basé sur le code source de MySQL et l'a appelé MariaDB.

### Prise d'empreinte
Il existe de nombreuses raisons pour lesquelles un serveur MySQL peut être accessible à partir d'un réseau externe. Néanmoins, c'est loin d'être une des meilleures pratiques, et on peut toujours trouver des bases de données auxquelles on peut accéder. Souvent, ces paramètres n'étaient censés être que temporaires mais ont été oubliés par les administrateurs. Cette configuration de serveur peut également être utilisée comme solution de contournement en raison d'un problème technique. Habituellement, le serveur MySQL s'exécute sur le port TCP 3306, et nous pouvons scanner ce port avec Nmap pour obtenir des informations plus détaillées.

#### Scan MySQL
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-21 00:53 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE     VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-brute: 
|   Accounts: 
|     root:<empty> - Valid credentials
|_  Statistics: Performed 45010 guesses in 5 seconds, average tps: 9002.0
|_mysql-databases: ERROR: Script execution failed (use -d to debug)
|_mysql-dump-hashes: ERROR: Script execution failed (use -d to debug)
| mysql-empty-password: 
|_  root account has empty password
| mysql-enum: 
|   Valid usernames: 
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     web:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 seconds, average tps: 10.0
| mysql-info: 
|   Protocol: 10
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Thread ID: 13
|   Capabilities flags: 65535
|   Some Capabilities: SupportsLoadDataLocal, SupportsTransactions, Speaks41ProtocolOld, LongPassword, DontAllowDatabaseTableColumn, Support41Auth, IgnoreSigpipes, SwitchToSSLAfterHandshake, FoundRows, InteractiveClient, Speaks41ProtocolNew, ConnectWithDatabase, IgnoreSpaceBeforeParenthesis, LongColumnFlag, SupportsCompression, ODBCClient, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: YTSgMfqvx\x0F\x7F\x16\&\x1EAeK>0
|_  Auth Plugin Name: caching_sha2_password
|_mysql-users: ERROR: Script execution failed (use -d to debug)
|_mysql-variables: ERROR: Script execution failed (use -d to debug)
|_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.21 seconds
```

Comme pour toutes nos analyses, nous devons être prudents avec les résultats et confirmer manuellement les informations obtenues car certaines informations pourraient s'avérer être un faux positif. Cette analyse ci-dessus en est un excellent exemple, car nous savons pertinemment que le serveur MySQL cible n'utilise pas un mot de passe vide pour l'utilisateur root, mais un mot de passe fixe. Nous pouvons tester cela avec la commande suivante: 

#### Interaction avec un serveur MySQL
```
dsgsec@htb[/htb]$ mysql -u root -h 10.129.14.132

ERROR 1045 (28000): Access denied for user 'root'@'10.129.14.1' (using password: NO)
```

Par exemple, si nous utilisons un mot de passe que nous avons deviné ou trouvé grâce à nos recherches, nous pourrons nous connecter au serveur MySQL et exécuter certaines commandes.

```
dsgsec@htb[/htb]$ mysql -u root -pP4SSw0rd -h 10.129.14.128

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 150165
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)                                                         
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.                                     
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.                           
      
MySQL [(none)]> show databases;                                                                          
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.006 sec)


MySQL [(none)]> select version();
+-------------------------+
| version()               |
+-------------------------+
| 8.0.27-0ubuntu0.20.04.1 |
+-------------------------+
1 row in set (0.001 sec)


MySQL [(none)]> use mysql;
MySQL [mysql]> show tables;
+------------------------------------------------------+
| Tables_in_mysql                                      |
+------------------------------------------------------+
| columns_priv                                         |
| component                                            |
| db                                                   |
| default_roles                                        |
| engine_cost                                          |
| func                                                 |
| general_log                                          |
| global_grants                                        |
| gtid_executed                                        |
| help_category                                        |
| help_keyword                                         |
| help_relation                                        |
| help_topic                                           |
| innodb_index_stats                                   |
| innodb_table_stats                                   |
| password_history                                     |
...SNIP...
| user                                                 |
+------------------------------------------------------+
37 rows in set (0.002 sec)
```