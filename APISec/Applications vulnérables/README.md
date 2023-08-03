Tout au long du cours, nous allons parcourir deux applications vulnérables, crAPI et vAPI. Ces deux éléments seront utilisés pour tester les outils et les techniques qui seront démontrés tout au long de ce cours. APIsec.ai a hébergé un laboratoire de piratage d'API que vous pouvez utiliser pour mettre en pratique vos compétences.

crAPI se trouve sur crapi.apisec.ai

vAPI se trouve sur  vapi.apisec.ai

 Si vous souhaitez configurer votre propre laboratoire, vous pouvez soit héberger les applications vulnérables sur votre hôte local, soit sur un système distinct. Vient ensuite une démonstration de la configuration de ces applications sur votre hôte local.

L'API Complètement Ridicule (crAPI)
===================================

<https://github.com/OWASP/crAPI>

`$mkdir ~/lab`

`$cd ~/lab`

`#sudo curl -o docker-compose.yml https://raw.githubusercontent.com/OWASP/crAPI/main/deploy/docker/docker-compose.yml`

`$ sudo docker-compose pull`

`$ sudo docker-compose -f docker-compose.yml --compatibility up -d`

Si vous rencontrez des problèmes pour l'installer localement, vous pouvez essayer la version de développement décrite ici :  <https://github.com/OWASP/crAPI>  OU ciblez celle qui est hébergée par APIsec.

Une fois l'installation terminée, vous devriez pouvoir vérifier que crAPI est en cours d'exécution en utilisant un navigateur Web et en naviguant vers [http://127.0.0.1:8888](http://127.0.0.1:8888/)  (page de destination crAPI) ou [http://127.0.0.1:8025](http://127.0.0.1:8025/)   (serveur crAPI Mailhog). Lorsque vous avez fini d'utiliser/de tester crAPI, vous pouvez l'arrêter avec docker-compose en utilisant la commande suivante :\
$sudo docker-compose stop

`vAPI`
======

vAPI sera utilisé pour de nombreuses évaluations tout au long de ce cours. Bien qu'APIsec héberge vAPI, il peut être utile d'avoir une version locale pour les tests.

vAPI :  <https://github.com/roottusk/vapi> 

$cd ~/lab\
$sudo git clone <https://github.com/roottusk/vapi.git>\
 $cd /vapi\
$sudo docker-compose up -d

Une fois vAPI en cours d'exécution, vous pouvez accéder à http://127.0.0.1/ vapi pour accéder à la page d'accueil vAPI. Une chose importante à noter est que vAPI est livré avec une collection et un environnement Postman prédéfinis. Vous pouvez y accéder dans le dossier vAPI/postman.\
![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/H29aKnQQRDOJpNBHHGJv_postman1.png)

Vous pouvez les importer dans Postman pour vous préparer aux tests pour les évaluations futures. Ouvrez simplement Postman, sélectionnez le bouton Importer (en haut à droite) et sélectionnez les deux documents vAPI JSON (voir l'image ci-dessus). Enfin, confirmez l'importation et sélectionnez le bouton Importer.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/7hfeGAPTtuy1XsdNnUqg_postman2.png)

Une autre chose à noter à propos de vAPI est que le dossier Resources contient des secrets qui seront nécessaires pour mener à bien certaines attaques. Le dossier des ressources se trouve ici.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/H9dTPd6TRsWCNxIAQVDX_postman3.png)

De nombreux laboratoires sont disponibles pour tester les outils et les techniques que vous apprenez dans ce cours. Découvrez quelques-uns de ces autres laboratoires vulnérables :

Portswigger

-   [Académie de la sécurité Web](https://portswigger.net/web-security)

EssayezHackMe

-   [Librairie](https://tryhackme.com/room/bookstoreoc)  (gratuit)
-   [IDOR](https://tryhackme.com/room/idor) (payant)
-   [GraphQL](https://tryhackme.com/room/carpediem1) (payant)

[HackTheBox](https://www.hackthebox.com/hacker/hacking-labs) (machines à la retraite)

-   Artisanat
-   Facteur
-   JSON
-   Nœud
-   Aider

Github (applications vulnérables)

-   [Pixel](https://github.com/DevSlop/Pixi)
-   [Chèvre API REST](https://github.com/optiv/rest-api-goat)
-   [Nœud DVWS](https://github.com/snoopysecurity/dvws-node)
-   [Websheep](https://github.com/marmicode/websheep)

Vous tirerez le meilleur parti de ce cours en mettant la main sur le clavier et en piratant les API. Après avoir appris un nouvel outil ou une nouvelle technique, je vous recommande fortement d'appliquer vos compétences à ces autres laboratoires.
