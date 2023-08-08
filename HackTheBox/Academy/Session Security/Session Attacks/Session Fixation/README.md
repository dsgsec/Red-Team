Fixation de session
===================

* * * * *

La fixation de session se produit lorsqu'un attaquant peut fixer un identifiant de session (valide). Comme vous pouvez l'imaginer, l'attaquant devra alors tromper la victime pour qu'elle se connecte à l'application en utilisant l'identifiant de session susmentionné. Si la victime le fait, l'attaquant peut procéder à une attaque de Session Hijacking (puisque l'identifiant de session est déjà connu).

De tels bogues se produisent généralement lorsque les identifiants de session (tels que les cookies) sont acceptés à partir *des chaînes de requête d'URL* ou *des données de publication* (plus d'informations à ce sujet dans un instant).

Les attaques de fixation de session sont généralement montées en trois étapes :

Étape 1 : L'attaquant parvient à obtenir un identifiant de session valide

L'authentification auprès d'une application n'est pas toujours nécessaire pour obtenir un identifiant de session valide, et un grand nombre d'applications attribuent des identifiants de session valides à quiconque les parcourt. Cela signifie également qu'un attaquant peut se voir attribuer un identifiant de session valide sans avoir à s'authentifier.

Note : Un attaquant peut également obtenir un identifiant de session valide en créant un compte sur l'application ciblée (si cela est possible).

Étape 2 : l'attaquant parvient à fixer un identifiant de session valide

Ce qui précède est un comportement attendu, mais il peut se transformer en une vulnérabilité de fixation de session si :

-   L'identifiant de session attribué avant la connexion reste le même après la connexion`and`
-   Les identifiants de session (tels que les cookies) sont acceptés à partir *des chaînes de requête d'URL* ou *des données de publication* et propagés à l'application

Si, par exemple, un paramètre lié à la session est inclus dans l'URL (et non dans l'en-tête du cookie) et que toute valeur spécifiée finit par devenir un identifiant de session, l'attaquant peut fixer une session.

Étape 3 : L'attaquant trompe la victime pour qu'elle établisse une session en utilisant l'identifiant de session mentionné ci-dessus

Tout ce que l'attaquant a à faire est de créer une URL et d'inciter la victime à la visiter. Si la victime le fait, l'application web attribuera alors cet identifiant de session à la victime.

L'attaquant peut alors procéder à une attaque par détournement de session puisque l'identifiant de session est déjà connu.

* * * * *

Exemple de fixation de session
------------------------------

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `oredirect.htb.net`) spécifié pour accéder à l'application.

Partie 1 : Identification de la fixation de la session

Accédez à `oredirect.htb.net`. Vous tomberez sur une URL au format ci-dessous :

`http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN VALUE>`

À l'aide des outils de développement Web (Maj+Ctrl+I dans le cas de Firefox), notez que l'application utilise un cookie de session nommé `PHPSESSID`et que la valeur du cookie est la même que la `token`valeur du paramètre sur l'URL.

![image](https://academy.hackthebox.com/storage/modules/153/18.png)

Si une valeur ou un identifiant de session valide spécifié dans le `token`paramètre de l'URL est propagé à la `PHPSESSID`valeur du cookie, nous avons probablement affaire à une vulnérabilité de fixation de session.

Voyons si c'est le cas, comme suit.

Partie 2 : Tentative d'exploitation de fixation de session

Ouvrez un `New Private Window`et accédez à`http://oredirect.htb.net/?redirect_uri=/complete.html&token=IControlThisCookie`

À l'aide des outils de développement Web (Maj+Ctrl+I dans le cas de Firefox), notez que la `PHPSESSID`valeur du cookie est`IControlThisCookie`

![image](https://academy.hackthebox.com/storage/modules/153/19.png)

Nous avons affaire à une vulnérabilité de fixation de session. Un attaquant pourrait envoyer une URL similaire à celle ci-dessus à une victime. Si la victime se connecte à l'application, l'attaquant pourrait facilement détourner sa session puisque l'identifiant de session est déjà connu (l'attaquant l'a fixé).

Remarque : Une autre façon d'identifier cela consiste à mettre aveuglément le nom et la valeur de l'identifiant de session dans l'URL, puis à l'actualiser.

Par exemple, supposons que nous recherchions `http://insecure.exampleapp.com/login`des bogues de fixation de session et que l'identifiant de session utilisé soit un cookie nommé `PHPSESSID`. Pour tester la fixation de session, nous pourrions essayer ce qui suit `http://insecure.exampleapp.com/login?PHPSESSID=AttackerSpecifiedCookieValue`et voir si la valeur de cookie spécifiée est propagée à l'application (comme nous l'avons fait dans l'exercice de laboratoire de cette section).

Vous trouverez ci-dessous le code vulnérable de l'exercice de laboratoire de cette section.

Code : php

```
<?php
    if (!isset($_GET["token"])) {
        session_start();
        header("Location: /?redirect_uri=/complete.html&token=" . session_id());
    } else {
        setcookie("PHPSESSID", $_GET["token"]);
    }
?>

```

Décomposons le morceau de code ci-dessus.

Code : php

```
if (!isset($_GET["token"])) {
     session_start();

```

Le morceau de code ci-dessus peut être traduit comme suit : Si le paramètre *de jeton* n'a pas été défini, démarrez une session (générez et fournissez un identifiant de session valide).

Code : php

```
header("Location: /?redirect_uri=/complete.html&token=" . session_id());

```

Le morceau de code ci-dessus peut être traduit comme suit : Redirigez l'utilisateur vers `/?redirect_uri=/complete.html&token=`puis appelez la fonction *session_id()* pour ajouter *session_id* à la valeur du jeton.

Code : php

```
 } else {
        setcookie("PHPSESSID", $_GET["token"]);
    }

```

Le morceau de code ci-dessus peut être traduit comme suit : Si le paramètre *token est déjà défini (instruction else), définissez **PHPSESSID* sur la valeur du paramètre *token . *Toute URL au format suivant `http://oredirect.htb.net/?redirect_uri=/complete.html&token=AttackerSpecifiedCookieValue`mettra à jour la valeur de *PHPSESSID* avec la valeur du paramètre *de jeton .*

À ce jour, nous avons couvert le détournement de session et la fixation de session. À l'avenir, voyons comment un chasseur de primes de bogues ou un testeur d'intrusion peut obtenir des identifiants de session valides qui peuvent ensuite être utilisés pour détourner la session d'un utilisateur.
