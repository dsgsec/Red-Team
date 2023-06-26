### Qu'est-ce qu'un cadre de commandement et de contrôle ?

Tout en essayant de digérer les différents composants d'un cadre C2 , cela peut être intimidant. Cependant, ils ne doivent pas l'être. Afin de mieux comprendre ce qu'est un framework C2 à son niveau le plus basique, pensez à un écouteur Netcat (le serveur C2) qui est capable de gérer plusieurs shells inversés rappelant à la fois (Agents C2). C'est un serveur mais pour les shells inversés. Contrairement à Netcat, presque tous les frameworks C2 nécessitent un générateur de charge utile spécial. Il s'agit généralement d'une fonctionnalité intégrée au framework lui-même. Par exemple, Metasploit est un framework C2 qui possède son propre générateur de charge utile, MSFVenom.

![661abd734022d8a1f3050e90d3fb6861](https://github.com/dsgsec/Red-Team/assets/82456829/94542913-c8b2-4eef-a7b3-7e49f9ed2ca9)

*Le diagramme ci-dessus illustre trois clients compromis rappelant un serveur C2 .\
*

Alors, qu'est-ce qui rend exactement les frameworks C2 meilleurs qu'un écouteur Netcat normal ? Il semble que tout ce que quelqu'un ait à faire est d'implémenter la gestion de session dans Netcat, et vous avez la même chose ? Bien que cela soit vrai, les frameworks C2 brillent dans leurs fonctionnalités "Post Exploitation".

### Structure de commandement et de contrôle

### Serveur C2

Afin de comprendre un cadre de commande et de contrôle, nous devons d'abord commencer par comprendre les différents composants d'un serveur C2 . Commençons par le composant le plus essentiel - Le serveur C2 lui-même . Le serveur C2 sert de plaque tournante vers laquelle les agents peuvent rappeler. Les agents contacteront périodiquement le serveur C2 et attendront les commandes de l'opérateur.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/5de5e62c5ba37013302790062b6b429e.png)

*Cette capture d'écran illustre un diagramme de serveur C2 de base .*

*\
*

### Agents / Charges utiles

Un agent est un programme généré par le framework C2 qui rappelle un écouteur sur un serveur C2. La plupart du temps, cet agent permet des fonctionnalités spéciales par rapport à un reverse shell standard. La plupart des frameworks C2 implémentent des pseudo-commandes pour faciliter la vie de l'opérateur C2. Quelques exemples de cela peuvent être une pseudo-commande pour télécharger ou télécharger un fichier sur le système. Il est important de savoir que les agents peuvent être hautement configurables, avec des ajustements sur la fréquence à laquelle les agents C2 balisent un écouteur sur un serveur C2 et bien plus encore.

### Les auditeurs

Au niveau le plus élémentaire, un écouteur est une application exécutée sur le serveur C2 qui attend un rappel sur un port ou un protocole spécifique. Quelques exemples de ceci sont DNS, HTTP et ou HTTPS.

### Balises

Une balise est le processus par lequel un agent C2 rappelle l'écouteur s'exécutant sur un serveur C2.

### Obscurcissement des rappels d'agent

### Minuteries de sommeil

L'un des éléments clés que certains analystes de la sécurité, antivirus et pare-feu de nouvelle génération recherchent lorsqu'ils tentent d'identifier le trafic de commande et de contrôle est le balisage et la vitesse à laquelle un périphérique se dirige vers un serveur C2 . Disons qu'un pare-feu a observé un trafic qui ressemble à ceci

-   TCP/443 - Durée de session 3s, 55 paquets envoyés, 10:00:05.000
-   TCP/443 - Durée de session 2s, 33 paquets envoyés, 10:00:10.000
-   TCP/443 - Durée de session 3s, 55 paquets envoyés, 10:00:15.000
-   TCP/443 - Durée de session 1s, 33 paquets envoyés, 10:00:20.000
-   TCP/443 - Durée de session 3s, 55 paquets envoyés, 10:00:25.000

Un modèle commence à se former. L'agent émet une balise toutes les 5 secondes ; cela signifie qu'il a une minuterie de sommeil de 5 secondes.

Gigue

Jitter prend la minuterie de mise en veille et y ajoute quelques variations ; notre balisage C2 peut maintenant présenter un schéma étrange qui peut montrer une activité plus proche d'un utilisateur moyen :

-   TCP/443 - Durée de session 3s, 55 paquets envoyés, 10:00:03.580
-   TCP/443 - Durée de session 2s, 33 paquets envoyés, 10:00:13.213
-   TCP/443 - Durée de session 3s, 55 paquets envoyés, 10:00:14.912
-   TCP/443 - Durée de session 1s, 33 paquets envoyés, 10:00:23.444
-   TCP/443 - Durée de session 3s, 55 paquets envoyés, 10:00:27.182

Le balisage est désormais défini sur un modèle semi-irrégulier qui le rend légèrement plus difficile à identifier parmi le trafic utilisateur régulier. Dans les frameworks C2 plus avancés, il peut être possible de modifier divers autres paramètres, comme la gigue "Fichier" ou l'ajout de données indésirables à la charge utile ou aux fichiers transmis pour le faire paraître plus grand qu'il ne l'est réellement.

Un exemple de code Python3 pour Jitter peut ressembler à ceci :

`import random`

`sleep = 60`

`jitter = random.randint(-30,30)`

`sleep = sleep + jitter\
`

Il est important de noter qu'il s'agit d'un exemple fondamental, mais il peut être beaucoup plus lourd en mathématiques, en définissant des limites supérieures et des limites inférieures, en prenant des pourcentages du dernier sommeil et en partant de là. Parce qu'il s'agit d'une salle d'introduction, nous vous épargnerons une formule compliquée.

### Types de charge utile

Tout comme un Reverse Shell classique, il existe deux principaux types de charges utiles que vous pouvez utiliser dans votre cadre C2 ; Charges utiles étagées et sans étape .

Charges utiles sans étape

Les charges utiles sans étape sont les plus simples des deux ; ils contiennent l' agent C2 complet et rappelleront le serveur C2 et commenceront le balisage immédiatement. Vous pouvez vous référer au schéma ci-dessous pour mieux comprendre le fonctionnement des charges utiles Stageless.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/e79d46d97f108842b9424ae5a134d2f8.png)

*Cette capture d'écran illustre une charge utile sans étape rappelant un serveur C2*

Les étapes pour établir le balisage C2 avec une charge utile Stageless sont les suivantes :

1.  La Victime télécharge et exécute le Dropper

2\.  Le balisage vers le serveur C2 commence

Charges utiles mises en scène

Les charges utiles mises en place nécessitent un rappel au serveur C2 pour télécharger des parties supplémentaires de l'agent C2. Ceci est communément appelé un "Dropper" car il est "déposé" sur la machine victime pour télécharger la deuxième étape de notre charge utile étagée. Il s'agit d'une méthode préférée par rapport aux charges utiles sans étape, car une petite quantité de code doit être écrite pour récupérer les parties supplémentaires de l'agent C2 à partir du serveur C2. Il facilite également l'obscurcissement du code pour contourner les programmes antivirus.\
![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/e6127ac6a295a1d9b01444757f711084.png)

*Ce diagramme illustre un dropper rappelant un serveur C2 pour sa deuxième étape.*

Les étapes pour établir le balisage C2 avec une charge utile Staged sont les suivantes :

1. La Victime télécharge et exécute le Dropper

2\. Le Dropper rappelle le serveur C2 pour l'étape 2

3\. Le serveur C2 renvoie l'étape 2 au poste de travail victime

4. L'étape 2 est chargée en mémoire sur le poste de travail de la victime 

5\. Le balisage C2 s'initialise et l'équipe rouge/les acteurs de la menace peuvent dialoguer avec la victime sur le serveur C2.

### Formats de charge utile

Comme vous le savez peut-être, les fichiers Windows PE (exécutables) ne sont pas le seul moyen d'exécuter du code sur un système. Certains frameworks C2 prennent en charge les charges utiles dans divers autres formats, par exemple :

-   Scripts PowerShell
    -   Qui peut contenir du code C# et peut être compilé et exécuté avec le commandlet Add-Type
-   Fichiers HTA
-   Fichiers JScript
-   Application/Scripts Visual Basic
-   Documents Microsoft Office

et beaucoup plus. Pour plus d'informations sur divers autres formats de charge utile, vous devriez consulter la salle [de militarisation](https://tryhackme.com/room/weaponization) dans le module d'accès initial.

### Modules

Les modules sont un élément central de tout cadre C2 ; ils ajoutent la possibilité de rendre les agents et le serveur C2 plus flexibles. Selon le framework C2, les scripts doivent être écrits dans différentes langues. Cobalt Strike a des "scripts d'agresseur", qui sont écrits dans le "langage de script d'agresseur". PowerShell Empire prend en charge plusieurs langues, les modules de Metasploit sont écrits en Ruby et bien d'autres sont écrits dans de nombreuses autres langues.

### Modules de post-exploitation

Les modules de post-exploitation sont simplement des modules qui traitent de tout après le point de compromis initial, cela peut être aussi simple que d'exécuter SharpHound.ps1 pour trouver des chemins de mouvement latéral, ou cela peut être aussi complexe que de vider LSASS et d'analyser les informations d'identification en mémoire. Pour plus d'informations sur la post-exploitation, reportez-vous à la salle [Post Exploitation Basics](https://tryhackme.com/room/postexploit) .

### Modules pivotants

L'un des derniers composants majeurs d'un cadre C2 est ses modules pivotants, qui facilitent l'accès aux segments de réseau restreints au sein du cadre C2. Si vous disposez d'un accès administrateur sur un système, vous pourrez peut-être ouvrir une "balise SMB", qui peut permettre à une machine d'agir en tant que proxy via le protocole SMB. Cela peut permettre aux machines d'un segment de réseau restreint de communiquer avec votre serveur C2.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/da7b0247ff1db8e98c9358c39a0c3d21.png)

*Ce diagramme illustre plusieurs victimes avec un pivot SMB rappelant un serveur C2.*

###

Le schéma ci-dessus montre comment les hôtes d'un segment de réseau restreint rappellent le serveur C2 :

1\.  Les victimes rappellent un canal nommé SMB sur une autre victime dans un segment de réseau non restreint.

2\.  La victime dans le segment de réseau non restreint rappelle le serveur C2 via une balise standard.

3\.  Le serveur C2 renvoie ensuite les commandes à la victime dans le segment de réseau non restreint.

4\.  La victime dans le segment de réseau non restreint transmet ensuite les instructions C2 aux hôtes du segment restreint.

### Face au monde 

Un obstacle important que tous les Red Teamers doivent surmonter est de placer l'infrastructure bien en vue. Il existe de nombreuses méthodes différentes pour le faire; l'une des méthodes les plus populaires s'appelle "Domain Fronting".

Façade de domaine

Domain Fronting utilise un bon hôte connu (par exemple) Cloudflare. Cloudflare gère une entreprise qui fournit des métriques améliorées sur les détails de connexion HTTP ainsi que la mise en cache des demandes de connexion HTTP pour économiser la bande passante. Les Red Teamers peuvent en abuser pour donner l'impression qu'un poste de travail ou un serveur communique avec une adresse IP connue et de confiance. Les résultats de géolocalisation s'afficheront là où se trouve le serveur Cloudflare le plus proche, et l'adresse IP apparaîtra comme propriété de Cloudflare.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/cd1ea19e9e0d7bef0d8ec6615061335b.png)

