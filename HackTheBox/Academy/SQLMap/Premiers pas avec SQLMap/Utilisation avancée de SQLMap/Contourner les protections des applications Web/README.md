Contourner les protections des applications Web
===============================================

* * * * *

Dans un scénario idéal, aucune protection ne sera déployée du côté cible, ce qui n'empêche pas une exploitation automatique. Sinon, nous pouvons nous attendre à des problèmes lors de l'exécution d'un outil automatisé de quelque nature que ce soit sur une telle cible. Néanmoins, de nombreux mécanismes sont incorporés dans SQLMap, ce qui peut nous aider à contourner avec succès ces protections.

* * * * *

Contournement du jeton anti-CSRF
--------------------------------

L'une des premières lignes de défense contre l'utilisation d'outils d'automatisation est l'incorporation de jetons anti-CSRF (c'est-à-dire Cross-Site Request Forgery) dans toutes les requêtes HTTP, en particulier celles générées à la suite du remplissage d'un formulaire Web.

En termes plus simples, chaque requête HTTP dans un tel scénario devrait avoir une valeur de jeton (valide) disponible uniquement si l'utilisateur a réellement visité et utilisé la page. Alors que l'idée originale était de prévenir les scénarios contenant des liens malveillants, dans lesquels le simple fait d'ouvrir ces liens aurait des conséquences indésirables pour les utilisateurs connectés inconscients (par exemple, ouvrir les pages d'administrateur et ajouter un nouvel utilisateur avec des informations d'identification prédéfinies), cette fonctionnalité de sécurité a également été renforcée par inadvertance. les applications contre l'automatisation (indésirable).

Néanmoins, SQLMap propose des options qui peuvent aider à contourner la protection anti-CSRF. À savoir, l'option la plus importante est `--csrf-token`. En spécifiant le nom du paramètre de jeton (qui devrait déjà être disponible dans les données de requête fournies), SQLMap tentera automatiquement d'analyser le contenu de la réponse cible et recherchera de nouvelles valeurs de jeton afin de pouvoir les utiliser dans la requête suivante.

De plus, même dans le cas où l'utilisateur ne spécifie pas explicitement le nom du jeton via `--csrf-token`, si l'un des paramètres fournis contient l'un des infixes courants (c'est-à-dire `csrf`, `xsrf`, `token`), l'utilisateur sera invité à le mettre à jour dans d'autres requêtes :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"

        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.9}
|_ -| . [']     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 22:18:01 /2020-09-18/

POST parameter 'csrf-token' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] y

```

* * * * *

Contournement de valeur unique
------------------------------

Dans certains cas, l'application Web peut exiger uniquement que des valeurs uniques soient fournies dans des paramètres prédéfinis. Un tel mécanisme est similaire à la technique anti-CSRF décrite ci-dessus, sauf qu'il n'est pas nécessaire d'analyser le contenu de la page Web. Ainsi, en s'assurant simplement que chaque requête a une valeur unique pour un paramètre prédéfini, l'application Web peut facilement empêcher les tentatives CSRF tout en évitant certains outils d'automatisation. `--randomize`Pour cela, il faut utiliser l'option , pointant vers le nom du paramètre contenant une valeur qui doit être randomisée avant d'être envoyée :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5 | grep URI

URI: http://www.example.com:80/?id=1&rp=99954
URI: http://www.example.com:80/?id=1&rp=87216
URI: http://www.example.com:80/?id=9030&rp=36456
URI: http://www.example.com:80/?id=1.%2C%29%29%27.%28%28%2C%22&rp=16689
URI: http://www.example.com:80/?id=1%27xaFUVK%3C%27%22%3EHKtQrg&rp=40049
URI: http://www.example.com:80/?id=1%29%20AND%209368%3D6381%20AND%20%287422%3D7422&rp=95185

```

* * * * *

Contournement des paramètres calculés
-------------------------------------

Un autre mécanisme similaire est celui où une application Web s'attend à ce qu'une valeur de paramètre appropriée soit calculée en fonction d'une ou plusieurs autres valeurs de paramètre. Le plus souvent, une valeur de paramètre doit contenir le résumé du message (par exemple `h=MD5(id)`) d'une autre. Pour contourner cela, il convient d'utiliser l'option `--eval`où un code Python valide est évalué juste avant que la requête ne soit envoyée à la cible :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI

