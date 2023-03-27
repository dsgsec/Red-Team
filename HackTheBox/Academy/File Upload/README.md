Intro pour les attaques de téléchargement de fichiers
===========================.

* * * * *

Le téléchargement des fichiers utilisateur est devenu une caractéristique clé pour la plupart des applications Web modernes afin d'autoriser l'extensibilité des applications Web avec des informations utilisateur. Un site Web de médias sociaux permet le téléchargement d'images de profil utilisateur et d'autres médias sociaux, tandis qu'un site Web d'entreprise peut permettre aux utilisateurs de télécharger des PDF et d'autres documents à usage d'entreprise.

Cependant, comme les développeurs d'applications Web permettent cette fonctionnalité, ils prennent également le risque d'autoriser les utilisateurs finaux de stocker leurs données potentiellement malveillantes sur le serveur back-end de l'application Web. Si l'entrée de l'utilisateur et les fichiers téléchargés ne sont pas correctement filtrés et validés, les attaquants peuvent être en mesure d'exploiter la fonction de téléchargement de fichiers pour effectuer des activités malveillantes, comme exécuter des commandes arbitraires sur le serveur arrière pour le contrôler.

Les vulnérabilités de téléchargement de fichiers sont parmi les vulnérabilités les plus courantes trouvées dans les applications Web et mobiles, comme nous pouvons le voir dans les derniers [rapports CVE] (https://www.cvedetails.com/vulnerabilité-list/cweid-434/vulnerabilities.html) . Nous remarquerons également que la plupart de ces vulnérabilités sont notées sous forme de vulnérabilités «élevées» ou «critiques», montrant le niveau de risque causé par le téléchargement de fichiers non sécurisé.

* * * * *

Types d'attaques de téléchargement de fichiers
----------------------------

La raison la plus courante derrière les vulnérabilités de téléchargement de fichiers est la validation et la vérification des fichiers faibles, qui peuvent ne pas être bien sécurisées pour empêcher les types de fichiers indésirables ou pourraient manquer complètement. Le pire type de vulnérabilité de téléchargement de fichiers possible est une vulnérabilité de fichiers arbitraires non authentifiée ». Avec ce type de vulnérabilité, une application Web permet à tout utilisateur non authentifié de télécharger n'importe quel type de fichier, ce qui en fait une étape de permettre à tout utilisateur d'exécuter du code sur le serveur arrière.

De nombreux développeurs Web utilisent divers types de tests pour valider l'extension ou le contenu du fichier téléchargé. Cependant, comme nous le verrons dans ce module, si ces filtres ne sont pas sécurisés, nous pourrons peut-être les contourner et atteindre des téléchargements de fichiers arbitraires pour effectuer nos attaques.

L'attaque la plus courante et la plus critique provoquée par des téléchargements de fichiers arbitraires consiste à «obtenir une exécution de commande distante» sur le serveur arrière en téléchargeant un shell Web ou en téléchargeant un script qui envoie un shell inversé. Un shell Web, comme nous le discuterons dans la section suivante, nous permet d'exécuter n'importe quelle commande que nous spécifions et peut être transformée en shell interactif pour énumérer facilement le système et exploiter davantage le réseau. Il peut également être possible de télécharger un script qui envoie un shell inversé à un écouteur sur notre machine, puis d'interagir avec le serveur distant de cette façon.

Dans certains cas, nous n'avons peut-être pas de téléchargements de fichiers arbitraires et ne pourrions pas télécharger un type de fichier spécifique. Même dans ces cas, il existe diverses attaques que nous pouvons être en mesure d'effectuer pour exploiter la fonctionnalité de téléchargement de fichiers si certaines protections de sécurité manquaient dans l'application Web.

Des exemples de ces attaques comprennent:

- Présentation d'autres vulnérabilités comme «XSS» ou «XXE».
- provoquant un «déni de service (DOS)» sur le serveur back-end.
- Écraser les fichiers et configurations système critiques.
- Et plein d'autres.

Enfin, une vulnérabilité de téléchargement de fichiers est non seulement causée par l'écriture de fonctions non sécurisées, mais est également souvent causée par l'utilisation de bibliothèques obsolètes qui peuvent être vulnérables à ces attaques. À la fin du module, nous allons passer par divers conseils et pratiques pour sécuriser nos applications Web par rapport aux types d'attaques de téléchargement de fichiers les plus courants, en plus de nouvelles recommandations pour empêcher les vulnérabilités de téléchargement de fichiers que nous pouvons manquer.