# Gestion des vulnérabilités

La gestion des vulnérabilités est essentielle pour que les organisations restent au top de la sécurité de leur réseau interne et externe et prennent conscience des services exposés et des vulnérabilités potentielles qui peuvent affecter la posture de sécurité de l'organisation. Il existe divers composants dans la gestion des vulnérabilités, tels que le maintien de la conformité, la compréhension des matrices de risques et l'utilisation d'outils d'analyse des vulnérabilités. Bien que l'analyse des vulnérabilités ne remplace pas les évaluations de sécurité pratiques régulières telles que les tests d'intrusion, il s'agit d'un élément essentiel d'un programme de sécurité de l'information bien équilibré. Nessus et OpenVAS sont tous deux des outils d'analyse de vulnérabilité bien connus qui fournissent des éditions communautaires gratuites et des éditions professionnelles payantes. Ce module couvre les différentes fonctionnalités de Nessus et OpenVAS pour maximiser l'efficacité des outils.

Dans ce module, nous aborderons:
+ La différence entre un test d'intrusion et une évaluation de vulnérabilité
+ Systèmes de notation des vulnérabilités
+ Installation et utilisation des scanners de vulnérabilité communs Nessus et OpenVAS
+ Signaler les résultats de l'évaluation des vulnérabilités

<hr>

## Évaluations de sécurité

Chaque organisation doit effectuer différents types d'évaluations de sécurité sur ses réseaux, ordinateurs et applications au moins de temps en temps. L'objectif principal de la plupart des types d'évaluations de sécurité est de trouver et de confirmer la présence de vulnérabilités, afin que nous puissions travailler pour les corriger, les atténuer ou les supprimer. Il existe différentes méthodes et méthodologies pour tester le niveau de sécurité d'un système informatique. Certains types d'évaluations de sécurité sont plus appropriés pour certains réseaux que pour d'autres. Mais ils ont tous un but dans l'amélioration de la cybersécurité. Toutes les organisations ont des exigences de conformité et une tolérance au risque différentes, sont confrontées à des menaces différentes et ont des modèles commerciaux différents qui déterminent les types de systèmes qu'elles exécutent en externe et en interne. Certaines organisations ont une posture de sécurité beaucoup plus mature que leurs homologues et peuvent se concentrer sur des simulations avancées d'équipes rouges menées par des tiers, tandis que d'autres travaillent encore à établir une sécurité de base. Quoi qu'il en soit, toutes les organisations doivent rester au courant des vulnérabilités héritées et récentes et disposer d'un système de détection et d'atténuation des risques pour leurs systèmes et leurs données.

## Évaluation de la vulnérabilité
Les évaluations de la vulnérabilité conviennent à toutes les organisations et à tous les réseaux. Une évaluation de la vulnérabilité est basée sur une norme de sécurité particulière, et la conformité à ces normes est analysée (par exemple, en passant par une liste de contrôle).

Une évaluation de la vulnérabilité peut être basée sur diverses normes de sécurité. Les normes applicables à un réseau particulier dépendront de nombreux facteurs. Ces facteurs peuvent inclure des réglementations de sécurité des données spécifiques à l'industrie et régionales, la taille et la forme du réseau d'une entreprise, les types d'applications qu'elle utilise ou développe, et son niveau de maturité de sécurité.

Les évaluations de vulnérabilité peuvent être effectuées indépendamment ou parallèlement à d'autres évaluations de sécurité en fonction de la situation d'une organisation.

## Pentest

On les appelle des tests de pénétration parce que les testeurs les effectuent pour déterminer si et comment ils peuvent pénétrer un réseau. Un pentest est un type de cyberattaque simulée, et les pentesters mènent des actions qu'un acteur menaçant peut effectuer pour voir si certains types d'exploits sont possibles. La principale différence entre un pentest et une véritable cyberattaque est que la première est effectuée avec le plein consentement légal de l'entité testée. Qu'un pentester soit un employé ou un sous-traitant tiers, il devra signer un long document juridique avec l'entreprise cible qui décrit ce qu'il est autorisé à faire et ce qu'il n'est pas autorisé à faire.

Comme pour une évaluation de vulnérabilité, un pentest efficace se traduira par un rapport détaillé rempli d'informations pouvant être utilisées pour améliorer la sécurité d'un réseau. Toutes sortes de pentests peuvent être effectués selon les besoins spécifiques d'une organisation.

Le pentesting boîte noire est effectué sans aucune connaissance de la configuration ou des applications d'un réseau. En règle générale, un testeur recevra soit un accès au réseau (ou un port Ethernet et devra contourner le contrôle d'accès au réseau NAC)) et rien d'autre (les obligeant à effectuer leur propre découverte des adresses IP) si le pentest est interne ou rien de plus que l'entreprise nom si le pentest est d'un point de vue externe. Ce type de pentesting est généralement mené par des tiers du point de vue d'un attaquant externe. Souvent, le client demandera au pentester de lui montrer les adresses IP/plages de réseau internes/externes découvertes afin qu'il puisse confirmer la propriété et noter tous les hôtes qui devraient être considérés comme hors de portée.

