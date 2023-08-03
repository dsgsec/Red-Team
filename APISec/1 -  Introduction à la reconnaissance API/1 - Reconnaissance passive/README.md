Reconnaissance passive
======================

##### [Reconnaissance API](https://university.apisec.ai/products/api-penetration-testing/categories/2150259092)

Reconnaissance passive
----------------------

La reconnaissance API passive  consiste à obtenir des informations sur une cible sans interagir directement avec les systèmes de la cible. Lorsque vous adoptez cette approche, votre objectif est de trouver et de documenter des informations publiques sur la surface d'attaque de votre cible. 

En règle générale, la reconnaissance passive exploite l'intelligence open source (OSINT), qui est des données collectées à partir de sources accessibles au public. Vous serez à la recherche de points de terminaison d'API, d'informations d'identification exposées, d'informations sur la version, de documentation sur l'API et d'informations sur l'objectif commercial de l'API. Tous les points de terminaison d'API découverts deviendront vos cibles ultérieurement, lors de la reconnaissance active. Les informations relatives aux informations d'identification vous aideront à tester en tant qu'utilisateur authentifié ou, mieux, en tant qu'administrateur. Les informations de version vous aideront à vous informer sur les actifs potentiels inappropriés et d'autres vulnérabilités passées. La documentation de l'API vous dira exactement comment tester l'API cible. Enfin, découvrir l'objectif commercial de l'API peut vous donner un aperçu des failles potentielles de la logique métier.

Lorsque vous collectez OSINT, il est tout à fait possible que vous tombiez sur une exposition de données critiques, telles que des clés API, des informations d'identification, des jetons Web JSON (JWT) et d'autres secrets qui conduiraient à une victoire instantanée. D'autres découvertes à haut risque incluraient des informations personnelles divulguées ou des données utilisateur sensibles telles que le SSN, les noms complets, les adresses e-mail et les informations de carte de crédit. Ces types de découvertes doivent être documentées et signalées immédiatement car elles présentent une faiblesse critique valable.

Nous allons maintenant découvrir quelques techniques de reconnaissance passive qui peuvent être utilisées pour découvrir des API. Y compris Google Dorking, Git Dorking, les référentiels d'API, la Way Back Machine et Shodan.

Google Dorking
--------------

Même sans aucune technique Dorking, trouver une API en tant qu'utilisateur final pourrait être aussi simple qu'une recherche rapide.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/3QiE3ZaXRLWKOYwRfCre_redditapisearch.png)

*Recherche Google pour l'API de Reddit.*

Cependant, il se peut que vous n'obteniez parfois pas les résultats exacts que vous espériez. Si vous obtenez trop de résultats non pertinents, vous pouvez déployer certaines techniques de Google Dorking pour découvrir plus efficacement les API.

| Requête Google Dorking                                  | Résultats attendus                                                                                                                                                              |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| inurl :"/wp-json/wp/v2/users"                           | Trouve tous les répertoires d'utilisateurs de l'API WordPress disponibles publiquement.                                                                                         |
| intitle:"index.of" intext:"api.txt"                     | Trouve les fichiers de clés d'API accessibles au public.                                                                                                                        |
| inurl :"/api/v1" intext :"index de /"                   | Trouve des répertoires d'API potentiellement intéressants.                                                                                                                      |
| ext:php inurl:"api.php?action="                         | Trouve tous les sites présentant une vulnérabilité d'injection SQL XenAPI. (Cette requête a été publiée en 2016 ; quatre ans plus tard, il y a actuellement 141 000 résultats.) |
| intitle:"index of" api_key OU "clé api" OU apiKey -pool | C'est l'une de mes requêtes préférées. Il répertorie les clés API potentiellement exposées.

GitDorking
----------

Que votre cible réalise ou non son propre développement, il vaut la peine de consulter GitHub ( www.github.com ) pour la divulgation d'informations sensibles. Les développeurs utilisent GitHub pour collaborer sur des projets logiciels. La recherche d'OSINT sur GitHub pourrait révéler les capacités, la documentation et les secrets de l'API de votre cible, tels que les clés API, les mots de passe et les jetons, qui pourraient s'avérer utiles lors d'une attaque. Semblable à Google Dorking, avec GitHub, vous pouvez spécifier des paramètres tels que :

nom de fichier : swagger.json

extension : .json

Commencez par rechercher sur GitHub le nom de votre organisation cible associé à des types d'informations potentiellement sensibles, tels que "clé api", "clés api", "apikey", "autorisation : porteur", "access_token", "secret" ou "token". ." Examinez ensuite les différents onglets du référentiel GitHub pour découvrir les points de terminaison de l'API et les faiblesses potentielles. Analysez le code source dans l'onglet Code, recherchez les bogues dans l'onglet Problèmes et examinez les modifications proposées dans l'onglet Demandes d'extraction.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/vJHflsacRw6gsCDkbd2c_GithubExposed.png)

