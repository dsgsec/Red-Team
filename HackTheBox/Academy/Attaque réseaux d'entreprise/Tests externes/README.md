Collecte d'informations externes
==============================

* * * * *

Nous commençons par un rapide balayage initial de Nmap par rapport à notre cible pour avoir une idée du terrain et voir à quoi nous avons affaire. Nous nous assurons d'enregistrer toutes les sorties d'analyse dans le sous-répertoire approprié de notre répertoire de projet.

```
dsgsec@htb[/htb]$ sudo nmap --open -oA inlanefreight_ept_tcp_1k -iL portée

À partir de Nmap 7.92 ( https://nmap.org ) au 20/06/2022 à 14h56 HAE
Rapport d'analyse Nmap pour 10.129.203.101
L'hôte est actif (latence de 0,12 s).
Non illustré : 989 ports tcp fermés (réinitialisés)
SERVICE DE L'ÉTAT DU PORT
21/tcp ouvert ftp
22/tcp ouvrir ssh
25/tcp ouvert smtp
domaine ouvert 53/tcp
80/tcp ouvert http
110/tcp ouvert pop3
111/tcp ouvrir rpcbind
143/tcp ouvrir l'imap
Images ouvertes 993/tcp
995/tcp ouvert pop3s
8080/tcp open http-proxy

Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 2,25 secondes

```

Nous remarquons 11 ports ouverts lors de notre analyse rapide des 1 000 ports TCP. Il semble que nous ayons affaire à un serveur Web qui exécute également des services supplémentaires tels que FTP, SSH, e-mail (SMTP, pop3 et IMAP), DNS et au moins deux ports liés aux applications Web.

