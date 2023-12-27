Dans cette tâche, nous continuerons à discuter des solutions de sécurité des hôtes.

## Journalisation et surveillance des événements de sécurité 

![980c1573a3ec3d309d0f2360c16b019a](https://github.com/dsgsec/Red-Team/assets/82456829/c4aaf0eb-2da0-4694-b72a-41a07edafe6a)

Par défaut, les systèmes d'exploitation enregistrent divers événements d'activité dans le système à l'aide de fichiers journaux. La fonction de journalisation des événements est à la disposition des administrateurs du système informatique et du réseau pour surveiller et analyser les événements importants, que ce soit du côté de l'hôte ou du réseau. Dans les réseaux coopérants, les équipes de sécurité utilisent la technique de journalisation des événements pour suivre et enquêter sur les incidents de sécurité. 

Il existe différentes catégories dans lesquelles le système d'exploitation Windows enregistre les informations sur les événements, notamment l'application, le système, la sécurité, les services, etc. De plus, les périphériques de sécurité et réseau stockent les informations sur les événements dans des fichiers journaux pour permettre aux administrateurs système d'avoir un aperçu de ce qui se passe. en cours.

Nous pouvons obtenir une liste des journaux d'événements disponibles sur la machine locale à l'aide de l'  applet de commande Get-EventLog  .

PowerShell

```
PS C:\Users\thm> Get-EventLog -List

  Max(K) Retain OverflowAction        Entries Log
  ------ ------ --------------        ------- ---
     512      7 OverwriteOlder             59 Active Directory Web Services
  20,480      0 OverwriteAsNeeded         512 Application
     512      0 OverwriteAsNeeded         170 Directory Service
 102,400      0 OverwriteAsNeeded          67 DNS Server
  20,480      0 OverwriteAsNeeded       4,345 System
  15,360      0 OverwriteAsNeeded       1,692 Windows PowerShell
```

Parfois, la liste des journaux d'événements disponibles vous donne un aperçu des applications et des services installés sur la machine ! Par exemple, nous pouvons voir que la machine locale dispose d'Active Directory, d'un serveur DNS , etc. Pour plus d'informations sur l'  applet de commande Get-EventLog  avec des exemples, visitez le  [site Web des documents Microsoft](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-eventlog?view=powershell-5.1) .

Dans les réseaux d'entreprise, le logiciel agent de journalisation est installé sur les clients pour collecter et rassembler les journaux de différents capteurs afin d'analyser et de surveiller les activités au sein du réseau. Nous en discuterons davantage dans la tâche Solution de sécurité réseau.

Moniteur système ( Sysmon )

![8c77acd6d831c3b9f4c5b5f0cdc0d08c](https://github.com/dsgsec/Red-Team/assets/82456829/f2774c0f-58c8-4c5e-a580-c3c40fef142c)

Le système Windows System Monitor   est un service et un pilote de périphérique. C'est l'une des suites Microsoft Sysinternals. L'  outil sysmon  n'est pas un outil essentiel (non installé par défaut), mais il commence à collecter et à enregistrer les événements une fois installé. Ces indicateurs de journaux peuvent aider considérablement les administrateurs système et les blue teamers à suivre et enquêter sur les activités malveillantes et à faciliter le dépannage général.

L'une des fonctionnalités intéressantes de l'  outil sysmon   est qu'il peut enregistrer de nombreux événements importants, et vous pouvez également créer vos propres règles et configurations à surveiller :

-   Création et terminaison du processus
-   Les connexions de réseau
-   Modification au dossier
-   Menaces à distance
-   Accès aux processus et à la mémoire
-   et plein d'autres

Pour en savoir plus sur  sysmon , visitez la page du document Windows [ici](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon) .

En tant que red teamer, l'un des principaux objectifs est de rester indétectable, il est donc essentiel de connaître ces outils et d'éviter de provoquer des événements générateurs et d'alerte. Voici quelques-unes des astuces qui peuvent être utilisées pour détecter si le  système  est disponible ou non sur la machine victime. 

Nous pouvons rechercher un processus ou un service nommé « Sysmon » dans le ou les services actuels comme suit :

PowerShell

```
PS C:\Users\thm> Get-Process | Where-Object { $_.ProcessName -eq "Sysmon" }

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    373      15    20212      31716              3316   0 Sysmon
```

ou recherchez les services comme suit,

PowerShell

```
PS C:\Users\thm> Get-CimInstance win32_service -Filter "Description = 'System Monitor service'"
# or
Get-Service | where-object {$_.DisplayName -like "*sysm*"}
```

Cela peut également être fait en vérifiant le registre Windows 

PowerShell

```
PS C:\Users\thm> reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Channels\Microsoft-Windows-Sysmon/Operational
```

Toutes ces commandes confirment si l'  outil sysmon  est installé. Une fois que nous l'avons détecté, nous pouvons essayer de trouver le fichier de configuration sysmon si nous disposons d'une autorisation de lecture pour comprendre ce que les administrateurs système surveillent.

PowerShell

```
PS C:\Users\thm> findstr /si '<ProcessCreate onmatch="exclude">' C:\tools\*
C:\tools\Sysmon\sysmonconfig.xml:
C:\tools\Sysmon\sysmonconfig.xml:
```

Pour plus de détails sur l'  outil système  Windows et comment l'utiliser dans les points de terminaison, nous vous suggérons d'essayer la salle TryHackMe : [Sysmon](https://tryhackme.com/room/sysmon) .

Système de détection/prévention des intrusions basé sur l'hôte (HIDS/HIPS)

![933d57959af6b3ac9396e688698bb358](https://github.com/dsgsec/Red-Team/assets/82456829/7bd04c76-f200-4055-a67b-439c65dc05b4)


HIDS signifie Système de détection d'intrusion basé sur l'hôte. Il s'agit d'un logiciel capable de surveiller et de détecter les activités anormales et malveillantes chez un hôte. L'objectif principal du HIDS est de détecter les activités suspectes et non de les empêcher. Le système de détection d'intrusion basé sur l'hôte ou sur le réseau fonctionne de deux manières, notamment :

-   IDS basé sur les signatures : il examine les sommes de contrôle et l'authentification des messages.
-   L'IDS basé sur les anomalies recherche les activités inattendues, notamment une utilisation anormale de la bande passante, des protocoles et des ports.

Les systèmes de prévention des intrusions basés sur l'hôte ( HIPS ) fonctionnent en sécurisant les activités du système d'exploitation où il est installé. C'est une solution de détection et de prévention contre les attaques et comportements anormaux notoires. HIPS est capable d'auditer les fichiers journaux de l'hôte, de surveiller les processus et de protéger les ressources système. HIPS est un mélange des meilleures fonctionnalités de produits telles que l'antivirus, l'analyse du comportement, le réseau, le pare-feu d'applications, etc.

Il existe également un IDS/IPS basé sur le réseau, que nous aborderons dans la tâche suivante.

Détection et réponse des points de terminaison ( EDR )

![92a60622a80d64cc6dbedaa6c207662b](https://github.com/dsgsec/Red-Team/assets/82456829/8f7f4cad-09ac-4347-b0d5-fec5dca09af2)

Il est également connu sous le nom de Endpoint Detection and Threat Response (EDTR). L' EDR est une solution de cybersécurité qui protège contre les logiciels malveillants et autres menaces. Les EDR peuvent rechercher des fichiers malveillants, surveiller les événements des points finaux, du système et du réseau et les enregistrer dans une base de données pour une analyse, une détection et une enquête plus approfondies. Les EDR constituent la nouvelle génération d'antivirus et détectent les activités malveillantes sur l'hôte en temps réel.

EDR analyse les données et le comportement du système pour créer des menaces de section, y compris

-   Logiciels malveillants, notamment virus, chevaux de Troie, logiciels publicitaires et enregistreurs de frappe
-   Chaînes d'exploitation
-   Rançongiciel

Vous trouverez ci-dessous quelques logiciels EDR courants pour les points finaux

-   Cylance
-   Grève de foule
-   Symantec
-   SentinelleOne
-   Beaucoup d'autres

Même si un attaquant a réussi à livrer sa charge utile et à contourner EDR lors de la réception du shell inversé, EDR est toujours en cours d'exécution et surveille le système. Cela peut nous empêcher de faire autre chose s'il signale une alerte.

Nous pouvons utiliser des scripts pour énumérer les produits de sécurité au sein de la machine, tels que  [Invoke-EDRChecker](https://github.com/PwnDexter/Invoke-EDRChecker)  et [SharpEDRChecker ](https://github.com/PwnDexter/SharpEDRChecker). Ils vérifient les produits antivirus, EDR et de surveillance de journalisation couramment utilisés en  vérifiant les métadonnées des fichiers, les processus, les DLL chargées dans les processus, services, pilotes et répertoires actuels.
