Reconnaissance active
=====================

##### [Reconnaissance API](https://university.apisec.ai/products/api-penetration-testing/categories/2150259092)

Reconnaissance API active
-------------------------

La reconnaissance active est le processus d'interaction directe avec la cible principalement grâce à l'utilisation du balayage. Nous utiliserons notre reconnaissance pour rechercher les API de notre cible et toute information utile.

Au cours de ce processus, vous analyserez les systèmes, énumérerez les ports ouverts et trouverez les ports qui ont des services utilisant HTTP. Une fois que vous avez trouvé des systèmes hébergeant HTTP, vous pouvez ouvrir un navigateur Web et examiner l'application Web. Vous pourriez trouver une API annoncée aux utilisateurs finaux ou vous devrez peut-être creuser plus profondément. Enfin, vous pouvez analyser l'application Web pour les répertoires liés à l'API. Essentiellement, vous allez construire la surface d'attaque de l'API de la cible. Pendant la reconnaissance active, nous utiliserons des outils tels que : nmap, OWASP Amass, gobuster, kiterunner et DevTools.

NmapName

Nmap est un outil puissant pour analyser les ports, rechercher les vulnérabilités, énumérer les services et découvrir les hôtes actifs. Pour la découverte d'API, vous devez exécuter deux scans Nmap en particulier : la détection générale et tous les ports. L'analyse de détection générale de Nmap utilise les scripts par défaut (-sC) et l'énumération de service (-sV) par rapport à une cible, puis enregistre la sortie dans trois formats pour un examen ultérieur ( -oX  pour XML, -oN pour Nmap, -oG pour greppable, ou -oA pour les trois):      

 `$ nmap -sC -sV  [adresse cible ou plage réseau]  -oA nomdesortie`

L'analyse de tous les ports Nmap vérifiera rapidement les 65 535 ports TCP pour les services en cours d'exécution, les versions d'application et le système d'exploitation hôte utilisé :

`$ nmap -p-  [adresse cible]  -oA allportscan`

Dès que l'analyse de détection générale commence à renvoyer des résultats, lancez l'analyse de tous les ports. Ensuite, commencez votre analyse pratique des résultats. Vous découvrirez très probablement les API en consultant les résultats liés au trafic HTTP et à d'autres indications de serveurs Web. Généralement, vous les trouverez exécutés sur les ports 80 et 443, mais une API peut être hébergée sur toutes sortes de ports différents. Une fois que vous avez découvert un serveur Web, vous pouvez effectuer une énumération HTTP à l'aide d'un script Nmap NSE (utilisez -p pour spécifier les ports que vous souhaitez tester).

`$ nmap -sV --script=http-enum <target> -p 80,443,8000,8080`

OWASP Amasser

OWASP Amass est un outil de ligne de commande qui peut cartographier le réseau externe d'une cible en collectant OSINT à partir de plus de 55 sources différentes. Vous pouvez le configurer pour effectuer des analyses passives ou actives. Si vous choisissez l'option active, Amass collectera des informations directement auprès de la cible en demandant ses informations de certificat. Sinon, il collecte des données à partir de moteurs de recherche (tels que Google, Bing et HackerOne), de sources de certificats SSL (telles que GoogleCT, Censys et FacebookCT), d'API de recherche (telles que Shodan, AlienVault, Cloudflare et GitHub) et du archives Web Wayback.

### Tirer le meilleur parti d'Amass avec les clés API

Avant de plonger dans l'utilisation d'Amass, nous devrions en tirer le meilleur parti en y ajoutant des clés API. Obtenons quelques clés API gratuites pour améliorer nos analyses Amass.

Tout d'abord, nous pouvons voir quelles sources de données sont disponibles pour Amass (payant et gratuit) en exécutant :\
`$ amass enum -list\`
![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/eex0psZsSx2Zyd9ekGIl_ActiveDiscovery1.PNG)

Ensuite, nous devrons créer un fichier de configuration pour y ajouter nos clés API.

`$ sudo curl https://raw.githubusercontent.com/OWASP/Amass/master/examples/config.ini >~/.config/amass/config.ini`

