Ingénierie inverse d'une API
============================

##### [Analyse des points finaux](https://university.apisec.ai/products/api-penetration-testing/categories/2150251353)

Dans ce module, je vais discuter d'un processus qui peut être utilisé pour rétroconcevoir une API. Si une API n'est pas documentée ou si la documentation n'est pas disponible, vous devrez créer votre propre collection de requêtes. Je vais démontrer deux méthodes différentes qui peuvent être déployées pour construire notre propre collection. Tout d'abord, nous utiliserons Postman pour collecter les demandes d'API et créer manuellement une collection. Cette méthode demande souvent plus de temps mais vaut la peine d'être connue au cas où vous seriez dans le pétrin. La deuxième méthode que nous apprendrons consiste à créer automatiquement une spécification d'API à l'aide de mitmproxy2swagger.

Créer une collection dans Postman
=================================

Dans le cas où il n'y a pas de documentation ni de fichier de spécification, vous devrez désosser l'API en fonction de vos interactions avec elle. Le mappage d'une API avec plusieurs points de terminaison et quelques méthodes peut rapidement devenir une surface d'attaque assez importante. Pour gérer ce processus, créez les requêtes sous une collection afin de pirater complètement l'API. Postman peut vous aider à suivre toutes ces demandes. Il existe deux façons de procéder manuellement à l'ingénierie inverse d'une API avec Postman. Une façon consiste à construire chaque requête. Bien que cela puisse être un peu lourd, cela vous permettra d'ajouter les demandes précises qui vous intéressent. L'autre méthode consiste à proxy le trafic Web via Postman, puis à l'utiliser pour capturer un flux de demandes. Ce processus facilite grandement la création de requêtes dans Postman, mais vous devrez supprimer ou ignorer les requêtes non liées..

Tout d'abord, lançons Postman.

$ postier

Ensuite, créez un espace de travail pour enregistrer vos collections. Pour ce cours, nous utiliserons l'espace de travail ACE. 

Pour créer votre propre collection dans Postman avec le proxy, utilisez le bouton Capturer les demandes, situé en bas à droite de la fenêtre Postman.  

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/V5C8ANdETAeCgv6MABrC_Reverse1.PNG)

Dans la fenêtre Capturer les demandes, sélectionnez Activer le proxy. Le port doit correspondre au numéro configuré dans FoxyProxy (5555). Ensuite, activez Postman Proxy, ajoutez votre URL cible au champ "L'URL doit contenir", puis cliquez sur le bouton Démarrer la capture.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/0Dn6R31mSkcUAVQP0vaA_Reverse2.PNG)

Ouvrez votre navigateur Web, accédez à la page de destination de crAPI et utilisez FoxyProxy pour activer l'option Postman. Vous pouvez maintenant utiliser méticuleusement l'application Web comme prévu. Méticuleux, car vous souhaitez capturer chaque élément de fonctionnalité disponible dans l'application. Cela signifie utiliser toutes les fonctionnalités de la cible. Cliquez sur les liens, créez un compte, connectez-vous au compte, visitez votre profil, publiez des commentaires dans un forum, etc. Cliquez essentiellement sur toutes les choses, mettez à jour les informations là où vous le pouvez et explorez l'application Web dans son intégralité. Votre niveau d'utilisation de l'application Web aura un effet domino sur les points de terminaison et les requêtes que vous testerez ultérieurement. Par exemple, si vous deviez effectuer les différentes actions de l'application Web, mais que vous oubliez de tester les points de terminaison de la communauté, vous aurez un angle mort dans la surface d'attaque de l'API. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/cyO2LccsSH2oTymcoqca_Reverse3.PNG)

Par exemple, assurez-vous d'effectuer toutes les actions sur la page mon profil :

-   ajouter une photo de profil
-   télécharger une vidéo personnelle
-   modifier l'adresse e-mail du compte
-   utilisez les trois points horizontaux pour trouver des options supplémentaires pour :
    -   changer de vidéo
    -   changer le nom de la vidéo
    -   partager une vidéo avec la communauté

Utilisez également le serveur crAPI MailHog pour tirer le meilleur parti de cette application. Le serveur MailHog est situé sur le port 8025 et sera utilisé pour enregistrer les véhicules des utilisateurs, réinitialiser les mots de passe et où les jetons de "changement d'e-mail" sont envoyés.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/RvbXZkznTXesyqp0W16y_Reverse4.PNG)

Une fois que vous avez capturé toutes les fonctionnalités que vous pouvez trouver avec l'exploration manuelle, vous voudrez arrêter le proxy. Ensuite, il est temps de construire la collection crAPI. Commencez par créer une nouvelle collection en sélectionnant le nouveau bouton (en haut à gauche de Postman), puis choisissez Collection.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/kznOoCoQRKwnauhqLCpa_Reverse5.PNG)

Allez-y et renommez la collection en crAPI Proxy Collection . Revenez à la session de débogage du proxy et ouvrez l'onglet Demandes.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/X4WQgyYSQ7eUohdrgHeQ_Reverse6.PNG)

Sélectionnez toutes les demandes que vous avez capturées et utilisez le lien "ajouter à la collection" mis en évidence ci-dessus. Sélectionnez la collection de proxys crAPI et organisez les demandes par points de terminaison. Avec toutes les demandes capturées ajoutées à la collection de proxy crAPI, vous pouvez organiser la collection en renommant toutes les demandes et en regroupant les demandes similaires dans des dossiers. Avant d'aller trop loin dans cet exercice, je vous recommande de consulter les instructions de documentation automatisées ci-dessous. 

