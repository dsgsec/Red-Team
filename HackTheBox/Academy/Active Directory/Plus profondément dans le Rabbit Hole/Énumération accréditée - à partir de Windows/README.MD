Énumération accréditée - à partir de Windows
============================================

* * * * *

Dans la section précédente, nous avons exploré certains outils que nous pouvons utiliser à partir de notre hôte d'attaque Linux pour l'énumération avec des informations d'identification de domaine valides. Dans cette section, nous allons expérimenter quelques outils d'énumération à partir d'un hôte d'attaque Windows, tels que SharpHound/BloodHound, PowerView/SharpView, Grouper2, Snaffler et certains outils intégrés utiles pour l'énumération AD. Certaines des données que nous recueillons au cours de cette phase peuvent fournir plus d'informations pour les rapports, et pas seulement conduire directement à des voies d'attaque. Selon le type d'évaluation, notre client peut être intéressé par toutes les conclusions possibles, de sorte que même des problèmes tels que la possibilité d'exécuter BloodHound librement ou certains attributs de compte d'utilisateur peuvent valoir la peine d'être inclus dans notre rapport en tant que conclusions à risque moyen ou dans une section d'annexe distincte. Tous les problèmes que nous découvrons ne doivent pas nécessairement viser à transmettre nos attaques.

À ce stade, nous nous intéressons à d'autres erreurs de configuration et problèmes d'autorisation qui pourraient entraîner des mouvements latéraux et verticaux. Nous souhaitons également obtenir une vue d'ensemble de la configuration du domaine, c'est-à-dire, existe-t-il des approbations avec d'autres domaines à l'intérieur et à l'extérieur de la forêt actuelle ? Nous sommes également intéressés par le pillage des partages de fichiers auxquels notre utilisateur a accès, car ceux-ci contiennent souvent des données sensibles telles que des informations d'identification qui peuvent être utilisées pour approfondir notre accès.

* * * * *

TTP
---

Le premier outil que nous allons explorer est le [module ActiveDirectory PowerShell](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) . Lorsque vous atterrissez sur un hôte Windows dans le domaine, en particulier celui utilisé par un administrateur, il est possible que vous trouviez des outils et des scripts précieux sur l'hôte.

* * * * *

Module PowerShell ActiveDirectory
---------------------------------

Le module ActiveDirectory PowerShell est un groupe d'applets de commande PowerShell permettant d'administrer un environnement Active Directory à partir de la ligne de commande. Il se compose de 147 applets de commande différentes au moment de la rédaction. Nous ne pouvons pas tous les couvrir ici, mais nous en examinerons quelques-uns qui sont particulièrement utiles pour énumérer les environnements AD. N'hésitez pas à explorer d'autres applets de commande incluses dans le module dans le laboratoire conçu pour cette section, et voyez quelles combinaisons et résultats intéressants vous pouvez créer.

Avant de pouvoir utiliser le module, nous devons d'abord nous assurer qu'il est importé. L' applet de commande [Get-Module](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-module?view=powershell-7.2) , qui fait partie du [module Microsoft.PowerShell.Core](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/?view=powershell-7.2) , répertorie tous les modules disponibles, leur version et les commandes potentielles à utiliser. C'est un excellent moyen de voir si quelque chose comme Git ou des scripts d'administrateur personnalisés sont installés. Si le module n'est pas chargé, exécutez- `Import-Module ActiveDirectory`le pour le charger et l'utiliser.

#### Découvrir les modules

  Découvrir les modules

```
PS C:\htb> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...

```

Nous verrons que le module ActiveDirectory n'est pas encore importé. Allons-y et importons-le.

#### Charger le module ActiveDirectory

  Charger le module ActiveDirectory

```
PS C:\htb> Import-Module ActiveDirectory
PS C:\htb> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAcc...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...

```

Maintenant que nos modules sont chargés, commençons. Tout d'abord, nous allons énumérer quelques informations de base sur le domaine avec l' applet de commande [Get-ADDomain](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-addomain?view=windowsserver2022-ps) .

### Obtenir des informations sur le domaine

  Charger le module ActiveDirectory

```
PS C:\htb> Get-ADDomain

AllowedDNSSuffixes                 : {}
ChildDomains                       : {LOGISTICS.INLANEFREIGHT.LOCAL}
ComputersContainer                 : CN=Computers,DC=INLANEFREIGHT,DC=LOCAL
DeletedObjectsContainer            : CN=Deleted Objects,DC=INLANEFREIGHT,DC=LOCAL
DistinguishedName                  : DC=INLANEFREIGHT,DC=LOCAL
DNSRoot                            : INLANEFREIGHT.LOCAL
DomainControllersContainer         : OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
DomainMode                         : Windows2016Domain
DomainSID                          : S-1-5-21-3842939050-3880317879-2865463114
ForeignSecurityPrincipalsContainer : CN=ForeignSecurityPrincipals,DC=INLANEFREIGHT,DC=LOCAL
Forest                             : INLANEFREIGHT.LOCAL
InfrastructureMaster               : ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
LastLogonReplicationInterval       :
LinkedGroupPolicyObjects           : {cn={DDBB8574-E94E-4525-8C9D-ABABE31223D0},cn=policies,cn=system,DC=INLANEFREIGHT,
                                     DC=LOCAL, CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=INLAN
                                     EFREIGHT,DC=LOCAL}
LostAndFoundContainer              : CN=LostAndFound,DC=INLANEFREIGHT,DC=LOCAL
ManagedBy                          :
Name                               : INLANEFREIGHT
NetBIOSName                        : INLANEFREIGHT
ObjectClass                        : domainDNS
ObjectGUID                         : 71e4ecd1-a9f6-4f55-8a0b-e8c398fb547a
ParentDomain                       :
PDCEmulator                        : ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
PublicKeyRequiredPasswordRolling   : True
QuotasContainer                    : CN=NTDS Quotas,DC=INLANEFREIGHT,DC=LOCAL
ReadOnlyReplicaDirectoryServers    : {}
ReplicaDirectoryServers            : {ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL}
RIDMaster                          : ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
SubordinateReferences              : {DC=LOGISTICS,DC=INLANEFREIGHT,DC=LOCAL,
                                     DC=ForestDnsZones,DC=INLANEFREIGHT,DC=LOCAL,
                                     DC=DomainDnsZones,DC=INLANEFREIGHT,DC=LOCAL,
                                     CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL}
SystemsContainer                   : CN=System,DC=INLANEFREIGHT,DC=LOCAL
UsersContainer                     : CN=Users,DC=INLANEFREIGHT,DC=LOCAL

```

