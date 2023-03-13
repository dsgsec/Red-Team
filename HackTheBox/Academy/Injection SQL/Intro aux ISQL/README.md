introduction aux injections SQL
======================

* * * * *

Maintenant que nous avons une idée générale du fonctionnement des requêtes MySQL et SQL, découvrons les injections SQL.

* * * * *

Utilisation de SQL dans les applications Web
------------------------------

Tout d'abord, voyons comment les applications Web utilisent les bases de données MySQL, dans ce cas, pour stocker et récupérer des données. Une fois qu'un SGBD est installé et configuré sur le serveur principal et qu'il est opérationnel, les applications Web peuvent commencer à l'utiliser pour stocker et récupérer des données.

Par exemple, dans une application Web `PHP` , nous pouvons nous connecter à notre base de données et commencer à utiliser la base de données `MySQL` via la syntaxe `MySQL` , directement dans `PHP`, comme suit :

Code : php

```
$conn = new mysqli("localhost", "root", "password", "users");
$query = "select * from logins" ;
$résultat = $conn->query($query);

```

Ensuite, la sortie de la requête sera stockée dans `$result`, et nous pourrons l'imprimer sur la page ou l'utiliser de toute autre manière. Le code PHP ci-dessous imprimera tous les résultats renvoyés de la requête SQL dans de nouvelles lignes :

Code : php

```
tandis que($ligne = $result->fetch_assoc() ){
echo $ligne["nom"]."<br>" ;
}

```

Les applications Web utilisent également généralement l'entrée de l'utilisateur lors de la récupération des données. Par exemple, lorsqu'un utilisateur utilise la fonction de recherche pour rechercher d'autres utilisateurs, son entrée de recherche est transmise à l'application Web, qui utilise l'entrée pour effectuer une recherche dans les bases de données :

Code : php

```
$searchInput = $_POST['findUser'] ;
$query = "select * from logins where username like '%$searchInput'" ;
$résultat = $conn->query($query);

```

"Si nous utilisons une entrée utilisateur dans une requête SQL, et si elle n'est pas codée de manière sécurisée, cela peut entraîner divers problèmes, tels que des vulnérabilités d'injection SQL."

* * * * *

Qu'est-ce qu'une injection ?
---------------------

Dans l'exemple ci-dessus, nous acceptons les entrées de l'utilisateur et les transmettons directement à la requête SQL sans nettoyage.

La désinfection fait référence à la suppression de tous les caractères spéciaux dans l'entrée de l'utilisateur, afin d'interrompre toute tentative d'injection.

L'injection se produit lorsqu'une application interprète à tort l'entrée de l'utilisateur comme du code réel plutôt que comme une chaîne, modifiant le flux de code et l'exécutant. Cela peut se produire en échappant aux limites d'entrée de l'utilisateur en injectant un caractère spécial comme (`'`), puis en écrivant le code à exécuter, comme le code JavaScript ou SQL dans les injections SQL. À moins que l'entrée utilisateur ne soit filtrée, il est très probable qu'elle exécute le code injecté et l'exécute.

* * * * *

Injection SQL
--------------

Une injection SQL se produit lorsque l'entrée de l'utilisateur est entrée dans la chaîne de requête SQL sans nettoyer ou filtrer correctement l'entrée. L'exemple précédent a montré comment l'entrée utilisateur peut être utilisée dans une requête SQL, et il n'a utilisé aucune forme de nettoyage des entrées :

Code : php

```
$searchInput = $_POST['findUser'] ;
$query = "select * from logins where username like '%$searchInput'" ;
$résultat = $conn->query($query);

```

Dans des cas typiques, le `searchInput` serait entré pour terminer la requête, renvoyant le résultat attendu. Toute entrée que nous tapons va dans la requête SQL suivante :

Code : sql

```
sélectionnez * parmi les connexions où le nom d'utilisateur ressemble à '%$searchInput'

```

Donc, si nous saisissons `admin`, cela devient `'%admin'`. Dans ce cas, si nous écrivons du code SQL, il serait simplement considéré comme un terme de recherche. Par exemple, si nous saisissons `SHOW DATABASES;`, il sera exécuté comme `'%SHOW DATABASES;'` L'application Web recherchera des noms d'utilisateur similaires à `SHOW DATABASES;`. Cependant, comme il n'y a pas de nettoyage, dans ce cas, nous pouvons ajouter un guillemet simple (`'`), qui terminera le champ de saisie de l'utilisateur, et après cela, nous pourrons écrire du code SQL réel. Par exemple, si nous recherchons `1 ; DROP TABLE users;`, l'entrée de recherche serait :

Code : php

```
'%1'; Utilisateurs DROP TABLE ;'

```