Entre-temps, nous avons exécuté une analyse complète des ports à l'aide de l'indicateur `-A` ([Options d'analyse agressive](https://nmap.org/book/man-misc-options.html)) pour effectuer une énumération supplémentaire, y compris Détection du système d'exploitation, analyse de version et analyse de script. Gardez à l'esprit qu'il s'agit d'une analyse plus intrusive que la simple exécution avec l'indicateur `-sV` pour l'analyse de version, et nous devons veiller à ce que les scripts exécutés avec l'analyse de script ne causent aucun problème.

```
dsgsec@htb[/htb]$ sudo nmap --open -p- -A -oA inlanefreight_ept_tcp_all_svc -iL portée

À partir de Nmap 7.92 ( https://nmap.org ) au 20/06/2022 à 15h27 HAE
Rapport d'analyse Nmap pour 10.129.203.101
L'hôte est actif (latence de 0,12 s).
Non illustré : 65524 ports tcp fermés (réinitialisés)
VERSION SERVICE À L'ÉTAT DU PORT
21/tcp ouvert ftp vsftpd 3.0.3
| ftp-anon : connexion FTP anonyme autorisée (code FTP 230)
|_-rw-r--r-- 1 0 0 38 30 mai 17:16 flag.txt
| ftp-syst :
| STAT :
| État du serveur FTP :
| Connecté à ::ffff:10.10.14.15
| Connecté en ftp
| TYPE : ASCII
| Aucune limite de bande passante de session
| Le délai d'expiration de la session en secondes est de 300
| La connexion de contrôle est en texte brut
| Les connexions de données seront en texte brut
| Au démarrage de la session, le nombre de clients était de 1
| vsFTPd 3.0.3 - sécurisé, rapide, stable
|_Fin de statut
22/tcp open ssh OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux ; protocole 2.0)
| ssh-hostkey :
| 3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
| 256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_ 256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
25/tcp open smtp Postfix smtpd
|_ssl-date : le caractère aléatoire de TLS ne représente pas le temps
| ssl-cert : Objet : commonName=ubuntu
| Nom alternatif du sujet : DNS:ubuntu
| Non valide avant : 2022-05-30T17:15:40
|_Non valide après : 2032-05-27T17:15:40
|_commandes smtp : ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
domaine ouvert 53/tcp
| chaînes d'empreintes digitales :
| DNSVersionBindReqTCP :
| version
| lier
| dns-nsid :
|_ bind.version :
80/tcp ouvrir http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header : Apache/2.4.41 (Ubuntu)
|_http-title : Inlanefreight
110/tcp ouvert pop3 Pigeonnier pop3d
|_ssl-date : le caractère aléatoire de TLS ne représente pas le temps
| ssl-cert : Objet : commonName=ubuntu
| Nom alternatif du sujet : DNS:ubuntu
| Non valide avant : 2022-05-30T17:15:40
|_Non valide après : 2032-05-27T17:15:40
|_pop3-capabilities : SASL TOP PIPELINING STLS RESP-CODES AUTH-RESP-CODE CAPA UIDL
111/tcp ouvert rpcbind 2-4 (RPC #100000)
| rpcinfo :
| version du programme port/proto service
| 100000 2,3,4 111/tcp rpcbind
| 100000 2,3,4 111/udp rpcbind
| 100000 3,4 111/tcp6 rpcbind
|_ 100000 3,4 111/udp6 rpcbind
143/tcp open imap Dovecot imapd (Ubuntu)
|_imap-capabilities: LITERAL+ LOGIN-REFERRALS plus Les capacités d'identification post-connexion pré-connexion répertoriées ont LOGINDISABLEDA0001 OK ENABLE IDLE STARTTLS SASL-IR IMAP4rev1
|_ssl-date : le caractère aléatoire de TLS ne représente pas le temps
| ssl-cert : Objet : commonName=ubuntu
| Nom alternatif du sujet : DNS:ubuntu
| Non valide avant : 2022-05-30T17:15:40
|_Non valide après : 2032-05-27T17:15:40
993/tcp open ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date : le caractère aléatoire de TLS ne représente pas le temps
| ssl-cert : Objet : commonName=ubuntu
| Nom alternatif du sujet : DNS:ubuntu
| Non valide avant : 2022-05-30T17:15:40
|_Non valide après : 2032-05-27T17:15:40
|_imap-capabilities : LITERAL+ LOGIN-REFERRALS AUTH=PLAINA0001 les capacités d'identification post-connexion ont été répertoriées OK ENABLE IDLE Pre-login SASL-IR IMAP4rev1
995/tcp open ssl/pop3 Pigeonnier pop3d
| ssl-cert : Objet : commonName=ubuntu
| Nom alternatif du sujet : DNS:ubuntu
| Non valide avant : 2022-05-30T17:15:40
|_Non valide après : 2032-05-27T17:15:40
|_ssl-date : le caractère aléatoire TLS ne se reproduit pastemps actuel
|_pop3-capabilities : SASL(PLAIN) TOP PIPELINING CAPA RESP-CODES AUTH-RESP-CODE USER UIDL
8080/tcp ouvrir http Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header : Apache/2.4.41 (Ubuntu)
| http-open-proxy : Proxy potentiellement OPEN.
|_Méthodes supportées :CONNEXION
|_http-title : Centre d'assistance
1 service non reconnu malgré le retour de données. Si vous connaissez le service/la version, veuillez soumettre l'empreinte digitale suivante à https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.92%I=7%D=6/20%Time=62B0CA68%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,39,"\x007\0\x06\x85\0\0\x01\0\x01\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\r\x0c" );
Aucune correspondance exacte du système d'exploitation pour l'hôte (si vous savez quel système d'exploitation s'exécute dessus, consultez https://nmap.org/submit/ ).
Empreinte TCP/IP :
Système d'exploitation : SCAN (V = 7,92 % E = 4 % D = 6/20 % OT = 21 % CT = 1 % CU = 36505 % PV = Y% DS = 2 % DC = T% G = Y% TM = 62B0CA8
Système d'exploitation : 8 % P = x86_64-pc-linux-gnu) SEQ (SP = 104 % GCD = 1 % ISR = 10B% TI = Z% CI = Z% II = I% TS = A) OPS
Système d'exploitation :(O1=M505ST11NW7%O2=M505ST11NW7%O3=M505NNT11NW7%O4=M505ST11NW7%O5=M505ST1
Système d'exploitation : 1NW7%O6=M505ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS :(R=Y%DF=Y%T=40%W=FAF0%O=M505NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A= S+%F=A
Système d'exploitation : S % RD = 0 % Q =) T2 ( R = N ) T3 ( R = N ) T4 ( R = Y % DF = Y % T = 40 % W = 0 % S = A % A = Z % F =R%O=%RD=0%Q=)T5(R
SE :=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T =40%W=0%S=A%A=Z%F
OS :=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD= 0%Q=)U1(R=Y%DF=N%
OS : T=40 %IPL=164 %UN=0 %RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40 % CD
OS :=S)

Distance réseau : 2 sauts
Informations sur le service : hôte : ubuntu ; OS : Unix, Linux ; CPE : cpe:/o:linux:linux_kernel

TRACEROUTE (utilisant le port 443/tcp)
ADRESSE HOP RTT
1 116,63 millisecondes 10.10.14.1
2 117,72 millisecondes 10.129.203.101

Détection du système d'exploitation et du service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 84,91 secondes

```

La première chose que nous pouvons voir est qu'il s'agit d'un hôte Ubuntu exécutant un proxy HTTP quelconque. Nous pouvons utiliser ce grep Nmap pratique [cheatsheet](https://github.com/leonjza/awesome-nmap-grep) pour "couper le bruit" et extraire les informations les plus utiles de l'analyse. Extrayons les services en cours d'exécution et les numéros de service, afin que nous les ayons à portée de main pour une enquête plus approfondie.

```
dsgsec@htb[/htb]$ egrep -v "^#|Status : Up" inlanefreight_ept_tcp_all_svc.gnmap | couper -d ' ' -f4- | tr ',' '\n' |\
sed -e 's/^[ \t]*//' | awk -F '/' '{imprimer $7}' | grep-v "^$" | trier | unique -c\
| trier -k 1 -nr

       2 Pigeonnier pop3d
       2 Pigeonnier imapd (Ubuntu)
       2 Apache httpd 2.4.41 ((Ubuntu))
       1 vsftpd 3.0.3
       1 suffixe smtpd
       1 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux ; protocole 2.0)
       1 2-4 (RPC #100000)

```

À partir de ces services d'écoute, il y a plusieurs choses que nous pouvons essayer immédiatement, mais puisque nous voyons que DNS est présent, essayons un transfert de zone DNS pour voir si nous pouvons énumérer des sous-domaines valides pour une exploration plus approfondie et étendre notre portée de test. Nous savons d'après la feuille de définition que le domaine principal est `INLANEFREIGHT.LOCAL`, alors voyons ce que nous pouvons trouver.

```
dsgsec@htb[/htb]$ dig axfr inlanefreight.local @10.129.203.101

; <<>> DiG 9.16.27-Debian <<>> axfr inlanefreight.local @10.129.203.101
;; options globales : +cmd
inlanefreight.local. 86400 IN SOA ns1.inlanfreight.local. dnsadmin.inlanefreight.local. 21 604800 86400 2419200 86400
inlanefreight.local. 86400 IN NS inlanefreight.local.
inlanefreight.local. 86400 DANS UN 127.0.0.1
blog.inlanefreight.local. 86400 DANS UN 127.0.0.1
carrieres.inlanefreight.local. 86400 DANS UN 127.0.0.1
dev.inlanefreight.local. 86400 DANS UN 127.0.0.1
gitlab.inlanefreight.local. 86400 DANS UN 127.0.0.1
ir.inlanefreight.local. 86400 DANS UN 127.0.0.1
status.inlanefreight.local. 86400 DANS UN 127.0.0.1
support.inlanefreight.local. 86400 DANS UN 127.0.0.1
tracking.inlanefreight.local. 86400 DANS UN 127.0.0.1
vpn.inlanefreight.local. 86400 DANS UN 127.0.0.1
inlanefreight.local. 86400 IN SOA ns1.inlanfreight.local. dnsadmin.inlanefreight.local. 21 604800 86400 2419200 86400
;; Temps de requête : 116 ms
;; SERVEUR : 10.129.203.101#53(10.129.203.101)
;; QUAND : Lun 20 juin 16:28:20 HAE 2022
;; Taille XFR : 14 enregistrements (messages 1, octets 448)

```

Le transfert de zone fonctionne, et nous trouvons 9 sous-domaines supplémentaires. Dans un engagement réel, si un transfert de zone DNS n'est pas possible, nous pourrions énumérer les sous-domaines de plusieurs façons. Le site [DNSDumpster.com](https://dnsdumpster.com/) est un pari rapide. Le module `Information Gathering - Web Edition` répertorie plusieurs méthodes pour [Passive Subdomain Enumeration](https://academy.hackthebox.com/module/144/section/1252) et [Active Subdomain Enumeration](https://academy. hackthebox.com/module/144/section/1256).

Si le DNS n'était pas en jeu, nous pourrions également effectuer une énumération vhost à l'aide d'un outil tel que `ffuf`. Essayons ici pour voir si nous trouvons autre chose que le transfert de zone a manqué. Nous utiliserons [cette](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/namelist.txt) liste de dictionnaires pour nous aider, qui se trouve à `/opt/useful/SecLists/Discovery/DNS/namelist.txt` sur la Pwnbox.

Pour fuzzer les vhosts, nous devons d'abord déterminer à quoi ressemble la réponse d'un vhost inexistant. Nous pouvons choisir tout ce que nous voulons ici; nous voulons juste provoquer une réponse, nous devons donc choisir quelque chose qui n'existe très probablement pas.

```
dsgsec@htb[/htb]$ curl -s -I http://10.129.203.101 -H "HÔTE : defnotvalid.inlanefreight.local" | grep "Longueur du contenu :"

Longueur du contenu : 15157

```

Essayer de spécifier `defnotvalid` dans l'en-tête de l'hôte nous donne une taille de réponse de `15157`. Nous pouvons en déduire que ce sera la même chose pour tout vhost invalide. Travaillons donc avec `ffuf`, en utilisant l'indicateur `-fs` pour filtrer les réponses de taille `15157` puisque nous savons qu'elles sont invalides.

```
dsgsec@htb[/htb]$ ffuf -w namelist.txt:FUZZ -u http://10.129.203.101/ -H 'Host:FUZZ.inlanefreight.local' -fs 15157

         /'___\ /'___\ /'___\
        /\ \__/ /\ \__/ __ __ /\ \__/
        \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
         \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
          \ \_\ \ \_\ \ \____/ \ \_\
           \/_/ \/_/ \/___/ \/_/

        v1.4.1-dev
________________________________________________

  :: Méthode : GET
  :: URL : http://10.129.203.101/
  :: Liste de mots : FUZZ: namelist.txt
  :: En-tête : Hôte : FUZZ.inlanefreight.local
  :: Suivre les redirections : false
  :: Calibrage : faux
  :: Délai d'attente : 10
  :: Fils : 40
  :: Matcher : état de la réponse : 200 204 301 302 307 401 403 405 500
  :: Filtre : Taille de réponse : 15157
________________________________________________

blog [Statut : 200, Taille : 8708, Mots : 1509, Lignes : 232, Durée : 143 ms]
carrières [Statut : 200, Taille : 51810, Mots : 22044, Lignes : 732, Durée : 153ms]
dev [Statut : 200, Taille : 2048, Mots : 643, Lignes : 74, Durée : 1262 ms]
gitlab [Statut : 302, Taille : 113, Mots : 5, Lignes : 1, Durée : 226 ms]
ir [Statut : 200, Taille : 28545, Mots : 2888, Lignes : 210, Durée : 1089 ms]
<CENSURÉ> [Statut : 200, Taille : 56, Mots : 3, Lignes : 4, Durée : 120 ms]
status [Statut : 200, Taille : 917, Mots : 112, Lignes : 43, Durée : 126 ms]
support [Statut : 200, Taille : 26635, Mots : 11730, Lignes : 523, Durée : 122ms]
suivi [Statut : 200, Taille : 35185, Mots : 10409, Lignes : 791, Durée : 124 ms]
vpn [Statut : 200, Taille : 1578, Mots : 414, Lignes : 35, Durée : 121 ms]
:: Progression : [151265/151265] :: Travail [1/1] :: 341 req/sec :: Durée : [0:07:33] :: Erreurs : 0 ::

```

En comparant les résultats, nous voyons un vhost qui ne faisait pas partie des résultats du transfert de zone DNS que nous avons effectué.

* * * * *

Résultats de l'énumération
-------------------

Dès notre énumération initiale, nous avons remarqué plusieurs ports intéressants ouverts que nous approfondirons dans la section suivante. Nous avons également rassemblé plusieurs sous-domaines/vhosts. Ajoutons-les à notre fichier `/etc/hosts` afin que nous puissions étudier chacun plus en détail.

```
dsgsec@htb[/htb]$ sudo tee -a /etc/hosts > /dev/null <<EOT

## hôtes inlanefreight
10.129.203.101 inlanefreight.local blog.inlanefreight.local careers.inlanefreight.local dev.inlanefreight.local gitlab.inlanefreight.local ir.inlanefreight.local status.inlanefreight.local support.inlanefreight.local tracking.inlanefreight.local vpn.inlanefreight. local
EOT

```

Dans la section suivante, nous approfondirons les résultats de l'analyse Nmap et verrons si nous pouvons trouver des services directement exploitables ou mal configurés.