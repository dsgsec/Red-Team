Outil intégrés
=======


Cette tâche porte sur :

-   `whois`
-   `dig`, `nslookup`,`host`
-   `traceroute`/`tracert`

Avant de commencer à utiliser l' `whois` outil, regardons WHOIS. WHOIS est un protocole de demande et de réponse qui suit la spécification [RFC 3912 ](https://www.ietf.org/rfc/rfc3912.txt). Un serveur WHOIS écoute sur le port TCP 43 les requêtes entrantes. Le bureau d'enregistrement de domaine est responsable de la maintenance des enregistrements WHOIS pour les noms de domaine qu'il loue. `whois`interrogera le serveur WHOIS pour fournir tous les enregistrements sauvegardés. Dans l'exemple suivant, nous pouvons voir `whois`nous fournit:

-   Serveur WHOIS du bureau d'enregistrement
-   URL du bureau d'enregistrement
-   Date de création de l'enregistrement
-   Date de mise à jour de l'enregistrement
-   Coordonnées et adresse du titulaire (sauf si elles ne sont pas divulguées pour des raisons de confidentialité)
-   Coordonnées et adresse de l'administrateur (sauf si elles ne sont pas divulguées pour des raisons de confidentialité)
-   Coordonnées et adresse du technicien (sauf si elles ne sont pas divulguées pour des raisons de confidentialité)

Terminal Pentesteur

```
pentester@TryHackMe$ whois thmredteam.com
[Querying whois.verisign-grs.com]
[Redirected to whois.namecheap.com]
[Querying whois.namecheap.com]
[whois.namecheap.com]
Domain name: thmredteam.com
Registry Domain ID: 2643258257_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.namecheap.com
Registrar URL: http://www.namecheap.com
Updated Date: 0001-01-01T00:00:00.00Z
Creation Date: 2021-09-24T14:04:16.00Z
Registrar Registration Expiration Date: 2022-09-24T14:04:16.00Z
Registrar: NAMECHEAP INC
Registrar IANA ID: 1068
Registrar Abuse Contact Email: abuse@namecheap.com
Registrar Abuse Contact Phone: +1.6613102107
Reseller: NAMECHEAP INC
Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
Registry Registrant ID:
Registrant Name: Withheld for Privacy Purposes
Registrant Organisation: Privacy service provided by Withheld for Privacy ehf
Registrant Street: Kalkofnsvegur 2
Registrant City: Reykjavik
Registrant State/Province: Capital Region
Registrant Postal Code: 101
Registrant Country: IS
Registrant Phone: +354.4212434
Registrant Phone Ext:
Registrant Fax:
Registrant Fax Ext:
Registrant Email: 4c9d5617f14e4088a4396b2f25430925.protect@withheldforprivacy.com
Registry Admin ID:
Admin Name: Withheld for Privacy Purposes
[...]
Tech Name: Withheld for Privacy Purposes
[...]
Name Server: kip.ns.cloudflare.comName Server: uma.ns.cloudflare.com
DNSSEC: unsigned
URL of the ICANN WHOIS Data Problem Reporting System: http://wdprs.internic.net/
>>> Last update of WHOIS database: 2021-10-13T10:42:40.11Z <<<
For more information on Whois status codes, please visit https://icann.org/epp
```

Comme nous pouvons le voir ci-dessus, il est possible d'obtenir beaucoup d'informations précieuses avec seulement un nom de domaine. Après une `whois`recherche, nous pourrions avoir de la chance et trouver des noms, des adresses e-mail, des adresses postales et des numéros de téléphone, en plus d'autres informations techniques. À la fin de la `whois`requête, nous trouvons les serveurs de noms faisant autorité pour le domaine en question.

Les requêtes DNS peuvent être exécutées avec de nombreux outils différents trouvés sur nos systèmes, en particulier les systèmes de type Unix. Un outil commun trouvé sur les systèmes de type Unix, Windows et macOS est`nslookup` . Dans la requête suivante, nous pouvons voir comment`nslookup` utilise le serveur DNS par défaut pour obtenir les enregistrements A et AAAA liés à notre domaine.

Terminal Pentesteur

