Transfert de port dynamique avec tunnel SSH et SOCKS
================================================= ==

* * * * *

Redirection de port en contexte
--------------------------

Le `port forwarding` est une technique qui nous permet de rediriger une demande de communication d'un port à un autre. Le transfert de port utilise TCP comme couche de communication principale pour fournir une communication interactive pour le port transféré. Cependant, différents protocoles de couche application tels que SSH ou même [SOCKS](https://en.wikipedia.org/wiki/SOCKS) (couche non application) peuvent être utilisés pour encapsuler le trafic transféré. Cela peut être efficace pour contourner les pare-feu et utiliser les services existants sur votre hôte compromis pour pivoter vers d'autres réseaux.

* * * * *

Transfert de port local SSH
-------------------------

Prenons un exemple à partir de l'image ci-dessous.

![](https://academy.hackthebox.com/storage/modules/158/11.png)

Remarque : chaque schéma de réseau présenté dans ce module est conçu pour illustrer les concepts abordés dans la section associée. L'adressage IP indiqué dans les schémas ne correspond pas toujours exactement aux environnements de laboratoire. Assurez-vous de vous concentrer sur la compréhension du concept, et vous constaterez que les diagrammes s'avéreront très utiles ! Après avoir lu cette section, assurez-vous de vous référer à nouveau à l'image ci-dessus pour renforcer les concepts.

Nous avons notre hôte d'attaque (10.10.15.x) et un serveur cible Ubuntu (10.129.x.x), que nous avons compromis. Nous allons analyser le serveur Ubuntu cible à l'aide de Nmap pour rechercher les ports ouverts.

#### Numérisation de la cible pivot

Balayage de la cible pivot

```
dsgsec@htb[/htb]$ nmap -sT -p22,3306 10.129.202.64

À partir de Nmap 7.92 ( https://nmap.org ) au 2022-02-24 12:12 EST
Rapport d'analyse Nmap pour 10.129.202.64
L'hôte est actif (latence de 0,12 s).

SERVICE DE L'ÉTAT DU PORT
22/tcp ouvrir ssh
3306/tcp fermé mysql

Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 0,68 seconde

```

La sortie Nmap indique que le port SSH est ouvert. Pour accéder au service MySQL, nous pouvons soit nous connecter en SSH au serveur et accéder à MySQL depuis le serveur Ubuntu, soit nous pouvons le transférer vers notre hôte local sur le port `1234` et y accéder localement. L'un des avantages d'y accéder localement est que si nous voulons exécuter un exploit à distance sur le service MySQL, nous ne pourrons pas le faire sans redirection de port. Cela est dû au fait que MySQL est hébergé localement sur le serveur Ubuntu sur le port `3306`. Nous allons donc utiliser la commande ci-dessous pour transférer notre port local (1234) via SSH vers le serveur Ubuntu.

#### Exécution du transfert de port local

Exécution de la redirection de port local

```
dsgsec@htb[/htb]$ ssh -L 1234:localhost:3306 Ubuntu@10.129.202.64

Mot de passe de ubuntu@10.129.202.64 :
Bienvenue dans Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-générique x86_64)

  * Documentation : https://help.ubuntu.com
  * Gestion : https://landscape.canonical.com
  * Assistance : https://ubuntu.com/advantage

   Informations système au jeu. 24 févr. 2022 17:23:20 UTC

   Charge système : 0,0
   Utilisation de / : 28,4 % de 13,72 Go
   Utilisation de la mémoire : 34 %
   Utilisation de l'échange : 0 %
   Processus : 175
   Utilisateurs connectés : 1
   Adresse IPv4 pour ens192 : 10.129.202.64
   Adresse IPv6 pour ens192 : dead:beef::250:56ff:feb9:52eb
   Adresse IPv4 pour ens224 : 172.16.5.129

  * Super-optimisé pour les petits espaces - découvrez comment nous avons réduit la mémoire
    empreinte de MicroK8s pour en faire le plus petit K8 complet autour.

    https://ubuntu.com/blog/microk8s-memory-optimisation

66 mises à jour peuvent être appliquées immédiatement.
45 de ces mises à jour sont des mises à jour de sécurité standard.
Pour voir ces mises à jour supplémentaires, exécutez : apt list --upgradable

```

La commande `-L` indique au client SSH de demander au serveur SSH de transférer toutes les données que nous envoyons via le port `1234` vers `localhost : 3306` sur le serveur Ubuntu. En faisant cela, nous devrions pouvoir accéder au service MySQL localement sur le port 1234. Nous pouvons utiliser Netstat ou Nmap pour interroger notre hôte local sur le port 1234 afin de vérifier si le service MySQL a été transféré.

#### Confirmation de la redirection de port avec Netstat

Confirmation de la redirection de port avec Netstat

```
dsgsec@htb[/htb]$ netstat -antp | grep 1234

(Tous les processus n'ont pas pu être identifiés, les informations de processus non détenues
  ne sera pas affiché, vous devrez être root pour tout voir.)
tcp 0 0 127.0.0.1:1234 0.0.0.0:* ÉCOUTER 4034/ssh
tcp6 0 0 ::1:1234 :::* ÉCOUTER 4034/ssh

```

#### Confirmation de la redirection de port avec Nmap

Confirmation de la redirection de port avec Nmap

```
dsgsec@htb[/htb]$ nmap -v -sV -p1234 localhost

À partir de Nmap 7.92 ( https://nmap.org ) au 2022-02-24 12:18 EST
NSE : 45 scripts chargés pour l'analyse.
Lancement de l'analyse Ping à 12:18
Analyse de l'hôte local (127.0.0.1) [2 ports]
Scan Ping terminé à 12:18, 0.01s écoulé (1 hôtes au total)
Lancement de Connect Scan à 12:18
Analyse de l'hôte local (127.0.0.1) [1 port]
Découverte du port ouvert 1234/tcp sur 127.0.0.1
Analyse de connexion terminée à 12h18, 0,01 s écoulée (1 port au total)
Lancement de l'analyse du service à 12:18
Analyse 1 service sur localhost (127.0.0.1)
Service après-vente terminén à 12:18, 0.12s écoulé (1 service sur 1 hôte)
NSE : analyse de script 127.0.0.1.
Lancement de l'ESN à 12h18
NSE terminée à 12:18, 0.01s écoulé
Lancement de l'ESN à 12h18
NSE terminé à 12:18, 0.00s écoulé
Rapport d'analyse Nmap pour localhost (127.0.0.1)
L'hôte est actif (latence de 0,0080 s).
Autres adresses pour localhost (non analysées) : ::1

VERSION SERVICE À L'ÉTAT DU PORT
1234/tcp ouvrir mysql MySQL 8.0.28-0ubuntu0.20.04.3

Lire les fichiers de données depuis : /usr/bin/../share/nmap
Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 1,18 secondes

```

De même, si nous voulons transférer plusieurs ports du serveur Ubuntu vers votre hôte local, vous pouvez le faire en incluant l'argument `local port:server:port` à votre commande ssh. Par exemple, la commande ci-dessous transfère le port 80 du serveur Web apache au port local de votre hôte d'attaque sur `8080`.

Confirmation de la redirection de port avec Nmap

```
dsgsec@htb[/htb]$ ssh -L 1234:localhost:3306 8080:localhost:80 ubuntu@10.129.202.64

```

* * * * *

Mise en place pour pivoter
-------------------

Maintenant, si vous tapez `ifconfig` sur l'hôte Ubuntu, vous constaterez que ce serveur possède plusieurs cartes réseau :

- Un connecté à notre hôte d'attaque (`ens192`)
- Un communiquant avec d'autres hôtes au sein d'un réseau différent (`ens224`)
- L'interface de bouclage (`lo`).

#### À la recherche d'opportunités de pivoter à l'aide d'ifconfig

À la recherche d'opportunités de pivoter à l'aide d'ifconfig

```
ubuntu@WEB01 :~$ ifconfig

ens192 : flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
         inet 10.129.202.64 masque de réseau 255.255.0.0 diffusion 10.129.255.255
         inet6 dead:beef::250:56ff:feb9:52eb prefixlen 64 scopeid 0x0<global>
         inet6 fe80::250:56ff:feb9:52eb prefixlen 64 scopeid 0x20<lien>
         ether 00:50:56:b9:52:eb txqueuelen 1000 (Ethernet)
         Paquets RX 35571 octets 177919049 (177,9 Mo)
         Erreurs de réception 0 abandonnées 0 dépassements 0 trame 0
         Paquets TX 10452 octets 1474767 (1,4 Mo)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

ens224 : flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
         inet 172.16.5.129 masque de réseau 255.255.254.0 diffusion 172.16.5.255
         inet6 fe80::250:56ff:feb9:a9aa prefixlen 64 scopeid 0x20<lien>
         ether 00:50:56:b9:a9:aa txqueuelen 1000 (Ethernet)
         Paquets RX 8251 octets 1125190 (1,1 Mo)
         Erreurs de réception 0 abandonnées 40 dépassements 0 trame 0
         Paquets TX 1538 octets 123584 (123,5 Ko)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

lo : flags=73<UP,LOOPBACK,RUNNING> mtu 65536
         inet 127.0.0.1 masque de réseau 255.0.0.0
         inet6 ::1 prefixlen 128 scopeid 0x10<hôte>
         boucle txqueuelen 1000 (bouclage local)
         Paquets RX 270 octets 22432 (22,4 Ko)
         Erreurs de réception 0 abandonnées 0 dépassements 0 trame 0
         Paquets TX 270 octets 22432 (22,4 Ko)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

```

Contrairement au scénario précédent où nous savions à quel port accéder, dans notre scénario actuel, nous ne savons pas quels services se trouvent de l'autre côté du réseau. Ainsi, nous pouvons analyser des plages d'adresses IP plus petites sur le réseau (`172.16.5.1-200`) ou l'ensemble du sous-réseau (`172.16.5.0/23`). Nous ne pouvons pas effectuer cette analyse directement à partir de notre hôte d'attaque car il n'a pas de routes vers le réseau `172.16.5.0/23` . Pour ce faire, nous devrons effectuer une `redirection de port dynamique` et `pivoter` nos paquets réseau via le serveur Ubuntu. Nous pouvons le faire en démarrant un `écouteur SOCKS` sur notre `hôte local` (hôte d'attaque personnelle ou Pwnbox), puis en configurant SSH pour transférer ce trafic via SSH vers le réseau (172.16.5.0/23) après la connexion à l'hôte cible. .

