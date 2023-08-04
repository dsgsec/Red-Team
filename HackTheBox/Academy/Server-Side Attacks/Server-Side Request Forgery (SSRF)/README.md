Server-Side Request Forgery (SSRF)
===================================

Les attaques Server-Side Request Forgery ( `SSRF`), répertoriées dans le top 10 de l'OWASP, nous permettent d'abuser des fonctionnalités du serveur pour effectuer des demandes de ressources internes ou externes au nom du serveur. Pour ce faire, nous devons généralement fournir ou modifier les URL utilisées par l'application cible pour lire ou soumettre des données. L'exploitation des vulnérabilités de SSRF peut entraîner :

-   Interagir avec des systèmes internes connus
-   Découverte des services internes via des scans de port
-   Divulgation de données locales/sensibles
-   Inclure des fichiers dans l'application cible
-   Fuite de hachages NetNTLM à l'aide de chemins UNC (Windows)
-   Réaliser l'exécution de code à distance

Nous pouvons généralement trouver des vulnérabilités SSRF dans les applications qui récupèrent des ressources distantes. Lors de la recherche de vulnérabilités SSRF, nous devons rechercher :

-   Parties des requêtes HTTP, y compris les URL
-   Importation de fichiers tels que HTML, PDF, images, etc.
-   Connexions au serveur distant pour récupérer des données
-   Importations de spécifications d'API
-   Tableaux de bord incluant ping et fonctionnalités similaires pour vérifier l'état des serveurs

Remarque : gardez toujours à l'esprit que le fuzzing d'applications Web doit faire partie de toute activité de test d'intrusion ou de chasse aux bugs. Cela étant dit, le fuzzing ne doit pas être limité aux seuls champs de saisie de l'utilisateur. Étendez également le fuzzing à certaines parties de la requête HTTP, telles que l'agent utilisateur.
