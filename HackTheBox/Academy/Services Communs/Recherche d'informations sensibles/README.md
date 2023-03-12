# Recherche d'informations sensibles

Lorsque nous attaquons un service, nous jouons généralement un rôle de détective, et nous devons collecter autant d'informations que possible et observer attentivement les détails. Par conséquent, chaque information est essentielle.

Imaginons que nous sommes dans un engagement avec un client, nous ciblons le courrier électronique, le FTP, les bases de données et le stockage, et notre objectif est d'obtenir l'exécution de code à distance (RCE) sur l'un de ces services. Nous avons commencé l'énumération et essayé l'accès anonyme à tous les services, et seul FTP a un accès anonyme. Nous avons trouvé un fichier vide dans le service FTP, mais avec le nom johnsmith, nous avons essayé johnsmith comme utilisateur et mot de passe FTP, mais cela n'a pas fonctionné. Nous essayons la même chose contre le service de messagerie et nous nous connectons avec succès. Avec l'accès aux e-mails, nous commençons à rechercher des e-mails contenant le mot mot de passe, nous en trouvons beaucoup, mais l'un d'eux contient les informations d'identification de John pour la base de données MSSQL. Nous accédons à la base de données et utilisons la fonctionnalité intégrée pour exécuter des commandes et obtenir avec succès RCE sur le serveur de base de données. Nous avons atteint notre objectif avec succès.

Un service mal configuré nous a permis d'accéder à une information qui peut initialement sembler insignifiante, johnsmith, mais cette information nous a ouvert les portes pour découvrir plus d'informations et enfin obtenir l'exécution de code à distance sur le serveur de base de données. C'est l'importance de prêter attention à chaque élément d'information, à chaque détail, lorsque nous énumérons et attaquons des services communs.

Les informations sensibles peuvent inclure, mais sans s'y limiter :
+ Noms d'utilisateur.
+ Adresses mail.
+ Mots de passe.
+ Enregistrements DNS.
+ Adresses IP.
+ Code source.
+ Fichiers de configuration.
+ PII.

Ce module couvrira certains services courants où nous pourrons trouver des informations intéressantes et découvrir différentes méthodes et outils que nous pouvons utiliser pour automatiser notre processus de découverte. Ces prestations comprennent :

+ Partages de fichiers.
+ E-mail.
+ Bases de données.

## Compréhension de ce que nous devons rechercher
Chaque cible est unique et nous devons nous familiariser avec notre cible, ses processus, ses procédures, son modèle commercial et son objectif. Une fois que nous comprenons notre cible, nous pouvons réfléchir aux informations essentielles pour eux et au type d'informations utiles pour notre attaque.

Il y a deux éléments clés pour trouver des informations sensibles :

+ Nous devons comprendre le service et son fonctionnement.
+ Nous devons savoir ce que nous recherchons.