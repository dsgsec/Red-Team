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

Dans la sortie ci-dessus, chaque carte réseau a un identifiant (`eth0`, `eth1`, `lo`, `tun0`) suivi d'informations d'adressage et de statistiques de trafic. L'interface de tunnel (tun0) indique qu'une connexion VPN est active. Lorsque nous nous connectons à l'un des serveurs VPN de HTB via Pwnbox ou notre propre hôte d'attaque, nous remarquons toujours qu'une interface de tunnel est créée et qu'une adresse IP lui est attribuée. Le VPN nous permet d'accéder aux environnements réseau du laboratoire hébergés par HTB. Gardez à l'esprit que ces réseaux de laboratoire ne sont pas accessibles sans avoir établi un tunnel. Le VPN chiffre le trafic et établit également un tunnel sur un réseau public (souvent Internet), via `NAT` sur une appliance de réseau public et dans le réseau interne/privé. Notez également les adresses IP attribuées à chaque carte réseau. L'adresse IP attribuée à eth0 (`134.122.100.200`) est une adresse IP routable publiquement. Cela signifie que les FAI achemineront le trafic provenant de cette adresse IP sur Internet. Nous verrons des adresses IP publiques sur des appareils qui font directement face à Internet, généralement hébergés dans des DMZ. Les autres cartes réseau ont des adresses IP privées, qui sont routables au sein des réseaux internes mais pas sur l'Internet public. Au moment de la rédaction, toute personne souhaitant communiquer via Internet doit disposer d'au moins une adresse IP publique attribuée à une interface sur l'appliance réseau qui se connecte à l'infrastructure physique se connectant à Internet. Rappelez-vous que NAT est couramment utilisé pour traduire les adresses IP privées en adresses IP publiques.

#### Utilisation d'ipconfig

Utiliser ipconfig

```
PS C:\Users\htb-student> ipconfig

Configuration IP de Windows

Adaptateur inconnu NordLynx :

    État des médias. . . . . . . . . . . : Média déconnecté
    Suffixe DNS spécifique à la connexion . :

Adaptateur Ethernet Ethernet0 2 :

    Suffixe DNS spécifique à la connexion . : .htb
    Adresse IPv6. . . . . . . . . . . : mort:boeuf::1a9
    Adresse IPv6. . . . . . . . . . . : mort:boeuf::f58b:6381:c648:1fb0
    Adresse IPv6 temporaire. . . . . . : mort:boeuf::dd0b:7cda:7118:3373
    Adresse IPv6 lien-local . . . . . : fe80::f58b:6381:c648:1fb0%8
    Adresse IPv4. . . . . . . . . . . : 10.129.221.36
    Masque de sous-réseau . . . . . . . . . . . : 255.255.0.0
    Passerelle par défaut . . . . . . . . . : fe80::250:56ff:feb9:df81%8
                                        10.129.0.1

Adaptateur Ethernet Ethernet :

    État des médias. . . . . . . . . . . : Média déconnecté
    Suffixe DNS spécifique à la connexion . :

```

