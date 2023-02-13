# Enumération des services

## MSSQL
Microsoft SQL (MSSQL) est le système de gestion de bases de données relationnelles basé sur SQL de Microsoft. Contrairement à MySQL, dont nous avons parlé dans la dernière section, MSSQL est une source fermée et a été initialement écrit pour fonctionner sur les systèmes d'exploitation Windows. Il est populaire parmi les administrateurs de bases de données et les développeurs lors de la création d'applications qui s'exécutent sur le framework .NET de Microsoft en raison de sa forte prise en charge native de .NET. Il existe des versions de MSSQL qui fonctionneront sous Linux et MacOS, mais nous rencontrerons plus probablement des instances MSSQL sur des cibles exécutant Windows.

### Prise d'empreinte
Il existe de nombreuses façons d'aborder l'empreinte du service MSSQL, plus nous pouvons obtenir de détails avec nos analyses, plus nous pourrons collecter d'informations utiles. NMAP a des scripts mssql par défaut qui peuvent être utilisés pour cibler le port TCP par défaut 1433 sur lequel MSSQL écoute.

L'analyse NMAP scriptée ci-dessous nous fournit des informations utiles. Nous pouvons voir que le nom d'hôte, le nom de l'instance de base de données, la version logicielle de MSSQL et les canaux nommés sont activés. Nous gagnerons à ajouter ces découvertes à nos notes.
```
dsgsec@htb[/htb]$ sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248

Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-08 09:40 EST
Nmap scan report for 10.129.201.248
Host is up (0.15s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: SQL-01
|   NetBIOS_Domain_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Domain_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763

Host script results:
| ms-sql-dac: 
|_  Instance: MSSQLSERVER; DAC port: 1434 (connection failed)
| ms-sql-info: 
|   Windows server name: SQL-01
|   10.129.201.248\MSSQLSERVER: 
|     Instance name: MSSQLSERVER
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|     TCP port: 1433
|     Named pipe: \\10.129.201.248\pipe\sql\query
|_    Clustered: false

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.52 seconds
```
### MSSQL Ping in Metasploit
```
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248

rhosts => 10.129.201.248


msf6 auxiliary(scanner/mssql/mssql_ping) > run

[*] 10.129.201.248:       - SQL Server information for 10.129.201.248:
[+] 10.129.201.248:       -    ServerName      = SQL-01
[+] 10.129.201.248:       -    InstanceName    = MSSQLSERVER
[+] 10.129.201.248:       -    IsClustered     = No
[+] 10.129.201.248:       -    Version         = 15.0.2000.5
[+] 10.129.201.248:       -    tcp             = 1433
[+] 10.129.201.248:       -    np              = \\SQL-01\pipe\sql\query
[*] 10.129.201.248:       - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
### Connexion avec Mssqlclient.py
Si nous pouvons deviner ou accéder aux informations d'identification, cela nous permet de nous connecter à distance au serveur MSSQL et de commencer à interagir avec les bases de données à l'aide de T-SQL (Transact-SQL). L'authentification avec MSSQL nous permettra d'interagir directement avec les bases de données via le moteur de base de données SQL. À partir de Pwnbox ou d'un hôte d'attaque personnel, nous pouvons utiliser mssqlclient.py d'Impacket pour nous connecter, comme indiqué dans la sortie ci-dessous. Une fois connecté au serveur, il peut être bon d'avoir un aperçu du terrain et de lister les bases de données présentes sur le système.

```
dsgsec@htb[/htb]$ python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(SQL-01): Line 1: Changed database context to 'master'.
[*] INFO(SQL-01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208) 
[!] Press help for extra shell commands

SQL> select name from sys.databases

name                                                                                                                               

--------------------------------------------------------------------------------------

master                                                                                                                             

tempdb                                                                                                                             

model                                                                                                                              

msdb                                                                                                                               

Transactions    
```

<hr>