URI: http://www.example.com:80/?id=1&h=c4ca4238a0b923820dcc509a6f75849b
URI: http://www.example.com:80/?id=1&h=c4ca4238a0b923820dcc509a6f75849b
URI: http://www.example.com:80/?id=9061&h=4d7e0d72898ae7ea3593eb5ebf20c744
URI: http://www.example.com:80/?id=1%2C.%2C%27%22.%2C%28.%29&h=620460a56536e2d32fb2f4842ad5a08d
URI: http://www.example.com:80/?id=1%27MyipGP%3C%27%22%3EibjjSu&h=db7c815825b14d67aaa32da09b8b2d42
URI: http://www.example.com:80/?id=1%29%20AND%209978%socks4://177.39.187.70:33283ssocks4://177.39.187.70:332833D1232%20AND%20%284955%3D4955&h=02312acd4ebe69e2528382dfff7fc5cc

```

* * * * *

Dissimulation d'adresse IP
--------------------------

Dans le cas où nous souhaitons cacher notre adresse IP, ou si une certaine application Web dispose d'un mécanisme de protection qui met sur liste noire notre adresse IP actuelle, nous pouvons essayer d'utiliser un proxy ou le réseau d'anonymat Tor. Un proxy peut être défini avec l'option `--proxy`(par exemple `--proxy="socks4://177.39.187.70:33283"`), où nous devons ajouter un proxy fonctionnel.

En plus de cela, si nous avons une liste de proxys, nous pouvons les fournir à SQLMap avec l'option `--proxy-file`. De cette façon, SQLMap parcourra séquentiellement la liste, et en cas de problème (par exemple, liste noire de l'adresse IP), il passera simplement de l'adresse actuelle à la suivante dans la liste. L'autre option est l'utilisation du réseau Tor pour fournir une anonymisation facile à utiliser, où notre adresse IP peut apparaître n'importe où parmi une grande liste de nœuds de sortie Tor. Lorsqu'il est correctement installé sur la machine locale, il doit y avoir un `SOCKS4`service proxy sur le port local 9050 ou 9150. En utilisant switch `--tor`, SQLMap essaiera automatiquement de trouver le port local et de l'utiliser de manière appropriée.

Si nous voulions être sûrs que Tor est correctement utilisé, pour éviter tout comportement indésirable, nous pourrions utiliser le commutateur `--check-tor`. Dans de tels cas, SQLMap se connectera au `https://check.torproject.org/`et vérifiera la réponse pour le résultat attendu (c'est-à-dire qu'elle `Congratulations`apparaîtra à l'intérieur).

* * * * *

Contournement WAF
-----------------

Chaque fois que nous exécutons SQLMap, dans le cadre des tests initiaux, SQLMap envoie une charge utile prédéfinie d'apparence malveillante en utilisant un nom de paramètre inexistant (par exemple) `?pfov=...`pour tester l'existence d'un WAF (Web Application Firewall). Il y aura un changement substantiel dans la réponse par rapport à l'original en cas de protection entre l'utilisateur et la cible. Par exemple, si l'une des solutions WAF les plus populaires (ModSecurity) est implémentée, il devrait y avoir une `406 - Not Acceptable`réponse après une telle demande.

