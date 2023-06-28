Création d'une campagne basée sur les menaces Intel
===========

Une campagne axée sur les menaces prendra toutes les connaissances et tous les sujets précédemment couverts et les combinera pour créer une campagne bien planifiée et bien documentée.

Le flux de tâches dans cette salle suit logiquement le même chemin que vous emprunteriez en tant qu'équipe rouge pour commencer à planifier une campagne,

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5e73cca6ec4fcf1309f2df86/room-content/5f49a9db84d1bc6a72cd784767f94bb1.png)

1.  Identifier le cadre et la chaîne de mise à mort générale
2.  Déterminer l'adversaire ciblé
3.  Identifier les TTP et IOC de l'adversaire
4.  Mappez les renseignements sur les menaces recueillis à une chaîne ou à un cadre de mise à mort
5.  Rédiger et tenir à jour la documentation de mission nécessaire
6.  Déterminer et utiliser les ressources d'engagement nécessaires (outils, modification C2 , domaines, etc.)

* * * * *

Dans cette tâche, nous suivrons le processus de réflexion d'une équipe rouge du début à la fin de la planification d'une campagne axée sur les menaces.

La partie la plus difficile de la planification d'une campagne axée sur les menaces peut être de cartographier deux cybercadres différents. Pour simplifier ce processus, nous avons fourni un tableau de base comparant la Lockheed Martin Cyber ​​Kill Chain et le framework MITRE ATT&CK .

| Cyber ​​Kill Chain        | AT&CT D'ONGLET                |
| ------------------------- | ----------------------------- |
| Reconnaissance            | Reconnaissance                |
| Armement                  | Exécution                     |
| Livraison                 | Accès initial                 |
| Exploitation              | Accès initial                 |
| Installation              | Persistance / Défense Évasion |
| Commandement et contrôle  | Commander et contrôler        |
| Actions sur les objectifs | Exfiltration / Impact<br><br> |

Pour commencer à exécuter cette tâche, téléchargez les ressources requises et lancez le site statique associé à cette tâche.

Votre équipe a déjà décidé d'utiliser la chaîne de destruction cybernétique de Lockheed Martin pour émuler  [APT 41](https://attack.mitre.org/groups/G0096/)  en tant qu'adversaire qui correspond le mieux aux objectifs et à la portée du client.