```
pentester@TryHackMe$ nslookup cafe.thmredteam.com
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	cafe.thmredteam.com
Address: 104.21.93.169
Name:	cafe.thmredteam.com
Address: 172.67.212.249
Name:	cafe.thmredteam.com
Address: 2606:4700:3034::ac43:d4f9
Name:	cafe.thmredteam.com
Address: 2606:4700:3034::6815:5da9
```

Un autre outil couramment trouvé sur les systèmes de type Unix est `dig`, abréviation de Domain Information Groper (dig). `dig`fournit de nombreuses options de requête et vous permet même de spécifier un serveur DNS différent à utiliser. Par exemple, nous pouvons utiliser le serveur DNS de Cloudflare :`dig @1.1.1.1 tryhackme.com` .

Terminal Pentesteur

```
pentester@TryHackMe$ dig cafe.thmredteam.com @1.1.1.1

; <<>> DiG 9.16.21-RH <<>> cafe.thmredteam.com @1.1.1.1
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 16698
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;cafe.thmredteam.com.		IN	A

;; ANSWER SECTION:
cafe.thmredteam.com.	3114	IN	A	104.21.93.169
cafe.thmredteam.com.	3114	IN	A	172.67.212.249

;; Query time: 4 msec
;; SERVER: 1.1.1.1#53(1.1.1.1)
;; WHEN: Thu Oct 14 10:44:11 EEST 2021
;; MSG SIZE  rcvd: 80
```

`host`est une autre alternative utile pour interroger les serveurs DNS pour les enregistrements DNS. Prenons l'exemple suivant.

Terminal Pentesteur

```
pentester@TryHackMe$ host cafe.thmredteam.com
cafe.thmredteam.com has address 172.67.212.249
cafe.thmredteam.com has address 104.21.93.169
cafe.thmredteam.com has IPv6 address 2606:4700:3034::ac43:d4f9
cafe.thmredteam.com has IPv6 address 2606:4700:3034::6815:5da9
```

Le dernier outil fourni avec les systèmes de type Unix est `traceroute`, ou sur les systèmes MS Windows, `tracert`. Comme son nom l'indique, il trace la route empruntée par les paquets de notre système vers l'hôte cible. La sortie de la console ci-dessous montre que `traceroute`nous avons fourni les routeurs (sauts) nous connectant au système cible. Il convient de souligner que certains routeurs ne répondent pas aux paquets envoyés par `traceroute`, et par conséquent, nous ne voyons pas leurs adresses IP ; a `*`est utilisé pour indiquer un tel cas.

Terminal Pentesteur

```
pentester@TryHackMe$ traceroute cafe.thmredteam.com
traceroute to cafe.thmredteam.com (172.67.212.249), 30 hops max, 60 byte packets
 1  _gateway (192.168.0.1)  3.535 ms  3.450 ms  3.398 ms
 2  * * *
 3  * * *
 4  * * *
 5  * * *
 6  * * *
 7  172.16.79.229 (172.16.79.229)  4.663 ms  6.417 ms  6.347 ms
 8  * * *
 9  172.16.49.1 (172.16.49.1)  6.688 ms 172.16.48.1 (172.16.48.1)  6.671 ms 172.16.49.1 (172.16.49.1)  6.651 ms
10  213.242.116.233 (213.242.116.233)  96.769 ms 81.52.187.243 (81.52.187.243)  96.634 ms  96.614 ms
11  bundle-ether302.pastr4.paris.opentransit.net (193.251.131.116)  96.592 ms  96.689 ms  96.671 ms
12  193.251.133.251 (193.251.133.251)  96.679 ms  96.660 ms  72.465 ms
13  193.251.150.10 (193.251.150.10)  72.392 ms 172.67.212.249 (172.67.212.249)  91.378 ms  91.306 ms
```

En résumé, on peut toujours compter sur :

-   `whois`pour interroger la base de données WHOIS
-   `nslookup`, `dig`, ou `host`pour interroger les serveurs DNS

Les bases de données WHOIS et les serveurs DNS contiennent des informations accessibles au public, et les interroger ne génèrent aucun trafic suspect.

De plus, nous pouvons compter sur Traceroute ( `traceroute`sur les systèmes Linux et macOS et`tracert` sur les systèmes MS Windows) pour découvrir les sauts entre notre système et l'hôte cible.
