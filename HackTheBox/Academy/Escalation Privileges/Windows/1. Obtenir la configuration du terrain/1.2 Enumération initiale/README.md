Énumération initiale
===================

* * * * *

Lors d'une évaluation, nous pouvons obtenir un shell à faibles privilèges sur un hôte Windows (joint à un domaine ou non) et avoir besoin d'effectuer une escalade de privilèges pour approfondir notre accès. Compromettre complètement l'hôte peut nous permettre d'accéder à des fichiers/partages de fichiers sensibles, nous donner la possibilité de capturer le trafic pour obtenir plus d'informations d'identification, ou d'obtenir des informations d'identification qui peuvent aider à approfondir notre accès ou même passer directement à l'administrateur de domaine dans un environnement Active Directory. Nous pouvons élever les privilèges à l'un des éléments suivants en fonction de la configuration du système et du type de données que nous rencontrons :

| |
| --- |
| Le compte hautement privilégié `NT AUTHORITY\SYSTEM` ou [LocalSystem](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) qui est un compte hautement privilégié avec plus de privilèges qu'un compte d'administrateur local et est utilisé pour exécuter la plupart des services Windows. |
| Le compte `administrateur` local intégré. Certaines organisations désactivent ce compte, mais beaucoup ne le font pas. Il n'est pas rare de voir ce compte réutilisé sur plusieurs systèmes dans un environnement client. |
| Un autre compte local membre du groupe local "Administrateurs". Tous les comptes de ce groupe auront les mêmes privilèges que le compte `administrateur` intégré. |
| Un utilisateur de domaine standard (non privilégié) qui fait partie du groupe local "Administrateurs". |
| Un administrateur de domaine (fortement privilégié dans l'environnement Active Directory) qui fait partie du groupe local "Administrateurs". |

L'énumération est la clé de l'escalade de privilèges. Lorsque nous obtenons un accès shell initial à l'hôte, il est essentiel d'acquérir une connaissance de la situation et de découvrir les détails relatifs à la version du système d'exploitation, au niveau de correctif, aux logiciels installés, aux privilèges actuels, aux appartenances aux groupes, etc. Passons en revue certains des points de données clés que nous devrions examiner après avoir obtenu l'accès initial. Il ne s'agit en aucun cas d'une liste exhaustive, et les divers scripts/outils d'énumération que nous avons abordés dans la section précédente couvrent tous ces points de données et bien d'autres encore. Néanmoins, il est essentiel de comprendre comment effectuer ces tâches manuellement, surtout si nous nous trouvons dans un environnement où nous ne pouvons pas charger d'outils en raison de restrictions de réseau, d'un manque d'accès à Internet ou de protections en place.

Cette [référence des commandes Windows](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands) est très pratique pour effectuer des tâches d'énumération manuelles.

* * * * *

Points de données clés
---------------

`Nom du système d'exploitation` : Connaître le type de système d'exploitation Windows (poste de travail ou serveur) et son niveau (Windows 7 ou 10, Server 2008, 2012, 2016, 2019, etc.) nous donnera une idée des types d'outils qui peuvent être disponibles (comme la version `PowerShell` ou son absence sur les anciens systèmes. Cela permettrait également d'identifier la version du système d'exploitation pour laquelle des exploits publics peuvent être disponibles.

`Version` : comme pour le système d'exploitation [version](https://en.wikipedia.org/wiki/Comparison_of_Microsoft_Windows_versions), il peut y avoir des exploits publics qui ciblent une vulnérabilité dans une version spécifique de Windows. Les exploits du système Windows peuvent provoquer une instabilité du système ou même un plantage complet. Soyez prudent lorsque vous les exécutez sur n'importe quel système de production et assurez-vous de bien comprendre l'exploit et les ramifications possibles avant d'en exécuter un.

`Services en cours d'exécution` : il est important de savoir quels services s'exécutent sur l'hôte, en particulier ceux qui s'exécutent en tant que `NT AUTHORITY\SYSTEM` ou un compte de niveau administrateur. Un service mal configuré ou vulnérable s'exécutant dans le contexte d'un compte privilégié peut être une victoire facile pour l'élévation des privilèges.

Jetons un regard plus approfondi.

* * * * *

Informations système
------------------

L'examen du système lui-même nous donnera une meilleure idée de la version exacte du système d'exploitation, du matériel utilisé, des programmes installés et des mises à jour de sécurité. Cela nous aidera à affiner notre chasse aux correctifs manquants et aux CVE associés que nous pourrions exploiter pour augmenter les privilèges. L'utilisation de la commande [tasklist](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tasklist) pour examiner les processus en cours d'exécution nous donnera une meilleure idée des applications en cours d'exécution sur le système.

#### Liste de tâches

Liste de tâches

```
C:\htb> tasklist /svc

Nom de l'image Services PID
========================= ======== ================= ===========================
Processus d'inactivité du système 0 N/A
Système 4 N/A
smss.exe 316 N/A
csrss.exe 424 N/A
wininit.exe 528 N/A
csrss.exe 540 N/A
winlogon.exe 612 N/A
services.exe 664 N/A
lsass.exe 672 KeyIso, SamSs, VaultSvc
svchost.exe 776 BrokerInfrastructure, DcomLaunch, LSM,
                                    PlugPlay, Alimentation, SystemEventsBroker
svchost.exe 836 RpcEptMapper, RpcSsLogonUI.exe 952 N/A
dwm.exe 964 N/A
svchost.exe 972 TermService
svchost.exe 1008 DHCP, EventLog, lmhosts, TimeBrokerSvc
svchost.exe 364 NcbService, PcaSvc, ScDeviceEnum, TrkWks,
                                    UALSVC, UmRdpService
<...SNIP...>

svchost.exe 1468 Wcmsvc
svchost.exe 1804 PolicyAgent
spoolsv.exe 1884 Spouleur
svchost.exe 1988 W3SVC, WAS
svchost.exe 1996 ftpsvc
svchost.exe 2004 AppHostSvc
FileZilla Server.exe 1140 FileZilla Server
inetinfo.exe 1164 IISADMIN
svchost.exe 1736 DiagTrack
svchost.exe 2084 StateRepository, tiledatamodelsvc
VGAuthService.exe 2100 VGAuthService
vmtoolsd.exe 2112 VMTools
MsMpEng.exe 2136 WinDefend

<...SNIP...>

Interface serveur FileZilla 5628 N/A
jusched.exe 5796 N/A
cmd.exe 4132 N/A
conhost.exe 4136 N/A
TrustedInstaller.exe 1120 TrustedInstaller
TiWorker.exe 1816 N/A
WmiApSrv.exe 2428 wmiApSrv
tasklist.exe 3596 N/A

```

Il est essentiel de se familiariser avec les processus Windows standard tels que [Session Manager Subsystem (smss.exe)](https://en.wikipedia.org/wiki/Session_Manager_Subsystem), [Client Server Runtime Subsystem (csrss.exe)]( https://en.wikipedia.org/wiki/Client/Server_Runtime_Subsystem), [WinLogon (winlogon.exe)](https://en.wikipedia.org/wiki/Winlogon), [Local Security Authority Subsystem Service (LSASS) ](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service) et [Service Host (svchost.exe)](https://en.wikipedia.org/wiki/Svchost.exe), entre autres et les services associés à eux. Être capable de repérer rapidement les processus/services standard aidera à accélérer notre énumération et nous permettra de nous concentrer sur les processus/services non standard, ce qui peut ouvrir une voie d'escalade des privilèges. Dans l'exemple ci-dessus, nous serions plus intéressés par l'exécution du serveur FTP `FileZilla` et tenterions d'énumérer la version pour rechercher des vulnérabilités publiques ou des erreurs de configuration telles que l'accès FTP anonyme, ce qui pourrait entraîner l'exposition de données sensibles ou plus.

D'autres processus tels que `MsMpEng.exe`, Windows Defender, sont intéressants car ils peuvent nous aider à déterminer les protections en place sur l'hôte cible que nous devrons peut-être éviter/contourner.

#### Afficher toutes les variables d'environnement

Les variables d'environnement expliquent beaucoup de choses sur la configuration de l'hôte. Pour en obtenir une impression, Windows propose la commande `set` . L'une des variables les plus négligées est `PATH`. Dans la sortie ci-dessous, rien ne sort de l'ordinaire. Cependant, il n'est pas rare de trouver des administrateurs (ou des applications) modifiant le `PATH`. Un exemple courant consiste à placer Python ou Java dans le chemin, ce qui permettrait l'exécution de Python ou . Fichiers JAR. Si le dossier placé dans le PATH est accessible en écriture par votre utilisateur, il peut être possible d'effectuer des injections de DLL contre d'autres applications. N'oubliez pas que lors de l'exécution d'un programme, Windows recherche d'abord ce programme dans le CWD (Current Working Directory), puis dans le PATH de gauche à droite. Cela signifie que si le chemin personnalisé est placé à gauche (avant C:\Windows\System32), il est beaucoup plus dangereux qu'à droite.

En plus du PATH, `set` peut également donner d'autres informations utiles telles que le HOME DRIVE. Dans les entreprises, il s'agira souvent d'un partage de fichiers. La navigation vers le partage de fichiers lui-même peut révéler d'autres répertoires accessibles. Il n'est pas rare de pouvoir accéder à un "répertoire informatique", qui contient une feuille de calcul d'inventaire comprenant des mots de passe. De plus, des partages sont utilisés pour les répertoires personnels afin que l'utilisateur puisse se connecter à d'autres ordinateurs et avoir la même expérience/fichiers/bureau/etc. ([Profils itinérants](https://docs.microsoft.com/en-us/windows-server/storage/folder-redirection/folder-redirection-rup-overview)). Cela peut également signifier que l'utilisateur emporte avec lui des éléments malveillants. Si un fichier est placé dans `USERPROFILE\AppData\Microsoft\Windows\Start Menu\Programs\Startup`, lorsque l'utilisateur se connecte à une autre machine, ce fichier s'exécute.

Afficher toutes les variables d'environnement

```
C:\htb> set

ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\Administrator\AppData\Roaming
CommonProgramFiles=C:\Program Files\Fichiers communs
CommonProgramFiles(x86)=C:\Program Files (x86)\Fichiers communs
CommonProgramW6432=C:\Program Files\Fichiers communs
NOMORDINATEUR=WINLPE-SRV01
ComSpec=C:\Windows\system32\cmd.exe
ACCUEILDRIVE=C :
HOMEPATH=\Utilisateurs\Administrateur
LOCALAPPDATA=C:\Utilisateurs\Administrateur\AppData\Local
LOGONSERVER=\\WINLPE-SRV01
NUMBER_OF_PROCESSORS=6
SE=Windows_NT
Chemin=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Users\Administrator\AppData\Local\Microsoft\ applications Windows ;
PennsylvanieTHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
PROCESSEUR_ARCHITECTURE=AMD64
PROCESSOR_IDENTIFIER=AMD64 Famille 23 Modèle 49 Étape 0, AuthenticAMD
NIVEAU_PROCESSEUR=23
PROCESSEUR_REVISION=3100
ProgramData=C:\ProgramData
ProgramFiles=C:\Program Files
ProgramFiles(x86)=C:\Program Files (x86)
ProgramW6432=C:\Fichiers de programme
INVITE=$P$G
PSModulePath=C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
PUBLIC=C:\Utilisateurs\Public
SESSIONNAME=Console
LecteurSystème=C :
SystemRoot=C:\Windows
TEMP=C:\Users\ADMINI~1\AppData\Local\Temp\1
TMP=C:\Users\ADMINI~1\AppData\Local\Temp\1
DOMAINEUTILISATEUR=WINLPE-SRV01
USERDOMAIN_ROAMINGPROFILE=WINLPE-SRV01
USERNAME=Administrateur
USERPROFILE=C:\Utilisateurs\Administrateur
windir=C:\Windows

```

#### Afficher les informations de configuration détaillées

La commande `systeminfo` indiquera si la boîte a été corrigée récemment et s'il s'agit d'une machine virtuelle. Si la boîte n'a pas été corrigée récemment, obtenir un accès de niveau administrateur peut être aussi simple que d'exécuter un exploit connu. Recherchez sur Google les bases de connaissances installées sous [HotFixes](https://www.catalog.update.microsoft.com/Search.aspx?q=hotfix) pour avoir une idée du moment où la boîte a été corrigée. Ces informations ne sont pas toujours présentes, car il est possible de masquer les logiciels de correctifs aux non-administrateurs. Le `Temps de démarrage du système` et la `Version du système d'exploitation` peuvent également être vérifiés pour avoir une idée du niveau de correctif. Si la boîte n'a pas été redémarrée depuis plus de six mois, il y a de fortes chances qu'elle ne soit pas non plus corrigée.

De plus, de nombreux guides diront que les informations sur le réseau sont importantes car elles pourraient indiquer une machine à double réseau (connectée à plusieurs réseaux). De manière générale, en ce qui concerne les entreprises, les appareils auront simplement accès à d'autres réseaux via une règle de pare-feu et n'auront pas de câble physique vers eux.

Afficher les informations de configuration détaillées

```
C:\htb> systeminfo

Nom d'hôte : WINLPE-SRV01
Nom du système d'exploitation : Microsoft Windows Server 2016 Standard
Version du système d'exploitation : 10.0.14393 N/A Build 14393
Fabricant du système d'exploitation : Microsoft Corporation
Configuration du système d'exploitation : serveur autonome
Type de construction du système d'exploitation : multiprocesseur gratuit
Propriétaire enregistré : Utilisateur Windows
Organisation enregistrée :
ID produit : 00376-30000-00299-AA303
Date d'installation d'origine : 24/03/2021, 15:46:32
Heure de démarrage du système : 25/03/2021, 09:24:36
Fabricant du système : VMware, Inc.
Modèle de système : VMware7,1
Type de système : PC basé sur x64
Processeur(s) : 3 processeur(s) installé(s).
                            [01] : AMD64 Famille 23 Modèle 49 Stepping 0 AuthenticAMD ~2994 Mhz
                            [02] : AMD64 Famille 23 Modèle 49 Stepping 0 AuthenticAMD ~2994 Mhz
                            [03] : AMD64 Famille 23 Modèle 49 Stepping 0 AuthenticAMD ~2994 Mhz
Version du BIOS : VMware, Inc. VMW71.00V.16707776.B64.2008070230, 8/7/2020
Répertoire Windows : C:\Windows
Répertoire système : C:\Windows\system32
Périphérique de démarrage : \Device\HarddiskVolume2
Paramètres régionaux du système : en-us ; anglais (États-Unis)
Paramètres régionaux d'entrée : en-us ; anglais (États-Unis)
Fuseau horaire : (UTC-08:00) Heure du Pacifique (États-Unis et Canada)
Mémoire physique totale : 6 143 Mo
Mémoire physique disponible : 3 474 Mo
Mémoire virtuelle : taille maximale : 10 371 Mo
Mémoire virtuelle : Disponible : 7 544 Mo
Mémoire virtuelle : en cours d'utilisation : 2 827 Mo
Emplacement(s) du fichier de page : C:\pagefile.sys
Domaine : GROUPE DE TRAVAIL
Serveur de connexion : \\WINLPE-SRV01
Correctif(s) : 3 correctif(s) installé(s).
                            [01] : KB3199986
                            [02] : KB5001078
                            [03] : KB4103723
Carte(s) réseau : 2 cartes réseau installées.
                            [01] : Connexion réseau Gigabit Intel(R) 82574L
                                  Nom de la connexion : Ethernet0
                                  DHCP activé : Oui
                                  Serveur DHCP : 10.129.0.1
                                  Adresse(s) IP
                                  [01] : 10.129.43.8
                                  [02] : fe80 :: e4db: 5ea3: 2775: 8d4d
                                  [03] : mort : bœuf : : e4db : 5ea3 : 2775 : 8d4d
                            [02] : adaptateur Ethernet vmxnet3
                                  Nom de la connexion : Ethernet1
                                  DHCP activé : non
                                  Adresse(s) IP
                                  [01] : 192.168.20.56
                                  [02] : fe80 :: f055: fefd: b1b: 9919
Exigences Hyper-V : Un hyperviseur a été détecté. Les fonctionnalités requises pour Hyper-V ne seront pas affichées.

```

#### Correctifs et mises à jour

Si `systeminfo` n'affiche pas les correctifs, ils peuvent être interrogés avec [WMI](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) à l'aide de la commande WMI binaire avec [QFE (Quick Fix Engineering)](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-quickfixengineering) pour afficher les correctifs.

Correctifs et mises à jour

```
C:\htb> wmic qfe

Légende CSName Description FixComments HotFixID InstallDate InstalledBy InstalledOn Nom ServicePackInEffect État
http://support.microsoft.com/?kbid=3199986 Mise à jour WINLPE-SRV01 KB3199986 NT AUTHORITY\SYSTEM 11/21/2016
https://support.microsoft.com/help/5001078 Mise à jour de sécurité WINLPE-SRV01 KB5001078 NT AUTHORITY\SYSTEM 3/25/2021
http://support.microsoft.com/?kbid=4103723 Mise à jour de sécurité WINLPE-SRV01 KB4103723 NT AUTHORITY\SYSTEM 3/25/2021

```

Nous pouvons également le faire avec PowerShell en utilisant [Get-Hotfix](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-hotfix?view=powershell-7.1) applet de commande.

Correctifs et mises à jour

```
PS C:\htb> Get-HotFix | ft -AutoSize

Source Description HotFixID installé par installé sur
------ ----------- -------- ----------- -----------
WINLPE-SRV01 Mise à jour KB3199986 NT AUTHORITY\SYSTEM 11/21/2016 00:00:00 AM
Mise à jour WINLPE-SRV01 KB4054590 WINLPE-SRV01\Administrateur 30/03/2021 00:00:00 AM
Mise à jour de sécurité WINLPE-SRV01 KB5001078 NT AUTHORITY\SYSTEM 25/03/2021 00:00:00
Mise à jour de sécurité WINLPE-SRV01 KB3200970 WINLPE-SRV01\Administrateur 13/04/2021 00:00:00

```

#### Programmes installés

WMI peut également être utilisé pour afficher les logiciels installés. Ces informations peuvent souvent nous guider vers des exploits difficiles à trouver. Est-ce que `FileZilla`/`Putty`/etc est installé ? Exécutez `LaZagne` pour vérifier si les informations d'identification stockées pour ces applications sont installées. En outre, certains programmes peuvent être installés et exécutés en tant que service vulnérable.

Programmes installés

```
C:\htb> wmic product get name

Nom
Exécution supplémentaire Microsoft Visual C++ 2019 X64 - 14.24.28127
Java 8 mise à jour 231 (64 bits)
Exécution supplémentaire Microsoft Visual C++ 2019 X86 - 14.24.28127
Outils VMware
Microsoft Visual C++ 2019 X64 Durée d'exécution minimale - 14.24.28127
Microsoft Visual C++ 2019 X86 Durée d'exécution minimale - 14.24.28127
Mise à jour automatique Java

<SNIP>

```

Bien sûr, nous pouvons également le faire avec PowerShell en utilisant [Get-WmiObject](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject?view= powershell-5.1) applet de commande.

Programmes installés

```
PS C:\htb> Get-WmiObject -Class Win32_Product |  select Name, Version

Version du nom
---- -------
Moteur de base de données SQL Server 2016 partagé 13.2.5026.0
Pilote Microsoft OLE DB pour SQL Server 18.3.0.0
Redistribuable Microsoft Visual C++ 2010 x64 - 10.0.40219 10.0.40219
Visionneuse d'aide Microsoft 2.3 2.3.28107
Redistribuable Microsoft Visual C++ 2010 x86 - 10.0.40219 10.0.40219
Microsoft Visual C++ 2013 x86 Durée d'exécution minimale - 12.0.21005 12.0.21005
Microsoft Visual C++ 2013 x86 Runtime supplémentaire - 12.0.21005 12.0.21005
Microsoft Visual C++ 2019 X64 Runtime supplémentaire - 14.28.29914 14.28.29914
Pilote Microsoft ODBC 13 pour SQL Server 13.2.5026.0
Moteur de base de données SQL Server 2016 partagé 13.2.5026.0
Services de moteur de base de données SQL Server 2016 13.2.5026.0
SQL Server Management Studio pour Reporting Services 15.0.18369.0
Fichiers de prise en charge de l'installation de Microsoft SQL Server 2008 10.3.5500.0
Tâches de post-installation SSMS 15.0.18369.0
Microsoft VSS Writer pour SQL Server 2016 13.2.5026.0
Java 8 mise à jour 231 (64 bits) 8.0.2310.11
Navigateur pour SQL Server 2016 13.2.5026.0
Services d'intégration 15.0.2000.130

<SNIP>

```

#### Afficher les processus en cours d'exécution

La commande [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) affiche les connexions TCP et UDP actives, ce qui nous donne une meilleure idée des services écoute sur quel(s) port(s) à la fois localement et accessible de l'extérieur. Nous pouvons trouver un service vulnérable accessible uniquement à l'hôte local (lorsqu'il est connecté à l'hôte) que nous pouvons exploiter pour élever les privilèges.

#### Netstat

Netstat

```
PS C:\htb> netstat -ano

Connexions actives

   Proto Adresse locale Adresse étrangère État PID
   TCP 0.0.0.0:21 0.0.0.0:0 ÉCOUTE 1096
   TCP 0.0.0.0:80 0.0.0.0:0 ÉCOUTE 4
   TCP 0.0.0.0:135 0.0.0.0:0 ÉCOUTE 840
   TCP0.0.0.0:445 0.0.0.0:0 ÉCOUTE 4
   TCP 0.0.0.0:1433 0.0.0.0:0 ÉCOUTE 3520
   TCP 0.0.0.0:3389 0.0.0.0:0 ÉCOUTE 968
<...SNIP...>

```

* * * * *

Informations sur l'utilisateur et le groupe
------------------------

Les utilisateurs sont souvent le maillon le plus faible d'une organisation, en particulier lorsque les systèmes sont correctement configurés et corrigés. Il est essentiel de comprendre les utilisateurs et les groupes du système, les membres de groupes spécifiques qui peuvent nous fournir un accès de niveau administrateur, les privilèges dont dispose notre utilisateur actuel, les informations sur la politique de mot de passe et tous les utilisateurs connectés que nous pouvons être en mesure de cibler. Nous pouvons trouver que le système est bien corrigé, mais un membre du répertoire des utilisateurs du groupe d'administrateurs locaux est navigable et contient un fichier de mot de passe tel que `logins.xlsx`, ce qui permet une victoire très facile.

#### Utilisateurs connectés

Il est toujours important de déterminer quels utilisateurs sont connectés à un système. Sont-ils inactifs ou actifs ? Pouvons-nous déterminer sur quoi ils travaillent? Bien que plus difficile à réaliser, nous pouvons parfois attaquer directement les utilisateurs pour augmenter les privilèges ou obtenir un accès supplémentaire. Lors d'un engagement évasif, nous aurions besoin de marcher légèrement sur un hôte avec d'autres utilisateurs travaillant activement dessus pour éviter la détection.

Utilisateurs connectés

```
C:\htb> query user

  NOM D'UTILISATEUR NOM DE SESSION ID ÉTAT TEMPS D'INACTIVITÉ TEMPS DE CONNEXION
>administrateur rdp-tcp#2 1 Actif . 25/03/2021 09:27

```

#### Utilisateur actuel

Lorsque nous accédons à un hôte, nous devons toujours vérifier dans quel contexte utilisateur notre compte s'exécute en premier. Parfois, nous sommes déjà SYSTEM ou équivalent ! Supposons que nous ayons accès en tant que compte de service. Dans ce cas, nous pouvons avoir des privilèges tels que `SeImpersonatePrivilege`, qui peuvent souvent être facilement abusés pour augmenter les privilèges à l'aide d'un outil tel que [Juicy Potato](https://github.com/ohpe/juicy-potato).

Utilisateur actuel

```
C:\htb> echo %USERNAME%

htb-étudiant

```

#### Privilèges de l'utilisateur actuel

Comme mentionné précédemment, connaître les privilèges de notre utilisateur peut grandement aider à augmenter les privilèges. Nous examinerons les privilèges des utilisateurs individuels et les voies d'escalade plus loin dans ce module.

Privilèges de l'utilisateur actuel

```
C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ========= ========
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

#### Informations sur le groupe d'utilisateurs actuel

Notre utilisateur a-t-il hérité de droits via son appartenance à un groupe ? Sont-ils privilégiés dans l'environnement de domaine Active Directory, qui pourrait être exploité pour accéder à davantage de systèmes ?

Informations sur le groupe d'utilisateurs actuel

```
C:\htb> whoami /groups

INFORMATIONS DU GROUPE
-----------------

Type de nom de groupe Attributs SID
===================================== ============ ==== ============ ================================== ================
Tout le monde Groupe connu S-1-1-0 Groupe obligatoire, Activé par défaut, Groupe activé
BUILTIN\Remote Desktop Users Alias S-1-5-32-555 Groupe obligatoire, Activé par défaut, Groupe activé
BUILTIN\Users Alias S-1-5-32-545 Groupe obligatoire, Activé par défaut, Groupe activé
NT AUTHORITY\REMOTE INTERACTIVE LOGON Groupe bien connu S-1-5-14 Groupe obligatoire, Activé par défaut, Groupe activé
NT AUTHORITY\INTERACTIVE Groupe connu S-1-5-4 Groupe obligatoire, Activé par défaut, Groupe activé
NT AUTHORITY\Authenticated Users Groupe bien connu S-1-5-11 Groupe obligatoire, Activé par défaut, Groupe activé
NT AUTHORITY\Cette organisation Groupe bien connu S-1-5-15 Groupe obligatoire, Activé par défaut, Groupe activé
NT AUTHORITY\Compte local Groupe bien connu S-1-5-113 Groupe obligatoire, Activé par défaut, Groupe activé
LOCAL Groupe connu S-1-2-0 Groupe obligatoire, Activé par défaut, Groupe activé
Authentification NT AUTHORITY\NTLM Groupe bien connu S-1-5-64-10 Groupe obligatoire, Activé par défaut, Groupe activé
Étiquette obligatoire\Étiquette de niveau obligatoire moyen S-1-16-8192

```

#### Obtenir tous les utilisateurs

Savoir quels autres utilisateurs sont sur le système est également important. Si nous avons obtenu un accès RDP à un hôte à l'aide des informations d'identification que nous avons capturées pour un utilisateur `bob` et que nous voyons un utilisateur `bob_adm` dans le groupe d'administrateurs locaux, il est utile de vérifier la réutilisation des informations d'identification. Pouvons-nous accéder au répertoire des profils d'utilisateurs pour tous les utilisateurs importants ? Nous pouvons trouver des fichiers précieux tels que des scripts avec des mots de passe ou des clés SSH dans le dossier Bureau, Documents ou Téléchargements d'un utilisateur.

Obtenir tous les utilisateurs

```
C:\htb> net user

Comptes d'utilisateur pour \\WINLPE-SRV01

-------------------------------------------------- -----------------------------
Invité du compte par défaut de l'administrateur
helpdesk htb-student jordan
sarah secsvc
La commande s'est terminée avec succès.

```

#### Obtenir tous les groupes

Savoir quels groupes non standard sont présents sur l'hôte peut nous aider à déterminer à quoi sert l'hôte, à quel point il est accédé, ou peut même conduire à découvrir une mauvaise configuration telle que tous les utilisateurs de domaine dans le bureau à distance ou les groupes d'administrateurs locaux.

Obtenir tous les groupes

```
C:\htb> net localgroup

Alias pour \\WINLPE-SRV01

-------------------------------------------------- -----------------------------
* Opérateurs d'assistance au contrôle d'accès
*Administrateurs
* Opérateurs de sauvegarde
*Accès DCOM au service de certificat
* Opérateurs cryptographiques
*Utilisateurs COM distribués
* Lecteurs de journaux d'événements
*Invités
* Administrateurs Hyper-V
*IIS_IUSRS
* Opérateurs de configuration réseau
* Utilisateurs du journal des performances
* Utilisateurs du moniteur de performances
* Utilisateurs expérimentés
* Opérateurs d'impression
* Serveurs de point de terminaison RDS
*Serveurs de gestion RDS
*Serveurs d'accès à distance RDS
* Utilisateurs de bureau à distance
*Utilisateurs de gestion à distance
*Réplicateur
* Administrateurs de répliques de stockage
* Groupe de comptes gérés par le système
*Utilisateurs
La commande s'est terminée avec succès.

```

#### Détails sur un groupe

Il vaut la peine de vérifier les détails pour tous les groupes non standard. Bien que peu probable, nous pouvons trouver un mot de passe ou d'autres informations intéressantes stockées dans la description du groupe. Au cours de notre énumération, nous pouvons découvrir les informations d'identification d'un autre utilisateur non administrateur qui est membre d'un groupe local qui peut être exploité pour élever les privilèges.

Détails sur un groupe

```
C:\htb> net localgroup administrators


Administrateurs de noms d'alias
Commentaire Les administrateurs ont un accès complet et illimité à l'ordinateur/au domaine

Membres

-------------------------------------------------- -----------------------------
Administrateur
bureau d'aide
sarah
secsvc
La commande s'est terminée avec succès.

```

#### Obtenir la politique de mot de passe et d'autres informations de compte

Obtenir la politique de mot de passe et d'autres informations sur le compte

```
C:\htb> net accounts

Forcer la déconnexion de l'utilisateur combien de temps après l'expiration du délai ? : Jamais
Âge minimum du mot de passe (jours) : 0
Âge maximal du mot de passe (jours) : 42
Longueur minimale du mot de passe : 0
Longueur de l'historique des mots de passe conservé : aucun
Seuil de verrouillage : Jamais
Durée de verrouillage (minutes) : 30
Fenêtre d'observation du verrouillage (minutes) : 30
Rôle informatique : SERVEUR
La commande s'est terminée avec succès.

```

* * * * *

Passer à autre chose
---------

Comme indiqué précédemment, il ne s'agit pas d'une liste exhaustive de commandes d'énumération. Les outils dont nous avons discuté dans la section précédente contribueront grandement à accélérer le processus de recensement et à s'assurer qu'il est complet et qu'aucun détail n'est laissé de côté. De nombreuses feuilles de triche sont disponibles pour nous aider, telles que [celle-ci](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md). Étudiez les outils et leur sortie et commencez à créer votre propre feuille de triche de commande, afin qu'elle soit facilement disponible au cas où vous rencontreriez un environnement qui nécessite la plupart ou la totalité de l'énumération manuelle.