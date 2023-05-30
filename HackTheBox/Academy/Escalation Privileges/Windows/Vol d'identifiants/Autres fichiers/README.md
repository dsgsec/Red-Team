Autres fichiers
===========

* * * * *

Il existe de nombreux autres types de fichiers que nous pouvons trouver sur un système local ou sur des lecteurs de partage réseau pouvant contenir des informations d'identification ou des informations supplémentaires pouvant être utilisées pour augmenter les privilèges. Dans un environnement Active Directory, nous pouvons utiliser un outil tel que [Snaffler](https://github.com/SnaffCon/Snaffler) pour explorer les lecteurs de partage réseau à la recherche d'extensions de fichiers intéressantes telles que `.kdbx`, `.vmdk`, `.vdhx`, `.ppk`, etc. Nous pouvons trouver un disque dur virtuel sur lequel nous pouvons monter et extraire des hachages de mots de passe d'administrateur local, une clé privée SSH pouvant être utilisée pour accéder à d'autres systèmes ou des instances d'utilisateurs stockant des mots de passe. dans des documents Excel/Word, des classeurs OneNote ou même le fichier `passwords.txt` classique. J'ai effectué de nombreux tests d'intrusion où un mot de passe trouvé sur un lecteur partagé ou un lecteur local a conduit soit à un accès initial, soit à une élévation des privilèges. De nombreuses entreprises fournissent à chaque employé un dossier sur un partage de fichiers mappé à leur identifiant d'utilisateur, c'est-à-dire le dossier `bjones` sur le partage `users` sur un serveur appelé `FILE01` avec des autorisations lâches appliquées (c'est-à-dire, tous les utilisateurs du domaine avec accès en lecture accès à tous les dossiers utilisateur). Nous trouvons souvent des utilisateurs qui enregistrent des données personnelles sensibles dans ces dossiers, ignorant qu'ils sont accessibles à tout le monde sur le réseau et pas seulement localement à leur poste de travail.

* * * * *

Recherche manuelle d'informations d'identification dans le système de fichiers
--------------------------------------------------

Nous pouvons rechercher le système de fichiers ou partager des lecteurs manuellement à l'aide des commandes suivantes de [cette feuille de triche](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-% 20Privilege%20Escalation.md#search-for-a-file-with-a-certain-filename)

#### Rechercher le contenu du fichier pour la chaîne - Exemple 1

Rechercher le contenu du fichier pour la chaîne - Exemple 1

```
C:\htb> cd c:\Users\htb-student\Documents & findstr /SI /M "password" *.xml *.ini *.txt

stuff.txt

```

#### Rechercher le contenu du fichier pour la chaîne - Exemple 2

Rechercher le contenu du fichier pour la chaîne - Exemple 2

```
C:\htb> findstr /si mot de passe *.xml *.ini *.txt *.config

stuff.txt : mot de passe : l#-x9r11_2_GL !

```

#### Rechercher le contenu du fichier pour la chaîne - Exemple 3

Rechercher le contenu du fichier pour la chaîne - Exemple 3

```
C:\htb> findstr /spin "mot de passe" *.*

stuff.txt:1:mot de passe : l#-x9r11_2_GL !

```

#### Rechercher le contenu du fichier avec PowerShell

Nous pouvons également effectuer des recherches à l'aide de PowerShell de différentes manières. Voici un exemple.

Rechercher le contenu du fichier avec PowerShell

```
PS C:\htb> select-string -Path C:\Users\htb-student\Documents\*.txt -Pattern password

stuff.txt:1:mot de passe : l#-x9r11_2_GL !

```

#### Recherche d'extensions de fichiers - Exemple 1

Recherche d'extensions de fichiers - Exemple 1

```
C:\htb> répertoire /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config*

c:\inetpub\wwwroot\web.config

```

#### Recherche d'extensions de fichiers - Exemple 2

Recherche d'extensions de fichiers - Exemple 2

```
C:\htb> où /R C:\ *.config

c:\inetpub\wwwroot\web.config

```

De même, nous pouvons rechercher dans le système de fichiers certaines extensions de fichiers avec une commande telle que :

#### Rechercher des extensions de fichier à l'aide de PowerShell

Rechercher des extensions de fichiers à l'aide de PowerShell

```
PS C:\htb> Get-ChildItem C:\ -Recurse -Include *.rdp, *.config, *.vnc, *.cred -ErrorAction Ignorer

     Répertoire : C:\inetpub\wwwroot

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
-a---- 25/05/2021 09:59 329 web.config

<SNIP>

```

* * * * *

Mots de passe des notes autocollantes
----------------------

Les gens utilisent souvent l'application StickyNotes sur les postes de travail Windows pour enregistrer des mots de passe et d'autres informations, sans se rendre compte qu'il s'agit d'un fichier de base de données. Ce fichier se trouve dans `C:\Users\<user>\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite` et vaut toujours la peine d'être recherché et examiné.

#### Recherche de fichiers de base de données StickyNotes

Recherche de fichiers de base de données StickyNotes

```
PS C:\htb>ls

     Répertoire : C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
-a---- 25/05/2021 11:59 20480 15cbbc93e90a4d56bf8d9a29305b8981.storage.session
-a---- 25/05/2021 11:59 AM 982 Ecs.dat
-a---- 25/05/2021 11:59 4096 plum.sqlite
-a---- 25/05/2021 11:59 32768 plum.sqlite-shm
-a---- 25/05/2021 12:00 197792 plum.sqlite-wal

```

