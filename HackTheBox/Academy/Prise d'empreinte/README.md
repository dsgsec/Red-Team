# Principes d'énumération

L'énumération est un terme largement utilisé en cybersécurité. Il s'agit de la collecte d'informations à l'aide de méthodes actives (scans) et passives (utilisation de fournisseurs tiers). Il est important de noter que l'OSINT est une procédure indépendante et doit être effectuée séparément de l'énumération, car l'OSINT est basée exclusivement sur la collecte passive d'informations et n'implique pas l'énumération active de la cible donnée. L'énumération est une boucle dans laquelle nous recueillons à plusieurs reprises des informations en fonction des données que nous avons ou que nous avons déjà découvertes.

Les informations peuvent être recueillies à partir de domaines, d'adresses IP, de services accessibles et de nombreuses autres sources.

Une fois que nous avons identifié les cibles dans l'infrastructure de notre client, nous devons examiner les services et protocoles individuels. Dans la plupart des cas, ce sont des services qui permettent la communication entre les clients, l'infrastructure, l'administration et les employés.

Si nous imaginons que nous avons été embauchés pour enquêter sur la sécurité informatique d'une entreprise, nous commencerons à développer une compréhension générale des fonctionnalités de l'entreprise. Par exemple, nous devons comprendre comment l'entreprise est structurée, quels services et fournisseurs tiers elle utilise, quelles mesures de sécurité peuvent être en place, et plus encore. C'est là que cette étape peut être un peu mal comprise car la plupart des gens se concentrent sur l'évidence et essaient de s'introduire de force dans les systèmes de l'entreprise au lieu de comprendre comment l'infrastructure est mise en place et quels aspects techniques et services sont nécessaires pour pouvoir offrir un prestation spécifique.

Un exemple d'une telle approche erronée pourrait être qu'après avoir trouvé des services d'authentification tels que SSH, RDP, WinRM, etc., nous essayons de forcer brutalement avec des mots de passe et des noms d'utilisateur communs/faibles. Malheureusement, le forçage brutal est une méthode bruyante et peut facilement conduire à une mise sur liste noire, rendant les tests supplémentaires impossibles. Cela peut se produire principalement si nous ne connaissons pas les mesures de sécurité défensives de l'entreprise et son infrastructure. Certains peuvent sourire de cette approche, mais l'expérience a montré que beaucoup trop de testeurs adoptent ce type d'approche.

Les principes d'énumération sont basés sur quelques questions qui faciliteront toutes nos investigations dans n'importe quelle situation imaginable. Dans la plupart des cas, de nombreux testeurs d'intrusion se concentrent principalement sur ce qu'ils peuvent voir et non sur ce qu'ils ne peuvent pas voir. Cependant, même ce que nous ne pouvons pas voir est pertinent pour nous et pourrait bien avoir une grande importance. La différence ici est que nous commençons à voir les composants et les aspects qui ne sont pas visibles à première vue avec notre expérience.

- Que pouvons-nous voir ?
- Quelles raisons peut-on avoir de le voir ?
- Quelle image ce que nous voyons crée-t-il pour nous ?
- Qu'est-ce qu'on y gagne ?
- Comment pouvons-nous l'utiliser?
- Que ne pouvons-nous pas voir ?
- Quelles raisons peut-il y avoir que nous ne voyons pas ?
- Quelle image résulte pour nous de ce que nous ne voyons pas ?

<hr>

## Méthodologie d'énumération
Considérez ces lignes comme une sorte d'obstacle, comme un mur, par exemple. Ce que nous faisons ici, c'est regarder autour de nous pour savoir où se trouve l'entrée, ou l'écart que nous pouvons franchir, ou grimper pour nous rapprocher de notre objectif. Théoriquement, il est aussi possible de traverser le mur la tête la première, mais bien souvent, il arrive que le spot dont on a défoncé l'écart avec beaucoup d'efforts et de temps avec force ne nous rapporte pas grand chose car il n'y a pas d'entrée à ce point de le mur pour passer au mur suivant.

Ces couches sont conçues comme suit :