Nous pouvons maintenant mettre à jour le fichier config.ini. Je vais démontrer le processus d'ajout de clés API avec Censys. Visitez simplement  <https://censys.io/register>  et créez un compte gratuit. Assurez-vous d'utiliser un e-mail valide car vous devrez vérifier l'accès à votre compte gratuit.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/ZMWysMVxSPp9CREpENjU_ActiveDiscovery2.PNG)

Une fois que vous avez obtenu votre ID d'API et votre secret, modifiez le fichier config.ini et ajoutez les informations d'identification au fichier.

`$ sudo nano ~/.config/amass/config.ini`

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/XYQwwA4JTwWOF7CH0d3T_ActiveDiscovery3.PNG)

De plus, comme pour toutes les informations d'identification, assurez-vous de ne pas les partager comme je viens de le faire. Si vous les avez partagés, utilisez simplement le bouton "Réinitialiser mon secret API" sur Censys.io. Vous pouvez répéter ce processus avec de nombreux comptes gratuits et clés API, puis vous transformerez OWASP Amass en une centrale électrique pour la reconnaissance des API.

`$ amass enum -active -d target-name.com |grep api`

legacy-api.target-name.com

api1-backup.nom-cible.com

api3-backup.nom-cible.com

Cette analyse pourrait révéler de nombreux sous-domaines d'API uniques, notamment legacy-api.target-name.com . Un point de terminaison d'API nommé legacy pourrait présenter un intérêt particulier car il semble indiquer une vulnérabilité de gestion des actifs inappropriée.   

Amass a plusieurs options de ligne de commande utiles. Utilisez la commande Intel pour collecter des certificats SSL, rechercher des enregistrements Whois inversés et trouver les ID ASN associés à votre cible. Commencez par fournir la commande avec les adresses IP cibles  

`$ amass intel -addr  [adresses IP cibles] `

Si cette analyse réussit, elle vous fournira des noms de domaine. Ces domaines peuvent ensuite être transmis à Intel avec l' option whois pour effectuer une recherche Whois inversée :    

`$ amass intel -d  [domaine cible]  --whois `

Cela pourrait vous donner une tonne de résultats. Concentrez-vous sur les résultats intéressants qui se rapportent à votre organisation cible. Une fois que vous avez une liste de domaines intéressants, passez à la sous-commande enum pour commencer à énumérer les sous-domaines. Si vous spécifiez l' option - passive , Amass s'abstiendra d'interagir directement avec votre cible :    

`$ amass enum -passive -d  [domaine cible] `

L' analyse d'énumération active effectuera une grande partie de la même analyse que l'analyse passive, mais elle ajoutera une résolution de nom de domaine, tentera des transferts de zone DNS et récupérera les informations de certificat SSL :  

`$ amasser enum -active -d  [domaine cible] `

Pour améliorer votre jeu, ajoutez l' option -brute aux sous-domaines brute-force, -w pour spécifier la liste de mots API_superlist, puis l' option -dir pour envoyer la sortie dans le répertoire de votre choix :      

`$ amass enum -active -brute -w /usr/share/wordlists/API_superlist -d  [domaine cible]  -dir  [nom du répertoire] `  

Annuaire Brute-force avec Gobuster
----------------------------------

