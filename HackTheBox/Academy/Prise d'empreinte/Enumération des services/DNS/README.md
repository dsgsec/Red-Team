# Enumération des services

## DNS

Le système de noms de domaine (DNS) fait partie intégrante d'Internet. Par exemple, via des noms de domaine, tels que academy.hackthebox.com ou www.hackthebox.com, nous pouvons atteindre les serveurs Web auxquels le fournisseur d'hébergement a attribué une ou plusieurs adresses IP spécifiques. Le DNS est un système de résolution des noms d'ordinateurs en adresses IP, et il n'a pas de base de données centrale. Simplifié, on peut l'imaginer comme une bibliothèque avec de nombreux annuaires téléphoniques différents. Les informations sont distribuées sur plusieurs milliers de serveurs de noms. Les serveurs DNS distribués à l'échelle mondiale traduisent les noms de domaine en adresses IP et contrôlent ainsi le serveur auquel un utilisateur peut accéder via un domaine particulier. Il existe plusieurs types de serveurs DNS utilisés dans le monde :

- Serveur racine DNS
- Serveur de noms faisant autorité
- Serveur de noms ne faisant pas autorité
- Serveur de cache
- Serveur de transfert
- Résolveur

| Type de serveur | Descriptif |
| --- | --- |
| `Serveur racine DNS` | Les serveurs racine du DNS sont responsables des domaines de premier niveau (`TLD`). En dernier lieu, ils ne sont demandés que si le serveur de noms ne répond pas. Ainsi, un serveur racine est une interface centrale entre les utilisateurs et le contenu sur Internet, car il relie le domaine et l'adresse IP. La [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/)(`ICANN`) coordonne le travail des serveurs de noms racine. Il existe `13` de tels serveurs racine dans le monde. |
| `Serveur de noms faisant autorité` | Les serveurs de noms faisant autorité détiennent l'autorité pour une zone particulière. Ils ne répondent qu'aux questions relevant de leur domaine de responsabilité et leurs informations sont contraignantes. Si un serveur de noms faisant autorité ne peut pas répondre à la requête d'un client, le serveur de noms racine prend le relais à ce stade. |
| `Serveur de noms ne faisant pas autorité` | Les serveurs de noms ne faisant pas autorité ne sont pas responsables d'une zone DNS particulière. Au lieu de cela, ils collectent eux-mêmes des informations sur des zones DNS spécifiques, ce qui se fait à l'aide d'une requête DNS récursive ou itérative. |
| `Mise en cache du serveur DNS` | Mise en cache Les serveurs DNS mettent en cache les informations d'autres serveurs de noms pendant une période spécifiée. Le serveur de noms faisant autorité détermine la durée de ce stockage. |
| `Serveur de transfert` | Les serveurs de transfert n'exécutent qu'une seule fonction: ils transmettent les requêtes DNS à un autre serveur DNS. |
| `Résolveur` | Les résolveurs ne sont pas des serveurs DNS faisant autorité, mais effectuent la résolution de noms localement sur l'ordinateur ou le routeur. |

![image](./ressources/tooldev-dns.png)

Différents enregistrements DNS sont utilisés pour les requêtes DNS, qui ont toutes des tâches différentes. De plus, des entrées distinctes existent pour différentes fonctions puisque nous pouvons configurer des serveurs de messagerie et d'autres serveurs pour un domaine.

| Enregistrement DNS | Descriptif |
| --- | --- |
| `A` | Renvoie une adresse IPv4 du domaine demandé en conséquence. |
| `AAA` | Renvoie une adresse IPv6 du domaine demandé. |
| `MX` | Renvoie les serveurs de messagerie responsables en conséquence. |
| `NS` | Renvoie les serveurs DNS (serveurs de noms) du domaine. |
| `TXT` | Cet enregistrement peut contenir diverses informations. Le polyvalent peut être utilisé, par exemple, pour valider la Google Search Console ou valider les certificats SSL. De plus, les entrées SPF et DMARC sont définies pour valider le trafic de messagerie et le protéger du spam. |
| `CNAME` | Cet enregistrement sert d'alias. Si le domaine www.hackthebox.eu doit pointer vers la même IP, et nous créons un enregistrement A pour l'un et un enregistrement CNAME pour l'autre. |
| `PTR` | L'enregistrement PTR fonctionne dans l'autre sens (recherche inversée). Il convertit les adresses IP en noms de domaine valides. |
| `SOA` | Fournit des informations sur la zone DNS correspondante et l'adresse e-mail du contact administratif. |

