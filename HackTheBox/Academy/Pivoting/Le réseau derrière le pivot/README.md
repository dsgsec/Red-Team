Le Réseau derrière le pivotement
==============================

* * * * *

Être capable de saisir suffisamment bien le concept de `pivoter` pour y parvenir dans le cadre d'un engagement nécessite une solide compréhension fondamentale de certains concepts clés de mise en réseau. Cette section sera un bref rappel sur les concepts de réseautage fondamentaux essentiels pour comprendre le pivotement.

Adressage IP et cartes réseau
--------------------

Chaque ordinateur qui communique sur un réseau a besoin d'une adresse IP. S'il n'en a pas, il n'est pas sur un réseau. L'adresse IP est attribuée dans le logiciel et généralement obtenue automatiquement à partir d'un serveur DHCP. Il est également courant de voir des ordinateurs avec des adresses IP attribuées de manière statique. L'attribution d'adresse IP statique est courante avec :

-   Les serveurs
- Routeurs
- Changer d'interfaces virtuelles
- Imprimantes
- Et tous les appareils qui fournissent des services critiques au réseau

Qu'elle soit attribuée `dynamiquement` ou `statiquement`, l'adresse IP est attribuée à un `contrôleur d'interface réseau` (`NIC`). Généralement, la carte réseau est appelée `Carte d'interface réseau` ou `Adaptateur réseau`. Un ordinateur peut avoir plusieurs cartes réseau (physiques et virtuelles), ce qui signifie qu'il peut se voir attribuer plusieurs adresses IP, ce qui lui permet de communiquer sur différents réseaux. L'identification des opportunités de pivot dépendra souvent des adresses IP spécifiques attribuées aux hôtes que nous compromettons, car elles peuvent indiquer les réseaux que les hôtes compromis peuvent atteindre. C'est pourquoi il est important pour nous de toujours rechercher des cartes réseau supplémentaires à l'aide de commandes telles que `ifconfig` (sous macOS et Linux) et `ipconfig` (sous Windows).

#### Utiliser ifconfig

Utiliser ifconfig

```
dsgsec@htb[/htb]$ ifconfig

eth0 : flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
         inet 134.122.100.200 masque de réseau 255.255.240.0 diffusion 134.122.111.255
         inet6 fe80::e973:b08d:7bdf:dc67 prefixlen 64 scopeid 0x20<lien>
         ether 12:ed:13:35:68:f5 txqueuelen 1000 (Ethernet)
         Paquets RX 8844 octets 803773 (784,9 Kio)
         Erreurs de réception 0 abandonnées 0 dépassements 0 trame 0
         Paquets TX 5698 octets 9713896 (9,2 Mio)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

eth1 : flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
         inet 10.106.0.172 masque de réseau 255.255.240.0 diffusion 10.106.15.255
         inet6 fe80::a5bf:1cd4:9bca:b3ae prefixlen 64 scopeid 0x20<lien>
         ether 4e:c7:60:b0:01:8d txqueuelen 1000 (Ethernet)
         Paquets RX 15 octets 1620 (1,5 Kio)
         Erreurs de réception 0 abandonnées 0 dépassements 0 trame 0
         Paquets TX 18 octets 1858 (1,8 Kio)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

lo : flags=73<UP,LOOPBACK,RUNNING> mtu 65536
         inet 127.0.0.1 masque de réseau 255.0.0.0
         inet6 ::1 prefixlen 128 scopeid 0x10<hôte>
         boucle txqueuelen 1000 (bouclage local)
         Paquets RX 19787 octets 10346966 (9,8 Mio)
         Erreurs de réception 0 abandonnées 0 dépassements 0 trame 0
         Paquets TX 19787 octets 10346966 (9,8 Mio)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

tun0 : flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST> mtu 1500
         inet 10.10.15.54 masque de réseau 255.255.254.0 destination 10.10.15.54
         inet6 fe80::c85a:5717:5e3a:38de prefixlen 64 scopeid 0x20<lien>
         inet6 dead:beef:2::1034 prefixlen 64 scopeid 0x0<global>
         unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00 txqueuelen 500 (UNSPEC)
         Paquets RX 0 octets 0 (0,0 B)
         Erreurs de réception 0 abandonnées 0 dépassements 0 trame 0
         Paquets TX 7 octets 336 (336,0 B)
         Erreurs TX 0 abandonnées 0 dépassements 0 porteuse 0 collisions 0

```

Dans la sortie ci-dessus, chaque carte réseau a un identifiant (`eth0`, `eth1`, `lo`, `tun0`) suivi d'informations d'adressage et de statistiques de trafic. L'interface de tunnel (tun0) indique qu'une connexion VPN est active. Lorsque nous nous connectons à l'un des serveurs VPN de HTB via Pwnbox ou notre propre hôte d'attaque, nous remarquons toujours qu'une interface de tunnel est créée et qu'une adresse IP lui est attribuée. Le VPN nous permet d'accéder aux environnements réseau du laboratoire hébergés par HTB. Gardez à l'esprit que ces réseaux de laboratoire ne sont pas accessibles sans avoir établi un tunnel. Le VPN chiffre le trafic et établit également un tunnel sur un réseau public (souvent Internet), via `NAT` sur une appliance de réseau public et dans le réseau interne/privé. Notez également les adresses IP attribuées à chaque carte réseau. L'adresse IP attribuée à eth0 (`134.122.100.200`) est une adresse IP routable publiquement. Cela signifie que les FAI achemineront le trafic provenant de cette adresse IP sur Internet. Nous verrons des adresses IP publiques sur des appareils qui font directement face à Internet, généralement hébergés dans des DMZ. Les autres cartes réseau ont des adresses IP privées, qui sont routables au sein des réseaux internes mais pas sur l'Internet public. Au moment de la rédaction, toute personne souhaitant communiquer via Internet doit disposer d'au moins une adresse IP publique attribuée à une interface sur l'appliance réseau qui se connecte à l'infrastructure physique se connectant à Internet. Rappelez-vous que NAT est couramment utilisé pour