| Couche | Descriptif | Catégories d'informations |
| --- | --- | --- |
| `1\. Présence Internet` | Identification de la présence sur Internet et de l'infrastructure accessible de l'extérieur. | Domaines, sous-domaines, vHosts, ASN, Netblocks, adresses IP, instances cloud, mesures de sécurité |
| `2\. Passerelle` | Identifier les mesures de sécurité possibles pour protéger l'infrastructure externe et interne de l'entreprise. | Pare-feu, DMZ, IPS/IDS, EDR, Proxies, NAC, Segmentation du réseau, VPN, Cloudflare |
| `3\. Services accessibles` | Identifiez les interfaces et les services accessibles qui sont hébergés en externe ou en interne. | Type de service, fonctionnalité, configuration, port, version, interface |
| `4\. Processus` | Identifiez les processus internes, les sources et les destinations associées aux services. | PID, Données traitées, Tâches, Source, Destination |
| `5\. Privilèges` | Identification des autorisations et privilèges internes aux services accessibles. | Groupes, Utilisateurs, Autorisations, Restrictions, Environnement |
| `6\. Configuration du système d'exploitation` | Identification des composants internes et configuration des systèmes. | Type de système d'exploitation, niveau de correctif, configuration réseau, environnement du système d'exploitation, fichiers de configuration, fichiers privés sensibles |

<hr>

## Couche n°1 : présence sur Internet
La première couche que nous devons passer est la couche "Présence Internet", où nous nous concentrons sur la recherche des cibles que nous pouvons enquêter. Si le périmètre dans le contrat nous permet de rechercher des hosts supplémentaires, cette couche est encore plus critique que pour les cibles fixes uniquement. Dans cette couche, nous utilisons différentes techniques pour trouver des domaines, des sous-domaines, des netblocks et de nombreux autres composants et informations qui présentent la présence de l'entreprise et de son infrastructure sur Internet.

L'objectif de cette couche est d'identifier tous les systèmes et interfaces cibles possibles qui peuvent être testés.

## Couche n°2 : Passerelle
Ici, nous essayons de comprendre l'interface de la cible joignable, comment elle est protégée et où elle se trouve dans le réseau. En raison de la diversité, des différentes fonctionnalités et de certaines procédures particulières, nous reviendrons plus en détail sur cette couche dans d'autres modules.

Le but est de comprendre à quoi nous avons affaire et à quoi nous devons faire attention.

## Couche n°3 : Services accessibles
Dans le cas des services accessibles, nous examinons chaque destination pour tous les services qu'elle offre. Chacun de ces services a un objectif spécifique qui a été installé pour une raison particulière par l'administrateur. Chaque service a certaines fonctions, qui conduisent donc également à des résultats spécifiques. Pour travailler efficacement avec eux, nous devons savoir comment ils fonctionnent. Sinon, il faut apprendre à les comprendre.

Cette couche vise à comprendre la raison et la fonctionnalité du système cible et à acquérir les connaissances nécessaires pour communiquer avec lui et l'exploiter efficacement à nos fins.

C'est la partie de l'énumération que nous traiterons principalement dans ce module.

## Couche n°4 : processus
Chaque fois qu'une commande ou une fonction est exécutée, des données sont traitées, qu'elles soient saisies par l'utilisateur ou générées par le système. Cela démarre un processus qui doit effectuer des tâches spécifiques, et ces tâches ont au moins une source et une cible.

Le but ici est de comprendre ces facteurs et d'identifier les dépendances entre eux.

## Couche n°5 : Privilèges
Chaque service s'exécute via un utilisateur spécifique dans un groupe particulier avec des autorisations et des privilèges définis par l'administrateur ou le système. Ces privilèges nous fournissent souvent des fonctions que les administrateurs négligent. Cela se produit souvent dans les infrastructures Active Directory et dans de nombreux autres environnements et serveurs d'administration spécifiques à chaque cas où les utilisateurs sont responsables de plusieurs zones d'administration.

Il est crucial de les identifier et de comprendre ce qui est et n'est pas possible avec ces privilèges.

