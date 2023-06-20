Énumération initiale du domaine
===============================

* * * * *

Nous sommes au tout début de notre test de pénétration axé sur AD contre Inlanefreight. Nous avons rassemblé des informations de base et obtenu une image de ce à quoi s'attendre du client via les documents de cadrage.

* * * * *

Mise en place
-------------

Pour cette première partie du test, nous partons sur un hôte d'attaque placé à l'intérieur du réseau pour nous. Il s'agit d'un moyen courant qu'un client peut choisir pour que nous effectuions un test d'intrusion interne. Une liste des types de configurations qu'un client peut choisir pour les tests comprend :

-   Une distribution de test de pénétration (généralement Linux) en tant que machine virtuelle dans leur infrastructure interne qui rappelle un hôte de saut que nous contrôlons via VPN, et dans lequel nous pouvons nous connecter en SSH.
-   Un appareil physique branché sur un port Ethernet qui nous rappelle via VPN, et auquel nous pouvons nous connecter en SSH.
-   Une présence physique à leur bureau avec notre ordinateur portable branché sur un port Ethernet.
-   Une machine virtuelle Linux dans Azure ou AWS avec accès au réseau interne auquel nous pouvons nous connecter via SSH en utilisant l'authentification par clé publique et notre adresse IP publique sur liste blanche.
-   Accès VPN à leur réseau interne (un peu limitant car nous ne pourrons pas effectuer certaines attaques telles que LLMNR/NBT-NS Poisoning).
-   Depuis un ordinateur portable d'entreprise connecté au VPN du client.
-   Sur un poste de travail géré (généralement Windows), physiquement assis dans leur bureau avec un accès Internet limité ou inexistant ou la possibilité d'intégrer des outils. Ils peuvent également choisir cette option, mais vous donner un accès Internet complet, un administrateur local et mettre la protection des terminaux en mode moniteur afin que vous puissiez utiliser les outils à volonté.
-   Sur un VDI (bureau virtuel) accessible à l'aide de Citrix ou similaire, avec l'une des configurations décrites pour le poste de travail géré généralement accessible via VPN à distance ou à partir d'un ordinateur portable d'entreprise.

Ce sont les configurations les plus courantes que j'ai vues, bien qu'un client puisse proposer une autre variante de l'une d'entre elles. Le client peut également choisir entre une approche "boîte grise" où il nous donne juste une liste d'adresses IP/plages de réseaux CIDR dans le champ d'application, ou une "boîte noire" où nous devons nous brancher et faire toutes les découvertes à l'aveugle en utilisant diverses techniques. Enfin, ils peuvent choisir entre évasif, non évasif ou évasif hybride (commencer « silencieux » et de plus en plus fort pour voir à quel seuil nous sommes détectés, puis passer à des tests non évasifs. Ils peuvent également choisir de nous faire commencer par aucune information d'identification ou du point de vue d'un utilisateur de domaine standard.

Notre client Inlanefreight a choisi l'approche suivante car il recherche une évaluation aussi complète que possible. À l'heure actuelle, leur programme de sécurité n'est pas suffisamment mature pour bénéficier de toute forme de test d'évasion ou d'une approche « boîte noire ».

-   Une VM pentest personnalisée au sein de leur réseau interne qui rappelle notre hôte de saut, et nous pouvons nous y connecter en SSH pour effectuer des tests.
-   Ils nous ont également fourni un hôte Windows sur lequel nous pouvons charger des outils si nécessaire.
-   Ils nous ont demandé de commencer à partir d'un point de vue non authentifié, mais nous ont également donné un compte d'utilisateur de domaine standard ( `htb-student`) qui peut être utilisé pour accéder à l'hôte d'attaque Windows.
-   Test "boîte grise". Ils nous ont donné la plage de réseau 172.16.5.0/23 et aucune autre information sur le réseau.
-   Tests non évasifs.

Nous n'avons pas reçu d'informations d'identification ni de carte détaillée du réseau interne.

* * * * *

Tâches
------

Nos tâches à accomplir pour cette section sont :

-   Énumérez le réseau interne, identifiez les hôtes, les services critiques et les pistes potentielles pour prendre pied.
-   Cela peut inclure des mesures actives et passives pour identifier les utilisateurs, les hôtes et les vulnérabilités dont nous pouvons tirer parti pour approfondir notre accès.
-   Documentez toutes les découvertes que nous rencontrons pour une utilisation ultérieure. Extrêmement important!

