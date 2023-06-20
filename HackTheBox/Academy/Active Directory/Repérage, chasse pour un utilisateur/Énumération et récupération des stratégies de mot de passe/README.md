Énumération et récupération des stratégies de mot de passe
==========================================================

* * * * *

Énumération de la stratégie de mot de passe - à partir de Linux - Credentialed
------------------------------------------------------------------------------

Comme indiqué dans la section précédente, nous pouvons extraire la stratégie de mot de passe du domaine de plusieurs manières, selon la configuration du domaine et si nous disposons ou non d'informations d'identification de domaine valides. Avec des informations d'identification de domaine valides, la politique de mot de passe peut également être obtenue à distance à l'aide d'outils tels que [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) ou `rpcclient`.

```
dsgsec@htb[/htb]$ crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Dumping password info for domain: INLANEFREIGHT
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Minimum password length: 8
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Password history length: 24
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Maximum password age: Not Set
SMB         172.16.5.5      445    ACADEMY-EA-DC01
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Password Complexity Flags: 000001
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Refuse Password Change: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password Store Cleartext: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password Lockout Admins: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password No Clear Change: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password No Anon Change: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  	Domain Password Complex: 1
SMB         172.16.5.5      445    ACADEMY-EA-DC01
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Minimum password age: 1 day 4 minutes
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Reset Account Lockout Counter: 30 minutes
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Locked Account Duration: 30 minutes
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Account Lockout Threshold: 5
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Forced Log off Time: Not Set

```

* * * * *

Énumération de la stratégie de mot de passe - à partir de Linux - Sessions SMB NULL
-----------------------------------------------------------------------------------

Sans informations d'identification, nous pourrons peut-être obtenir la politique de mot de passe via une session SMB NULL ou une liaison anonyme LDAP. La première se fait via une session SMB NULL. Les sessions SMB NULL permettent à un attaquant non authentifié de récupérer des informations du domaine, telles qu'une liste complète des utilisateurs, des groupes, des ordinateurs, des attributs de compte d'utilisateur et la politique de mot de passe du domaine. Les erreurs de configuration de session SMB NULL sont souvent le résultat de la mise à niveau des contrôleurs de domaine hérités, entraînant finalement des configurations non sécurisées, qui existaient par défaut dans les anciennes versions de Windows Server.

Lors de la création d'un domaine dans les versions antérieures de Windows Server, un accès anonyme était accordé à certains partages, ce qui permettait l'énumération des domaines. Une session SMB NULL peut être énumérée facilement. Pour l'énumération, nous pouvons utiliser des outils tels que `enum4linux`, `CrackMapExec`, `rpcclient`, etc.

Nous pouvons utiliser [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) pour vérifier un contrôleur de domaine pour l'accès à la session SMB NULL.

Une fois connecté, nous pouvons émettre une commande RPC pour `querydominfo`obtenir des informations sur le domaine et confirmer l'accès à la session NULL.

#### Utilisation de rpcclient

  Utilisation de rpcclient

```
dsgsec@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
Total Groups:	0
Total Aliases:	37
Sequence No:	1
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1

```

Nous pouvons également obtenir la politique de mot de passe. Nous pouvons voir que la politique de mot de passe est relativement faible, autorisant un mot de passe minimum de 8 caractères.

#### Obtention de la politique de mot de passe à l'aide de rpcclient

  Obtention de la politique de mot de passe à l'aide de rpcclient

```
rpcclient $> querydominfo

Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
Total Groups:	0
Total Aliases:	37
Sequence No:	1
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
rpcclient $> getdompwinfo
min_password_length: 8
password_properties: 0x00000001
	DOMAIN_PASSWORD_COMPLEX

```

* * * * *

