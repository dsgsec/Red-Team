Analyse des jetons
------------------

Lorsqu'ils sont implémentés correctement, les jetons peuvent être un excellent outil qui peut être utilisé pour authentifier et autoriser les utilisateurs. Cependant, si quelque chose ne va pas lors de la génération, du traitement ou de la manipulation des jetons, ils peuvent devenir nos clés du royaume.

Dans cette section, nous examinerons le processus qui peut être utilisé avec Burp Suite pour analyser les jetons. L'utilisation de ce processus peut vous aider à identifier les jetons prévisibles et faciliter les attaques de falsification de jetons. Pour analyser les jetons, nous prendrons notre demande d'authentification de l'API crAPI et la transmettrons à Burp Suite.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/nd3qXFW7QH6FbSKD9JXf_APIToken9.PNG)

Ensuite, vous devrez cliquer avec le bouton droit sur la demande et la transmettre à Sequencer. Dans Sequencer, nous pourrons faire en sorte que Burp Suite envoie des milliers de requêtes au fournisseur et effectue une analyse des jetons reçus en réponse. Cette analyse pourrait démontrer qu'un processus de création de jeton faible est utilisé.

Accédez à l'onglet Séquenceur et sélectionnez la demande que vous avez transmise. Ici, nous pouvons utiliser la capture en direct pour interagir avec la cible et récupérer des jetons en direct dans une réponse à analyser. Pour que ce processus fonctionne, vous devrez définir l'emplacement personnalisé du jeton dans la réponse. Sélectionnez le bouton Configurer à droite de Emplacement personnalisé. Mettez en surbrillance le jeton trouvé entre guillemets et cliquez sur OK.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/9Wong3kdSiUldvMBVsa8_APIToken10.PNG)

Une fois le jeton défini, vous pouvez démarrer la capture en direct. À ce stade, vous pouvez soit attendre que la capture traite des milliers de demandes, soit utiliser le bouton Analyser maintenant pour voir les résultats plus tôt.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/YyDwKWp3SaaNZ6Jk8G1i_APIToken11.PNG)

L'utilisation de Sequencer contre crAPI montre que les jetons générés semblent avoir suffisamment de caractère aléatoire et de complexité pour ne pas être prévisibles. Ce n'est pas parce que votre cible vous envoie un jeton apparemment complexe qu'elle est à l'abri de la falsification de jeton. Sequencer est excellent pour montrer que certains jetons complexes sont en fait très prévisibles. Si un fournisseur d'API génère des jetons de manière séquentielle, même si le jeton comportait plus de 20 caractères, il se peut que de nombreux caractères du jeton ne changent pas réellement. Faciliter la prédiction et la création de nos propres jetons valides.

Pour voir à quoi ressemble une analyse d'un processus de génération de jetons médiocre, effectuez une analyse des "mauvais jetons" situés sur le référentiel Github des API de piratage ( [https://raw.githubusercontent.com/hAPI-hacker/Hacking-APIs/main/ bad_tokens](https://raw.githubusercontent.com/hAPI-hacker/Hacking-APIs/main/bad_tokens) ). Cette fois-ci, nous utiliserons l'option de chargement manuel pour fournir notre propre ensemble de mauvais jetons. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/ITRLN0MQNegrXA6cn97r_APIToken12.PNG)

