# Enumération des services

## SMTP

### Introduction

Le protocole SMTP (Simple Mail Transfer Protocol) est un protocole d'envoi d'e-mails sur un réseau IP. Il peut être utilisé entre un client de messagerie et un serveur de messagerie sortant ou entre deux serveurs SMTP. SMTP est souvent combiné avec les protocoles IMAP ou POP3, qui peuvent récupérer des e-mails et envoyer des e-mails. En principe, il s'agit d'un protocole client-serveur, bien que SMTP puisse être utilisé entre un client et un serveur et entre deux serveurs SMTP. Dans ce cas, un serveur agit effectivement en tant que client.

Par défaut, les serveurs SMTP acceptent les demandes de connexion sur le port 25. Cependant, les nouveaux serveurs SMTP utilisent également d'autres ports tels que le port TCP 587. Ce port est utilisé pour recevoir le courrier des utilisateurs/serveurs authentifiés, en utilisant généralement la commande STARTTLS pour changer le texte en clair existant. connexion à une connexion cryptée. Les données d'authentification sont protégées et ne sont plus visibles en clair sur le réseau. Au début de la connexion, l'authentification se produit lorsque le client confirme son identité avec un nom d'utilisateur et un mot de passe. Les e-mails peuvent alors être transmis. À cette fin, le client envoie au serveur les adresses de l'expéditeur et du destinataire, le contenu de l'e-mail et d'autres informations et paramètres. Une fois l'e-mail transmis, la connexion est de nouveau interrompue. Le serveur de messagerie commence alors à envoyer l'e-mail à un autre serveur SMTP.

SMTP fonctionne sans cryptage sans autre mesure et transmet toutes les commandes, données ou informations d'authentification en texte brut. Pour empêcher la lecture non autorisée des données, le SMTP est utilisé en conjonction avec le cryptage SSL/TLS. Dans certaines circonstances, un serveur utilise un port autre que le port TCP standard 25 pour la connexion chiffrée, par exemple, le port TCP 465.

Une fonction essentielle d'un serveur SMTP est d'empêcher le spam en utilisant des mécanismes d'authentification qui permettent uniquement aux utilisateurs autorisés d'envoyer des e-mails. À cette fin, la plupart des serveurs SMTP modernes prennent en charge l'extension de protocole ESMTP avec SMTP-Auth. Après avoir envoyé son e-mail, le client SMTP, également appelé Mail User Agent (MUA), le convertit en un en-tête et un corps et les télécharge tous les deux sur le serveur SMTP. Celui-ci dispose d'un agent de transfert de courrier (MTA), la base logicielle pour l'envoi et la réception d'e-mails. Le MTA vérifie la taille et le spam de l'e-mail, puis le stocke. Pour soulager le MTA, il est parfois précédé d'un Mail Submission Agent (MSA), qui vérifie la validité, c'est-à-dire l'origine du courrier électronique. Ce MSA est également appelé serveur relais. Celles-ci sont très importantes plus tard, car la soi-disant attaque par relais ouvert peut être effectuée sur de nombreux serveurs SMTP en raison d'une configuration incorrecte. Nous discuterons de cette attaque et de la manière d'en identifier le point faible un peu plus tard. Le MTA recherche alors dans le DNS l'adresse IP du serveur de messagerie destinataire.

A leur arrivée sur le serveur SMTP de destination, les paquets de données sont réassemblés pour former un e-mail complet. De là, l'agent de distribution du courrier (MDA) le transfère dans la boîte aux lettres du destinataire.

Client (MUA)	➞	Submission Agent (MSA)	➞	Open Relay (MTA)	➞	Mail Delivery Agent (MDA)	➞	Mailbox (POP3/IMAP)

### Utilisation de SMTP

#### Telnet - HELO/EHLO
Pour interagir avec le serveur SMTP, nous pouvons utiliser l'outil telnet pour initialiser une connexion TCP avec le serveur SMTP. L'initialisation proprement dite de la session se fait avec la commande mentionnée ci-dessus, HELO ou EHLO.
```
dsgsec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 


HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb


EHLO mail1

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```
#### Telnet - VRFY
La commande VRFY peut être utilisée pour énumérer les utilisateurs existants sur le système. Cependant, cela ne fonctionne pas toujours. Selon la configuration du serveur SMTP, le serveur SMTP peut émettre le code 252 et confirmer l'existence d'un utilisateur qui n'existe pas sur le système. Une liste de tous les codes de réponse SMTP peut être trouvée ici.

```
sgsec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 

VRFY root

252 2.0.0 root


VRFY cry0l1t3

252 2.0.0 cry0l1t3


VRFY testuser

252 2.0.0 testuser


VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa

252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```


####  Envoyer un mail

```
dsgsec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


EHLO inlanefreight.htb

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING


MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.

250 2.0.0 Ok: queued as 6E1CF1681AB


QUIT

221 2.0.0 Bye
Connection closed by foreign host.
```
### Prise d'empreinte

Les scripts Nmap par défaut incluent les commandes smtp, qui utilisent la commande EHLO pour répertorier toutes les commandes possibles pouvant être exécutées sur le serveur SMTP cible.

#### Nmap
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -sC -sV -p25

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-27 17:56 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00025s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

#### Nmap open-relay
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-30 02:29 CEST
NSE: Loaded 1 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.00s elapsed
Initiating ARP Ping Scan at 02:29
Scanning 10.129.14.128 [1 port]
Completed ARP Ping Scan at 02:29, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 02:29
Completed Parallel DNS resolution of 1 host. at 02:29, 0.03s elapsed
Initiating SYN Stealth Scan at 02:29
Scanning 10.129.14.128 [1 port]
Discovered open port 25/tcp on 10.129.14.128
Completed SYN Stealth Scan at 02:29, 0.06s elapsed (1 total ports)
NSE: Script scanning 10.129.14.128.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.07s elapsed
Nmap scan report for 10.129.14.128
Host is up (0.00020s latency).

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-open-relay: Server is an open relay (16/16 tests)
|  MAIL FROM:<> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@nmap.scanme.org> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@ESMTP> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest%nmap.scanme.org@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest%nmap.scanme.org@ESMTP>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org">
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest%nmap.scanme.org">
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org"@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org@ESMTP>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<@[10.129.14.128]:relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<@ESMTP:relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@[10.129.14.128]>
|_ MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@ESMTP>
MAC Address: 00:00:00:00:00:00 (VMware)

NSE: Script Post-scanning.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds
           Raw packets sent: 2 (72B) | Rcvd: 2 (72B)
```

#### Scan users
```
smtp-user-enum -U footprinting-wordlist.txt -t 10.129.14.248 -w 25 -M VRFY
```