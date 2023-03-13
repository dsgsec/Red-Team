Clause syndicale
============

* * * * *

Jusqu'à présent, nous n'avons manipulé la requête d'origine que pour subvertir la logique de l'application Web et contourner l'authentification, à l'aide de l'opérateur `OR` et des commentaires. Cependant, un autre type d'injection SQL consiste à injecter des requêtes SQL entières exécutées avec la requête d'origine. Cette section le démontrera en utilisant la clause MySQL `Union` pour faire `SQL Union Injection`.

* * * * *

syndicat
-----

Avant de commencer à en savoir plus sur Union Injection, nous devons d'abord en savoir plus sur la clause SQL Union. La clause [Union](https://dev.mysql.com/doc/refman/8.0/en/union.html) est utilisée pour combiner les résultats de plusieurs instructions `SELECT` . Cela signifie que grâce à une injection `UNION` , nous serons en mesure de `SELECT` et de vider les données de tout le SGBD, à partir de plusieurs tables et bases de données. Essayons d'utiliser l'opérateur `UNION` dans un exemple de base de données. Voyons d'abord le contenu de la table `ports` :

```
mysql> SELECT * FROM ports ;

+----------+-----------+
| code | ville |
+----------+-----------+
| CN SHA | Shangai |
| SG SIN | Singapour |
| ZZ-21 | Shenzhen |
+----------+-----------+
3 rangées en série (0.00 sec)

```

Voyons ensuite le résultat des tables `ships` :

```
mysql> SELECT * FROM navires ;

+----------+-----------+
| Navire | ville |
+----------+-----------+
| Morisson | New-York |
+----------+-----------+
1 rangées dans l'ensemble (0,00 sec)

```

Essayons maintenant d'utiliser `UNION` pour combiner les deux résultats :

```
mysql> SELECT * FROM ports UNION SELECT * FROM navires ;

+----------+-----------+
| code | ville |
+----------+-----------+
| CN SHA | Shangai |
| SG SIN | Singapour |
| Morisson | New-York |
| ZZ-21 | Shenzhen |
+----------+-----------+
4 rangées en série (0.00 sec)

```

Comme nous pouvons le voir, `UNION` a combiné la sortie des deux instructions `SELECT` en une seule, de sorte que les entrées de la table `ports` et de la table `ships` ont été combinées en une seule sortie avec quatre lignes. Comme nous pouvons le voir, certaines lignes appartiennent à la table `ports` tandis que d'autres appartiennent à la table `ships` .

Remarque : Les types de données des colonnes sélectionnées sur toutes les positions doivent être les mêmes.

* * * * *

Colonnes paires
------------

Une instruction `UNION` ne peut fonctionner que sur des instructions `SELECT` avec un nombre égal de colonnes. Par exemple, si nous essayons `UNION` deux requêtes dont les résultats contiennent un nombre différent de colonnes, nous obtenons l'erreur suivante :

```
mysql> SELECT ville FROM ports UNION SELECT * FROM navires;

ERREUR 1222 (21000) : Les instructions SELECT utilisées ont un nombre différent de colonnes

```

La requête ci-dessus génère une erreur, car la première `SELECT` renvoie une colonne et la seconde `SELECT` en renvoie deux. Une fois que nous avons deux requêtes qui renvoient le même nombre de colonnes, nous pouvons utiliser l'opérateur `UNION` pour extraire des données d'autres tables et bases de données.

Par exemple, si la requête est :

Code : sql

```
SELECT * FROM produits WHERE product_id = 'user_input'

```

Nous pouvons injecter une requête `UNION` dans l'entrée, de sorte que les lignes d'une autre table soient renvoyées :

Code : sql

```
SELECT * from products where product_id = '1' UNION SELECT username, password from passwords-- '

```

La requête ci-dessus renvoie les entrées `username` et `password` de la table `passwords` , en supposant que la table `products` comporte deux colonnes.

* * * * *

Colonnes inégales
---------------

Nous découvrirons que la requête d'origine n'aura généralement pas le même nombre de colonnes que la requête SQL que nous voulons exécuter, nous devrons donc contourner cela. Par exemple, supposons que nous n'ayons qu'une seule colonne. Dans ce cas, nous voulons `SELECT`, nous pouvons mettre des données inutiles pour les colonnes requises restantes afin que le nombre total de colonnes avec lesquelles nous sommes `UNION`ing reste le même que la requête d'origine.