*Ce diagramme montre un exemple de balise HTTP provenant d'un appareil compromis.*

###

Le schéma ci-dessus illustre le fonctionnement de Domain Fronting :

1\.  L' opérateur C2 dispose d'un domaine qui transmet toutes les requêtes via Cloudflare.

2\.  La victime se dirige vers le domaine C2 .

3.  Cloudflare transmet la requête, puis examine l'en-tête de l'hôte et relaie le trafic vers le bon serveur.

4\. Le serveur C2 répond ensuite à Cloudflare avec les commandes C2.

5. La victime reçoit alors la commande de Cloudflare.

Profils C2

La technique suivante porte plusieurs noms par plusieurs produits différents, "NGINX Reverse Proxy", "Apache Mod_Proxy/Mod_Rewrite", "Malleable HTTP C2 Profiles", et bien d'autres. Cependant, ils sont tous plus ou moins les mêmes. Toutes les fonctionnalités du proxy permettent plus ou moins à un utilisateur de contrôler des éléments spécifiques de la requête HTTP entrante. Supposons qu'une demande de connexion entrante ait un en-tête "X-C2-Server" ; nous pourrions extraire explicitement cet en-tête en utilisant la technologie spécifique qui est à votre disposition (Reverse Proxy, Mod_Proxy/Rewrite, Malleable C2 Profile, etc.) et nous assurer que votre serveur C2 répond avec des réponses basées sur C2. Alors que si un utilisateur normal interrogeait le serveur HTTP, il pourrait voir une page Web générique. Tout dépend de votre configuration.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/22eac0e3ab2de3f61d57e858cee3e33e.png)

