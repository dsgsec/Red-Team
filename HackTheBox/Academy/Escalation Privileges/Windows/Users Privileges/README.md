Présentation des privilèges Windows
===========================

* * * * *

[Les privilèges](https://docs.microsoft.com/en-us/windows/win32/secauthz/privileges) dans Windows sont des droits qu'un compte peut se voir accorder pour effectuer diverses opérations sur le système local, telles que la gestion des services , chargement des pilotes, arrêt du système, débogage d'une application, etc. Les privilèges sont différents des droits d'accès, qu'un système utilise pour accorder ou refuser l'accès à des objets sécurisables. Les privilèges d'utilisateur et de groupe sont stockés dans une base de données et accordés via un jeton d'accès lorsqu'un utilisateur se connecte à un système. Un compte peut avoir des privilèges locaux sur un ordinateur spécifique et des privilèges différents sur différents systèmes si le compte appartient à un domaine Active Directory. Chaque fois qu'un utilisateur tente d'effectuer une action privilégiée, le système examine le jeton d'accès de l'utilisateur pour voir si le compte dispose des privilèges requis et, le cas échéant, vérifie s'ils sont activés. La plupart des privilèges sont désactivés par défaut. Certains peuvent être activés en ouvrant une console administrative cmd.exe ou PowerShell, tandis que d'autres peuvent être activés manuellement.

L'objectif d'une évaluation est souvent d'obtenir un accès administratif à un système ou à plusieurs systèmes. Supposons que nous puissions nous connecter à un système en tant qu'utilisateur avec un ensemble spécifique de privilèges. Dans ce cas, nous pourrons peut-être tirer parti de cette fonctionnalité intégrée pour augmenter directement les privilèges ou utiliser les privilèges attribués au compte cible pour approfondir notre accès dans la poursuite de notre objectif ultime.

* * * * *

Processus d'autorisation Windows
-----------------------------

Les principaux de sécurité sont tout ce qui peut être authentifié par le système d'exploitation Windows, y compris les comptes d'utilisateur et d'ordinateur, les processus qui s'exécutent dans le contexte de sécurité ou un autre compte d'utilisateur/d'ordinateur, ou les groupes de sécurité auxquels ces comptes appartiennent. Les principaux de sécurité sont le principal moyen de contrôler l'accès aux ressources sur les hôtes Windows. Chaque principal de sécurité est identifié par un [identifiant de sécurité (SID)] unique (https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/security-identifiers-in-windows). Lorsqu'un principal de sécurité est créé, il se voit attribuer un SID qui reste attribué à ce principal pendant toute sa durée de vie.

Le schéma ci-dessous décrit le processus d'autorisation et de contrôle d'accès Windows à un niveau élevé, montrant, par exemple, le processus démarré lorsqu'un utilisateur tente d'accéder à un objet sécurisable tel qu'un dossier sur un partage de fichiers. Au cours de ce processus, le jeton d'accès de l'utilisateur (y compris son SID d'utilisateur, les SID de tous les groupes dont il est membre, la liste des privilèges et d'autres informations d'accès) est comparé aux [entrées de contrôle d'accès (ACE)](https://docs.microsoft .com/en-us/windows/win32/secauthz/access-control-entries) dans le [descripteur de sécurité] de l'objet (https://docs.microsoft.com/en-us/windows/win32/secauthz/security-descriptors ) (contenant des informations de sécurité sur un objet sécurisable, telles que les droits d'accès (voir ci-dessous) accordés aux utilisateurs ou aux groupes). Une fois cette comparaison terminée, une décision est prise d'accorder ou de refuser l'accès. L'ensemble de ce processus se produit presque instantanément chaque fois qu'un utilisateur tente d'accéder à une ressource sur un hôte Windows. Dans le cadre de nos activités d'énumération et d'escalade de privilèges, nous tentons d'utiliser et d'abuser des droits d'accès et de tirer parti ou de nous insérer dans ce processus d'autorisation pour favoriser notre accès vers notre objectif.

![image](https://academy.hackthebox.com/storage/modules/67/auth_process.png)

[Source de l'image](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-principals)

* * * * *

Droits et privilèges sous Windows
--------------------------------

Windows contient de nombreux groupes qui accordent à leurs membres des droits et privilèges puissants. Beaucoup d'entre eux peuvent être utilisés de manière abusive pour élever les privilèges à la fois sur un hôte Windows autonome et dans un environnement de domaine Active Directory. En fin de compte, ceux-ci peuvent être utilisés pour obtenir des privilèges d'administrateur de domaine, d'administrateur local ou de système sur un poste de travail, un serveur ou un contrôleur de domaine Windows (DC). Certains de ces groupes sont énumérés ci-dessous.

| Groupe | Descriptif |
| --- | --- |
| Administrateurs par défaut | Les administrateurs de domaine et les administrateurs d'entreprise sont des "super" groupes. |
| Opérateurs de serveur | Les membres peuvent modifier les services, accéder aux partages SMB et sauvegarder les fichiers. |
| Opérateurs de sauvegarde | Les membres sont autorisés à se connecter localement aux DC et doivent être considérés comme des administrateurs de domaine. Ils peuvent faire des clichés instantanés de la base de données SAM/NTDS, lire le registre à distance et accéder au système de fichiers sur le DC via SMB. Ce groupe est parfois ajouté au groupe Opérateurs de sauvegarde local sur des non-DC. |
| Opérateurs d'impression | Les membres peuvent se connecter localement aux DC et "inciter" Windows à charger un pilote malveillant. |
| Administrateurs Hyper-V | S'il existe des contrôleurs de domaine virtuels, tous les administrateurs de virtualisation, tels que les membres des administrateurs Hyper-V, doivent être considérés comme des administrateurs de domaine. |
| Opérateurs de compte | Les membres peuvent modifier des comptes et des groupes non protégés dans le domaine. |
| Bureau à distancehaut Utilisateurs | Les membres ne reçoivent aucune autorisation utile par défaut, mais se voient souvent accorder des droits supplémentaires tels que `Autoriser la connexion via les services de bureau à distance` et peuvent se déplacer latéralement à l'aide du protocole RDP. |
| Utilisateurs de gestion à distance | Les membres peuvent se connecter aux DC avec PSRemoting (ce groupe est parfois ajouté au groupe local de gestion à distance sur les non-DC). |
| Propriétaires créateurs de stratégie de groupe | Les membres peuvent créer de nouveaux objets de stratégie de groupe, mais doivent recevoir des autorisations supplémentaires pour lier les objets de stratégie de groupe à un conteneur tel qu'un domaine ou une unité d'organisation. |
| Administrateurs de schéma | Les membres peuvent modifier la structure du schéma Active Directory et déjouer tout groupe/GPO à créer en ajoutant un compte compromis à l'ACL d'objet par défaut. |
| Administrateurs DNS | Les membres peuvent charger une DLL sur un DC, mais n'ont pas les autorisations nécessaires pour redémarrer le serveur DNS. Ils peuvent charger une DLL malveillante et attendre un redémarrage en tant que mécanisme de persistance. Le chargement d'une DLL entraîne souvent le plantage du service. Un moyen plus fiable d'exploiter ce groupe consiste à [créer un enregistrement WPAD](https://cube0x0.github.io/Pocing-Beyond-DA/). |

* * * * *

Attribution des droits d'utilisateur
----------------------

En fonction de l'appartenance au groupe et d'autres facteurs tels que les privilèges attribués via le domaine et la stratégie de groupe locale, les utilisateurs peuvent disposer de divers droits attribués à leur compte. Cet article Microsoft sur [l'attribution des droits d'utilisateur](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment) fournit une explication détaillée de chaque des droits d'utilisateur pouvant être définis dans Windows ainsi que des considérations de sécurité applicables à chaque droit. Vous trouverez ci-dessous certaines des principales attributions de droits d'utilisateur, qui sont des paramètres appliqués à l'hôte local. Ces droits permettent aux utilisateurs d'effectuer des tâches sur le système telles que se connecter localement ou à distance, accéder à l'hôte depuis le réseau, arrêter le serveur, etc.

| Paramètre [Constante](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) | Nom du paramètre | Affectation standard | Descriptif |
| --- | --- | --- | --- |
| SeNetworkLogonRight | [Accéder à cet ordinateur depuis le réseau](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/access-this-computer-from-the-network) | Administrateurs, Utilisateurs Authentifiés | Détermine quels utilisateurs peuvent se connecter à l'appareil à partir du réseau. Ceci est requis par les protocoles réseau tels que SMB, NetBIOS, CIFS et COM+. |
| SeRemoteInteractiveLogonRight | [Autoriser la connexion via les services Bureau à distance] (https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/allow-log-on-through-remote-desktop- services) | Administrateurs, utilisateurs de bureau à distance | Ce paramètre de stratégie détermine quels utilisateurs ou groupes peuvent accéder à l'écran de connexion d'un appareil distant via une connexion aux services Bureau à distance. Un utilisateur peut établir une connexion aux services Bureau à distance à un serveur particulier mais ne peut pas se connecter à la console de ce même serveur. |
| SeBackupPrivilège | [Sauvegarder des fichiers et des répertoires](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/back-up-files-and-directories) | Administrateurs | Ce droit d'utilisateur détermine quels utilisateurs peuvent contourner les autorisations de fichiers et de répertoires, de registre et d'autres objets persistants à des fins de sauvegarde du système. |
| SeSecurityPrivilege | [Gérer le journal d'audit et de sécurité](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/manage-auditing-and-security-log) | Administrateurs | Ce paramètre de stratégie détermine quels utilisateurs peuvent spécifier des options d'audit d'accès aux objets pour des ressources individuelles telles que des fichiers, des objets Active Directory et des clés de registre. Ces objets spécifient leurs listes de contrôle d'accès système (SACL). Un utilisateur auquel ce droit d'utilisateur a été attribué peut également afficher et effacer le journal de sécurité dans l'Observateur d'événements. |
| SeTakeOwnershipPrivilege | [S'approprier des fichiers ou d'autres objets](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other- objets) | Administrateurs | Ce paramètre de stratégie détermine quels utilisateurs peuvent s'approprier tout objet sécurisable de l'appareil, y compris les objets Active Directory, les fichiers et dossiers NTFS, les imprimantes, les clés de registre, les services, les processus et les threads. |
| SeDebugPrivilege | [Programmes de débogage](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs) | Administrateurs | Ce paramètre de stratégie détermine quels utilisateurs peuvent se connecter ou ouvrir n'importe quel processus, même un processus dont ils ne sont pas propriétaires. Les développeurs qui déboguent leurs applications n'ont pas besoin de ce droit d'utilisateur. Les développeurs qui déboguent de nouveaux composants système ont besoin de ce droit d'utilisateur. Ce droit d'utilisateur permet d'accéder aux composants sensibles et critiques du système d'exploitation. |
| SeImpersonatePrivilege | [Usurper l'identité d'un client après l'authentification](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/emprunter l'identité d'un client après authentification) | Administrateurs, Service local, Service réseau, Service | Ce paramètre de stratégie détermine les programmes autorisés à se faire passer pour un utilisateur ou un autre compte spécifié et à agir au nom de l'utilisateur. |
| SeLoadDriverPrivilege | [Charger et décharger les pilotes de périphérique](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/load-and-unload-device-drivers) | Administrateurs | Ce paramètre de stratégie détermine quels utilisateurs peuvent charger et décharger dynamiquement des pilotes de périphérique. Ce droit d'utilisateur n'est pas requis si un pilote signé pour le nouveau matériel existe déjà dans le fichier driver.cab sur le périphérique. Les pilotes de périphérique s'exécutent en tant que code hautement privilégié. |
| SeRestorePrivilege | [Restaurer des fichiers et des répertoires](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/restore-files-and-directories) | Administrateurs | Ce paramètre de sécurité détermine quels utilisateurs peuvent contourner les autorisations de fichier, de répertoire, de registre et d'autres objets persistants lorsqu'ils restaurent des fichiers et des répertoires sauvegardés. Il détermine quels utilisateurs peuvent définir des principaux de sécurité valides en tant que propriétaire d'un objet. |

De plus amples informations sont disponibles [ici](https://4sysops.com/archives/user-rights-assignment-in-windows-server-2016/).

En tapant la commande `whoami /priv` vous obtiendrez une liste de tous les droits d'utilisateur attribués à votre utilisateur actuel. Certains droits ne sont disponibles que pour les utilisateurs administratifs et ne peuvent être répertoriés/exploités que lors de l'exécution d'une session cmd ou PowerShell avec élévation de privilèges. Ces concepts de droits élevés et [Contrôle de compte d'utilisateur (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control -works) sont des fonctionnalités de sécurité introduites avec Windows Vista pour empêcher par défaut les applications de s'exécuter avec des autorisations complètes, sauf si nécessaire. Si nous comparons et contrastons les droits dont nous disposons en tant qu'administrateur dans une console non élevée par rapport à une console élevée, nous verrons qu'ils diffèrent considérablement.

Vous trouverez ci-dessous les droits disponibles pour un compte d'administrateur local sur un système Windows.

#### Droits de l'administrateur local – Élevés

Si nous exécutons une fenêtre de commande élevée, nous pouvons voir la liste complète des droits dont nous disposons :

Droits d'utilisateur de l'administrateur local - Elevés

```
PS C:\htb> whoami

winlpe-srv01\administrateur

PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
======================================== ========= ================================================= ======= ========
SeIncreaseQuotaPrivilege Ajuster les quotas de mémoire pour un processus Désactivé
SeSecurityPrivilege Gérer le journal d'audit et de sécurité Désactivé
SeTakeOwnershipPrivilege S'approprier des fichiers ou d'autres objets Désactivé
SeLoadDriverPrivilege Charger et décharger les pilotes de périphérique Désactivé
Performances système du profil SeSystemProfilePrivilege Désactivé
SeSystemtimePrivilege Modifier l'heure du système Désactivé
Processus unique du profil SeProfileSingleProcessPrivilege Désactivé
SeIncreaseBasePriorityPrivilege Augmenter la priorité de planification Désactivé
SeCreatePagefilePrivilege Créer un fichier d'échange Désactivé
SeBackupPrivilege Sauvegarder des fichiers et des répertoires Désactivé
SeRestorePrivilege Restaurer les fichiers et les répertoires Désactivé
SeShutdownPrivilege Arrêter le système Désactivé
Programmes de débogage SeDebugPrivilege désactivés
SeSystemEnvironmentPrivilege Modifier les valeurs de l'environnement du micrologiciel Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeRemoteShutdownPrivilege Forcer l'arrêt à partir d'un système distant Désactivé
SeUndockPrivilege Supprimer l'ordinateur de la station d'accueil Désactivé
SeManageVolumePrivilege Effectuer des tâches de maintenance de volume Désactivé
SeImpersonatePrivilege Emprunter l'identité d'un client après l'authentification Activé
SeCreateGlobalPrivilege Créer des objets globauxActivé
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé
SeTimeZonePrivilege Changer le fuseau horaire Désactivé
SeCreateSymbolicLinkPrivilege Créer des liens symboliques Désactivé
SeDelegateSessionUserImpersonatePrivilege Obtenir un jeton d'emprunt d'identité pour un autre utilisateur dans la même session Désactivé

```

Lorsqu'un privilège est répertorié pour notre compte dans l'état "Désactivé", cela signifie que notre compte dispose du privilège spécifique attribué. Néanmoins, il ne peut pas être utilisé dans un jeton d'accès pour effectuer les actions associées tant qu'il n'est pas activé. Windows ne fournit pas de commande intégrée ni d'applet de commande PowerShell pour activer les privilèges. Nous avons donc besoin de scripts pour nous aider. Nous verrons comment abuser de divers privilèges tout au long de ce module et diverses manières d'activer des privilèges spécifiques dans notre processus actuel. Un exemple est ce PowerShell [script](https://www.powershellgallery.com/packages/PoshPrivilege/0.3.0.0/Content/Scripts%5CEnable-Privilege.ps1) qui peut être utilisé pour activer certains privilèges, ou ce [script ](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) qui peut être utilisé pour ajuster les privilèges des jetons.

Un utilisateur standard, en revanche, a considérablement moins de droits.

#### Droits d'utilisateur standard

Droits d'utilisateur standards

```
PS C:\htb> whoami

winlpe-srv01\htb-étudiant

PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ========= ========
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

Les droits des utilisateurs augmentent en fonction des groupes dans lesquels ils sont placés ou des privilèges qui leur sont attribués. Vous trouverez ci-dessous un exemple des droits accordés aux utilisateurs du groupe "Opérateurs de sauvegarde". Les utilisateurs de ce groupe ont d'autres droits que l'UAC restreint actuellement. Pourtant, nous pouvons voir à partir de cette commande qu'ils ont le [SeShutdownPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/shut-down-the- system), ce qui signifie qu'ils peuvent arrêter un contrôleur de domaine qui pourrait provoquer une interruption de service massive s'ils se connectent localement à un contrôleur de domaine (pas via RDP ou WinRM).

#### Droits des opérateurs de sauvegarde

Droits des opérateurs de sauvegarde

```
PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ========= ========
SeShutdownPrivilege Arrêter le système Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

* * * * *

Détection
---------

Ce [post](https://blog.palantir.com/windows-privilege-abuse-auditing-detection-and-defense-3078a403d74e) vaut la peine d'être lu pour plus d'informations sur les privilèges Windows, ainsi que sur la détection et la prévention des abus, en particulier en enregistrant l'événement [4672 : Privilèges spéciaux attribués à la nouvelle connexion](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672) qui générera un événement si certains éléments sensibles les privilèges sont attribués à une nouvelle session de connexion. Cela peut être ajusté de plusieurs manières, par exemple en surveillant les privilèges qui ne doivent *jamais* être attribués ou ceux qui ne doivent jamais être attribués qu'à des comptes spécifiques.

* * * * *

Passer à autre chose
---------

En tant qu'attaquants et défenseurs, nous devons revoir la composition de ces groupes. Il n'est pas rare de trouver des utilisateurs apparemment peu privilégiés ajoutés à un ou plusieurs de ces groupes, qui peuvent être utilisés pour compromettre un hôte unique ou un accès supplémentaire au sein d'un environnement Active Directory. Nous discuterons des implications de certains des droits les plus courants et effectuerons des exercices sur la façon d'augmenter les privilèges si nous obtenons l'accès à un utilisateur avec certains de ces droits attribués à son compte.