C'est ce qu'on appelle le `tunneling SSH` sur le `proxy SOCKS`. SOCKS signifie `Socket Secure`, un protocole qui permet de communiquer avec les serveurs sur lesquels vous avez mis en place des restrictions de pare-feu. Contrairement à la plupart des cas où vous initiez une connexion pour vous connecter à un service, dans le cas de SOCKS, le trafic initial est généré par un client SOCKS, qui se connecte au serveur SOCKS contrôlé par l'utilisateur qui souhaite accéder à un service sur le client. -côté. Une fois la connexion établie, le trafic réseau peut être acheminé via le serveur SOCKS au nom du client connecté.

Cette technique est souvent utilisée pour contourner les restrictions mises en place par les pare-feu et permettre à une entité externe de contourner le pare-feu et d'accéder à un service dans l'environnement protégé par un pare-feu. Un autre avantage de l'utilisation du proxy SOCKS pour le pivotement et le transfert de données est que les proxys SOCKS peuvent pivoter en créant une route vers un serveur externe à partir de "réseaux NAT". Les proxys SOCKS sont actuellement de deux types : `SOCKS4` et `SOCKS5`. SOCKS4 ne fournit aucune authentification ni prise en charge UDP, alors que SOCKS5 le fournit. Prenons un exemple de l'image ci-dessous où nous avons un réseau NAT de 172.16.5.0/23, auquel nous ne pouvons pas accéder directement.

