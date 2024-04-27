Méthodes de contrôle d'accès basées sur des paramètres
------------------------------------------------------

Certaines applications déterminent les droits d'accès ou le rôle de l'utilisateur lors de la connexion, puis stockent ces informations dans un emplacement contrôlable par l'utilisateur. Cela pourrait être:

-   Un champ caché.
-   Un cookie.
-   Un paramètre de chaîne de requête prédéfini.

L'application prend des décisions de contrôle d'accès en fonction de la valeur soumise. Par exemple:

`https://insecure-website.com/login/home.jsp?admin=true https://insecure-website.com/login/home.jsp?role=1`

Cette approche n'est pas sécurisée car un utilisateur peut modifier la valeur et les fonctionnalités d'accès auxquelles il n'est pas autorisé, telles que les fonctions administratives.