La sortie directement ci-dessus provient de l'émission `ipconfig` sur un système Windows. Nous pouvons voir que ce système a plusieurs adaptateurs, mais un seul d'entre eux a des adresses IP attribuées. Il existe des adresses [IPv6](https://www.cisco.com/c/en/us/solutions/ipv6/overview.html) et un adresse [IPv4](https://en.wikipedia.org/wiki/IPv4 ) adresse. Ce module se concentrera principalement sur les réseaux exécutant IPv4 car il reste le mécanisme d'adressage IP le plus courant dans les réseaux locaux d'entreprise. Nous remarquerons que certains adaptateurs, comme celui de la sortie ci-dessus, auront une adresse IPv4 et une adresse IPv6 attribuées dans une [configuration à double pile](https://www.cisco.com/c/dam/en_us/solutions/ industries/docs/gov/IPV6at_a_glance_c45-625859.pdf) permettant d'accéder aux ressources via IPv4 ou IPv6.

Chaque adresse IPv4 aura un `masque de sous-réseau` correspondant. Si une adresse IP est comme un numéro de téléphone, le masque de sous-réseau est comme l'indicatif régional. N'oubliez pas que le masque de sous-réseau définit la partie `réseau` et `hôte` d'une adresse IP. Lorsque le trafic réseau est destiné à une adresse IP située sur un réseau différent, l'ordinateur envoie le trafic à la "passerelle par défaut" qui lui est attribuée. La passerelle par défaut est généralement l'adresse IP attribuée à une carte réseau sur une appliance agissant en tant que routeur pour un réseau local donné. Dans le contexte du pivotement, nous devons être conscients des réseaux sur lesquels un hôte sur lequel nous atterrissons peut atteindre, donc documenter autant d'informations d'adressage IP que possible sur un engagement peut s'avérer utile.

* * * * *

Routage
-------

Il est courant de penser à un appareil réseau qui nous connecte à Internet lorsque l'on pense à un routeur, mais techniquement, n'importe quel ordinateur peut devenir un routeur et participer au routage. Certains des défis auxquels nous serons confrontés dans ce module nous obligent à faire en sorte qu'un hôte pivot achemine le trafic vers un autre réseau. L'une des façons dont nous verrons cela est l'utilisation d'AutoRoute, qui permet à notre boîte d'attaque d'avoir des `routes` vers des réseaux cibles accessibles via un hôte pivot. L'une des principales caractéristiques d'un routeur est qu'il dispose d'une table de routage qu'il utilise pour transférer le trafic en fonction de l'adresse IP de destination. Regardons cela sur Pwnbox en utilisant les commandes `netstat -r` ou `ip route`.

#### Table de routage sur Pwnbox

Table de routage sur Pwnbox

```
dsgsec@htb[/htb]$ netstat -r

Table de routage IP du noyau
Destination Gateway Genmask Drapeaux MSS Window irtt Iface
par défaut 178.62.64.1 0.0.0.0 UG 0 0 0 eth0
10.10.10.0 10.10.14.1 255.255.254.0 UG 0 0 0 tun0
10.10.14.0 0.0.0.0 255.255.254.0 U 0 0 0 tun0
10.106.0.0 0.0.0.0 255.255.240.0 U 0 0 0 eth1
10.129.0.0 10.10.14.1 255.255.0.0 UG 0 0 0 tun0
178.62.64.0 0.0.0.0 255.255.192.0 U 0 0 0 eth0

```

Nous remarquerons que Pwnbox, les distributions Linux, Windows et de nombreux autres systèmes d'exploitation ont une table de routage pour aider le système à prendre des décisions de routage. Lorsqu'un paquet est créé et a une destination avant qu'il ne quitte l'ordinateur, la table de routage est utilisée pour décider où l'envoyer. Par exemple, si nous essayons de nous connecter à une cible avec l'adresse IP `10.129.10.25`, nous pourrions dire à partir de la table de routage où le paquet serait envoyé pour y arriver. Il serait transmis à une `Gateway` hors de la carte réseau correspondante (`Iface`). Pwnbox n'utilise aucun protocole de routage (EIGRP, OSPF, BGP, etc...) pour apprendre chacune de ces routes. Il a découvert ces routes via ses propres interfaces directement connectées (eth0, eth1, tun0). Les appareils autonomes désignés comme routeurs apprendront généralement les itinéraires en utilisant une combinaison de création d'itinéraire statique, de protocoles de routage dynamique et d'interfaces directement connectées. Tout trafic destiné à des réseaux non présents dans la table de routage sera envoyé à la `route par défaut`, qui peut également être appelée passerelle par défaut ou passerelle de dernier recours. Lorsque vous recherchez des opportunités de pivot, il peut être utile de consulter la table de routage des hôtes pour identifier les réseaux que nouspouvons être en mesure d'atteindre ou quels itinéraires nous devrons peut-être ajouter.

* * * * *

Protocoles, services et ports
---------------------------

Les "protocoles" sont les règles qui régissent les communications réseau. De nombreux protocoles et services ont des "ports" correspondants qui agissent comme des identifiants. Les ports logiques ne sont pas des éléments physiques auxquels nous pouvons toucher ou brancher quoi que ce soit. Ils sont dans des logiciels affectés à des applications. Lorsque nous voyons une adresse IP, nous savons qu'elle identifie un ordinateur qui peut être accessible sur un réseau. Lorsque nous voyons un port ouvert lié à cette adresse IP, nous savons qu'il identifie une application à laquelle nous pouvons nous connecter. La connexion à des ports spécifiques sur lesquels un appareil "écoute" peut souvent nous permettre d'utiliser des ports et des protocoles qui sont "autorisés" dans le pare-feu pour prendre pied sur le réseau.

Prenons, par exemple, un serveur Web utilisant HTTP (`souvent en écoute sur le port 80`). Les administrateurs ne doivent pas bloquer le trafic entrant sur le port 80. Cela empêcherait quiconque de visiter le site Web qu'ils hébergent. Il s'agit souvent d'un moyen d'accéder à l'environnement réseau, "par le même port que transite le trafic légitime". Nous ne devons pas négliger le fait qu'un `port source` est également généré pour suivre les connexions établies côté client d'une connexion. Nous devons garder à l'esprit les ports que nous utilisons pour nous assurer que lorsque nous exécutons nos charges utiles, ils se reconnectent aux auditeurs prévus que nous avons configurés. Nous ferons preuve de créativité avec l'utilisation des ports tout au long de ce module.

Pour un examen plus approfondi des concepts fondamentaux de mise en réseau, veuillez consulter le module [Introduction à la mise en réseau](https://academy.hackthebox.com/course/preview/introduction-to-networking).

* * * * *

Un conseil de LTNB0B : dans ce module, nous allons pratiquer de nombreux outils et techniques différents pour pivoter à travers les hôtes et transmettre des services locaux ou distants à notre hôte d'attaque pour accéder à des cibles connectées à différents réseaux. Ce module augmente progressivement en difficulté, fournissant des réseaux multi-hôtes pour mettre en pratique ce qui est appris. Je vous encourage fortement à pratiquer de nombreuses méthodes différentes de manière créative au fur et à mesure que vous commencez à comprendre les concepts. Essayez peut-être même de dessiner les topologies du réseau à l'aide d'outils de création de diagrammes de réseau lorsque vous faites face à des défis. Lorsque je cherche des opportunités de pivoter, j'aime utiliser des outils comme [Draw.io](https://draw.io/) pour créer un visuel de l'environnement réseau dans lequel je me trouve. Il s'agit d'un excellent outil de documentation car Bien. Ce module est très amusant et mettra vos compétences en réseautage à l'épreuve. Amusez-vous et n'arrêtez jamais d'apprendre!