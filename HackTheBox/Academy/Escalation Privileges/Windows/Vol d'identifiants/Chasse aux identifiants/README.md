Chasse aux identifiants
==================

* * * * *

Les diplômes peuvent nous ouvrir de nombreuses portes lors de nos évaluations. Nous pouvons trouver des informations d'identification lors de notre énumération d'escalade de privilèges qui peuvent mener directement à un accès administrateur local, nous donner un pied dans l'environnement de domaine Active Directory, ou même être utilisées pour élever les privilèges au sein du domaine. Il existe de nombreux endroits où nous pouvons trouver des informations d'identification sur un système, certaines plus évidentes que d'autres.

* * * * *

Fichiers de configuration des applications
-------------------------------

#### Recherche de fichiers

Contrairement aux meilleures pratiques, les applications stockent souvent les mots de passe dans des fichiers de configuration en texte clair. Supposons que nous obtenions l'exécution d'une commande dans le contexte d'un compte utilisateur non privilégié. Dans ce cas, nous pourrons peut-être trouver des informations d'identification pour leur compte administrateur ou un autre compte local ou de domaine privilégié. Nous pouvons utiliser l'utilitaire [findstr](https://ss64.com/nt/findstr.html) pour rechercher ces informations sensibles.

Recherche de fichiers

```
PS C:\htb> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml

```

Les informations IIS sensibles telles que les informations d'identification peuvent être stockées dans un fichier `web.config` . Pour le site Web IIS par défaut, il peut se trouver à `C:\inetpub\wwwroot\web.config`, mais il peut exister plusieurs versions de ce fichier à différents emplacements, que nous pouvons rechercher de manière récursive.

* * * * *

Fichiers de dictionnaire
----------------

#### Fichiers de dictionnaire Chrome

Un autre cas intéressant est celui des fichiers de dictionnaire. Par exemple, des informations sensibles telles que des mots de passe peuvent être saisies dans un client de messagerie ou une application basée sur un navigateur, qui souligne les mots qu'il ne reconnaît pas. L'utilisateur peut ajouter ces mots à son dictionnaire pour éviter le soulignement rouge gênant.

Fichiers de dictionnaire Chrome

```
PS C:\htb> gc 'C:\Users\htb-student\AppData\Local\Google\Chrome\User Data\Default\Custom Dictionary.txt' | Mot de passe Select-String

Mot de passe1234 !

```

* * * * *

Fichiers d'installation sans assistance
-----------------------------

Les fichiers d'installation sans assistance peuvent définir des paramètres de connexion automatique ou des comptes supplémentaires à créer dans le cadre de l'installation. Les mots de passe dans `unattend.xml` sont stockés en texte brut ou encodés en base64.

#### Unattend.xml

Code : xml

```
<?xml version="1.0" encoding="utf-8" ?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
     <paramètres pass="spécialiser">
         <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/ 2002/État" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
             <Connexion automatique>
                 <Mot de passe>
                     <Valeur>local_4dmin_p@ss</Valeur>
                     <PlainText>vrai</PlainText>
                 </Password>
                 <Activé>vrai</Activé>
                 <LogonCount>2</LogonCount>
                 <Nom d'utilisateur>Administrateur</Nom d'utilisateur>
             </Connexion automatique>
             <NomOrdinateur>*</NomOrdinateur>
         </composant>
     </paramètres>

```

Bien que ces fichiers doivent être automatiquement supprimés dans le cadre de l'installation, les administrateurs système peuvent avoir créé des copies du fichier dans d'autres dossiers lors du développement de l'image et du fichier de réponses.

* * * * *

Fichier d'historique PowerShell
------------------------

#### Commande pour

À partir de Powershell 5.0 dans Windows 10, PowerShell stocke l'historique des commandes dans le fichier :

- `C:\Users\<nom d'utilisateur>\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`.

#### Confirmation du chemin d'enregistrement de l'historique PowerShell

