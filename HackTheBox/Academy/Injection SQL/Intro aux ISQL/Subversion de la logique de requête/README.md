Subversion de la logique de requête
======================

* * * * *

Maintenant que nous avons une idée de base du fonctionnement des instructions SQL, commençons par l'injection SQL. Avant de commencer à exécuter des requêtes SQL entières, nous allons d'abord apprendre à modifier la requête d'origine en injectant l'opérateur `OR` et en utilisant des commentaires SQL pour renverser la logique de la requête d'origine. Un exemple basique de ceci est le contournement de l'authentification Web, que nous allons démontrer dans cette section.

* * * * *

Contournement de l'authentification
---------------------

Considérez la page de connexion administrateur suivante.

![admin_panel](https://academy.hackthebox.com/storage/modules/33/admin_panel.png)

Nous pouvons nous connecter avec les informations d'identification de l'administrateur `admin / p@ssw0rd`.

![admin_creds](https://academy.hackthebox.com/storage/modules/33/admin_creds.png)

La page affiche également la requête SQL en cours d'exécution pour mieux comprendre comment nous allons renverser la logique de la requête. Notre objectif est de vous connecter en tant qu'utilisateur administrateur sans utiliser le mot de passe existant. Comme nous pouvons le voir, la requête SQL en cours d'exécution est :

Code : sql

```
SELECT * FROM logins WHERE username='admin' AND password = 'p@ssw0rd';

```

La page enregistre les informations d'identification, puis utilise l'opérateur `AND` pour sélectionner les enregistrements correspondant au nom d'utilisateur et au mot de passe indiqués. Si la base de données `MySQL` renvoie des enregistrements correspondants, les informations d'identification sont valides, de sorte que le code `PHP` évaluera la condition de tentative de connexion comme `true`. Si la condition est évaluée à "true", l'enregistrement d'administrateur est renvoyé et notre connexion est validée. Voyons ce qui se passe lorsque nous saisissons des informations d'identification incorrectes.

![admin_incorrect](https://academy.hackthebox.com/storage/modules/33/admin_incorrect.png)

Comme prévu, la connexion a échoué en raison d'un mot de passe erroné, ce qui a entraîné un `faux` résultat de l'opération `ET` .

* * * * *

Découverte SQLi
--------------

Avant de commencer à subvertir la logique de l'application Web et à tenter de contourner l'authentification, nous devons d'abord tester si le formulaire de connexion est vulnérable à l'injection SQL. Pour ce faire, nous allons essayer d'ajouter l'une des charges utiles ci-dessous après notre nom d'utilisateur et voir si cela provoque des erreurs ou modifie le comportement de la page :

| Charge utile | URL encodée |
| --- | --- |
| `'` | `%27` |
| `"` | `%22` |
| `#` | `%23` |
| `;` | `%3B` |
| `)` | `%29` |

Remarque : Dans certains cas, nous devrons peut-être utiliser la version encodée en URL de la charge utile. Un exemple de ceci est lorsque nous mettons notre charge utile directement dans l'URL 'c'est-à-dire. Requête HTTP GET'.

Alors, commençons par injecter un seul guillemet :

![quote_error](https://academy.hackthebox.com/storage/modules/33/quote_error.png)

Nous constatons qu'une erreur SQL a été renvoyée au lieu du message `Échec de la connexion` . La page a généré une erreur car la requête résultante était :

Code : sql

```
SELECT * FROM logins WHERE username=''' AND password = 'quelque chose';

```

Comme indiqué dans la section précédente, la citation que nous avons saisie a entraîné un nombre impair de citations, provoquant une erreur de syntaxe. Une option serait de commenter le reste de la requête et d'écrire le reste de la requête dans le cadre de notre injection pour former une requête de travail. Une autre option consiste à utiliser un nombre pair de guillemets dans notre requête injectée, de sorte que la requête finale fonctionnerait toujours.

* * * * *

OU Injection
------------

Nous aurions besoin que la requête renvoie toujours `true`, quels que soient le nom d'utilisateur et le mot de passe saisis, pour contourner l'authentification. Pour ce faire, nous pouvons abuser de l'opérateur `OR` dans notre injection SQL.

Comme indiqué précédemment, la documentation MySQL pour [operation precedence](https://dev.mysql.com/doc/refman/8.0/en/operator-precedence.html) indique que l'opérateur `AND` sera évalué avant le ` OR` opérateur. Cela signifie que s'il y a au moins une condition `TRUE` dans la requête entière avec un opérateur `OR` , la requête entière sera évaluée comme `TRUE` puisque l'opérateur `OR` renvoie `TRUE` si l'un de ses opérandes est "VRAI".

Un exemple de condition qui renverra toujours `true` est `'1'='1'`. Cependant, pour que la requête SQL continue de fonctionner et conserve un nombre pair de guillemets, au lieu d'utiliser ('1'='1'), nous supprimerons le dernier guillemet et utiliserons ('1'='1), de sorte que le seul restant citation de la requête d'origine serait à sa place.

Ainsi, si nous injectons la condition ci-dessous et que nous avons un opérateur `OR` entre celle-ci et la condition d'origine, elle doit toujours renvoyer `true` :

Code : sql

```
administrateur' ou '1'='1

```

La requête finale devrait être la suivante :

Code : sql

```
SELECT * FROM logins WHERE username='admin' or '1'='1' AND password = 'something';

```

Cela signifie ce qui suit :

- Si le nom d'utilisateur est `admin`\
     `OU`
- Si `1=1` renvoie `true` 'qui renvoie toujours `true`'\
     `ET`
- Si le mot de passe est `quelque chose`

![or_inject_diagram](https://academy.hackthebox.com/storage/modules/33/or_inject_diagram.png)

L'opérateur `AND` sera évalué en premier et renverra `false`. Ensuite, l'opérateur `OR` serait évalué, et si l'une des déclarations est `true`, il renverrait `true`. Étant donné que `1=1` renvoie toujours `true`, cette requête renverra `true` et nous accordera l'accès.

Remarque : La charge utile que nous avons utilisée above est l'une des nombreuses charges utiles de contournement d'authentification que nous pouvons utiliser pour subvertir la logique d'authentification. Vous pouvez trouver une liste complète des charges utiles de contournement d'authentification SQLi dans [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass), chacune fonctionnant sur un certain type de Requêtes SQL.

* * * * *

Contournement d'authentification avec opérateur OR
-------------------------------------

Essayons ceci comme nom d'utilisateur et voyons la réponse. ![inject_success](https://academy.hackthebox.com/storage/modules/33/inject_success.png)

Nous avons pu nous connecter avec succès en tant qu'administrateur. Cependant, que se passe-t-il si nous ne connaissons pas un nom d'utilisateur valide ? Essayons la même requête avec un nom d'utilisateur différent cette fois.

![notadmin_fail](https://academy.hackthebox.com/storage/modules/33/notadmin_fail.png)

La connexion a échoué, car `notAdmin` n'existe pas dans le tableau et a généré une fausse requête dans l'ensemble.

![notadmin_diagram](https://academy.hackthebox.com/storage/modules/33/notadmin_diagram_1.png)

Pour réussir à vous reconnecter, nous aurons besoin d'une requête globale "true". Ceci peut être réalisé en injectant une condition `OR` dans le champ du mot de passe, de sorte qu'il renverra toujours `true`. Essayons `something' ou '1'='1` comme mot de passe.

![password_or_injection](https://academy.hackthebox.com/storage/modules/33/password_or_injection.png)

La condition `OR` supplémentaire a généré une requête `true` dans l'ensemble, car la clause `WHERE` renvoie tout dans le tableau et l'utilisateur présent dans la première ligne est connecté. Dans ce cas, les deux conditions renverront `true `, nous n'avons pas besoin de fournir un nom d'utilisateur et un mot de passe de test et pouvons commencer directement par l'injection `'` et nous connecter avec seulement `' ou '1' = '1`.

![basic_auth_bypass](https://academy.hackthebox.com/storage/modules/33/basic_auth_bypass.png)

Cela fonctionne puisque la requête est évaluée à `true` quel que soit le nom d'utilisateur ou le mot de passe.