## Couche n ° 6 : configuration du système d'exploitation
Ici, nous recueillons des informations sur le système d'exploitation réel et sa configuration à l'aide d'un accès interne. Cela nous donne un bon aperçu de la sécurité interne des systèmes et reflète les compétences et les capacités des équipes administratives de l'entreprise.

L'objectif ici est de voir comment les administrateurs gèrent les systèmes et quelles informations internes sensibles nous pouvons en tirer.

<hr>

## Informations de domaine

Les informations de domaine sont un élément central de tout test d'intrusion, et il ne s'agit pas seulement des sous-domaines mais de l'ensemble de la présence sur Internet. Par conséquent, nous recueillons des informations et essayons de comprendre la fonctionnalité de l'entreprise et quelles technologies et structures sont nécessaires pour que les services soient offerts avec succès et efficacité.

Ce type d'informations est collecté passivement sans analyses directes et actives. En d'autres termes, nous restons cachés et naviguons en tant que "clients" ou "visiteurs" pour éviter les connexions directes avec l'entreprise qui pourraient nous exposer. Les sections pertinentes de l'OSINT ne sont qu'une infime partie de la profondeur de l'OSINT et ne décrivent que quelques-unes des nombreuses façons d'obtenir des informations de cette manière. Plus d'approches et de stratégies pour cela peuvent être trouvées dans le module OSINT: Corporate Recon.

Cependant, lors de la collecte passive d'informations, nous pouvons utiliser des services tiers pour mieux comprendre l'entreprise. Cependant, la première chose que nous devrions faire est d'examiner le site Web principal de l'entreprise. Ensuite, nous devons lire les textes en gardant à l'esprit les technologies et les structures nécessaires à ces services.

Par exemple, de nombreuses entreprises informatiques proposent des services de développement d'applications, d'IoT, d'hébergement, de science des données et de sécurité informatique, en fonction de leur secteur d'activité. Si nous rencontrons un service avec lequel nous n'avions pas grand-chose à voir auparavant, il est logique et nécessaire de s'y attaquer et de savoir en quoi il consiste et quelles sont les opportunités qui s'offrent à lui. Ces services nous donnent également un bon aperçu de la façon dont l'entreprise peut être structurée.

Par exemple, cette partie est la combinaison entre le premier principe et le deuxième principe d'énumération. Nous prêtons attention à ce que nous voyons et nous ne voyons pas. Nous voyons les services mais pas leur fonctionnalité. Cependant, les services sont liés à certains aspects techniques nécessaires pour fournir un service. Par conséquent, nous adoptons le point de vue du développeur et examinons le tout de son point de vue. Ce point de vue nous permet d'acquérir de nombreuses connaissances techniques sur la fonctionnalité.

<hr>

## Présence en ligne
Une fois que nous avons une compréhension de base de l'entreprise et de ses services, nous pouvons avoir une première impression de sa présence sur Internet. Supposons qu'une entreprise de taille moyenne nous ait embauché pour tester l'ensemble de son infrastructure dans une perspective de boîte noire. Cela signifie que nous n'avons reçu qu'un ensemble d'objectifs et que nous devons obtenir nous-mêmes toutes les informations complémentaires.

Remarque : N'oubliez pas que les exemples ci-dessous seront différents des exercices pratiques et ne donneront pas les mêmes résultats. Cependant, les exemples sont basés sur de vrais tests de pénétration et illustrent comment et quelles informations peuvent être obtenues.

Le premier point de présence sur Internet peut être le certificat SSL du site Web principal de l'entreprise que nous pouvons examiner. Souvent, un tel certificat comprend plus qu'un simple sous-domaine, ce qui signifie que le certificat est utilisé pour plusieurs domaines, et ceux-ci sont très probablement toujours actifs.

