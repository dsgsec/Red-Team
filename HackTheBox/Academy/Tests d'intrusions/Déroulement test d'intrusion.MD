# Pentest

## Présentation des tests d'intrusion

Dans les sciences sociales, un processus est considéré comme un terme désignant la séquence dirigée d'événements. Dans un contexte opérationnel et organisationnel, les processus sont appelés plus précisément processus de travail, d'entreprise, de production ou de création de valeur. Les processus sont un autre nom pour les programmes exécutés dans les systèmes informatiques, qui font généralement partie du logiciel système.

Il est également essentiel de faire la distinction entre les processus déterministes et stochastiques. Un processus déterministe est un processus dans lequel chaque état est causalement dépendant et déterminé par d'autres états et événements antérieurs. Un processus stochastique est un processus dans lequel un état ne découle d'autres états qu'avec une certaine probabilité. Ici, seules des conditions statistiques peuvent être supposées. Pour nous, plusieurs des définitions ci-dessus se chevauchent. Nous utilisons la définition du processus de test d'intrusion des sciences sociales pour représenter un cours d'événements liés à des processus déterministes. C'est parce que toutes nos démarches sont basées sur les événements et les résultats que nous pouvons découvrir ou provoquer.

**Un processus de test d'intrusion est défini par des étapes et des événements successifs effectués par le testeur d'intrusion pour trouver un chemin vers l'objectif prédéfini.**

Les processus décrivent une séquence spécifique d'opérations dans un laps de temps particulier qui conduit au résultat souhaité. Il est également essentiel de noter que les processus ne représentent pas une recette fixe et ne sont pas un guide étape par étape. Nos processus de test d'intrusion doivent donc être grossiers et flexibles. Après tout, chaque client a une infrastructure, des désirs et des attentes uniques.

<hr>

## Étapes des tests d'intrusion

La manière la plus efficace de les représenter et de les définir consiste à passer par des étapes interdépendantes. Nous constatons souvent dans nos recherches que ces processus se présentent sous la forme d'un processus circulaire. Si nous regardons cela de plus près et imaginons que même une seule composante du processus circulaire ne s'applique pas, l'ensemble du processus est perturbé. Strictement défini, l'ensemble du processus échoue. Si nous supposons que nous commençons ce processus depuis le début, mais déjà avec les informations nouvellement acquises, il s'agit déjà d'une nouvelle approche de processus qui n'annule pas la précédente.

Le problème est qu'avec ces représentations et ces approches, il n'y a souvent rien sur quoi se rabattre pour étendre notre processus de test d'intrusion. Comme nous en avons discuté, il n'y a pas de guide étape par étape que nous pouvons suivre, mais des étapes qui permettent de varier et d'adapter les étapes et les approches individuelles de manière flexible aux résultats et aux informations que nous recevons. Nous pouvons développer notre propre playbook pour diverses choses que nous essayons à différentes étapes d'un test de pénétration, mais chaque environnement est différent et nous devons donc nous adapter constamment.

### Pré-Engagement

Le **pré-engagement** consiste à éduquer le client et à ajuster le contrat. Tous les tests nécessaires et leurs composants sont strictement définis et enregistrés contractuellement. Lors d'une réunion en face à face ou d'une conférence téléphonique, de nombreux arrangements sont pris, tels que :
+ Accord de non-divulgation
+ Objectifs
+ Portée
+ Estimation du temps
+ Règles d'engagement


### La collecte d'informations

La collecte d'informations décrit comment nous obtenons des informations sur les composants nécessaires de différentes manières. Nous recherchons des informations sur l'entreprise cible et les logiciels et matériels utilisés pour trouver des failles de sécurité potentielles que nous pouvons exploiter pour prendre pied.

### Évaluation de la vulnérabilité
Une fois arrivés à l'étape d'évaluation des vulnérabilités, nous analysons les résultats de notre étape de collecte d'informations, en recherchant les vulnérabilités connues dans les systèmes, les applications et les différentes versions de chacun pour découvrir les vecteurs d'attaque possibles. L'évaluation de la vulnérabilité est l'évaluation des vulnérabilités potentielles, à la fois manuellement et par des moyens automatisés. Ceci est utilisé pour déterminer le niveau de menace et la sensibilité de l'infrastructure réseau d'une entreprise aux cyberattaques.

### Exploitation
Dans la phase d'exploitation, nous utilisons les résultats pour tester nos attaques contre les vecteurs potentiels et les exécutons contre les systèmes cibles pour obtenir un accès initial à ces systèmes.

### Post-exploitation
A ce stade du test d'intrusion, nous avons déjà accès à la machine exploitée et nous nous assurons que nous y avons toujours accès même si des modifications et des changements sont apportés. Au cours de cette phase, nous pouvons essayer d'élever nos privilèges pour obtenir les droits les plus élevés possibles et rechercher des données sensibles telles que des informations d'identification ou d'autres données que le client souhaite protéger (pillage). Parfois, nous effectuons une post-exploitation pour démontrer à un client l'impact de notre accès. D'autres fois, nous effectuons une post-exploitation en tant qu'entrée du processus de mouvement latéral décrit ci-après.

