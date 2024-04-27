Lecture de fichiers arbitraires via Path Traversal
---------------------------------------------------------

Imaginez une application d'achat qui affiche des images d'articles à vendre. Cela peut charger une image en utilisant le code HTML suivant:

`<img src="/loadImage?filename=218.png">`

Le `loadImage` URL prend un `filename` paramètre et retourne le contenu du fichier spécifié. Les fichiers d'image sont stockés sur le disque dans l'emplacement `/var/www/images/`. Pour retourner une image, l'application ajoute le nom de fichier demandé à ce répertoire de base et utilise une API de système de fichiers pour lire le contenu du fichier. En d'autres termes, l'application lit à partir du chemin de fichier suivant:

`/var/www/images/218.png`

Cette application n'implémente aucune défense contre les attaques de parcours de chemin. En conséquence, un attaquant peut demander l'URL suivante pour récupérer le `/etc/passwd` fichier du système de fichiers du serveur:

`https://insecure-website.com/loadImage?filename=../../../etc/passwd`

Cela provoque la lecture de l'application à partir du chemin de fichier suivant:

`/var/www/images/../../../etc/passwd`

La séquence `../` est valide dans un chemin de fichier, et signifie augmenter d'un niveau dans la structure du répertoire. Les trois consécutifs `../` les séquences s'élèvent de `/var/www/images/` à la racine du système de fichiers, et donc le fichier qui est réellement lu est:

`/etc/passwd`
