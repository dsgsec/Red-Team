![0e4c217223af2541ddb04d61ddd9753d](https://github.com/dsgsec/Red-Team/assets/82456829/44dc505d-c316-438d-a5be-e1e42af18aab)

Dans cette tâche, nous supposons que vous avez accès à `cmd`un hôte Microsoft Windows. Vous avez peut-être obtenu cet accès en exploitant une vulnérabilité et en obtenant un shell ou un shell inversé. Vous avez peut-être également installé une porte dérobée ou configuré un serveur SSH sur un système que vous avez exploité. Dans tous les cas, les commandes ci-dessous nécessitent`cmd` d'être exécutées.

Dans cette tâche, nous nous concentrons sur l'énumération d'un hôte MS Windows. Pour énumérer MS Active Directory, nous vous encourageons à consulter la salle [Énumération d'Active Directory . ](https://tryhackme.com/room/adenumeration)Si vous êtes intéressé par une élévation de privilèges sur un hôte MS Windows, nous vous recommandons la salle [Windows Privesc 2.0](https://tryhackme.com/room/windowsprivesc20) .

Nous vous recommandons de cliquer sur « Démarrer AttackBox » et « Démarrer la machine » afin de pouvoir expérimenter et répondre aux questions à la fin de cette tâche.

### Système

Une commande pouvant nous fournir des informations détaillées sur le système, telles que son numéro de build et les correctifs installés, serait `systeminfo`. Dans l'exemple ci-dessous, nous pouvons voir quels correctifs ont été installés.

Terminal

```
C:\>systeminfo

Host Name:                 WIN-SERVER-CLI
OS Name:                   Microsoft Windows Server 2022 Standard
OS Version:                10.0.20348 N/A Build 20348
OS Manufacturer:           Microsoft Corporation
[...]
Hotfix(s):                 3 Hotfix(s) Installed.
                           [01]: KB5013630
                           [02]: KB5013944
                           [03]: KB5012673
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
[...]
```

Vous pouvez vérifier les mises à jour installées à l'aide de `wmic qfe get Caption,Description`. Ces informations vous donneront une idée de la rapidité avec laquelle les systèmes sont corrigés et mis à jour.

Terminal

```
C:\>wmic qfe get Caption,Description
Caption                                     Description
http://support.microsoft.com/?kbid=5013630  Update
https://support.microsoft.com/help/5013944  Security Update
                                            Update
```

Vous pouvez vérifier les services Windows installés et démarrés à l'aide de `net start`. Attendez-vous à avoir une longue liste ; la sortie ci-dessous a été coupée.

Terminal

```
C:\>net start
These Windows services are started:

   Base Filtering Engine
   Certificate Propagation
   Client License Service (ClipSVC)
   COM+ Event System
   Connected User Experiences and Telemetry
   CoreMessaging
   Cryptographic Services
   DCOM Server Process Launcher
   DHCP Client
   DNS Client
[...]
   Windows Time
   Windows Update
   WinHTTP Web Proxy Auto-Discovery Service
   Workstation

The command completed successfully.
```

Si vous n'êtes intéressé que par les applications installées, vous pouvez émettre des fichiers `wmic product get name,version,vendor`. Si vous exécutez cette commande sur la machine virtuelle connectée, vous obtiendrez quelque chose de similaire au résultat suivant.

Terminal

```
C:\>wmic product get name,version,vendor
Name                                                            Vendor                                   Version
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29910     Microsoft Corporation                    14.28.29910
[...]
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29910  Microsoft Corporation                    14.28.29910
```

### Utilisateurs

Pour savoir qui vous êtes, vous pouvez courir `whoami`; de plus, pour savoir de quoi vous êtes capable, c'est-à-dire vos privilèges, vous pouvez utiliser `whoami /priv`. Un exemple est présenté dans la sortie du terminal ci-dessous.

Terminal

```
C:\>whoami
win-server-cli\strategos

> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== =======
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Enabled
SeSecurityPrivilege                       Manage auditing and security log                                   Enabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Enabled
[...]
```

De plus, vous pouvez utiliser `whoami /groups`pour savoir à quels groupes vous appartenez. La sortie du terminal ci-dessous montre que cet utilisateur appartient à `NT AUTHORITY\Local account and member of Administrators group`d'autres groupes.

Terminal

```
C:\>whoami /groups

GROUP INFORMATION
-----------------

Group Name                                                    Type             SID          Attributes
============================================================= ================ ============ ===============================================================
Everyone                                                      Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account and member of Administrators group Well-known group S-1-5-114    Mandatory group, Enabled by default, Enabled group
BUILTIN\Administrators                                        Alias            S-1-5-32-544 Mandatory group, Enabled by default, Enabled group, Group owner
[...]
```

Vous pouvez afficher les utilisateurs en exécutant `net user`.

Terminal

```
C:\>net user

User accounts for \\WIN-SERVER-CLI

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
michael                  peter                    strategos
WDAGUtilityAccount
The command completed successfully.
```

Vous pouvez découvrir les groupes disponibles en indiquant `net group`si le système est un contrôleur de domaine Windows ou `net localgroup`autre, comme indiqué dans le terminal ci-dessous.

Terminal

```
C:\>net localgroup

Aliases for \\WIN-SERVER-CLI

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Administrators
*Backup Operators
*Certificate Service DCOM Access
*Cryptographic Operators
*Device Owners
[...]
```

Vous pouvez lister les utilisateurs appartenant au groupe des administrateurs locaux à l'aide de la commande `net localgroup administrators`.

Terminal

```
C:\>net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
michael
peter
strategos
The command completed successfully.
```

Utilisez `net accounts`pour voir les paramètres locaux sur une machine ; de plus, vous pouvez l'utiliser `net accounts /domain`si la machine appartient à un domaine. Cette commande permet d'en savoir plus sur la politique de mot de passe, telle que la longueur minimale du mot de passe, l'âge maximum du mot de passe et la durée du verrouillage.

### La mise en réseau

Vous pouvez utiliser la `ipconfig`commande pour en savoir plus sur la configuration réseau de votre système. Si vous souhaitez connaître tous les paramètres liés au réseau, vous pouvez utiliser `ipconfig /all`. La sortie du terminal ci-dessous montre la sortie lors de l'utilisation de `ipconfig`. Par exemple, nous aurions pu l'utiliser `ipconfig /all`si nous voulions apprendre les serveurs DNS .

Terminal

```
C:\>ipconfig

Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : localdomain
   Link-local IPv6 Address . . . . . : fe80::3dc5:78ef:1274:a740%5
   IPv4 Address. . . . . . . . . . . : 10.20.30.130
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.20.30.2
```

Sous MS Windows, nous pouvons utiliser `netstat`pour obtenir diverses informations, telles que les ports sur lesquels le système écoute, quelles connexions sont actives et qui les utilise. Dans cet exemple, nous utilisons les options `-a`pour afficher tous les ports d'écoute et connexions actives. Le `-b`nous permet de trouver le binaire impliqué dans la connexion, tandis que `-n`est utilisé pour éviter de résoudre les adresses IP et les numéros de port. Enfin, `-o`affichez l'ID du processus ( PID ).

Dans la sortie partielle présentée ci-dessous, nous pouvons voir que `netstat -abno`le serveur écoute sur les ports TCP 22, 135, 445 et 3389. Les processus`sshd.exe` , `RpcSs`et `TermService`se trouvent respectivement sur les ports `22`, `135`et `3389`. De plus, nous pouvons voir deux connexions établies au serveur SSH comme indiqué par l'état`ESTABLISHED` .

Terminal

```
C:\>netstat -abno

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:22             0.0.0.0:0              LISTENING       2016
 [sshd.exe]
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       924
  RpcSs
 [svchost.exe]
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
 Can not obtain ownership information
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       416
  TermService
 [svchost.exe]
[...]
  TCP    10.20.30.130:22        10.20.30.1:39956       ESTABLISHED     2016
 [sshd.exe]
  TCP    10.20.30.130:22        10.20.30.1:39964       ESTABLISHED     2016
 [sshd.exe]
[...]

```

Vous pourriez penser que vous pouvez obtenir un résultat identique en analysant les ports du système cible ; cependant, cela est inexact pour deux raisons. Un pare-feu empêche peut-être l'hôte d'analyse d'atteindre des ports réseau spécifiques. De plus, l'analyse des ports d'un système génère une quantité considérable de trafic, contrairement à `netstat`, qui ne génère aucun bruit.

Enfin, il convient de mentionner que l'utilisation `arp -a`vous aide à découvrir d'autres systèmes sur le même réseau local qui ont récemment communiqué avec votre système. ARP signifie Protocole de résolution d'adresse ; `arp -a`affiche les entrées ARP actuelles , c'est-à-dire les adresses physiques des systèmes sur le même réseau local qui ont communiqué avec votre système. Un exemple de sortie est présenté ci-dessous. Cela indique que ces adresses IP ont communiqué d'une manière ou d'une autre avec notre système ; la communication peut être une tentative de connexion ou même un simple ping. Notez que cela`10.10.255.255` ne représente pas un système car il s'agit de l'adresse de diffusion du sous-réseau.

Terminal

```
C:\>arp -a

Interface: 10.10.204.175 --- 0x4
  Internet Address      Physical Address      Type
  10.10.0.1             02-c8-85-b5-5a-aa     dynamic
  10.10.16.117          02-f2-42-76-fc-ef     dynamic
  10.10.122.196         02-48-58-7b-92-e5     dynamic
  10.10.146.13          02-36-c1-4d-05-f9     dynamic
  10.10.161.4           02-a8-58-98-1a-d3     dynamic
  10.10.217.222         02-68-10-dd-be-8d     dynamic
  10.10.255.255         ff-ff-ff-ff-ff-ff     static
```