Code contient le code source actuel, les fichiers Lisez-moi et d'autres fichiers. Cet onglet vous fournira le nom du dernier développeur qui s'est engagé sur le fichier donné, le moment où cet engagement s'est produit, les contributeurs et le code source réel. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Dn6kLSBGmQ2mI5S484Ao_GithubCode.png)

À l'aide de l'onglet Code, vous pouvez consulter le code dans sa forme actuelle ou utiliser ctrl -F pour rechercher des termes susceptibles de vous intéresser (tels que API, clé et secret). De plus, affichez l'historique des validations du code en utilisant le bouton Historique situé près du coin supérieur droit de l'image ci-dessus. Si vous avez rencontré un problème ou un commentaire qui vous a amené à croire qu'il y avait autrefois des vulnérabilités associées au code, vous pouvez rechercher des commits historiques pour voir si les vulnérabilités sont toujours visibles.

Lorsque vous examinez un commit, utilisez le bouton Fractionner pour voir une comparaison côte à côte des versions de fichier afin de trouver l'endroit exact où une modification du code a été apportée. ![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/I7Zza4TSRgG0dQ5NFdQg_GithubHistory.png)Le bouton Split (en haut à droite de l'image ci-dessus) vous permet de séparer le code précédent (à gauche) et le code mis à jour (à droite). Ici, vous pouvez voir un engagement dans une application qui a supprimé la clé d'API Google Maps du code, révélant à la fois la clé et le point de terminaison d'API pour lequel elle était utilisée.

L'onglet Problèmes est un espace où les développeurs peuvent suivre les bogues, les tâches et les demandes de fonctionnalités. Si un problème est ouvert, il y a de fortes chances que la vulnérabilité soit toujours présente dans le code (voir Figure 6-9).

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/S0xHhF3QWyPOOVFIewWQ_GithubIssue.png)

Ceci est un exemple d'un problème GitHub ouvert qui fournit l'emplacement exact d'une clé API exposée dans le code d'une application. Si le problème est fermé, notez la date du problème, puis recherchez dans l'historique des commits toute modification à ce moment-là. 

L'onglet Demandes d'extraction est un endroit qui permet aux développeurs de collaborer sur les modifications du code. Si vous examinez ces modifications proposées, vous pourriez parfois avoir de la chance et trouver une exposition d'API en cours de résolution. Comme cette modification n'a pas encore été fusionnée avec le code, nous pouvons facilement voir que la clé API est toujours exposée sous l'onglet Fichiers modifiés. L'onglet Fichiers modifiés révèle la section de code que le développeur tente de modifier. 

Si vous ne trouvez pas de faiblesses dans un dépôt GitHub, utilisez-le plutôt pour développer le profil de votre cible. Prenez note des langages de programmation utilisés, des informations sur les points de terminaison de l'API et de la documentation d'utilisation, qui s'avéreront toutes utiles à l'avenir.

TruffePorc 
-----------

TruffleHog est un excellent outil pour découvrir automatiquement les secrets exposés. Vous pouvez simplement utiliser l'exécution Docker suivante pour lancer une analyse TruffleHog du Github de votre cible.

 $ sudo docker run -it -v "$PWD:/pwd" trufflesecurity/trufflehog:latest github --org=target-name

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/01s4gYuoQmq9GgdZg4oG_TruffleHogv3.png)

Dans l'exemple ci-dessus, vous pouvez voir que l'organisation ciblée était Venmo et les résultats de l'analyse indiquent les URL qui doivent faire l'objet d'une enquête pour les secrets potentiellement divulgués. En plus de rechercher Github, TruffleHog peut également être utilisé pour rechercher des secrets dans d'autres sources telles que Git, Gitlab, Amazon S3, le système de fichiers et Syslog. Pour explorer ces autres options, utilisez le drapeau "-h". Pour plus d'informations, consultez  <https://github.com/trufflesecurity/trufflehog> . 

Répertoires d'API
=================

Programmableweb.com est une source incontournable d'informations relatives à l'API ( <https://www.programmableweb.com/apis/directory> ). Pour en savoir plus sur les API, vous pouvez utiliser leur API University. Pour recueillir des informations sur votre cible, utilisez l'API Directory, une base de données consultable de plus de 23 000 API. Attendez-vous à trouver des points de terminaison d'API, des informations sur la version, des informations sur la logique métier, l'état de l'API, le code source, les SDK, des articles, la documentation de l'API et un journal des modifications.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/Pl5zVW8Rde4fCFd32jz3_twilioapidirectory.png)

*Page de l'API Twilio sur programmableweb.com.*

Cliquez sur les différents onglets de la liste des répertoires et notez les informations que vous trouvez. Pour voir l'emplacement du point de terminaison de l'API, l'emplacement du portail et le modèle d'authentification. Pour afficher des informations sur l'historique des versions des API, sélectionnez une version spécifique répertoriée sous l'onglet Versions. Dans ce cas, les liens du portail et du point de terminaison mènent également à la documentation de l'API. Dans le cas de Twilio, vous pouvez voir toutes les spécifications liées à la version actuelle de l'API REST.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/8Zn6Z9PIRse67JCxJz8s_twilioapidirectory2.png)