L'enregistrement SOA se trouve dans le fichier de zone d'un domaine et spécifie qui est responsable du fonctionnement du domaine et comment les informations DNS du domaine sont gérées.

```
dsgsec@htb[/htb]$ dig soa www.inlanefreight.com

; <<>> DiG 9.16.27-Debian <<>> soa www.inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15876
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.inlanefreight.com.         IN      SOA

;; AUTHORITY SECTION:
inlanefreight.com.      900     IN      SOA     ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

;; Query time: 16 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jan 05 12:56:10 GMT 2023
;; MSG SIZE  rcvd: 128
```

**Le point (.) est remplacé par un arobase (@) dans l'adresse e-mail. Dans cet exemple, l'adresse e-mail de l'administrateur est awsdns-hostmaster@amazon.com.**

## Configuration par défaut

Il existe de nombreux types de configuration différents pour le DNS. Nous n'aborderons donc que les plus importantes pour mieux illustrer le principe de fonctionnement d'un point de vue administratif. Tous les serveurs DNS fonctionnent avec trois types différents de fichiers de configuration:

- fichiers de configuration DNS locaux
- fichiers de zones
- fichiers de résolution de noms inversés

## Prise d'empreinte DNS

L'empreinte sur les serveurs DNS se fait à la suite des requêtes que nous envoyons. Ainsi, tout d'abord, le serveur DNS peut être interrogé pour savoir quels autres serveurs de noms sont connus. Nous le faisons en utilisant l'enregistrement NS et la spécification du serveur DNS que nous voulons interroger en utilisant le caractère @. En effet, s'il existe d'autres serveurs DNS, nous pouvons également les utiliser et interroger les enregistrements. Cependant, d'autres serveurs DNS peuvent être configurés différemment et, en plus, peuvent être permanents pour d'autres zones.

```
dsgsec@htb[/htb]$ dig ns inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> ns inlanefreight.htb @10.129.14.128
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45010
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: ce4d8681b32abaea0100000061475f73842c401c391690c7 (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      NS

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:04:03 CEST 2021
;; MSG SIZE  rcvd: 107
```
## DIG - Version Query

```
dsgsec@htb[/htb]$ dig CH TXT version.bind 10.129.120.85

; <<>> DiG 9.10.6 <<>> CH TXT version.bind
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47786
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; ANSWER SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1"

;; ADDITIONAL SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1-Debian"

;; Query time: 2 msec
;; SERVER: 10.129.120.85#53(10.129.120.85)
;; WHEN: Wed Jan 05 20:23:14 UTC 2023
;; MSG SIZE  rcvd: 101
```
Nous pouvons utiliser l'option ANY pour afficher tous les enregistrements disponibles. Cela amènera le serveur à nous montrer toutes les entrées disponibles qu'il est prêt à divulguer. Il est important de noter que toutes les entrées des zones ne seront pas affichées.

## DIG - ANY Query
```
dsgsec@htb[/htb]$ dig any inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> any inlanefreight.htb @10.129.14.128
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7649
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 064b7e1f091b95120100000061476865a6026d01f87d10ca (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      ANY

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:42:13 CEST 2021
;; MSG SIZE  rcvd: 437
```

## Transfert de zone

Le transfert de zone fait référence au transfert de zones vers un autre serveur dans DNS, qui se produit généralement via le port TCP 53. Cette procédure est abrégée Zone de transfert complet asynchrone (AXFR). Étant donné qu'une panne DNS a généralement de graves conséquences pour une entreprise, le fichier de zone est presque invariablement conservé à l'identique sur plusieurs serveurs de noms. Lorsque des modifications sont apportées, il faut s'assurer que tous les serveurs ont les mêmes données. La synchronisation entre les serveurs concernés est réalisée par transfert de zone. A l'aide d'une clé secrète rndc-key, que nous avons vu initialement dans la configuration par défaut, les serveurs s'assurent qu'ils communiquent avec leur propre maître ou esclave. Le transfert de zone implique le simple transfert de fichiers ou d'enregistrements et la détection de divergences dans les ensembles de données des serveurs concernés.

