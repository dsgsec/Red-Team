Moteurs de recherche spécialisés
=======================



### Concernant le WHOIS et le DNS

Au-delà des outils de requête WHOIS et DNS standard que nous avons couverts dans la tâche 3, il existe des tiers qui offrent des services payants pour les données WHOIS historiques. Un exemple est l'historique WHOIS, qui fournit un historique des données WHOIS et peut être utile si le registrant du domaine n'a pas utilisé la confidentialité WHOIS lors de l'enregistrement du domaine.

Il existe une poignée de sites Web qui offrent des services DNS avancés qui sont gratuits. Certains de ces sites Web offrent des fonctionnalités riches et pourraient avoir une salle complète dédiée à l'exploration d'un domaine. Pour l'instant, nous allons nous concentrer sur les principaux aspects liés au DNS. Nous considérerons ce qui suit :

-   [AfficherDNS.info](https://viewdns.info/)
-   [Plateforme de renseignements sur les menaces](https://threatintelligenceplatform.com/)

### AfficherDNS.info

[ViewDNS.info](https://viewdns.info/) propose *une recherche IP inversée* . Initialement, chaque serveur Web utiliserait une ou plusieurs adresses IP ; cependant, aujourd'hui, il est courant de rencontrer des serveurs d'hébergement mutualisé. Avec l'hébergement partagé, une adresse IP est partagée entre plusieurs serveurs Web différents avec des noms de domaine différents. Avec la recherche IP inversée, à partir d'un nom de domaine ou d'une adresse IP, vous pouvez trouver les autres noms de domaine à l'aide d'une ou plusieurs adresses IP spécifiques.

Dans la figure ci-dessous, nous avons utilisé la recherche IP inversée pour trouver d'autres serveurs partageant les mêmes adresses IP utilisées par `cafe.thmredteam.com`. Par conséquent, il est important de noter que connaître l'adresse IP ne mène pas nécessairement à un seul site Web.

![a6a57e946b742bf9439430c1669382a5](https://github.com/dsgsec/Red-Team/assets/82456829/ff21e174-f518-4e79-9952-9a73da0fd366)

### Plateforme de renseignements sur les menaces

[Threat Intelligence Platform ](https://threatintelligenceplatform.com/)vous demande de fournir un nom de domaine ou une adresse IP, et il lancera une série de tests allant des vérifications de logiciels malveillants aux requêtes WHOIS et DNS . Les résultats WHOIS et DNS sont similaires aux résultats que nous obtiendrions en utilisant`whois` et `dig`, mais Threat Intelligence Platform les présente d'une manière plus lisible et visuellement attrayante. Il y a des informations supplémentaires que nous obtenons avec notre rapport. Par exemple, après avoir recherché `thmredteam.com`, nous voyons que les enregistrements de serveur de noms (NS) ont été résolus en leurs adresses IPv4 et IPv6 respectives, comme indiqué dans la figure ci-dessous.

![c920bcf224185c47c4ba54b300079e48](https://github.com/dsgsec/Red-Team/assets/82456829/7ed38a79-a7cf-449f-a71f-1e09dffd028e)

D'autre part, lorsque nous recherchions `cafe.thmredteam.com`, nous pouvions également obtenir une liste d'autres domaines sur la même adresse IP. Le résultat que nous voyons dans la figure ci-dessous est similaire aux résultats que nous avons obtenus en utilisant [ViewDNS.info](https://viewdns.info/) .

![c920bcf224185c47c4ba54b300079e48](https://github.com/dsgsec/Red-Team/assets/82456829/eb8da046-d543-48cb-8bff-b909ede4eaa3)

### Moteurs de recherche spécialisés

#### Censys

[Censys Search](https://search.censys.io/) peut fournir de nombreuses informations sur les adresses IP et les domaines. Dans cet exemple, nous recherchons l'une des adresses IP qui `cafe.thmredteam.com`se résout en. Nous pouvons facilement en déduire que l'adresse IP que nous avons recherchée appartient à Cloudflare. Nous pouvons voir des informations relatives aux ports 80 et 443, entre autres ; cependant, il est clair que cette adresse IP est utilisée pour des sites Web de serveurs autres que `cafe.thmredteam.com`. En d'autres termes, cette adresse IP appartient à une société autre que notre client, [Organic Cafe](https://cafe.thmredteam.com/) . Il est essentiel de faire cette distinction afin que nous ne sondions pas des systèmes en dehors de la portée de notre contrat.

![efc8f98cc9e721707d4bad477340e120](https://github.com/dsgsec/Red-Team/assets/82456829/5d684c10-4ee1-4e0b-b727-00649838047d)

#### Shodan

Vous vous souvenez peut-être d'avoir utilisé [Shodan](https://www.shodan.io/) dans la salle [de reconnaissance passive](https://tryhackme.com/room/passiverecon) . Dans cette section, nous allons montrer comment utiliser Shodan à partir de la ligne de commande.

Pour utiliser correctement Shodan à partir de la ligne de commande, vous devez créer un compte avec [Shodan](https://www.shodan.io/) , puis configurer  `shodan`pour utiliser votre clé API à l'aide de la commande,  `shodan init API_KEY`.

Vous pouvez utiliser différents filtres selon le [type de votre compte Shodan](https://account.shodan.io/billing) . Pour en savoir plus sur ce que vous pouvez faire avec `shodan`, nous vous suggérons de consulter [Shodan CLI](https://cli.shodan.io/) . Démontrons un exemple simple de recherche d'informations sur l'une des adresses IP que nous avons obtenues à partir de `nslookup cafe.thmredteam.com`. En utilisant `shodan host IP_ADDRESS`, nous pouvons obtenir l'emplacement géographique de l'adresse IP et des ports ouverts, comme indiqué ci-dessous.

Terminal Pentesteur

```
pentester@TryHackMe$ shodan host 172.67.212.249

172.67.212.249
City:                    San Francisco
Country:                 United States
Organisation:            Cloudflare, Inc.
Updated:                 2021-11-22T05:55:54.787113
Number of open ports:    5

Ports:
     80/tcp
    443/tcp
	|-- SSL Versions: -SSLv2, -SSLv3, -TLSv1, -TLSv1.1, TLSv1.2, TLSv1.3
   2086/tcp
   2087/tcp
   8080/tcp
```
