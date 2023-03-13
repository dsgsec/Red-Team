Introduction
============

* * * * *

La plupart des applications Web modernes utilisent une structure de base de données sur le back-end. Ces bases de données sont utilisées pour stocker et récupérer des données liées à l'application Web, du contenu Web réel aux informations et contenus de l'utilisateur, etc. Pour rendre les applications Web dynamiques, l'application Web doit interagir avec la base de données en temps réel. Au fur et à mesure que les requêtes HTTP(S) arrivent de l'utilisateur, le back-end de l'application Web envoie des requêtes à la base de données pour créer la réponse. Ces requêtes peuvent inclure des informations provenant de la requête HTTP(S) ou d'autres informations pertinentes.

![dbms_architecture](https://academy.hackthebox.com/storage/modules/33/db_request_3.png)

Lorsque les informations fournies par l'utilisateur sont utilisées pour construire la requête dans la base de données, les utilisateurs malveillants peuvent tromper la requête pour qu'elle soit utilisée pour autre chose que ce que le programmeur d'origine avait prévu, fournissant à l'utilisateur un accès pour interroger la base de données à l'aide d'une attaque connue sous le nom d'injection SQL ( SQLi).

L'injection SQL fait référence aux attaques contre les bases de données relationnelles telles que `MySQL` (alors que les injections contre les bases de données non relationnelles, telles que MongoDB, sont des injections NoSQL). Ce module se concentrera sur `MySQL` pour introduire les concepts d'injection SQL.

* * * * *

Injection SQL (SQLi)
--------------------

De nombreux types de vulnérabilités d'injection sont possibles dans les applications Web, telles que l'injection HTTP, l'injection de code et l'injection de commande. L'exemple le plus courant, cependant, est l'injection SQL. Une injection SQL se produit lorsqu'un utilisateur malveillant tente de transmettre une entrée qui modifie la requête SQL finale envoyée par l'application Web à la base de données, permettant à l'utilisateur d'effectuer d'autres requêtes SQL involontaires directement sur la base de données.

Il y a plusieurs façons d'accomplir ceci. Pour qu'une injection SQL fonctionne, l'attaquant doit d'abord injecter du code SQL, puis subvertir la logique de l'application Web en modifiant la requête d'origine ou en en exécutant une entièrement nouvelle. Tout d'abord, l'attaquant doit injecter du code en dehors des limites d'entrée utilisateur attendues, afin qu'il ne soit pas exécuté comme une simple entrée utilisateur. Dans le cas le plus basique, cela se fait en injectant un guillemet simple (`'`) ou un guillemet double (`"`) pour échapper aux limites d'entrée de l'utilisateur et injecter des données directement dans la requête SQL.

Une fois qu'un attaquant peut injecter, il doit chercher un moyen d'exécuter une requête SQL différente. Cela peut être fait en utilisant du code SQL pour créer une requête de travail qui exécute à la fois la requête SQL prévue et la nouvelle requête SQL. Il existe de nombreuses façons d'y parvenir, comme l'utilisation de [stacked](https://www.sqlinjection.net/stacked-queries/) requêtes ou l'utilisation de [Union](https://www.mysqltutorial.org/sql-union- mysql.aspx/) requêtes. Enfin, pour récupérer la sortie de notre nouvelle requête, nous devons l'interpréter ou la capturer sur le front-end de l'application Web.

* * * * *

Cas d'utilisation et impact
--------------------

Une injection SQL peut avoir un impact considérable, surtout si les privilèges sur le serveur principal et la base de données sont très laxistes.

Tout d'abord, nous pouvons récupérer des informations secrètes/sensibles qui ne devraient pas nous être visibles, comme les identifiants et mots de passe des utilisateurs ou les informations de carte de crédit, qui peuvent ensuite être utilisées à d'autres fins malveillantes. Les injections SQL provoquent de nombreuses violations de mots de passe et de données contre des sites Web, qui sont ensuite réutilisés pour voler des comptes d'utilisateurs, accéder à d'autres services ou effectuer d'autres actions néfastes.

Un autre cas d'utilisation de l'injection SQL consiste à subvertir la logique d'application Web prévue. L'exemple le plus courant consiste à contourner la connexion sans transmettre une paire valide d'informations d'identification de nom d'utilisateur et de mot de passe. Un autre exemple est l'accès à des fonctionnalités verrouillées pour des utilisateurs spécifiques, comme les panneaux d'administration. Les attaquants peuvent également être en mesure de lire et d'écrire des fichiers directement sur le serveur principal, ce qui peut, à son tour, conduire à placer des portes dérobées sur le serveur principal, à en prendre le contrôle direct et, éventuellement, à prendre le contrôle de l'ensemble du serveur. site Internet.

* * * * *

La prévention
----------

Les injections SQL sont généralement causées par des applications Web mal codées ou des privilèges de serveur principal et de bases de données mal sécurisés. Plus tard, nous discuterons des moyens de réduire les risques d'être vulnérable aux injections SQL grâce à des méthodes de codage sécurisées telles que la désinfection et la validation des entrées utilisateur et les privilèges et contrôles appropriés des utilisateurs principaux.