Cela imprimera des informations utiles telles que le SID du domaine, le niveau fonctionnel du domaine, tous les domaines enfants, etc. Ensuite, nous utiliserons l' applet de commande [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps) . Nous filtrerons les comptes avec la `ServicePrincipalName`propriété renseignée. Cela nous donnera une liste des comptes qui peuvent être sensibles à une attaque Kerberoasting, que nous couvrirons en profondeur après la section suivante.

#### Get-ADUser

  Get-ADUser

```
PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

DistinguishedName    : CN=adfs,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled              : True
GivenName            : Sharepoint
Name                 : adfs
ObjectClass          : user
ObjectGUID           : 49b53bea-4bc4-4a68-b694-b806d9809e95
SamAccountName       : adfs
ServicePrincipalName : {adfsconnect/azure01.inlanefreight.local}
SID                  : S-1-5-21-3842939050-3880317879-2865463114-5244
Surname              : Admin
UserPrincipalName    :

DistinguishedName    : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled              : True
GivenName            : Jessica
Name                 : BACKUPAGENT
ObjectClass          : user
ObjectGUID           : 2ec53e98-3a64-4706-be23-1d824ff61bed
SamAccountName       : backupagent
ServicePrincipalName : {backupjob/veam001.inlanefreight.local}
SID                  : S-1-5-21-3842939050-3880317879-2865463114-5220
Surname              : Systemmailbox 8Cc370d3-822A-4Ab8-A926-Bb94bd0641a9
UserPrincipalName    :

<SNIP>

```

Une autre vérification intéressante que nous pouvons exécuter à l'aide du module ActiveDirectory consiste à vérifier les relations d'approbation de domaine à l'aide de l' applet de commande [Get-ADTrust.](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps)

#### Vérification des relations d'approbation

  Vérification des relations d'approbation

```
PS C:\htb> Get-ADTrust -Filter *

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=LOGISTICS.INLANEFREIGHT.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : False
IntraForest             : True
IsTreeParent            : False
IsTreeRoot              : False
Name                    : LOGISTICS.INLANEFREIGHT.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : f48a1169-2e58-42c1-ba32-a6ccb10057ec
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : LOGISTICS.INLANEFREIGHT.LOCAL
TGTDelegation           : False
TrustAttributes         : 32
TrustedPolicy           :
TrustingPolicy          :
TrustType               : Uplevel
UplevelOnly             : False
UsesAESKeys             : False
UsesRC4Encryption       : False

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=FREIGHTLOGISTICS.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : True
IntraForest             : False
IsTreeParent            : False
IsTreeRoot              : False
Name                    : FREIGHTLOGISTICS.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : 1597717f-89b7-49b8-9cd9-0801d52475ca
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : FREIGHTLOGISTICS.LOCAL
TGTDelegation           : False
TrustAttributes         : 8
TrustedPolicy           :
TrustingPolicy          :
TrustType               : Uplevel
UplevelOnly             : False
UsesAESKeys             : False
UsesRC4Encryption       : False

```

Cette applet de commande imprimera toutes les relations d'approbation du domaine. Nous pouvons déterminer s'il s'agit d'approbations au sein de notre forêt ou avec des domaines dans d'autres forêts, le type d'approbation, la direction de l'approbation et le nom du domaine avec lequel la relation est établie. Cela sera utile plus tard lorsque vous chercherez à tirer parti des relations de confiance enfant-parent et à attaquer à travers les fiducies forestières. Ensuite, nous pouvons collecter des informations sur le groupe AD à l'aide de l'applet de commande [Get-ADGroup](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroup?view=windowsserver2022-ps) .

#### Énumération de groupe

  Énumération de groupe

```
PS C:\htb> Get-ADGroup -Filter * | select name

name
----
Administrators
Users
Guests
Print Operators
Backup Operators
Replicator
Remote Desktop Users
Network Configuration Operators
Performance Monitor Users
Performance Log Users
Distributed COM Users
IIS_IUSRS
Cryptographic Operators
Event Log Readers
Certificate Service DCOM Access
RDS Remote Access Servers
RDS Endpoint Servers
RDS Management Servers
Hyper-V Administrators
Access Control Assistance Operators
Remote Management Users
Storage Replica Administrators
Domain Computers
Domain Controllers
Schema Admins
Enterprise Admins
Cert Publishers
Domain Admins

<SNIP>

```

Nous pouvons prendre les résultats et renvoyer des noms intéressants dans l'applet de commande pour obtenir des informations plus détaillées sur un groupe particulier, comme ceci :

#### Informations détaillées sur le groupe

  Informations détaillées sur le groupe

```
PS C:\htb> Get-ADGroup -Identity "Backup Operators"

DistinguishedName : CN=Backup Operators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
GroupCategory     : Security
GroupScope        : DomainLocal
Name              : Backup Operators
ObjectClass       : group
ObjectGUID        : 6276d85d-9c39-4b7c-8449-cad37e8abc38
SamAccountName    : Backup Operators
SID               : S-1-5-32-551

```

