Biscuits de forçage brutal
==========================

* * * * *

Lorsque le protocole HTTP est né, il n'était pas nécessaire de suivre les états de connexion. Un utilisateur a demandé une ressource, le serveur a répondu et la connexion a été fermée. C'était en 1991 et les sites Web étaient assez différents de ce à quoi nous sommes habitués aujourd'hui. Comme vous pouvez l'imaginer, presque aucune application Web moderne ne pourrait fonctionner de cette façon, car elles proposent un contenu différent en fonction de qui le demande. Un panier, des préférences, des messages, etc. sont de bons exemples de contenu personnalisé. Heureusement, alors qu'il développait la première application de commerce électronique, le précurseur du WWW a écrit une nouvelle norme, `cookies`.

À l'époque, un cookie, envoyé en en-tête à chaque requête du navigateur vers l'application Web, était utilisé pour conserver tous les détails de la session de l'utilisateur, tels que son panier, y compris les produits choisis, la quantité et le prix. Des problèmes sont apparus très rapidement. La principale préoccupation aujourd'hui est la sécurité. On sait qu'on ne peut pas faire confiance au client lorsqu'il s'agit d'autoriser une modification ou une vue, mais à l'époque, le problème était aussi celui de la taille de la requête. Les cookies ont alors commencé à être plus légers et à être définis comme un identifiant unique qui fait référence à un `session`stocké côté serveur. Lorsqu'un visiteur affiche ses cookies, l'application vérifie tous les détails en regardant la session correcte côté serveur.

Bien que nous sachions qu'une application Web peut définir de nombreux cookies, nous savons également qu'ils `one or two`sont généralement pertinents pour la session. Les cookies liés à la session sont utilisés pour "discriminer" les utilisateurs les uns des autres. D'autres cookies peuvent être liés à la langue, aux informations sur l'acceptation de la confidentialité ou à une clause de non-responsabilité relative aux cookies, entre autres. Ces cookies pourraient être altérés sans impact significatif puisqu'ils ne sont pas liés à la sécurité de l'application.

Nous savons qu'il existe également d'autres moyens de suivre les utilisateurs, par exemple, le jeton déjà discuté `HTTP Authentication`ou un jeton dans la page comme `ViewState`. `HTTP Authentication`n'est pas courant sur les applications Internet, du moins pas en tant que couche principale d'authentification. Cependant, cela pourrait être la barrière de sécurité appropriée si nous voulons protéger une application Web si `before`un utilisateur atteint même le formulaire de connexion.

