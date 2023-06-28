Engagement de la Red Team
=========================

Pour suivre les menaces émergentes, les engagements de l'équipe rouge ont été conçus pour déplacer l'attention des tests de pénétration réguliers vers un processus qui nous permet de voir clairement les capacités de notre équipe défensive à  détecter  et  à répondre  à un véritable acteur menaçant. Ils ne remplacent pas les tests d'intrusion traditionnels, mais les complètent en se concentrant sur la détection et la réponse plutôt que sur la prévention.

L'équipe rouge est un terme emprunté à l'armée. Dans les exercices militaires, un groupe jouerait le rôle d'une équipe rouge pour simuler des techniques d'attaque afin de tester les capacités de réaction d'une équipe en défense, généralement appelée  équipe bleue , contre des stratégies adverses connues. Traduits dans le monde de la cybersécurité, les engagements de l'équipe rouge consistent à imiter les tactiques, techniques et procédures (TTP) d'un véritable acteur menaçant   afin que nous puissions mesurer la manière dont notre équipe bleue y répond et, en fin de compte, améliorer les contrôles de sécurité en place.

Chaque engagement de l'équipe rouge commencera par définir des objectifs clairs, souvent appelés  joyaux de la couronne  ou  drapeaux , allant de la compromission d'un hôte critique donné au vol d'informations sensibles à la cible. Habituellement, l'équipe bleue ne sera pas informée de tels exercices pour éviter d'introduire des biais dans leur analyse. L'équipe rouge fera tout son possible pour atteindre les objectifs tout en restant non détectée et en évitant tous les mécanismes de sécurité existants tels que les pare-feu, antivirus, EDR, IPS et autres. Remarquez comment, lors d'un engagement d'équipe rouge, tous les hôtes d'un réseau ne seront pas vérifiés pour les vulnérabilités. Un véritable attaquant n'aurait besoin que de trouver un seul chemin vers son objectif et n'est pas intéressé à effectuer des analyses bruyantes que l'équipe bleue pourrait détecter.

En prenant le même réseau qu'avant, sur un engagement d'équipe rouge où le but est de compromettre le serveur intranet, nous envisagerions un moyen d'atteindre notre objectif tout en interagissant le moins possible avec les autres hôtes. Pendant ce temps, la capacité de l'équipe bleue à détecter et à réagir en conséquence à l'attaque peut être évaluée : 

![6e1f7ed550b706def50cb8df996caa8e](https://github.com/dsgsec/Red-Team/assets/82456829/3631758b-8f4c-4187-bf88-98e36c75efd6)

Il est important de noter que l'objectif final de tels exercices ne devrait jamais être pour l'équipe rouge de "battre" l'équipe bleue, mais plutôt de simuler suffisamment de TTP pour que l'équipe bleue apprenne à réagir de manière adéquate à une menace réelle en cours. Si nécessaire, ils peuvent modifier ou ajouter des contrôles de sécurité qui contribuent à améliorer leurs capacités de détection.

Les engagements de l'équipe rouge améliorent également les tests de pénétration réguliers en prenant en compte plusieurs surfaces d'attaque :

-   Infrastructure technique :  Comme dans un test de pénétration régulier, une équipe rouge tentera de découvrir les vulnérabilités techniques, en mettant beaucoup plus l'accent sur la furtivité et l'évasion.
-   Ingénierie sociale : Cibler des personnes par le biais de campagnes de phishing, d'appels téléphoniques ou de médias sociaux pour les inciter à révéler des informations qui devraient être privées.
-   Intrusion physique : utilisation de techniques telles que le crochetage, le clonage RFID, exploitation des faiblesses des dispositifs de contrôle d'accès électroniques pour accéder aux zones restreintes des installations.

Selon les ressources disponibles, l'exercice de l'équipe rouge peut se dérouler de plusieurs manières :

-   Engagement total : simulez le flux de travail complet d'un attaquant, de la compromission initiale jusqu'à ce que les objectifs finaux aient été atteints.
-   Violation supposée : commencez par supposer que l'attaquant a déjà pris le contrôle de certains actifs et essayez d'atteindre les objectifs à partir de là. Par exemple, l'équipe rouge pourrait recevoir l'accès aux informations d'identification de certains utilisateurs ou même à un poste de travail du réseau interne.
-   Exercice sur table :   Une simulation sur table où les scénarios sont discutés entre les équipes rouges et bleues pour évaluer comment elles réagiraient théoriquement à certaines menaces. Idéal pour les situations où faire des simulations en direct peut être compliqué.
