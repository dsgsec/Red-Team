## Principes d'énumération

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