Si vous utilisez le bouton Analyser maintenant et laissez Sequencer exécuter son analyse. Consultez l'analyse au niveau des caractères qui révèle que le jeton à 12 caractères alphanumériques utilise les mêmes caractères pour les 8 premières positions. Les trois derniers caractères de ces jetons ont quelques variations. En prenant note des trois derniers caractères de ces jetons, vous pouvez remarquer que les possibilités consistent en deux lettres minuscules suivies d'un chiffre (aa#). Avec ces informations, vous pouvez forcer toutes les possibilités en moins de 7 000 requêtes. Ensuite, vous pouvez prendre ces jetons et envoyer des requêtes à un point de terminaison tel que */identity/api/v2/user/dashboard. *  En fonction des résultats de vos requêtes, recherchez parmi les noms d'utilisateur et les e-mails pour trouver les utilisateurs que vous souhaitez attaquer.

Attaques JWT
------------

Les jetons Web JSON (JWT) sont l'un des types de jetons d'API les plus répandus, car ils fonctionnent dans une grande variété de langages de programmation, notamment Python, Java, Node.js et Ruby. Ces jetons sont sensibles à toutes sortes d'erreurs de mauvaise configuration qui peuvent rendre les jetons vulnérables à plusieurs attaques supplémentaires. Ces attaques pourraient vous fournir des informations sensibles, vous accorder un accès non autorisé de base ou même un accès administratif à une API. Ce module vous guidera à travers quelques attaques que vous pouvez utiliser pour tester et casser les JWT mal implémentés. 

Les JWT se composent de trois parties, toutes encodées en base64 et séparées par des points : l'en-tête, la charge utile et la signature. JWT.io est un débogueur Web JWT gratuit que vous pouvez utiliser pour vérifier ces jetons. Vous pouvez repérer un JWT car il se compose de trois points et commence par "ey". Ils commencent par "ey" car c'est ce qui se passe lorsque vous encodez en base64 une accolade suivie d'un guillemet, ce qui est la façon dont un JWT décodé commence toujours. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/ECaM1KVCSFK59Lk84dh3_APIToken2.PNG)

Ici vous pouvez voir l'exemple JWT :

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6ImhhcGloYWNrZXIiLCJpYXQiOjE1MTYyMzkwMjJ9.U1Agy_OvIULTQwBrYx0dlWVzcBqcI90no 2pAcoy4-uo

Nous pouvons prendre ce JWT et décoder les parties individuelles. L'en-tête est le premier segment. Vous pouvez simplement faire écho à cette partie du jeton et la décoder en base64 pour voir ce que contient l'en-tête :

`$ echo eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9|base64 -d\
{"alg":"HS256","typ":"JWT"}`

L'algorithme (alg) utilisé pour ce jeton est HS256 et le type de jeton (typ) est JWT. Ensuite, nous pouvons faire la même chose avec la charge utile :

`$ echo eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6ImhhcGloYWNrZXIiLCJpYXQiOjE1MTYyMzkwMjJ9|base64 -d\
{"sub":"1234567890","name":"hapihacker","iat":1516239022}`

Désormais, ce jeton ne contient que le sujet (sub), le nom d'utilisateur (name) et l'heure à laquelle le jeton a été émis (iat). Plus d'informations pourraient être contenues dans le corps, en fait, tout ce que le fournisseur voudrait ajouter ici, il le peut. Un corps JWT peut être une excellente source de divulgation d'informations. Considérez un JWT qui contient le nom d'utilisateur, l'e-mail, le mot de passe et le rôle se cachant derrière un décodage base64. Si nous exécutons la même commande pour la signature JWT, nous verrons ce qui suit :

`$ echo U1Agy_OvIULTQwBrYx0dlWVzcBqcI90no2pAcoy4-uo|base64 -d`\
`SP base64: invalid input`

Enfin, la signature est la sortie de HMAC utilisée pour la validation du jeton et générée avec l'algorithme spécifié dans l'en-tête. Pour créer la signature, l'API base64 encode l'en-tête et la charge utile, puis applique l'algorithme de hachage et un secret. Le secret peut être sous la forme d'un mot de passe ou d'une chaîne secrète, telle qu'une clé de 256 bits. Sans connaissance du secret, la charge utile du JWT restera codée. 

`HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`

Si vous souhaitez en savoir plus sur les JWT, consultez  <https://jwt.io/introduction> . 

Si vous avez capturé le JWT d'un autre utilisateur ou peut-être découvert un jeton divulgué lors de la reconnaissance, vous pouvez essayer de l'envoyer au fournisseur et le faire passer pour le vôtre. Il est possible que le jeton divulgué n'ait pas encore expiré et puisse être fait passer pour le vôtre. Vous pouvez utiliser ce jeton dans les demandes d'accès à l'API en tant qu'utilisateur spécifié dans la charge utile. Plus généralement, cependant, vous obtiendrez un JWT en vous authentifiant auprès d'une API et le fournisseur répondra avec un JWT. Afin d'obtenir un JWT de crAPI, nous devrons tirer parti de notre demande d'authentification.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/KLm0YxhSJqLNA996u7NQ_APIToken1.PNG)

Le jeton que nous recevons de crAPI ne dit pas nécessairement "JWT : n'importe où, mais nous pouvons facilement repérer le "ey" et trois segments séparés par des points. La première étape pour attaquer un JWT est de le décoder et de l'analyser. Si nous prenons ce jeton et ajoutez-le au débogueur JWT c'est ce que nous voyons.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/GIea6TT5Q6qAwky9M56n_APIToken3.PNG)

Dans cet exemple, nous pouvons voir que l'algorithme est défini sur HS512, l'e-mail de notre compte, iat, exp, et la signature actuelle est invalide. Si nous pouvions compromettre le secret de la signature, nous devrions être en mesure de signer notre propre JWT et potentiellement accéder au compte de n'importe quel utilisateur valide. Ensuite, nous apprendrons à utiliser des outils automatisés pour nous aider avec diverses attaques JWT.

Automatisation des attaques JWT avec JWT_Tool
---------------------------------------------

 Le JSON Web Token Toolkit ou JWT_Tool est un excellent outil de ligne de commande que nous pouvons utiliser pour analyser et attaquer les JWT. Avec cela, nous serons en mesure d'analyser les JWT, de rechercher les faiblesses, de forger des jetons et des secrets de signature de force brute. Si vous avez suivi le module de configuration, vous devriez pouvoir utiliser l'alias jwt_tool pour voir les options d'utilisation :

$jwt_tool

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/qj5xhsy1REKeeXMFhe8R_APIToken6.PNG)

 Certaines des options à noter incluent :