*Un analyste de l'appareil compromis et de la sécurité contacte un serveur C2 , seul l'appareil compromis récupère une balise C2 - l'analyste récupère le site Web de Cloudflare.*

Le schéma ci-dessus illustre le fonctionnement des profils C2 :

1\.  La victime envoie des balises au serveur C2 avec un en-tête personnalisé dans la requête HTTP, tandis qu'un analyste SOC a une requête HTTP normale.

2.  Les requêtes sont transmises via Cloudflare

3\.  Le serveur C2 reçoit la demande et recherche l'en-tête personnalisé, puis évalue comment répondre en fonction du profil C2.

4\. Le serveur C2 répond au client et répond à l'appareil Analyst/Compromised.

Comme les requêtes HTTPS sont cryptées, l'extraction d'en-têtes spécifiques (ex : X- C2 -Server ou Host) peut être impossible. En utilisant les profils C2, nous pourrons peut-être cacher notre serveur C2 aux regards indiscrets d'un analyste de sécurité. Pour plus d'informations sur la puissance des profils C2, consultez cet article de blog intitulé [Understanding Malleable C2 Profiles for Cobalt Strike](https://blog.zsec.uk/cobalt-strike-profiles/) .

Dans la tâche 7, nous expliquerons et explorerons une autre technique appelée "Redirecteurs". Nous acquerrons une expérience pratique de la configuration de Metasploit et d'Apache 2 pour montrer comment un redirecteur est configuré.