Nous allons commencer à partir de notre hôte d'attaque Linux sans identifiants d'utilisateur de domaine. C'est une chose courante de commencer un pentest de cette manière. De nombreuses organisations voudront voir ce que vous pouvez faire d'un point de vue aveugle, comme celui-ci, avant de vous fournir des informations supplémentaires pour le test. Cela donne un aperçu plus réaliste des voies potentielles qu'un adversaire devrait utiliser pour infiltrer le domaine. Cela peut les aider à voir ce qu'un attaquant pourrait faire s'il obtient un accès non autorisé via Internet (c'est-à-dire une attaque de phishing), un accès physique au bâtiment, un accès sans fil depuis l'extérieur (si le réseau sans fil touche l'environnement AD), ou même un employé voyou. En fonction du succès de cette phase,

Vous trouverez ci-dessous quelques-uns des points de données clés que nous devrions rechercher en ce moment et noter dans notre outil de prise de notes de choix et enregistrer la sortie de l'analyse/de l'outil dans des fichiers chaque fois que possible.

#### Points de données clés

| Point de données | Description |
| --- | --- |
| `AD Users` | Nous essayons d'énumérer les comptes d'utilisateurs valides que nous pouvons cibler pour la pulvérisation de mot de passe. |
| `AD Joined Computers` | Les ordinateurs clés incluent les contrôleurs de domaine, les serveurs de fichiers, les serveurs SQL, les serveurs Web, les serveurs de messagerie Exchange, les serveurs de base de données, etc. |
| `Key Services` | Kerberos, NetBIOS, LDAP, DNS |
| `Vulnerable Hosts and Services` | Tout ce qui peut être une victoire rapide. (c'est-à-dire un hébergeur facile à exploiter et à prendre pied) |

* * * * *

TTP
---

L'énumération d'un environnement AD peut être écrasante si on l'aborde sans plan. Il y a une abondance de données stockées dans AD, et cela peut prendre beaucoup de temps à passer au crible s'ils ne sont pas examinés par étapes progressives, et nous manquerons probablement des choses. Nous devons établir un plan de match pour nous-mêmes et l'aborder pièce par pièce. Tout le monde travaille de manière légèrement différente, donc à mesure que nous acquérons plus d'expérience, nous commencerons à développer notre propre méthodologie reproductible qui nous convient le mieux. Quelle que soit la façon dont nous procédons, nous commençons généralement au même endroit et recherchons les mêmes points de données. Nous allons expérimenter de nombreux outils dans cette section et les suivantes. Il est important de reproduire chaque exemple et même d'essayer de recréer des exemples avec différents outils pour voir comment ils fonctionnent différemment, apprendre leur syntaxe et trouver quelle approche nous convient le mieux.

Nous commencerons par `passive`identifier tous les hôtes du réseau, puis `active`validerons les résultats pour en savoir plus sur chaque hôte (quels services sont en cours d'exécution, noms, vulnérabilités potentielles, etc.). Une fois que nous savons quels hôtes existent, nous pouvons procéder à sonder ces hôtes, à la recherche de toutes les données intéressantes que nous pouvons en glaner. Après avoir accompli ces tâches, nous devrions nous arrêter et nous regrouper et examiner les informations dont nous disposons. À ce stade, nous espérons disposer d'un ensemble d'informations d'identification ou d'un compte d'utilisateur à cibler pour prendre pied sur un hôte joint à un domaine ou avoir la possibilité de commencer l'énumération des informations d'identification à partir de notre hôte d'attaque Linux.

Examinons quelques outils et techniques pour nous aider dans cette énumération.

### Identification des hôtes

Tout d'abord, prenons le temps d'écouter le réseau et de voir ce qui se passe. Nous pouvons utiliser `Wireshark`et `TCPDump`pour "mettre notre oreille sur le fil" et voir quels hôtes et types de trafic réseau nous pouvons capturer. Ceci est particulièrement utile si l'approche d'évaluation est la « boîte noire ». Nous remarquons certaines requêtes et réponses [ARP , ](https://en.wikipedia.org/wiki/Address_Resolution_Protocol)[MDNS](https://en.wikipedia.org/wiki/Multicast_DNS) , et d'autres paquets de base [de couche deux](https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/topic-map/layer-2-understanding.html) (puisque nous sommes sur un réseau commuté, nous sommes limités au domaine de diffusion actuel), dont certains que nous pouvons voir ci-dessous. C'est un bon début qui nous donne quelques informations sur la configuration du réseau du client.

Faites défiler vers le bas, générez la cible, connectez-vous à l'hôte d'attaque Linux en utilisant `xfreerdp`et lancez Wireshark pour commencer à capturer le trafic.

#### Démarrez Wireshark sur ea-attack01

  Démarrez Wireshark sur ea-attack01

```
┌─[htb-student@ea-attack01]─[~]
└──╼ $sudo -E wireshark

11:28:20.487     Main Warn QStandardPaths: runtime directory '/run/user/1001' is not owned by UID 0, but a directory permissions 0700 owned by UID 1001 GID 1002
<SNIP>

```

#### Sortie Wireshark

![image](https://academy.hackthebox.com/storage/modules/143/ea-wireshark.png)

-   Les paquets ARP nous informent des hôtes : 172.16.5.5, 172.16.5.25 172.16.5.50, 172.16.5.100 et 172.16.5.125.

![image](https://academy.hackthebox.com/storage/modules/143/ea-wireshark-mdns.png)

-   MDNS nous fait connaître l'hébergeur ACADEMY-EA-WEB01.

Si nous sommes sur un hôte sans interface graphique (ce qui est typique), nous pouvons utiliser [tcpdump](https://linux.die.net/man/8/tcpdump) , [net-creds](https://github.com/DanMcInerney/net-creds) et [NetMiner](http://www.netminer.com/main/main-read.do) , etc., pour exécuter les mêmes fonctions. Nous pouvons également utiliser tcpdump pour enregistrer une capture dans un fichier .pcap, la transférer vers un autre hôte et l'ouvrir dans Wireshark.

#### Sortie Tcpdump

  Sortie Tcpdump

```
dsgsec@htb[/htb]$ sudo tcpdump -i ens224

```

![image](https://academy.hackthebox.com/storage/modules/143/tcpdump-example.png)

Il n'y a pas une seule bonne façon d'écouter et de capturer le trafic réseau. Il existe de nombreux outils capables de traiter les données du réseau. Wireshark et tcpdump ne sont que quelques-uns des plus faciles à utiliser et des plus connus. Selon l'hôte sur lequel vous vous trouvez, vous disposez peut-être déjà d'un outil de surveillance du réseau intégré, tel que , `pktmon.exe`qui a été ajouté à toutes les éditions de Windows 10. Remarque pour les tests, c'est toujours une bonne idée d'enregistrer le trafic PCAP que vous capture. Vous pouvez le revoir plus tard pour rechercher plus d'indices, et cela constitue d'excellentes informations supplémentaires à inclure lors de la rédaction de vos rapports.

Notre premier regard sur le trafic réseau nous a indiqué quelques hôtes via `MDNS`et `ARP`. Utilisons maintenant un outil appelé `Responder`pour analyser le trafic réseau et déterminer si quelque chose d'autre dans le domaine apparaît.

[Responder](https://github.com/lgandx/Responder-Windows) est un outil conçu pour écouter, analyser et empoisonner `LLMNR`les demandes `NBT-NS`et `MDNS`les réponses. Il a beaucoup plus de fonctions, mais pour l'instant, tout ce que nous utilisons est l'outil en mode Analyse. Cela écoutera passivement le réseau et n'enverra aucun paquet empoisonné. Nous couvrirons cet outil plus en profondeur dans les sections suivantes.

#### Premier répondant

Code : bash

```
sudo responder -I ens224 -A

```

#### Résultats du répondeur

![image](https://academy.hackthebox.com/storage/modules/143/responder-example.gif)

Lorsque nous démarrons Responder avec le mode d'analyse passive activé, nous verrons les demandes circuler dans notre session. Notez ci-dessous que nous avons trouvé quelques hôtes uniques non mentionnés précédemment dans nos captures Wireshark. Cela vaut la peine de les noter car nous commençons à construire une belle liste cible d'adresses IP et de noms d'hôte DNS.

Nos vérifications passives nous ont donné quelques hôtes à noter pour une énumération plus approfondie. Effectuons maintenant quelques vérifications actives en commençant par un rapide balayage ICMP du sous-réseau à l'aide de `fping`.

[Fping](https://fping.org/) nous offre une capacité similaire à l'application ping standard en ce sens qu'elle utilise les requêtes et les réponses ICMP pour atteindre et interagir avec un hôte. Là où fping brille, c'est dans sa capacité à émettre des paquets ICMP contre une liste de plusieurs hôtes à la fois et sa capacité de script. En outre, il fonctionne de manière circulaire, interrogeant les hôtes de manière cyclique au lieu d'attendre le retour de plusieurs demandes à un seul hôte avant de continuer. Ces vérifications nous aideront à déterminer si quelque chose d'autre est actif sur le réseau interne. ICMP n'est pas un guichet unique, mais c'est un moyen facile d'avoir une première idée de ce qui existe. D'autres ports ouverts et protocoles actifs peuvent pointer vers de nouveaux hôtes pour un ciblage ultérieur. Voyons-le en action.

#### Contrôles actifs FPing

Ici, nous allons commencer `fping`par quelques drapeaux : `a`pour afficher les cibles actives, `s`pour imprimer les statistiques à la fin de l'analyse, `g`pour générer une liste de cibles à partir du réseau CIDR et `q`pour ne pas afficher les résultats par cible.

  Contrôles actifs FPing

```
dsgsec@htb[/htb]$ fping -asgq 172.16.5.0/23

172.16.5.5
172.16.5.25
172.16.5.50
172.16.5.100
172.16.5.125
172.16.5.200
172.16.5.225
172.16.5.238
172.16.5.240

     510 targets
       9 alive
     501 unreachable
       0 unknown addresses

    2004 timeouts (waiting for response)
    2013 ICMP Echos sent
       9 ICMP Echo Replies received
    2004 other ICMP received

 0.029 ms (min round trip time)
 0.396 ms (avg round trip time)
 0.799 ms (max round trip time)
       15.366 sec (elapsed real time)

```

La commande ci-dessus valide quels hôtes sont actifs sur le `/23`réseau et le fait discrètement au lieu de spammer le terminal avec des résultats pour chaque adresse IP de la liste cible. Nous pouvons combiner les résultats réussis et les informations que nous avons glanées lors de nos vérifications passives dans une liste pour une analyse plus détaillée avec Nmap. À partir de la `fping`commande, nous pouvons voir 9 "hôtes en direct", y compris notre hôte d'attaque.

Remarque : Les résultats de l'analyse dans le réseau cible seront différents de la sortie de la commande dans cette section en raison de la taille du réseau du laboratoire. Il est toujours utile de reproduire chaque exemple pour s'exercer au fonctionnement de ces outils et noter chaque hôte en direct dans ce laboratoire.

#### Numérisation Nmap

Maintenant que nous avons une liste d'hôtes actifs au sein de notre réseau, nous pouvons énumérer davantage ces hôtes. Nous cherchons à déterminer quels services chaque hôte exécute, à identifier les hôtes critiques tels que `Domain Controllers`et `web servers`, et à identifier les hôtes potentiellement vulnérables à sonder ultérieurement. Avec notre concentration sur AD, après avoir fait un large balayage, il serait sage de nous concentrer sur les protocoles standard généralement vus accompagnant les services AD, tels que DNS, SMB, LDAP et Kerberos pour n'en nommer que quelques-uns. Vous trouverez ci-dessous un exemple rapide d'un scan Nmap simple.

Code : bash

```
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum

```

L' analyse [-A (options d'analyse agressives)](https://nmap.org/book/man-misc-options.html) exécutera plusieurs fonctions. L'un des plus importants est une énumération rapide des ports bien connus pour inclure les services Web, les services de domaine, etc. Pour notre fichier hosts.txt, certains de nos résultats de Responder et fping se chevauchent (nous avons trouvé le nom et l'adresse IP), donc pour faire simple, seule l'adresse IP a été introduite dans hosts.txt pour l'analyse.

#### Faits saillants des résultats NMAP

  Faits saillants des résultats NMAP

```
Nmap scan report for inlanefreight.local (172.16.5.5)
Host is up (0.069s latency).
Not shown: 987 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-04-04 15:12:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
|_ssl-date: 2022-04-04T15:12:53+00:00; -1s from scanner time.
| ssl-cert: Subject:
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
<SNIP>
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-DC01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2022-04-04T15:12:45+00:00
<SNIP>
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: ACADEMY-EA-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

```

Nos analyses nous ont fourni la norme de nommage utilisée par NetBIOS et DNS, nous pouvons voir que certains hôtes ont RDP ouvert, et ils nous ont orientés vers le principal `Domain Controller`pour le domaine INLANEFREIGHT.LOCAL (ACADEMY-EA-DC01.INLANEFREIGHT. LOCAL). Les résultats ci-dessous montrent des résultats intéressants concernant un hôte peut-être obsolète (pas dans notre laboratoire actuel).

  Faits saillants des résultats NMAP

```
dsgsec@htb[/htb]$ nmap -A 172.16.5.100

Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 13:42 EDT
Nmap scan report for 172.16.5.100
Host is up (0.071s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/7.5
| http-methods:
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  https?
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7600 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2008 R2 10.50.1600.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-04-08T17:38:25
|_Not valid after:  2052-04-08T17:38:25
|_ssl-date: 2022-04-08T17:43:53+00:00; 0s from scanner time.
| ms-sql-ntlm-info:
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-CTX1
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-CTX1.INLANEFREIGHT.LOCAL
|_  Product_Version: 6.1.7600
Host script results:
| smb2-security-mode:
|   2.1:
|_    Message signing enabled but not required
| ms-sql-info:
|   172.16.5.100:1433:
|     Version:
|       name: Microsoft SQL Server 2008 R2 RTM
|       number: 10.50.1600.00
|       Product: Microsoft SQL Server 2008 R2
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_nbstat: NetBIOS name: ACADEMY-EA-CTX1, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:c7:1c (VMware)
| smb-os-discovery:
|   OS: Windows Server 2008 R2 Standard 7600 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::-
|   Computer name: ACADEMY-EA-CTX1
|   NetBIOS computer name: ACADEMY-EA-CTX1\x00
|   Domain name: INLANEFREIGHT.LOCAL
|   Forest name: INLANEFREIGHT.LOCAL
|   FQDN: ACADEMY-EA-CTX1.INLANEFREIGHT.LOCAL
|_  System time: 2022-04-08T10:43:48-07:00

<SNIP>

```

Nous pouvons voir à partir de la sortie ci-dessus que nous avons un hôte potentiel exécutant un système d'exploitation obsolète (Windows 7, 8 ou Server 2008 basé sur la sortie). Cela nous intéresse car cela signifie que des systèmes d'exploitation hérités s'exécutent dans cet environnement AD. Cela signifie également qu'il est possible que des exploits plus anciens comme EternalBlue, MS08-067 et d'autres fonctionnent et nous fournissent un shell de niveau SYSTEM. Aussi étrange que cela puisse paraître d'avoir des hôtes exécutant des logiciels hérités ou des systèmes d'exploitation en fin de vie, cela reste courant dans les environnements des grandes entreprises. Vous aurez souvent des processus ou des équipements tels qu'une chaîne de production ou le CVC construit sur l'ancien système d'exploitation et en place depuis longtemps. La mise hors ligne d'équipements comme celui-ci est coûteuse et peut nuire à une organisation, de sorte que les hôtes hérités sont souvent laissés en place. Ils essaieront probablement de construire une enveloppe extérieure rigide de pare-feu, d'IDS/IPS et d'autres solutions de surveillance et de protection autour de ces systèmes. Si vous pouvez vous y retrouver, c'est un gros problème et cela peut être une prise de pied rapide et facile. Avant d'exploiter des systèmes hérités, cependant, nous devons alerter notre client et obtenir son approbation par écrit au cas où une attaque entraînerait une instabilité du système ou entraînerait l'arrêt d'un service ou de l'hôte. Ils peuvent préférer que nous nous contentions d'observer, de signaler et d'avancer sans exploiter activement le système. nous devons alerter notre client et obtenir son approbation par écrit au cas où une attaque entraînerait une instabilité du système ou entraînerait l'arrêt d'un service ou de l'hôte. Ils peuvent préférer que nous nous contentions d'observer, de signaler et d'avancer sans exploiter activement le système. nous devons alerter notre client et obtenir son approbation par écrit au cas où une attaque entraînerait une instabilité du système ou entraînerait l'arrêt d'un service ou de l'hôte. Ils peuvent préférer que nous nous contentions d'observer, de signaler et d'avancer sans exploiter activement le système.

Les résultats de ces analyses nous indiqueront où nous commencerons à rechercher des pistes potentielles d'énumération de domaine, pas seulement l'analyse de l'hôte. Nous devons trouver notre chemin vers un compte d'utilisateur de domaine. En regardant nos résultats, nous avons trouvé plusieurs serveurs qui hébergent des services de domaine (DC01, MX01, WS01, etc.). Maintenant que nous savons ce qui existe et quels services sont en cours d'exécution, nous pouvons interroger ces serveurs et tenter d'énumérer les utilisateurs. Assurez-vous d'utiliser l' `-oA`indicateur comme meilleure pratique lors de l'exécution d'analyses Nmap. Cela garantira que nous avons nos résultats d'analyse dans plusieurs formats à des fins de journalisation et dans des formats pouvant être manipulés et introduits dans d'autres outils.

Nous devons être conscients des analyses que nous exécutons et de leur fonctionnement. Certaines des analyses scriptées Nmap exécutent des vérifications de vulnérabilité actives contre un hôte qui pourrait provoquer une instabilité du système ou le mettre hors ligne, causant des problèmes pour le client ou pire. Par exemple, l'exécution d'une vaste analyse de découverte sur un réseau avec des dispositifs tels que des capteurs ou des contrôleurs logiques pourrait potentiellement les surcharger et perturber l'équipement industriel du client, entraînant une perte de produit ou de capacité. Prenez le temps de comprendre les analyses que vous utilisez avant de les exécuter dans l'environnement d'un client.

Nous reviendrons très probablement sur ces résultats plus tard pour une énumération plus approfondie, alors ne les oubliez pas. Nous devons trouver notre chemin vers un compte d'utilisateur de domaine ou `SYSTEM`un accès de niveau sur un hôte joint à un domaine afin que nous puissions prendre pied et commencer le vrai plaisir. Plongeons dans la recherche d'un compte utilisateur.

* * * * *

Identification des utilisateurs
-------------------------------

Si notre client ne nous fournit pas d'utilisateur avec lequel commencer les tests (ce qui est souvent le cas), nous devrons trouver un moyen de prendre pied dans le domaine en obtenant des informations d'identification en texte clair ou un hachage de mot de passe NTLM pour un utilisateur. , un shell SYSTEM sur un hôte joint à un domaine ou un shell dans le contexte d'un compte d'utilisateur de domaine. L'obtention d'un utilisateur valide avec des informations d'identification est essentielle dans les premières étapes d'un test d'intrusion interne. Cet accès (même au niveau le plus bas) ouvre de nombreuses opportunités pour effectuer des énumérations et même des attaques. Examinons une façon de commencer à rassembler une liste d'utilisateurs valides dans un domaine à utiliser plus tard dans notre évaluation.

### Kerbrute - Énumération interne du nom d'utilisateur AD

[Kerbrute](https://github.com/ropnop/kerbrute) peut être une option plus furtive pour l'énumération des comptes de domaine. Il tire parti du fait que les échecs de pré-authentification Kerberos ne déclenchent souvent pas de journaux ou d'alertes. Nous utiliserons Kerbrute en conjonction avec les listes d'utilisateurs `jsmith.txt`ou d [' Insidetrust](https://github.com/insidetrust/statistically-likely-usernames) . Ce référentiel contient de nombreuses listes d'utilisateurs différentes qui peuvent être extrêmement utiles lors de la tentative d'énumération des utilisateurs lors du démarrage à partir d'une perspective non authentifiée. Nous pouvons pointer Kerbrute vers le DC que nous avons trouvé plus tôt et lui fournir une liste de mots. L'outil est rapide et nous obtiendrons des résultats nous indiquant si les comptes trouvés sont valides ou non, ce qui est un excellent point de départ pour lancer des attaques telles que la pulvérisation de mots de passe, que nous aborderons en détail plus tard dans ce module.`jsmith2.txt`[](https://github.com/insidetrust/statistically-likely-usernames)

Pour commencer avec Kerbrute, nous pouvons télécharger [des binaires précompilés](https://github.com/ropnop/kerbrute/releases/latest) pour l'outil de test à partir de Linux, Windows et Mac, ou nous pouvons le compiler nous-mêmes. Il s'agit généralement de la meilleure pratique pour tout outil que nous introduisons dans un environnement client. Pour compiler les binaires à utiliser sur le système de notre choix, nous clonons d'abord le dépôt :

#### Cloner le dépôt Kerbrute GitHub

  Cloner le dépôt Kerbrute GitHub

```
dsgsec@htb[/htb]$ sudo git clone https://github.com/ropnop/kerbrute.git

Cloning into 'kerbrute'...
remote: Enumerating objects: 845, done.
remote: Counting objects: 100% (47/47), done.
remote: Compressing objects: 100% (36/36), done.
remote: Total 845 (delta 18), reused 28 (delta 10), pack-reused 798
Receiving objects: 100% (845/845), 419.70 KiB | 2.72 MiB/s, done.
Resolving deltas: 100% (371/371), done.

```

Taper `make help`nous montrera les options de compilation disponibles.

#### Liste des options de compilation

  Liste des options de compilation

```
dsgsec@htb[/htb]$ make help

help:            Show this help.
windows:  Make Windows x86 and x64 Binaries
linux:  Make Linux x86 and x64 Binaries
mac:  Make Darwin (Mac) x86 and x64 Binaries
clean:  Delete any binaries
all:  Make Windows, Linux and Mac x86/x64 Binaries

```

Nous pouvons choisir de compiler un seul binaire ou type `make all`et d'en compiler un chacun pour une utilisation sur les systèmes Linux, Windows et Mac (une version x86 et x64 pour chacun).

#### Compilation pour plusieurs plates-formes et architectures

  Compilation pour plusieurs plates-formes et architectures

```
dsgsec@htb[/htb]$ sudo make all

go: downloading github.com/spf13/cobra v1.1.1
go: downloading github.com/op/go-logging v0.0.0-20160315200505-970db520ece7
go: downloading github.com/ropnop/gokrb5/v8 v8.0.0-20201111231119-729746023c02
go: downloading github.com/spf13/pflag v1.0.5
go: downloading github.com/jcmturner/gofork v1.0.0
go: downloading github.com/hashicorp/go-uuid v1.0.2
go: downloading golang.org/x/crypto v0.0.0-20201016220609-9e8e0b390897
go: downloading github.com/jcmturner/rpc/v2 v2.0.2
go: downloading github.com/jcmturner/dnsutils/v2 v2.0.0
go: downloading github.com/jcmturner/aescts/v2 v2.0.0
go: downloading golang.org/x/net v0.0.0-20200114155413-6afb5195e5aa
cd /tmp/kerbrute
rm -f kerbrute kerbrute.exe kerbrute kerbrute.exe kerbrute.test kerbrute.test.exe kerbrute.test kerbrute.test.exe main main.exe
rm -f /root/go/bin/kerbrute
Done.
Building for windows amd64..

<SNIP>

```

Le répertoire nouvellement créé `dist`contiendra nos binaires compilés.

#### Lister les binaires compilés dans dist

  Lister les binaires compilés dans dist

```
dsgsec@htb[/htb]$ ls dist/

kerbrute_darwin_amd64  kerbrute_linux_386  kerbrute_linux_amd64  kerbrute_windows_386.exe  kerbrute_windows_amd64.exe

```

Nous pouvons ensuite tester le binaire pour nous assurer qu'il fonctionne correctement. Nous utiliserons la version x64 sur l'hôte d'attaque Parrot Linux fourni dans l'environnement cible.

#### Test du binaire kerbrute_linux_amd64

  Test du binaire kerbrute_linux_amd64

```
dsgsec@htb[/htb]$ ./kerbrute_linux_amd64

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _\
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

This tool is designed to assist in quickly bruteforcing valid Active Directory accounts through Kerberos Pre-Authentication.
It is designed to be used on an internal Windows domain with access to one of the Domain Controllers.
Warning: failed Kerberos Pre-Auth counts as a failed login and WILL lock out accounts

Usage:
  kerbrute [command]

  <SNIP>

```

Nous pouvons ajouter l'outil à notre PATH pour le rendre facilement accessible depuis n'importe où sur l'hôte.

#### Ajout de l'outil à notre chemin

  Ajout de l'outil à notre chemin

```
dsgsec@htb[/htb]$ echo $PATH
/home/htb-student/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/share/games:/usr/local/sbin:/usr/sbin:/sbin:/snap/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/home/htb-student/.dotnet/tools

```

#### Déplacer le binaire

  Déplacer le binaire

```
dsgsec@htb[/htb]$ sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute

```

Nous pouvons maintenant taper à `kerbrute`partir de n'importe quel emplacement sur le système et pourrons accéder à l'outil. N'hésitez pas à suivre votre système et à pratiquer les étapes ci-dessus. Passons maintenant en revue un exemple d'utilisation de l'outil pour rassembler une liste initiale de noms d'utilisateur.

#### Énumération des utilisateurs avec Kerbrute

  Énumération des utilisateurs avec Kerbrute

```
dsgsec@htb[/htb]$ kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users

2021/11/17 23:01:46 >  Using KDC(s):
2021/11/17 23:01:46 >   172.16.5.5:88
2021/11/17 23:01:46 >  [+] VALID USERNAME:       jjones@INLANEFREIGHT.LOCAL
2021/11/17 23:01:46 >  [+] VALID USERNAME:       sbrown@INLANEFREIGHT.LOCAL
2021/11/17 23:01:46 >  [+] VALID USERNAME:       tjohnson@INLANEFREIGHT.LOCAL
2021/11/17 23:01:50 >  [+] VALID USERNAME:       evalentin@INLANEFREIGHT.LOCAL

 <SNIP>

2021/11/17 23:01:51 >  [+] VALID USERNAME:       sgage@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       jshay@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       jhermann@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       whouse@INLANEFREIGHT.LOCAL
2021/11/17 23:01:51 >  [+] VALID USERNAME:       emercer@INLANEFREIGHT.LOCAL
2021/11/17 23:01:52 >  [+] VALID USERNAME:       wshepherd@INLANEFREIGHT.LOCAL
2021/11/17 23:01:56 >  Done! Tested 48705 usernames (56 valid) in 9.940 seconds

```

Nous pouvons voir à partir de notre sortie que nous avons validé 56 utilisateurs dans le domaine INLANEFREIGHT.LOCAL et cela n'a pris que quelques secondes pour le faire. Nous pouvons maintenant prendre ces résultats et créer une liste à utiliser dans les attaques ciblées par pulvérisation de mot de passe.

* * * * *

Identification des vulnérabilités potentielles
----------------------------------------------

Le compte [système local](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account)`NT AUTHORITY\SYSTEM` est un compte intégré dans les systèmes d'exploitation Windows. Il a le plus haut niveau d'accès dans le système d'exploitation et est utilisé pour exécuter la plupart des services Windows. Il est également très courant que des services tiers s'exécutent par défaut dans le contexte de ce compte. Un `SYSTEM`compte sur un `domain-joined`hôte pourra énumérer Active Directory en se faisant passer pour le compte d'ordinateur, qui est essentiellement un autre type de compte d'utilisateur. Avoir un accès au niveau SYSTEM dans un environnement de domaine équivaut presque à avoir un compte d'utilisateur de domaine.

Il existe plusieurs façons d'obtenir un accès au niveau SYSTEM sur un hôte, y compris, mais sans s'y limiter :

-   Exploits Windows distants tels que MS08-067, EternalBlue ou BlueKeep.
-   Abus d'un service exécuté dans le contexte de `SYSTEM account`, ou abus des `SeImpersonate`privilèges du compte de service à l'aide de [Juicy Potato](https://github.com/ohpe/juicy-potato) . Ce type d'attaque est possible sur les anciens systèmes d'exploitation Windows, mais pas toujours possible avec Windows Server 2019.
-   Failles d'escalade de privilèges locaux dans les systèmes d'exploitation Windows tels que le planificateur de tâches Windows 10 0-day.
-   Obtenir un accès administrateur sur un hôte joint à un domaine avec un compte local et utiliser Psexec pour lancer une fenêtre SYSTEM cmd

En obtenant un accès au niveau SYSTEM sur un hôte joint à un domaine, vous pourrez effectuer des actions telles que, mais sans s'y limiter :

-   Énumérez le domaine à l'aide d'outils intégrés ou d'outils offensifs tels que BloodHound et PowerView.
-   Effectuez des attaques Kerberoasting / ASREPRoasting dans le même domaine.
-   Exécutez des outils tels que Inveigh pour collecter les hachages Net-NTLMv2 ou effectuer des attaques de relais SMB.
-   Effectuez une emprunt d'identité de jeton pour détourner un compte d'utilisateur de domaine privilégié.
-   Effectuez des attaques ACL.

* * * * *

Un mot d'avertissement
----------------------

Gardez à l'esprit la portée et le style du test lorsque vous choisissez un outil à utiliser. Si vous effectuez un test d'intrusion non évasif, avec tout à l'air libre et que le personnel du client sait que vous êtes là, peu importe généralement la quantité de bruit que vous faites. Cependant, lors d'un test d'intrusion évasif, d'une évaluation contradictoire ou d'un engagement d'équipe rouge, vous essayez d'imiter les outils, tactiques et procédures d'un attaquant potentiel. Dans cet esprit, `stealth`est préoccupant. Lancer Nmap sur un réseau entier n'est pas exactement silencieux, et de nombreux outils que nous utilisons couramment lors d'un test d'intrusion déclencheront des alarmes pour un SOC ou un Blue Teamer formé et préparé. Assurez-vous toujours de clarifier l'objectif de votre évaluation avec le client par écrit avant qu'elle ne commence.

* * * * *

Trouvons un utilisateur
-----------------------

Dans les quelques sections suivantes, nous allons rechercher un compte d'utilisateur de domaine en utilisant des techniques telles que l'empoisonnement LLMNR/NBT-NS et la pulvérisation de mots de passe. Ces attaques sont d'excellents moyens de prendre pied mais doivent être exercées avec prudence et une compréhension des outils et des techniques. Maintenant, traquons un compte d'utilisateur afin que nous puissions passer à la phase suivante de notre évaluation et commencer à séparer le domaine pièce par pièce et à creuser profondément pour une multitude de mauvaises configurations et de défauts.
