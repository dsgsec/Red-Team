Tunnellisation RDP et SOCKS avec SocksOverRDP
========================================

* * * * *

Il arrive souvent, au cours d'une évaluation, que nous soyons limités à un réseau Windows et que nous ne soyons pas en mesure d'utiliser SSH pour pivoter. Nous devrions utiliser les outils disponibles pour les systèmes d'exploitation Windows dans ces cas. [SocksOverRDP](https://github.com/nccgroup/SocksOverRDP) est un exemple d'outil qui utilise `Dynamic Virtual Channels` (`DVC`) de la fonctionnalité Remote Desktop Service de Windows. DVC est responsable de la tunnellisation des paquets sur la connexion RDP. Quelques exemples d'utilisation de cette fonctionnalité seraient le transfert de données du presse-papiers et le partage audio. Cependant, cette fonctionnalité peut également être utilisée pour tunnelliser des paquets arbitraires sur le réseau. Nous pouvons utiliser `SocksOverRDP` pour tunneler nos paquets personnalisés, puis passer par proxy. Nous utiliserons l'outil [Proxifier](https://www.proxifier.com/) comme serveur proxy.

Nous pouvons commencer par télécharger les fichiers binaires appropriés sur notre hôte d'attaque pour effectuer cette attaque. Avoir les fichiers binaires sur notre hôte d'attaque nous permettra de les transférer sur chaque cible si nécessaire. Nous aurons besoin:

1. [SocksOverRDP x64 binaires](https://github.com/nccgroup/SocksOverRDP/releases)

2. [Proxifier Portable Binary](https://www.proxifier.com/download/#win-tab)

- Nous pouvons rechercher `ProxifierPE.zip`

Nous pouvons ensuite nous connecter à la cible à l'aide de xfreerdp et copier le fichier `SocksOverRDPx64.zip` sur la cible. Depuis la cible Windows, nous devrons ensuite charger le fichier SocksOverRDP.dll à l'aide de regsvr32.exe.

#### Chargement de SocksOverRDP.dll à l'aide de regsvr32.exe

Chargement de SocksOverRDP.dll à l'aide de regsvr32.exe

```
C:\Users\htb-student\Desktop\SocksOverRDP-x64> regsvr32.exe SocksOverRDP-Plugin.dll

```

![](https://academy.hackthebox.com/storage/modules/158/socksoverrdpdll.png)

Nous pouvons maintenant nous connecter à 172.16.5.19 via RDP en utilisant `mstsc.exe`, et nous devrions recevoir une invite indiquant que le plug-in SocksOverRDP est activé et qu'il écoutera sur 127.0.0.1:1080. Nous pouvons utiliser les identifiants `victor:pass@123` pour nous connecter à 172.16.5.19.

![](https://academy.hackthebox.com/storage/modules/158/pivotingtoDC.png)

Nous devrons transférer le fichier SocksOverRDPx64.zip ou simplement le fichier SocksOverRDP-Server.exe vers 172.16.5.19. Nous pouvons ensuite démarrer SocksOverRDP-Server.exe avec des privilèges d'administrateur.

![](https://academy.hackthebox.com/storage/modules/158/executingsocksoverrdpserver.png)

Lorsque nous revenons à notre objectif d'ancrage et vérifions avec Netstat, nous devrions voir notre écouteur SOCKS démarré sur 127.0.0.1:1080.

#### Confirmation du démarrage de l'écouteur SOCKS

Confirmation du démarrage de l'écouteur SOCKS

```
C:\Users\htb-student\Desktop\SocksOverRDP-x64> netstat -antb | findstr 1080

   TCP 127.0.0.1:1080 0.0.0.0:0 ÉCOUTE

```

Après avoir démarré notre écouteur, nous pouvons transférer Proxifier portable vers la cible Windows 10 (sur le réseau 10.129.x.x), et le configurer pour transférer tous nos paquets vers 127.0.0.1:1080. Proxifier acheminera le trafic via l'hôte et le port donnés. Voir le clip ci-dessous pour une présentation rapide de la configuration de Proxifier.

#### Configuration du proxy

![](https://academy.hackthebox.com/storage/modules/158/configuringproxifier.gif)

Avec Proxifier configuré et en cours d'exécution, nous pouvons démarrer mstsc.exe, et il utilisera Proxifier pour faire pivoter tout notre trafic via 127.0.0.1:1080, qui le tunnelera sur RDP vers 172.16.5.19, qui l'acheminera ensuite vers 172.16.6.155 à l'aide de SocksOverRDP-server.exe.

![](https://academy.hackthebox.com/storage/modules/158/rdpsockspivot.png)

#### Considérations sur les performances RDP

Lors de l'interaction avec nos sessions RDP sur un engagement, nous pouvons nous retrouver confrontés à des performances lentes dans une session donnée, en particulier si nous gérons plusieurs sessions RDP simultanément. Si tel est le cas, nous pouvons accéder à l'onglet `Expérience` dans mstsc.exe et définir `Performance` sur `Modem`.

![](https://academy.hackthebox.com/storage/modules/158/rdpexpen.png)

* * * * *

Remarque : lors de la création de votre cible, nous vous demandons d'attendre 3 à 5 minutes jusqu'à ce que l'ensemble du laboratoire avec toutes les configurations soit configuré afin que la connexion à votre cible fonctionne parfaitement.