Documentation automatique
-------------------------

Tout d'abord, nous commencerons par proxy tout le trafic des applications Web à l'aide de mitmweb.

Utilisez simplement un terminal et lancez :\
$ mitmweb

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/6KuYXxA4RqKE8PzXIvPn_mitmproxy.png)

Cela créera un écouteur proxy utilisant le port 8080. Vous pouvez ensuite ouvrir un navigateur et utiliser FoxyProxy pour envoyer votre navigateur par proxy au port 8080 à l'aide de notre option Burp Suite.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/1pY8hBghTnNH9r97h1Yi_FoxyProxySet.png)

Une fois le proxy configuré, vous pouvez à nouveau utiliser l'application Web cible comme prévu.

*Remarque : si vous faites de l'ingénierie inverse sur crapi.apisec.ai, vous pouvez rencontrer des problèmes de certificat si vous n'avez pas encore ajouté le certificat mitmweb. Veuillez revenir à Kali Linux and More MITMweb Certificate Setup pour obtenir des instructions.*

Chaque requête créée à partir de vos actions sera capturée par le proxy mitmweb. Vous pouvez voir le trafic capturé en utilisant un navigateur pour visiter le serveur Web mitmweb situé à l'adresse http://127.0.0.1:8081.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/cfW71QnfSeCLslFvzQgA_mitmproxybrowser.png)

Continuez à explorer l'application Web cible jusqu'à ce qu'il n'y ait plus rien à faire. Une fois que vous avez épuisé ce qui peut être fait avec votre application Web cible, revenez au serveur Web mitmweb et cliquez sur Fichier > Enregistrer  pour enregistrer les requêtes capturées.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/eEp9PvDRHedaSZsYqI8g_mitmproxySave.png)

Sélectionner  Enregistrer créera un fichier appelé flux. Nous pouvons utiliser le fichier "flows" pour créer notre propre documentation API. À l'aide d'un excellent outil appelé mitmproxy2swagger, nous pourrons transformer notre trafic capturé en un fichier YAML Open API 3.0 pouvant être visualisé dans un navigateur et importé en tant que collection dans Postman.

Tout d'abord, exécutez ce qui suit :

$sudo mitmproxy2swagger -i /Téléchargements/flux -o spec.yml -p [http://crapi.apisec.ai](http://crapi.apisec.ai/)  -f flux

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/4TryEayCTsCoLdiBTsZS_mitmproxy2swaggerStep1.png)

Après l'avoir exécuté, vous devrez modifier le fichier spec.yml pour voir si mitmproxy2swagger a ignoré trop de points de terminaison. L'extraction de spec.yml révèle que plusieurs points de terminaison ont été ignorés et que le titre du fichier peut être mis à jour.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/WTp4ptmR4S6Xhll23NNw_mitmproxy2swagger01.png)

Mettez à jour le fichier YAML afin que "ignore :" soit supprimé des points de terminaison que vous souhaitez inclure. Une fois que vous avez modifié le fichier, il devrait ressembler à l'image ci-dessous. Notez que le "titre" a été mis à jour en "crAPI Swagger" et que les points de terminaison ne contiennent plus "ignore :".

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/PWw45TD2RkS1rnZ2z4gJ_mitmproxy2swaggerStep2.png)

 Enregistrez le fichier spec.yml mis à jour et exécutez à nouveau mitmproxy2swagger. Cette fois-ci, ajoutez le drapeau "--examples" pour améliorer la documentation de votre API.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/y7ouQ9QSQoS0LHJr9yuT_mitmproxy2swaggerStep3.png)

Après avoir exécuté avec succès mitmproxy2swagger une deuxième fois, votre documentation rétro-conçue devrait être prête. Vous pouvez valider la documentation en visitant  <https://editor.swagger.io/>  et en important votre fichier de spécifications dans l'éditeur Swagger. Utilisez  File>Import file et sélectionnez votre fichier spec.yml. Si tout s'est déroulé comme prévu, vous devriez voir quelque chose comme l'image ci-dessous. C'est une assez bonne indication de succès, mais pour être sûr que nous pouvons également importer ce fichier en tant que collection Postman, nous pouvons ainsi nous préparer à attaquer l'API cible.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/3bNj0h5nQPyQELqabkNv_crapiSwagger1.png)

 Pour importer ce fichier en tant que collection, nous devrons ouvrir Postman. En haut à gauche de votre espace de travail Postman, vous pouvez cliquer sur le bouton "Importer". Ensuite, sélectionnez le fichier spec.yml et importez la collection.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/KzKQLRSXRdNpGtwrtdSo_mitmproxy2swaggerStep4.png)

Une fois que vous avez importé le fichier, vous devriez voir une collection d'API relativement simple qui peut être utilisée pour analyser l'API cible et l'exploiter avec de futures attaques.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/u4Ukhy2OQGGqO6qJYQA5_crapiCollection.png)

Avec une collection préparée, vous devriez maintenant être prêt à utiliser l'API cible telle qu'elle a été conçue. Cela vous permettra de voir les différents points de terminaison et de comprendre ce qui est nécessaire pour faire des demandes réussies. Dans le prochain module, nous commencerons à travailler avec l'API et apprendrons à analyser diverses demandes et réponses.