Nous pouvons copier les trois fichiers `plum.sqlite*` sur notre système et les ouvrir avec un outil tel que [DB Browser for SQLite](https://sqlitebrowser.org/dl/) et afficher la colonne `Text` dans la table `Note` avec la requête `sélectionner le texte de la note ;`.

![image](https://academy.hackthebox.com/storage/modules/67/stickynote.png)

#### Affichage des données de notes autocollantes à l'aide de PowerShell

Cela peut également être fait avec PowerShell à l'aide du [module PSSQLite](https://github.com/RamblingCookieMonster/PSSQLite). Tout d'abord, importez le module, pointez vers une source de données (dans ce cas, le fichier de base de données SQLite utilisé par l'application StickNotes), puis interrogez la table `Note` et recherchez toute donnée intéressante. Cela peut également être fait depuis notre machine d'attaque après avoir téléchargé le fichier `.sqlite` ou à distance via WinRM.

Affichage des données de notes autocollantes à l'aide de PowerShell

```
PS C:\htb> Set-ExecutionPolicy Bypass -Scope Process

Modification de la stratégie d'exécution
La politique d'exécution vous protège des scripts auxquels vous ne faites pas confiance. La modification de la stratégie d'exécution peut exposer
aux risques de sécurité décrits dans la rubrique d'aide about_Execution_Policies à l'adresse
https://go.microsoft.com/fwlink/?LinkID=135170. Voulez-vous changer la politique d'exécution ?
[O] Oui [A] Oui à tous [N] Non [L] Non à tous [S] Suspendre [?] Aide (la valeur par défaut est "N") : A

PS C:\htb> cd .\PSSQLite\
PS C:\htb> Import-Module .\PSSQLite.psd1
PS C:\htb> $db = 'C:\Users\htb-student\AppData\Local\Packages\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\LocalState\plum.sqlite'
PS C:\htb> Invoke-SqliteQuery -Database $db -Query "SELECT Text FROM Note" | pi-envelopper

Texte
----
\id=de368df0-6939-4579-8d38-0fda521c9bc4 vCenter
\id=e4adae4c-a40b-48b4-93a5-900247852f96
\id=1a44a631-6fff-4961-a4df-27898e9e1e65 racine : Vc3nt3R_adm1n !
\id=c450fc5f-dc51-4412-b4ac-321fd41c522a Démo thycotique demain à 10h

```

#### Chaînes pour afficher le contenu du fichier de base de données

Nous pouvons également les copier dans notre boîte d'attaque et rechercher dans les données à l'aide de la commande `strings` , qui peut être moins efficace en fonction de la taille de la base de données.

Chaînes pour afficher le contenu du fichier de base de données

```
dsgsec@htb[/htb]$ chaînes plum.sqlite-wal

CRÉER TABLE "Note" (
"Texte" varchar ,
"WindowPosition" varchar ,
"EstOuvert" entier ,
"EstToujoursAuTop" entier ,
"CreationNoteIdAnchor" varchar ,
"Thème" varchar ,
"IsFutureNote" entier ,
"RemoteId" varchar ,
"ChangeKey" varchar ,
"LastServerVersion" varchar ,
Entier "RemoteSchemaVersion" ,
"IsRemoteDataInvalid" entier ,
Entier "PendingInsightsScan" ,
"Tapez" varchar ,
"Id" varchar clé primaire non nulle ,
"IdParent" varchar ,
"Créé à" bigint ,
"Supprimé à" bigint ,
"Mis à jour à" bigint )'
indexsqlite_autoindex_Note_1Note
af907b1b-1eef-4d29-b238-3ea74f7ffe5caf907b1b-1eef-4d29-b238-3ea74f7ffe5c
Uaf907b1b-1eef-4d29-b238-3ea74f7ffe5c
Jaune93b49900-6530-42e0-b35c-2663989ae4b3af907b1b-1eef-4d29-b238-3ea74f7ffe5c
U 93b49900-6530-42e0-b35c-2663989ae4b3

< SNIP >

\id=011f29a4-e37f-451d-967e-c42b818473c2 vCenter
\id=34910533-ddcf-4ac4-b8ed-3d1f10be9e61 d'accord*
\id=ffaea2ff-b4fc-4a14-a431-998dc833208c root:Vc3nt3R_adm1n!ManagedPosition=Yellow93b49900-6530-42e0-b35c-2663989ae4b3af907b1b-1eef-4d29-b238-3ea74f 7ffe5c

<SNIP>

```

* * * * *

Autres fichiers d'intérêt
------------------------

#### Autres fichiers intéressants

Certains autres fichiers dans lesquels nous pouvons trouver des informations d'identification incluent les éléments suivants :

Autres fichiers intéressants

```
%SYSTEMDRIVE%\pagefile.sys
%WINDIR%\debug\NetSetup.log
%WINDIR%\repair\sam
%WINDIR%\réparation\système
%WINDIR%\réparation\logiciel, %WINDIR%\réparation\sécurité
%WINDIR%\iis6.log
%WINDIR%\system32\config\AppEvent.Evt
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
%WINDIR%\system32\CCM\logs\*.log
%USERPROFILE%\ntuser.dat
%USERPROFILE%\LocalS~1\Tempor~1\Content.IE5\index.dat
%WINDIR%\System32\drivers\etc\hosts
C:\ProgramData\Configs\*
C:\Program Files\Windows PowerShell\*

```

Certains des scripts d'énumération d'escalade de privilèges répertoriés précédemment dans ce module recherchent la plupart, sinon la totalité, des fichiers/extensions mentionnés dans cette section. Néanmoins, nous devons comprendre comment les rechercher manuellement et ne pas nous fier uniquement aux outils. De plus, nous pouvons trouver des fichiers intéressants que les scripts d'énumération ne recherchent pas et souhaiter modifier les scripts pour les inclure.