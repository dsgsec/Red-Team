Utilisation des commentaires
==============

* * * * *

Cette section apprendra à utiliser les commentaires pour subvertir la logique des requêtes SQL plus avancées et se retrouver avec une requête SQL fonctionnelle pour contourner le processus d'authentification de connexion.

* * * * *

commentaires
--------

Comme tout autre langage, SQL permet également l'utilisation de commentaires. Les commentaires sont utilisés pour documenter les requêtes ou ignorer une certaine partie de la requête. Nous pouvons utiliser deux types de commentaires de ligne avec MySQL `-- `et `#`, en plus d'un commentaire en ligne `/**/` (bien que cela ne soit généralement pas utilisé dans les injections SQL). Le `-- `peut être utilisé comme suit :

```
mysql> SELECT username FROM logins ; -- Sélectionne les noms d'utilisateur dans le tableau des connexions

+---------------+
| nom d'utilisateur |
+---------------+
| administrateur |
| administrateur |
| jean |
| tom |
+---------------+
4 rangées en série (0.00 sec)

```

Remarque : en SQL, l'utilisation de deux tirets seulement n'est pas suffisante pour commencer un commentaire. Donc, il doit y avoir un espace vide après eux, donc le commentaire commence par (-- ), avec un espace à la fin. Il s'agit parfois d'une URL encodée sous la forme (--+), car les espaces dans les URL sont encodés sous la forme (+). Pour que ce soit clair, nous ajouterons un autre (-) à la fin (-- -), pour montrer l'utilisation d'un caractère espace.

Le symbole `#` peut également être utilisé.

```
mysql> SELECT * FROM logins WHERE username = 'admin'; # Vous pouvez placer n'importe quoi ici ET mot de passe = 'quelque chose'

+----+----------+----------+---------------------+
| identifiant | nom d'utilisateur | mot de passe | date_d'adhésion |
+----+----------+----------+---------------------+
| 1 | administrateur | p@ssw0rd | 2020-07-02 00:00:00 |
+----+----------+----------+---------------------+
1 ligne dans l'ensemble (0,00 sec)

```

Conseil : si vous saisissez votre charge utile dans l'URL d'un navigateur, un symbole (#) est généralement considéré comme une balise et ne sera pas transmis dans le cadre de l'URL. Afin d'utiliser (#) comme commentaire dans un navigateur, nous pouvons utiliser '%23', qui est un symbole URL encodé (#).

Le serveur ignorera la partie de la requête contenant `AND password = 'something'` lors de l'évaluation.

* * * * *

Contournement d'authentification avec commentaires
-------------------------

Revenons à notre exemple précédent et injectons `admin'-- `comme nom d'utilisateur. La requête finale sera :

Code : sql

```
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'quelque chose';

```

Comme nous pouvons le voir à partir de la coloration syntaxique, le nom d'utilisateur est désormais `admin` et le reste de la requête est désormais ignoré en tant que commentaire. De plus, de cette façon, nous pouvons nous assurer que la requête ne présente aucun problème de syntaxe.

Essayons de les utiliser sur la page de connexion, et connectez-vous avec le nom d'utilisateur `admin'-- `et n'importe quoi comme mot de passe :

![admin_dash](https://academy.hackthebox.com/storage/modules/33/admin_dash.png)

Comme nous le voyons, nous avons pu contourner l'authentification, car la nouvelle requête modifiée vérifie le nom d'utilisateur, sans aucune autre condition.

* * * * *

Un autre exemple
---------------

SQL prend en charge l'utilisation de parenthèses si l'application doit vérifier des conditions particulières avant d'autres. Les expressions entre parenthèses ont priorité sur les autres opérateurs et sont évaluées en premier. Regardons un scénario comme celui-ci :

![paranthesis_fail](https://academy.hackthebox.com/storage/modules/33/paranthesis_fail.png)

La requête ci-dessus garantit que l'identifiant de l'utilisateur est toujours supérieur à 1, ce qui empêchera quiconque de se connecter en tant qu'administrateur. De plus, nous voyons également que le mot de passe a été haché avant d'être utilisé dans la requête. Cela nous empêchera d'injecter via le champ du mot de passe car l'entrée est changée en un hachage.

Essayons de nous connecter avec des informations d'identification valides `admin / p@ssw0rd` pour voir la réponse.

![paranthesis_valid_fail](https://academy.hackthebox.com/storage/modules/33/paranthesis_valid_fail.png)

Comme prévu, la connexion a échoué même si nous avons fourni des informations d'identification valides, car l'ID de l'administrateur est égal à 1. Essayons donc de nous connecter avec les informations d'identification d'un autre utilisateur, comme `tom`.

![tom_login](https://academy.hackthebox.com/storage/modules/33/tom_login.png)

La connexion en tant qu'utilisateur avec un identifiant différent de 1 a réussi. Alors, comment pouvons-nous nous connecter en tant qu'administrateur ? Nous savons de la section précédente sur les commentaires que nous pouvons les utiliser pour commenter le reste de la requête. Essayons donc d'utiliser `admin'-- `comme nom d'utilisateur.

![paranthesis_error](https://academy.hackthebox.com/storage/modules/33/paranthesis_error.png)

La connexion a échoué en raison d'une erreur de syntaxe, car une parenthèse fermée n'équilibrait pas la parenthèse ouverte. Pour exécuter la requête avec succès, nous devrons ajouter une parenthèse fermante. Essayons d'utiliser le nom d'utilisateur `admin')-- `pour fermer et commenter le reste.

![paranthesis_success](https://academy.hackthebox.com/storage/modules/33/paranthesis_success.png)

La requête a réussi et nous nous sommes connectés en tant qu'administrateur. La requête finale à la suite de notre entrée est :

Code : sql

```
SELECT * FROM connexions où (nom d'utilisateur='admin')

```

La requête ci-dessus ressemble à celle de l'exemple précédent et renvoie la ligne contenant admin.