Gobuster peut être utilisé pour forcer brutalement les URI et les sous-domaines DNS à partir de la ligne de commande. (Si vous préférez une interface utilisateur graphique, consultez Dirbuster d'OWASP.) Dans Gobuster, vous pouvez utiliser des listes de mots pour les répertoires et sous-domaines communs pour demander automatiquement chaque élément de la liste de mots et les envoyer à un serveur Web et filtrer les réponses intéressantes du serveur. Les résultats générés par Gobuster vous fourniront le chemin de l'URL et les codes de réponse d'état HTTP. (Bien que vous puissiez forcer brutalement les URI avec Intruder de Burp Suite, Burp Community Edition est beaucoup plus lent que Gobuster.)

Chaque fois que vous utilisez un outil de force brute, vous devrez équilibrer la taille de la liste de mots et le temps nécessaire pour obtenir des résultats. Kali a des listes de mots de répertoire stockées sous /usr/share/wordlists/dirbuster qui sont complètes mais prendront un certain temps. Au lieu de cela, vous pouvez utiliser une liste de mots liée à l'API, ce qui accélérera vos analyses Gobuster car la liste de mots est relativement courte et ne contient que des répertoires liés aux API.  

L'exemple suivant utilise une liste de mots spécifique à l'API pour rechercher les répertoires sur une adresse IP :
```
$ gobuster dir -u nom-cible.com :8000 -w /home/hapihacker/api/wordlists/common_apis_160

================================================= ======

Gobuster

par OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)

================================================= ======

[+] URL : http://192.168.195.132:8000

[+] Méthode : GET

[+] Fils : 10

[+] Liste de mots : /home/hapihacker/api/wordlists/common_apis_160

[+] Codes d'état négatifs : 404

[+] Agent utilisateur : gobuster

[+] Délai d'attente : 10 s

================================================= ======

09:40:11 Démarrage de gobuster en mode d'énumération de répertoire

================================================= ======

/api (Statut : 200) [Taille : 253]

/admin (Statut : 500) [Taille : 1179]

/admins (Statut : 500) [Taille : 1179]

/login (Statut : 200) [Taille : 2833]

/register (Statut : 200) [Taille : 2846]

Une fois que vous avez trouvé des répertoires d'API comme le répertoire /api affiché dans cette sortie, soit par exploration, soit par force brute, vous pouvez utiliser Burp pour les approfondir. Gobuster a des options supplémentaires, et vous pouvez les lister en utilisant l' option -h :    

$ gobuster répertoire -h 

Si vous souhaitez ignorer certains codes d'état de réponse, utilisez l'option -b . Si vous souhaitez voir des codes d'état supplémentaires, utilisez -x . Vous pouvez améliorer une recherche Gobuster avec ce qui suit :  

$ gobuster répertoire -u 

://targetaddress/ -w /usr/share/wordlists/api_list/common_apis_160 -x 200,202,301 -b 302
```

Gobuster fournit un moyen rapide d'énumérer les URL actives pour trouver des chemins d'API.

Kiterunner
----------

Kiterunner est un excellent outil qui a été développé et publié par Assetnote. Kiterunner est actuellement le meilleur outil disponible pour découvrir les endpoints et les ressources de l'API. Alors que les outils de force brute d'annuaire comme Gobuster/Dirbuster/ fonctionnent pour découvrir les chemins d'URL, ils reposent généralement sur des requêtes HTTP GET standard. Kiterunner utilisera non seulement toutes les méthodes de requête HTTP communes aux API (GET, POST, PUT et DELETE), mais imitera également les structures de chemin d'API courantes. En d'autres termes, au lieu de demander GET /api/v1/user/create , Kiterunner essaiera POST /api/v1/user/create , imitant une requête plus réaliste.  

Vous pouvez effectuer une analyse rapide de l'URL ou de l'adresse IP de votre cible comme ceci :

`$ kr scan HTTP://127.0.0.1  -w ~/api/wordlists/data/kiterunner/routes-large.kite `

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/nXzTsPZ8R6C79ghxe2DO_ActiveDiscovery10.PNG)

 Comme vous pouvez le voir, Kiterunner vous fournira une liste de chemins intéressants. Le fait que le serveur réponde uniquement aux demandes de certains chemins /api/ indique que l'API existe.  

Notez que nous avons effectué cette analyse sans aucun en-tête d'autorisation, ce que l'API cible requiert probablement. Je montrerai comment utiliser Kiterunner avec les en-têtes d'autorisation au chapitre 7 . 

Si vous souhaitez utiliser une liste de mots texte plutôt qu'un fichier . kite  , utilisez l' option brute avec le fichier texte de votre choix :  

`$ kr brute  <target>  -w ~/api/wordlists/data/automated/nameofwordlist.txt `

Si vous avez plusieurs cibles, vous pouvez enregistrer une liste de cibles séparées par des lignes sous forme de fichier texte et utiliser ce fichier comme cible. Vous pouvez utiliser l'un des formats d'URI séparés par des lignes suivants en entrée :