Par exemple, nous pouvons utiliser n'importe quelle chaîne comme données indésirables, et la requête renverra la chaîne comme sortie pour cette colonne. Si nous `UNION` avec la chaîne `"junk"`, la requête `SELECT` serait `SELECT "junk" from passwords`, qui renverra toujours `junk`. Nous pouvons aussi utiliser des nombres. Par exemple, la requête `SELECT 1 from passwords` renverra toujours `1` comme résultat.

Remarque : Lorsque vous remplissez d'autres colonnes avec des données indésirables, nous devons nous assurer que le type de données correspond au type de données des colonnes, sinon la requête renverra une erreur. Par souci de simplicité, nous utiliserons des nombres comme données indésirables, ce qui deviendra également utile pour suivre nos positions de charges utiles, comme nous le verrons plus tard.

Conseil : pour une injection SQL avancée, nous pouvons simplement utiliser 'NULL' pour remplir d'autres colonnes, car 'NULL' convient à tous les types de données.

La table `products` comporte deux colonnes dans l'exemple ci-dessus, nous devons donc `UNION` avec deux colonnes. Si nous ne voulions obtenir qu'une seule colonne 'par ex. `username`', nous devons faire `username, 2`, de sorte que nous ayons le même nombre de colonnes :

Code : sql

```
SELECT * parmi les produits où product_id = '1' UNION SELECT nom d'utilisateur, 2 parmi les mots de passe

```

Si nous avions plus de colonnes dans la table de la requête d'origine, nous devons ajouter plus de nombres pour créer les colonnes requises restantes. Par exemple, si l'orequête originale utilisée `SELECT` sur une table à quatre colonnes, notre injection `UNION` serait :

Code : sqlUnion Injection
===============

* * * * *

Maintenant que nous savons comment fonctionne la clause Union et comment l'utiliser, apprenons à l'utiliser dans nos injections SQL. Prenons l'exemple suivant :