Maintenant que nous en savons plus sur le groupe, obtenons une liste des membres à l'aide de l' applet de commande [Get-ADGroupMember](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroupmember?view=windowsserver2022-ps) .

### Appartenance à un groupe

  Informations détaillées sur le groupe

```
PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"

distinguishedName : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
name              : BACKUPAGENT
objectClass       : user
objectGUID        : 2ec53e98-3a64-4706-be23-1d824ff61bed
SamAccountName    : backupagent
SID               : S-1-5-21-3842939050-3880317879-2865463114-5220

```

Nous pouvons voir qu'un compte, `backupagent`, appartient à ce groupe. Il convient de noter cela car si nous pouvons prendre en charge ce compte de service par le biais d'une attaque, nous pourrions utiliser son appartenance au groupe Opérateurs de sauvegarde pour prendre en charge le domaine. Nous pouvons effectuer ce processus pour les autres groupes afin de bien comprendre la configuration de l'appartenance au domaine. Essayez de répéter le processus avec quelques groupes différents. Vous verrez que ce processus peut être fastidieux et qu'il nous restera une énorme quantité de données à parcourir. Nous devons savoir comment faire cela avec des outils intégrés tels que le module ActiveDirectory PowerShell, mais nous verrons plus loin dans cette section à quel point des outils comme BloodHound peuvent accélérer ce processus et rendre nos résultats beaucoup plus précis et organisés.

L'utilisation du module ActiveDirectory sur un hôte peut être un moyen plus furtif d'effectuer des actions que de déposer un outil sur un hôte ou de le charger en mémoire et d'essayer de l'utiliser. De cette façon, nos actions pourraient potentiellement se fondre davantage. Ensuite, nous allons parcourir l'outil PowerView, qui possède de nombreuses fonctionnalités pour simplifier l'énumération et approfondir le domaine.

* * * * *

PowerView
---------

[PowerView](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon) est un outil écrit en PowerShell pour nous aider à prendre conscience de la situation dans un environnement AD. Tout comme BloodHound, il fournit un moyen d'identifier où les utilisateurs sont connectés sur un réseau, d'énumérer les informations de domaine telles que les utilisateurs, les ordinateurs, les groupes, l'ACLS, les fiducies, de rechercher des partages de fichiers et des mots de passe, d'effectuer Kerberoasting, etc. Il s'agit d'un outil très polyvalent qui peut nous fournir un excellent aperçu de la posture de sécurité du domaine de notre client. Il nécessite plus de travail manuel pour déterminer les erreurs de configuration et les relations au sein du domaine que BloodHound mais, lorsqu'il est utilisé correctement, il peut nous aider à identifier les erreurs de configuration subtiles.

Examinons certaines des fonctionnalités de PowerView et voyons quelles données il renvoie. Le tableau ci-dessous décrit certaines des fonctions les plus utiles offertes par PowerView.

| Commande | Description |
| --- | --- |
| `Export-PowerViewCSV` | Ajouter les résultats à un fichier CSV |
| `ConvertTo-SID` | Convertir un nom d'utilisateur ou de groupe en sa valeur SID |
| `Get-DomainSPNTicket` | Demande le ticket Kerberos pour un compte de nom principal de service (SPN) spécifié |
| Fonctions de domaine/LDAP : |  |
| `Get-Domain` | Renverra l'objet AD pour le domaine actuel (ou spécifié) |
| `Get-DomainController` | Renvoie une liste des contrôleurs de domaine pour le domaine spécifié |
| `Get-DomainUser` | Renverra tous les utilisateurs ou objets utilisateur spécifiques dans AD |
| `Get-DomainComputer` | Renverra tous les ordinateurs ou objets informatiques spécifiques dans AD |
| `Get-DomainGroup` | Renverra tous les groupes ou objets de groupe spécifiques dans AD |
| `Get-DomainOU` | Rechercher tous les objets UO ou des objets spécifiques dans AD |
| `Find-InterestingDomainAcl` | Recherche les ACL d'objets dans le domaine avec des droits de modification définis sur des objets non intégrés |
| `Get-DomainGroupMember` | Renverra les membres d'un groupe de domaine spécifique |
| `Get-DomainFileServer` | Renvoie une liste de serveurs fonctionnant probablement comme des serveurs de fichiers |
| `Get-DomainDFSShare` | Renvoie une liste de tous les systèmes de fichiers distribués pour le domaine actuel (ou spécifié) |
| Fonctions GPO : |  |
| `Get-DomainGPO` | Renverra tous les GPO ou objets GPO spécifiques dans AD |
| `Get-DomainPolicy` | Renvoie la stratégie de domaine par défaut ou la stratégie de contrôleur de domaine pour le domaine actuel |
| Fonctions d'énumération informatique : |  |
| `Get-NetLocalGroup` | Énumère les groupes locaux sur la machine locale ou distante |
| `Get-NetLocalGroupMember` | Énumère les membres d'un groupe local spécifique |
| `Get-NetShare` | Renvoie les partages ouverts sur la machine locale (ou distante) |
| `Get-NetSession` | Retournera les informations de session pour la machine locale (ou distante) |
| `Test-AdminAccess` | Teste si l'utilisateur actuel a un accès administratif à la machine locale (ou distante) |
| Fonctions "méta" filetées : |  |
| `Find-DomainUserLocation` | Trouve les machines sur lesquelles des utilisateurs spécifiques sont connectés |
| `Find-DomainShare` | Trouve les partages accessibles sur les machines du domaine |
| `Find-InterestingDomainShareFile` | Recherche des fichiers correspondant à des critères spécifiques sur des partages lisibles dans le domaine |
| `Find-LocalAdminAccess` | Rechercher des machines sur le domaine local où l'utilisateur actuel dispose d'un accès administrateur local |
| Fonctions d'approbation de domaine : |  |
| `Get-DomainTrust` | Renvoie les approbations de domaine pour le domaine actuel ou un domaine spécifié |
| `Get-ForestTrust` | Renvoie toutes les approbations de forêt pour la forêt actuelle ou une forêt spécifiée |
| `Get-DomainForeignUser` | Énumère les utilisateurs qui sont dans des groupes en dehors du domaine de l'utilisateur |
| `Get-DomainForeignGroupMember` | Énumère les groupes avec des utilisateurs en dehors du domaine du groupe et renvoie chaque membre étranger |
| `Get-DomainTrustMapping` | Énumérera toutes les approbations pour le domaine actuel et toutes les autres vues. |

