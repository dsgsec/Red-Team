Scénario et coup d'envoi
==================

* * * * *

Notre client, Inlanefreight, a engagé notre société, Acme Security, Ltd., pour effectuer un test de pénétration externe complet afin d'évaluer la sécurité de son périmètre. Le client nous a demandé d'identifier autant de vulnérabilités que possible ; par conséquent, les tests évasifs ne sont pas nécessaires. Ils aimeraient voir quel type d'accès un utilisateur anonyme peut obtenir sur Internet. Selon les règles d'engagement (RoE), si nous pouvons violer la DMZ et prendre pied dans le réseau interne, ils aimeraient que nous voyions jusqu'où nous pouvons prendre cet accès, jusqu'à et y compris la compromission du domaine Active Directory. Le client n'a pas fourni les informations d'identification de l'application Web, du VPN ou de l'utilisateur Active Directory. Les plages de domaines et de réseaux suivantes sont susceptibles d'être testées :

| Tests externes | Tests internes |
| --- | --- |
| 10.129.x.x (hôte cible "externe") | 172.16.8.0/23 |
| *.inlanefreight.local (tous les sous-domaines) | 172.16.9.0/23 |
| | INLANEFREIGHT.LOCAL (domaine Active Directory) |

Le client a fourni le domaine principal et les réseaux internes, mais n'a pas donné de détails sur les sous-domaines exacts dans cette portée ni sur les hôtes "en direct" que nous rencontrerons au sein du réseau. Ils aimeraient que nous effectuions une découverte pour voir quel type de visibilité un attaquant peut gagner contre leur réseau externe (et interne si un pied est atteint).

Les techniques de test automatisées telles que l'énumération et l'analyse des vulnérabilités sont autorisées, mais nous devons travailler avec soin pour ne pas perturber le service. Les éléments suivants ne sont pas couverts par cette évaluation :

- Hameçonnage/ingénierie sociale contre tout employé ou client d'Inlanefreight
- Attaques physiques contre les installations d'Inlanefreight
- Actions destructrices ou tests de déni de service (DoS)
- Modifications de l'environnement sans le consentement écrit du personnel informatique autorisé d'Inlanefreight

* * * * *

Lancement du projet
---------------

À ce stade, nous avons un cahier des charges signé par la direction de notre entreprise et un membre autorisé du service informatique d'Inlanefreight. Ce document SoW répertorie les spécificités des tests, notre méthodologie, le calendrier et les réunions et livrables convenus. Le client a également signé un document séparé sur les règles d'engagement (RoE), communément appelé document d'autorisation de test. Ce document est essentiel à avoir en main avant de commencer les tests et répertorie la portée de tous les types d'évaluation (URL, adresses IP individuelles, plages de réseau CIDR et informations d'identification, le cas échéant). Ce document répertorie également le personnel clé de la société de test et d'Inlanefreight (un minimum de deux contacts pour chaque côté, y compris leur numéro de téléphone portable et leur adresse e-mail). Le document répertorie également des détails tels que la date de début et de fin des tests et la fenêtre de test autorisée.

Nous avons eu une semaine pour les tests et deux jours supplémentaires pour rédiger notre projet de rapport (sur lequel nous devrions travailler au fur et à mesure). Le client nous a autorisés à tester 24h/24 et 7j/7, mais nous a demandé d'exécuter toute analyse de vulnérabilité lourde en dehors des heures normales de bureau (après 18h00, heure de Londres). Nous avons vérifié tous les documents nécessaires et avons les signatures requises des deux côtés, et le champ d'application est entièrement rempli, nous sommes donc prêts à partir d'un point de vue administratif.

* * * * *

Début des tests
----------------

C'est lundi matin à la première heure et nous sommes prêts à commencer les tests. Notre machine virtuelle de test est configurée et prête à fonctionner, et nous avons configuré un squelette de prise de notes et une structure de répertoires pour prendre des notes à l'aide de notre outil de prise de notes préféré. Pendant que nos analyses de découverte initiales s'exécutent, comme toujours, nous remplirons autant que possible le modèle de rapport. C'est une petite efficacité que nous pouvons gagner en attendant la fin des analyses pour optimiser le temps dont nous disposons pour les tests. Nous avons rédigé l'e-mail suivant pour signaler le début des tests et copié tout le personnel nécessaire.

![texte](https://academy.hackthebox.com/storage/modules/163/start_testing.png)

Nous cliquons sur envoyer sur l'e-mail et lançons notre collecte d'informations externes.