Une fois arrivé sur un réseau inconnu, notre premier objectif est d'identifier où nous nous trouvons et ce à quoi nous pouvons accéder. Au cours de l'engagement de l'équipe rouge, nous devons comprendre à quel système cible nous avons affaire, quel service la machine fournit, dans quel type de réseau nous nous trouvons. Ainsi, l'énumération de la machine compromise après avoir obtenu l'accès initial est la clé pour répondre à ces questions. des questions. Cette tâche portera sur les types courants de réseaux auxquels nous pouvons être confrontés au cours de l'engagement.

La segmentation du réseau est une couche supplémentaire de sécurité réseau divisée en plusieurs sous-réseaux. Il est utilisé pour améliorer la sécurité et la gestion du réseau. Par exemple, il est utilisé pour empêcher l'accès non autorisé aux actifs les plus précieux de l'entreprise, tels que les données clients, les dossiers financiers, etc.

Les réseaux locaux virtuels (VLAN) sont une technique de réseau utilisée dans la segmentation du réseau pour contrôler les problèmes de réseau, tels que les problèmes de diffusion sur le réseau local, et améliorer la sécurité. Les hôtes du VLAN ne peuvent communiquer qu'avec d'autres hôtes du même réseau VLAN. 

Si vous souhaitez en savoir plus sur les principes fondamentaux du réseau, nous vous suggérons d'essayer le module TryHackMe suivant : [ Fondamentaux du réseau ](https://tryhackme.com/module/network-fundamentals).

Réseaux internes

Les réseaux internes sont des sous-réseaux segmentés et séparés en fonction de l'importance du périphérique interne ou de l'importance de l'accessibilité de ses données. L'objectif principal du ou des réseaux internes est de partager des informations, des communications plus rapides et plus faciles, des outils de collaboration, des systèmes opérationnels et des services réseau au sein d'une organisation. Dans un réseau d'entreprise, les administrateurs réseau ont l'intention d'utiliser la segmentation du réseau pour diverses raisons, notamment le contrôle du trafic réseau, l'optimisation des performances du réseau et l'amélioration de la sécurité. 

![f86b9cce1276f4c317bcb4bae7686891](https://github.com/dsgsec/Red-Team/assets/82456829/de81b057-28d3-4fe0-8ceb-c7b54512fa76)


Le diagramme précédent est un exemple du concept simple de segmentation de réseau puisque le réseau est divisé en deux réseaux. Le premier concerne les postes de travail des employés et les appareils personnels. Le second concerne les périphériques réseau privés et internes qui fournissent des services internes tels que DNS , Web interne, services de messagerie, etc.

Une zone démilitarisée ( DMZ )

Un réseau DMZ est un réseau périphérique qui protège et ajoute une couche de sécurité supplémentaire au réseau local interne d'une entreprise contre le trafic non fiable. Une conception courante de DMZ est un sous-réseau situé entre l'Internet public et les réseaux internes.

La conception d'un réseau au sein de l'entreprise dépend de ses exigences et de ses besoins. Par exemple, supposons qu'une entreprise fournisse des services publics tels qu'un site Web, DNS , FTP, proxy, VPN, etc. Dans ce cas, elle peut concevoir un réseau DMZ pour isoler et activer le contrôle d'accès sur le trafic du réseau public, le trafic non fiable.

![df4d771470f80491ece99e42ee574ebf](https://github.com/dsgsec/Red-Team/assets/82456829/02095589-e8d6-4424-99fd-ed33fbf1492a)

Dans le diagramme précédent, nous représentons en rouge le trafic réseau vers le réseau DMZ , qui n'est pas fiable (provient directement d'Internet). Le trafic réseau vert entre le réseau interne est le trafic contrôlé qui peut passer par un ou plusieurs dispositifs de sécurité réseau.

L'énumération du système et du réseau interne est l'étape de découverte, qui permet à l'attaquant de se renseigner sur le système et le réseau interne. Sur la base des informations obtenues, nous les utilisons pour traiter un mouvement latéral ou une élévation de privilèges afin d'obtenir plus de privilèges sur le système ou l' environnement AD .

Énumération du réseau

Il y a diverses choses à vérifier liées aux aspects réseau tels que les ports TCP et UDP et les connexions établies, les tables de routage, les tables ARP, etc.

Commençons par vérifier les ports ouverts TCP et UDP de la machine cible. Cela peut être fait à l'aide de la  commande netstat  comme indiqué ci-dessous.

Invite de commande

```
PS C:\Users\thm> netstat -na

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING
```

La sortie révèle les ports ouverts ainsi que les connexions établies. Ensuite, listons la table ARP , qui contient l'adresse IP et l'adresse physique des ordinateurs qui ont communiqué avec les machines cibles au sein du réseau. Cela pourrait être utile pour voir les communications au sein du réseau afin d'analyser les autres machines à la recherche de ports ouverts et de vulnérabilités.

Invite de commande

```
PS C:\Users\thm> arp -a

Interface: 10.10.141.51 --- 0xa
  Internet Address      Physical Address      Type
  10.10.0.1             02-c8-85-b5-5a-aa     dynamic
  10.10.255.255         ff-ff-ff-ff-ff-ff     static
```

Services réseau internes

Il fournit un accès aux communications réseau privé et interne pour les périphériques réseau internes. Un exemple de services réseau est un DNS interne , des serveurs Web, des applications personnalisées, etc. Il est important de noter que les services réseau internes ne sont pas accessibles en dehors du réseau. Cependant, une fois que nous aurons un premier accès à l'un des réseaux qui accèdent à ces services réseau, ils seront accessibles et disponibles pour les communications. 

Nous aborderons davantage d'applications et de services Windows dans la tâche 9, notamment le DNS et les applications Web personnalisées.