-   -h pour afficher des options d'aide plus détaillées
-   -t pour spécifier l'URL cible
-   -M pour spécifier le mode de numérisation
    -   pb pour effectuer un audit du playbook (tests par défaut)
    -   à effectuer tous les tests
-   -rc pour ajouter des cookies de requête
-   -rh pour ajouter des en-têtes de requête
-   -rc pour ajouter des cookies de requête
-   -pd pour ajouter des données POST

Assurez-vous également de consulter le Wiki pour plus d'informations  [https://github.com/ticarpi/jwt_tool/wiki ](https://github.com/ticarpi/jwt_tool/wiki) .

Pour effectuer une analyse de base d'un JWT, utilisez simplement jwt_tool avec votre JWT capturé pour voir des informations similaires au débogueur JWT. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/dXvREvE5QPqQsuYj27Nu_APIToken4.PNG)

Comme vous pouvez le voir, jwt_tool rend les valeurs d'en-tête et de charge utile claires et nettes. De plus, jwt_tool dispose d'un "Playbook Scan" qui peut être utilisé pour cibler une application Web et rechercher les vulnérabilités JWT courantes. Vous pouvez exécuter cette analyse en utilisant les éléments suivants :

`$ jwt_tool -t http://target-name.com/ -rh "Authorization: Bearer JWT_Token" -M pb`

Dans le cas de crAPI, nous exécuterons :

`$jwt_tool -t http://127.0.0.1:8888/identity/api/v2/user/dashboard -rh "Authorization: Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWFhQGVtYWlsLmNvbSIsImlhdCI6MTY1ODUwNjQ0NiwiZXhwIjoxNjU4NTkyODQ2fQ.BLMqSjLZQ9P2cxcUP5UCAmFKVMjxhlB4uVeIu2__6zoJCJoFnDqTqKxGfrMcq1lMW97HxBVDnYNC7eC-pl0XYQ" -M pb`

 ![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/6kF9yBftQIKxVSbWRZff_APIToken5.PNG)

Au cours de cette analyse des erreurs de configuration courantes, JWT_Tool a testé les différentes revendications trouvées dans le JWT (sub, iat, exp)

