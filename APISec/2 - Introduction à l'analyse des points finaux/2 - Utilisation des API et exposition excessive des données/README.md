Utilisation des API et exposition excessive des données
=======================================================

##### [Analyse des points finaux](https://university.apisec.ai/products/api-penetration-testing/categories/2150251353)

Documentation API
=================

Maintenant que vous savez comment trouver des API en direct et la documentation pertinente, nous allons brièvement examiner l'utilisation d'une API comme prévu et comment vous pouvez découvrir une exposition excessive des données. La prochaine étape de notre processus de piratage d'API consistera à utiliser la documentation pour s'authentifier auprès de l'API cible et commencer à former des demandes.

Bien que la documentation de l'API soit simple, il y a quelques éléments à surveiller. La présentation est généralement la première section de la documentation de l'API. Généralement trouvé au début de la doc, il fournira une introduction de haut niveau sur la façon de se connecter et d'utiliser l'API. En outre, il peut contenir des informations sur l'authentification et la limitation du débit.

Consultez la documentation pour connaître les fonctionnalités ou les actions que vous pouvez effectuer à l'aide de l'API donnée. Ceux-ci seront représentés par une combinaison d'une méthode HTTP (GET, PUT, POST, DELETE) et d'un point de terminaison. Les API de chaque organisation seront différentes, mais vous pouvez vous attendre à trouver des fonctionnalités liées à la gestion des comptes d'utilisateurs, des options pour télécharger et télécharger des données, différentes façons de demander des informations, etc.

Lorsque vous faites une demande à un point de terminaison, assurez-vous de noter les exigences de la demande . Les exigences peuvent inclure une certaine forme d'authentification, des paramètres, des variables de chemin, des en-têtes et des informations incluses dans le corps de la demande. La documentation de l'API doit vous dire ce qu'elle exige de vous et mentionner à quelle partie de la demande appartiennent ces informations. Si la documentation fournit des exemples, utilisez-les pour vous aider. En règle générale, vous pouvez remplacer les exemples de valeurs par ceux que vous recherchez. Le tableau ci-dessous décrit certaines des conventions souvent utilisées dans ces exemples.

#### Conventions de documentation de l'API

|

Convention

 |

Exemple

 |

Signification

 |
|

: ou {} 

 |

/ID de l'utilisateur

/ID de l'utilisateur}

/utilisateur/2727

/Nom d'utilisateur du compte

/Nom d'utilisateur du compte}

/compte/scuttleph1sh

 |

Les deux-points ou les accolades sont utilisés par certaines API pour indiquer une variable de chemin. En d'autres termes, ":id" représente la variable d'un numéro d'identification et "{username}" représente le nom d'utilisateur du compte auquel vous essayez d'accéder.

 |
|

[]

 |

/api/v1/user?find=[nom]

 |

Les crochets indiquent que l'entrée est facultative.

 |
|

||

 |

"bleu" || "vert" || "rouge"

 |

Les doubles barres représentent différentes valeurs possibles pouvant être utilisées.

 |

