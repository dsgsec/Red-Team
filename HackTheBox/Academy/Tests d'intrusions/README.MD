# Pentest

## Présentation des tests d'intrusion

L'informatique fait partie intégrante de presque toutes les entreprises. La quantité de données critiques et confidentielles stockées dans les systèmes informatiques ne cesse de croître, tout comme la dépendance vis-à-vis du fonctionnement ininterrompu des systèmes informatiques utilisés. Par conséquent, les attaques contre les réseaux d'entreprise, la perturbation de la disponibilité du système et d'autres moyens de causer des dommages importants à une entreprise (comme les attaques de rançongiciels) deviennent de plus en plus courants. Des informations importantes sur l'entreprise obtenues par des failles de sécurité et des cyberattaques peuvent être vendues à des concurrents, divulguées sur des forums publics ou utilisées à d'autres fins néfastes. Les défaillances du système sont délibérément déclenchées car elles sont de plus en plus difficiles à contrer.

Un test d'intrusion (Pentest) est une tentative d'attaque organisée, ciblée et autorisée visant à tester l'infrastructure informatique et ses défenseurs afin de déterminer leur sensibilité aux vulnérabilités de sécurité informatique. Un pentest utilise des méthodes et des techniques que de vrais attaquants utilisent. En tant que testeurs d'intrusion, nous appliquons diverses techniques et analyses pour évaluer l'impact qu'une vulnérabilité particulière ou une chaîne de vulnérabilités peut avoir sur la confidentialité, l'intégrité et la disponibilité des systèmes informatiques et des données d'une organisation.

<hr>

## Gestion des risques

En général, cela fait également partie de la gestion des risques d'une entreprise. L'objectif principal de la gestion des risques de sécurité informatique est d'identifier, d'évaluer et d'atténuer tout risque potentiel qui pourrait nuire à la confidentialité, à l'intégrité et à la disponibilité des systèmes d'information et des données d'une organisation et de réduire le risque global à un niveau acceptable. Cela comprend l'identification des menaces potentielles, l'évaluation de leurs risques et la prise des mesures nécessaires pour les réduire ou les éliminer. Cela se fait en mettant en œuvre les contrôles et politiques de sécurité appropriés, y compris le contrôle d'accès, le cryptage et d'autres mesures de sécurité. En prenant le temps de gérer correctement les risques de sécurité des systèmes informatiques d'une organisation, il est possible de s'assurer que les données sont conservées en toute sécurité.

Lors d'un pentest, nous préparons une documentation détaillée sur les démarches entreprises et les résultats obtenus. Cependant, il est de la responsabilité du client ou de l'opérateur de ses systèmes sous investigation de rectifier les vulnérabilités constatées. Notre rôle est de conseiller de confiance pour signaler les vulnérabilités, les étapes de reproduction détaillées et fournir des recommandations de correction appropriées, mais nous n'allons pas appliquer de correctifs ou apporter des modifications de code, etc. Il est important de noter qu'un pentest ne surveille pas l'infrastructure informatique. ou systèmes, mais un instantané momentané de l'état de la sécurité. Une déclaration à cet égard devrait être reflétée dans notre rapport de test d'intrusion livrable.

<hr>

## Évaluations de la vulnérabilité

L'analyse de vulnérabilité est un terme générique qui peut inclure des évaluations de vulnérabilité ou de sécurité et des tests de pénétration. Contrairement à un test d'intrusion, les évaluations de vulnérabilité ou de sécurité sont effectuées à l'aide d'outils purement automatisés. Les systèmes sont vérifiés par rapport aux problèmes connus et aux vulnérabilités de sécurité en exécutant des outils d'analyse tels que Nessus, Qualys, OpenVAS, etc. Dans la plupart des cas, ces vérifications automatisées ne permettent pas d'adapter les attaques aux configurations du système cible. C'est pourquoi les tests manuels effectués par un testeur humain expérimenté sont essentiels.

<hr>

## Les tests

Une partie essentielle du processus est le point de départ à partir duquel nous devons effectuer notre pentest. Chaque pentest peut être réalisé sous deux angles différents :

+ **Externe** ou **Interne**

## Test de pénétration externe
De nombreux pentests sont effectués d'un point de vue externe ou en tant qu'utilisateur anonyme sur Internet. La plupart des clients veulent s'assurer qu'ils sont aussi protégés que possible contre les attaques sur leur périmètre de réseau externe. Nous pouvons effectuer des tests depuis notre propre hôte (en utilisant, espérons-le, une connexion VPN pour éviter que notre FAI ne nous bloque) ou depuis un VPS. Certains clients ne se soucieront pas de la furtivité, tandis que d'autres nous demanderont de procéder aussi discrètement que possible et d'approcher les systèmes cibles pour éviter d'être bannis par les pare-feu et les systèmes IDS/IPS et éviter de déclencher une alarme. Ils peuvent demander une approche furtive ou "hybride" où nous devenons progressivement "plus bruyants" pour tester leurs capacités de détection. En fin de compte, notre objectif ici est d'accéder aux hôtes externes, d'obtenir des données sensibles ou d'accéder au réseau interne.

## Test de pénétration interne
Contrairement à un pentest externe, un pentest interne consiste à effectuer des tests depuis le réseau de l'entreprise. Cette étape peut être exécutée après avoir pénétré avec succès le réseau de l'entreprise via le pentest externe ou à partir d'un scénario de violation supposé. Les pentests internes peuvent également accéder à des systèmes isolés sans aucun accès Internet, ce qui nécessite généralement notre présence physique dans les locaux du client.

<hr>

## Types de tests d'intrusions
Peu importe comment nous commençons le pentest, le type de pentest joue un rôle important. Ce type détermine la quantité d'informations mises à notre disposition. Nous pouvons réduire ces types aux suivants :

| Taper | Informations fournies |
| --- | --- |
| `Boîte noire` | 'Minimum'. Seules les informations essentielles, telles que les adresses IP et les domaines, sont fournies. |
| `Greybox` | "Étendu". Dans ce cas, nous recevons des informations supplémentaires, telles que des URL spécifiques, des noms d'hôte, des sous-réseaux, etc. |
| `Whitebox` | "Maximum". Ici, tout nous est révélé. Cela nous donne une vue interne de toute la structure, ce qui nous permet de préparer une attaque en utilisant des informations internes. Nous pouvons recevoir des configurations détaillées, des informations d'identification d'administrateur, le code source de l'application Web, etc. |
| `Red-Teaming` | Peut inclure des tests physiques et de l'ingénierie sociale, entre autres. Peut être combiné avec l'un des types ci-dessus. |
| `Purple-Teaming` | Il peut être combiné avec l'un des types ci-dessus. Cependant, il se concentre sur une collaboration étroite avec les défenseurs. |

<hr>

## Types d'environnements de test
Outre la méthode de test et le type de test, une autre considération est ce qui doit être testé, qui peut être résumé dans les catégories suivantes :

| | | | | |
| --- | --- | --- | --- | --- |
| Réseau | Application Web | Mobile | API | Clients lourds |
| IdO | Nuage | Code source | Sécurité physique | Employés |
| Hôtes | Serveur | Politiques de sécurité | Pare-feu | IDS/IPS |

Il est important de noter que ces catégories peuvent souvent être mélangées. Tous les composants de test répertoriés peuvent être inclus en fonction du type de test à effectuer. Nous allons maintenant changer de vitesse et couvrir le processus de pénétration en profondeur pour voir comment chaque phase est décomposée et dépend de la précédente.