Ce tableau n'est pas exhaustif pour ce que propose PowerView, mais il inclut de nombreuses fonctions que nous utiliserons à plusieurs reprises. Pour en savoir plus sur PowerView, consultez le [module Active Directory PowerView](https://academy.hackthebox.com/course/preview/active-directory-powerview) . Ci-dessous, nous allons en expérimenter quelques-uns.

La première est la fonction [Get-DomainUser](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainUser/) . Cela nous fournira des informations sur tous les utilisateurs ou des utilisateurs spécifiques que nous spécifions. Ci-dessous, nous l'utiliserons pour récupérer des informations sur un utilisateur spécifique, `mmorgan`.

#### Informations sur l'utilisateur du domaine

  Informations sur l'utilisateur du domaine

```
PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

name                 : Matthew Morgan
samaccountname       : mmorgan
description          :
memberof             : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar
                       Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security
                       Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security
                       Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL...}
whencreated          : 10/27/2021 5:37:06 PM
pwdlastset           : 11/18/2021 10:02:57 AM
lastlogontimestamp   : 2/27/2022 6:34:25 PM
accountexpires       : NEVER
admincount           : 1
userprincipalname    : mmorgan@inlanefreight.local
serviceprincipalname :
mail                 :
useraccountcontrol   : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH

```

Nous avons vu quelques informations utilisateur de base avec PowerView. Énumérons maintenant quelques informations sur les groupes de domaines. Nous pouvons utiliser la fonction [Get-DomainGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainGroupMember/) pour récupérer des informations spécifiques au groupe. L'ajout du `-Recurse`commutateur indique à PowerView que s'il trouve des groupes faisant partie du groupe cible (appartenance à un groupe imbriqué), il doit répertorier les membres de ces groupes. Par exemple, la sortie ci-dessous montre que le `Secadmins`groupe fait partie du `Domain Admins`groupe via l'appartenance au groupe imbriqué. Dans ce cas, nous pourrons voir tous les membres de ce groupe qui héritent des droits d'administrateur de domaine via leur appartenance au groupe.

#### Appartenance à un groupe récursif

  Appartenance à un groupe récursif

```
PS C:\htb>  Get-DomainGroupMember -Identity "Domain Admins" -Recurse

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : svc_qualys
MemberDistinguishedName : CN=svc_qualys,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass       : user
MemberSID               : S-1-5-21-3842939050-3880317879-2865463114-5613

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : sp-admin
MemberDistinguishedName : CN=Sharepoint Admin,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass       : user
MemberSID               : S-1-5-21-3842939050-3880317879-2865463114-5228

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Secadmins
GroupDistinguishedName  : CN=Secadmins,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : spong1990
MemberDistinguishedName : CN=Maggie
                          Jablonski,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass       : user
MemberSID               : S-1-5-21-3842939050-3880317879-2865463114-1965

<SNIP>

```

Ci-dessus, nous avons effectué un examen récursif du `Domain Admins`groupe pour répertorier ses membres. Nous savons maintenant qui cibler pour une élévation potentielle des privilèges. Comme avec le module AD PowerShell, nous pouvons également énumérer les mappages d'approbation de domaine.

#### Énumération de confiance

  Énumération de confiance

```
PS C:\htb> Get-DomainTrustMapping

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM

SourceName      : LOGISTICS.INLANEFREIGHT.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

```

