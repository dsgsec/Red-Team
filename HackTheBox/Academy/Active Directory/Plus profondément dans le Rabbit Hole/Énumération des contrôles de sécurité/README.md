Énumération des contrôles de sécurité
=====================================

* * * * *

Après avoir pris pied, nous pourrions utiliser cet accès pour avoir une idée de l'état défensif des hôtes, énumérer davantage le domaine maintenant que notre visibilité n'est plus aussi restreinte et, si nécessaire, travailler à "vivre de la terre" en utilisant outils qui existent nativement sur les hôtes. Il est important de comprendre les contrôles de sécurité en place dans une organisation car les produits utilisés peuvent affecter les outils que nous utilisons pour notre énumération AD, ainsi que l'exploitation et la post-exploitation. Comprendre les protections auxquelles nous pouvons être confrontés nous aidera à éclairer nos décisions concernant l'utilisation des outils et nous aidera à planifier notre plan d'action en évitant ou en modifiant certains outils. Certaines organisations ont des protections plus strictes que d'autres, et certaines n'appliquent pas les contrôles de sécurité de la même manière.

Remarque : Cette section est destinée à présenter les contrôles de sécurité possibles en place dans un domaine, mais n'a pas de composant interactif. L'énumération et le contournement des contrôles de sécurité sortent du cadre de ce module, mais nous voulions donner un aperçu des technologies possibles que nous pourrions rencontrer lors d'une évaluation.

* * * * *

Windows Defender
----------------

