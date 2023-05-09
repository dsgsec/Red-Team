Service d'énumération et d'exploitation
==================================

* * * * *

Services d'écoute
------------------

Nos analyses Nmap ont révélé quelques services intéressants :

- Port 21 : FTP
- Port 22 : SSH
- Port 25 : SMTP
- Port 53 : DNS
- Port 80 : HTTP
- Ports 110/143/993/995 : imap & pop3
- Port 111 : rpcbind

Nous avons déjà effectué un transfert de zone DNS lors de notre collecte d'informations initiale, ce qui a généré plusieurs sous-domaines que nous approfondirons plus tard. D'autres attaques DNS ne valent pas la peine d'être tentées dans notre environnement actuel.

* * * * *

FTP
---

Commençons par FTP sur le port 21. Le Nmap Aggressive Scan a découvert qu'une connexion FTP anonyme était possible. Confirmons cela manuellement.

```
dsgsec@htb[/htb]$ ftp 10.129.203.101

Connecté au 10.129.203.101.
220 (vsFTPd 3.0.3)
Nom (10.129.203.101 : testeur) : anonyme
331 Veuillez spécifier le mot de passe.
Mot de passe:
230 Connexion réussie.
Le type de système distant est UNIX.
Utiliser le mode binaire pour transférer des fichiers.
ftp>ls
200 Commande PORT réussie. Envisagez d'utiliser le PASV.
150 Voici la liste des répertoires.
-rw-r--r-- 1 0 0 38 30 mai 17:16 flag.txt
226 Envoi répertoire OK.
ftp>

```

La connexion avec l'utilisateur "anonyme" et un mot de passe vide fonctionne. Il ne semble pas que nous puissions accéder à des fichiers intéressants à part un, et nous ne pouvons pas non plus changer de répertoire.

```
ftp> mettre test.txt

local : test.txt distant : test.txt
200 Commande PORT réussie. Envisagez d'utiliser le PASV.
550 Autorisation refusée.

```

Nous ne sommes pas non plus en mesure de télécharger un fichier.

