Avant d'effectuer d'autres actions, nous devons acquérir des connaissances générales sur les solutions de sécurité en place. N'oubliez pas qu'il est important d'énumérer les méthodes de détection antivirus et de sécurité sur un point final afin de rester aussi non détecté que possible et de réduire les risques d'être détecté.

Cette tâche abordera la solution de sécurité courante utilisée dans les réseaux d'entreprise, divisée en  solutions de sécurité hôte  et  réseau . 

## Solutions de sécurité hôte

![0e69d6b953d1c9c11d8581d31ea246d8](https://github.com/dsgsec/Red-Team/assets/82456829/b918b77b-1324-45e8-987d-a970121addd9)

Il s'agit d'un ensemble d'applications logicielles utilisées pour surveiller et détecter les activités anormales et malveillantes au sein de l'hôte, notamment :

1.  Logiciel antivirus

2.  Microsoft Windows Défenseur
3.  Pare-feu basé sur l'hôte
4.  Journalisation et surveillance des événements de sécurité

5.  Système de détection d'intrusion basé sur l'hôte ( HIDS )/Système de prévention d'intrusion basé sur l'hôte (HIPS)
6.  Détection et réponse des points de terminaison ( EDR )

Examinons plus en détail les solutions de sécurité basées sur l'hôte que nous pouvons rencontrer lors de l'engagement de l'équipe rouge.

## Logiciel antivirus ( AV )

Les logiciels antivirus, également appelés anti-malware, sont principalement utilisés pour surveiller, détecter et empêcher l'exécution de logiciels malveillants au sein de l'hôte.  La plupart des applications logicielles antivirus utilisent des fonctionnalités bien connues, notamment l'analyse en arrière-plan, les analyses complètes du système et les définitions de virus. Lors de l'analyse en arrière-plan, le logiciel antivirus fonctionne en temps réel et analyse tous les fichiers ouverts et utilisés en arrière-plan. L'analyse complète du système est essentielle lors de la première installation de l'antivirus. La partie la plus intéressante concerne les définitions de virus, où le logiciel antivirus répond au virus prédéfini. C'est pourquoi le logiciel antivirus doit être mis à jour de temps en temps.

Il existe diverses techniques de détection utilisées par l'antivirus, notamment

-   Détection basée sur les signatures
-   Détection heuristique
-   Détection basée sur le comportement

La détection basée sur les signatures  est l'une des techniques courantes et traditionnelles utilisées dans les logiciels antivirus pour identifier les fichiers malveillants. Souvent, les chercheurs ou les utilisateurs soumettent leurs fichiers infectés à une plate-forme de moteur antivirus pour une analyse plus approfondie par les fournisseurs d'antivirus , et si cela s'avère malveillant, la signature est enregistrée dans leur base de données. Le logiciel antivirus  compare le fichier analysé avec une base de données de signatures connues pour détecter d'éventuelles attaques et logiciels malveillants côté client. Si nous avons un match, alors il considère une menace.

La détection heuristique  utilise l'apprentissage automatique pour décider si nous possédons ou non le fichier malveillant. Il analyse et analyse statiquement en temps réel afin de trouver des propriétés suspectes dans le code de l'application ou de vérifier si elle utilise des API Windows ou système inhabituelles. Il ne s'appuie pas sur l'attaque basée sur les signatures pour prendre des décisions, ou c'est parfois le cas. Cela dépend de la mise en œuvre du logiciel antivirus.

Enfin,  la détection basée sur le comportement  repose sur la surveillance et l'examen de l'exécution des applications pour détecter des comportements anormaux et des activités inhabituelles, telles que la création/mise à jour de valeurs dans les clés de registre, la suppression/création de processus, etc.

En tant que red teamer, il est essentiel de savoir si un antivirus existe ou non. Cela nous empêche de faire ce que nous essayons de faire. Nous pouvons énumérer les logiciels AV à l'aide d'outils intégrés à Windows, tels que  wmic .

PowerShell

```
PS C:\Users\thm> wmic /namespace:\\root\securitycenter2 path antivirusproduct
```

Cela peut également être fait en utilisant PowerShell , qui donne le même résultat.

PowerShell

```
PS C:\Users\thm> Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct

displayName              : Bitdefender Antivirus
instanceGuid             : {BAF124F4-FA00-8560-3FDE-6C380446AEFB}
pathToSignedProductExe   : C:\Program Files\Bitdefender\Bitdefender Security\wscfix.exe
pathToSignedReportingExe : C:\Program Files\Bitdefender\Bitdefender Security\bdservicehost.exe
productState             : 266240
timestamp                : Wed, 15 Dec 2021 12:40:10 GMT
PSComputerName           :

displayName              : Windows Defender
instanceGuid             : {D58FFC3A-813B-4fae-9E44-DA132C9FAA36}
pathToSignedProductExe   : windowsdefender://
pathToSignedReportingExe : %ProgramFiles%\Windows Defender\MsMpeng.exe
productState             : 393472
timestamp                : Fri, 15 Oct 2021 22:32:01 GMT
PSComputerName           :
```

En conséquence, un antivirus tiers (Bitdefender Antivirus) et Windows Defender sont installés sur l'ordinateur. Notez  que les serveurs Windows peuvent ne pas avoir  d'espace de noms SecurityCenter2 , ce qui peut ne pas fonctionner sur la machine virtuelle connectée . Au lieu de cela, cela fonctionne pour les postes de travail Windows ! 

## Microsoft Windows Défenseur

Microsoft Windows Defender est un outil de sécurité antivirus préinstallé qui s'exécute sur les points finaux. Il utilise divers algorithmes pour la détection, notamment l'apprentissage automatique, l'analyse du Big Data, la recherche approfondie sur la résistance aux menaces et l'infrastructure cloud Microsoft pour la protection contre les logiciels malveillants et les virus. MS Defender fonctionne selon trois modes de protection : modes actif, passif et désactivé. 

Le mode actif est utilisé lorsque MS Defender s'exécute comme logiciel antivirus principal sur la machine où il assure la protection et la correction. Le mode passif est exécuté lorsqu'un logiciel antivirus tiers est installé. Par conséquent, il fonctionne comme un logiciel antivirus secondaire où il analyse les fichiers et détecte les menaces mais ne fournit pas de solutions correctives. Enfin, le mode Désactiver correspond au moment où MS Defender est désactivé ou désinstallé du système.

 Nous pouvons utiliser la commande PowerShell suivante pour  vérifier  l'état du service de Windows Defender :

PowerShell

```
PS C:\Users\thm> Get-Service WinDefend

Status   Name               DisplayName
------   ----               -----------
Running  WinDefend          Windows Defender Antivirus Service
```

Ensuite, nous pouvons commencer à utiliser la cmdlet Get-MpComputerStatus pour obtenir l'état actuel de Windows Defender. Cependant, il  fournit l'état actuel des éléments de la solution de sécurité , notamment  l'anti-spyware, l'antivirus, LoavProtection, la protection en temps réel, etc. Nous pouvons utiliser  select pour spécifier ce dont nous avons besoin comme suit,   

PowerShell

```
PS C:\Users\thm> Get-MpComputerStatus | select RealTimeProtectionEnabled

RealTimeProtectionEnabled
-------------------------
                    False
```

En conséquence,  MpComputerStatus  indique si Windows Defender est activé ou non.

3\.  Pare- feu basé sur l'hôte : il s'agit d'un outil de sécurité installé et exécuté sur une machine hôte qui peut empêcher et bloquer les tentatives d'attaque des attaquants ou des équipes rouges. Il est donc essentiel d'énumérer et de rassembler des détails sur le pare-feu et ses règles au sein de la machine à laquelle nous avons un accès initial.  

![397c848fb415b72ce235915dea420900](https://github.com/dsgsec/Red-Team/assets/82456829/f0f967f7-7dd2-4874-a3a2-6db4d3420d4d)

L'objectif principal du pare-feu basé sur l'hôte est de contrôler le trafic entrant et sortant qui transite par l'interface de l'appareil. Il protège l'hôte des appareils non fiables qui se trouvent sur le même réseau. Un pare-feu moderne basé sur l'hôte utilise plusieurs niveaux d'analyse du trafic, y compris l'analyse des paquets, lors de l'établissement de la connexion.

Un pare-feu agit comme un contrôle d'accès au niveau de la couche réseau. Il est capable d'autoriser et de refuser les paquets réseau. Par exemple, un pare-feu peut être configuré pour bloquer les paquets ICMP envoyés via la  commande ping depuis d'autres machines du même réseau. Les pare-feu de nouvelle génération peuvent également inspecter d'autres couches OSI, telles que les couches d'application. Par conséquent, il peut détecter et bloquer l'injection SQL et d'autres attaques au niveau de la couche application. 

PowerShell

```
PS C:\Users\thm> Get-NetFirewallProfile | Format-Table Name, Enabled

Name    Enabled
----    -------
Domain     True
Private    True
Public     True
```

Si nous disposons de privilèges d'administrateur sur l'utilisateur actuel avec lequel nous nous sommes connectés, nous essayons de désactiver un ou plusieurs profils de pare-feu à l'aide de la   cmdlet Set-NetFirewallProfile .

PowerShell

```
PS C:\Windows\system32> Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled False
PS C:\Windows\system32> Get-NetFirewallProfile | Format-Table Name, Enabled
---- -------
Domain False
Private False
Public False
```

Nous pouvons également apprendre et vérifier les règles actuelles  du pare-feu , qu'elles autorisent ou refusent le pare-feu.

PowerShell

```
PS C:\Users\thm> Get-NetFirewallRule | select DisplayName, Enabled, Description

DisplayName                                                                  Enabled
-----------                                                                  -------
Virtual Machine Monitoring (DCOM-In)                                           False
Virtual Machine Monitoring (Echo Request - ICMPv4-In)                          False
Virtual Machine Monitoring (Echo Request - ICMPv6-In)                          False
Virtual Machine Monitoring (NB-Session-In)                                     False
Virtual Machine Monitoring (RPC)                                               False
SNMP Trap Service (UDP In)                                                     False
SNMP Trap Service (UDP In)                                                     False
Connected User Experiences and Telemetry                                        True
Delivery Optimization (TCP-In)                                                  True
```

Lors de l'engagement de l'équipe rouge, nous n'avons aucune idée de ce que le pare-feu bloque. Cependant, nous pouvons profiter de certaines applets de commande PowerShell telles que Test-NetConnection  et  TcpClient . Supposons que nous sachions qu'un pare-feu est en place et que nous devons tester la connexion entrante sans outils supplémentaires, nous pouvons alors procéder comme suit : 

PowerShell

```
PS C:\Users\thm> Test-NetConnection -ComputerName 127.0.0.1 -Port 80

ComputerName     : 127.0.0.1
RemoteAddress    : 127.0.0.1
RemotePort       : 80
InterfaceAlias   : Loopback Pseudo-Interface 1
SourceAddress    : 127.0.0.1
TcpTestSucceeded : True

PS C:\Users\thm> (New-Object System.Net.Sockets.TcpClient("127.0.0.1", "80")).Connected
True
```

En conséquence, nous pouvons confirmer que la connexion entrante sur le port 80 est ouverte et autorisée dans le pare-feu. Notez que nous pouvons également tester des cibles distantes dans le même réseau ou nom de domaine en spécifiant dans l'  argument -ComputerName  pour  Test-NetConnection .
