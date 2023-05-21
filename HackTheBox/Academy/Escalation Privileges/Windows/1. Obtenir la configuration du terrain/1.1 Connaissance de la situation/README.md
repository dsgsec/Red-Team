Connaissance de la situation
=====================

* * * * *

Dans n'importe quelle situation, que ce soit dans notre vie de tous les jours ou lors d'un projet tel qu'un test d'intrusion réseau, il est toujours important de s'orienter dans l'espace et dans le temps. Nous ne pouvons pas fonctionner et réagir efficacement sans une compréhension de notre environnement actuel. Nous avons besoin de ces informations pour prendre des décisions éclairées sur nos prochaines étapes pour opérer de manière proactive plutôt que réactive. Lorsque nous atterrissons sur un système Windows ou Linux avec l'intention d'augmenter ensuite les privilèges, il y a plusieurs choses que nous devons toujours rechercher pour planifier nos prochains mouvements. Nous pouvons trouver d'autres hôtes auxquels nous pouvons accéder directement, des protections en place qui devront être contournées, ou constater que certains outils ne fonctionneront pas contre le système en question.

* * * * *

Informations sur le réseau
-------------------

La collecte d'informations sur le réseau est une partie cruciale de notre dénombrement. Nous pouvons constater que l'hôte est à double hébergement et que compromettre l'hôte peut nous permettre de nous déplacer latéralement dans une autre partie du réseau à laquelle nous ne pouvions pas accéder auparavant. Double hébergement signifie que l'hôte ou le serveur appartient à deux ou plusieurs réseaux différents et, dans la plupart des cas, dispose de plusieurs interfaces réseau virtuelles ou physiques. Nous devons toujours consulter les [tables de routage](https://en.wikipedia.org/wiki/Routing_table) pour afficher des informations sur le réseau local et les réseaux qui l'entourent. Nous pouvons également recueillir des informations sur le domaine local (si l'hôte fait partie d'un environnement Active Directory), y compris les adresses IP des contrôleurs de domaine. Il est également important d'utiliser la commande [arp](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/arp) pour afficher le cache ARP de chaque interface et afficher les autres hôtes. l'hôte a récemment communiqué avec. Cela pourrait nous aider avec le mouvement latéral après l'obtention des informations d'identification. Cela pourrait être une bonne indication des hôtes auxquels les administrateurs se connectent via RDP ou WinRM à partir de cet hôte.

Ces informations sur le réseau peuvent aider directement ou indirectement à notre élévation de privilèges locaux. Cela peut nous conduire sur une autre voie vers un système auquel nous pouvons accéder ou augmenter les privilèges sur ou révéler des informations que nous pouvons utiliser pour un mouvement latéral afin de poursuivre notre accès après avoir augmenté les privilèges sur le système actuel.

#### Interface(s), adresse(s) IP, informations DNS

Interface(s), adresse(s) IP, informations DNS

```
C:\htb> ipconfig /all

Configuration IP de Windows

    Nom d'hôte . . . . . . . . . . . . : WINLPE-SRV01
    Suffixe DNS principal . . . . . . . :
    Type de nœud . . . . . . . . . . . . : Hybride
    Routage IP activé. . . . . . . . : Non
    Proxy WINS activé. . . . . . . . : Non
    Liste de recherche de suffixe DNS. . . . . . : .htb

Adaptateur Ethernet Ethernet1 :

    Suffixe DNS spécifique à la connexion . :
    Description . . . . . . . . . . . : adaptateur Ethernet vmxnet3
    Adresse physique. . . . . . . . . : 00-50-56-B9-C5-4B
    DHCP activé. . . . . . . . . . . : Non
    Configuration automatique activée. . . . : Oui
    Adresse IPv6 lien-local . . . . . : fe80::f055:fefd:b1b:9919%9(Préféré)
    Adresse IPv4. . . . . . . . . . . : 192.168.20.56(Préféré)
    Masque de sous-réseau . . . . . . . . . . . : 255.255.255.0
    Passerelle par défaut . . . . . . . . . : 192.168.20.1
    IAID DHCPv6. . . . . . . . . . . : 151015510
    DUID client DHCPv6. . . . . . . . : 00-01-00-01-27-ED-DB-68-00-50-56-B9-90-94
    Serveurs DNS. . . . . . . . . . . : 8.8.8.8
    NetBIOS sur Tcpip. . . . . . . . : Activé

Adaptateur Ethernet Ethernet0 :

    Suffixe DNS spécifique à la connexion . : .htb
    Description . . . . . . . . . . . : Connexion réseau Intel(R) 82574L Gigabit
    Adresse physique. . . . . . . . . : 00-50-56-B9-90-94
    DHCP activé. . . . . . . . . . . : Oui
    Configuration automatique activée. . . . : Oui
    Adresse IPv6. . . . . . . . . . . : dead:beef::e4db:5ea3:2775:8d4d(Préféré)
    Adresse IPv6 lien-local . . . . . : fe80::e4db:5ea3:2775:8d4d%4(Préféré)
    Adresse IPv4. . . . . . . . . . . : 10.129.43.8(Préféré)
    Masque de sous-réseau . . . . . . . . . . . : 255.255.0.0
    Bail obtenu. . . . . . . . . . : jeudi 25 mars 2021 09:24:45
    Le bail expire. . . . . . . . . . : lundi 29 mars 2021 13:28:44
    Passerelle par défaut . . . . . . . . . : fe80::250:56ff:feb9:4ddf%4
                                        10.129.0.1
    Serveur DHCP . . . . . . . . . . . : 10.129.0.1
    IAID DHCPv6. . . . . . . . . . . : 50352214
    DUID client DHCPv6. . . . . . . . : 00-01-00-01-27-ED-DB-68-00-50-56-B9-90-94
    Serveurs DNS. . . . . . . . . . . : 1.1.1.1
                                        8.8.8.8
    NetBIOS sur Tcpip. . . . . . . . : Activé

Adaptateur de tunnel isatap..htb :

    État des médias. . . . . . . . . . . : Média déconnecté
    Suffixe DNS spécifique à la connexion . : .htb
    Description . . . . . . . . . . . : Adaptateur Microsoft ISATAP
    Adresse physique. . . . . . . . . : 00-00-00-00-00-00-00-E0
    DHCP activé. . . . . . . . . . . : Non
    Configuration automatique activée. . . . : Oui

Adaptateur de tunnel Teredo Tunneling Pseudo-Interfacee :

    État des médias. . . . . . . . . . . : Média déconnecté
    Suffixe DNS spécifique à la connexion . :
    Description . . . . . . . . . . . : Teredo Tunneling Pseudo-Interface
    Adresse physique. . . . . . . . . : 00-00-00-00-00-00-00-E0
    DHCP activé. . . . . . . . . . . : Non
    Configuration automatique activée. . . . : Oui

Adaptateur de tunnel isatape.{02D6F04C-A625-49D1-A85D-4FB454FBB3DB} :

    État des médias. . . . . . . . . . . : Média déconnecté
    Suffixe DNS spécifique à la connexion . :
    Description . . . . . . . . . . . : Adaptateur Microsoft ISATAP #2
    Adresse physique. . . . . . . . . : 00-00-00-00-00-00-00-E0
    DHCP activé. . . . . . . . . . . : Non
    Configuration automatique activée. . . . : Oui

```

#### Tableau ARP

Tableau ARP

```
C:\htb> arp -a

Interface : 10.129.43.8 --- 0x4
   Adresse Internet Type d'adresse physique
   10.129.0.1 00-50-56-b9-4d-df dynamique
   10.129.43.12 00-50-56-b9-da-ad dynamique
   10.129.43.13 00-50-56-b9-5b-9f dynamique
   10.129.255.255 ff-ff-ff-ff-ff-ff statique
   224.0.0.22 01-00-5e-00-00-16 statique
   224.0.0.252 01-00-5e-00-00-fc statique
   224.0.0.253 01-00-5e-00-00-fd statique
   239.255.255.250 01-00-5e-7f-ff-fa statique
   255.255.255.255 ff-ff-ff-ff-ff-ff statique

Interface : 192.168.20.56 --- 0x9
   Adresse Internet Type d'adresse physique
   192.168.20.255 ff-ff-ff-ff-ff-ff statique
   224.0.0.22 01-00-5e-00-00-16 statique
   224.0.0.252 01-00-5e-00-00-fc statique
   239.255.255.250 01-00-5e-7f-ff-fa statique
   255.255.255.255 ff-ff-ff-ff-ff-ff statique

```

#### Table de routage

Table de routage

```
C:\htb> impression de route

================================================= =========================
Liste des interfaces
   9...00 50 56 b9 c5 4b ...... adaptateur Ethernet vmxnet3
   4...00 50 56 b9 90 94 ......Connexion réseau Intel(R) 82574L Gigabit
   1.................................Interface de bouclage logiciel 1
   3...00 00 00 00 00 00 00 e0 Adaptateur Microsoft ISATAP
   5...00 00 00 00 00 00 00 e0 Pseudo-interface de tunnellisation Teredo
  13...00 00 00 00 00 00 00 e0 Adaptateur Microsoft ISATAP #2
================================================= =========================

Table de routage IPv4
================================================= =========================
Itinéraires actifs :
Métrique d'interface de passerelle de masque de réseau de destination du réseau
           0.0.0.0 0.0.0.0 10.129.0.1 10.129.43.8 25
           0.0.0.0 0.0.0.0 192.168.20.1 192.168.20.56 271
        10.129.0.0 255.255.0.0 En liaison 10.129.43.8 281
       10.129.43.8 255.255.255.255 En liaison 10.129.43.8 281
    10.129.255.255 255.255.255.255 En liaison 10.129.43.8 281
         127.0.0.0 255.0.0.0 En liaison 127.0.0.1 331
         127.0.0.1 255.255.255.255 En liaison 127.0.0.1 331
   127.255.255.255 255.255.255.255 En liaison 127.0.0.1 331
      192.168.20.0 255.255.255.0 En liaison 192.168.20.56 271
     192.168.20.56 255.255.255.255 En liaison 192.168.20.56 271
    192.168.20.255 255.255.255.255 En liaison 192.168.20.56 271
         224.0.0.0 240.0.0.0 En liaison 127.0.0.1 331
         224.0.0.0 240.0.0.0 En liaison 10.129.43.8 281
         224.0.0.0 240.0.0.0 En liaison 192.168.20.56 271
   255.255.255.255 255.255.255.255 En liaison 127.0.0.1 331
   255.255.255.255 255.255.255.255 En liaison 10.129.43.8 281
   255.255.255.255 255.255.255.255 En liaison 192.168.20.56 271
================================================= =========================
Routes persistantes :
   Adresse réseau Netmask Adresse passerelle Métrique
           0.0.0.0 0.0.0.0 192.168.20.1 Par défaut
================================================= =========================

Table de routage IPv6
================================================= =========================
Itinéraires actifs :
  Si passerelle de destination du réseau métrique
   4 281 ::/0 fe80::250:56ff:feb9:4ddf
   1 331 ::1/128 En liaison
   4 281 morts:beef::/64 En liaison
   4 281 morts:boeuf::e4db:5ea3:2775:8d4d/128
                                     En lien
   4 281 fe80 ::/64 en liaison
   9 271 fe80 ::/64 En liaison
   4 281 fe80::e4db:5ea3:2775:8d4d/128
                                     En lien
   9 271 fe80::f055:fefd:b1b:9919/128
                                     En lien
   1 331 ff00 ::/8 En liaison
   4 281 ff00 ::/8 En liaison
   9 271 ff00 ::/8 En liaison
================================================= =========================
Routes persistantes :
   Aucun

```

* * * * *

Énumération des protections
------------------------

L'environnement le plus moderneLes clients disposent d'une sorte d'antivirus ou d'un service Endpoint Detection and Response (EDR) en cours d'exécution pour surveiller, alerter et bloquer les menaces de manière proactive. Ces outils peuvent interférer avec le processus de dénombrement. Ils présenteront très probablement une sorte de défi pendant le processus d'escalade des privilèges, surtout si nous utilisons une sorte d'exploit ou d'outil public PoC. L'énumération des protections en place nous aidera à nous assurer que nous utilisons des méthodes qui ne sont pas bloquées ou détectées et nous aidera si nous devons créer des charges utiles personnalisées ou modifier des outils avant de les compiler.

De nombreuses organisations utilisent une sorte de solution de liste blanche d'applications pour contrôler les types d'applications et de fichiers que certains utilisateurs peuvent exécuter. Cela peut être utilisé pour tenter d'empêcher les utilisateurs non administrateurs d'exécuter `cmd.exe` ou `powershell.exe` ou d'autres fichiers binaires et types de fichiers non nécessaires à leur travail quotidien. Une solution populaire proposée par Microsoft est [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview). Nous pouvons utiliser la cmdlet [GetAppLockerPolicy](https://docs.microsoft.com/en-us/powershell/module/applocker/get-applockerpolicy?view=windowsserver2019-ps) pour énumérer les paramètres locaux, effectifs (appliqués) et stratégies AppLocker de domaine. Cela nous aidera à voir quels fichiers binaires ou types de fichiers peuvent être bloqués et si nous devrons effectuer une sorte de contournement d'AppLocker pendant notre énumération ou avant d'exécuter un outil ou une technique pour augmenter les privilèges.

Dans un engagement réel, le client disposera probablement de protections en place qui détectent les outils/scripts les plus courants (y compris ceux présentés dans la section précédente). Il existe des moyens de les gérer, et l'énumération des protections utilisées peut nous aider à modifier nos outils dans un environnement de laboratoire et à les tester avant de les utiliser sur un système client. Certains outils EDR détectent ou même bloquent l'utilisation de fichiers binaires courants tels que `net.exe`, `tasklist`, etc. exécuté via cmd.exe. Une énumération précoce et une compréhension approfondie de l'environnement du client et des solutions de contournement par rapport aux solutions AV et EDR courantes peuvent nous faire gagner du temps lors d'un engagement non évasif et faire ou défaire un engagement évasif.

#### Vérifier l'état de Windows Defender

Vérifier l'état de Windows Defender

```
PS C:\htb> Get-MpComputerStatus

AMEngineVersion : 1.1.17900.7
AMProductVersion : 4.10.14393.2248
AMServiceEnabled : Vrai
AMServiceVersion : 4.10.14393.2248
AntispywareEnabled : Vrai
AntispywareSignatureAge : 1
AntispywareSignatureDernière mise à jour : 28/03/2021 02:59:13
AntispywareSignatureVersion : 1.333.1470.0
AntivirusActivé : Vrai
AntivirusSignatureAge : 1
AntivirusSignatureDernière mise à jour : 28/03/2021 02:59:12
AntivirusSignatureVersion : 1.333.1470.0
BehaviorMonitorEnabled : Faux
ID ordinateur : 54AF7DE4-3C7E-4DA0-87AC-831B045B9063
État de l'ordinateur : 0
FullScanAge : 4294967295
FullScanEndTime :
FullScanStartTime :
IoavProtectionEnabled : Faux
Dernière source d'analyse complète : 0
LastQuickScanSource : 0
NISabled : Faux
Version NISEngine : 0.0.0.0
NISSignatureÂge : 4294967295
NISSignatureLastUpdated :
NISSignatureVersion : 0.0.0.0
OnAccessProtectionEnabled : Faux
QuickScanAge : 4294967295
QuickScanEndTime :
QuickScanStartTime :
RealTimeProtectionEnabled : Faux
RealTimeScanDirection : 0
PSNomOrdinateur :

```

#### Liste des règles AppLocker

Répertorier les règles AppLocker

```
PS C:\htb> Get-AppLockerPolicy -Effective | sélectionnez -ExpandProperty RuleCollections

Conditions de l'éditeur : {*\*\*,0.0.0.0-*}
Exceptions de l'éditeur : {}
PathExceptions : {}
HashExceptions : {}
Identifiant : a9e18c21-ff8f-43cf-b9fc-db40eed693ba
Nom : (Règle par défaut) Toutes les applications packagées signées
Description : permet aux membres du groupe Tout le monde d'exécuter des applications packagées signées.
UserOrGroupSid : S-1-1-0
Action : Autoriser

Conditions de chemin : {%PROGRAMFILES%\*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : 921cc481-6e17-4653-8f75-050b80acca20
Nom : (Règle par défaut) Tous les fichiers situés dans le dossier Program Files
Description : permet aux membres du groupe Tout le monde d'exécuter des applications situées dans Program Files
                       dossier.
UserOrGroupSid : S-1-1-0
Action : Autoriser

Conditions du chemin : {%WINDIR%\*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Nom : (Règle par défaut) Tous les fichiers situés dans le dossier Windows
Description : permet aux membres du groupe Tout le monde d'exécuter des applications situées dans le dossier Windows.
UserOrGroupSid : S-1-1-0
Action : Autoriser

Conditions du chemin : {*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : fd686d83-a829-4351-8ff4-27c7de5755d2
Nom : (Règle par défaut) Tous les fichiers
Description : permet aux membres du groupe Administrateurs local d'exécuter toutes les applications.
UserOrGroupSid : S-1-5-32-544
Action : Autoriser

Conditions de l'éditeur : {*\*\*,0.0.0.0-*}
Exceptions de l'éditeur : {}
PathExceptions : {}
HashExceptions : {}
Identifiant : b7af7102-efde-4369-8a89-7a6a392d1473
Nom : (Règle par défaut) Tous les fichiers Windows Installer signés numériquement
Description : permet aux membres du groupe Tout le monde d'exécuter des fichiers Windows Installer signés numériquement.
UserOrGroupSid : S-1-1-0
Action : Autoriser

PathConditions : {%WINDIR%\Installer\*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : 5b290184-345a-4453-b184-45305f6d9a54
Nom : (Règle par défaut) Tous les fichiers Windows Installer dans %systemdrive%\Windows\Installer
Description : permet aux membres du groupe Tout le monde d'exécuter tous les fichiers Windows Installer situés dans
                       %systemdrive%\Windows\Installer.
UserOrGroupSid : S-1-1-0
Action : Autoriser

PathConditions : {*.*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : 64ad46ff-0d71-4fa0-a30b-3f3d30c5433d
Nom : (Règle par défaut) Tous les fichiers Windows Installer
Description : permet aux membres du groupe local Administrateurs d'exécuter tous les fichiers Windows Installer.
UserOrGroupSid : S-1-5-32-544
Action : Autoriser

Conditions de chemin : {%PROGRAMFILES%\*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : 06dce67b-934c-454f-a263-2515c8796a5d
Nom : (Règle par défaut) Tous les scripts situés dans le dossier Program Files
Description : permet aux membres du groupe Tout le monde d'exécuter des scripts situés dans le dossier Program Files.
UserOrGroupSid : S-1-1-0
Action : Autoriser

Conditions du chemin : {%WINDIR%\*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : 9428c672-5fc3-47f4-808a-a0011f36dd2c
Nom : (Règle par défaut) Tous les scripts situés dans le dossier Windows
Description : permet aux membres du groupe Tout le monde d'exécuter des scripts situés dans le dossier Windows.
UserOrGroupSid : S-1-1-0
Action : Autoriser

Conditions du chemin : {*}
PathExceptions : {}
Exceptions de l'éditeur : {}
HashExceptions : {}
Identifiant : ed97d0cb-15ff-430f-b82c-8d7832957725
Nom : (Règle par défaut) Tous les scripts
Description : permet aux membres du groupe Administrateurs local d'exécuter tous les scripts.
UserOrGroupSid : S-1-5-32-544
Action : Autoriser

```

#### Tester la stratégie AppLocker

Tester la stratégie AppLocker

```
PS C:\htb> Get-AppLockerPolicy -Local | Test-AppLockerPolicy -chemin C:\Windows\System32\cmd.exe -Utilisateur Tout le monde

FilePath PolicyDecision MatchingRule
-------- -------------- ------------
C:\Windows\System32\cmd.exe Refusé c:\windows\system32\cmd.exe

```

* * * * *

Prochaines étapes
----------

Maintenant que nous avons rassemblé des informations réseau sur l'hôte et énuméré toutes les protections en place, nous pouvons décider quels outils ou techniques manuelles utiliser dans les phases d'énumération suivantes et toutes les voies d'attaque possibles supplémentaires au sein du réseau.