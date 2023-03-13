Inclusion de fichiers locaux (LFI)
Maintenant que nous comprenons ce que sont les vulnérabilités d'inclusion de fichiers et comment elles se produisent, nous pouvons commencer à apprendre comment nous pouvons exploiter ces vulnérabilités dans différents scénarios pour pouvoir lire le contenu des fichiers locaux sur le serveur principal.

LFI de base
L'exercice que nous avons à la fin de cette section nous montre un exemple d'application Web qui permet aux utilisateurs de définir leur langue sur l'anglais ou l'espagnol :
   
http://<SERVER_IP> :<PORT>/


Si nous sélectionnons une langue en cliquant dessus (par exemple espagnol), nous voyons que le texte du contenu change en espagnol :
   
http://<SERVER_IP> :<PORT>/index.php?language=es.php


Nous remarquons également que l'URL inclut un paramètre de langue qui est maintenant défini sur la langue que nous avons sélectionnée (es.php). Le contenu peut être modifié de plusieurs manières pour correspondre à la langue que nous avons spécifiée. Il peut extraire le contenu d'une table de base de données différente en fonction du paramètre spécifié, ou il peut charger une version entièrement différente de l'application Web. Cependant, comme indiqué précédemment, le chargement d'une partie de la page à l'aide de moteurs de modèles est la méthode la plus simple et la plus couramment utilisée.

Ainsi, si l'application Web extrait effectivement un fichier qui est maintenant inclus dans la page, nous pourrons peut-être modifier le fichier extrait pour lire le contenu d'un fichier local différent. Deux fichiers lisibles courants disponibles sur la plupart des serveurs principaux sont /etc/passwd sous Linux et C:\Windows\boot.ini sous Windows. Alors, changeons le paramètre de es en /etc/passwd :
   
http://<SERVER_IP> :<PORT>/index.php?language=/etc/passwd


Comme nous pouvons le voir, la page est en effet vulnérable, et nous sommes en mesure de lire le contenu du fichier passwd et d'identifier quels utilisateurs existent sur le serveur principal.

Traversée de chemin
Dans l'exemple précédent, nous lisons un fichier en spécifiant son chemin absolu (par exemple /etc/passwd). Cela fonctionnerait si toute l'entrée était utilisée dans la fonction include() sans aucun ajout, comme dans l'exemple suivant :

Code : php
inclure($_GET['langue']);
Dans ce cas, si nous essayons de lire /etc/passwd, la fonction include() récupérera directement ce fichier. Cependant, dans de nombreuses occasions, les développeurs Web peuvent ajouter ou préfixer une chaîne au paramètre de langue. Par exemple, le paramètre de langue peut être utilisé pour le nom de fichier et peut être ajouté après un répertoire, comme suit :

