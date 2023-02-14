# DNS

## C'est quoi le DNS

Le DNS est l'annuaire téléphonique d'Internet. Les noms de domaine tels que hackthebox.com et inlanefreight.com permettent aux gens d'accéder au contenu sur Internet. Les adresses IP (Internet Protocol) sont utilisées pour communiquer entre les navigateurs Web. Le DNS convertit les noms de domaine en adresses IP, permettant aux navigateurs d'accéder aux ressources sur Internet.

Chaque appareil connecté à Internet possède une adresse IP unique que d'autres machines utilisent pour le localiser. Les serveurs DNS minimisent le besoin pour les gens d'apprendre des adresses IP comme 104.17.42.72 en IPv4 ou des adresses IP alphanumériques modernes plus sophistiquées comme 2606:4700::6811:2b48 en IPv6. Lorsqu'un utilisateur tape www.facebook.com dans son navigateur Web, une traduction doit se produire entre ce que l'utilisateur tape et l'adresse IP requise pour accéder à la page Web www.facebook.com.

Certains des avantages de l'utilisation de DNS sont :

+ Il permet d'utiliser des noms à la place des numéros pour identifier les hôtes.
+ Il est beaucoup plus facile de se souvenir d'un nom que de se souvenir d'un numéro.
+ En redirigeant simplement un nom vers la nouvelle adresse numérique, un serveur peut modifier les adresses numériques sans avoir à avertir tout le monde sur Internet.
+ Un seul nom peut faire référence à plusieurs hôtes répartissant la charge de travail entre différents serveurs.
Il existe une hiérarchie de noms dans la structure DNS. La racine du système, ou le niveau le plus élevé, n'a pas de nom.

Les serveurs de noms TLD, les domaines de premier niveau, peuvent être comparés à une seule étagère de livres dans une bibliothèque. La dernière partie d'un nom d'hôte est hébergée par ce serveur de noms, qui est l'étape suivante dans la recherche d'une adresse IP spécifique (dans www.facebook.com, le serveur TLD est com). La plupart des TLD ont été délégués à des gestionnaires de pays individuels, qui reçoivent des codes du tableau ISO-3166-1. Ceux-ci sont connus sous le nom de domaines de premier niveau de code de pays ou ccTLD gérés par une agence des Nations Unies.

Il existe également un petit nombre de domaines de premier niveau "génériques" (gTLD) qui ne sont pas associés à un pays ou à une région spécifique. Les gestionnaires de TLD se sont vu confier la responsabilité des procédures et des politiques d'attribution des noms de domaine de second niveau (SLD) et des hiérarchies de noms de niveau inférieur, conformément aux conseils de politique spécifiés dans la norme ISO-3166-1.

Un gestionnaire pour chaque nation organise les domaines de code de pays. Ces gestionnaires assurent un service public pour le compte de la communauté Internet. Les enregistrements de ressources sont les résultats des requêtes DNS et ont la structure suivante :

