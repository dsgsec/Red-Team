# Normes d'évaluation

Les tests d'intrusion et les évaluations de vulnérabilité doivent être conformes à des normes spécifiques pour être accrédités et acceptés par les gouvernements et les autorités judiciaires. Ces normes aident à garantir que l'évaluation est effectuée de manière approfondie d'une manière généralement convenue pour augmenter l'efficacité de ces évaluations et réduire la probabilité d'une attaque contre l'organisation.

## Normes de conformité
Chaque organisme de conformité réglementaire a ses propres normes de sécurité de l'information que les organisations doivent respecter pour conserver leur accréditation. Les grands acteurs de la conformité en matière de sécurité de l'information sont PCI, HIPAA, FISMA et ISO 27001.

Ces accréditations sont nécessaires car elles certifient qu'une organisation a fait évaluer son environnement par un fournisseur tiers. Les organisations s'appuient également sur ces accréditations pour les opérations commerciales, car certaines entreprises ne feront pas d'affaires sans accréditations spécifiques des organisations.

## Norme de sécurité des données de l'industrie des cartes de paiement (PCI DSS)
La norme de sécurité des données de l'industrie des cartes de paiement (PCI DSS) est une norme communément connue en matière de sécurité des informations qui met en œuvre les exigences pour les organisations qui gèrent les cartes de crédit. Conformément aux réglementations gouvernementales, les organisations qui stockent, traitent ou transmettent des données de titulaire de carte doivent mettre en œuvre les directives PCI DSS. Cela inclut les banques ou les magasins en ligne qui gèrent leurs propres solutions de paiement (par exemple, Amazon).

Les exigences PCI DSS incluent l'analyse interne et externe des actifs. Par exemple, toutes les données de carte de crédit en cours de traitement ou de transmission doivent être effectuées dans un environnement de données de titulaire de carte (CDE). L'environnement CDE doit être correctement segmenté des actifs normaux. Les environnements CDE sont séparés de l'environnement habituel d'une organisation pour protéger les données des titulaires de carte contre toute compromission lors d'une attaque et limiter l'accès interne aux données.

## Loi sur la portabilité et la responsabilité de l'assurance maladie (HIPAA)
HIPAA est la loi sur la portabilité et la responsabilité de l'assurance maladie, qui est utilisée pour protéger les données des patients. HIPAA n'exige pas nécessairement des analyses ou des évaluations de vulnérabilité ; cependant, une évaluation des risques et une identification des vulnérabilités sont nécessaires pour conserver l'accréditation HIPAA.

## Loi fédérale sur la gestion de la sécurité de l'information (FISMA)
`La loi fédérale sur la gestion de la sécurité de l'information (FISMA) est un ensemble de normes et de lignes directrices utilisées pour protéger les opérations et les informations du gouvernement. La loi exige qu'une organisation fournisse la documentation et la preuve d'un programme de gestion des vulnérabilités pour maintenir la disponibilité, la confidentialité et l'intégrité des systèmes informatiques.

## ISO 27001
ISO 27001 est une norme utilisée dans le monde entier pour gérer la sécurité de l'information. ISO 27001 exige des organisations qu'elles effectuent des analyses trimestrielles externes et internes.

Bien que la conformité soit essentielle, elle ne doit pas conduire à un programme de gestion des vulnérabilités. La gestion de la vulnérabilité doit tenir compte du caractère unique d'un environnement et de l'appétence au risque associée pour une organisation.

L'Organisation internationale de normalisation (ISO) maintient des normes techniques pour à peu près tout ce que vous pouvez imaginer. La norme ISO 27001 traite de la sécurité de l'information. La conformité à la norme ISO 27001 dépend du maintien d'un système de gestion de la sécurité de l'information efficace. Pour garantir la conformité, les organisations doivent effectuer des tests d'intrusion d'une manière soigneusement conçue.


<hr>

# Normes de test d'intrusion'
Les tests d'intrusion ne doivent pas être effectués sans règles ou directives. Il doit toujours y avoir une portée spécifiquement définie pour un pentest, et le propriétaire d'un réseau doit avoir un contrat légal signé avec les pentesters décrivant ce qu'ils sont autorisés à faire et ce qu'ils ne sont pas autorisés à faire. Le pentesting doit également être mené de manière à minimiser les dommages causés aux ordinateurs et aux réseaux d'une entreprise. Les testeurs d'intrusion doivent éviter d'apporter des modifications dans la mesure du possible (comme changer le mot de passe d'un compte) et limiter la quantité de données supprimées du réseau d'un client. Par exemple, au lieu de supprimer des documents sensibles d'un partage de fichiers, une capture d'écran des noms de dossier devrait suffire à prouver le risque.

En plus de la portée et de la légalité, il existe également diverses normes de pentesting, selon le type de système informatique évalué. Voici quelques-unes des normes les plus courantes que vous pouvez utiliser en tant que pentester.

## SPT
La norme d'exécution des tests d'intrusion (PTES) peut être appliquée à tous les types de tests d'intrusion. Il décrit les phases d'un test d'intrusion et comment elles doivent être menées. Voici les sections du PTES :

+ Interactions pré-engagement
+ La collecte de renseignements
+ Modélisation des menaces
+ Analyse de vulnérabilité
+ Exploitation
+ Post-exploitation
+ Rapports

## OSSTMM
OSSTMM est le manuel de méthodologie de test de sécurité Open Source, un autre ensemble de directives que les pentesters peuvent utiliser pour s'assurer qu'ils font correctement leur travail. Il peut être utilisé avec d'autres normes de pentest.

L'OSSTMM est divisé en cinq canaux différents pour cinq domaines différents de pentesting :

+ Sécurité humaine (les êtres humains sont sujets à des exploits d'ingénierie sociale)
+ Sécurité physique
+ Communications sans fil (y compris, mais sans s'y limiter, les technologies telles que WiFi et Bluetooth)
+ Télécommunications
+ Réseaux de données

## NIST
Le NIST (National Institute of Standards and Technology) est bien connu pour son NIST Cybersecurity Framework, un système de conception de politiques et de procédures de réponse aux incidents. Le NIST dispose également d'un cadre de test de pénétration. Les phases du cadre NIST comprennent :
+ Planification
+ Découverte
+ Attaque
+ Rapports

## OWASP
OWASP signifie Open Web Application Security Project. Il s'agit généralement de l'organisation de référence pour définir les normes de test et classer les risques pour les applications Web.

L'OWASP maintient quelques normes différentes et des guides utiles pour l'évaluation de diverses technologies :
+ Guide de test de sécurité Web (WSTG)
+ Guide de test de sécurité mobile (MSTG)
+ Méthodologie de test de sécurité du micrologiciel