Le test en boîte grise se fait avec un peu de connaissance du réseau qu'ils testent, d'un point de vue équivalent à un employé qui ne travaille pas dans le service informatique, comme un réceptionniste ou un agent du service client. Le client donnera généralement au testeur des plages de réseau dans le champ d'application ou des adresses IP individuelles dans une situation de boîte grise.

Le test d'intrusion boîte blanche est généralement effectué en donnant au testeur d'intrusion un accès complet à tous les systèmes, configurations, documents de construction, etc., et au code source si les applications Web sont concernées. Le but ici est de découvrir autant de défauts que possible qu'il serait difficile ou impossible de découvrir aveuglément dans un laps de temps raisonnable.

Souvent, les pentesters se spécialisent dans un domaine particulier. Les testeurs de pénétration doivent avoir une connaissance de nombreuses technologies différentes, mais auront toujours une spécialité.

Les pentesters d'applications évaluent les applications Web, les applications client lourd, les API et les applications mobiles. Ils seront souvent versés dans la révision du code source et capables d'évaluer une application Web donnée du point de vue de la boîte noire ou de la boîte blanche (généralement une révision de code sécurisée).

Les pentesters de réseau ou d'infrastructure évaluent tous les aspects d'un réseau informatique, y compris ses périphériques réseau tels que les routeurs et les pare-feu, les postes de travail, les serveurs et les applications. Ces types de testeurs d'intrusion doivent généralement avoir une solide compréhension de la mise en réseau, de Windows, de Linux, d'Active Directory et d'au moins un langage de script. Les scanners de vulnérabilité du réseau, tels que Nessus, peuvent être utilisés avec d'autres outils lors du pentest du réseau, mais le scan de vulnérabilité du réseau n'est qu'une partie d'un bon pentest. Il est important de noter qu'il existe différents types de pentests (évasif, non évasif, évasif hybride). Un scanner tel que Nessus ne serait utilisé que lors d'un pentest non évasif dont le but est de trouver un maximum de failles dans le réseau. De plus, l'analyse des vulnérabilités ne serait qu'une petite partie de ce type de test d'intrusion. Les scanners de vulnérabilité sont utiles mais limités et ne peuvent pas remplacer le contact humain et d'autres outils et techniques.

Les pentesters physiques tentent de tirer parti des faiblesses de la sécurité physique et des défaillances des processus pour accéder à une installation telle qu'un centre de données ou un immeuble de bureaux.

+ Pouvez-vous ouvrir une porte de manière involontaire ?
+ Pouvez-vous suivre quelqu'un dans le centre de données ?
+ Pouvez-vous ramper à travers un évent?

Les pentesters d'ingénierie sociale testent les êtres humains.
+ Les employés peuvent-ils être dupés par le phishing, le vishing (hameçonnage par téléphone) ou d'autres escroqueries ?
+ Un pentester d'ingénierie sociale peut-il s'approcher d'une réceptionniste et lui dire : "Oui, je travaille ici ?"

Le pentesting est le plus approprié pour les organisations ayant un niveau de maturité de sécurité moyen ou élevé. La maturité de la sécurité mesure le degré de développement du programme de cybersécurité d'une entreprise, et la maturité de la sécurité prend des années à se construire. Cela implique d'embaucher des professionnels de la cybersécurité compétents, d'avoir des politiques de sécurité et une application bien conçues (telles que la gestion de la configuration, des correctifs et des vulnérabilités), des normes de renforcement de base pour tous les types d'appareils du réseau, une conformité réglementaire stricte, des plans de réponse aux cyberincidents bien exécutés, un CSIRT chevronné (équipe de réponse aux incidents de sécurité informatique), un processus de contrôle des modifications établi, un CISO (chef de la sécurité de l'information), un CTO (directeur technique), des tests de sécurité fréquents effectués au cours de l'année

## Autres types d'évaluations de sécurité

### Audits de sécurité
Les évaluations de vulnérabilité sont effectuées parce qu'une organisation choisit de les mener, et elle peut contrôler comment et quand elles sont évaluées. Les audits de sécurité sont différents. Les audits de sécurité sont généralement des exigences extérieures à l'organisation, et ils sont généralement mandatés par des agences gouvernementales ou des associations industrielles pour s'assurer qu'une organisation est conforme aux réglementations de sécurité spécifiques.

Par exemple, tous les détaillants, restaurants et fournisseurs de services en ligne et hors ligne qui acceptent les principales cartes de crédit (Visa, MasterCard, AMEX, etc.) doivent se conformer à la norme PCI-DSS "Payment Card Industry Data Security Standard". PCI DSS est une réglementation appliquée par le Conseil des normes de sécurité de l'industrie des cartes de paiement, une organisation dirigée par des sociétés de cartes de crédit et des entités du secteur des services financiers. Une entreprise qui accepte les paiements par carte de crédit et de débit peut être auditée pour sa conformité à la norme PCI DSS, et la non-conformité peut entraîner des amendes et ne plus être autorisée à accepter ces méthodes de paiement.

Indépendamment des réglementations pour lesquelles une organisation peut être auditée, il est de sa responsabilité d'effectuer des évaluations de vulnérabilité pour s'assurer qu'elle est conforme avant qu'elle ne soit soumise à un audit de sécurité surprise.

## Bugbounty
Les programmes de primes aux bogues sont mis en œuvre par toutes sortes d'organisations. Ils invitent les membres du grand public, avec certaines restrictions (généralement pas d'analyse automatisée), à trouver des vulnérabilités de sécurité dans leurs applications. Les chasseurs de primes de bogues peuvent être payés de quelques centaines de dollars à des centaines de milliers de dollars pour leurs découvertes, ce qui est un petit prix à payer pour une entreprise afin d'éviter qu'une vulnérabilité critique d'exécution de code à distance ne tombe entre de mauvaises mains.