Essayons ceci en utilisant [enum4linux](https://labs.portcullis.co.uk/tools/enum4linux) . `enum4linux`est un outil construit autour de la [suite d'outils Samba](https://www.samba.org/samba/docs/current/man-html/samba.7.html) `nmblookup` , `net`et à utiliser pour l'énumération des hôtes et des domaines Windows `rpcclient`. `smbclient`Il peut être trouvé préinstallé sur de nombreuses distributions de tests d'intrusion, y compris Parrot Security Linux. Ci-dessous, nous avons un exemple de sortie affichant des informations pouvant être fournies par `enum4linux`. Voici quelques outils d'énumération courants et les ports qu'ils utilisent :

| Outil | Ports |
| --- | --- |
| nmblookup | 137/UDP |
| nbtstat | 137/UDP |
| filet | 139/TCP, 135/TCP, TCP et UDP 135 et 49152-65535 |
| rpcclient | 135/TCP |
| smbclient | 445/TCP |

#### Utiliser enum4linux

  Utiliser enum4linux

```
dsgsec@htb[/htb]$ enum4linux -P 172.16.5.5

<SNIP>

 ==================================================
|    Password Policy Information for 172.16.5.5    |
 ==================================================

[+] Attaching to 172.16.5.5 using a NULL share
[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:172.16.5.5)

[+] Trying protocol 445/SMB...
[+] Found domain(s):

	[+] INLANEFREIGHT
	[+] Builtin

[+] Password Info for Domain: INLANEFREIGHT

	[+] Minimum password length: 8
	[+] Password history length: 24
	[+] Maximum password age: Not Set
	[+] Password Complexity Flags: 000001

		[+] Domain Refuse Password Change: 0
		[+] Domain Password Store Cleartext: 0
		[+] Domain Password Lockout Admins: 0
		[+] Domain Password No Clear Change: 0
		[+] Domain Password No Anon Change: 0
		[+] Domain Password Complex: 1

	[+] Minimum password age: 1 day 4 minutes
	[+] Reset Account Lockout Counter: 30 minutes
	[+] Locked Account Duration: 30 minutes
	[+] Account Lockout Threshold: 5
	[+] Forced Log off Time: Not Set

[+] Retieved partial password policy with rpcclient:

Password Complexity: Enabled
Minimum Password Length: 8

enum4linux complete on Tue Feb 22 17:39:29 2022

```

L'outil [enum4linux-ng](https://github.com/cddmp/enum4linux-ng) est une réécriture de `enum4linux`Python, mais possède des fonctionnalités supplémentaires telles que la possibilité d'exporter des données sous forme de fichiers YAML ou JSON qui peuvent ensuite être utilisés pour traiter davantage les données ou les transmettre à d'autres outils. Il prend également en charge la sortie couleur, entre autres fonctionnalités

#### Utiliser enum4linux-ng

  Utiliser enum4linux-ng

```
dsgsec@htb[/htb]$ enum4linux-ng -P 172.16.5.5 -oA ilfreight

ENUM4LINUX - next generation

<SNIP>

 =======================================
|    RPC Session Check on 172.16.5.5    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user session
[-] Could not establish random user session: STATUS_LOGON_FAILURE

 =================================================
|    Domain Information via RPC for 172.16.5.5    |
 =================================================
[+] Domain: INLANEFREIGHT
[+] SID: S-1-5-21-3842939050-3880317879-2865463114
[+] Host is part of a domain (not a workgroup)
 =========================================================
|    Domain Information via SMB session for 172.16.5.5    |
========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: ACADEMY-EA-DC01
NetBIOS domain name: INLANEFREIGHT
DNS domain: INLANEFREIGHT.LOCAL
FQDN: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

 =======================================
|    Policies via RPC for 172.16.5.5    |
 =======================================
[*] Trying port 445/tcp
[+] Found policy:
domain_password_information:
  pw_history_length: 24
  min_pw_length: 8
  min_pw_age: 1 day 4 minutes
  max_pw_age: not set
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: true
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: 5
domain_logoff_information:
  force_logoff_time: not set

Completed after 5.41 seconds

```

Enum4linux-ng nous a fourni une sortie un peu plus claire et une sortie JSON et YAML pratique en utilisant le `-oA`drapeau.

#### Affichage du contenu de ilfreight.json

  Affichage du contenu de ilfreight.json

```
dsgsec@htb[/htb]$ cat ilfreight.json

{
    "target": {
        "host": "172.16.5.5",
        "workgroup": ""
    },
    "credentials": {
        "user": "",
        "password": "",
        "random_user": "yxditqpc"
    },
    "services": {
        "SMB": {
            "port": 445,
            "accessible": true
        },
        "SMB over NetBIOS": {
            "port": 139,
            "accessible": true
        }
    },
    "smb_dialects": {
        "SMB 1.0": false,
        "SMB 2.02": true,
        "SMB 2.1": true,
        "SMB 3.0": true,
        "SMB1 only": false,
        "Preferred dialect": "SMB 3.0",
        "SMB signing required": true
    },
    "sessions_possible": true,
    "null_session_possible": true,

<SNIP>

```

Énumération de session nulle - à partir de Windows
--------------------------------------------------

Il est moins courant de faire ce type d'attaque de session nulle à partir de Windows, mais nous pourrions utiliser la commande `net use \\host\ipc$ "" /u:""`pour établir une session nulle à partir d'une machine Windows et confirmer si nous pouvons effectuer plus de ce type d'attaque.

#### Établir une session nulle à partir de Windows

  Établir une session nulle à partir de Windows

```
C:\htb> net use \\DC01\ipc$ "" /u:""
The command completed successfully.

```

Nous pouvons également utiliser une combinaison nom d'utilisateur/mot de passe pour tenter de nous connecter. Voyons quelques erreurs courantes lors de la tentative d'authentification :

#### Erreur : Le compte est désactivé

  Erreur : Le compte est désactivé

```
C:\htb> net use \\DC01\ipc$ "" /u:guest
System error 1331 has occurred.

This user can't sign in because this account is currently disabled.

```

#### Erreur : le mot de passe est incorrect

  Erreur : le mot de passe est incorrect

```
C:\htb> net use \\DC01\ipc$ "password" /u:guest
System error 1326 has occurred.

The user name or password is incorrect.

```

#### Erreur : Le compte est verrouillé (Politique de mot de passe)

  Erreur : Le compte est verrouillé (Politique de mot de passe)

```
C:\htb> net use \\DC01\ipc$ "password" /u:guest
System error 1909 has occurred.

The referenced account is currently locked out and may not be logged on to.

```

Énumération de la stratégie de mot de passe - à partir de Linux - LDAP Anonymous Bind
-------------------------------------------------------------------------------------

[Les liaisons anonymes LDAP](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled) permettent aux attaquants non authentifiés de récupérer des informations du domaine, telles qu'une liste complète des utilisateurs, des groupes, des ordinateurs, des attributs de compte d'utilisateur et la politique de mot de passe du domaine. Il s'agit d'une configuration héritée et, depuis Windows Server 2003, seuls les utilisateurs authentifiés sont autorisés à lancer des requêtes LDAP. Nous voyons encore cette configuration de temps en temps, car un administrateur peut avoir besoin de configurer une application particulière pour autoriser les liaisons anonymes et donner plus que la quantité d'accès prévue, donnant ainsi aux utilisateurs non authentifiés l'accès à tous les objets dans AD.

Avec une liaison anonyme LDAP, nous pouvons utiliser des outils d'énumération spécifiques à LDAP tels que `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, etc., pour extraire la stratégie de mot de passe. Avec [ldapsearch](https://linux.die.net/man/1/ldapsearch) , cela peut être un peu lourd mais faisable. Un exemple de commande pour obtenir la stratégie de mot de passe est la suivante :

#### Utilisation de ldapsearch

  Utilisation de ldapsearch

```
dsgsec@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

forceLogoff: -9223372036854775808
lockoutDuration: -18000000000
lockOutObservationWindow: -18000000000
lockoutThreshold: 5
maxPwdAge: -9223372036854775808
minPwdAge: -864000000000
minPwdLength: 8
modifiedCountAtLastProm: 0
nextRid: 1002
pwdProperties: 1
pwdHistoryLength: 24

```

Ici, nous pouvons voir la longueur minimale du mot de passe de 8, le seuil de verrouillage de 5 et la complexité du mot de passe est définie ( `pwdProperties`définie sur `1`).

* * * * *

Énumération de la stratégie de mot de passe - à partir de Windows
-----------------------------------------------------------------

Si nous pouvons nous authentifier sur le domaine à partir d'un hôte Windows, nous pouvons utiliser des binaires Windows intégrés, par exemple `net.exe`pour récupérer la politique de mot de passe. Nous pouvons également utiliser divers outils tels que PowerView, CrackMapExec porté sur Windows, SharpMapExec, SharpView, etc.

L'utilisation de commandes intégrées est utile si nous atterrissons sur un système Windows et ne pouvons pas y transférer d'outils, ou si nous sommes positionnés sur un système Windows par le client, mais n'avons aucun moyen d'y accéder. Un exemple utilisant le binaire net.exe intégré est :

#### Utilisation de net.exe

  Utilisation de net.exe

```
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.

```

Ici, nous pouvons glaner les informations suivantes :

-   Les mots de passe n'expirent jamais (âge maximal du mot de passe défini sur Illimité)
-   La longueur minimale du mot de passe est de 8, donc des mots de passe faibles sont probablement utilisés
-   Le seuil de verrouillage est de 5 mots de passe erronés
-   Les comptes sont restés verrouillés pendant 30 minutes

Cette politique de mot de passe est excellente pour la pulvérisation de mot de passe. Le minimum de huit caractères signifie que nous pouvons essayer des mots de passe faibles courants tels que `Welcome1`. Le seuil de verrouillage de 5 signifie que nous pouvons tenter 2 à 3 pulvérisations (pour être sûr) toutes les 31 minutes sans risquer de verrouiller des comptes. Si un compte a été verrouillé, il se déverrouillera automatiquement (sans intervention manuelle d'un administrateur) après 30 minutes, mais nous devons éviter de verrouiller `ANY`les comptes à tout prix.

PowerView est également très pratique pour cela :

#### Utilisation de PowerView

  Utilisation de PowerView

```
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy

Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=1; MaximumPasswordAge=-1; MinimumPasswordLength=8; PasswordComplexity=1;
                 PasswordHistorySize=24; LockoutBadCount=5; ResetLockoutCount=30; LockoutDuration=30;
                 RequireLogonToChangePassword=0; ForceLogoffWhenHourExpire=0; ClearTextPassword=0;
                 LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
Version        : @{signature="$CHICAGO$"; Revision=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Path           : \\INLANEFREIGHT.LOCAL\sysvol\INLANEFREIGHT.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHI
                 NE\Microsoft\Windows NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy

```

PowerView nous a donné le même résultat que notre `net accounts`commande, juste dans un format différent, mais a également révélé que la complexité du mot de passe est activée ( `PasswordComplexity=1`).

Comme avec Linux, nous avons de nombreux outils à notre disposition pour récupérer la politique de mot de passe sur un système Windows, qu'il s'agisse de notre système d'attaque ou d'un système fourni par le client. PowerView/SharpView sont toujours de bons paris, tout comme CrackMapExec, SharpMapExec et d'autres. Le choix des outils dépend de l'objectif de l'évaluation, des considérations de furtivité, de tout antivirus ou EDR en place et d'autres restrictions potentielles sur l'hôte cible. Couvrons quelques exemples.

* * * * *

Analyse de la politique de mot de passe
---------------------------------------

Nous avons maintenant retiré la politique de mot de passe de plusieurs façons. Passons en revue la politique du domaine INLANEFREIGHT.LOCAL pièce par pièce.

-   La longueur minimale du mot de passe est de 8 (8 est très courant, mais de nos jours, nous voyons de plus en plus d'organisations appliquer un mot de passe de 10 à 14 caractères, ce qui peut supprimer certaines options de mot de passe pour nous, mais n'atténue pas complètement le vecteur de pulvérisation de mot de passe)
-   Le seuil de verrouillage du compte est de 5 (il n'est pas rare de voir un seuil inférieur tel que 3 ou même aucun seuil de verrouillage défini)
-   La durée de verrouillage est de 30 minutes (cela peut être plus ou moins élevé selon l'organisation), donc si nous verrouillons accidentellement (évitons !!) un compte, il se déverrouillera après le passage de la fenêtre de 30 minutes
-   Les comptes se déverrouillent automatiquement (dans certaines organisations, un administrateur doit déverrouiller manuellement le compte). Nous ne voulons jamais verrouiller des comptes lors de la pulvérisation de mots de passe, mais nous voulons surtout éviter de verrouiller des comptes dans une organisation où un administrateur devrait intervenir et déverrouiller des centaines (ou des milliers) de comptes à la main/script
-   La complexité du mot de passe est activée, ce qui signifie qu'un utilisateur doit choisir un mot de passe avec 3/4 des éléments suivants : une lettre majuscule, une lettre minuscule, un chiffre, un caractère spécial ( `Password1`ou `Welcome1`satisferait l'exigence de "complexité" ici, mais sont toujours des mots de passe clairement faibles) .

La politique de mot de passe par défaut lors de la création d'un nouveau domaine est la suivante, et de nombreuses organisations n'ont jamais modifié cette politique :

| Politique | Valeur par défaut |
| --- | --- |
| Appliquer l'historique des mots de passe | 24 jours |
| Âge maximal du mot de passe | 42 jours |
| Âge minimal du mot de passe | Un jour |
| Longueur minimale du mot de passe | 7 |
| Le mot de passe doit répondre aux exigences de complexité | Activé |
| Stockez les mots de passe à l'aide d'un cryptage réversible | Désactivé |
| Durée de verrouillage du compte | Pas encore défini |
| Seuil de verrouillage du compte | 0 |
| Réinitialiser le compteur de verrouillage de compte après | Pas encore défini |

* * * * *

Prochaines étapes
-----------------

Maintenant que nous avons la politique de mot de passe en main, nous devons créer une liste d'utilisateurs cibles pour effectuer notre attaque par pulvérisation de mot de passe. N'oubliez pas que parfois nous ne pourrons pas obtenir la politique de mot de passe si nous effectuons une pulvérisation de mot de passe externe (ou si nous sommes sur une évaluation interne et ne pouvons pas récupérer la politique en utilisant l'une des méthodes présentées ici). Dans ces cas, nous`MUST`faites preuve d'une extrême prudence pour ne pas verrouiller les comptes. Nous pouvons toujours demander à notre client sa politique de mot de passe si l'objectif est une évaluation aussi complète que possible. Si demander la politique ne correspond pas aux attentes de l'évaluation ou si le client ne veut pas la fournir, nous devons exécuter une, maximum deux, tentatives de pulvérisation de mot de passe (que nous soyons internes ou externes) et attendre plus d'une heure entre tentatives si nous décidons en effet de tenter deux. Bien que la plupart des organisations aient un seuil de verrouillage de 5 tentatives de mot de passe incorrect, une durée de verrouillage de 30 minutes et que les comptes se déverrouillent automatiquement, nous ne pouvons pas toujours compter sur cela pour être normal. J'ai vu de nombreuses organisations avec un seuil de verrouillage de 3, nécessitant qu'un administrateur intervienne et déverrouille les comptes manuellement.

`We do not want to be the pentester that locks out every account in the organization!`

Préparons-nous maintenant à lancer nos attaques par pulvérisation de mot de passe en rassemblant une liste d'utilisateurs cibles.