Nous pouvons utiliser la fonction [Test-AdminAccess](https://powersploit.readthedocs.io/en/latest/Recon/Test-AdminAccess/) pour tester l'accès administrateur local sur la machine actuelle ou sur une machine distante.

#### Test de l'accès administrateur local

  Test de l'accès administrateur local

```
PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01

ComputerName    IsAdmin
------------    -------
ACADEMY-EA-MS01    True

```

Ci-dessus, nous avons déterminé que l'utilisateur que nous utilisons actuellement est un administrateur sur l'hôte ACADEMY-EA-MS01. Nous pouvons effectuer la même fonction pour chaque hôte pour voir où nous avons un accès administratif. Nous verrons plus tard dans quelle mesure BloodHound effectue ce type de vérification. Nous pouvons maintenant vérifier les utilisateurs avec l'ensemble d'attributs SPN, ce qui indique que le compte peut être soumis à une attaque Kerberoasting.

#### Recherche d'utilisateurs avec un SPN défini

  Recherche d'utilisateurs avec un SPN défini

```
PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName

serviceprincipalname                          samaccountname
--------------------                          --------------
adfsconnect/azure01.inlanefreight.local       adfs
backupjob/veam001.inlanefreight.local         backupagent
d0wngrade/kerberoast.inlanefreight.local      d0wngrade
kadmin/changepw                               krbtgt
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433 sqldev
MSSQLSvc/SPSJDB.inlanefreight.local:1433      sqlprod
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351 sqlqa
sts/inlanefreight.local                       solarwindsmonitor
testspn/kerberoast.inlanefreight.local        testspn
testspn2/kerberoast.inlanefreight.local       testspn2

```

Testez d'autres fonctions de l'outil jusqu'à ce que vous soyez à l'aise de l'utiliser. Nous verrons PowerView plusieurs fois au fur et à mesure de notre progression dans ce module.

* * * * *

SharpView
---------

PowerView fait partie de la boîte à outils PowerShell offensive PowerSploit, désormais obsolète. L'outil a reçu des mises à jour de BC-Security dans le cadre de leur cadre [Empire 4](https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1) . Empire 4 est le fork de BC-Security du projet Empire original et est activement maintenu à partir d'avril 2022. Nous montrons des exemples tout au long de ce module en utilisant la version de développement de PowerView car c'est un excellent outil de reconnaissance dans un environnement Active Directory, et est toujours extrêmement puissant et utile dans les réseaux AD modernes même si la version originale n'est pas maintenue. La version BC-SECURITY de [PowerView](https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1) a de nouvelles fonctions telles que `Get-NetGmsa`, utilisées pour rechercher [des comptes de service gérés de groupe](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview), qui est hors de portée de ce module. Cela vaut la peine de jouer avec les deux versions pour voir les différences subtiles entre les versions anciennes et actuellement maintenues.

Un autre outil qui mérite d'être expérimenté est SharpView, un port .NET de PowerView. La plupart des mêmes fonctions prises en charge par PowerView peuvent être utilisées avec SharpView. Nous pouvons taper un nom de méthode avec `-Help`pour obtenir une liste d'arguments.

  Recherche d'utilisateurs avec un SPN défini

```
PS C:\htb> .\SharpView.exe Get-DomainUser -Help

Get_DomainUser -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -SPN <Boolean> -AdminCount <Boolean> -AllowDelegation <Boolean> -DisallowDelegation <Boolean> -TrustedToAuth <Boolean> -PreauthNotRequired <Boolean> -KerberosPreauthNotRequired <Boolean> -NoPreauth <Boolean> -Domain <String> -LDAPFilter <String> -Filter <String> -Properties <String[]> -SearchBase <String> -ADSPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimeLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -FindOne <Boolean> -ReturnOne <Boolean> -Credential <NetworkCredential> -Raw <Boolean> -UACFilter <UACEnum>

```

Ici, nous pouvons utiliser SharpView pour énumérer des informations sur un utilisateur spécifique, tel que l'utilisateur `forend`, que nous contrôlons.

  Recherche d'utilisateurs avec un SPN défini

```
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend

[Get-DomainSearcher] search base: LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] filter string: (&(samAccountType=805306368)(|(samAccountName=forend)))
objectsid                      : {S-1-5-21-3842939050-3880317879-2865463114-5614}
samaccounttype                 : USER_OBJECT
objectguid                     : 53264142-082a-4cb8-8714-8158b4974f3b
useraccountcontrol             : NORMAL_ACCOUNT
accountexpires                 : 12/31/1600 4:00:00 PM
lastlogon                      : 4/18/2022 1:01:21 PM
lastlogontimestamp             : 4/9/2022 1:33:21 PM
pwdlastset                     : 2/28/2022 12:03:45 PM
lastlogoff                     : 12/31/1600 4:00:00 PM
badPasswordTime                : 4/5/2022 7:09:07 AM
name                           : forend
distinguishedname              : CN=forend,OU=IT Admins,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
whencreated                    : 2/28/2022 8:03:45 PM
whenchanged                    : 4/9/2022 8:33:21 PM
samaccountname                 : forend
memberof                       : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share G Drive,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL}
cn                             : {forend}
objectclass                    : {top, person, organizationalPerson, user}
badpwdcount                    : 0
countrycode                    : 0
usnchanged                     : 3259288
logoncount                     : 26618
primarygroupid                 : 513
objectcategory                 : CN=Person,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
dscorepropagationdata          : {3/24/2022 3:58:07 PM, 3/24/2022 3:57:44 PM, 3/24/2022 3:52:58 PM, 3/24/2022 3:49:31 PM, 7/14/1601 10:36:49 PM}
usncreated                     : 3054181
instancetype                   : 4
codepage                       : 0

```

Expérimentez avec SharpView sur l'hôte MS01 et recréez autant d'exemples PowerView que possible. Bien que l'évasion ne soit pas dans le cadre de ce module, SharpView peut être utile lorsqu'un client s'est durci contre l'utilisation de PowerShell ou que nous devons éviter d'utiliser PowerShell.

* * * * *

Actions
-------

Les partages permettent aux utilisateurs d'un domaine d'accéder rapidement aux informations pertinentes pour leurs rôles quotidiens et de partager du contenu avec leur organisation. Lorsqu'ils sont correctement configurés, les partages de domaine nécessitent qu'un utilisateur soit joint au domaine et qu'il s'authentifie lors de l'accès au système. Des autorisations seront également en place pour garantir que les utilisateurs ne peuvent accéder et voir que ce qui est nécessaire pour leur rôle quotidien. Des partages trop permissifs peuvent potentiellement entraîner la divulgation accidentelle d'informations sensibles, en particulier celles contenant des données médicales, juridiques, personnelles, RH, etc. conduire à la divulgation de données sensibles telles que des fichiers de configuration ou des fichiers d'authentification comme des clés SSH ou des mots de passe stockés de manière non sécurisée. Nous souhaitons identifier tout problème de ce type pour nous assurer que le client n'expose aucune donnée à des utilisateurs qui n'ont pas besoin d'y accéder pour leur travail quotidien et qu'il respecte toutes les exigences légales/réglementaires auxquelles il est soumis (HIPAA, PCI, etc. .). Nous pouvons utiliser PowerView pour rechercher des partages, puis nous aider à les parcourir ou utiliser diverses commandes manuelles pour rechercher des chaînes courantes telles que des fichiers avec`pass`dans le nom. Cela peut être un processus fastidieux et nous pouvons manquer des choses, en particulier dans les grands environnements. Prenons maintenant le temps d'explorer l'outil `Snaffler`et de voir comment il peut nous aider à identifier ces problèmes avec plus de précision et d'efficacité.

* * * * *

Snaffler
-----

[Snaffler](https://github.com/SnaffCon/Snaffler) est un outil qui peut nous aider à acquérir des informations d'identification ou d'autres données sensibles dans un environnement Active Directory. Snaffler fonctionne en obtenant une liste d'hôtes dans le domaine, puis en énumérant ces hôtes pour les partages et les répertoires lisibles. Une fois cela fait, il parcourt tous les répertoires lisibles par notre utilisateur et recherche les fichiers qui pourraient servir à améliorer notre position dans l'évaluation. Snaffler nécessite qu'il soit exécuté à partir d'un hôte joint à un domaine ou dans un contexte d'utilisateur de domaine.

Pour exécuter Snaffler, nous pouvons utiliser la commande ci-dessous :

#### Exécution du filet

Code : bash

```
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data

```

Le `-s`lui dit d'imprimer les résultats sur la console pour nous, le `-d`spécifie le domaine dans lequel rechercher et le `-o`dit à Snaffler d'écrire les résultats dans un fichier journal. L' `-v`option est le niveau de verbosité. Il est généralement `data`préférable car il n'affiche que les résultats à l'écran, il est donc plus facile de commencer à parcourir les exécutions de l'outil. Snaffler peut produire une quantité considérable de données, nous devons donc généralement sortir dans un fichier et le laisser s'exécuter, puis y revenir plus tard. Il peut également être utile de fournir la sortie brute de Snaffler aux clients en tant que données supplémentaires lors d'un test de pénétration, car cela peut les aider à se concentrer sur les actions de grande valeur qui doivent être verrouillées en premier.

#### Filet en action

  Filet en action

```
PS C:\htb> .\Snaffler.exe  -d INLANEFREIGHT.LOCAL -s -v data

 .::::::.:::.    :::.  :::.    .-:::::'.-:::::':::    .,:::::: :::::::..
;;;`    ``;;;;,  `;;;  ;;`;;   ;;;'''' ;;;'''' ;;;    ;;;;'''' ;;;;``;;;;
'[==/[[[[, [[[[[. '[[ ,[[ '[[, [[[,,== [[[,,== [[[     [[cccc   [[[,/[[['
  '''    $ $$$ 'Y$c$$c$$$cc$$$c`$$$'`` `$$$'`` $$'     $$""   $$$$$$c
 88b    dP 888    Y88 888   888,888     888   o88oo,.__888oo,__ 888b '88bo,
  'YMmMY'  MMM     YM YMM   ''` 'MM,    'MM,  ''''YUMMM''''YUMMMMMMM   'W'
                         by l0ss and Sh3r4 - github.com/SnaffCon/Snaffler

2022-03-31 12:16:54 -07:00 [Share] {Black}(\\ACADEMY-EA-MS01.INLANEFREIGHT.LOCAL\ADMIN$)
2022-03-31 12:16:54 -07:00 [Share] {Black}(\\ACADEMY-EA-MS01.INLANEFREIGHT.LOCAL\C$)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-MX01.INLANEFREIGHT.LOCAL\address)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\User Shares)
2022-03-31 12:16:54 -07:00 [Share] {Green}(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\ZZZ_archive)
2022-03-31 12:17:18 -07:00 [Share] {Green}(\\ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL\CertEnroll)
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.kdb$|289B|3/31/2022 12:09:22 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\GroupBackup.kdb) .kdb
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.key$|299B|3/31/2022 12:05:33 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\ShowReset.key) .key
2022-03-31 12:17:19 -07:00 [Share] {Green}(\\ACADEMY-EA-FILE.INLANEFREIGHT.LOCAL\UpdateServicesPackages)
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.kwallet$|302B|3/31/2022 12:04:45 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\WriteUse.kwallet) .kwallet
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.key$|298B|3/31/2022 12:05:10 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\ProtectStep.key) .key
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.ppk$|275B|3/31/2022 12:04:40 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\StopTrace.ppk) .ppk
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.key$|301B|3/31/2022 12:09:17 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\WaitClear.key) .key
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.sqldump$|312B|3/31/2022 12:05:30 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\DenyRedo.sqldump) .sqldump
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.sqldump$|310B|3/31/2022 12:05:02 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\AddPublish.sqldump) .sqldump
2022-03-31 12:17:19 -07:00 [Share] {Green}(\\ACADEMY-EA-FILE.INLANEFREIGHT.LOCAL\WsusContent)
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.keychain$|295B|3/31/2022 12:08:42 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\SetStep.keychain) .keychain
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.tblk$|279B|3/31/2022 12:05:25 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\FindConnect.tblk) .tblk
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.psafe3$|301B|3/31/2022 12:09:33 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\GetUpdate.psafe3) .psafe3
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.keypair$|278B|3/31/2022 12:09:09 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Infosec\UnprotectConvertTo.keypair) .keypair
2022-03-31 12:17:19 -07:00 [File] {Black}<KeepExtExactBlack|R|^\.tblk$|280B|3/31/2022 12:05:17 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\ExportJoin.tblk) .tblk
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.mdf$|305B|3/31/2022 12:09:27 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\FormatShow.mdf) .mdf
2022-03-31 12:17:19 -07:00 [File] {Red}<KeepExtExactRed|R|^\.mdf$|299B|3/31/2022 12:09:14 PM>(\\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL\Department Shares\IT\Development\LockConfirm.mdf) .mdf