Comprendre les conventions de documentation vous aidera à créer des requêtes bien formées et à dépanner les instances où l'API ne répond pas comme prévu. Pour mieux comprendre la documentation de l'API, prenons les spécifications crAPI Swagger rétro-conçues et importons-les dans Postman. À l'aide de l'éditeur Swagger (https://editor .swagger.io), importez le fichier crAPI Swagger que nous avons créé dans le module précédent. Dans la spécification crAPI Swagger, nous pouvons voir qu'il existe plusieurs chemins différents pour les points de terminaison commençant par /identity, /community et /workshop.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/tvmP92upTSKuuzFbal4A_UsingAPI1.PNG)

L'utilisation de l'éditeur Swagger nous permet d'avoir une représentation visuelle des points de terminaison API de notre cible. En parcourant et en développant les demandes, vous pouvez voir le point de terminaison, les paramètres, le corps de la demande et des exemples de réponses. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/cajspgGBROmR18GBQaTz_UsingAPI2.PNG)

 La demande crAPI POST ci-dessus nécessite des valeurs JSON envoyées sur le corps de la demande et l'on s'attend à ce que ces valeurs soient sous la forme d'une chaîne. L'examen de la documentation nous donne également l'occasion de voir l'objectif des différents points de terminaison ainsi que certains des schémas de dénomination utilisés pour les objets de données. L'examen de la documentation vous mènera à des requêtes intéressantes à cibler dans vos attaques. Même à ce stade, vous pourriez découvrir des vulnérabilités potentielles telles que l'exposition excessive aux données.

Modification des variables de collection Postman
================================================

Lorsque vous commencez à travailler avec une nouvelle collection dans Postman, c'est toujours une bonne idée d'avoir un aperçu du terrain, en vérifiant les variables de collection. Vous pouvez vérifier vos variables de collection Postman en utilisant l'éditeur de collection.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/j96Inp9PR2mEM4Zz1I9L_UsingAPI8.PNG)

Vous pouvez accéder à l'éditeur de collection en utilisant votre collection crAPI Swagger, en sélectionnant les trois cercles sur le côté droit d'une collection et en choisissant "Modifier". La sélection de l'onglet Variables nous montrera que la variable "baseUrl" est utilisée.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/YZWAKkLHTMySimU9EzyF_UsingAPI10.PNG)

Assurez-vous que la valeur actuelle de baseUrl correspond à l'URL de votre cible. Si votre cible est localhost, elle doit correspondre à l'image ci-dessus. Si votre cible est le laboratoire ACE, la valeur actuelle doit être http://crapi.apisec.ai. Une fois que vous aurez mis à jour une valeur dans l'éditeur, vous devrez utiliser le bouton Enregistrer situé en haut à droite de Postman.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/qySSCGnSGJmbUtASYUpQ_UsingAPI11.PNG)

Mise à jour de l'autorisation de collecte du facteur
====================================================

Afin d'utiliser Postman pour effectuer des requêtes API autorisées, nous devrons ajouter un jeton valide à nos requêtes. Cela peut être fait pour toutes les demandes d'une collection en ajoutant une méthode d'autorisation à la collection. En utilisant l'onglet Autorisation, dans l'éditeur de collection, nous devrons sélectionner le bon type d'autorisation. Pour crAPI, ce sera un Bearer Token.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/asqDlJ4OR4qY9bg8ak1b_UsingAPI9.PNG)

Les jetons sont généralement fournis après une tentative d'authentification réussie. Pour crAPI, nous pourrons obtenir un jeton porteur une fois que nous nous authentifierons avec succès avec la requête POST à ​​/identity/api/auth/login. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/1XtUYT0FRJe0ZLeYI2aZ_UsingAPI12.PNG)

Accédez à la demande de connexion POST dans votre collection et mettez à jour les valeurs pour "email" et "password" pour qu'elles correspondent au compte que vous avez créé. Si vous ne vous en souvenez pas, vous devrez revenir en arrière et créer un compte. Une fois que vous vous êtes authentifié avec succès, vous devriez recevoir une réponse contenant un jeton Bearer comme celui vu ci-dessus. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/PPWP8Yq8SqqPdjBEmK4h_UsingAPI13.PNG)

Copiez le jeton trouvé entre les guillemets et collez cette valeur dans l'onglet d'autorisation de l'éditeur de collection. Assurez-vous d'enregistrer la mise à jour dans la collection. Vous devriez maintenant pouvoir utiliser l'API crAPI en tant qu'utilisateur autorisé et effectuer des requêtes réussies à l'aide de Postman.

OWASP API 3 : Exposition excessive des données 
===============================================

Une exposition excessive des données se produit lorsqu'un fournisseur d'API renvoie un objet de données complet, généralement en fonction du client pour filtrer les informations dont il a besoin. Du point de vue d'un attaquant, le problème de sécurité ici n'est pas que trop d'informations sont envoyées, mais plutôt la sensibilité des données envoyées. Cette vulnérabilité peut être découverte dès que vous êtes en mesure de commencer à faire des demandes. Les demandes d'API intéressantes incluent les comptes d'utilisateurs, les publications sur les forums, les publications sur les réseaux sociaux et les informations sur les groupes (comme les profils d'entreprise).

Ingrédients pour une exposition excessive des données :

-   Une réponse contenant plus d'informations que ce qui a été demandé
-   Informations sensibles pouvant être exploitées dans des attaques plus complexes

Si un fournisseur d'API répond avec un objet de données complet, la première chose qui pourrait vous avertir d'une exposition excessive des données est simplement la taille de la réponse. 

