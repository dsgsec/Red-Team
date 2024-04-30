Qu'est-ce que SSRF?
-------------------

La falsification de requêtes côté serveur est une vulnérabilité de sécurité Web qui permet à un attaquant de faire en sorte que l'application côté serveur effectue des requêtes vers un emplacement non souhaité.

Dans une attaque SSRF typique, l'attaquant peut amener le serveur à se connecter à des services internes uniquement au sein de l'infrastructure de l'organisation. Dans d'autres cas, ils peuvent être en mesure de forcer le serveur à se connecter à des systèmes externes arbitraires. Cela pourrait divulguer des données sensibles, telles que les informations d'identification d'autorisation.

![SSRF](https://portswigger.net/web-security/images/server-side%20request%20forgery.svg)

Attaques SSRF contre le serveur
-------------------------------

Dans une attaque SSRF contre le serveur, l'attaquant provoque l'application de faire une requête HTTP vers le serveur qui héberge l'application, via son interface réseau de bouclage. Cela implique généralement de fournir une URL avec un nom d'hôte comme `127.0.0.1` (une adresse IP réservée qui pointe vers l'adaptateur de bouclage) ou `localhost` (un nom couramment utilisé pour le même adaptateur).

Par exemple, imaginez une application d'achat qui permet à l'utilisateur de voir si un article est en stock dans un magasin particulier. Pour fournir les informations de stock, l'application doit interroger diverses API REST back-end en transmettant l'URL au point de terminaison API back-end concerné via une requête HTTP front-end. Lorsqu'un utilisateur affiche l'état du stock d'un article, son navigateur effectue la demande suivante:

`POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118 stockApi=http://stock.weliketoshop.net:8080/product/stock/check%3FproductId%3D6%26storeId%3D1`

Cela amène le serveur à faire une demande à l'URL spécifiée, à récupérer l'état du stock et à le renvoyer à l'utilisateur.

Dans cet exemple, un attaquant peut modifier la requête pour spécifier une URL locale au serveur:

`POST /product/stock HTTP/1.0 Content-Type: application/x-www-form-urlencoded Content-Length: 118 stockApi=http://localhost/admin`

Le serveur récupère le contenu de la `/admin` URL et le renvoie à l'utilisateur.

Un attaquant peut visiter le `/admin` URL, mais la fonctionnalité d'administration n'est normalement accessible qu'aux utilisateurs authentifiés. Cela signifie qu'un attaquant ne verra rien d'intéressant. Toutefois, si la demande à la `/admin` L'URL provient de la machine locale, les contrôles d'accès normaux sont contournés. L'application accorde un accès complet à la fonctionnalité administrative, car la demande semble provenir d'un emplacement de confiance.

Attaques SSRF contre le serveur - Suite
---------------------------------------

Pourquoi les applications se comportent-elles de cette manière et font-elles implicitement confiance aux requêtes provenant de la machine locale? Cela peut se produire pour diverses raisons:

-   La vérification du contrôle d'accès peut être implémentée dans un composant différent qui se trouve devant le serveur d'applications. Lorsqu'une connexion est établie vers le serveur, la vérification est contournée.
-   À des fins de reprise après sinistre, l'application peut autoriser l'accès administratif sans se connecter, à tout utilisateur provenant de la machine locale. Cela permet à un administrateur de récupérer le système s'il perd ses informations d'identification. Cela suppose que seul un utilisateur pleinement fiable viendrait directement du serveur.
-   L'interface d'administration peut écouter sur un numéro de port différent de l'application principale et peut ne pas être accessible directement par les utilisateurs.

Ce type de relations de confiance, où les demandes provenant de la machine locale sont traitées différemment des demandes ordinaires, font souvent de SSRF une vulnérabilité critique.
