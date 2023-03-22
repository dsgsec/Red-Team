Tunnellisation ICMP avec SOCKS
=========================

* * * * *

Le tunneling ICMP encapsule votre trafic dans des `paquets ICMP` contenant des `demandes d'écho` et des `réponses`. Le tunneling ICMP ne fonctionnerait que lorsque les réponses ping sont autorisées dans un réseau protégé par un pare-feu. Lorsqu'un hôte au sein d'un réseau protégé par un pare-feu est autorisé à envoyer un ping à un serveur externe, il peut encapsuler son trafic dans la requête d'écho ping et l'envoyer à un serveur externe. Le serveur externe peut valider ce trafic et envoyer une réponse appropriée, ce qui est extrêmement utile pour l'exfiltration de données et la création de tunnels pivots vers un serveur externe.

Nous utiliserons l'outil [ptunnel-ng](https://github.com/utoni/ptunnel-ng) pour créer un tunnel entre notre serveur Ubuntu et notre hôte d'attaque. Une fois qu'un tunnel est créé, nous pourrons faire passer notre trafic par proxy via le `ptunnel-ng client`. Nous pouvons démarrer le `serveur ptunnel-ng` sur l'hôte pivot cible. Commençons par configurer ptunnel-ng.

* * * * *

Configuration et utilisation de ptunnel-ng
-----------------------------

Si ptunnel-ng n'est pas sur notre hôte d'attaque, nous pouvons cloner le projet en utilisant git.

#### Clonage Ptunnel-ng

Clonage Ptunnel-ng

```
dsgsec@htb[/htb]$ git clone https://github.com/utoni/ptunnel-ng.git

```

Une fois le référentiel ptunnel-ng cloné sur notre hôte d'attaque, nous pouvons exécuter le script `autogen.sh` situé à la racine du répertoire ptunnel-ng.

#### Construire Ptunnel-ng avec Autogen.sh

Construire Ptunnel-ng avec Autogen.sh

```
dsgsec@htb[/htb]$ sudo ./autogen.sh

```

Après avoir exécuté authgen.sh, ptunnel-ng peut être utilisé côté client et côté serveur. Nous allons maintenant devoir transférer le référentiel de notre hôte d'attaque vers l'hôte cible. Comme dans les sections précédentes, nous pouvons utiliser SCP pour transférer les fichiers. Si nous voulons transférer l'intégralité du dépôt et les fichiers qu'il contient, nous devrons utiliser l'option `-r` avec SCP.

#### Transfert de Ptunnel-ng vers l'hôte Pivot

Transfert de Ptunnel-ng vers l'hôte pivot

```
dsgsec@htb[/htb]$ scp -r ptunnel-ng ubuntu@10.129.202.64:~/

```

Avec ptunnel-ng sur l'hôte cible, nous pouvons démarrer le côté serveur du tunnel ICMP en utilisant la commande directement ci-dessous.

#### Démarrage du serveur ptunnel-ng sur l'hôte cible

Démarrage du serveur ptunnel-ng sur l'hôte cible

```
ubuntu@WEB01 :~/ptunnel-ng/src$ sudo ./ptunnel-ng -r10.129.202.64 -R22

[sudo] mot de passe pour ubuntu :
./ptunnel-ng : /lib/x86_64-linux-gnu/libselinux.so.1 : aucune information de version disponible (requis par ./ptunnel-ng)
[inf] : Démarrage de ptunnel-ng 1.42.
[inf] : (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf] : (c) 2017-2019 Toni Uhlig, <matzeton@googlemail.com>
[inf] : Éléments de sécurité par Sébastien Raveau, <sebastien.raveau@epita.fr>
[inf] : transfert des paquets ping entrants via TCP.
[inf] : le proxy Ping écoute en mode privilégié.
[inf] : Supprimer les privilèges maintenant.

```

L'adresse IP qui suit `-r` doit être l'adresse IP sur laquelle nous voulons que ptunnel-ng accepte les connexions. Dans ce cas, quelle que soit l'adresse IP accessible depuis notre hôte d'attaque, nous l'utiliserions. Nous aurions intérêt à utiliser cette même réflexion et considération lors d'un engagement réel.

De retour sur l'hôte d'attaque, nous pouvons tenter de nous connecter au serveur ptunnel-ng (`-p <ipAddressofTarget>`) mais assurez-vous que cela se produit via le port local 2222 (`-l2222`). La connexion via le port local 2222 nous permet d'envoyer du trafic via le tunnel ICMP.

#### Connexion au serveur ptunnel-ng à partir de l'hôte d'attaque

Connexion au serveur ptunnel-ng à partir de l'hôte d'attaque

```
dsgsec@htb[/htb]$ sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22

[inf] : Démarrage de ptunnel-ng 1.42.
[inf] : (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf] : (c) 2017-2019 Toni Uhlig, <matzeton@googlemail.com>
[inf] : Éléments de sécurité par Sébastien Raveau, <sebastien.raveau@epita.fr>
[inf] : relayer les paquets à partir des flux TCP entrants.

```

Une fois le tunnel ICMP ptunnel-ng établi avec succès, nous pouvons tenter de nous connecter à la cible en utilisant SSH via le port local 2222 (`-p2222`).

#### Tunnellisation d'une connexion SSH via un tunnel ICMP

