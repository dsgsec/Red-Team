Introduction
============
La sécurité des opérations ( OPSEC ) est un terme inventé par l'armée américaine. Dans le domaine de la cybersécurité, commençons par la définition fournie par [le NIST](https://csrc.nist.gov/glossary/term/opsec) :

> "Processus systématique et éprouvé par lequel les adversaires potentiels peuvent se voir refuser des informations sur les capacités et les intentions en identifiant, contrôlant et protégeant les preuves généralement non classifiées de la planification et de l'exécution d'activités sensibles. Le processus comprend cinq étapes : identification des informations critiques, analyse des menaces, analyse des vulnérabilités, évaluation des risques et application des contre-mesures appropriées. »

Plongeons-nous dans la définition du point de vue de l'équipe rouge. En tant que membre de l'équipe rouge, vos adversaires potentiels sont l'équipe bleue et les tiers. L'équipe bleue est considérée comme un adversaire car nous attaquons les systèmes qu'ils sont chargés de surveiller et de défendre. Les exercices de l'équipe rouge contre l'équipe bleue sont courants pour aider une organisation à comprendre quelles menaces existent dans un environnement donné et à mieux préparer son équipe bleue si une véritable attaque malveillante se produit. En tant que membres de l'équipe rouge, même si nous respectons la loi et sommes autorisés à attaquer des systèmes dans un périmètre défini, cela ne change rien au fait que nous agissons contre les objectifs de l'équipe bleue et essayons de contourner leurs contrôles de sécurité. L'équipe bleue veut protéger leurs systèmes, tandis que nous voulons les pénétrer.

Interdire à tout adversaire potentiel la capacité de recueillir des informations sur nos capacités et nos intentions est essentiel au maintien de l'OPSEC . L'OPSEC est un processus d' *identification* , *de contrôle* et *de protection* de toute information liée à la planification et à l'exécution de nos activités. Des cadres tels que [Cyber ​​Kill Chain de Lockheed Martin](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)  et  [MITRE ATT&CK ](https://attack.mitre.org/)aident les défenseurs à identifier les objectifs qu'un adversaire tente d'atteindre. MITRE ATT&CK est sans doute à l'avant-garde du signalement et de la classification des tactiques, techniques et procédures (TTP) de l'adversaire et offre une base de connaissances accessible au public en tant que renseignements sur les menaces et rapports d'incidents accessibles au public comme principale source de données.

![2cda611755b8a85867f37ce46c20aff7](https://github.com/dsgsec/Red-Team/assets/82456829/7b1e94fc-49e1-4aa5-92ed-3810ba19b7be)


Le processus OPSEC comporte cinq étapes :

1.  Identifier les informations critiques
2.  Analyser les menaces
3.  Analyser les vulnérabilités
4.  Évaluer les risques
5.  Appliquer les contre-mesures appropriées

![c50e97c624bc345971f81e80fb3c2a3e](https://github.com/dsgsec/Red-Team/assets/82456829/daa5787c-4536-4000-abc1-5af9ca81c423)

Si l'adversaire découvre que vous scannez son réseau avec Nmap (l'équipe bleue dans notre cas), il devrait facilement pouvoir découvrir l'adresse IP utilisée. Par exemple, si vous utilisez cette même adresse IP pour héberger un site de phishing, il ne sera pas très difficile pour l'équipe bleue de relier les deux événements et de les attribuer au même acteur.

L'OPSEC n'est pas une solution ou un ensemble de règles ; L'OPSEC est un processus en cinq étapes visant à empêcher les adversaires d'accéder à toute information critique (définie dans la tâche 2). Nous plongerons dans chaque étape et verrons comment nous pouvons améliorer l'OPSEC dans le cadre des opérations de notre équipe rouge.
