Inclusion de fichiers locaux (LFI)
==========================

Maintenant que nous comprenons ce que sont les vulnérabilités d'inclusion de fichiers et comment elles se produisent, nous pouvons commencer à apprendre comment nous pouvons exploiter ces vulnérabilités dans différents scénarios pour pouvoir lire le contenu des fichiers locaux sur le serveur principal.

* * * * *

LFI de base
---------

L'exercice que nous avons à la fin de cette section nous montre un exemple d'application Web qui permet aux utilisateurs de définir leur langue sur l'anglais ou l'espagnol :

![](https://academy.hackthebox.com/storage/modules/23/basic_lfi_lang.png)

Si nous sélectionnons une langue en cliquant dessus (par exemple `Espagnol`), nous constatons que le texte du contenu devient espagnol :

![](https://academy.hackthebox.com/storage/modules/23/basic_lfi_es.png)

Nous remarquons également que l'URL inclut un paramètre `language` qui est maintenant défini sur la langue que nous avons sélectionnée (`es.php`). Le contenu peut être modifié de plusieurs manières pour correspondre à la langue que nous avons spécifiée. Il peut extraire le contenu d'une table de base de données différente en fonction du paramètre spécifié, ou il peut charger une version entièrement différente de l'application Web. Cependant, comme indiqué précédemment, le chargement d'une partie de la page à l'aide de moteurs de modèles est la méthode la plus simple et la plus couramment utilisée.

Ainsi, si l'application Web extrait effectivement un fichier qui est maintenant inclus dans la page, nous pourrons peut-être modifier le fichier extrait pour lire le contenu d'un fichier local différent. Deux fichiers lisibles courants disponibles sur la plupart des serveurs principaux sont `/etc/passwd` sous Linux et `C:\Windows\boot.ini` sous Windows. Modifions donc le paramètre de `es` à `/etc/passwd` :

![](https://academy.hackthebox.com/storage/modules/23/basic_lfi_lang_passwd.png)

Comme nous pouvons le constater, la page est en effet vulnérable et nous sommes en mesure de lire le contenu du fichier `passwd` et d'identifier les utilisateurs existants sur le serveur principal.

* * * * *

Traversée de chemin
--------------

Dans l'exemple précédent, nous lisons un fichier en spécifiant son `chemin absolu` (par exemple `/etc/passwd`). Cela fonctionnerait si toute l'entrée était utilisée dans la fonction `include()` sans aucun ajout, comme dans l'exemple suivant :

Code : php

```
inclure($_GET['langue']);

```

Dans ce cas, si nous essayons de lire `/etc/passwd`, la fonction `include()` récupèrera ce fichier directement. Cependant, dans de nombreux cas, les développeurs Web peuvent ajouter ou précéder une chaîne au paramètre `language` . Par exemple, le paramètre `language` peut être utilisé pour le nom de fichier et peut être ajouté après un répertoire, comme suit :

Code : php

```
include("./langues/" . $_GET['langue']);

```

Dans ce cas, si nous essayons de lire `/etc/passwd`, alors le chemin passé à `include()` serait (`./languages//etc/passwd`), et comme ce fichier n'existe pas, nous ne pourra rien lire :

![](https://academy.hackthebox.com/storage/modules/23/traversal_passwd_failed.png)

Comme prévu, l'erreur détaillée renvoyée nous montre la chaîne transmise à la fonction `include()` , indiquant qu'il n'y a pas `/etc/passwd` dans le répertoire des langues.

Remarque : Nous n'activons les erreurs PHP sur cette application Web qu'à des fins éducatives, afin que nous puissions bien comprendre comment l'application Web gère nos entrées. Pour les applications Web de production, de telles erreurs ne doivent jamais être affichées. De plus, toutes nos attaques devraient être possibles sans erreur, car elles ne s'appuient pas sur elles.

Nous pouvons facilement contourner cette restriction en parcourant les répertoires à l'aide de "chemins relatifs". Pour ce faire, nous pouvons ajouter `../` avant notre nom de fichier, qui fait référence au répertoire parent. Par exemple, si le chemin complet du répertoire des langues est `/var/www/html/languages/`, l'utilisation de `../index.php` fera référence au fichier `index.php` du répertoire parent (c'est-à-dire `/var/www/html/index.php`).

Ainsi, nous pouvons utiliser cette astuce pour remonter plusieurs répertoires jusqu'à ce que nous atteignions le chemin racine (c'est-à-dire `/`), puis spécifier notre chemin de fichier absolu (par exemple `../../../../etc/passwd `), et le fichier doit exister :

![](https://academy.hackthebox.com/storage/modules/23/traversal_passwd.png)

Comme nous pouvons le voir, cette fois, nous avons pu lire le fichier quel que soit le répertoire dans lequel nous nous trouvions. Cette astuce fonctionnerait même si le paramètre entier était utilisé dans la fonction `include()` , nous pouvons donc utiliser cette technique par défaut, et ça devrait fonctionner dans les deux cas. De plus, si nous étions au chemin racine (`/`) et utilisions `../` alors nous resterions toujours dans le chemin racine. Donc, si nous n'étions pas sûrs du répertoire dans lequel se trouve l'application Web, nous pouvons ajouter `../` plusieurs fois, et cela ne devrait pas casser le chemin (même si nous le faisons une centaine de fois !).

Astuce : Il peut toujours être utile d'être efficace et de ne pas ajouter `../` plusieurs fois inutiles, surtout si nous écrivions un rapport ou écrivions un exploit. Donc, essayez toujours de trouver le nombre minimum de `../` qui fonctionne et utilisez-le. Vous pouvez également calculer le nombre de répertoires éloignés du chemin racine et en utiliser autant. Par exemple, avec `/var/www/html/` nous sommes à `3` répertoires du chemin racine, nous pouvons donc utiliser `../` 3 fois (c'est-à-dire `../../../`) .

* * ** *

Préfixe du nom de fichier
---------------

Dans notre exemple précédent, nous avons utilisé le paramètre `language` après le répertoire, afin de pouvoir parcourir le chemin pour lire le fichier `passwd` . À certaines occasions, notre entrée peut être ajoutée après une chaîne différente. Par exemple, il peut être utilisé avec un préfixe pour obtenir le nom de fichier complet, comme dans l'exemple suivant :

Code : php

```
include("lang_" . $_GET['langue']);

```

Dans ce cas, si nous essayons de parcourir le répertoire avec `../../../etc/passwd`, la chaîne finale serait `lang_../../../etc/passwd`, qui est invalide:

![](https://academy.hackthebox.com/storage/modules/23/lfi_another_example1.png)

Comme prévu, l'erreur nous indique que ce fichier n'existe pas. ainsi, au lieu d'utiliser directement la traversée de chemin, nous pouvons préfixer un `/` avant notre charge utile, et cela devrait considérer le préfixe comme un répertoire, puis nous devrions ignorer le nom de fichier et pouvoir parcourir les répertoires :

![](https://academy.hackthebox.com/storage/modules/23/lfi_another_example_passwd1.png)

Remarque : Cela peut ne pas toujours fonctionner, car dans cet exemple, un répertoire nommé `lang_/` peut ne pas exister, de sorte que notre chemin relatif peut ne pas être correct. De plus, `tout préfixe ajouté à notre entrée peut casser certaines techniques d'inclusion de fichiers` dont nous parlerons dans les sections à venir, comme l'utilisation de wrappers et de filtres PHP ou RFI.

* * * * *

Extensions ajoutées
-------------------

Un autre exemple très courant est lorsqu'une extension est ajoutée au paramètre `language` , comme suit :

Code : php

```
inclure($_GET['langue'] . ".php");

```

C'est assez courant, car dans ce cas, nous n'aurions pas à écrire l'extension à chaque fois que nous devions changer de langue. Cela peut également être plus sûr car cela peut nous limiter à n'inclure que des fichiers PHP. Dans ce cas, si nous essayons de lire `/etc/passwd`, alors le fichier inclus serait `/etc/passwd.php`, qui n'existe pas :

![](https://academy.hackthebox.com/storage/modules/23/lfi_extension_failed.png)
Journal d'empoisonnement
=============

Nous avons vu dans les sections précédentes que si nous incluons un fichier contenant du code PHP, il sera exécuté, tant que la fonction vulnérable dispose des privilèges `Execute` . Les attaques dont nous parlerons dans cette section reposent toutes sur le même concept : écrire du code PHP dans un champ que nous contrôlons et qui est enregistré dans un fichier journal (c'est-à-dire `poison`/`contaminate` le fichier journal), puis inclure ce fichier journal pour exécuter le code PHP. Pour que cette attaque fonctionne, l'application Web PHP doit avoir des privilèges de lecture sur les fichiers journalisés, qui varient d'un serveur à l'autre.

Comme c'était le cas dans la section précédente, toutes les fonctions suivantes dotées des privilèges `Exécuter` devraient être vulnérables à ces attaques :

| Fonction | Lire le contenu | Exécuter | URL distante |
| --- | :-: | :-: | :-: |
| PHP | | | |
| `include()`/`include_once()` | ✅ | ✅ | ✅ |
| `require()`/`require_once()` | ✅ | ✅ | ❌ |
| NodeJS | | | |
| `res.render()` | ✅ | ✅ | ❌ |
| Java | | | |
| `importer` | ✅ | ✅ | ✅ |
| .NET | | | |
| `inclure` | ✅ | ✅ | ✅ |

* * * * *

Empoisonnement de session PHP
---------------------

La plupart des applications Web PHP utilisent des cookies `PHPSESSID` , qui peuvent contenir des données spécifiques relatives à l'utilisateur sur le back-end, afin que l'application Web puisse suivre les détails de l'utilisateur via ses cookies. Ces détails sont stockés dans `session` fichiers sur le back-end, et enregistrés dans `/var/lib/php/sessions/` sous Linux et dans `C:\Windows\Temp\` sous Windows. Le nom du fichier qui contient les données de nos utilisateurs correspond au nom de notre cookie `PHPSESSID` avec le préfixe `sess_` . Par exemple, si le cookie `PHPSESSID` est défini sur `el4ukv0kqbvoirg7nkp4dncpk3`, son emplacement sur le disque sera `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`.

La première chose que nous devons faire dans une attaque d'empoisonnement de session PHP est d'examiner notre fichier de session PHPSESSID et de voir s'il contient des données que nous pouvons contrôler et empoisonner. Alors, commençons par vérifier si nous avons un cookie `PHPSESSID` défini pour notre session : ![image](https://academy.hackthebox.com/storage/modules/23/rfi_cookies_storage.png)

Comme nous pouvons le voir, la valeur de notre cookie `PHPSESSID` est `nhhv8i0o6ua4g88bkdl9u1fdsd`, elle doit donc être stockée dans `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`. Essayons d'inclure ce fichier de session via la vulnérabilité LFI et d'afficher son contenu :

![](https://academy.hackthebox.com/storage/modules/23/rfi_session_include.png)

Remarque : Comme vous pouvez facilement le deviner, la valeur du cookie diffère d'une session à l'autre. Vous devez donc utiliser la valeur du cookie que vous trouvez dans votre propre session pour effectuer la même attaque.

Nous pouvons voir que le fichier de session contient deux valeurs : `page`, qui affiche la page de langue sélectionnée, et `preference`, qui affiche la langue sélectionnée. La valeur `preference` n'est pas sous notre contrôle, car nous ne l'avons spécifiée nulle part et doit être spécifiée automatiquement. Cependant, la valeur `page` est sous notre contrôle, car nous pouvons la contrôler via le paramètre `?language=` .

Essayons de définir la valeur de `page` une valeur personnalisée (par exemple `paramètre de langue`) et voyons si cela change dans le fichier de session. Nous pouvons fairedonc en visitant simplement la page avec `?language=session_poisoning` spécifié, comme suit :

Code : URL

```
http://<SERVER_IP> :<PORT>/index.php?language=session_poisoning

```

Maintenant, incluons à nouveau le fichier de session pour en examiner le contenu :

![](https://academy.hackthebox.com/storage/modules/23/lfi_poisoned_sessid.png)

Cette fois, le fichier de session contient `session_poisoning` au lieu de `es.php`, ce qui confirme notre capacité à contrôler la valeur de `page` dans le fichier de session. Notre prochaine étape consiste à effectuer l'étape d'"empoisonnement" en écrivant du code PHP dans le fichier de session. Nous pouvons écrire un shell Web PHP de base en remplaçant le paramètre `?language=` par un shell Web codé en URL, comme suit :

Code : URL

```
http://<SERVER_IP> :<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E

```

Enfin, nous pouvons inclure le fichier de session et utiliser `&cmd=id` pour exécuter une commande :

![](https://academy.hackthebox.com/storage/modules/23/rfi_session_id.png)

Remarque : Pour exécuter une autre commande, le fichier de session doit à nouveau être empoisonné avec le shell Web, car il est remplacé par `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` après notre dernière inclusion. Idéalement, nous utiliserions le shell Web empoisonné pour écrire un shell Web permanent dans le répertoire Web, ou enverrions un shell inversé pour une interaction plus facile.

* * * * *

Empoisonnement du journal du serveur
--------------------

 `Apache` et `Nginx` conservent divers fichiers journaux, tels que `access.log` et `error.log`. Le fichier `access.log` contient diverses informations sur toutes les requêtes adressées au serveur, y compris l'en-tête `User-Agent` de chaque requête. Comme nous pouvons contrôler l'en-tête `User-Agent` dans nos requêtes, nous pouvons l'utiliser pour empoisonner les journaux du serveur comme nous l'avons fait ci-dessus.

Une fois empoisonné, nous devons inclure les journaux via la vulnérabilité LFI, et pour cela, nous devons avoir un accès en lecture sur les journaux. Les journaux `Nginx` sont lisibles par défaut par les utilisateurs à faibles privilèges (par exemple `www-data`), tandis que les journaux `Apache` ne sont lisibles que par les utilisateurs disposant de privilèges élevés (par exemple `root`/`adm` groupes). Cependant, sur les serveurs `Apache` anciens ou mal configurés, ces journaux peuvent être lisibles par les utilisateurs à faibles privilèges.

Par défaut, les journaux `Apache` sont situés dans `/var/log/apache2/` sous Linux et dans `C:\xampp\apache\logs\` sous Windows, tandis que les journaux `Nginx` sont situés dans `/var/log /nginx/` sous Linux et dans `C:\nginx\log\` sous Windows. Cependant, les journaux peuvent se trouver à un emplacement différent dans certains cas, nous pouvons donc utiliser une [liste de mots LFI](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) pour rechercher leurs emplacements, comme nous le verrons dans la section suivante.

Essayons donc d'inclure le journal d'accès Apache à partir de `/var/log/apache2/access.log`, et voyons ce que nous obtenons :

![](https://academy.hackthebox.com/storage/modules/23/rfi_access_log.png)

Comme nous pouvons le voir, nous pouvons lire le journal. Le journal contient l' `adresse IP distante`, la `page de demande`, le `code de réponse` et l'en-tête `User-Agent` . Comme mentionné précédemment, l'en-tête `User-Agent` est contrôlé par nous via les en-têtes de requête HTTP, nous devrions donc pouvoir empoisonner cette valeur.

Conseil : les journaux ont tendance à être volumineux, et leur chargement dans une vulnérabilité LFI peut prendre un certain temps à se charger, voire planter le serveur dans le pire des cas. Soyez donc prudent et efficace avec eux dans un environnement de production et n'envoyez pas de demandes inutiles.

Pour ce faire, nous utiliserons `Burp Suite` pour intercepter notre requête LFI précédente et modifier l'en-tête `User-Agent` en `Apache Log Poisoning` : ![image](https://academy.hackthebox.com/storage/ modules/23/rfi_repeater_ua.png)

Remarque : Comme toutes les requêtes adressées au serveur sont enregistrées, nous pouvons empoisonner toute requête adressée à l'application Web, et pas nécessairement celle du LFI comme nous l'avons fait ci-dessus.

Comme prévu, notre valeur User-Agent personnalisée est visible dans le fichier journal inclus. Maintenant, nous pouvons empoisonner l'en-tête `User-Agent` en le définissant sur un shell Web PHP de base : ![image](https://academy.hackthebox.com/storage/modules/23/rfi_cmd_repeater.png)

Nous pouvons également empoisonner le journal en envoyant une requête via cURL, comme suit :

```
dsgsec@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php" -A '<?php system($_GET["cmd"]); ?>'

```

Comme le journal devrait maintenant contenir du code PHP, la vulnérabilité LFI devrait exécuter ce code et nous devrions pouvoir obtenir l'exécution de code à distance. Nous pouvons spécifier une commande à exécuter avec (`?cmd=id`) : ![image](https://academy.hackthebox.com/storage/modules/23/rfi_id_repeater.png)

Nous voyons que nous avons exécuté avec succès la commande. La même attaque peut également être effectuée sur les journaux `Nginx`.

Conseil : L'en-tête `User-Agent` s'affiche également sur les fichiers de processus sous le répertoire Linux `/proc/` . Nous pouvons donc essayer d'inclure les fichiers `/proc/self/environ` ou `/proc/self/fd/N` (où N est un PID généralement compris entre 0 et 50), et nous pourrons peut-être effectuer la même attaque. sur ces fichiers. Cela peut devenir pratique au cas où nous n'aurions pas accès en lecture aux journaux du serveur, cependant, ces fichiers ne peuvent également être lus que par des utilisateurs privilégiés.

Enfin, il existe d'autres techniques similaires d'empoisonnement des journaux que nous pouvons utiliser sur varijournaux système, selon les journaux auxquels nous avons accès en lecture. Voici quelques-uns des journaux de service que nous pouvons lire :

- `/var/log/sshd.log`
- `/var/log/mail`
- `/var/log/vsftpd.log`

Nous devrions d'abord essayer de lire ces journaux via LFI, et si nous y avons accès, nous pouvons essayer de les empoisonner comme nous l'avons fait ci-dessus. Par exemple, si les services `ssh` ou `ftp` nous sont exposés et que nous pouvons lire leurs journaux via LFI, nous pouvons essayer de nous y connecter et de définir le nom d'utilisateur sur le code PHP, et après avoir inclus leurs journaux, le PHP le code s'exécuterait. Il en va de même pour les services `mail` , car nous pouvons envoyer un e-mail contenant du code PHP, et lors de son inclusion dans le journal, le code PHP s'exécutera. Nous pouvons généraliser cette technique à tous les journaux qui enregistrent un paramètre que nous contrôlons et que nous pouvons lire à travers la vulnérabilité LFI.
Il existe plusieurs techniques que nous pouvons utiliser pour contourner cela, et nous en discuterons dans les sections à venir.

Exercice : Essayez de lire n'importe quel fichier php (par exemple, index.php) via LFI, et voyez si vous obtiendriez son code source ou si le fichier est rendu au format HTML à la place.

* * * * *

Attaques de second ordre
--------------------

Comme nous pouvons le voir, les attaques LFI peuvent prendre différentes formes. Une autre attaque LFI courante et un peu plus avancée est une "attaque de second ordre". Cela se produit car de nombreuses fonctionnalités d'application Web peuvent extraire de manière non sécurisée des fichiers du serveur principal en fonction de paramètres contrôlés par l'utilisateur.

Par exemple, une application Web peut nous permettre de télécharger notre avatar via une URL telle que (`/profile/$username/avatar.png`). Si nous créons un nom d'utilisateur LFI malveillant (par exemple `../../../etc/passwd`), il peut être possible de changer le fichier extrait vers un autre fichier local sur le serveur et de le récupérer à la place de notre avatar .

Dans ce cas, nous empoisonnerions une entrée de base de données avec une charge utile LFI malveillante dans notre nom d'utilisateur. Ensuite, une autre fonctionnalité d'application Web utiliserait cette entrée empoisonnée pour effectuer notre attaque (c'est-à-dire télécharger notre avatar en fonction de la valeur du nom d'utilisateur). C'est pourquoi cette attaque est appelée une attaque de "second ordre".

Les développeurs négligent souvent ces vulnérabilités, car elles peuvent protéger contre la saisie directe de l'utilisateur (par exemple, à partir d'un paramètre `?page` ), mais ils peuvent faire confiance aux valeurs extraites de leur base de données, comme notre nom d'utilisateur dans ce cas. Si nous parvenions à empoisonner notre nom d'utilisateur lors de notre inscription, alors l'attaque serait possible.

L'exploitation des vulnérabilités LFI à l'aide d'attaques de second ordre est similaire à ce dont nous avons discuté dans cette section. La seule différence est que nous devons repérer une fonction qui extrait un fichier en fonction d'une valeur que nous contrôlons indirectement, puis essayer de contrôler cette valeur pour exploiter la vulnérabilité.