Comme on le voit dans le PDF (pratique) des commandes Windows, publié par Microsoft [ici](https://download.microsoft.com/download/5/8/9/58911986-D4AD-4695-BF63-F734CD4DF8F2/ws-commands. pdf), il existe de nombreuses commandes qui peuvent transmettre des informations d'identification sur la ligne de commande. Nous pouvons voir dans l'exemple ci-dessous que les informations d'identification administratives locales spécifiées par l'utilisateur pour interroger le journal des événements d'application à l'aide de [wevutil](https://ss64.com/nt/wevtutil.html).

Confirmation du chemin d'enregistrement de l'historique PowerShell

```
PS C:\htb> (Get-PSReadLineOption).HistorySavePath

C:\Users\htb-student\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

```

#### Lecture du fichier d'historique de PowerShell

Une fois que nous connaissons l'emplacement du fichier (le chemin par défaut est ci-dessus), nous pouvons essayer de lire son contenu en utilisant `gc`.

Lecture du fichier d'historique de PowerShell

```
PS C:\htb> gc (Get-PSReadLineOption).HistorySavePath

directeur
Temp cd
sauvegardes md
cp c:\inetpub\wwwroot\* .\backups\
Set-ExecutionPolicy Bypass -Scope Process -Force ; [System.Net.ServicePointManager] ::SecurityProtocol = [System.Net.ServicePointManager] ::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://www.powershellgallery.com/packages/MrAToolbox/1.0.1/Content/Get-IISSite.ps1'))
. .\Get-IISsite.ps1
Get-IISsite -Server WEB02 -web "Site Web par défaut"
wevtutil qe Application "/q:*[Application [(EventID=3005)]]" /f:text /rd:true /u:WEB02\administrator /p:5erv3rAdmin ! /r:WEB02

```

Nous pouvonsutilisez également ce one-liner pour récupérer le contenu de tous les fichiers d'historique Powershell auxquels nous pouvons accéder en tant qu'utilisateur actuel. Cela peut également être extrêmement utile en tant qu'étape post-exploitation. Nous devons toujours revérifier ces fichiers une fois que nous avons un administrateur local si notre accès précédent ne nous a pas permis de lire les fichiers pour certains utilisateurs. Cette commande suppose que le chemin de sauvegarde par défaut est utilisé.

Lecture du fichier d'historique de PowerShell

```
PS C:\htb> foreach($user in ((ls C:\users).fullname)){cat "$user\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt" -ErrorAction SilentlyContinue}

directeur
Temp cd
sauvegardes md
cp c:\inetpub\wwwroot\* .\backups\
Set-ExecutionPolicy Bypass -Scope Process -Force ; [System.Net.ServicePointManager] ::SecurityProtocol = [System.Net.ServicePointManager] ::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://www.powershellgallery.com/packages/MrAToolbox/1.0.1/Content/Get-IISSite.ps1'))
. .\Get-IISsite.ps1
Get-IISsite -Server WEB02 -web "Site Web par défaut"
wevtutil qe Application "/q:*[Application [(EventID=3005)]]" /f:text /rd:true /u:WEB02\administrator /p:5erv3rAdmin ! /r:WEB02

```

* * * * *

Identifiants PowerShell
----------------------

Les informations d'identification PowerShell sont souvent utilisées pour les tâches de script et d'automatisation afin de stocker facilement les informations d'identification chiffrées. Les informations d'identification sont protégées à l'aide [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API), ce qui signifie généralement qu'elles ne peuvent être déchiffrées que par le même utilisateur sur le même ordinateur sur lequel elles ont été créées.

Prenons, par exemple, le script suivant `Connect-VC.ps1`, qu'un administrateur système a créé pour se connecter facilement à un serveur vCenter.

Code : PowerShell

```
# Connect-VC.ps1
# Get-Credential | Export-Clixml -Path 'C:\scripts\pass.xml'
$encryptedPassword = Import-Clixml -Path 'C:\scripts\pass.xml'
$decryptedPassword = $encryptedPassword.GetNetworkCredential().Password
Connect-VIServer -Server 'VC-01' -User 'bob_adm' -Mot de passe $encryptedString

```

#### Décryptage des informations d'identification PowerShell

Si nous avons obtenu l'exécution de la commande dans le contexte de cet utilisateur ou si nous pouvons abuser de DPAPI, nous pouvons récupérer les informations d'identification en texte clair à partir de `encrypted.xml`. L'exemple ci-dessous suppose le premier.

Décryptage des informations d'identification PowerShell

```
PS C:\htb> $credential = Import-Clixml -Path 'C:\scripts\pass.xml'
PS C:\htb> $credential.GetNetworkCredential().nom d'utilisateur

bob

PS C:\htb> $credential.GetNetworkCredential().password

Str0ng3ncryptedP@ss !

```