Une autre source pour trouver plus de sous-domaines est crt.sh. Cette source est les journaux de transparence des certificats. La transparence des certificats est un processus destiné à permettre la vérification des certificats numériques émis pour les connexions Internet cryptées. La norme (RFC 6962) prévoit la journalisation de tous les certificats numériques délivrés par une autorité de certification dans des journaux à l'épreuve des audits. Ceci est destiné à permettre la détection de certificats faux ou émis de manière malveillante pour un domaine. Les fournisseurs de certificats SSL comme Let's Encrypt partagent cela avec l'interface Web crt.sh, qui stocke les nouvelles entrées dans la base de données pour y accéder ultérieurement.

### En ligne de commande
```json
dsgsec@htb[/htb]$ curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .

[
  {
    "issuer_ca_id": 23451835427,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "matomo.inlanefreight.com",
    "name_value": "matomo.inlanefreight.com",
    "id": 50815783237226155,
    "entry_timestamp": "2021-08-21T06:00:17.173",
    "not_before": "2021-08-21T05:00:16",
    "not_after": "2021-11-19T05:00:15",
    "serial_number": "03abe9017d6de5eda90"
  },
  {
    "issuer_ca_id": 6864563267,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "matomo.inlanefreight.com",
    "name_value": "matomo.inlanefreight.com",
    "id": 5081529377,
    "entry_timestamp": "2021-08-21T06:00:16.932",
    "not_before": "2021-08-21T05:00:16",
    "not_after": "2021-11-19T05:00:15",
    "serial_number": "03abe90104e271c98a90"
  },
  {
    "issuer_ca_id": 113123452,
    "issuer_name": "C=US, O=Let's Encrypt, CN=R3",
    "common_name": "smartfactory.inlanefreight.com",
    "name_value": "smartfactory.inlanefreight.com",
    "id": 4941235512141012357,
    "entry_timestamp": "2021-07-27T00:32:48.071",
    "not_before": "2021-07-26T23:32:47",
    "not_after": "2021-10-24T23:32:45",
    "serial_number": "044bac5fcc4d59329ecbbe9043dd9d5d0878"
  },
```
<hr>

## Shodan

Une fois que nous avons vu quels hôtes peuvent être étudiés plus en détail, nous pouvons générer une liste d'adresses IP avec un ajustement mineur de la commande cut et les exécuter via Shodan.

Shodan peut être utilisé pour trouver des appareils et des systèmes connectés en permanence à Internet, comme l'Internet des objets (IoT). Il recherche sur Internet les ports TCP/IP ouverts et filtre les systèmes en fonction de termes et de critères spécifiques. Par exemple, les ports HTTP ou HTTPS ouverts et d'autres ports de serveur pour FTP, SSH, SNMP, Telnet, RTSP ou SIP sont recherchés. En conséquence, nous pouvons trouver des appareils et des systèmes, tels que des caméras de surveillance, des serveurs, des systèmes de maison intelligente, des contrôleurs industriels, des feux de signalisation et des contrôleurs de trafic, ainsi que divers composants de réseau.

### Shodan - IP List
```bash
dsgsec@htb[/htb]$ for i in $(cat subdomainlist);do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt;done
dsgsec@htb[/htb]$ for i in $(cat ip-addresses.txt);do shodan host $i;done

10.129.24.93
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-09-01T09:02:11.370085
Number of open ports:    2

Ports:
     80/tcp nginx 
    443/tcp nginx 
	
10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-08-30T22:25:31.572717
Number of open ports:    3

Ports:
     22/tcp OpenSSH (7.6p1 Ubuntu-4ubuntu0.3)
     80/tcp nginx 
    443/tcp nginx 
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, -TLSv1.3, TLSv1.2
        |-- Diffie-Hellman Parameters:
                Bits:          2048
                Generator:     2
				
10.129.27.22
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-09-01T15:39:55.446281
Number of open ports:    8

Ports:
     25/tcp  
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2, TLSv1.3
     53/tcp  
     53/udp  
     80/tcp Apache httpd 
     81/tcp Apache httpd 
    110/tcp  
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2
    111/tcp  
    443/tcp Apache httpd 
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2, TLSv1.3
        |-- Diffie-Hellman Parameters:
                Bits:          2048
                Generator:     2
                Fingerprint:   RFC3526/Oakley Group 14
    444/tcp  
		
10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Updated:                 2021-08-30T22:25:31.572717
Number of open ports:    3

Ports:
     22/tcp OpenSSH (7.6p1 Ubuntu-4ubuntu0.3)
     80/tcp nginx 
    443/tcp nginx 
        |-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, -TLSv1.3, TLSv1.2
        |-- Diffie-Hellman Parameters:
                Bits:          2048
                Generator:     2
```