Windows Defender (ou [Microsoft Defender](https://en.wikipedia.org/wiki/Microsoft_Defender) après la mise à jour Windows du 10 mai 2020) s'est considérablement amélioré au fil des ans et, par défaut, bloquera des outils tels que `PowerView`. Il existe des moyens de contourner ces protections. Ces moyens seront traités dans d'autres modules. Nous pouvons utiliser l'applet de commande PowerShell intégrée [Get-MpComputerStatus](https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=win10-ps) pour obtenir l'état actuel de Defender. Ici, nous pouvons voir que le `RealTimeProtectionEnabled`paramètre est défini sur `True`, ce qui signifie que Defender est activé sur le système.

#### Vérification du statut de Defender avec Get-MpComputerStatus

  Vérification du statut de Defender avec Get-MpComputerStatus

```
PS C:\htb> Get-MpComputerStatus

AMEngineVersion                 : 1.1.17400.5
AMProductVersion                : 4.10.14393.0
AMServiceEnabled                : True
AMServiceVersion                : 4.10.14393.0
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 9/2/2020 11:31:50 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
AntivirusSignatureAge           : 1
AntivirusSignatureLastUpdated   : 9/2/2020 11:31:51 AM
AntivirusSignatureVersion       : 1.323.392.0
BehaviorMonitorEnabled          : False
ComputerID                      : 07D23A51-F83F-4651-B9ED-110FF2B83A9C
ComputerState                   : 0
FullScanAge                     : 4294967295
FullScanEndTime                 :
FullScanStartTime               :
IoavProtectionEnabled           : False
LastFullScanSource              : 0
LastQuickScanSource             : 2
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
NISSignatureAge                 : 4294967295
NISSignatureLastUpdated         :
NISSignatureVersion             : 0.0.0.0
OnAccessProtectionEnabled       : False
QuickScanAge                    : 0
QuickScanEndTime                : 9/3/2020 12:50:45 AM
QuickScanStartTime              : 9/3/2020 12:49:49 AM
RealTimeProtectionEnabled       : True
RealTimeScanDirection           : 0
PSComputerName                  :

```

* * * * *

AppLocker
---------

Une liste blanche d'applications est une liste d'applications logicielles ou d'exécutables approuvés qui sont autorisés à être présents et exécutés sur un système. L'objectif est de protéger l'environnement contre les logiciels malveillants nuisibles et les logiciels non approuvés qui ne correspondent pas aux besoins commerciaux spécifiques d'une organisation. [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker) est la solution de liste blanche d'applications de Microsoft et permet aux administrateurs système de contrôler les applications et les fichiers que les utilisateurs peuvent exécuter. Il fournit un contrôle granulaire sur les exécutables, les scripts, les fichiers d'installation Windows, les DLL, les applications packagées et les programmes d'installation d'applications packagées. Il est courant pour les organisations de bloquer cmd.exe et PowerShell.exe et d'accéder en écriture à certains répertoires, mais tout cela peut être contourné. Les organisations se concentrent également souvent sur le blocage de l' `PowerShell.exe`exécutable, mais oublient les autres[Emplacements exécutables PowerShell](https://www.powershelladmin.com/wiki/PowerShell_Executables_File_System_Locations) tels que `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe`ou `PowerShell_ISE.exe`. Nous pouvons voir que c'est le cas dans les `AppLocker`règles présentées ci-dessous. Tous les utilisateurs de domaine ne sont pas autorisés à exécuter l'exécutable PowerShell 64 bits situé à :

`%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe`

Ainsi, nous pouvons simplement l'appeler à partir d'autres endroits. Parfois, nous rencontrons des `AppLocker`politiques plus strictes qui nécessitent plus de créativité pour les contourner. Ces moyens seront traités dans d'autres modules.

#### Utilisation de l'applet de commande Get-AppLockerPolicy

  Utilisation de l'applet de commande Get-AppLockerPolicy

```
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

PathConditions      : {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 3d57af4a-6cf8-4e5b-acfc-c2c2956061fa
Name                : Block PowerShell
Description         : Blocks Domain Users from using PowerShell on workstations
UserOrGroupSid      : S-1-5-21-2974783224-3764228556-2640795941-513
Action              : Deny

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Name                : (Default Rule) All files located in the Windows folder
Description         : Allows members of the Everyone group to run applications that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : fd686d83-a829-4351-8ff4-27c7de5755d2
Name                : (Default Rule) All files
Description         : Allows members of the local Administrators group to run all applications.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow

```

* * * * *

Mode langage contraint PowerShell
---------------------------------

[Le mode de langage contraint](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/) de PowerShell verrouille de nombreuses fonctionnalités nécessaires pour utiliser PowerShell efficacement, telles que le blocage des objets COM, l'autorisation uniquement des types .NET approuvés, les flux de travail basés sur XAML, les classes PowerShell, etc. Nous pouvons rapidement énumérer si nous sommes en mode langage complet ou en mode langage contraint.

#### Mode de langue d'énumération

  Mode de langue d'énumération

```
PS C:\htb> $ExecutionContext.SessionState.LanguageMode

ConstrainedLanguage

```

* * * * *

TOURS
-----

La [solution de mot de passe de l'administrateur local Microsoft (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) est utilisée pour randomiser et faire pivoter les mots de passe de l'administrateur local sur les hôtes Windows et empêcher le mouvement latéral. Nous pouvons énumérer les utilisateurs du domaine qui peuvent lire le mot de passe LAPS défini pour les machines sur lesquelles LAPS est installé et les machines sur lesquelles LAPS n'est pas installé. Le [LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) facilite grandement cela avec plusieurs fonctions. L'un est l'analyse `ExtendedRights`de tous les ordinateurs avec LAPS activé. Cela affichera les groupes spécifiquement délégués pour lire les mots de passe LAPS, qui sont souvent des utilisateurs de groupes protégés. Un compte qui a joint un ordinateur à un domaine reçoit`All Extended Rights`sur cet hôte, et ce droit donne au compte la possibilité de lire les mots de passe. L'énumération peut afficher un compte d'utilisateur qui peut lire le mot de passe LAPS sur un hôte. Cela peut nous aider à cibler des utilisateurs AD spécifiques qui peuvent lire les mots de passe LAPS.

#### Utilisation de Find-LAPSDelegatedGroups

  Utilisation de Find-LAPSDelegatedGroups

```
PS C:\htb> Find-LAPSDelegatedGroups

OrgUnit                                             Delegated Groups
-------                                             ----------------
OU=Servers,DC=INLANEFREIGHT,DC=LOCAL                INLANEFREIGHT\Domain Admins
OU=Servers,DC=INLANEFREIGHT,DC=LOCAL                INLANEFREIGHT\LAPS Admins
OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL           INLANEFREIGHT\Domain Admins
OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL           INLANEFREIGHT\LAPS Admins
OU=Web Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\Domain Admins
OU=Web Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\LAPS Admins
OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\Domain Admins
OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\LAPS Admins
OU=File Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\Domain Admins
OU=File Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\LAPS Admins
OU=Contractor Laptops,OU=Workstations,DC=INLANEF... INLANEFREIGHT\Domain Admins
OU=Contractor Laptops,OU=Workstations,DC=INLANEF... INLANEFREIGHT\LAPS Admins
OU=Staff Workstations,OU=Workstations,DC=INLANEF... INLANEFREIGHT\Domain Admins
OU=Staff Workstations,OU=Workstations,DC=INLANEF... INLANEFREIGHT\LAPS Admins
OU=Executive Workstations,OU=Workstations,DC=INL... INLANEFREIGHT\Domain Admins
OU=Executive Workstations,OU=Workstations,DC=INL... INLANEFREIGHT\LAPS Admins
OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\Domain Admins
OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\LAPS Admins

```

Le `Find-AdmPwdExtendedRights`vérifie les droits sur chaque ordinateur avec LAPS activé pour tous les groupes avec accès en lecture et les utilisateurs avec "Tous les droits étendus". Les utilisateurs avec "Tous les droits étendus" peuvent lire les mots de passe LAPS et peuvent être moins protégés que les utilisateurs des groupes délégués, cela vaut donc la peine d'être vérifié.

#### Utilisation de Find-AdmPwdExtendedRights

  Utilisation de Find-AdmPwdExtendedRights

```
PS C:\htb> Find-AdmPwdExtendedRights

ComputerName                Identity                    Reason
------------                --------                    ------
EXCHG01.INLANEFREIGHT.LOCAL INLANEFREIGHT\Domain Admins Delegated
EXCHG01.INLANEFREIGHT.LOCAL INLANEFREIGHT\LAPS Admins   Delegated
SQL01.INLANEFREIGHT.LOCAL   INLANEFREIGHT\Domain Admins Delegated
SQL01.INLANEFREIGHT.LOCAL   INLANEFREIGHT\LAPS Admins   Delegated
WS01.INLANEFREIGHT.LOCAL    INLANEFREIGHT\Domain Admins Delegated
WS01.INLANEFREIGHT.LOCAL    INLANEFREIGHT\LAPS Admins   Delegated

```

Nous pouvons utiliser la `Get-LAPSComputers`fonction pour rechercher des ordinateurs sur lesquels LAPS est activé lorsque les mots de passe expirent, et même les mots de passe aléatoires en texte clair si notre utilisateur y a accès.

#### Utilisation de Get-LAPSComputers

  Utilisation de Get-LAPSComputers

```
PS C:\htb> Get-LAPSComputers

ComputerName                Password       Expiration
------------                --------       ----------
DC01.INLANEFREIGHT.LOCAL    6DZ[+A/[]19d$F 08/26/2020 23:29:45
EXCHG01.INLANEFREIGHT.LOCAL oj+2A+[hHMMtj, 09/26/2020 00:51:30
SQL01.INLANEFREIGHT.LOCAL   9G#f;p41dcAe,s 09/26/2020 00:30:09
WS01.INLANEFREIGHT.LOCAL    TCaG-F)3No;l8C 09/26/2020 00:46:04

```

* * * * *

Conclusion
----------

Comme nous l'avons vu dans cette section, plusieurs autres techniques d'énumération AD utiles sont à notre disposition pour déterminer quelles protections sont en place. Cela vaut la peine de vous familiariser avec tous ces outils et techniques et de les ajouter à votre arsenal d'options. Continuons maintenant notre énumération du domaine INLANEFREIGHT.LOCAL d'un point de vue accrédité.
