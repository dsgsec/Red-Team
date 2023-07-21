Pare-feu et contournement IDS/IPS
=================================

* * * * *

`Nmap`nous donne de nombreuses façons différentes de contourner les règles de pare-feu et IDS/IPS. Ces méthodes incluent la fragmentation des paquets, l'utilisation de leurres et d'autres que nous aborderons dans cette section.

* * * * *

Pare-feu
--------

Un pare-feu est une mesure de sécurité contre les tentatives de connexion non autorisées à partir de réseaux externes. Chaque système de sécurité pare-feu est basé sur un composant logiciel qui surveille le trafic réseau entre le pare-feu et les connexions de données entrantes et décide comment gérer la connexion en fonction des règles qui ont été définies. Il vérifie si des paquets réseau individuels sont transmis, ignorés ou bloqués. Ce mécanisme est conçu pour empêcher les connexions indésirables qui pourraient être potentiellement dangereuses.

* * * * *

IDS/IPS
-------

Comme le pare-feu, le système de détection d'intrusion ( `IDS`) et le système de prévention d'intrusion ( `IPS`) sont également des composants logiciels. `IDS`analyse le réseau à la recherche d'attaques potentielles, les analyse et signale toute attaque détectée. `IPS`complète `IDS`en prenant des mesures défensives spécifiques si une attaque potentielle aurait dû être détectée. L'analyse de telles attaques est basée sur le pattern matching et les signatures. Si des modèles spécifiques sont détectés, comme une analyse de détection de service, `IPS`cela peut empêcher les tentatives de connexion en attente.

* * * * *

#### Déterminer les pare-feu et leurs règles

Nous savons déjà que lorsqu'un port est affiché comme filtré, cela peut avoir plusieurs raisons. Dans la plupart des cas, les pare-feu ont défini certaines règles pour gérer des connexions spécifiques. Les paquets peuvent être soit `dropped`, soit `rejected`. Les `dropped`paquets sont ignorés et aucune réponse n'est renvoyée par l'hôte.

Ceci est différent pour `rejected`les paquets renvoyés avec un `RST`indicateur. Ces paquets contiennent différents types de codes d'erreur ICMP ou ne contiennent rien du tout.

Ces erreurs peuvent être :

-   Réseau inaccessible
-   Net Interdit
-   Hôte inaccessible
-   Hôte interdit
-   Port inaccessible
-   Proto inaccessible

La méthode d'analyse TCP ACK ( ) de Nmap `-sA`est beaucoup plus difficile à filtrer pour les pare-feux et les systèmes IDS/IPS que `-sS`les analyses SYN ( ) ou Connect ( `sT`) classiques car elles n'envoient qu'un paquet TCP avec uniquement le `ACK`drapeau. Lorsqu'un port est fermé ou ouvert, l'hôte doit répondre par un `RST`indicateur. Contrairement aux connexions sortantes, toutes les tentatives de connexion (avec le `SYN`drapeau) à partir de réseaux externes sont généralement bloquées par des pare-feu. Cependant, les paquets avec l' `ACK`indicateur sont souvent passés par le pare-feu car le pare-feu ne peut pas déterminer si la connexion a d'abord été établie à partir du réseau externe ou du réseau interne.

Si nous regardons ces scans, nous verrons comment les résultats diffèrent.

#### SYN-Scan

  SYN-Scan

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:56 CEST
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:22 S ttl=53 id=22412 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:25 S ttl=50 id=62291 iplen=44  seq=4092255222 win=1024 <mss 1460>
SENT (0.0278s) TCP 10.10.14.2:57347 > 10.129.2.28:21 S ttl=58 id=38696 iplen=44  seq=4092255222 win=1024 <mss 1460>
RCVD (0.0329s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=40884 iplen=72 ]
RCVD (0.0341s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
RCVD (1.0386s) TCP 10.129.2.28:22 > 10.10.14.2:57347 SA ttl=64 id=0 iplen=44  seq=1153454414 win=64240 <mss 1460>
SENT (1.1366s) TCP 10.10.14.2:57348 > 10.129.2.28:25 S ttl=44 id=6796 iplen=44  seq=4092320759 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.0053s latency).

PORT   STATE    SERVICE
21/tcp filtered ftp
22/tcp open     ssh
25/tcp filtered smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds

