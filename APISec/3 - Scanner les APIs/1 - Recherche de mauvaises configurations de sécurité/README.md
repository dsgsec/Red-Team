Recherche de mauvaises configurations de sécurité
=================================================

##### [API d'analyse](https://university.apisec.ai/products/api-penetration-testing/categories/2150251355)

Introduction
============

Une fois que vous avez découvert une API et que vous l'avez utilisée comme prévu, vous pouvez effectuer une analyse de vulnérabilité de base. Une bonne raison d'effectuer vos tests dans cet ordre est d'éviter que l'une de vos analyses ne déclenche un contrôle de sécurité tel qu'un WAF qui entraîne le blocage de votre trafic. Idéalement, une analyse de vulnérabilité vous aidera à trouver des faiblesses que vous pourrez ensuite tester, confirmer et exploiter. De manière réaliste, les analyses de vulnérabilité sont rarement précises à 100 % et identifient rarement, voire jamais, tous les problèmes présents. Ainsi, nous n'utiliserons pas les analyses de vulnérabilité pour déterminer toutes les faiblesses d'une application, mais à la place, nous utiliserons les résultats de l'analyse pour guider et concentrer nos tests.

Lorsque les analyses de vulnérabilité sont appliquées de manière générique aux API Web, le résultat le plus courant est de recevoir des résultats faussement négatifs. Des résultats faussement négatifs se produisent lorsque les analyses de vulnérabilité ne détectent ni ne signalent des problèmes existants. Pour la plupart des organisations, cela peut entraîner un faux sentiment de sécurité car les analyses sont revenues sans aucune preuve de faiblesses actuelles. L'état actuel de nombreux scanners de vulnérabilités gratuits et payants est qu'ils n'ont pas été conçus pour les API Web et ne détectent souvent pas la plupart des vulnérabilités répertoriées dans le Top 10 de la sécurité des API OWASP. Ces scanners de vulnérabilités, cependant, font un travail décent pour détecter [API7 :2019 Mauvaise configuration de la sécurité](https://github.com/OWASP/API-Security/blob/master/2019/en/src/0xa7-security-misconfiguration.md "Projet de sécurité de l'API OWASP"). La mauvaise configuration de la sécurité comprend des correctifs système manquants, des fonctionnalités inutiles activées, un manque de cryptage de transit sécurisé, des en-têtes de sécurité faibles, des messages d'erreur détaillés et des erreurs de configuration de la politique de partage des ressources cross-origin (CORS). Dans ce module, nous nous concentrerons sur la configuration d'OWASP ZAP pour découvrir en profondeur les erreurs de configuration de la sécurité des API et tirer le meilleur parti d'un scanner de vulnérabilité.

Tout d'abord, si vous souhaitez voir comment une analyse générique peut entraîner des résultats faussement négatifs, vous pouvez en faire l'expérience par vous-même. Vous pouvez le faire en scannant crAPI avec Nikto, un scanner de vulnérabilité d'application Web. Ouvrez un terminal et lancez :

$nikto -h [http://crapi.apisec.ai](http://crapi.apisec.ai/)

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/1JfV14LsRYGj7k5utleN_Niktoscan.png)

Si vous exécutez cette analyse, vous devriez remarquer quelques choses. L'analyse Nikto révèle que le serveur d'applications Web exécute la plate-forme OpenResty avec la version. De plus, les en-têtes X-Frame-Options et X-XSS-Protection sont manquants ou mal configurés. Outre ces découvertes, les résultats manquent le gambit des vulnérabilités liées à l'API que contient crAPI. Gardez à l'esprit que l'application crAPI a été conçue avec toutes les vulnérabilités du Top 10 de la sécurité de l'API OWASP.

Analyser les API avec OWASP ZAP
===============================

Comme Nikto, une analyse générique automatisée OWASP ZAP rencontrera les mêmes problèmes avec des résultats faux négatifs. Cependant, vous pouvez configurer une analyse ZAP pour mieux travailler avec les API Web. La première chose que nous allons faire est de lancer une analyse non authentifiée de la surface d'attaque. Nous pouvons brancher l'URL cible, mais pour améliorer ces résultats et nous assurer de tout atteindre, nous pouvons importer le fichier de spécification de l'API de la cible.

 ![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/3QhpAQLTQqG1Sb8MPTft_ScanningAPIs1.PNG)