### L'Aucune Attaque

Si jamais vous rencontrez un JWT utilisant "aucun" comme algorithme, vous avez trouvé une victoire facile. Après avoir décodé le jeton, vous devriez pouvoir voir clairement l'en-tête, la charge utile et la signature. À partir de là, vous pouvez modifier les informations contenues dans la charge utile pour qu'elles soient comme vous le souhaitez. Par exemple, vous pouvez remplacer le nom d'utilisateur par quelque chose qui est probablement utilisé par le compte administrateur du fournisseur (comme root, admin, administrator, test ou adm), comme indiqué ici : { "username": "root", "iat": 1516239022 } Une fois que vous avez modifié la charge utile, utilisez le décodeur de Burp Suite pour encoder la charge utile avec base64 ; puis insérez-le dans le JWT. Il est important de noter que puisque l'algorithme est défini sur "aucun", toute signature qui était présente peut être supprimée. En d'autres termes, vous pouvez tout supprimer après la troisième période dans le JWT.

### L'attaque par interrupteur d'algorithme

Il est possible que le fournisseur d'API ne vérifie pas correctement les JWT. Si tel est le cas, nous pourrons peut-être inciter un fournisseur à accepter un JWT avec un algorithme modifié. L'une des premières choses que vous devriez essayer est d'envoyer un JWT sans inclure la signature. Cela peut être fait en effaçant complètement la signature et en laissant le dernier point en place, comme ceci : VwZXJhZG1pbiI6dHJ1ZX0. Si cela ne réussit pas, essayez de modifier le champ d'en-tête de l'algorithme sur "aucun". Décodez le JWT, mettez à jour la valeur "alg" sur "none", encodez l'en-tête en base64 et envoyez-le au fournisseur. Pour simplifier ce processus, vous pouvez également utiliser jwt_tool pour créer rapidement un jeton avec l'algorithme commuté sur aucun.

`$ jwt_tool eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWFhQGVtYWlsLmNvbSIsImlhdCI6MTY1ODg1NTc0MCwiZXhwIjoxNjU4OTQyMTQwfQ._EcnSozcUnL5y9SFOgOVBMabx_UAr6Kg0Zym-LH_zyjReHrxU_ASrrR6OysLa6k7wpoBxN9vauhkYNHepOcrlA -X a\
`

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/sgoingGnRZ2Y7RwYu5xF_APIToken7.PNG)

En cas de succès, revenez à l'attaque Aucune. Cependant, si nous essayons cela avec crAPI, l'attaque ne réussit pas. Vous pouvez également utiliser JWT_Tool pour créer un jeton aucun.

Un scénario plus probable que le fournisseur n'acceptant aucun algorithme est qu'il accepte plusieurs algorithmes. Par exemple, si le fournisseur utilise RS256 mais ne limite pas les valeurs d'algorithme acceptables, nous pourrions modifier l'algorithme en HS256. Ceci est utile, car RS256 est un schéma de chiffrement asymétrique, ce qui signifie que nous avons besoin à la fois de la clé privée du fournisseur et d'une clé publique afin de hacher avec précision la signature JWT. Pendant ce temps, HS256 est un cryptage symétrique, donc une seule clé est utilisée à la fois pour la signature et la vérification du jeton. Si vous pouvez découvrir et obtenir la clé publique RS256 du fournisseur, puis basculer l'algorithme de RS256 vers HS256, il est possible que vous puissiez utiliser la clé publique RS256 comme clé HS256. Il utilise le format jwt_tool TOKEN -X k -pk public-key.pem, comme indiqué ci-dessous. Vous devrez enregistrer la clé publique capturée sous forme de fichier sur votre ordinateur attaquant (vous pouvez simuler cette attaque en prenant n'importe quelle clé publique et en l'enregistrant sous le nom public-key-pem).