```

#### ACK-Scan

  ACK-Scan

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 14:57 CEST
SENT (0.0422s) TCP 10.10.14.2:49343 > 10.129.2.28:21 A ttl=49 id=12381 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:22 A ttl=41 id=5146 iplen=40  seq=0 win=1024
SENT (0.0423s) TCP 10.10.14.2:49343 > 10.129.2.28:25 A ttl=49 id=5800 iplen=40  seq=0 win=1024
RCVD (0.1252s) ICMP [10.129.2.28 > 10.10.14.2 Port 21 unreachable (type=3/code=3) ] IP [ttl=64 id=55628 iplen=68 ]
RCVD (0.1268s) TCP 10.129.2.28:22 > 10.10.14.2:49343 R ttl=64 id=0 iplen=40  seq=1660784500 win=0
SENT (1.3837s) TCP 10.10.14.2:49344 > 10.129.2.28:25 A ttl=59 id=21915 iplen=40  seq=0 win=1024
Nmap scan report for 10.129.2.28
Host is up (0.083s latency).

PORT   STATE      SERVICE
21/tcp filtered   ftp
22/tcp unfiltered ssh
25/tcp filtered   smtp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds

```

| Options de numérisation | Description |
| --- | --- |
| `10.129.2.28` | Analyse la cible spécifiée. |
| `-p 21,22,25` | Analyse uniquement les ports spécifiés. |
| `-sS` | Effectue une analyse SYN sur les ports spécifiés. |
| `-sA` | Effectue une analyse ACK sur les ports spécifiés. |
| `-Pn` | Désactive les demandes d'écho ICMP. |
| `-n` | Désactive la résolution DNS. |
| `--disable-arp-ping` | Désactive le ping ARP. |
| `--packet-trace` | Affiche tous les paquets envoyés et reçus. |

Veuillez prêter attention aux paquets RCVD et à son drapeau défini que nous recevons de notre cible. Avec le scan SYN ( `-sS`) notre cible essaie d'établir la connexion TCP en renvoyant un paquet avec les `SA`drapeaux SYN-ACK ( ) définis et avec le scan ACK ( `-sA`) nous obtenons le `RST`drapeau car le port TCP 22 est ouvert. Pour le port TCP 25, nous ne recevons aucun paquet en retour, ce qui indique que les paquets seront abandonnés.

* * * * *

Détecter IDS/IPS
----------------

Contrairement aux pare-feu et à leurs règles, la détection des systèmes IDS/IPS est beaucoup plus difficile car ce sont des systèmes passifs de surveillance du trafic. `IDS systems`examiner toutes les connexions entre les hôtes. Si l'IDS trouve des paquets contenant le contenu ou les spécifications définis, l'administrateur en est averti et prend les mesures appropriées dans le pire des cas.

`IPS systems`prendre des mesures configurées par l'administrateur indépendamment pour prévenir automatiquement les attaques potentielles. Il est essentiel de savoir qu'IDS et IPS sont des applications différentes et qu'IPS sert de complément à IDS.

Plusieurs serveurs privés virtuels ( `VPS`) avec différentes adresses IP sont recommandés pour déterminer si de tels systèmes se trouvent sur le réseau cible lors d'un test d'intrusion. Si l'administrateur détecte une telle attaque potentielle sur le réseau cible, la première étape consiste à bloquer l'adresse IP d'où provient l'attaque potentielle. En conséquence, nous ne pourrons plus accéder au réseau en utilisant cette adresse IP, et notre fournisseur de services Internet ( `ISP`) sera contacté et bloqué de tout accès à Internet.

-   `IDS systems`seuls sont généralement là pour aider les administrateurs à détecter les attaques potentielles sur leur réseau. Ils peuvent alors décider comment gérer ces connexions. Nous pouvons déclencher certaines mesures de sécurité de la part d'un administrateur, par exemple en analysant de manière agressive un seul port et son service. Selon que des mesures de sécurité spécifiques sont prises, nous pouvons détecter si le réseau dispose ou non d'applications de surveillance.

-   Une méthode pour déterminer si tel `IPS system`est présent dans le réseau cible consiste à analyser à partir d'un seul hôte ( `VPS`). Si à tout moment cet hôte est bloqué et n'a pas accès au réseau cible, nous savons que l'administrateur a pris des mesures de sécurité. En conséquence, nous pouvons poursuivre notre test d'intrusion avec un autre `VPS`.