<SNIP>

```

Nous pouvons trouver des mots de passe, des clés SSH, des fichiers de configuration ou d'autres données pouvant être utilisées pour approfondir notre accès. Snaffler code la couleur de la sortie pour nous et nous fournit un aperçu des types de fichiers trouvés dans les partages.

Maintenant que nous disposons d'une multitude de données sur le domaine INLANEFREIGHT.LOCAL (et, espérons-le, de notes claires et de la sortie du fichier journal !), nous avons besoin d'un moyen de les corréler et de les visualiser. Approfondissons `BloodHound`et voyons à quel point cet outil peut être puissant lors de toute évaluation de sécurité axée sur AD.

* * * * *

### SharpHound

Comme indiqué dans la section précédente, `Bloodhound`est un outil open source exceptionnel qui peut identifier les chemins d'attaque dans un environnement AD en analysant les relations entre les objets. Les testeurs d'intrusion et les équipes bleues peuvent bénéficier de l'apprentissage de l'utilisation de BloodHound pour visualiser les relations dans le domaine. Lorsqu'il est utilisé correctement et associé à des requêtes Cipher personnalisées, BloodHound peut trouver des failles à fort impact, mais difficiles à découvrir, qui sont présentes dans le domaine depuis des années.

Tout d'abord, nous devons nous authentifier en tant qu'utilisateur de domaine à partir d'un hôte d'attaque Windows positionné dans le réseau (mais non joint au domaine) ou transférer l'outil vers un hôte joint au domaine. Il existe de nombreuses façons d'y parvenir dans le module [de transfert de fichiers](https://academy.hackthebox.com/course/preview/file-transfers) . Pour nos besoins, nous travaillerons déjà avec SharpHound.exe sur l'hôte d'attaque, mais cela vaut la peine d'expérimenter le transfert de l'outil vers l'hôte d'attaque depuis Pwnbox ou notre propre machine virtuelle en utilisant des méthodes telles qu'un serveur HTTP Python, smbserver.py d'Impacket, etc.

Si nous exécutons SharpHound avec l' `--help`option, nous pouvons voir les options qui s'offrent à nous.

#### SharpHound en action

  SharpHound en action

```
PS C:\htb>  .\SharpHound.exe --help