Demande

GET /api/v1/user?=CloudStrife

Réponse

200 OKHTTP 1.1

--couper--

{"id": "5501",

"fname": "Nuage",

"lname": "Conflit",

"privilège": "utilisateur",

"représentant": [

     "name": "Don Corénéo",

     "id": "2203",

     "email": " dcorn@gmail.com ",

     "privilège": "administrateur",

     "MFA": faux\
     ]

}

Dans la réponse, nous voyons que non seulement les informations de l'utilisateur demandé sont fournies, mais aussi les données sur l'administrateur qui a créé le compte de l'utilisateur. Inclure les informations de l'administrateur dans une demande comme celle-ci est un exemple d'exposition excessive des données, car cela va au-delà de ce qui a été demandé et expose des informations sensibles telles que le nom, l'e-mail, le rôle et le statut multifacteur de l'administrateur.

Maintenant, si nous revenons à crAPI, examinons la spécification à l'aide de l'éditeur Swagger pour voir si nous pouvons repérer des demandes intéressantes potentielles. Étant donné que nous recherchons des données qui nous sont renvoyées, nous nous concentrerons sur les requêtes crAPI GET. La première de ces requêtes répertoriées dans la documentation crAPI Swagger est GET /identity/api/v2/user/dashboard.![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/T1xPrikRYGXHldfjZ0ts_UsingAPI3.PNG)

Le but de cette requête est de remplir le tableau de bord d'un utilisateur. Il existe de nombreuses informations intéressantes, mais les informations ici seront spécifiques au demandeur, en fonction de son jeton d'accès. Cela nous donne une idée de certains des schémas de dénomination des clés d'objet et des informations potentiellement sensibles à rechercher. Des informations telles que "id", "name", "email", "number", "available_credit" et "role" seraient toutes intéressantes à découvrir sur les autres utilisateurs. Nous devrions donc examiner d'autres demandes pour voir si certaines incluent l'une d'entre elles. 

Si vous réfléchissez aux différents types de points de terminaison ( /identité, /communauté et /atelier), réfléchissez à ceux qui sont susceptibles d'impliquer les informations d'autres utilisateurs. La communauté ressemble à quelque chose qui impliquerait d'autres utilisateurs, alors regardons une requête GET associée.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/1Zyr5SxHSgq1IBYjv1J9_UsingAPI4.PNG)

Cette requête GET est utilisée pour mettre à jour le forum de la communauté. Découvrez certaines des données renvoyées :

 ![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/lkwPK1QOTuhy0sOGGoGT_UsingAPI5.PNG)

Dans ce post du forum, un objet "author" avec "nickname", "email" et "vehicleid" est retourné. Cela pourrait être intéressant. Maintenant, c'est un excellent exemple pour voir ce qui est représenté visuellement dans un navigateur Web par rapport à ce qui existe dans la réponse de l'API dans les coulisses.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/x385f6RkTTdgluIktSTi_UsingAPI6.PNG)

Comme vous pouvez le constater, aucune des informations sensibles intéressantes ne se trouve dans le forum de la communauté. Cependant, si nous interceptons les demandes d'API qui remplissent les messages récents sur le forum, nous constaterons que le fournisseur envoie un objet de données complet en fonction du client pour filtrer les informations sensibles. Dépendre du client pour filtrer les informations ne nous empêchera pas de pouvoir capturer des données sensibles.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/DM563qiCTrCUoZGtZa7X_UsingAPI7.PNG)

L'utilisation du répéteur de Burp Suite pour la requête GET à /community/api/v2/community/posts/recent révèle toutes les données sensibles que nous espérions trouver. Cette instance d'Exposition excessive de données révèle des noms d'utilisateur, des e-mails, des identifiants et des identifiants de véhicule qui peuvent tous s'avérer utiles lors d'attaques supplémentaires. 

Vous devriez maintenant avoir une assez bonne idée de la façon de commencer à utiliser une API comme prévu. Cela aide vraiment à savoir comment une API répondra aux demandes échouées et réussies. Faites-vous une idée des différentes fonctions prévues par l'API, afin de mieux comprendre où concentrer vos efforts d'attaque. N'oubliez pas qu'à ce stade, vous pouvez déjà découvrir des vulnérabilités cruciales.