Test.com

Test2.com:443

http://test3.com

http://test4.com

http://test5.com:8888/api

L'une des fonctionnalités les plus intéressantes de Kiterunner est la possibilité de rejouer les demandes. Ainsi, non seulement vous aurez un résultat intéressant à étudier, mais vous pourrez également disséquer exactement pourquoi cette demande est intéressante. Pour rejouer une requête, copiez toute la ligne de contenu dans Kiterunner, collez-la à l'aide de l' option de relecture ko et incluez la liste de mots que vous avez utilisée :  
```
$ kr kb replay "GET     414 [    183,    7,   8]

://192.168.50.35:8888/api/privatisations/count 0cf6841b1e7ac8badc6e237ab300a90ca873d571" -w

~/api/wordlists/data/kiterunner/routes-large.kite
```

L'exécution de ceci rejouera la demande et vous fournira la réponse HTTP. Vous pouvez ensuite examiner le contenu pour voir s'il y a quelque chose qui mérite une enquête. J'examine normalement les résultats intéressants, puis je les teste à l'aide de Postman et Burp Suite.

DevTools
--------

DevTools contient des outils de piratage d'applications Web très sous-estimés. Les étapes suivantes vous aideront à filtrer facilement et systématiquement des milliers de lignes de code afin de trouver des informations sensibles dans les sources de page. Commencez par ouvrir votre page cible, puis ouvrez DevTools avec F12 ou ctr - shift -I. Ajustez la fenêtre DevTools jusqu'à ce que vous disposiez de suffisamment d'espace pour travailler. Sélectionnez l'onglet Réseau puis actualisez la page (CTRL+r). 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/KAIntko1RBud1q6MVwBQ_ActiveDiscovery5.PNG)

Vous pouvez utiliser l'outil de filtrage pour rechercher n'importe quel terme de votre choix, tel que "API", "v1" ou "graphql". Il s'agit d'un moyen rapide de trouver les points de terminaison d'API en cours d'utilisation. Vous pouvez également laisser l'onglet Devtools Network ouvert pendant que vous effectuez des actions sur la page Web. Par exemple, voyons ce qui se passe si nous laissons les DevTools ouverts pendant que nous nous authentifions auprès de crAPI. Vous devriez voir apparaître une nouvelle demande. À ce stade, vous pouvez approfondir la demande en cliquant avec le bouton droit sur l'une des demandes et en sélectionnant "Modifier et renvoyer".

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/JHIANpvkSDONA1eM3T0S_ActiveDiscovery7.PNG)

Cela vous permettra de vérifier la demande dans le navigateur, de modifier les en-têtes/corps de la demande et de la renvoyer au fournisseur d'API. Bien qu'il s'agisse d'une excellente fonctionnalité de DevTools, vous souhaiterez peut-être passer à un navigateur destiné à interagir avec les API. Vous pouvez utiliser DevTools pour migrer des demandes individuelles vers Postman à l'aide de cURL.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/dKkYPAvmRTM4i4y825GQ_ActiveDiscovery7-5.PNG)

Une fois que vous avez copié la demande souhaitée, ouvrez Postman. Sélectionnez Importer et cliquez sur l'onglet "Texte brut". Collez la requête cURL et sélectionnez importer.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/NPybx1iQaeK63RN9bsTz_ActiveDiscovery8.PNG)

 Une fois la demande importée, elle contiendra tous les en-têtes nécessaires et le corps de la demande nécessaires pour effectuer des demandes supplémentaires dans Postman. C'est un excellent moyen d'interagir rapidement avec une API et d'interagir avec une seule requête API. Pour créer automatiquement une collection Postman plus complète, consultez le module suivant qui concerne l'ingénierie inverse d'une API.

La reconnaissance est extrêmement importante lors du test des API. La découverte des points de terminaison d'API est une première étape nécessaire lors de l'attaque d'API. Une bonne reconnaissance a également l'avantage supplémentaire de vous fournir potentiellement les clés du château sous la forme de clés API, de mots de passe, de jetons et d'autres divulgations d'informations utiles.