$ jwt_tool eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJ1c2VyYWFhQGVtYWlsLmNvbSIsImlhdCI6MTY1ODg1NTc0MCwiZXhwIjoxNjU4OTQyMTQwfQ._EcnSozcUnL5y9SFOgOVBMabx_UAr6Kg0Zym- LH_zyjReHrxU_ASrrR6OysLa6k7wpoBxN9vauhkYNHepOcrlA -X k -pk clé-publique-pem

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/EjdckbmdRds9jtFtPQuV_APIToken8.PNG)

Une fois que vous avez exécuté la commande, JWT_Tool vous fournira un nouveau jeton à utiliser contre le fournisseur d'API. Si le fournisseur est vulnérable, vous pourrez détourner d'autres jetons, puisque vous avez maintenant la clé requise pour signer les jetons. Essayez de répéter le processus, cette fois en créant un nouveau jeton basé sur d'autres utilisateurs de l'API (autres utilisateurs découverts ou utilisateurs administrateurs génériques).

### Attaque de fissure JWT

L'attaque JWT Crack tente de déchiffrer le secret utilisé pour le hachage de signature JWT, nous donnant un contrôle total sur le processus de création de nos propres JWT valides. Les attaques de hachage comme celle-ci ont lieu hors ligne et n'interagissent pas avec le fournisseur. Par conséquent, nous n'avons pas à nous soucier de causer des ravages en envoyant des millions de requêtes à un fournisseur d'API. Vous pouvez utiliser JWT_Tool ou un outil comme Hashcat pour déchiffrer les secrets JWT. Vous alimenterez votre hash cracker avec une liste de mots. Le pirate de hachage hachera ensuite ces mots et comparera les valeurs à la signature hachée d'origine pour déterminer si l'un de ces mots a été utilisé comme secret de hachage. Si vous effectuez une attaque par force brute à long terme de chaque possibilité de personnage, vous pouvez utiliser les GPU dédiés qui alimentent Hashcat au lieu de JWT_Tool. Cela étant dit, Tout d'abord, utilisons Crunch, un outil de génération de mot de passe, pour créer une liste de toutes les combinaisons de caractères possibles à utiliser contre crAPI.

$crunch 5 5 -o crAPIpw.txt

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/i1ehRKgXTCmcjlbXPOgw_APIToken14.PNG)

Nous pouvons utiliser ce fichier de mots de passe qui contient toutes les combinaisons de caractères possibles créées pour les mots de passe à 5 caractères contre notre jeton crAPI.Pour effectuer une attaque JWT Crack à l'aide de JWT_Tool, utilisez la commande suivante : $ jwt_tool TOKEN -C -d /wordlist.txt L'option -C indique que vous allez mener une attaque par hachage, et l'option -d spécifie le dictionnaire ou la liste de mots que vous utiliserez contre le hachage. JWT_Tool renverra soit « CORRECT key ! » pour chaque valeur du dictionnaire ou indiquez une tentative infructueuse avec "clé introuvable dans le dictionnaire".

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/8UQdy9aeRQ23dlWZlTIz_APIToken15.PNG)

Maintenant que nous avons la bonne clé secrète utilisée pour signer les jetons de crAPI, nous devrions pouvoir générer nos propres jetons de confiance. Pour tester nos nouvelles capacités, vous pouvez soit créer un deuxième compte utilisateur dans crAPI, soit utiliser un e-mail que vous avez déjà découvert. J'ai créé un compte nommé "superadmin".

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/FODOQw7DTwC3kHmuja8Z_APIToken17.PNG)

Vous pouvez ajouter votre jeton d'utilisateur au débogueur JWT, ajouter le secret nouvellement découvert et mettre à jour la revendication "sub" à tout e-mail enregistré auprès de crAPI.

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/vpYghHEQ66YKkhLNVMJQ_APIToken18.PNG)

Utilisez le jeton que vous avez généré dans le débogueur et ajoutez-le à votre Postman Collection. Faites une demande à un point de terminaison qui prouvera votre accès tel que GET /identity/api/v2/user/dashboard. 

![](https://kajabi-storefronts-production.kajabi-cdn.com/kajabi-storefronts-production/site/2147573912/products/ANGMgdURImqiRbga6ZBt_APIToken18.PNG)

 Bravo!