Par conséquent, nous savons que nous devons être plus silencieux avec nos scans et, dans le meilleur des cas, masquer toutes les interactions avec le réseau cible et ses services.

* * * * *

Leurres
-------

Il existe des cas dans lesquels les administrateurs bloquent en principe des sous-réseaux spécifiques de différentes régions. Cela empêche tout accès au réseau cible. Un autre exemple est quand IPS devrait nous bloquer. Pour cette raison, la méthode de balayage Leurre ( `-D`) est le bon choix. Avec cette méthode, Nmap génère diverses adresses IP aléatoires insérées dans l'en-tête IP pour dissimuler l'origine du paquet envoyé. Avec cette méthode, nous pouvons générer aléatoirement ( `RND`) un nombre spécifique (par exemple : `5`) d'adresses IP séparées par deux-points (`:`). Notre véritable adresse IP est ensuite placée au hasard entre les adresses IP générées. Dans l'exemple suivant, notre adresse IP réelle est donc placée en deuxième position. Un autre point critique est que les leurres doivent être vivants. Sinon, le service sur la cible peut être inaccessible en raison des mécanismes de sécurité SYN-flood.

#### Numérisation à l'aide de leurres

  Numérisation à l'aide de leurres

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 16:14 CEST
SENT (0.0378s) TCP 102.52.161.59:59289 > 10.129.2.28:80 S ttl=42 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0378s) TCP 10.10.14.2:59289 > 10.129.2.28:80 S ttl=59 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 210.120.38.29:59289 > 10.129.2.28:80 S ttl=37 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 191.6.64.171:59289 > 10.129.2.28:80 S ttl=38 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 184.178.194.209:59289 > 10.129.2.28:80 S ttl=39 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
SENT (0.0379s) TCP 43.21.121.33:59289 > 10.129.2.28:80 S ttl=55 id=29822 iplen=44  seq=3687542010 win=1024 <mss 1460>
RCVD (0.1370s) TCP 10.129.2.28:80 > 10.10.14.2:59289 SA ttl=64 id=0 iplen=44  seq=4056111701 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.099s latency).

PORT   STATE SERVICE
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.15 seconds

```

| Options de numérisation | Description |
| --- | --- |
| `10.129.2.28` | Analyse la cible spécifiée. |
| `-p 80` | Analyse uniquement les ports spécifiés. |
| `-sS` | Effectue une analyse SYN sur les ports spécifiés. |
| `-Pn` | Désactive les demandes d'écho ICMP. |
| `-n` | Désactive la résolution DNS. |
| `--disable-arp-ping` | Désactive le ping ARP. |
| `--packet-trace` | Affiche tous les paquets envoyés et reçus. |
| `-D RND:5` | Génère cinq adresses IP aléatoires qui indiquent l'adresse IP source d'où provient la connexion. |

Les paquets usurpés sont souvent filtrés par les FAI et les routeurs, même s'ils proviennent de la même plage de réseaux. Par conséquent, nous pouvons également spécifier les adresses IP de nos serveurs VPS et les utiliser en combinaison avec la `IP ID`manipulation " " dans les en-têtes IP pour scanner la cible.

Un autre scénario serait que seuls des sous-réseaux individuels n'aient pas accès aux services spécifiques du serveur. Nous pouvons donc également spécifier manuellement l'adresse IP source ( `-S`) pour tester si nous obtenons de meilleurs résultats avec celle-ci. Les leurres peuvent être utilisés pour les analyses SYN, ACK, ICMP et les analyses de détection du système d'exploitation. Examinons donc un tel exemple et déterminons quel système d'exploitation il est le plus susceptible d'être.

#### Test de la règle de pare-feu

  Test de la règle de pare-feu

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p445 -O

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:23 CEST
Nmap scan report for 10.129.2.28
Host is up (0.032s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Too many fingerprints match this host to give specific OS details
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.14 seconds

```