SharpHound 1.0.3
Copyright (C) 2022 SpecterOps

  -c, --collectionmethods    (Default: Default) Collection Methods: Container, Group, LocalGroup, GPOLocalGroup,
                             Session, LoggedOn, ObjectProps, ACL, ComputerOnly, Trusts, Default, RDP, DCOM, DCOnly

  -d, --domain               Specify domain to enumerate

  -s, --searchforest         (Default: false) Search all available domains in the forest

  --stealth                  Stealth Collection (Prefer DCOnly whenever possible!)

  -f                         Add an LDAP filter to the pregenerated filter.

  --distinguishedname        Base DistinguishedName to start the LDAP search at

  --computerfile             Path to file containing computer names to enumerate

  <SNIP>

```

Nous allons commencer par exécuter le collecteur SharpHound.exe à partir de l'hôte d'attaque MS01.

  SharpHound en action

```
PS C:\htb> .\SharpHound.exe -c All --zipfilename ILFREIGHT

2022-04-18T13:58:22.1163680-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-04-18T13:58:22.1163680-07:00|INFORMATION|Initializing SharpHound at 1:58 PM on 4/18/2022
2022-04-18T13:58:22.6788709-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-04-18T13:58:23.0851206-07:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.LOCAL
2022-04-18T13:58:53.9132950-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 67 MB RAM
2022-04-18T13:59:15.7882419-07:00|INFORMATION|Producer has finished, closing LDAP channel
2022-04-18T13:59:16.1788930-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2022-04-18T13:59:23.9288698-07:00|INFORMATION|Status: 3793 objects finished (+3793 63.21667)/s -- Using 112 MB RAM
2022-04-18T13:59:45.4132561-07:00|INFORMATION|Consumers finished, closing output channel
Closing writers
2022-04-18T13:59:45.4601086-07:00|INFORMATION|Output channel closed, waiting for output task to complete
2022-04-18T13:59:45.8663528-07:00|INFORMATION|Status: 3809 objects finished (+16 46.45122)/s -- Using 110 MB RAM
2022-04-18T13:59:45.8663528-07:00|INFORMATION|Enumeration finished in 00:01:22.7919186
2022-04-18T13:59:46.3663660-07:00|INFORMATION|SharpHound Enumeration Completed at 1:59 PM on 4/18/2022! Happy Graphing