Remarquez comment nous avons ajouté un guillemet simple (') après "1", afin d'échapper aux limites de l'entrée utilisateur dans ('%$searchInput').

Ainsi, la requête SQL finale exécutée serait la suivante :

Code : sql

```
sélectionnez * parmi les connexions où le nom d'utilisateur est comme '%1' ; Utilisateurs DROP TABLE ;'

```

Comme nous pouvons le voir à partir de la coloration syntaxique, nous pouvons échapper aux limites de la requête d'origine et exécuter également notre requête nouvellement injectée. `Une fois la requête exécutée, la table `users` sera supprimée.`

Remarque : Dans l'exemple ci-dessus, par souci de simplicité, nous avons ajouté une autre requête SQL après un point-virgule (;). Bien que ce ne soit pas possible avec MySQL, c'est possible avec MSSQL et PostgreSQL. Dans les sections à venir, nous aborderons les véritables méthodes d'injection de requêtes SQL dans MySQL.

* * * * *

Erreurs de syntaxe
--------------

L'exemple précédent d'injection SQL renverrait une erreur :

Code : php

```
Erreur : près de la ligne 1 : près de "'" : erreur de syntaxe

```

Cela est dû au dernier caractère de fin, où nous avons un seul guillemet supplémentaire (`'`) qui n'est pas fermé, ce qui provoque une erreur de syntaxe SQL lors de l'exécution :

Code : sql

```
sélectionnez * parmi les connexions où le nom d'utilisateur est comme '%1' ; Utilisateurs DROP TABLE ;'

```

Dans ce cas, nous n'avions qu'un seul caractère de fin, car notre entrée de la requête de recherche était proche de la fin de la requête SQL. Cependant, l'entrée de l'utilisateur va généralement au milieu de la requête SQL, et le reste de la ouLa requête SQL initiale vient après.

Pour réussir l'injection, nous devons nous assurer que la requête SQL nouvellement modifiée est toujours valide et ne contient aucune erreur de syntaxe après notre injection. Dans la plupart des cas, nous n'aurions pas accès au code source pour trouver la requête SQL d'origine et développer une injection SQL appropriée pour créer une requête SQL valide. Alors, comment serions-nous capables d'injecter dans la requête SQL avec succès ?

Une réponse consiste à utiliser `commentaires`, et nous en discuterons dans une section ultérieure. Une autre consiste à faire fonctionner la syntaxe de la requête en passant plusieurs guillemets simples, comme nous le verrons ensuite (`'`).

Maintenant que nous comprenons les bases des injections SQL, commençons à apprendre quelques utilisations pratiques.

* * * * *

Types d'injections SQL
------------------------

Les injections SQL sont classées en fonction de la manière et de l'endroit où nous récupérons leur sortie.

![dbms_architecture](https://academy.hackthebox.com/storage/modules/33/types_of_sqli.jpg)

Dans des cas simples, la sortie de la requête prévue et de la nouvelle requête peut être imprimée directement sur le front-end, et nous pouvons la lire directement. C'est ce qu'on appelle l'injection SQL ` in-band` et il existe deux types : `Basé sur l'union` et `Basé sur l'erreur`.

Avec l'injection SQL `Union Based` , nous devrons peut-être spécifier l'emplacement exact, "c'est-à-dire la colonne", que nous pouvons lire, afin que la requête dirige la sortie à imprimer là-bas. Quant à l'injection SQL `Basée sur les erreurs` , elle est utilisée lorsque nous pouvons obtenir les erreurs `PHP` ou `SQL` dans le front-end, et nous pouvons donc intentionnellement provoquer une erreur SQL qui renvoie la sortie de notre requête.

Dans des cas plus compliqués, nous pouvons ne pas obtenir la sortie imprimée, nous pouvons donc utiliser la logique SQL pour récupérer la sortie caractère par caractère. C'est ce qu'on appelle l'injection SQL `aveugle` et il existe également deux types : `Boolean Based` et `Time Based`.

Avec l'injection SQL `Boolean Based` , nous pouvons utiliser des instructions conditionnelles SQL pour contrôler si la page renvoie une sortie, "c'est-à-dire la réponse à la requête d'origine", si notre instruction conditionnelle renvoie `true`. Comme pour les injections SQL basées sur le temps, nous utilisons des instructions conditionnelles SQL qui retardent la réponse de la page si l'instruction conditionnelle renvoie `true` à l'aide de la fonction `Sleep()` .

Enfin, dans certains cas, nous n'avons peut-être pas un accès direct à la sortie, nous devrons donc peut-être diriger la sortie vers un emplacement distant, "c'est-à-dire un enregistrement DNS", puis tenter de la récupérer à partir de là. C'est ce qu'on appelle l'injection SQL "hors bande".

Dans ce module, nous nous concentrerons uniquement sur l'introduction des injections SQL en découvrant l'injection SQL basée sur l'union.