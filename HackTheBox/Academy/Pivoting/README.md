# Introduction au pivot, au tunneling et à la redirection de port

* * * * *

![](https://academy.hackthebox.com/storage/modules/158/PivotingandTunnelingVisualized.gif)

Au cours d'un `engagement d'équipe rouge`, d'un `test d'intrusion` ou d'une `évaluation Active Directory`, nous nous retrouvons souvent dans une situation où nous avons peut-être déjà compromis les `informations d'identification`, `clés ssh`, `hachages`, ou `access tokens` pour passer à un autre hôte, mais il se peut qu'aucun autre hôte ne soit directement accessible depuis notre hôte d'attaque. Dans de tels cas, nous devrons peut-être utiliser un `hôte pivot` que nous avons déjà compromis pour trouver un moyen d'atteindre notre prochaine cible. L'une des choses les plus importantes à faire lors de l'atterrissage sur un hôte pour la première fois est de vérifier notre `niveau de privilège`, `les connexions réseau` et le potentiel `VPN ou autre logiciel d'accès à distance`. Si un hôte possède plusieurs adaptateurs réseau, nous pouvons probablement l'utiliser pour passer à un autre segment de réseau. Le pivotement est essentiellement l'idée de "se déplacer vers d'autres réseaux via un hôte compromis pour trouver plus de cibles sur différents segments de réseau".

Il existe de nombreux termes différents utilisés pour décrire un hôte compromis que nous pouvons utiliser pour "pivoter" vers un segment de réseau auparavant inaccessible. Certains des plus courants sont :

- `Hôte pivot`
- "Procuration"
- 'Pied'
- "Système Beach Head"
- `Sauter l'hôte`

L'utilisation principale du pivot est de vaincre la segmentation (à la fois physiquement et virtuellement) pour accéder à un réseau isolé. Le "tunneling", en revanche, est un sous-ensemble du pivotement. Le tunneling encapsule le trafic réseau dans un autre protocole et achemine le trafic à travers celui-ci. Pensez-y comme ceci :

Nous avons une `clé` que nous devons envoyer à un partenaire, mais nous ne voulons pas que quiconque voit notre colis sache qu'il s'agit d'une clé. Nous obtenons donc un jouet en peluche et cachons la clé à l'intérieur avec des instructions sur ce qu'il fait. Nous emballons ensuite le jouet et l'envoyons à notre partenaire. Quiconque inspecte la boîte verra un simple jouet en peluche, sans se rendre compte qu'il contient autre chose. Seul notre partenaire saura que la clé est cachée à l'intérieur et apprendra comment y accéder et l'utiliser une fois livrée.

Les applications typiques telles que les VPN ou les navigateurs spécialisés ne sont qu'une autre forme de tunnellisation du trafic réseau.

* * * * *

Nous rencontrerons inévitablement plusieurs termes différents utilisés pour décrire la même chose dans l'industrie informatique et infosec. Avec le pivotement, nous remarquerons que cela est souvent appelé `Mouvement latéral`.

"N'est-ce pas la même chose que pivoter ?"

La réponse à cela n'est pas exactement. Prenons une seconde pour comparer et opposer `Mouvement latéral` avec `Pivoter et Tunneling`, car il peut y avoir une certaine confusion quant à la raison pour laquelle certains les considèrent comme des concepts différents.

* * * * *

Mouvement latéral, pivotement et tunnel comparés
--------------------------------------------------

#### Mouvement latéral

Le mouvement latéral peut être décrit comme une technique utilisée pour améliorer notre accès à des `hôtes`, `applications` et `services` supplémentaires au sein d'un environnement réseau. Le mouvement latéral peut également nous aider à accéder à des ressources de domaine spécifiques dont nous pourrions avoir besoin pour élever nos privilèges. Le déplacement latéral permet souvent une élévation des privilèges entre les hôtes. En plus de l'explication que nous avons fournie pour ce concept, nous pouvons également étudier comment d'autres organisations respectées expliquent le mouvement latéral. Consultez ces deux explications lorsque le temps le permet :

[Explication du réseau Palo Alto] (https://www.paloaltonetworks.com/cyberpedia/what-is-lateral-movement)

[Explication de MITRE](https://attack.mitre.org/tactics/TA0008/)

Un exemple pratique de `Mouvement latéral` serait :

Lors d'une évaluation, nous avons obtenu un premier accès à l'environnement cible et avons pu prendre le contrôle du compte administrateur local. Nous avons effectué une analyse du réseau et trouvé trois autres hôtes Windows sur le réseau. Nous avons tenté d'utiliser les mêmes informations d'identification d'administrateur local et l'un de ces appareils partageait le même compte d'administrateur. Nous avons utilisé les informations d'identification pour nous déplacer latéralement vers cet autre appareil, ce qui nous a permis de compromettre davantage le domaine.

#### Pivotant

Utilisation de plusieurs hôtes pour franchir les limites du "réseau" auxquelles vous n'auriez généralement pas accès. Il s'agit plutôt d'un objectif ciblé. L'objectif ici est de nous permettre d'aller plus loin dans un réseau en compromettant les hôtes ou l'infrastructure ciblés.

Un exemple pratique de `Pivoter` serait :

Au cours d'un engagement délicat, la cible avait son réseau physiquement et logiquement séparé. Cette séparation nous a rendu difficile de nous déplacer et d'atteindre nos objectifs. Nous avons dû rechercher sur le réseau et compromettre un hôte qui s'est avéré être le poste de travail d'ingénierie utilisé pour entretenir et surveiller l'équipement dans l'environnement opérationnel, soumettre des rapports et effectuer d'autres tâches administratives dans l'environnement de l'entreprise. Cet hôte s'est avéré être à double hébergement (ayant plusieurs cartes réseau physiques connectées à différents réseaux). S'il n'avait pas accès aux réseaux d'entreprise et opérationnels, nous n'aurions pas été unble de pivoter car nous devions terminer notre évaluation.

#### Tunnelage

Nous nous retrouvons souvent à utiliser divers protocoles pour acheminer le trafic vers/hors d'un réseau où il y a une chance que notre trafic soit détecté. Par exemple, utiliser HTTP pour masquer notre trafic Command & Control d'un serveur que nous possédons à l'hôte victime. La clé ici est l'obscurcissement de nos actions pour éviter d'être détecté aussi longtemps que possible. Nous utilisons des protocoles avec des mesures de sécurité renforcées telles que HTTPS sur TLS ou SSH sur d'autres protocoles de transport. Ces types d'actions permettent également des tactiques telles que l'exfiltration de données hors d'un réseau cible ou la livraison de plus de charges utiles et d'instructions dans le réseau.

Voici un exemple pratique de "tunneling" :

L'une des façons dont nous avons utilisé le tunneling était de créer notre trafic pour qu'il se cache dans HTTP et HTTPS. C'est une façon courante de maintenir le commandement et le contrôle (C2) des hôtes que nous avions compromis au sein d'un réseau. Nous avons masqué nos instructions dans les requêtes GET et POST qui apparaissaient comme du trafic normal et, pour un œil non averti, ressembleraient à une requête Web ou à une réponse à n'importe quel ancien site Web. Si le paquet était correctement formé, il serait transmis à notre serveur de contrôle. Si ce n'était pas le cas, il serait redirigé vers un autre site Web, ce qui pourrait décourager le défenseur de le vérifier.

Pour résumer, nous devrions considérer ces tactiques comme des choses distinctes. Le mouvement latéral nous aide à nous étendre au sein d'un réseau, élevant nos privilèges, tandis que le pivotement nous permet de plonger plus profondément dans les réseaux accédant à des environnements auparavant inaccessibles. Gardez cette comparaison à l'esprit lorsque vous parcourez ce module.

* * * * *

Maintenant que nous avons été initiés au module et que nous avons défini et comparé le mouvement latéral, le pivotement et le tunneling, plongeons dans certains des concepts de mise en réseau qui nous permettent d'effectuer ces tactiques.