*La page des spécifications de Twilio sur programmableweb.com.*

Sur la page des spécifications de Twilio, vous pouvez apprendre toutes sortes d'informations utiles sur l'API. Par exemple, l'URL du point de terminaison de l'API est répertoriée, le forum de l'API, l'URL d'assistance aux développeurs, le modèle d'authentification, etc. sont tous répertoriés sur cette page. Au bas de la page Spécifications, vous pouvez également voir des articles liés à l'API et aux développeurs de l'API, qui pourraient tous s'avérer utiles lors de l'attaque de l'API.

Shodan
------

Shodan est le moteur de recherche incontournable pour les appareils accessibles depuis Internet. Shodan analyse régulièrement l'ensemble de l'espace d'adressage IPv4 pour les systèmes avec des ports ouverts et rend publiques les informations collectées sur https://shodan.io . Vous pouvez utiliser Shodan pour découvrir des API externes et obtenir des informations sur les ports ouverts de votre cible, ce qui est utile si vous ne disposez que d'une adresse IP ou du nom d'une organisation à partir de laquelle travailler. Comme avec Google dorks, vous pouvez rechercher Shodan avec désinvolture en entrant le nom de domaine ou les adresses IP de votre cible ; vous pouvez également utiliser des paramètres de recherche comme vous le feriez lors de la rédaction de requêtes Google. Le tableau suivant montre quelques requêtes Shodan utiles.

| Requêtes Shodan                      | But                                                                                                                                                                                                                                        |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| nom d'hôte :"nomcible.com"           | L'utilisation du nom d'hôte effectuera une recherche Shodan de base pour le nom de domaine de votre cible. Cela doit être combiné avec les requêtes suivantes pour obtenir des résultats spécifiques à votre cible.                        |
| "type de contenu : application/json" | Les API doivent avoir leur type de contenu défini sur JSON ou XML. Cette requête filtrera les résultats qui répondent avec JSON.                                                                                                           |
| "type de contenu : application/xml"  | Cette requête filtrera les résultats qui répondent avec XML.                                                                                                                                                                               |
| "200 d'accord"                       | Vous pouvez ajouter "200 OK" à vos requêtes de recherche pour obtenir des résultats dont les requêtes ont abouti. Cependant, si une API n'accepte pas le format de la demande de Shodan, elle émettra probablement une réponse 300 ou 400. |
| "wp-json"                            | Cela recherchera des applications Web à l'aide de l'API WordPress.             

Vous pouvez ajouter "200 OK" à vos requêtes de recherche pour obtenir des résultats dont les requêtes ont abouti. Cependant, si une API n'accepte pas le format de la demande de Shodan, elle émettra probablement une réponse 300 ou 400.

 |
|

"wp-json"

 |

Cela recherchera des applications Web à l'aide de l'API WordPress.

 |

La machine à remonter le temps
==============================

La Wayback Machine est une archive de diverses pages Web au fil du temps. C'est idéal pour la reconnaissance passive de l'API, car cela vous permet de vérifier les modifications historiques apportées à votre cible. Si, par exemple, la cible a déjà annoncé une API partenaire sur sa page de destination, mais la cache maintenant derrière un portail authentifié, vous pourrez peut-être repérer ce changement à l'aide de la Wayback Machine. Un autre cas d'utilisation serait de voir les modifications apportées à la documentation de l'API existante. Si l'API n'a pas été bien gérée au fil du temps, il est possible que vous trouviez des points de terminaison retirés qui existent toujours, même si le fournisseur d'API pense qu'ils sont retirés. Celles-ci sont connues sous le nom d'API Zombie. Les API zombies relèvent de la vulnérabilité Improper Assets Management dans la liste OWASP API Security Top 10. 

 ![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/FGPF2fPT8ysfQHtTHQbk_Wayback1.PNG)

Vérifiez les différences entre la documentation de l'API. Plus tard, lorsque vous testerez activement l'API, assurez-vous de tester en utilisant d'anciens points de terminaison,

Conclusion
----------

Dans ce module, nous avons couvert certaines techniques de base qui peuvent être déployées pour recueillir des informations sur la surface d'attaque de l'API de votre cible. La première étape pour pirater les API d'une cible consiste à découvrir autant d'informations que possible à leur sujet. L'utilisation de ces techniques de reconnaissance peut non seulement vous aider à découvrir l'existence d'API, mais peut également conduire à des découvertes de vulnérabilités critiques. Ensuite, nous nous concentrerons sur les techniques de reconnaissance active.
