DNS attaquant
=============

* * * * *

Le [Domain Name System](https://www.cloudflare.com/learning/dns/what-is-dns/) (`DNS`) traduit les noms de domaine (par exemple, hackthebox.com) en adresses IP numériques (par exemple, , 104.17.42.72). DNS est principalement `UDP/53`, mais DNS s'appuiera davantage sur `TCP/53` au fil du temps. DNS a toujours été conçu pour utiliser à la fois les ports UDP et TCP 53 dès le départ, UDP étant la valeur par défaut, et revient à l'utilisation de TCP lorsqu'il ne peut pas communiquer sur UDP, généralement lorsque la taille du paquet est trop grande pour être transmise en un seul Paquet UDP. Étant donné que presque toutes les applications réseau utilisent le DNS, les attaques contre les serveurs DNS représentent l'une des menaces les plus répandues et les plus importantes aujourd'hui.

* * * * *

Énumération
-----------

Le DNS contient des informations intéressantes pour une organisation. Comme indiqué dans la section Informations sur le domaine du [module Footprinting](https://academy.hackthebox.com/course/preview/footprinting), nous pouvons comprendre le fonctionnement d'une entreprise et les services qu'elle fournit, ainsi que les tiers fournisseurs de services comme les e-mails.

Les options Nmap `-sC` (scripts par défaut) et `-sV` (analyse de version) peuvent être utilisées pour effectuer une énumération initiale par rapport aux serveurs DNS cibles :

```
dsgsec@htb[/htb]# nmap -p53 -Pn -sV -sC 10.10.110.213

À partir de Nmap 7.80 ( https://nmap.org ) au 2020-10-29 03:47 EDT
Rapport d'analyse Nmap pour 10.10.110.213
L'hôte est actif (latence de 0,017 s).

VERSION SERVICE À L'ÉTAT DU PORT
53/tcp domaine ouvert ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)

```

* * * * *

Transfert de zone DNS
-----------------

Une zone DNS est une partie de l'espace de noms DNS géré par une organisation ou un administrateur spécifique. Étant donné que DNS comprend plusieurs zones DNS, les serveurs DNS utilisent les transferts de zone DNS pour copier une partie de leur base de données vers un autre serveur DNS. À moins qu'un serveur DNS ne soit configuré correctement (limitant les adresses IP pouvant effectuer un transfert de zone DNS), n'importe qui peut demander à un serveur DNS une copie de ses informations de zone puisque les transferts de zone DNS ne nécessitent aucune authentification. De plus, le service DNS s'exécute généralement sur un port UDP ; cependant, lors de l'exécution du transfert de zone DNS, il utilise un port TCP pour une transmission de données fiable.

Un attaquant pourrait exploiter cette vulnérabilité de transfert de zone DNS pour en savoir plus sur l'espace de noms DNS de l'organisation cible, augmentant ainsi la surface d'attaque. Pour l'exploitation, nous pouvons utiliser l'utilitaire `dig` avec l'option de type de requête DNS `AXFR` pour vider l'intégralité des espaces de noms DNS d'un serveur DNS vulnérable :

#### DIG - Transfert de zone AXFR

DIG - Transfert de zone AXFR

```
dsgsec@htb[/htb]# creuser AXFR @ns1.inlanefreight.htb inlanefreight.htb

; <<>> DiG 9.11.5-P1-1-Debian <<>> axfr inlanefrieght.htb @10.129.110.213
;; options globales : +cmd
inlanefrieght.htb. 604800 Hôte local SOA IN. racine.localhost. 2 604800 86400 2419200 604800
inlanefrieght.htb. 604800 EN AAAA ::1
inlanefrieght.htb. 604800 IN hôte local NS.
inlanefrieght.htb. 604800 DANS UN 10.129.110.22
admin.inlanefrieght.htb. 604800 DANS UN 10.129.110.21
hr.inlanefrieght.htb. 604800 DANS UN 10.129.110.25
support.inlanefrieght.htb. 604800 DANS UN 10.129.110.28
inlanefrieght.htb. 604800 Hôte local SOA IN. racine.localhost. 2 604800 86400 2419200 604800
;; Temps de requête : 28 ms
;; SERVEUR : 10.129.110.213#53(10.129.110.213)
;; QUAND : Lundi 11 octobre 17:20:13 HAE 2020
;; Taille XFR : 8 enregistrements (messages 1, octets 289)

```

Des outils tels que [Fierce](https://github.com/mschwager/fierce) peuvent également être utilisés pour énumérer tous les serveurs DNS du domaine racine et rechercher un transfert de zone DNS :

DIG - Transfert de zone AXFR

```
dsgsec@htb[/htb]# féroce --domain zonetransfer.me

NS : nsztm2.digi.ninja. nsztm1.digi.ninja.
SOA : nsztm1.digi.ninja. (81.4.108.41)
Zone : succès
{<nom DNS @> : '@ 7200 IN SOA nsztm1.digi.ninja. robin.digi.ninja. 2019100801 '
                '172800 900 1209600 3600\n'
                '@ 300 IN HINFO "Casio fx-700G" "Windows XP"\n'
                '@ 301 EN TXT '
                '"google-site-verification=tyP28J7JAUHA9fw2sHXMgcCC0I6XBmmoVi04VlMewxA"\n'
                '@ 7200 IN MX 0 ASPMX.L.GOOGLE.COM.\n'
                '@ 7200 IN MX 10 ALT1.ASPMX.L.GOOGLE.COM.\n'
                '@ 7200 IN MX 10 ALT2.ASPMX.L.GOOGLE.COM.\n'
                '@ 7200 IN MX 20 ASPMX2.GOOGLEMAIL.COM.\n'
                '@ 7200 IN MX 20 ASPMX3.GOOGLEMAIL.COM.\n'
                '@ 7200 IN MX 20 ASPMX4.GOOGLEMAIL.COM.\n'
                '@ 7200 IN MX 20 ASPMX5.GOOGLEMAIL.COM.\n'
                '@ 7200 DANS UN 5.196.105.14\n'
                '@ 7200 IN NS nsztm1.digi.ninja.\n'
                '@ 7200 IN NS nsztm2.digi.ninja.',
  <nom DNS _acme-challenge> : '_acme-challenge 301 IN TXT '
                              '"6Oa05hbUJ9xSsvYy7pApQvwCUSSGgxvrbdizjePEsZI"',
  <nom DNS _sip._tcp> : '_sip._tcp 14000 IN SRV 0 0 5060 www',
  <Nom DNS 14.105.196.5.IN-ADDR.ARPA> : '14.105.196.5.IN-ADDR.ARPA 7200 IN PTR '
                                        'www',
  <nom DNS asfdbauthdns> : 'asfdbauthdns 7900 IN AFSDB 1 asfdbbox',
  <Nom DNS asfdbbox> : 'asfdbbox 7200 IN A 127.0.0.1',
  <nom DNS asfdbvolume> : 'asfdbvolume 7800 IN AFSDB 1 asfdbbox',
  <Nom DNS canberra-office> : 'canberra-office 7200 IN A 202.14.81.230',
  <nom DNS cmdexec> : 'cmdexec 300 IN TXT "; ls"',
  <DNS name contact> : 'contact 2592000 IN TXT "N'oubliez pas d'appeler ou d'envoyer un e-mail à Pippa '
                      'au +44 123 4567890 ou pippa@zonetransfer.me lors de '
                      'DNS change"',
  <nom DNS dc-office> : 'dc-office 7200 IN A 143.228.181.132',
  <nom DNS deadbeef> : 'deadbeef 7201 IN AAAA dead:beaf::',
  <nom DNS dr> : 'dr 300 IN LOC 53 20 56.558 N 1 38 33.526 W 0.00m',
  <Nom DNS DZC> : 'DZC 7200 IN TXT "AbCdEfG"',
  <nom DNS email> : 'email 2222 IN NAPTR 1 1 "P" "E2U+email" "" '
                    'email.zonetransfer.me\n'
                    'courriel 7200 AU 74.125.206.26',
  <DNS name Hello> : 'Hello 7200 IN TXT "Hi to Josh and all his class"',
  <nom DNS home> : 'home 7200 IN A 127.0.0.1',
  <Info nom DNS> : 'Info 7200 IN TXT "Service ZoneTransfer.me fourni par Robin '
                   'Bois - robin@digi.ninja. Voir '
                   'http://digi.ninja/projects/zonetransferme.php pour plus'
                   'information."',
  <nom DNS interne> : 'interne 300 IN NS intns1\ninterne 300 IN NS intns2',
  <Nom DNS intns1> : 'intns1 300 IN A 81.4.108.41',
  <Nom DNS intns2> : 'intns2 300 IN A 167.88.42.94',
  <bureau du nom DNS> : 'bureau 7200 IN A 4.23.39.254',
  <nom DNS ipv6actnow.org> : 'ipv6actnow.org 7200 IN AAAA '
                             '2001:67c:2e8:11::c100:1332',
...COUPER...

```

* * * * *

Prises de contrôle de domaine et énumération de sous-domaine
----------------------------------------

La "prise de contrôle de domaine" consiste à enregistrer un nom de domaine inexistant pour prendre le contrôle d'un autre domaine. Si les attaquants trouvent un domaine expiré, ils peuvent revendiquer ce domaine pour effectuer d'autres attaques telles que l'hébergement de contenu malveillant sur un site Web ou l'envoi d'un e-mail de phishing en exploitant le domaine revendiqué.

La reprise de domaine est également possible avec des sous-domaines appelés `reprise de sous-domaine`. Un enregistrement de nom canonique DNS (`CNAME`) est utilisé pour mapper différents domaines à un domaine parent. De nombreuses organisations utilisent des services tiers comme AWS, GitHub, Akamai, Fastly et d'autres réseaux de diffusion de contenu (CDN) pour héberger leur contenu. Dans ce cas, ils créent généralement un sous-domaine et le font pointer vers ces services. Par exemple,

DIG - Transfert de zone AXFR

```
sub.target.com. 60 IN CNAME autredomaine.com

```

Le nom de domaine (par exemple, `sub.target.com`) utilise un enregistrement CNAME vers un autre domaine (par exemple, `anotherdomain.com`). Supposons que `anotherdomain.com` expire et soit disponible pour que quiconque revendique le domaine, car le serveur DNS de `target.com` possède l'enregistrement `CNAME`. Dans ce cas, toute personne qui enregistre `anotherdomain.com` aura un contrôle total sur `sub.target.com` jusqu'à ce que l'enregistrement DNS soit mis à jour.

#### Énumération des sous-domaines

Avant d'effectuer une reprise de sous-domaine, nous devons énumérer les sous-domaines d'un domaine cible à l'aide d'outils tels que [Subfinder](https://github.com/projectdiscovery/subfinder). Cet outil peut récupérer des sous-domaines à partir de sources ouvertes comme [DNSdumpster](https://dnsdumpster.com/). D'autres outils comme [Sublist3r](https://github.com/aboul3la/Sublist3r) peuvent également être utilisés pour forcer brutalement des sous-domaines en fournissant une liste de mots pré-générée :

Énumération des sous-domaines

```
dsgsec@htb[/htb]# ./subfinder -d inlanefreight.com -v

         _ __ _ _
____ _| |__ / _(_)_ _ __| |___ _ _
(_-< || | '_ \ _| | ' \/ _ / -_) '_|
/__/\_,_|_.__/_| |_|_||__\__,_\___|_| v2.4.5
                 projetdécouverte.io

[WRN] À utiliser avec prudence. Vous êtes responsable de vos actes
[WRN] Les développeurs n'assument aucune responsabilité et ne sont pas responsables de toute utilisation abusive ou dommage.
[WRN] En utilisant subfinder, vous acceptez également les termes des API utilisées.

[INF] Énumération des sous-domaines pour inlanefreight.com
[alienvault] www.inlanefreight.com
[dnsdumpster] ns1.inlanefreight.com
[dnsdumpster] ns2.inlanefreight.com
...couper...
[bufferover] La source a pris 2.193235338s pour l'énumération
ns2.inlanefreight.com
www.inlanefreight.com
ns1.inlanefreight.com
support.inlanefreight.com
[INF] Trouvé 4 sous-domaines pour inlanefreight.com en 20 secondes 11 millisecondes

```

Une excellente alternative est un outil appelé [Subbrute](https://github.com/TheRook/subbrute). Cet outil nous permet d'utiliser des résolveurs auto-définis et d'effectuer des attaques de force brute DNS pures lors de tests de pénétration internes sur des hôtes qui n'ont pas accès à Internet.

#### Subbrute

Subbrute

```
dsgsec@htb[/htb]$ git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
dsgsec@htb[/htb]$ cd subbrute
dsgsec@htb[/htb]$ echo "ns1.inlanefreight.com" > ./resolvers.txt
dsgsec@htb[/htb]$ ./subbrute inlanefreight.com -s ./names.txt -r ./resolvers.txt

Avertissement : moins de 16 résolveurs par processus, envisagez d'ajouter plus de serveurs de noms à resolvers.txt.
inlanefreight.com
ns2.inlanefreight.com
www.inlanefreight.com
ms1.inlanefreight.com
support.inlanefreight.com

<SNIP>

```

Parfois dansLes configurations physiques internes sont mal sécurisées, ce que nous pouvons exploiter pour télécharger nos outils depuis une clé USB. Un autre scénario serait que nous ayons atteint un hôte interne par pivotement et que nous souhaitions travailler à partir de là. Bien sûr, il existe d'autres alternatives, mais cela ne fait pas de mal de connaître les voies et possibilités alternatives.

L'outil a trouvé quatre sous-domaines associés à `inlanefreight.com`. À l'aide de la commande `nslookup` ou `host` , nous pouvons énumérer les enregistrements `CNAME` pour ces sous-domaines.

Subbrute

```
dsgsec@htb[/htb]# hôte support.inlanefreight.com

support.inlanefreight.com est un alias pour inlanefreight.s3.amazonaws.com

```

Le sous-domaine `support` a un enregistrement d'alias pointant vers un compartiment AWS S3. Cependant, l'URL `https://support.inlanefreight.com` affiche une erreur `NoSuchBucket` indiquant que le sous-domaine est potentiellement vulnérable à une prise de contrôle de sous-domaine. Maintenant, nous pouvons reprendre le sous-domaine en créant un compartiment AWS S3 avec le même nom de sous-domaine.

![](https://academy.hackthebox.com/storage/modules/116/s3.png)

Le référentiel [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz) est également une excellente référence pour une vulnérabilité de prise de contrôle de sous-domaine. Il indique si les services cibles sont vulnérables à une prise de contrôle de sous-domaine et fournit des instructions sur l'évaluation de la vulnérabilité.

* * * * *

Usurpation DNS
------------

L'usurpation de DNS est également appelée DNS Cache Poisoning. Cette attaque consiste à alerter les enregistrements DNS légitimes avec de fausses informations afin qu'ils puissent être utilisés pour rediriger le trafic en ligne vers un site Web frauduleux. Voici des exemples de chemins d'attaque pour l'empoisonnement du cache DNS :

- Un attaquant pourrait intercepter la communication entre un utilisateur et un serveur DNS pour acheminer l'utilisateur vers une destination frauduleuse au lieu d'une destination légitime en effectuant une attaque Man-in-the-Middle (`MITM`).

- L'exploitation d'une vulnérabilité trouvée dans un serveur DNS pourrait donner le contrôle du serveur à un attaquant pour modifier les enregistrements DNS.

#### Empoisonnement du cache DNS local

Du point de vue du réseau local, un attaquant peut également effectuer un empoisonnement du cache DNS à l'aide d'outils MITM tels que [Ettercap](https://www.ettercap-project.org/) ou [Bettercap](https://www.bettercap.org/ ).

Pour exploiter l'empoisonnement du cache DNS via `Ettercap`, nous devons d'abord modifier le fichier `/etc/ettercap/etter.dns` pour mapper le nom de domaine cible (par exemple, `inlanefreight.com`) qu'ils veulent usurper et le nom de l'attaquant Adresse IP (par exemple `192.168.225.110`) vers laquelle ils souhaitent rediriger un utilisateur :

Empoisonnement du cache DNS local

```
dsgsec@htb[/htb]# chat /etc/ettercap/etter.dns

inlanefreight.com A 192.168.225.110
*.inlanefreight.com A 192.168.225.110

```

Ensuite, démarrez l'outil `Ettercap` et recherchez les hôtes actifs sur le réseau en accédant à `Hosts > Scan for Hosts`. Une fois terminé, ajoutez l'adresse IP cible (par exemple `192.168.152.129`) à Target1 et ajoutez une adresse IP de passerelle par défaut (par exemple `192.168.152.2`) à Target2.

![](https://academy.hackthebox.com/storage/modules/116/target.png)

Activez l'attaque `dns_spoof` en accédant à `Plugins > Gérer les plug-ins`. Cela envoie à la machine cible de fausses réponses DNS qui résoudront `inlanefreight.com` à l'adresse IP `192.168.225.110` :

![](https://academy.hackthebox.com/storage/modules/116/etter_plug.png)

Après une attaque par usurpation de DNS réussie, si un utilisateur victime provenant de la machine cible `192.168.152.129` visite le domaine `inlanefreight.com` sur un navigateur Web, il sera redirigé vers une `Fake page` hébergée sur l'adresse IP. `192.168.225.110` :

![](https://academy.hackthebox.com/storage/modules/116/etter_site.png)

De plus, un ping provenant de l'adresse IP cible `192.168.152.129` vers `inlanefreight.com` doit également être résolu en `192.168.225.110` :

Empoisonnement du cache DNS local

```
C:\>ping inlanefreight.com

Ping inlanefreight.com [192.168.225.110] avec 32 octets de données :
Réponse de 192.168.225.110 : octets=32 temps<1ms TTL=64
Réponse de 192.168.225.110 : octets=32 temps<1ms TTL=64
Réponse de 192.168.225.110 : octets=32 temps<1ms TTL=64
Réponse de 192.168.225.110 : octets=32 temps<1ms TTL=64

Statistiques de ping pour 192.168.225.110 :
     Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte de 0 %),
Temps d'aller-retour approximatifs en millisecondes :
     Minimum = 0 ms, Maximum = 0 ms, Moyenne = 0 ms

```

Voici quelques exemples d'attaques DNS courantes. Il existe d'autres attaques plus avancées qui seront traitées dans des modules ultérieurs.