[ViewState](https://www.w3big.com/aspnet/aspnet-viewstate.html) est un excellent exemple de jeton de sécurité dans la page, utilisé par défaut par `.NET`les applications Web comme n'importe quel site SharePoint. `ViewState`est inclus en tant que champ masqué dans les formulaires HTML. Il est construit comme un objet sérialisé contenant des informations utiles sur l'utilisateur/la session en cours (d'où vient l'utilisateur, où l'utilisateur peut aller, ce que l'utilisateur peut voir ou modifier, etc.) A pourrait être facilement décodé s'il n'est pas `ViewState token`chiffré . Cependant, la principale préoccupation est qu'il pourrait souffrir d'une vulnérabilité qui conduit à l'exécution de code à distance même s'il est chiffré. Les cookies de session peuvent souffrir des mêmes vulnérabilités qui peuvent affecter les jetons de réinitialisation de mot de passe. Ils peuvent être prévisibles, cassés ou falsifiés.

* * * * *

Falsification du jeton de cookie
--------------------------------

Comme les jetons de réinitialisation de mot de passe, les jetons de session peuvent également être basés sur des informations devinables. Souvent, les applications Web maison proposent une gestion de session personnalisée et des mécanismes de création de cookies personnalisés pour avoir à portée de main les détails liés à l'utilisateur. Les données les plus courantes que nous pouvons trouver dans les cookies sont les autorisations d'utilisateur. Qu'un utilisateur soit un administrateur, un opérateur ou un utilisateur de base, il s'agit d'informations qui peuvent faire partie des données utilisées pour créer le cookie.

Malheureusement, comme déjà discuté, il n'est pas rare de voir des jetons générés à partir de valeurs importantes, telles que l'ID utilisateur, les autorisations, l'heure, etc. Imaginez un scénario dans lequel une partie de la fonctionnalité d'une application Web consiste à récupérer le texte en clair à partir d'une valeur codée/cryptée. . Cela signifie qu'un cryptage unidirectionnel tel que le hachage est hors de question. Bien sûr, il n'y a aucune raison réelle de stocker des données dans des cookies de session car tout ce qui fait partie de la session doit être géré et validé côté serveur, et leur contenu doit être complètement aléatoire.

Considérons l'exemple suivant.

![](https://academy.hackthebox.com/storage/modules/80/11-cookielogin.png)

La ligne 4 de la réponse du serveur affiche un `Set-Cookie`en-tête qui définit a `SESSIONID`sur `757365723A6874623B726F6C653A75736572`. Si nous décodons cette valeur hexadécimale en ASCII, nous voyons qu'elle contient notre `userid`, `htb`et notre rôle (standard `user`).

```
dsgsec@htb[/htb]$ echo -n 757365723A6874623B726F6C653A75736572 | xxd -r -p; echo

user:htb;role:user

```

Nous pourrions essayer d'élever nos privilèges au sein de l'application en modifiant le `SESSIONID`cookie pour qu'il contienne `role:admin`. Cette action peut même nous permettre de contourner complètement l'authentification.

* * * * *

Jeton Se souvenir de moi
------------------------

Nous pourrions considérer un `rememberme`jeton comme un cookie de session qui dure plus longtemps que d'habitude. `rememberme`les jetons durent généralement au moins sept jours, voire un mois entier. Compte tenu de leur longue durée de vie, `rememberme`les jetons pourraient être plus faciles à forcer. Si l'algorithme utilisé pour générer un `rememberme`jeton ou sa longueur n'est pas suffisamment sécurisé, un attaquant pourrait tirer parti du délai prolongé pour le deviner. Presque toutes les attaques contre les jetons de réinitialisation de mot de passe et les cookies génériques peuvent également être montées contre `rememberme`des jetons, et les mesures de sécurité qu'un développeur doit mettre en place sont presque les mêmes.

* * * * *

Jeton crypté ou codé
--------------------

Les cookies peuvent également contenir le résultat du cryptage d'une séquence de données. Bien sûr, un algorithme de cryptage faible pourrait entraîner une élévation des privilèges ou un contournement de l'authentification, tout comme l'encodage simple le pourrait. L'utilisation d'un chiffrement faible ralentira un attaquant. Par exemple, nous savons que les chiffrements ECB conservent certains modèles de texte en clair originaux, et CBC pourrait être attaqué à l'aide d'une [attaque oracle de rembourrage](https://en.wikipedia.org/wiki/Padding_oracle_attack) .

Étant donné que les attaques de cryptographie nécessitent quelques bases mathématiques, nous avons développé un module dédié plutôt que de vous submerger avec trop d'extras ici.

Souvent, les applications Web codent les jetons. Par exemple, certains algorithmes de codage, tels que le codage hexadécimal et base64, peuvent être reconnus par un œil exercé simplement en jetant un coup d'œil rapide au jeton lui-même. D'autres sont plus délicats. Un jeton pourrait être transformé d'une manière ou d'une autre en utilisant, par exemple, XOR ou des fonctions de compression avant d'être encodé. Nous vous suggérons de toujours vérifier les octets magiques lorsque vous avez une séquence d'octets qui vous semble indésirable, car ils peuvent vous aider à identifier le format. Wikipedia a une liste de [signatures de fichiers](https://en.wikipedia.org/wiki/List_of_file_signatures) courantes pour nous aider avec ce qui précède.

Prenez cette recette chez [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)To_Hex('Space',0)Gunzip(/breakpoint)&input=SDRzSUFDNGtLR0FBL3dYQU1RMEFBQURDTUxVb29QYVB4UzRNZm4vWUJBQUFBQT09) . Le jeton est une chaîne base64 valide, qui se traduit par un ensemble d'octets hexadécimaux apparemment inutiles. Les octets magiques sont `1F 8B`, une recherche rapide sur la page des signatures de fichiers de Wikipedia indique qu'il pourrait s'agir d'un texte compressé. En mettant en pause `To hex`et en activant `Gunzip`la recette CyberChef que nous venons de relier, nous pouvons voir qu'il s'agit bien d'un contenu compressé.

Comme dit, un œil averti repérera probablement les encodeurs simplement en regardant la chaîne. [Decodify](https://github.com/s0md3v/Decodify) est un outil écrit pour automatiser les devinettes de décodage. Il ne prend pas en charge de nombreux algorithmes mais peut facilement aider à en repérer certains de base. CyberChef propose une liste massive de décodeurs, mais ils doivent être utilisés manuellement et vérifiés un par un.

Commencez par un exemple de base sur cette [recette](https://gchq.github.io/CyberChef/#recipe=From_Hex('Auto'/breakpoint)Bzip2_Decompress(false/breakpoint)&input=NDI1YTY4MzkzMTQxNTkyNjUzNTkwOWExYWM0NDAwMDAwMzBiODAxNjIwMTA5MDFkMTI0MDAwMjAwMDIyODY0ZjQ4M2RhYTEwMDAwMmE1MzlkYTYwOGYwY2Y4YmI5MjI5YzI4NDgwNGQwZDYyMjA) chez CyberChef. La recette commence par un composant en pause afin que nous puissions faire une étape à la fois.

Le texte encodé contient des caractères qui ne ressemblent pas à du code hexadécimal ASCII imprimable (imprimable ASCII) qui pourrait être imprimé sur un terminal et est compris entre 0x20 et 0x7f). Nous pouvons voir certains 0x00 et 0x09 qui sont en dehors de la plage imprimable. Nous devons donc exclure tout algorithme Base*, comme Base64 ou Base32, car ils sont construits avec un sous-ensemble d'ASCII imprimable. La chaîne ressemble toujours à une chaîne codée en hexadécimal, nous pouvons donc essayer de forcer une opération *From hex* en annulant la pause du premier bloc en cliquant sur le bouton pause pour le griser. Comme prévu, tout ce que nous avons est indésirable. En inspectant la chaîne, nous pouvons voir qu'elle commence par *42 5a* . Vérification de la page Wikipedia [Liste des signatures de fichiers](https://en.wikipedia.org/wiki/List_of_file_signatures) à la recherche de ces deux octets, également appelés *octets magiques*, nous voyons qu'ils font référence à l'algorithme bzip. Après avoir trouvé un candidat possible, nous reprenons la pause. Deuxième étape : décompresser Bzip2. La chaîne résultante est *ZW5jb2Rpbmdfcm94* et ne nous dit pas grand-chose : il pourrait y avoir une autre étape d'encodage. Nous savons que lorsqu'il est nécessaire de déplacer des données vers et depuis une application Web, un encodeur très courant est Base64, nous essayons donc de lui donner une chance.

Recherchez *Base64* dans le formulaire de recherche en haut à gauche et glissez-déposez *De Base64* à notre recette : nous avons notre chaîne décodée, comme vous pouvez le voir.

Pour devenir plus à l'aise avec CyberChef, nous vous suggérons de vous entraîner en encodant avec différents algorithmes et en inversant le flux.

Parfois, les cookies sont définis avec des valeurs aléatoires ou pseudo-aléatoires, et un décodage facile ne mène pas à une attaque réussie. C'est pourquoi nous avons souvent besoin d'automatiser la création et le test de ces cookies. Supposons que nous ayons une application Web qui crée des cookies persistants à partir de la chaîne *user_name:persistentcookie:random_5digit_value* , puis encode en base64 applique ROT13 et convertit en hexadécimal pour le stocker dans une base de données afin qu'il puisse être vérifié plus tard. Et supposons que nous sachions ou soupçonnions qu'un `htbadmin`cookie persistant a été utilisé. ROT13 est un cas particulier de Caesar Cypher où chaque caractère est tourné de 13 positions. Il était assez couramment utilisé dans le passé, mais même s'il est presque mort de nos jours, c'est une alternative intéressante à la compression bz2 pour cet exemple.

Même si l'espace des valeurs aléatoires est très petit, un test manuel est hors de question. Nous avons donc créé une preuve de concept Python pour montrer le flux possible pour automatiser ce type d'attaque. Téléchargez le script [automate_cookie_tampering.py](https://academy.hackthebox.com/storage/modules/80/scripts/automate_cookie_tampering_py.txt) , lisez et comprenez le code.

Souvent, lorsque les développeurs pensent au chiffrement, ils le considèrent comme une mesure de sécurité solide. Pourtant, dans ce cas, ils manquent le contexte : étant donné qu'il n'est vraiment pas nécessaire de stocker des données dans des cookies de session et qu'ils doivent être purement aléatoires, il n'est pas nécessaire de les chiffrer ou de les encoder.

* * * * *

Jeton de session faible
-----------------------

Même lorsque les cookies sont générés à l'aide d'une forte randomisation, ce qui donne une chaîne difficile à deviner, il est possible que le jeton ne soit pas assez long. Cela peut poser problème si l'application Web testée compte de nombreux utilisateurs simultanés, tant du point de vue fonctionnel que de la sécurité. Supposons que l'espace ne soit pas suffisant à cause du [paradoxe de l'anniversaire](https://en.wikipedia.org/wiki/Birthday_problem) . Dans ce cas, deux utilisateurs peuvent recevoir le même jeton. L'application Web doit toujours vérifier si une application nouvellement générée existe déjà et la régénérer si c'est le cas. Ce comportement permet à un attaquant de forcer plus facilement le jeton et d'en obtenir un valide.

En suivant l'exemple que nous avons vu en parlant `Short token`de la `Predictable reset token`section, nous pourrions essayer de forcer brutalement un cookie de session. Le temps nécessaire dépendra de la longueur et du jeu de caractères utilisé pour créer le jeton lui-même. Étant donné qu'il s'agit d'un jeu de devinettes, nous pensons qu'une approche véritablement progressive qui commence par `aaaaaa`ne `zzzzzz`rapporterait pas de dividendes. C'est pourquoi nous préférons utiliser `John the Ripper`, qui génère des valeurs non linéaires, pour notre session de force brute.

Nous devrions examiner certaines valeurs de cookie, et après avoir observé que la longueur est de six et que le jeu de caractères se compose de caractères minuscules et de chiffres, nous pouvons lancer notre attaque.

[Wfuzz](https://github.com/xmendez/wfuzz/) peut être alimenté par un autre programme à l'aide d'un tuyau classique, et nous l'utiliserons `John`comme feeder. Nous définissons `John`en mode incrémentiel, en utilisant le jeu de caractères intégré "LowerNum" qui correspond à notre observation ( `--incremental=LowerNum`), nous spécifions une longueur de mot de passe de 6 caractères ( `--min-length=6 --max-length=6`), et nous demandons également `John`d'imprimer la sortie en tant que stdout ( `--stdout`). Ensuite, `wfuzz`utilise la charge utile de stdin ( `-z stdin`) et fuzz le cookie "HTBSESS" ( `-b HTBSESS=FUZZ`), en recherchant la chaîne `"Welcome"`( `--ss "Welcome"`) dans les réponses du serveur pour l'URL donnée.

```
dsgsec@htb[/htb]$ john --incremental=LowerNum --min-length=6 --max-length=6 --stdout| wfuzz -z stdin -b HTBSESS=FUZZ --ss "Welcome" -u https://brokenauthentication.hackthebox.eu/profile.php

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://brokenauth.hackthebox.eu/
Total requests: <<unknown>>

===================================================================
ID           Response   Lines    Word     Chars       Payload
===================================================================

Created directory: ~/john-the-ripper/297/.john
Press 'q' or Ctrl-C to abort, almost any other key for status
000009897:   200        9 L      31 W     274 Ch      "abaney"

```

Cette attaque peut prendre beaucoup de temps et est irréalisable si le jeton est long. Les chances sont plus élevées si les cookies durent plus longtemps qu'une seule session, mais ne vous attendez pas à une victoire rapide ici. Nous vous encourageons à vous exercer à utiliser un fichier PHP que vous pouvez télécharger [ici](https://academy.hackthebox.com/storage/modules/80/scripts/bruteforce_cookie_php.txt) . Veuillez lire attentivement le fichier et essayer de lui faire imprimer le message de félicitations.
