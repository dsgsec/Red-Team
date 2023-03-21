Pivotement SSH avec Sshuttle
==========================

* * * * *

[Sshuttle](https://github.com/sshuttle/sshuttle) est un autre outil écrit en Python qui supprime la nécessité de configurer des proxychains. Cependant, cet outil ne fonctionne que pour pivoter sur SSH et ne fournit pas d'autres options pour pivoter sur des serveurs proxy TOR ou HTTPS. `Sshuttle` peut être extrêmement utile pour automatiser l'exécution d'iptables et ajouter des règles de pivot pour l'hôte distant. Nous pouvons configurer le serveur Ubuntu comme point pivot et acheminer tout le trafic réseau de Nmap avec sshuttle en utilisant l'exemple plus loin dans cette section.

Une utilisation intéressante de sshuttle est que nous n'avons pas besoin d'utiliser des chaînes proxy pour nous connecter aux hôtes distants. Installons sshuttle via notre hôte pivot Ubuntu et configurons-le pour se connecter à l'hôte Windows via RDP.

#### Installation de la navette

Installation de la navette

```
dsgsec@htb[/htb]$ sudo apt-get install sshuttle

Lecture des listes de paquets... Terminé
Construction de l'arborescence des dépendances... Terminé
Lecture des informations d'état... Terminé
Les packages suivants ont été installés automatiquement et ne sont plus nécessaires :
   alsa-tools golang-1.15 golang-1.15-doc golang-1.15-go golang-1.15-src
   golang-1.16-src libcmis-0.5-5v5 libct4 libgvm20 liblibreoffice-java
   libmotif-common libqrcodegencpp1 libunoloader-java libxm4
   linux-headers-5.10.0-6parrot1-common python-babel-localedata
   python3-aiofiles python3-babel python3-fastapi python3-pydantic
   python3-slowapi python3-starlette python3-uvicorn sqsh ure-java
Utilisez 'sudo apt autoremove' pour les supprimer.
Forfaits suggérés :
   autossh
Les NOUVEAUX packages suivants seront installés :
   navette
0 mis à jour, 1 nouvellement installé, 0 à supprimer et 4 non mis à jour.
Besoin d'obtenir 91,8 Ko d'archives.
Après cette opération, 508 ko d'espace disque supplémentaire seront utilisés.
Obtenez :1 https://ftp-stud.hs-esslingen.de/Mirrors/archive.parrotsec.org rolling/main amd64 sshuttle all 1.0.5-1 [91,8 kB]
Récupéré 91,8 ko en 2 s (52,1 ko/s)
Sélection du package sshuttle précédemment désélectionné.
(Lecture de la base de données... 468019 fichiers et répertoires actuellement installés.)
Préparation du déballage .../sshuttle_1.0.5-1_all.deb ...
Déballage de la navette (1.0.5-1) ...
Configuration de la navette (1.0.5-1) ...
Traitement des déclencheurs pour man-db (2.9.4-2) ...
Traitement des déclencheurs pour doc-base (0.11.1) ...
Traitement de 1 fichier doc-base ajouté...
Analyse des lanceurs d'applications
Suppression des lanceurs en double ou des lanceurs cassés
Les lanceurs sont mis à jour

```

Pour utiliser sshuttle, nous spécifions l'option `-r` pour se connecter à la machine distante avec un nom d'utilisateur et un mot de passe. Ensuite, nous devons inclure le réseau ou l'adresse IP que nous voulons acheminer via l'hôte pivot, dans notre cas, il s'agit du réseau 172.16.5.0/23.

#### Navette en cours d'exécution

Navette en cours d'exécution

```
dsgsec@htb[/htb]$ sudo sshuttle -r ubuntu@10.129.202.64 172.16.5.0/23 -v

Démarrage du proxy sshuttle (version 1.1.0).
c : Démarrage du gestionnaire de pare-feu avec la commande : ['/usr/bin/python3', '/usr/local/lib/python3.9/dist-packages/sshuttle/__main__.py', '-v', '--method ', 'auto', '--pare-feu']
fw : Démarrage du pare-feu avec Python version 3.9.2
fw : nom de la méthode prête nat.
c : IPv6 activé : Utilisation de l'adresse d'écoute IPv6 par défaut ::1
c : Méthode : nat
c : IPv4 : activé
c : IPv6 : activé
c : UDP : désactivé (non disponible avec la méthode nat)
c : DNS : désactivé (disponible)
c : Utilisateur : désactivé (disponible)
c : sous-réseaux à transférer via l'hôte distant (type, IP, cidr mask width, startPort, endPort) :
c : (<AddressFamily.AF_INET : 2>, '172.16.5.0', 32, 0, 0)
c : Sous-réseaux à exclure du transfert :
c : (<AddressFamily.AF_INET : 2>, '127.0.0.1', 32, 0, 0)
c : (<AddressFamily.AF_INET6 : 10>, '::1', 128, 0, 0)
c : redirecteur TCP en écoute ('::1', 12300, 0, 0).
c : redirecteur TCP en écoute ('127.0.0.1', 12300).
c : Démarrage du client avec Python version 3.9.2
c : Connexion au serveur...
Mot de passe de ubuntu@10.129.202.64 :
  s : Exécution du serveur sur un hôte distant avec /usr/bin/python3 (version 3.8.10)
  s : paramètre de contrôle de latence = Vrai
  s : auto-nets : Faux
c : Connecté au serveur.
fw : mise en place.
fw : ip6tables -w -t nat -N sshuttle-12300
fw : ip6tables -w -t nat -F sshuttle-12300
fw : ip6tables -w -t nat -I SORTIE 1 -j sshuttle-12300
fw : ip6tables -w -t nat -I PREROUTING 1 -j sshuttle-12300
fw : ip6tables -w -t nat -A sshuttle-12300 -j RETURN -m addrtype --dst-type LOCAL
fw: ip6tables -w -t nat -A sshuttle-12300 -j RETOUR --dest ::1/128 -p tcp
fw : iptables -w -t nat -N sshuttle-12300
fw : iptables -w -t nat -F sshuttle-12300
fw: iptables -w -t nat -I SORTIE 1 -j sshuttle-12300
fw: iptables -w -t nat -I PREROUTING 1 -j sshuttle-12300
fw: iptables -w -t nat -A sshuttle-12300 -j RETURN -m addrtype --dst-type LOCAL
fw: iptables -w -t nat -A sshuttle-12300 -j RETURN --dest 127.0.0.1/32 -p tcp
fw : iptables -w -t nat -A sshuttle-12300 -j REDIRECT --dest 172.16.5.0/32 -p tcp --to-ports 12300

```

Avec cette commande, sshuttle crée une entrée dans nos `iptables` pour rediriger tout le trafic vers le réseau 172.16.5.0/23 via l'hôte pivot.

#### Routage du trafic via les routes iptables

Routage du trafic via les routes iptables

```
dsgsec@htb[/htb]$ nmap -v-sV -p3389 172.16.5.19 -A -Pn

Découverte d'hôte désactivée (-Pn). Toutes les adresses seront marquées "up" et les temps de balayage peuvent être plus lents.
À partir de Nmap 7.92 ( https://nmap.org ) au 2022-03-08 11:16 EST
NSE : 155 scripts chargés pour l'analyse.
NSE : Script Pre-scanning.
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de la résolution DNS parallèle d'un hôte. à 11:16
Résolution DNS parallèle terminée de 1 hôte. à 11h16, 0.15s écoulé
Lancement de Connect Scan à 11:16
Balayage 172.16.5.19 [1 port]
Scan de connexion terminé à 11:16, 2.00s se sont écoulés (1 ports au total)
Lancement de l'analyse du service à 11:16
NSE : analyse de script 172.16.5.19.
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Rapport d'analyse Nmap pour 172.16.5.19
L'hôte est en place.

VERSION SERVICE À L'ÉTAT DU PORT
3389/tcp open ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info :
| Nom_Cible : INLANEFREIGHT
| NetBIOS_Domain_Name : INLANEFREIGHT
| Nom_ordinateur_NetBIOS : DC01
| Nom_de_domaine_DNS : inlanefreight.local
| Nom_ordinateur_DNS : DC01.inlanefreight.local
| Version_du_produit : 10.0.17763
|_ Heure_système : 2022-08-14T02:58:25+00:00
|_ssl-date : 2022-08-14T02:58:25+00:00 ; +7s à partir de l'heure du scanner.
| ssl-cert : Objet : commonName=DC01.inlanefreight.local
| Émetteur : commonName=DC01.inlanefreight.local
| Type de clé publique : rsa
| Bits de clé publique : 2048
| Algorithme de signature : sha256WithRSAEncryption
| Non valide avant : 2022-08-13T02:51:48
| Non valide après : 2023-02-12T02:51:48
| MD5 : 58a1 27de 5f06 fea6 0e18 9a02 f0de 982b
|_SHA-1 : f490 dc7d 3387 9962 745a 9ef8 8c15 d20e 477f 88cb
Informations sur le service : système d'exploitation : Windows ; CPE : cpe:/o:microsoft:windows

Résultats du script hôte :
|_clock-skew : moyenne : 6 s, écart : 0 s, médiane : 6 s

NSE : Script Post-scanning.
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lancement de l'ESN à 11h16
NSE terminé à 11:16, 0.00s écoulé
Lire les fichiers de données depuis : /usr/bin/../share/nmap
Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 4,07 secondes

```

Nous pouvons maintenant utiliser n'importe quel outil directement sans utiliser de proxychains.

* * * * *

Remarque : lors de la création de votre cible, nous vous demandons d'attendre 3 à 5 minutes jusqu'à ce que l'ensemble du laboratoire avec toutes les configurations soit configuré afin que la connexion à votre cible fonctionne parfaitement. SSH vers la cible avec l'utilisateur "`ubuntu`" et le mot de passe "`HTB_@cademy_stdnt!`"