```

Ensuite, nous pouvons exfiltrer l'ensemble de données sur notre propre machine virtuelle ou l'ingérer dans l'outil d'interface graphique BloodHound sur MS01. Nous pouvons le faire sur MS01 en tapant `bloodhound`dans une console CMD ou PowerShell. Les informations d'identification doivent être enregistrées, mais saisissez-les `neo4j: HTB_@cademy_stdnt!`si une invite s'affiche. Ensuite, cliquez sur le `Upload Data`bouton sur le côté droit, sélectionnez le fichier zip nouvellement généré et cliquez sur `Open`. Une `Upload Progress`fenêtre apparaîtra. Une fois que tous les fichiers .json sont terminés à 100 %, cliquez sur le X en haut de cette fenêtre.

Nous pouvons commencer par taper `domain:`dans la barre de recherche en haut à gauche et choisir `INLANEFREIGHT.LOCAL`parmi les résultats. Prenez un moment pour parcourir l'onglet d'informations sur le nœud. Comme nous pouvons le voir, il s'agirait d'une entreprise assez importante avec plus de 550 hôtes à cibler et des fiducies avec deux autres domaines.

Voyons maintenant quelques requêtes prédéfinies dans l' `Analysis`onglet. La requête `Find Computers with Unsupported Operating Systems`est idéale pour trouver des systèmes d'exploitation obsolètes et non pris en charge exécutant des logiciels hérités. Ces systèmes sont relativement courants dans les réseaux d'entreprise (en particulier les environnements plus anciens), car ils exécutent souvent des produits qui ne peuvent pas encore être mis à jour ou remplacés. Garder ces hôtes à proximité peut économiser de l'argent, mais ils peuvent également ajouter des vulnérabilités inutiles au réseau. Les hôtes plus anciens peuvent être sensibles aux anciennes vulnérabilités d'exécution de code à distance comme [MS08-067](https://support.microsoft.com/en-us/topic/ms08-067-vulnerability-in-server-service-could-allow-remote-code-execution-ac7878fc-be69-7143-472d-2507a179cd15). Si nous rencontrons ces anciens hébergeurs lors d'une évaluation, nous devons être prudents avant de les attaquer (voire de vérifier auprès de notre client) car ils peuvent être fragiles et exécuter une application ou un service critique. Nous pouvons conseiller à notre client de segmenter autant que possible ces hôtes du reste du réseau s'ils ne peuvent pas encore les supprimer, mais nous devrions également leur recommander de commencer à élaborer un plan de mise hors service et de les remplacer.

Cette requête montre deux hôtes, l'un exécutant Windows 7 et l'autre exécutant Windows Server 2008 (les deux ne sont pas "en direct" dans notre laboratoire). Parfois, nous verrons des hôtes qui ne sont plus sous tension mais qui apparaissent toujours sous forme d'enregistrements dans AD. Nous devons toujours valider s'ils sont "en direct" ou non avant de faire des recommandations dans nos rapports. Nous pouvons rédiger une découverte à haut risque pour les systèmes d'exploitation hérités ou une recommandation de bonnes pratiques pour nettoyer les anciens enregistrements dans AD.

#### Systèmes d'exploitation non pris en charge

![texte](https://academy.hackthebox.com/storage/modules/143/unsupported.png)

Nous verrons souvent des utilisateurs avec des droits d'administrateur locaux sur leur hôte (peut-être temporairement pour installer un logiciel, et les droits n'ont jamais été supprimés), ou ils occupent un rôle suffisamment élevé dans l'organisation pour exiger ces droits (qu'ils en aient besoin ou pas). D'autres fois, nous verrons des droits d'administrateur locaux excessifs répartis dans l'ensemble de l'organisation, tels que plusieurs groupes du service informatique avec un administrateur local sur des groupes de serveurs ou même l'ensemble du groupe d'utilisateurs du domaine avec un administrateur local sur un ou plusieurs hôtes. Cela peut nous être bénéfique si nous reprenons un compte utilisateur avec ces droits sur une ou plusieurs machines. Nous pouvons exécuter la requête`Find Computers where Domain Users are Local Admin`pour voir rapidement s'il existe des hôtes où tous les utilisateurs ont des droits d'administrateur local. Si tel est le cas, alors tout compte que nous contrôlons peut généralement être utilisé pour accéder au(x) hôte(s) en question, et nous pouvons être en mesure de récupérer les informations d'identification de la mémoire ou de trouver d'autres données sensibles.

#### Administrateurs locaux

![texte](https://academy.hackthebox.com/storage/modules/143/local-admin.png)

Ceci n'est qu'un aperçu des requêtes utiles que nous pouvons exécuter. Au fur et à mesure que nous poursuivons dans ce module, vous en verrez plusieurs autres qui peuvent être utiles pour trouver d'autres faiblesses dans le domaine. Pour une étude plus approfondie sur BloodHound, consultez le module [Active Directory Bloodhound](https://academy.hackthebox.com/course/preview/active-directory-bloodhound) . Prenez le temps d'essayer chacune des requêtes de l' `Analysis`onglet pour vous familiariser avec l'outil. Il vaut également la peine d'expérimenter des [requêtes Cypher personnalisées](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/) en les collant dans la `Raw Query`zone en bas de l'écran.

Gardez à l'esprit qu'au cours de l'engagement, nous devrions documenter chaque fichier transféré vers et depuis les hôtes du domaine et où ils ont été placés sur le disque. C'est une bonne pratique si nous devons déconflicter nos actions avec le client. De plus, selon la portée de l'engagement, vous voulez vous assurer de couvrir vos traces et de nettoyer tout ce que vous mettez dans l'environnement à la fin de l'engagement.

* * * * *

Nous avons une excellente image de la disposition, des forces et des faiblesses du domaine. Nous avons des informations d'identification pour plusieurs utilisateurs et avons énuméré une multitude d'informations telles que les utilisateurs, les groupes, les ordinateurs, les GPO, les ACL, les droits d'administrateur local, les droits d'accès (RDP, WinRM, etc.), les comptes configurés avec des noms principaux de service (SPN), et plus. Nous avons des notes détaillées et une multitude de résultats et avons expérimenté de nombreux outils différents pour pratiquer l'énumération AD avec et sans informations d'identification des hôtes d'attaque Linux et Windows. Que se passe-t-il si nous sommes limités avec le shell dont nous disposons ou si nous n'avons pas la possibilité d'importer des outils ? Notre client peut nous demander d'effectuer tous les travaux à partir d'un hôte géré à l'intérieur de son réseau sans accès Internet et sans moyen de charger nos outils. Nous pourrions atterrir sur un hôte comme`SYSTEM`après une attaque réussie, mais soyez dans une position où il est très difficile ou impossible de charger des outils. On fait quoi alors? Dans la section suivante, nous verrons comment effectuer des actions tout en "vivant de la terre".