Les grandes entreprises avec de grandes bases de clients et une maturité élevée en matière de sécurité sont appropriées pour les programmes de primes de bogues. Ils doivent avoir une équipe dédiée au tri et à l'analyse des rapports de bogues et être dans une situation où ils peuvent supporter des étrangers à la recherche de vulnérabilités dans leurs produits.

Des entreprises comme Microsoft et Apple sont idéales pour avoir des programmes de bugbounty en raison de leurs millions de clients et de leur solide maturité en matière de sécurité.

## Évaluation red-team
Les entreprises disposant de budgets plus importants et de plus de ressources peuvent embaucher leurs propres red-team dédiées ou utiliser les services de sociétés de conseil tierces pour effectuer des évaluations red-team. Une équipe rouge est composée de professionnels de la sécurité offensive qui ont une expérience considérable des tests d'intrusion. Une équipe rouge joue un rôle essentiel dans la posture de sécurité d'une organisation.

Une équipe rouge est un type de pentesting de boîte noire évasive, simulant toutes sortes de cyberattaques du point de vue d'un acteur de menace externe. Ces évaluations ont généralement un objectif final (par exemple, atteindre un serveur ou une base de données critique, etc.). Les évaluateurs ne signalent que les vulnérabilités qui ont conduit à la réalisation de l'objectif, pas autant de vulnérabilités que possible comme avec un test d'intrusion.

Si une entreprise a sa propre équipe rouge interne, son travail consiste à effectuer des tests de pénétration plus ciblés avec une connaissance d'initié de son réseau. Une équipe rouge doit constamment être engagée dans des campagnes d'équipe rouge. Les campagnes pourraient être basées sur de nouveaux cyber-exploits découverts grâce aux actions de groupes de menaces persistantes avancées (APT), par exemple. D'autres campagnes pourraient cibler des types spécifiques de vulnérabilités pour les explorer en détail une fois qu'une organisation en a été informée.

Idéalement, si une entreprise peut se le permettre et a développé sa maturité en matière de sécurité, elle devrait effectuer elle-même des évaluations régulières des vulnérabilités, engager des tiers pour effectuer des tests d'intrusion ou des évaluations d'équipe rouge et, le cas échéant, constituer une équipe rouge interne pour effectuer des tests de pente en boîte grise et blanche avec des paramètres et des portées plus spécifiques.

## Évaluation purple-team
Ma blue-team est composée de spécialistes de la sécurité défensive. Ce sont souvent des personnes qui travaillent dans un SOC (centre des opérations de sécurité) ou un CSIRT (équipe de réponse aux incidents de sécurité informatique). Souvent, ils ont aussi de l'expérience en criminalistique numérique. Donc, si les équipes bleues sont défensives et les équipes rouges sont offensives, le rouge mélangé au bleu est violet.

**Qu'est-ce qu'une équipe violette ?**

Les purple-team sont formées lorsque des spécialistes de la sécurité offensive et défensive travaillent ensemble avec un objectif commun, améliorer la sécurité de leur réseau. Les équipes rouges trouvent des problèmes de sécurité, et les équipes bleues apprennent ces problèmes auprès de leurs équipes rouges et s'efforcent de les résoudre. Une évaluation purple-team est comme une évaluation red-team, mais l'équipe bleue est également impliquée à chaque étape. L'équipe bleue peut même jouer un rôle dans la conception des campagnes. "Nous devons améliorer notre conformité PCI DSS. Alors regardons l'équipe rouge tester nos systèmes de point de vente et fournir des commentaires et des commentaires actifs pendant leur travail."