![](https://academy.hackthebox.com/storage/modules/33/ports_cn.png)

Nous voyons une potentielle injection SQL dans les paramètres de recherche. Nous appliquons les étapes SQLi Discovery en injectant un guillemet simple (`'`), et nous obtenons une erreur :

![](https://academy.hackthebox.com/storage/modules/33/ports_quote.png)

Puisque nous avons causé une erreur, cela peut signifier que la page est vulnérable à l'injection SQL. Ce scénario est idéal pour l'exploitation via l'injection basée sur Union, car nous pouvons voir les résultats de nos requêtes.

* * * * *

Détecter le nombre de colonnes
------------------------

Avant d'aller de l'avant et d'exploiter les requêtes basées sur Union, nous devons trouver le nombre de colonnes sélectionnées par le serveur. Il existe deux méthodes pour détecter le nombre de colonnes :

- Utiliser `ORDER BY`
- Utiliser `UNION`

#### Utiliser ORDRE PAR

La première façon de détecter le nombre de colonnes consiste à utiliser la fonction `ORDER BY` , dont nous avons parlé précédemment. Nous devons injecter une requête qui trie les résultats selon une colonne que nous avons spécifiée, "c'est-à-dire la colonne 1, la colonne 2, etc.", jusqu'à ce que nous obtenions une erreur indiquant que la colonne spécifiée n'existe pas.

Par exemple, nous pouvons commencer par `ordonner par 1`, trier par la première colonne et réussir, car le tableau doit avoir au moins une colonne. Ensuite, nous ferons `commander par 2` puis `commander par 3` jusqu'à ce que nous atteignions un nombre qui renvoie une erreur, ou que la page n'affiche aucune sortie, ce qui signifie que ce numéro de colonne n'existe pas. La dernière colonne réussie que nous avons triée avec succès nous donne le nombre total de colonnes.

Si nous n'avons pas réussi à trier par 4, cela signifie que le tableau comporte trois colonnes, ce qui correspond au nombre de colonnes que nous avons pu trier avec succès. Revenons à notre exemple précédent et essayons la même chose, avec la charge utile suivante :

Code : sql

```
' commander par 1-- -

```

Rappel : Nous ajoutons un tiret supplémentaire (-) à la fin, pour vous indiquer qu'il y a un espace après (--).

Comme on le voit, on obtient un résultat normal :

![](https://academy.hackthebox.com/storage/modules/33/ports_cn.png)

Essayons ensuite de trier par la deuxième colonne, avec la charge utile suivante :

Code : sql

```
' commander par 2-- -

```

Nous obtenons toujours les résultats. On remarque qu'ils sont triés différemment, comme prévu :

![](https://academy.hackthebox.com/storage/modules/33/order_by_2.jpg)

Nous faisons de même pour les colonnes `3` et `4` et obtenons les résultats. Cependant, lorsque nous essayons de  `ORDER BY` la colonne 5, nous obtenons l'erreur suivante :

![](https://academy.hackthebox.com/storage/modules/33/order_by_5.jpg)

Cela signifie que ce tableau a exactement 4 colonnes.

#### Utiliser UNION

L'autre méthode consiste à tenter une injection Union avec un nombre différent de colonnes jusqu'à ce que nous récupérions les résultats. La première méthode renvoie toujours les résultats jusqu'à ce que nous rencontrions une erreur, tandis que cette méthode donne toujours une erreur jusqu'à ce que nous obtenions un succès. Nous pouvons commencer par injecter une requête `UNION` à 3 colonnes :

Code : sql

```
cn' UNION sélectionner 1,2,3-- -

```

Nous obtenons une erreur indiquant que le nombre de colonnes ne correspond pas :

![](https://academy.hackthebox.com/storage/modules/33/ports_columns_diff.png)

Alors, essayons quatre colonnes et voyons la réponse :

Code : sql

```
cn' UNION sélectionner 1,2,3,4-- -

```

![](https://academy.hackthebox.com/storage/modules/33/ports_columns_correct.png)

Cette fois, nous obtenons les résultats avec succès, ce qui signifie une fois de plus que le tableau comporte 4 colonnes. Nous pouvons utiliser l'une ou l'autre méthode pour déterminer le nombre de colonnes. Une fois que nous connaissons le nombre de colonnes, nous savons comment former notre charge utile et nous pouvons passer à l'étape suivante.

* * * * *

Lieu d'injection
---------------------

Alors qu'une requête peut renvoyer plusieurs colonnes, l'application Web peut n'en afficher que certaines. Ainsi, si nous injectons notre requête dans une colonne qui n'est pas imprimée sur la page, nous n'obtiendrons pas sa sortie. C'est pourquoi nous devons déterminer quelles colonnes sont imprimées sur la page, pour déterminer où placer notre injection. Dans l'exemple précédent, alors que la requête injectée renvoyait 1, 2, 3 et 4, nous n'avons vu que 2, 3 et 4 s'afficher sur la page en tant que données de sortie :

![](https://academy.hackthebox.com/storage/modules/33/ports_columns_correct.png)

Il est très courant que toutes les colonnes ne soient pas affichées à l'utilisateur. Par exemple, le champ ID est souvent utilisé pour lier différentes tables entre elles, mais l'utilisateur n'a pas besoin de le voir. Cela nous indique que les colonnes 2 et 3 et 4 sont imprimées pour placer notre injection dans l'une d'elles. `Nous ne pouvons pas placer notre injection au début, sinon sa sortie ne sera pas imprimée.`

C'est l'avantage d'utiliser des nombres comme données indésirables, car cela facilite le suivi des colonnes imprimées, nous savons donc dans quelle colonne placer notre requête. Pour tester que nous pouvons obtenir des données réelles de la base de données "plutôt que de simples chiffres", nous pouvons utiliser la requête SQL `@@version` commetestez et placez-le dans la deuxième colonne à la place du chiffre 2 :

Code : sql

```
cn' UNION select 1,@@version,3,4-- -

```

![](https://academy.hackthebox.com/storage/modules/33/db_version_1.jpg)

Comme nous pouvons le voir, nous pouvons afficher la version de la base de données. Nous savons maintenant comment former nos charges utiles d'injection Union SQL pour obtenir avec succès la sortie de notre requête imprimée sur la page. Dans la section suivante, nous verrons comment énumérer la base de données et obtenir des données à partir d'autres tables et bases de données.

Plein écran  Terminer

```
UNION SELECT nom d'utilisateur, 2, 3, 4 parmi les mots de passe-- '

```

Cette requête renverrait :

```
mysql> SELECT * des produits où product_id UNION SELECT username, 2, 3, 4 from passwords-- '

+-----------+-----------+-----------+-----------+
| produit_1 | produit_2 | produit_3 | produit_4 |
+-----------+-----------+-----------+-----------+
| administrateur | 2 | 3 | 4 |
+-----------+-----------+-----------+-----------+

```

Comme nous pouvons le voir, notre sortie recherchée de la requête '`UNION SELECT username from passwords`' se trouve dans la première colonne de la deuxième ligne, tandis que les nombres remplissent les colonnes restantes.