<hr>

## Enregistrement DNS

```
dsgsec@htb[/htb]$ dig any inlanefreight.com

; <<>> DiG 9.16.1-Ubuntu <<>> any inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52058
;; flags: qr rd ra; QUERY: 1, ANSWER: 17, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;inlanefreight.com.             IN      ANY

;; ANSWER SECTION:
inlanefreight.com.      300     IN      A       10.129.27.33
inlanefreight.com.      300     IN      A       10.129.95.250
inlanefreight.com.      3600    IN      MX      1 aspmx.l.google.com.
inlanefreight.com.      3600    IN      MX      10 aspmx2.googlemail.com.
inlanefreight.com.      3600    IN      MX      10 aspmx3.googlemail.com.
inlanefreight.com.      3600    IN      MX      5 alt1.aspmx.l.google.com.
inlanefreight.com.      3600    IN      MX      5 alt2.aspmx.l.google.com.
inlanefreight.com.      21600   IN      NS      ns.inwx.net.
inlanefreight.com.      21600   IN      NS      ns2.inwx.net.
inlanefreight.com.      21600   IN      NS      ns3.inwx.eu.
inlanefreight.com.      3600    IN      TXT     "MS=ms92346782372"
inlanefreight.com.      21600   IN      TXT     "atlassian-domain-verification=IJdXMt1rKCy68JFszSdCKVpwPN"
inlanefreight.com.      3600    IN      TXT     "google-site-verification=O7zV5-xFh_jn7JQ31"
inlanefreight.com.      300     IN      TXT     "google-site-verification=bow47-er9LdgoUeah"
inlanefreight.com.      3600    IN      TXT     "google-site-verification=gZsCG-BINLopf4hr2"
inlanefreight.com.      3600    IN      TXT     "logmein-verification-code=87123gff5a479e-61d4325gddkbvc1-b2bnfghfsed1-3c789427sdjirew63fc"
inlanefreight.com.      300     IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.24.8 ip4:10.129.27.2 ip4:10.72.82.106 ~all"
inlanefreight.com.      21600   IN      SOA     ns.inwx.net. hostmaster.inwx.net. 2021072600 10800 3600 604800 3600

;; Query time: 332 msec
;; SERVER: 127.0.0.53#53(127.0.0.53)
;; WHEN: Mi Sep 01 18:27:22 CEST 2021
;; MSG SIZE  rcvd: 940
```
Regardons ce que nous avons appris ici et revenons à nos principes. Nous voyons un enregistrement IP, des serveurs de messagerie, des serveurs DNS, des enregistrements TXT et un enregistrement SOA.

- Enregistrements A : Nous reconnaissons les adresses IP qui pointent vers un (sous-)domaine spécifique via l'enregistrement A. Ici, nous n'en voyons qu'un que nous connaissons déjà.

- Enregistrements MX : les enregistrements du serveur de messagerie nous indiquent quel serveur de messagerie est responsable de la gestion des e-mails pour l'entreprise. Étant donné que cela est géré par Google dans notre cas, nous devrions le noter et l'ignorer pour l'instant.

- Enregistrements NS : ces types d'enregistrements indiquent les serveurs de noms utilisés pour résoudre le FQDN en adresses IP. La plupart des hébergeurs utilisent leurs propres serveurs de noms, ce qui facilite l'identification de l'hébergeur.

- Enregistrements TXT : ce type d'enregistrement contient souvent des clés de vérification pour différents fournisseurs tiers et d'autres aspects de sécurité du DNS, tels que SPF, DMARC et DKIM, qui sont chargés de vérifier et de confirmer l'origine des e-mails envoyés. Ici, nous pouvons déjà voir des informations précieuses si nous regardons de plus près les résultats.