#### Analyser en utilisant une adresse IP source différente

  Analyser en utilisant une adresse IP source différente

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-22 01:16 CEST
Nmap scan report for 10.129.2.28
Host is up (0.010s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%), Linux 2.6.32 - 3.10 (96%), Linux 3.4 - 3.10 (95%), Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), Synology DiskStation Manager 5.2-5644 (94%), Linux 2.6.32 - 2.6.35 (94%), Linux 2.6.32 - 3.5 (94%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 4.11 seconds

```

| Options de numérisation | Description |
| --- | --- |
| `10.129.2.28` | Analyse la cible spécifiée. |
| `-n` | Désactive la résolution DNS. |
| `-Pn` | Désactive les demandes d'écho ICMP. |
| `-p 445` | Analyse uniquement les ports spécifiés. |
| `-O` | Effectue une analyse de détection du système d'exploitation. |
| `-S` | Analyse la cible en utilisant une adresse IP source différente. |
| `10.129.2.200` | Spécifie l'adresse IP source. |
| `-e tun0` | Envoie toutes les requêtes via l'interface spécifiée. |

* * * * *

Proxy DNS
---------

Par défaut, `Nmap`effectue une résolution DNS inversée sauf indication contraire pour trouver des informations plus importantes sur notre cible. Ces requêtes DNS sont également transmises dans la plupart des cas car le serveur Web donné est censé être trouvé et visité. Les requêtes DNS sont effectuées sur le `UDP port 53`. Le `TCP port 53`n'était auparavant utilisé que pour le soi-disant " `Zone transfers`" entre les serveurs DNS ou le transfert de données de plus de 512 octets. De plus en plus, cela change en raison des extensions IPv6 et DNSSEC. Ces modifications entraînent l'envoi de nombreuses requêtes DNS via le port TCP 53.

Cependant, `Nmap`nous donne toujours un moyen de spécifier nous-mêmes les serveurs DNS ( `--dns-server <ns>,<ns>`). Cette méthode pourrait nous être fondamentale si nous sommes dans une zone démilitarisée ( `DMZ`). Les serveurs DNS de l'entreprise sont généralement plus fiables que ceux d'Internet. Ainsi, par exemple, nous pourrions les utiliser pour interagir avec les hôtes du réseau interne. Comme autre exemple, nous pouvons utiliser `TCP port 53`comme port source ( `--source-port`) pour nos scans. Si l'administrateur utilise le pare-feu pour contrôler ce port et ne filtre pas correctement IDS/IPS, nos paquets TCP seront approuvés et transmis.

#### SYN-Scan d'un port filtré

  SYN-Scan d'un port filtré

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-21 22:50 CEST
SENT (0.0417s) TCP 10.10.14.2:33436 > 10.129.2.28:50000 S ttl=41 id=21939 iplen=44  seq=736533153 win=1024 <mss 1460>
SENT (1.0481s) TCP 10.10.14.2:33437 > 10.129.2.28:50000 S ttl=46 id=6446 iplen=44  seq=736598688 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT      STATE    SERVICE
50000/tcp filtered ibm-db2

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds

```

#### SYN-Scan depuis le port DNS

  SYN-Scan depuis le port DNS

```
dsgsec@htb[/htb]$ sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53

SENT (0.0482s) TCP 10.10.14.2:53 > 10.129.2.28:50000 S ttl=58 id=27470 iplen=44  seq=4003923435 win=1024 <mss 1460>
RCVD (0.0608s) TCP 10.129.2.28:50000 > 10.10.14.2:53 SA ttl=64 id=0 iplen=44  seq=540635485 win=64240 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up (0.013s latency).

PORT      STATE SERVICE
50000/tcp open  ibm-db2
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds

```

| Options de numérisation | Description |
| --- | --- |
| `10.129.2.28` | Analyse la cible spécifiée. |
| `-p 50000` | Analyse uniquement les ports spécifiés. |
| `-sS` | Effectue une analyse SYN sur les ports spécifiés. |
| `-Pn` | Désactive les demandes d'écho ICMP. |
| `-n` | Désactive la résolution DNS. |
| `--disable-arp-ping` | Désactive le ping ARP. |
| `--packet-trace` | Affiche tous les paquets envoyés et reçus. |
| `--source-port 53` | Effectue les analyses à partir du port source spécifié. |

* * * * *

Maintenant que nous avons découvert que le pare-feu accepte `TCP port 53`, il est très probable que les filtres IDS/IPS soient également configurés beaucoup plus faibles que les autres. Nous pouvons tester cela en essayant de nous connecter à ce port en utilisant `Netcat`.

#### Connectez-vous au port filtré

  Connectez-vous au port filtré

```
dsgsec@htb[/htb]$ ncat -nv --source-port 53 10.129.2.28 50000

Ncat: Version 7.80 ( https://nmap.org/ncat )
Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd

```

`\
`