### Mouvement latéral
Le mouvement latéral décrit le mouvement au sein du réseau interne de notre entreprise cible pour accéder à des hôtes supplémentaires au même niveau de privilège ou à un niveau supérieur. Il s'agit souvent d'un processus itératif combiné à des activités de post-exploitation jusqu'à ce que nous atteignions notre objectif. Par exemple, nous prenons pied sur un serveur Web, augmentons les privilèges et trouvons un mot de passe dans le registre. Nous effectuons une énumération plus poussée et voyons que ce mot de passe fonctionne pour accéder à un serveur de base de données en tant qu'utilisateur administrateur local. À partir de là, nous pouvons piller les données sensibles de la base de données et trouver d'autres informations d'identification pour approfondir notre accès au réseau. Dans cette étape, nous utiliserons généralement de nombreuses techniques basées sur les informations trouvées sur l'hôte ou le serveur exploité.

### Preuve de concept
Au cours de cette étape, nous documentons, étape par étape, les étapes que nous avons suivies pour parvenir à compromettre le réseau ou à obtenir un certain niveau d'accès. Notre objectif est de brosser un tableau de la façon dont nous avons pu enchaîner plusieurs faiblesses pour atteindre notre objectif afin qu'ils puissent voir une image claire de la façon dont chaque vulnérabilité s'intègre et aider à hiérarchiser leurs efforts de remédiation. Si nous ne documentons pas bien nos étapes, il est difficile pour le client de comprendre ce que nous avons pu faire et, par conséquent, rend ses efforts de remédiation plus difficiles. Si possible, nous pourrions créer un ou plusieurs scripts pour automatiser les étapes que nous avons prises pour aider notre client à reproduire nos résultats. Nous couvrons cela en profondeur dans le module Documentation et rapports.

### Post-Engagement
Au cours de la post-engagement, une documentation détaillée est préparée pour les administrateurs et la direction de l'entreprise cliente afin de comprendre la gravité des vulnérabilités trouvées. A ce stade, nous nettoyons également toutes les traces de nos actions sur tous les hôtes et serveurs. Au cours de cette étape, nous créons les livrables pour notre client, organisons une réunion de présentation du rapport et faisons parfois une présentation aux dirigeants de l'entreprise cible ou à leur conseil d'administration. Enfin, nous archiverons nos données de test conformément à nos obligations contractuelles et à la politique de l'entreprise. Nous conservons généralement ces données pendant une période définie ou jusqu'à ce que nous procédions à une évaluation post-résolution (retest) pour tester les correctifs du client.

<hr>

## Importance

Nous devons internaliser cette procédure et l'utiliser comme base pour tous nos engagements techniques. Les composantes de chaque étape nous permettent de comprendre précisément dans quels domaines nous devons nous améliorer et où se situent la plupart de nos difficultés et lacunes dans nos connaissances. Par exemple, nous pouvons considérer un site Web comme une cible que nous devons étudier.

| Scène | Descriptif |
| --- | --- |
| `1\. Pré-Engagement` | La première étape consiste à créer tous les documents nécessaires dans la phase de pré-engagement, à discuter des objectifs de l'évaluation et à clarifier toutes les questions. |
| `2\. Collecte d'informations` | Une fois les activités de préengagement terminées, nous examinons le site Web existant de l'entreprise que nous avons été chargés d'évaluer. Nous identifions les technologies utilisées et apprenons comment fonctionne l'application Web. |
| `3\. Évaluation de la vulnérabilité` | Avec ces informations, nous pouvons rechercher des vulnérabilités connues et enquêter sur des fonctionnalités douteuses qui peuvent permettre des actions involontaires. |
| `4\. Exploitation` | Une fois que nous avons trouvé des vulnérabilités potentielles, nous préparons notre code d'exploitation, nos outils et notre environnement et testons le serveur Web pour ces vulnérabilités potentielles. |
| `5\. Post-Exploitation` | Une fois que nous avons réussi à exploiter la cible, nous passons à la collecte d'informations et examinons le serveur Web de l'intérieur. Si nous trouvons des informations sensibles au cours de cette étape, nous essayons d'élever nos privilèges (selon le système et les configurations). |
| `6\. Mouvement latéral` | Si d'autres serveurs et hôtes du réseau interne sont concernés, nous essayons alors de nous déplacer sur le réseau et d'accéder à d'autres hôtes et serveurs en utilisant les informations que nous avons recueillies. |
| `7\. Preuve de concept` | Nous créons une preuve de concept qui prouve que ces vulnérabilités existent et automatisons même potentiellement les étapes individuelles qui déclenchent ces vulnérabilités. |
| `8\. Post-Engagement` | Enfin, la documentation est complétée et présentée à notre client sous la forme d'un rapport formel livrable. Par la suite, nous pouvons organiser une réunion de présentation du rapport pour clarifier tout ce qui concerne nos tests ou nos résultats et fournir tout le soutien nécessaire au personnel chargé de remédier à nos conclusions. |