Code : php
include("./langues/" . $_GET['langue']);
Dans ce cas, si nous essayons de lire /etc/passwd, alors le chemin passé à include() serait (./languages//etc/passwd), et comme ce fichier n'existe pas, nous ne pourrons pas lire quoi que ce soit:
   
http://<SERVER_IP> :<PORT>/index.php?language=/etc/passwd


Comme prévu, l'erreur détaillée renvoyée nous montre la chaîne passée à la fonction include(), indiquant qu'il n'y a pas de /etc/passwd dans le répertoire des langues.

Remarque : nous n'activons les erreurs PHP sur cette application Web qu'à des fins éducatives, afin que nous puissions bien comprendre comment l'application Web gère nos entrées. Pour les applications Web de production, de telles erreurs ne doivent jamais être affichées. De plus, toutes nos attaques devraient être possibles sans erreur, car elles ne s'appuient pas sur elles.

Nous pouvons facilement contourner cette restriction en parcourant les répertoires à l'aide de chemins relatifs. Pour ce faire, nous pouvons ajouter ../ avant notre nom de fichier, qui fait référence au répertoire parent. Par exemple, si le chemin complet du répertoire des langues est /var/www/html/langues/, alors l'utilisation de ../index.php ferait référence au fichier index.php du répertoire parent (c'est-à-dire /var/www/html /index.php).

Ainsi, nous pouvons utiliser cette astuce pour remonter plusieurs répertoires jusqu'à ce que nous atteignions le chemin racine (c'est-à-dire /), puis spécifier notre chemin de fichier absolu (par exemple ../../../../etc/passwd), et le fichier doit exister :
   
http://<SERVER_IP> :<PORT>/index.php?language=../../../../etc/passwd


Comme nous pouvons le voir, cette fois, nous avons pu lire le fichier quel que soit le répertoire dans lequel nous nous trouvions. Cette astuce fonctionnerait même si le paramètre entier était utilisé dans la fonction include(), nous pouvons donc utiliser cette technique par défaut, et elle devrait fonctionner dans les deux cas. De plus, si nous étions au chemin racine (/) et utilisions ../ alors nous resterions toujours dans le chemin racine. Donc, si nous n'étions pas sûrs du répertoire dans lequel se trouve l'application Web, nous pouvons ajouter ../ plusieurs fois, et cela ne devrait pas casser le chemin (même si nous le faisons une centaine de fois !).

Astuce : Il peut toujours être utile d'être efficace et de ne pas rajouter des ../ inutiles à plusieurs reprises, surtout si l'on rédigeait un rapport ou écrivait un exploit. Donc, essayez toujours de trouver le nombre minimum de ../ qui fonctionne et utilisez-le. Vous pouvez également calculer le nombre de répertoires éloignés du chemin racine et en utiliser autant. Par exemple, avec /var/www/html/ nous sommes à 3 répertoires du chemin racine, nous pouvons donc utiliser ../ 3 fois (c'est-à-dire ../../../).

Préfixe du nom de fichier
Dans notre exemple précédent, nous avons utilisé le paramètre de langue après le répertoire, afin que nous puissions parcourir le chemin pour lire le fichier passwd. À certaines occasions, notre entrée peut être ajoutée après une chaîne différente. Par exemple, il peut être utilisé avec un préfixe pour obtenir le nom complet du fichier,comme l'exemple suivant :

Code : php
include("lang_" . $_GET['langue']);
Dans ce cas, si nous essayons de parcourir le répertoire avec ../../../etc/passwd, la chaîne finale serait lang_../../../etc/passwd, qui n'est pas valide :
   
http://<SERVER_IP> :<PORT>/index.php?language=../../../etc/passwd


Comme prévu, l'erreur nous indique que ce fichier n'existe pas. ainsi, au lieu d'utiliser directement la traversée de chemin, nous pouvons préfixer un / avant notre charge utile, et cela devrait considérer le préfixe comme un répertoire, puis nous devrions ignorer le nom de fichier et pouvoir parcourir les répertoires :
   
http://<SERVER_IP> :<PORT>/index.php?language=/../../../etc/passwd


Remarque : Cela peut ne pas toujours fonctionner, car dans cet exemple, un répertoire nommé lang_/ peut ne pas exister, de sorte que notre chemin relatif peut ne pas être correct. De plus, tout préfixe ajouté à notre entrée peut casser certaines techniques d'inclusion de fichiers dont nous parlerons dans les sections à venir, comme l'utilisation de wrappers et de filtres PHP ou RFI.

Extensions ajoutées
Un autre exemple très courant est lorsqu'une extension est ajoutée au paramètre de langue, comme suit :

Code : php
inclure($_GET['langue'] . ".php");
C'est assez courant, car dans ce cas, nous n'aurions pas à écrire l'extension à chaque fois que nous devions changer de langue. Cela peut également être plus sûr car cela peut nous limiter à n'inclure que des fichiers PHP. Dans ce cas, si nous essayons de lire /etc/passwd, alors le fichier inclus serait /etc/passwd.php, qui n'existe pas :
   
http://<SERVER_IP> :<PORT>/extension/index.php?language=/etc/passwd


Il existe plusieurs techniques que nous pouvons utiliser pour contourner cela, et nous en discuterons dans les sections à venir.

Exercice : essayez de lire n'importe quel fichier php (par exemple index.php) via LFI, et voyez si vous obtiendriez son code source ou si le fichier est plutôt rendu au format HTML.

Attaques de second ordre
Comme nous pouvons le voir, les attaques LFI peuvent prendre différentes formes. Une autre attaque LFI courante et un peu plus avancée est une attaque de second ordre. Cela se produit car de nombreuses fonctionnalités d'application Web peuvent extraire de manière non sécurisée des fichiers du serveur principal en fonction de paramètres contrôlés par l'utilisateur.

Par exemple, une application Web peut nous permettre de télécharger notre avatar via une URL telle que (/profile/$username/avatar.png). Si nous créons un nom d'utilisateur LFI malveillant (par exemple ../../../etc/passwd), il peut être possible de changer le fichier extrait vers un autre fichier local sur le serveur et de le récupérer à la place de notre avatar.

Dans ce cas, nous empoisonnerions une entrée de base de données avec une charge utile LFI malveillante dans notre nom d'utilisateur. Ensuite, une autre fonctionnalité d'application Web utiliserait cette entrée empoisonnée pour effectuer notre attaque (c'est-à-dire télécharger notre avatar en fonction de la valeur du nom d'utilisateur). C'est pourquoi cette attaque est appelée attaque de second ordre.

Les développeurs négligent souvent ces vulnérabilités, car elles peuvent protéger contre la saisie directe de l'utilisateur (par exemple, à partir d'un paramètre ?page), mais ils peuvent faire confiance aux valeurs extraites de leur base de données, comme notre nom d'utilisateur dans ce cas. Si nous parvenions à empoisonner notre nom d'utilisateur lors de notre inscription, alors l'attaque serait possible.

L'exploitation des vulnérabilités LFI à l'aide d'attaques de second ordre est similaire à ce dont nous avons discuté dans cette section. La seule différence est que nous devons repérer une fonction qui extrait un fichier en fonction d'une valeur que nous contrôlons indirectement, puis essayer de contrôler cette valeur pour exploiter la vulnérabilité.