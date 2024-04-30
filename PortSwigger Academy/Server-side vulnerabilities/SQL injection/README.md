Qu'est-ce que l'injection SQL (SQLi)?
-------------------------------------

L'injection SQL (SQLi) est une vulnérabilité de sécurité Web qui permet à un attaquant d'interférer avec les requêtes qu'une application effectue sur sa base de données. Cela peut permettre à un attaquant de voir des données qu'il n'est normalement pas en mesure de récupérer. Cela peut inclure des données appartenant à d'autres utilisateurs ou toute autre donnée à laquelle l'application peut accéder. Dans de nombreux cas, un attaquant peut modifier ou supprimer ces données, entraînant des modifications persistantes du contenu ou du comportement de l'application.

Dans certaines situations, un attaquant peut escalader une attaque par injection SQL pour compromettre le serveur sous-jacent ou une autre infrastructure back-end. Il peut également leur permettre d'effectuer des attaques par déni de service.

Comment détecter les vulnérabilités d'injection SQL
---------------------------------------------------

Vous pouvez détecter l'injection SQL manuellement à l'aide d'un ensemble systématique de tests sur chaque point d'entrée de l'application. Pour ce faire, vous devez généralement soumettre:

-   Le caractère de citation unique `'` et rechercher des erreurs ou d'autres anomalies.
-   Une syntaxe spécifique à SQL qui évalue à la valeur de base (originale) du point d'entrée, et à une valeur différente, et recherche des différences systématiques dans les réponses de l'application.
-   Des conditions booléennes telles que `OR 1=1` et `OR 1=2`, et recherchez les différences dans les réponses de l'application.
-   Les charges utiles conçues pour déclencher des retards de temps lors de l'exécution dans une requête SQL, et rechercher des différences dans le temps nécessaire pour répondre.
-   Les charges utiles OAST conçues pour déclencher une interaction réseau hors bande lors de l'exécution dans une requête SQL et surveiller les interactions résultantes.

Vous pouvez également trouver la majorité des vulnérabilités d'injection SQL rapidement et de manière fiable à l'aide de Burp Scanner.


Récupération des données cachées
--------------------------------

Imaginez une application d'achat qui affiche des produits dans différentes catégories. Lorsque l'utilisateur clique sur le **Cadeaux** catégorie, leur navigateur demande l'URL:

`https://insecure-website.com/products?category=Gifts`

Cela amène l'application à faire une requête SQL pour récupérer les détails des produits pertinents de la base de données:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

Cette requête SQL demande à la base de données de retourner:

-   tous les détails (`*`')
-   de la `products` table
-   où le `category` est `Gifts`
-   et `released` est `1`.

La restriction `released = 1` est utilisé pour cacher les produits qui ne sont pas libérés. Nous pourrions supposer pour les produits inédits, `released = 0`.

Récupération des données cachées - Suite
----------------------------------------

L'application n'implémente aucune défense contre les attaques par injection SQL. Cela signifie qu'un attaquant peut construire l'attaque suivante, par exemple:

`https://insecure-website.com/products?category=Gifts'--`

Il en résulte la requête SQL:

`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`

Fondamentalement, notez que `--` est un indicateur de commentaire en SQL. Cela signifie que le reste de la requête est interprété comme un commentaire, le supprimant efficacement. Dans cet exemple, cela signifie que la requête ne comprend plus `AND released = 1`. En conséquence, tous les produits sont affichés, y compris ceux qui ne sont pas encore publiés.

Vous pouvez utiliser une attaque similaire pour que l'application affiche tous les produits dans n'importe quelle catégorie, y compris les catégories qu'ils ne connaissent pas:

`https://insecure-website.com/products?category=Gifts'+OR+1=1--`

Il en résulte la requête SQL:

`SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`

La requête modifiée renvoie tous les éléments où `category` est `Gifts`, ou `1` est égal à `1`. Comme `1=1` est toujours vrai, la requête renvoie tous les éléments.

#### Avertissement

Faites attention lors de l'injection de la condition `OR 1=1` dans une requête SQL. Même s'il semble inoffensif dans le contexte dans lequel vous injectez, il est courant que les applications utilisent les données d'une seule requête dans plusieurs requêtes différentes. Si votre condition atteint un `UPDATE` ou `DELETE` déclaration, par exemple, il peut entraîner une perte accidentelle de données.


Logique d'application subversive
--------------------------------

Imaginez une application qui permet aux utilisateurs de se connecter avec un nom d'utilisateur et un mot de passe. Si un utilisateur envoie le nom d'utilisateur `wiener` et le mot de passe `bluecheese`, l'application vérifie les informations d'identification en effectuant la requête SQL suivante:

`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`

Si la requête renvoie les détails d'un utilisateur, la connexion est réussie. Sinon, il est rejeté.

Dans ce cas, un attaquant peut se connecter en tant qu'utilisateur sans avoir besoin d'un mot de passe. Ils peuvent le faire en utilisant la séquence de commentaires SQL `--` pour supprimer la vérification du mot de passe du `WHERE` clause de la requête. Par exemple, en soumettant le nom d'utilisateur `administrator'--` et un mot de passe vide entraîne la requête suivante:

`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`

Cette requête renvoie l'utilisateur dont `username` est `administrator` et enregistre avec succès l'attaquant en tant qu'utilisateur.