D'autres attaques, telles qu'une attaque FTP Bounce, sont peu probables et nous n'avons pas encore d'informations sur le réseau interne. La recherche d'exploits publics pour vsFTPd 3.0.3 n'affiche [ceci](https://www.exploit-db.com/exploits/49719) PoC pour un `Remote Denial of Service`, ce qui n'entre pas dans le cadre de nos tests . Le forçage brutal ne nous aidera pas ici non plus puisque nous ne connaissons aucun nom d'utilisateur.

Cela ressemble à une impasse. Allons-nous en.

* * * * *

SSH
---

La prochaine étape est SSH. Nous allons commencer par une capture de bannière :

```
dsgsec@htb[/htb]$ nc -nv 10.129.203.101 22

(INCONNU) [10.129.203.101] 22 (ssh) ouvert
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5

```

Cela nous montre que l'hôte exécute OpenSSH version 8.2, qui ne présente aucune vulnérabilité connue au moment de la rédaction. Nous pourrions essayer de forcer brutalement les mots de passe, mais nous n'avons pas de liste de noms d'utilisateur valides, ce serait donc un coup dans le noir. Il est également douteux que nous puissions forcer brutalement le mot de passe root. Nous pouvons essayer quelques combinaisons telles que `admin:admin`, `root:toor`, `admin:Welcome`, `admin:Pass123` mais sans succès.

```
dsgsec@htb[/htb]$ ssh admin@10.129.203.101

L'authenticité de l'hôte '10.129.203.101 (10.129.203.101)' ne peut pas être établie.
L'empreinte digitale de la clé ECDSA est SHA256:3I77Le3AqCEUd+1LBAraYTRTF74wwJZJiYcnwfF5yAs.
Voulez-vous vraiment continuer à vous connecter (oui/non/[empreinte digitale]) ? Oui
Avertissement : Ajout permanent de '10.129.203.101' (ECDSA) à la liste des hôtes connus.
Mot de passe de admin@10.129.203.101 :
Autorisation refusée, veuillez réessayer.

```

SSH ressemble également à une impasse. Voyons ce que nous avons d'autre.

* * * * *

### Services de messagerie

SMTP est intéressant. Nous pouvons consulter la section [Attaquer les services de messagerie](https://academy.hackthebox.com/module/116/section/1173) du module `Attaquer les services communs` pour obtenir de l'aide. Dans une évaluation réelle, nous pourrions utiliser un site Web tel que [MXToolbox](https://mxtoolbox.com/) ou l'outil `dig` pour énumérer les enregistrements MX.

Faisons une autre analyse du port 25 pour rechercher les erreurs de configuration.

```
dsgsec@htb[/htb]$ sudo nmap -sV -sC -p25 10.129.203.101

À partir de Nmap 7.92 ( https://nmap.org ) au 20/06/2022 à 18h55 HAE
Rapport d'analyse Nmap pour inlanefreight.local (10.129.203.101)
L'hôte est actif (latence de 0,11 s).

VERSION SERVICE À L'ÉTAT DU PORT
25/tcp open smtp Postfix smtpd
| ssl-cert : Objet : commonName=ubuntu
| Nom alternatif du sujet : DNS:ubuntu
| Non valide avant : 2022-05-30T17:15:40
|_Non valide après : 2032-05-27T17:15:40
|_commandes smtp : ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
|_ssl-date : le caractère aléatoire de TLS ne représente pas le temps
Informations sur le service : Hôte : Ubuntu

Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 5,37 secondes

```

Ensuite, nous vérifierons toute erreur de configuration liée à l'authentification. Nous pouvons essayer d'utiliser la commande `VRFY` pour énumérer les utilisateurs du système.

```
dsgsec@htb[/htb]$ telnet 10.129.203.101 25

Essayer 10.129.203.101...
Connecté au 10.129.203.101.
Le caractère d'échappement est '^]'.
220 ubuntu ESMTP Postfix (Ubuntu)
Racine VRFY
252 2.0.0 racine
VRFY www-données
252 2.0.0 www-données
VRFY utilisateur aléatoire
550 5.1.1 <randomuser> : adresse du destinataire rejetée : utilisateur inconnu dans la table des destinataires locaux

```

Nous pouvons voir que la commande `VRFY` n'est pas désactivée et nous pouvons l'utiliser pour énumérer les utilisateurs valides. Cela pourrait potentiellement être exploité pour rassembler une liste d'utilisateurs que nous pourrions utiliser pour monter une attaque par force brute de mot de passe contre le FTP et SServices SH et peut-être d'autres. Bien qu'il s'agisse d'un risque relativement faible, il convient de le noter comme un résultat "faible" pour notre rapport, car nos clients doivent réduire autant que possible leur surface d'attaque externe. S'il n'y a pas de raison commerciale valable pour activer cette commande, nous devons leur conseiller de la désactiver.

Nous pourrions essayer d'énumérer plus d'utilisateurs avec un outil tel que [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum) pour enfoncer le clou et potentiellement trouver plus d'utilisateurs. Cela ne vaut généralement pas la peine de passer beaucoup de temps à forcer l'authentification pour les services externes. Cela pourrait entraîner une interruption de service, donc même si nous pouvons créer une liste d'utilisateurs, nous pouvons essayer quelques mots de passe faibles et passer à autre chose.

Nous pourrions répéter ce processus avec les commandes `EXPN` et `RCPT TO` , mais cela n'apportera rien de plus.

Le protocole `POP3` peut également être utilisé pour énumérer les utilisateurs en fonction de sa configuration. Nous pouvons réessayer d'énumérer les utilisateurs du système avec la commande `USER` et si le serveur répond par `+OK`, l'utilisateur existe sur le système. Cela ne fonctionne pas pour nous. En sondant le port 995, le port SSL/TLS pour POP3 ne donne rien non plus.

```
dsgsec@htb[/htb]$ telnet 10.129.203.101 110

Essayer 10.129.203.101...
Connecté au 10.129.203.101.
Le caractère d'échappement est '^]'.
+OK Dovecot (Ubuntu) prêt.
utilisateur www-données
-ERR [AUTH] Authentification en clair non autorisée sur les connexions non sécurisées (SSL/TLS).

```

Le [Footprinting](https://academy.hackthebox.com/module/112/section/1073) module contient plus d'informations sur les services communs et les principes d'énumération et mérite d'être revu après avoir parcouru cette section.

Nous voudrions examiner plus en détail la mise en œuvre de la messagerie électronique du client dans une évaluation du monde réel. S'ils utilisent Office 365 ou Exchange sur site, nous pourrons peut-être monter une attaque par pulvérisation de mot de passe qui pourrait donner accès aux boîtes de réception des e-mails ou potentiellement au réseau interne si nous pouvons utiliser un mot de passe de messagerie valide pour nous connecter via VPN. Nous pouvons également rencontrer un relais ouvert, dont nous pourrions éventuellement abuser pour le phishing en envoyant des e-mails en tant qu'utilisateurs inventés ou en usurpant un compte de messagerie pour rendre un e-mail officiel et tenter d'inciter les employés à entrer des informations d'identification ou à exécuter une charge utile. Le phishing est hors de portée pour cette évaluation particulière et le sera probablement pour la plupart des tests de pénétration externes, donc ce type de vulnérabilité mériterait d'être confirmé et signalé si nous le rencontrions, mais nous ne devrions pas aller plus loin qu'une simple validation sans vérifier avec le client d'abord. Cependant, cela pourrait être extrêmement utile lors d'une évaluation complète de l'équipe rouge.

Nous pouvons quand même le vérifier, mais ne trouvons pas de relais ouvert, ce qui est bon pour notre client !

```
dsgsec@htb[/htb]$ nmap -p25 -Pn --script smtp-open-relay 10.129.203.101

À partir de Nmap 7.92 ( https://nmap.org ) au 20/06/2022 à 19h14 HAE
Rapport d'analyse Nmap pour inlanefreight.local (10.129.203.101)
L'hôte est actif (latence de 0,12 s).

SERVICE DE L'ÉTAT DU PORT
25/tcp ouvert smtp
|_smtp-open-relay : le serveur ne semble pas être un relais ouvert, tous les tests ont échoué

Nmap done : 1 adresse IP (1 hôte en place) scannée en 24,30 secondes

```

* * * * *

Passer à autre chose
---------

Le port 111 est le service `rpcbind` qui ne doit pas être exposé à l'extérieur. Nous pourrions donc rédiger un résultat `Low` pour les `Services exposés inutiles` ou similaire. Ce port peut être sondé pour identifier le système d'exploitation ou potentiellement recueillir des informations sur les services disponibles. Nous pouvons essayer de le sonder avec la commande [rpcinfo](https://linux.die.net/man/8/rpcinfo) ou Nmap. Cela fonctionne, mais nous ne récupérons rien d'utile. Encore une fois, il convient de noter que le client est conscient de ce qu'il expose, mais que nous ne pouvons rien en faire d'autre.

```
dsgsec@htb[/htb]$ rpcinfo 10.129.203.101

    version du programme netid adresse propriétaire du service
     100000 4 tcp6 ::.0.111 super-utilisateur portmapper
     100000 3 tcp6 ::.0.111 super-utilisateur portmapper
     100000 4 udp6 ::.0.111 superutilisateur portmapper
     100000 3 udp6 ::.0.111 portmapper superutilisateur
     100000 4 tcp 0.0.0.0.0.111 superutilisateur portmapper
     100000 3 tcp 0.0.0.0.0.111 superutilisateur portmapper
     100000 2 tcp 0.0.0.0.0.111 superutilisateur portmapper
     100000 4 udp 0.0.0.0.0.111 superutilisateur portmapper
     100000 3 udp 0.0.0.0.0.111 superutilisateur portmapper
     100000 2 udp 0.0.0.0.0.111 superutilisateur portmapper
     100000 4 superutilisateur local /run/rpcbind.sock portmapper
     100000 3 superutilisateur local /run/rpcbind.sock portmapper

```

Il vaut la peine de consulter ce guide HackTricks sur [Pentesting rpcbind](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rpcbind) pour une prise de conscience future concernant ce service.

Le dernier port est le port `80`, qui, comme nous le savons, est le service HTTP. Nous savons qu'il existe probablement plusieurs applications Web basées sur le sousl'énumération de domaine et vhost que nous avons effectuée précédemment. Passons donc au web. Nous n'avons toujours pas de pied ou quoi que ce soit à part une poignée de découvertes à risque moyen et faible. Dans les environnements modernes, nous voyons rarement des services exploitables en externe comme un serveur FTP vulnérable ou similaire qui conduira à l'exécution de code à distance (RCE). Ne dites jamais jamais, cependant. Nous avons vu des choses plus folles, il vaut donc toujours la peine d'explorer toutes les possibilités. La plupart des organisations auxquelles nous sommes confrontés seront les plus susceptibles d'être attaquées via leurs applications Web, car celles-ci présentent souvent une vaste surface d'attaque. Nous passerons donc généralement la majeure partie de notre temps lors d'un test de pénétration externe à énumérer et à attaquer les applications Web.