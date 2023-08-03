Introduction à la reconnaissance API
====================================

##### [Reconnaissance API](https://university.apisec.ai/products/api-penetration-testing/categories/2150259092)

Introduction
============

Pour cibler les API, vous devez d'abord être en mesure de les trouver. Dans ce module, nous apprendrons à découvrir la surface d'attaque API d'une cible à l'aide de techniques de reconnaissance passives et actives.

La façon dont une API est censée être consommée déterminera la facilité avec laquelle elle peut être trouvée. Les API publiques sont conçues pour être facilement trouvées et utilisées par les utilisateurs finaux. Les API publiques peuvent être entièrement publiques sans authentification ou être destinées à être utilisées par des utilisateurs authentifiés. L'authentification pour une API publique dépend principalement de la sensibilité des données traitées. Si une API publique ne gère que des informations publiques, il n'y a pas besoin d'authentification, dans la plupart des autres cas, l'authentification sera requise. Les API publiques sont destinées à être consommées par les utilisateurs finaux. Afin de faciliter cela, les fournisseurs d'API partagent une documentation qui sert de manuel d'instructions pour une API donnée. Cette documentation doit être conviviale pour l'utilisateur final et relativement simple à trouver.

Les API partenaires sont destinées à être utilisées exclusivement par les partenaires du fournisseur. Ceux-ci pourraient être plus difficiles à trouver si vous n'êtes pas un partenaire. Les API partenaires peuvent être documentées, mais la documentation est souvent limitée au partenaire.

Les API privées sont destinées à être utilisées, en privé, au sein d'une organisation. Ces API sont souvent moins documentées que les API partenaires, voire pas du tout, et si une documentation existe, elle est encore plus difficile à trouver. 

Dans tous les cas où la documentation de l'API n'est pas disponible, vous devrez savoir comment procéder à l'ingénierie inverse des demandes d'API. Nous couvrirons l'ingénierie inverse dans un module de suivi. Pour l'instant, nous nous concentrerons sur la détection des API et leur utilisation comme décrit dans la documentation découverte.

Indicateurs API Web
-------------------

Les API destinées à l'usage des consommateurs sont censées être facilement découvertes. En règle générale, le fournisseur d'API commercialisera son API auprès des développeurs qui souhaitent être des consommateurs. Ainsi, il sera souvent très facile de trouver des API, simplement en utilisant une application Web en tant qu'utilisateur final. Le but ici est de trouver des API à attaquer et cela peut être accompli en découvrant l'API elle-même ou la documentation de l'API. Si vous pouvez trouver l'API et la documentation de la cible en tant qu'utilisateur final, alors mission accomplie, vous avez découvert une API avec succès.

Une autre façon de trouver une API fournie par une cible consiste à parcourir la page de destination de la cible. Parcourez une page de destination pour trouver des liens vers une API ou un portail de développement. Lors de la recherche d'API, plusieurs signes indiquent que vous avez découvert l'existence d'une API Web. Soyez à l'affût des schémas de nommage d'URL évidents :

[https://nom-cible.com/api/v1](https://target-name.com/api/v1) 

<https://api.target-name.com/v1> 

[https://nom-cible.com/docs](https://target-name.com/v1)

[https://dev.target-name.com/rest](https://target-name.com/v1)

Recherchez les indicateurs d'API dans les noms de répertoire tels que :\
*/api, /api/v1, /v1, /v2, /v3, /rest, /swagger, /swagger.json, /doc, /docs, /graphql, /graphiql, / altair, /aire de jeux*

De plus, les sous-domaines peuvent également être des indicateurs d'API Web :

api .nom-cible.com

uat .nom-cible.com

dev .nom-cible.com

développeur .target-name.com

test .nom-cible.com

Les en-têtes de requête et de réponse HTTP sont un autre indicateur des API Web. L'utilisation de JSON ou XML peut être un bon indicateur que vous avez découvert une API. 

*En-têtes de requête et de réponse HTTP contenant "Content-Type : application/json, application/xml"*

*Surveillez également les réponses HTTP qui incluent des déclarations telles que :\
{"message": "Jeton d'autorisation manquant"}*

L'un des indicateurs les plus évidents d'une API serait les informations recueillies à l'aide de sources tierces telles que Github et les répertoires d'API.

Gitub :  <https://github.com/> 

Facteur Explorer : <https://www.postman.com/explore/apis>

Répertoire de l'API ProgrammableWeb :  <https://www.programmableweb.com/apis/directory> 

Gourou des API :  <https://apis.guru/> 

API publiques Projet Github : <https://github.com/public-apis/public-apis> 

Hub RapidAPI :  <https://rapidapi.com/search/> 

Lorsque vous recherchez les API d'une cible, utilisez l'application Web d'une cible telle qu'elle a été conçue. Utilisez un navigateur, accédez à l'application Web et voyez si une API est annoncée. Une fois que vous avez une idée du fonctionnement de l'application Web, approfondissez vos connaissances en déployant des techniques de reconnaissance passives et actives.