Les données d'origine d'une zone se trouvent sur un serveur DNS, appelé serveur de noms primaire pour cette zone. Cependant, pour augmenter la fiabilité, réaliser une simple répartition de charge, ou protéger le primaire des attaques, on installe en pratique dans la quasi-totalité des cas un ou plusieurs serveurs supplémentaires, appelés serveurs de noms secondaires pour cette zone. Pour certains domaines de premier niveau (TLD), il est obligatoire de rendre les fichiers de zone des domaines de second niveau accessibles sur au moins deux serveurs.

Les entrées DNS ne sont généralement créées, modifiées ou supprimées que sur le serveur principal. Cela peut être fait en éditant manuellement le fichier de zone concerné ou automatiquement par une mise à jour dynamique à partir d'une base de données. Un serveur DNS qui sert de source directe pour synchroniser un fichier de zone est appelé un maître. Un serveur DNS qui obtient des données de zone d'un maître est appelé un esclave. Un primaire est toujours un maître, tandis qu'un secondaire peut être à la fois un esclave et un maître.

L'esclave récupère l'enregistrement SOA de la zone concernée auprès du maître à certains intervalles, le soi-disant temps de rafraîchissement, généralement une heure, et compare les numéros de série. Si le numéro de série de l'enregistrement SOA du maître est supérieur à celui de l'esclave, les jeux de données ne correspondent plus.

## DIG - AXFR Zone Transfer

```
dsgsec@htb[/htb]$ dig axfr inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr inlanefreight.htb @10.129.14.128
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 4 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:51:19 CEST 2021
;; XFR size: 9 records (messages 1, bytes 520)
```

Si l'administrateur utilisait un sous-réseau pour l'option d'autorisation de transfert à des fins de test ou comme solution de contournement ou le définissait sur n'importe lequel, tout le monde interrogerait l'intégralité du fichier de zone sur le serveur DNS. De plus, d'autres zones peuvent être interrogées, qui peuvent même afficher des adresses IP internes et des noms d'hôte.

## DIG - AXFR Zone Transfer - Internal
```
dsgsec@htb[/htb]$ dig axfr internal.inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr internal.inlanefreight.htb @10.129.14.128
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN   TXT     "MS=ms97310371"
internal.inlanefreight.htb. 604800 IN   TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN   TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN   NS      ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb. 604800 IN A     10.129.34.16
dc2.internal.inlanefreight.htb. 604800 IN A     10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A   10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A      10.129.34.136
vpn.internal.inlanefreight.htb. 604800 IN A     10.129.1.6
ws1.internal.inlanefreight.htb. 604800 IN A     10.129.1.34
ws2.internal.inlanefreight.htb. 604800 IN A     10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A    10.129.18.2
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:53:11 CEST 2021
;; XFR size: 15 records (messages 1, bytes 664)
```
Les enregistrements A individuels avec les noms d'hôte peuvent également être découverts à l'aide d'une attaque par force brute. Pour ce faire, nous avons besoin d'une liste de noms d'hôtes possibles, que nous utilisons pour envoyer les demandes dans l'ordre. De telles listes sont fournies, par exemple, par SecLists.

Une option serait d'exécuter une boucle for dans Bash qui répertorie ces entrées et envoie la requête correspondante au serveur DNS souhaité.

## Bruteforce de sous-domaines
```
dsgsec@htb[/htb]$ for sub in $(cat /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
```

De nombreux outils différents peuvent être utilisés pour cela, et la plupart d'entre eux fonctionnent de la même manière. L'un de ces outils est par exemple DNSenum.

```
dsgsec@htb[/htb]$ dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb

dnsenum VERSION:1.2.6

-----   inlanefreight.htb   -----


Host's addresses:
__________________



Name Servers:
______________

ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136


Mail (MX) Servers:
___________________



Trying Zone Transfers and getting Bind Versions:
_________________________________________________

unresolvable name: ns.inlanefreight.htb at /usr/bin/dnsenum line 900 thread 1.

Trying Zone Transfer for inlanefreight.htb on ns.inlanefreight.htb ...
AXFR record query failed: no nameservers


Brute forcing with /home/cry0l1t3/Pentesting/SecLists/Discovery/DNS/subdomains-top1million-110000.txt:
_______________________________________________________________________________________________________

ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136
mail1.inlanefreight.htb.                 604800   IN    A        10.129.18.201
app.inlanefreight.htb.                   604800   IN    A        10.129.18.15
ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136

...SNIP...
done.
```