Tunnellisation d'une connexion SSH via un tunnel ICMP

```
dsgsec@htb[/htb]$ ssh -p2222 -lubuntu 127.0.0.1

Mot de passe de ubuntu@127.0.0.1 :
Bienvenue dans Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-générique x86_64)

  * Documentation : https://help.ubuntu.com
  * Gestion : https://landscape.canonical.com
  * Assistance : https://ubuntu.com/advantage

   Informations système au mer. 11 mai 2022 15:10:15 UTC

   Charge système : 0,0
   Utilisation de / : 39,6 % de 13,72 Go
   Utilisation de la mémoire : 37 %
   Utilisation de l'échange : 0 %
   Processus : 183
   Utilisateurs connectés : 1
   Adresse IPv4 pour ens192 : 10.129.202.64
   Adresse IPv6 pour ens192 : dead:beef::250:56ff:feb9:52eb
   Adresse IPv4 pour ens224 : 172.16.5.129

  * Super-optimisé pour les petits espaces - découvrez comment nous avons réduit la mémoire
    empreinte de MicroK8s pour en faire le plus petit K8 complet autour.

    https://ubuntu.com/blog/microk8s-memory-optimisation

144 mises à jour peuvent être appliquées immédiatement.
97 de ces mises à jourtes sont des mises à jour de sécurité standard.
Pour voir ces mises à jour supplémentaires, exécutez : apt list --upgradable

Dernière connexion : mer 11 mai 14:53:22 2022 à partir du 10.10.14.18
ubuntu@WEB01 :~$

```

S'il est configuré correctement, nous pourrons entrer des informations d'identification et avoir une session SSH tout au long du tunnel ICMP.

Du côté client et serveur de la connexion, nous remarquerons que ptunnel-ng nous donne des journaux de session et des statistiques de trafic associées au trafic qui passe par le tunnel ICMP. C'est une façon de confirmer que notre trafic passe du client au serveur en utilisant ICMP.

#### Affichage des statistiques de trafic du tunnel

Affichage des statistiques de trafic du tunnel

```
inf] : demande de tunnel entrante du 10.10.14.18.
[inf] : Démarrage d'une nouvelle session à 10.129.202.64:22 avec ID 20199
[inf] : session fermée reçue du pair distant.
[inf] :
Statistiques de session :
[inf] : E/S : 0,00/ 0,00 Mo ICMP E/S/R : 248/ 22/ 0 Perte : 0,0 %
[inf] :

```

Nous pouvons également utiliser ce tunnel et SSH pour effectuer une redirection de port dynamique afin de nous permettre d'utiliser les proxychains de différentes manières.

#### Activation du transfert de port dynamique via SSH

Activation du transfert de port dynamique via SSH

```
dsgsec@htb[/htb]$ ssh -D 9050 -p2222 -lubuntu 127.0.0.1

Mot de passe de ubuntu@127.0.0.1 :
Bienvenue dans Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-générique x86_64)
<extrait>

```

Nous pourrions utiliser des proxychains avec Nmap pour analyser les cibles sur le réseau interne (172.16.5.x). Sur la base de nos découvertes, nous pouvons tenter de nous connecter à la cible.

#### Proxychaining via le tunnel ICMP

Proxychaining via le tunnel ICMP

```
dsgsec@htb[/htb]$ proxychains nmap -sV -sT 172.16.5.19 -p3389

ProxyChains-3.1 (http://proxychains.sf.net)
À partir de Nmap 7.92 ( https://nmap.org ) au 2022-05-11 11:10 EDT
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<><>-OK
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|Chaîne en S|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Rapport d'analyse Nmap pour 172.16.5.19
L'hôte est actif (latence de 0,12 s).

VERSION SERVICE À L'ÉTAT DU PORT
3389/tcp open ms-wbt-server Microsoft Terminal Services
Informations sur le service : système d'exploitation : Windows ; CPE : cpe:/o:microsoft:windows

Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 8,78 secondes

```

* * * * *

Considérations sur l'analyse du trafic réseau
---------------------------------------

Il est important que nous confirmions que les outils que nous utilisons fonctionnent comme annoncé et que nous les avons configurés et que nous les exploitons correctement. Dans le cas du tunneling du trafic via différents protocoles enseignés dans cette section avec le tunneling ICMP, nous pouvons bénéficier de l'analyse du trafic que nous générons avec un analyseur de paquets comme `Wireshark`. Regardez attentivement le court clip ci-dessous.

![](https://academy.hackthebox.com/storage/modules/158/analyzingTheTraffic.gif)

Dans la première partie de ce clip, une connexion est établie via SSH sans utiliser de tunnel ICMP. Nous pouvons remarquer que le trafic `TCP` & `SSHv2` est capturé.

La commande utilisée dans le clip : `ssh ubuntu@10.129.202.64`

Dans la deuxième partie de ce clip, une connexion est établie via SSH à l'aide du tunneling ICMP. Notez le type de trafic qui est capturé lorsque cela est effectué.

Commande utilisée dans le clip : `ssh -p2222 -lubuntu 127.0.0.1`

* * * * *

Remarque : lors de la création de votre cible, nous vous demandons d'attendre 3 à 5 minutes jusqu'à ce que l'ensemble du laboratoire avec toutes les configurations soit configuré afin que la connexion à votre cible fonctionne parfaitement.