![](https://academy.hackthebox.com/storage/modules/158/22.png)

Dans l'image ci-dessus, l'attaque host démarre le client SSH et demande au serveur SSH de lui permettre d'envoyer des données TCP via le socket ssh. Le serveur SSH répond par un accusé de réception, et le client SSH commence alors à écouter sur `localhost:9050`. Quelles que soient les données que vous envoyez ici, elles seront diffusées sur l'ensemble du réseau (172.16.5.0/23) via SSH. Nous pouvons utiliser la commande ci-dessous pour effectuer cette redirection de port dynamique.

#### Activation du transfert de port dynamique avec SSH

Activation du transfert de port dynamique avec SSH

```
dsgsec@htb[/htb]$ ssh -D 9050 ubuntu@10.129.202.64

```

L'argument `-D` demande au serveur SSH d'activer le transfert de port dynamique. Une fois cette option activée, nous aurons besoin d'un outil capable d'acheminer les paquets de n'importe quel outil sur le port `9050`. Nous pouvons le faire en utilisant l'outil `proxychains`, qui est capable de rediriger les connexions TCP via les serveurs proxy TOR, SOCKS et HTTP/HTTPS et nous permet également de chaîner plusieurs serveurs proxy ensemble. À l'aide de proxychains, nous pouvons également masquer l'adresse IP de l'hôte demandeur, car l'hôte récepteur ne verra que l'adresse IP de l'hôte pivot. Les chaînes de proxy sont souvent utilisées pour forcer le "trafic TCP" d'une application à passer par des proxys hébergés tels que `SOCKS4`/`SOCKS5`, `TOR` ou `HTTP`/`HTTPS` proxies.

Pour informer proxychains que nous devons utiliser le port 9050, nous devons modifier le fichier de configuration proxychains situé dans `/etc/proxychains.conf`. Nous pouvons ajouter `socks4 127.0.0.1 9050` à la dernière ligne si elle n'y est pas déjà.

#### Vérification de /etc/proxychains.conf

Vérification de /etc/proxychains.conf

```
dsgsec@htb[/htb]$ tail -4 /etc/proxychains.conf

# en attendant
# valeurs par défaut définies sur "tor"
socks4 127.0.0.1 9050

```

Désormais, lorsque vous démarrez Nmap avec proxychains à l'aide de la commande ci-dessous, tous les paquets de Nmap seront acheminés vers le port local 9050, où notre client SSH écoute, qui transmettra tous les paquets via SSH au réseau 172.16.5.0/23.

#### Utilisation de Nmap avec Proxychains

Utilisation de Nmap avec Proxychains

```
dsgsec@htb[/htb]$ proxychains nmap -v -sn 172.16.5.1-200

ProxyChains-3.1 (http://proxychains.sf.net)

À partir de Nmap 7.92 ( https://nmap.org ) au 2022-02-24 12:30 EST
Lancement de l'analyse Ping à 12h30
Balayage de 10 hôtes [2 ports/hôte]
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.2:80-<--timeout
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.5:80-<><>-OK
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.6:80-<--timeout
RTTVAR est passé à plus de 2,3 secondes, diminuant à 2,0

<SNIP>

```

Cette partie de l'emballage de toutes vos données Nmap à l'aide de proxychains et de leur transfert vers un serveur distant s'appelle `tunneling SOCKS`. Une autre remarque importante à retenir ici est que nous ne pouvons effectuer qu'une `analyse complète de la connexion TCP` sur les chaînes de proxy. La raison en est que les chaînes proxy ne peuvent pas comprendre les paquets partiels. Si vous envoyez des paquets partiels comme des scans à moitié connectés, cela renverra des résultats incorrects. Nous devons également nous assurer que nous sommes conscients du fait que les vérifications "host-alive" peuvent ne pas fonctionner sur les cibles Windows, car le pare-feu Windows Defender bloque les requêtes ICMP (pings traditionnels) par défaut.

[Une analyse complète de la connexion TCP](https://nmap.org/book/scan-methods-connect-scan.html) sans ping sur toute une plage de réseau prendra beaucoup de temps. Ainsi, pour ce module, nous nous concentrerons principalement sur l'analyse d'hôtes individuels, ou sur de plus petites plages d'hôtes dont nous savons qu'elles sont actives, qui dans ce cas sera un hôte Windows à `172.16.5.19`.

Nous allons effectuer une analyse du système à distance à l'aide de la commande ci-dessous.

#### Énumération de la cible Windows via Proxychains

Énumération de la cible Windows via Proxychains

```
dsgsec@htb[/htb]$ proxychains nmap -v -Pn -sT 172.16.5.19

ProxyChains-3.1 (http://proxychains.sf.net)
Découverte d'hôte désactivée (-Pn). Toutes les adresses seront marquées "up" et les temps de balayage peuvent être plus lents.
À partir de Nmap 7.92 ( https://nmap.org ) au 2022-02-24 12:33 EST
Lancement de la résolution DNS parallèle d'un hôte. à 12:33
Résolution DNS parallèle terminée de 1 hôte. à 12:33, 0.15s écoulé
Lancement de Connect Scan à 12:33
Balayage 172.16.5.19 [1000 ports]
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:1720-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:587-<--timeout
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:445-<><>-OK
Découverte du port ouvert 445/tcp sur 172.16.5.19
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:8080-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:23-<--timeout
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:135-<><>-OK
Découverte du port ouvert 135/tcp sur 172.16.5.19
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:110-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:21-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:554-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-1172.16.5.19:25-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:5900-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:1025-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:143-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:199-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:993-<--timedehors
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:995-<--timeout
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Découverte du port ouvert 3389/tcp sur 172.16.5.19
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:443-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:113-<--timeout
|Chaîne S|-<>-127.0.0.1:9050-<><>-172.16.5.19:8888-<--timeout
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:139-<><>-OK
Découverte du port ouvert 139/tcp sur 172.16.5.19

```

L'analyse Nmap montre plusieurs ports ouverts, dont le `port RDP` (3389). Semblable à l'analyse Nmap, nous pouvons également faire pivoter `msfconsole` via des chaînes proxy pour effectuer des analyses RDP vulnérables à l'aide de modules auxiliaires Metasploit. Nous pouvons démarrer msfconsole avec proxychains.

* * * * *

Utilisation de Metasploit avec Proxychains
---------------------------------

Nous pouvons également ouvrir Metasploit à l'aide de proxychains et envoyer tout le trafic associé via le proxy que nous avons établi.

Énumération de la cible Windows via Proxychains

```
dsgsec@htb[/htb]$ proxychains msfconsole

ProxyChains-3.1 (http://proxychains.sf.net)

      .~+P``````-o+ :. -o+ :.
.+oooyysyyssyyssyddh++os-````` ``````````````` `
+++++++++++++++++++++++sydhyoyso/:.````...`...-///::+ohhyosyyosyy/+om++ : ooo///o
++++///////~~~~///////++++++++++++++++ooyysoysosso++++++++++++++++++++///oossosy
--.` .-.-...-////++++++++++++++++////////~~//////+++ +++++++++///
                                 `...............` `...-/////...`

                                   .::::::::::-. .::::::-
                                 .hmMMMMMMMMMMNddds\...//M\\.../hddddmMMMMMMNon
                                  :Nm-/NMMMMMMMMMMMM$$NMMMMm&&MMMMMMMMMMMMMMa
                                  .sm/`-yMMMMMMMMMMMM$$MMMMMN&&MMMMMMMMMMMMh`
                                   -Nd` :MMMMMMMMMM$$MMMMMN&&MMMMMMMMMMMMh`
                                    -Nh` .yMMMMMMMMMM$$MMMMMN&&MMMMMMMMMMm/
     `oo/``-hd : `` .sNd :MMMMMMMMMM$$MMMMMN&&MMMMMMMMMMm/
       .yNmMMh//+syysso-`````` -mh` :MMMMMMMMMM$$MMMMMN&&MMMMMMMMMMd
     .shMMMMN//dmNMMMMMMMMMMMMs` `:```-o++++oooo+:/ooooo+:+o+++oooo++/
     `///omh//dMMMMMMMMMMMMMMMN/:::::/+ooso--/ydh//+s+/ossssso:--syN///os:
           /MMMMMMMMMMMMMMMMMMd. `/++-.-yy/...osydh/-+oo:-`o//...oyodh+
           -hMMmssddd+:dMMmNMMh. `.-=mmk.//^^^\\.^^`:++:^^o://^^^\\` ::
           .sMMmo. -dMd--:mN/` ||--X--|| ||--X--||
........../yddy/:...+hmo-...hdd:............\\=v=//...... ......\\=v=//.........
================================================= ==============================
=====================+----------------------------------- ----+=========================
=====================| La première session est morte de dysenterie. |=========================
=====================+----------------------------------- ----+=========================
================================================= ==============================

                      Appuyez sur ENTER pour évaluer la situation

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%% Date : 25 avril 1848 %%%%%%%%%%%%%%% %%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%% Météo: Il fait toujours frais au labo %%%%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%% Santé: Surpoids %%%%%%%%%%%%%%%%%%%%% %%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%% Caféine: 12975 mg %%%%%%%%%%%%%%%%%%%%%% %%%%%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%% piraté : toutes les choses %%%%%%%%%%%%%%%%%%% %%%%%%%%%%%
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%% %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

                         Appuyez sur la barre d'espace pour continuer

        =[ metasploit v6.1.27-dev ]
+ -- --=[ 2196 exploits - 1162 auxiliaires - 400 post ]
+ -- --=[ 596 payloads - 45 encodeurs - 10 nops ]
+ -- --=[ 9 évasion ]

Astuce Metasploit : les noms d'adaptateur peuvent être utilisés pour les paramètres IP
définir LHOST eth0

msf6 >

```

Utilisons le module auxiliaire `rdp_scanner` pour vérifier si l'hôte sur le réseau interne écoute sur 3389.

#### Utilisation du module rdp_scanner

Utilisation du module rdp_scanner

```
msf6 > rechercher rdp_scanner

Modules correspondants
================

    # Nom Divulgation Date Rang Vérification Description
    - ---- --------------- ---- ----- -----------
    0 auxiliaire/scanner/rdp/rdp_scanner normal Non Identifier les points finaux utilisant le protocole RDP (Remote Desktop Protocol)

Interagir avec un module par nom ou index. Par exemple info 0, utilisez 0 ou utilisez auxiliaire/scanner/rdp/rdp_scanner

msf6> utiliser 0
msf6 auxiliaire (scanner/rdp/rdp_scanner) > définir les rhosts 172.16.5.19
rhosts => 172.16.5.19
msf6 auxiliaire (scanner/rdp/rdp_scanner) > exécuter
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK

[*] 172.16.5.19:3389 - RDP détecté sur 172.16.5.19:3389 (name:DC01) (domain:DC01) (domain_fqdn:DC01) (server_fqdn:DC01) (os_version:10.0.17763) (nécessite NLA : non)
[*] 172.16.5.19:3389 - Scanné 1 des 1 hôtes (100% complet)
[*] Exécution du module auxiliaire terminée

```

Au bas de la sortie ci-dessus, nous pouvons voir le port RDP ouvert avec la version du système d'exploitation Windows.

Selon le niveau d'accès que nous avons à cet hôte lors d'une évaluation, nous pouvons essayer d'exécuter un exploit ou de nous connecter en utilisant les informations d'identification recueillies. Pour ce module, nous allons nous connecter à l'hôte distant Windows via le tunnel SOCKS. Cela peut être fait en utilisant `xfreerdp`. Dans notre cas, l'utilisateur est `victor` et le mot de passe est `pass@123`

#### Utilisation de xfreerdp avec Proxychains

Utilisation de xfreerdp avec Proxychains

```
dsgsec@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123

ProxyChains-3.1 (http://proxychains.sf.net)
[13:02:42:481] [4829:4830] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex réinitialisant l'état d'erreur
[13:02:42:482] [4829:4830] [INFO][com.freerdp.client.common.cmdline] - chargement de channelEx rdpdr
[13:02:42:482] [4829:4830] [INFO][com.freerdp.client.common.cmdline] - chargement de channelEx rdpsnd
[13:02:42:482] [4829:4830] [INFO][com.freerdp.client.common.cmdline] - chargement de channelEx cliprdr

```

La commande xfreerdp nécessitera l'acceptation d'un certificat RDP avant d'établir avec succès la session. Après l'avoir accepté, nous devrions avoir une session RDP, pivotant via le serveur Ubuntu.

#### Pivot RDP réussi

![RDP Pivot](https://academy.hackthebox.com/storage/modules/158/proxychaining.png)