Vous pouvez le faire en sélectionnant l'importation et en choisissant le fichier de spécification approprié. Pour crAPI, sélectionnez le fichier specs.yml que nous avons créé lors de la rétro-ingénierie de crAPI et assurez-vous d'ajouter l'URL que vous attendez (http://crapi.apisec.ai ou http://127.0.0.1:8888). 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/oEsdxfxQGiPyRW2Nxobg_ScanningAPIs2.PNG)

Après avoir ajouté le chemin du fichier et l'URL cible, sélectionnez l'importation. Vous pouvez maintenant voir la fenêtre Sites remplie avec les points de terminaison et les demandes d'API de la cible.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/2APuCeXwS6S6Npz32QhO_ScanningAPIs3.PNG)

 Vous pouvez cliquer avec le bouton droit sur la racine, dans ce cas [http://crapi.apisec.ai,](http://crapi.apisec.ai%2C/)  et choisir de faire une analyse active. Une fois cette analyse terminée, vous pouvez trouver les résultats sous l'onglet Alertes. Ici, vous trouverez généralement une mauvaise configuration de sécurité affectant l'hôte cible.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/tXyxubIySxasnFlUXUtw_ScanningAPIs5.PNG)

La prochaine étape que vous pouvez effectuer pour améliorer les résultats de l'analyse consiste à effectuer une analyse authentifiée. Le moyen le plus simple d'effectuer une analyse authentifiée consiste à utiliser l'option d'exploration manuelle.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/FMTNLZyOSGKGoPJABPJ4_ScanningAPIs7.PNG)

Définissez l'URL de votre cible, assurez-vous que le HUD est activé et choisissez "Lancer le navigateur". 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Ac8Y0rHqSqile9WdaoWu_ScanningAPIs8.PNG)

Une fois que vous avez choisi d'explorer manuellement, vous devriez voir le lancement du HUD dans un navigateur. Ici, vous pouvez sélectionner "Continuer vers votre cible". Semblable au travail que nous avons effectué lors du module "Reverse Engineering APIs", nous allons parcourir et utiliser l'application web en tant qu'utilisateur final.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/967oh1osRLeGnyntOPWb_ScanningAPIs9.PNG)

Effectuez à nouveau toutes ces actions. Ouvrez un autre compte, connectez-vous et utilisez les différentes fonctionnalités. Assurez-vous d'utiliser le HUD pour effectuer certaines actions. En haut à gauche du HUD, vous pouvez ajouter votre cible à la portée des tests. Une fois que vous vous êtes authentifié auprès de l'application et que vous avez effectué un ensemble d'actions de base, vous pouvez effectuer une analyse active.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/WkGjSKhARZKFnh8sCMho_ScanningAPIs11.PNG)

Sur le côté droit du HUD, vous pouvez activer le mode d'attaque. Cela commencera à analyser et à effectuer des tests authentifiés de la cible. Selon l'échelle de l'application Web que vous ciblez, cette analyse peut prendre un certain temps. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/xZQvLSUcSQue7u0Gzmew_ScanningAPIs12.PNG)

Comme vous pouvez le voir ci-dessus, il existe de nombreuses découvertes plus intéressantes que l'analyse générique initiale qui a été exécutée. À partir de là, vous devriez étudier les résultats. Travaillez-les et déterminez quels sont les résultats réels et quels sont les faux positifs. Une chose importante à noter est que crAPI est vulnérable à tous les Top 10 de la sécurité des API OWASP. Sur la base des résultats actuels de l'analyse, nous pouvons voir que des erreurs de configuration de sécurité et des vulnérabilités d'injection peuvent être présentes. Même avec une analyse authentifiée, il nous manque de nombreuses autres vulnérabilités présentes dans l'application, c'est pourquoi nous devrons développer des techniques de test supplémentaires dans les modules à venir. Ensuite, nous concentrerons nos efforts de test sur les deux principaux sujets du Top 10 de la sécurité des API OWASP, l'authentification et l'autorisation.
