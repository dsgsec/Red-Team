Introduction - Reconnaissance
===============
"Connais ton ennemi, connais son épée." a écrit Miyamoto Musashi dans son livre, A Book of Five Rings: The Classic Guide to Strategy. Il a également écrit: "Vous gagnez des batailles en connaissant le timing de l'ennemi et en utilisant un timing auquel l'ennemi ne s'attend pas." Bien que cela ait été écrit lorsque les épées et les lances gagnaient des batailles, cela s'applique également au cyberespace, où les attaques sont lancées via des claviers et des paquets fabriqués. Plus vous en savez sur l'infrastructure et le personnel de votre cible, mieux vous pouvez orchestrer vos attaques.

Dans une opération d'équipe rouge, vous pouvez commencer avec rien de plus qu'un nom d'entreprise, à partir duquel vous devez commencer à collecter des informations sur la cible. C'est là que la reconnaissance entre en jeu. La reconnaissance (recon) peut être définie comme une enquête préliminaire ou une observation de votre cible (client) sans l'alerter de vos activités. Si vos activités de reconnaissance créent trop de bruit, l'autre partie serait alertée, ce qui pourrait diminuer la probabilité de votre succès.

Les tâches de cette salle couvrent les sujets suivants :

-   Types d'activités de reconnaissance
-   Reconnaissance basée sur WHOIS et DNS
-   Recherche avancée
-   Recherche par image
-   Piratage Google
-   Moteurs de recherche spécialisés
-   Reconnaissance
-   Maltego
  
![66c311960c9ad71cbda3709d1c9804e2](https://github.com/dsgsec/Red-Team/assets/82456829/71b0d4f7-a9ae-4852-9b03-b3ec4863d16b)


Certains objectifs spécifiques que nous couvrirons incluent :

-   Découvrir les sous-domaines liés à notre entreprise cible
-   Collecte d'informations accessibles au public sur un hôte et des adresses IP
-   Recherche d'adresses e-mail liées à la cible
-   Découvrir les identifiants de connexion et les mots de passe divulgués
-   Localisation de documents et de feuilles de calcul ayant fait l'objet d'une fuite

La reconnaissance peut être décomposée en deux parties : la reconnaissance passive et la reconnaissance active, comme expliqué dans la tâche 2. Dans cette salle, nous nous concentrerons sur la reconnaissance passive, c'est-à-dire les techniques qui n'alertent pas la cible ou ne créent pas de « bruit ». Dans les salles ultérieures, nous utiliserons des outils de reconnaissance actifs qui ont tendance à être bruyants par nature.

## Taxonomie de la reconnaissance

La reconnaissance (recon) peut être classée en deux parties :

1.  Passive Recon : peut être effectué en regardant passivement
2.  Active Recon : nécessite d'interagir avec la cible pour la provoquer afin d'observer sa réponse.

La reconnaissance passive ne nécessite pas d'interaction avec la cible. En d'autres termes, vous n'envoyez aucun paquet ou demande à la cible ou aux systèmes que votre cible possède. Au lieu de cela, la reconnaissance passive s'appuie sur des informations accessibles au public qui sont collectées et conservées par un tiers. Open Source Intelligence (OSINT) est utilisé pour collecter des informations sur la cible et peut être aussi simple que de consulter le profil de réseau social accessible au public d'une cible. Les exemples d'informations que nous pourrions collecter incluent les noms de domaine, les blocs d'adresses IP, les adresses e-mail, les noms des employés et les offres d'emploi. Dans la tâche à venir, nous verrons comment interroger les enregistrements DNS et développer les sujets de la salle [de reconnaissance passive](https://tryhackme.com/room/passiverecon) et introduire des outils avancés pour vous aider dans votre reconnaissance.

La reconnaissance active nécessite d'interagir avec la cible en envoyant des requêtes et des paquets et en observant si et comment elle répond. Les réponses recueillies - ou l'absence de réponses - nous permettraient d'élargir l'image que nous avons commencé à développer en utilisant la reconnaissance passive. Un exemple de reconnaissance active consiste à utiliser Nmap pour analyser les sous-réseaux cibles et les hôtes actifs. D'autres exemples peuvent être trouvés dans la salle [de reconnaissance active](https://tryhackme.com/room/activerecon) . Certaines informations que nous voudrions découvrir incluent les hôtes en direct, les serveurs en cours d'exécution, les services d'écoute et les numéros de version.

La reconnaissance active peut être classée comme :

1.  Reconnaissance externe : menée en dehors du réseau de la cible et se concentre sur les actifs externes évaluables à partir d'Internet. Un exemple est l'exécution de Nikto depuis l'extérieur du réseau de l'entreprise.
2.  Reconnaissance interne : effectuée à partir du réseau de l'entreprise cible. En d'autres termes, le pentester ou le red teamer peut être physiquement situé à l'intérieur du bâtiment de l'entreprise. Dans ce scénario, ils pourraient utiliser un hôte exploité sur le réseau de la cible. Un exemple serait d'utiliser Nessus pour analyser le réseau interne à l'aide de l'un des ordinateurs de la cible.