| |
| --- |
| `Enregistrement de ressource` | Un nom de domaine, généralement un nom de domaine complet, est la première partie d'un enregistrement de ressource. Si vous n'utilisez pas de nom de domaine complet, le nom de la zone où se trouve l'enregistrement sera ajouté à la fin du nom. |
| `TTL` | En secondes, la durée de vie ("TTL") est par défaut la valeur minimale spécifiée dans l'enregistrement SOA. |
| `Classe d'enregistrement` | Internet, Hésiode ou Chaos |
| `Début de l'autorité` (`SOA`) | Il doit être le premier dans un fichier de zone car il indique le début d'une zone. Chaque zone ne peut avoir qu'un seul enregistrement `SOA` et contient en outre les valeurs de la zone, telles qu'un numéro de série et plusieurs délais d'expiration. |
| `Serveurs de noms` (`NS`) | La base de données distribuée est liée par `NS` Records. Ils sont en charge du serveur de noms faisant autorité d'une zone et de l'autorité d'une zone enfant sur un serveur de noms. |
| `Adresses IPv4` (`A`) | L'enregistrement A n'est qu'un mappage entre un nom d'hôte et une adresse IP. Les zones "Transférer" sont celles avec des enregistrements `A`. |
| `Pointeur` (`PTR`) | L'enregistrement PTR est un mappage entre une adresse IP et un nom d'hôte. Les zones "inversées" sont celles qui ont des enregistrements `PTR`. |
| `Nom canonique` (`CNAME`) | Un nom d'hôte d'alias est mappé à un nom d'hôte d'enregistrement `A` à l'aide de l'enregistrement `CNAME`. |
| `Échange de messagerie` (`MX`) | L'enregistrement `MX` identifie un hôte qui acceptera les e-mails d'un hôte spécifique. Une valeur de priorité a été affectée à l'hôte spécifié. Plusieurs enregistrements MX peuvent exister sur le même hôte et une liste hiérarchisée est constituée des enregistrements d'un hôte spécifique. |

## Nslookup & DIG
Maintenant que nous avons une compréhension claire de ce qu'est le DNS, examinons l'utilitaire de ligne de commande Nslookup. Supposons qu'un client nous demande d'effectuer un test d'intrusion externe. Par conséquent, nous devons d'abord nous familiariser avec leur infrastructure et identifier les hébergeurs accessibles au public. Nous pouvons le découvrir en utilisant différents types de requêtes DNS. Avec Nslookup, nous pouvons rechercher des serveurs de noms de domaine sur Internet et leur demander des informations sur les hôtes et les domaines. Bien que l'outil dispose de deux modes, interactif et non interactif, nous nous concentrerons principalement sur le module non interactif.

Nous pouvons interroger les enregistrements A en soumettant simplement un nom de domaine. Mais nous pouvons également utiliser le paramètre -query pour rechercher des enregistrements de ressources spécifiques. Certains exemples sont:

### Querying: A Records
```
dsgsec@htb[/htb]$ export TARGET="facebook.com"
dsgsec@htb[/htb]$ nslookup $TARGET

Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
Name:	facebook.com
Address: 31.13.92.36
Name:	facebook.com
Address: 2a03:2880:f11c:8083:face:b00c:0:25de
```

Nous pouvons également spécifier un serveur de noms si nécessaire en ajoutant @<nameserver/IP> à la commande. Contrairement à nslookup, DIG nous montre quelques informations supplémentaires qui peuvent être importantes.

```
dsgsec@htb[/htb]$ dig facebook.com @1.1.1.1

; <<>> DiG 9.16.1-Ubuntu <<>> facebook.com @1.1.1.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 58899
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;facebook.com.                  IN      A

;; ANSWER SECTION:
facebook.com.           169     IN      A       31.13.92.36

;; Query time: 20 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mo Okt 18 16:03:17 CEST 2021
;; MSG SIZE  rcvd: 57
```

### Requête: Enregistrement A pour un sous-domaine
```
dsgsec@htb[/htb]$ export TARGET=www.facebook.com
dsgsec@htb[/htb]$ nslookup -query=A $TARGET

Server:		1.1.1.1
Address:	1.1.1.1#53

Non-authoritative answer:
www.facebook.com	canonical name = star-mini.c10r.facebook.com.
Name:	star-mini.c10r.facebook.com
Address: 31.13.92.36
```

```
dsgsec@htb[/htb]$ dig a www.facebook.com @1.1.1.1

; <<>> DiG 9.16.1-Ubuntu <<>> a www.facebook.com @1.1.1.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15596
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;www.facebook.com.              IN      A

;; ANSWER SECTION:
www.facebook.com.       3585    IN      CNAME   star-mini.c10r.facebook.com.
star-mini.c10r.facebook.com. 45 IN      A       31.13.92.36

;; Query time: 16 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Mo Okt 18 16:11:48 CEST 2021
;; MSG SIZE  rcvd: 90
```