En cas de détection positive, pour identifier le mécanisme de protection réel, SQLMap utilise une bibliothèque tierce [identYwaf](https://github.com/stamparm/identYwaf) , contenant les signatures de 80 solutions WAF différentes. Si nous voulons ignorer complètement ce test heuristique (c'est-à-dire produire moins de bruit), nous pouvons utiliser switch `--skip-waf`.

* * * * *

Contournement de la liste noire des agents utilisateurs
-------------------------------------------------------

En cas de problèmes immédiats (par exemple, le code d'erreur HTTP 5XX dès le début) lors de l'exécution de SQLMap, l'une des premières choses à laquelle nous devrions penser est la potentielle liste noire de l'agent utilisateur par défaut utilisé par SQLMap (par exemple) `User-agent: sqlmap/1.4.9 (http://sqlmap.org)`.

Il est trivial de contourner cela avec le switch `--random-agent`, qui modifie l'agent utilisateur par défaut avec une valeur choisie au hasard parmi un large pool de valeurs utilisées par les navigateurs.

Remarque : Si une forme de protection est détectée pendant l'exécution, nous pouvons nous attendre à des problèmes avec la cible, voire avec d'autres mécanismes de sécurité. La raison principale est le développement continu et les nouvelles améliorations de ces protections, laissant une marge de manœuvre de plus en plus réduite aux attaquants.

* * * * *

Scripts de falsification
------------------------

Enfin, l'un des mécanismes les plus populaires implémentés dans SQLMap pour contourner les solutions WAF/IPS sont les scripts dits de « falsification ». Les scripts de falsification sont un type spécial de scripts (Python) écrits pour modifier les requêtes juste avant d'être envoyées à la cible, dans la plupart des cas pour contourner une certaine protection.

Par exemple, l'un des scripts de falsification les plus populaires [consiste](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/between.py) à remplacer toutes les occurrences de l'opérateur supérieur à ( `>`) par `NOT BETWEEN 0 AND #`et de l'opérateur égal ( `=`) par `BETWEEN # AND #`. De cette façon, de nombreux mécanismes de protection primitifs (principalement axés sur la prévention des attaques XSS) sont facilement contournés, du moins à des fins SQLi.

Les scripts de falsification peuvent être enchaînés les uns après les autres au sein de l' `--tamper`option (par exemple `--tamper=between,randomcase`), où ils sont exécutés en fonction de leur priorité prédéfinie. Une priorité est prédéfinie pour éviter tout comportement indésirable, car certains scripts modifient les charges utiles en modifiant leur syntaxe SQL (par exemple [ifnull2ifisnull](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/ifnull2ifisnull.py) ). En revanche, certains scripts de falsification ne se soucient pas du contenu interne (par exemple [appendnullbyte](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/appendnullbyte.py) ).

Les scripts de falsification peuvent modifier n'importe quelle partie de la requête, bien que la majorité modifie le contenu de la charge utile. Les scripts de falsification les plus notables sont les suivants :

| Tamper-Script | Description |
| --- | --- |
| `0eunion` | Remplace les instances deUNION avece0UNION |
| `base64encode` | Base64 encode tous les caractères d'une charge utile donnée |
| `between` | Remplace l'opérateur supérieur à ( `>`) par `NOT BETWEEN 0 AND #`et l'opérateur égal ( `=`) par`BETWEEN # AND #` |
| `commalesslimit` | Remplace les instances (MySQL) comme `LIMIT M, N`par `LIMIT N OFFSET M`leur homologue |
| `equaltolike` | Remplace toutes les occurrences de l'opérateur égal ( `=`) par sa `LIKE`contrepartie |
| `halfversionedmorekeywords` | Ajoute un commentaire versionné (MySQL) avant chaque mot-clé |
| `modsecurityversioned` | Adopte une requête complète avec un commentaire versionné (MySQL) |
| `modsecurityzeroversioned` | Adopte une requête complète avec un commentaire (MySQL) sans version |
| `percentage` | Ajoute un signe de pourcentage ( `%`) devant chaque caractère (par exemple SELECT -> %S%E%L%E%C%T) |
| `plus2concat` | Remplace l'opérateur plus (`+` ) par la fonction (MsSQL) homologue CONCAT() |
| `randomcase` | Remplace chaque caractère de mot-clé par une valeur de casse aléatoire (par exemple SELECT -> SEleCt) |
| `space2comment` | Remplace le caractère espace ( ) par des commentaires `/ |
| `space2dash` | Remplace le caractère espace ( ) par un commentaire tiret ( `--`) suivi d'une chaîne aléatoire et d'une nouvelle ligne ( `\n`) |
| `space2hash` | Remplace les instances (MySQL) du caractère espace ( ) par un caractère dièse ( `#`) suivi d'une chaîne aléatoire et d'une nouvelle ligne ( `\n`) |
| `space2mssqlblank` | Remplace (MsSQL) les instances du caractère espace ( ) par un caractère vide aléatoire provenant d'un ensemble valide de caractères alternatifs |
| `space2plus` | Remplace le caractère espace ( ) par plus ( `+`) |
| `space2randomblank` | Remplace le caractère espace ( ) par un caractère vide aléatoire provenant d'un ensemble valide de caractères alternatifs. |
| `symboliclogical` | Remplace les opérateurs logiques AND et OR par leurs homologues symboliques ( `&&`et `||`) |
| `versionedkeywords` | Entoure chaque mot-clé non-fonctionnel d'un commentaire versionné (MySQL) |
| `versionedmorekeywords` | Entoure chaque mot-clé d'un commentaire versionné (MySQL) |

Pour obtenir une liste complète des scripts de falsification implémentés, ainsi que la description ci-dessus, switch `--list-tampers`peut être utilisé. Nous pouvons également développer des scripts Tamper personnalisés pour tout type d'attaque personnalisé, comme un SQLi de second ordre.

* * * * *

Divers contournements
---------------------

Parmi les autres mécanismes de contournement de la protection, il y en a deux autres qui méritent d'être mentionnés. Le premier est le `Chunked`codage de transfert, activé à l'aide du commutateur `--chunked`, qui divise le corps de la requête POST en « morceaux ». Les mots-clés SQL sur liste noire sont répartis entre des morceaux de manière à ce que la requête les contenant puisse passer inaperçue.

L'autre mécanisme de contournement est le `HTTP parameter pollution`( `HPP`), où les charges utiles sont divisées de la même manière que dans le cas de `--chunked`différents mêmes paramètres nommés valeurs (par exemple `?id=1&id=UNION&id=SELECT&id=username,password&id=FROM&id=users...`), qui sont concaténés par la plate-forme cible si elle la prend en charge (par exemple `ASP`).
