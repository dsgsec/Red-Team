Bruteforce Passwords
==============================

* * * * *

Après avoir réussi à `username enumeration`, un attaquant n'est souvent qu'à une étape de l'objectif de contourner l'authentification, et cette étape est le mot de passe de l'utilisateur. Les mots de passe sont la principale, sinon la seule, mesure de sécurité pour la plupart des applications. Malgré sa popularité, cette mesure n'est pas toujours perçue comme importante à la fois par les utilisateurs finaux et les administrateurs/mainteneurs. Par conséquent, l'attention qu'il reçoit n'est généralement pas suffisante. Wikipédia a une [page](https://en.wikipedia.org/wiki/List_of_the_most_common_passwords) qui répertorie les mots de passe les plus courants avec un classement pour chaque année à partir de 2011. Si vous jetez un coup d'œil à ce [tableau](https://en.wikipedia.org/wiki/List_of_the_most_common_passwords#SplashData) , vous pouvez voir que les gens ne sont pas si prudents.

* * * * *

Problèmes de mot de passe
-------------------------

Historiquement parlant, les mots de passe souffraient de trois problèmes importants. Le premier réside dans le nom lui-même. Très souvent, les utilisateurs pensent qu'un mot de passe peut être juste un mot et non une phrase. Le deuxième problème est que les utilisateurs définissent principalement des mots de passe faciles à retenir. Ces mots de passe sont généralement faibles ou suivent un modèle prévisible. Même si un utilisateur choisit un mot de passe plus complexe, celui-ci sera généralement écrit sur un Post-it ou enregistré en texte clair. Il n'est pas rare non plus de trouver le mot de passe écrit dans le champ d'indice. Le deuxième problème de mot de passe s'aggrave lorsqu'une exigence de rotation fréquente des mots de passe pour accéder aux réseaux d'entreprise entre en jeu. Cette exigence se traduit généralement par des mots de passe tels que `Spring2020`, `Autumn2020`ou `CompanynameTown1`, `CompanynameTown2`etc.

Récemment , [le NIST](https://pages.nist.gov/800-63-3/sp800-63b.html) , l'Institut national des normes et de la technologie, a actualisé ses directives concernant les tests de politique de mot de passe, les exigences d'âge du mot de passe et les règles de composition du mot de passe.

Le changement pertinent est :

```
Verifiers SHOULD NOT impose other composition rules (e.g., requiring mixtures of different character types or prohibiting consecutively repeated characters) for memorized secrets. Verifiers SHOULD NOT require memorized secrets to be changed arbitrarily (e.g., periodically).

```

Enfin, c'est un fait connu que de nombreux utilisateurs réutilisent le même mot de passe sur plusieurs services. Une fuite de mot de passe ou un compromis sur l'un d'entre eux permettra à un attaquant d'accéder à un large éventail de sites Web ou d'applications. Cette attaque est connue sous le nom `Credential stuffing`et va de pair avec la génération de listes de mots, enseignée dans le [module Cracking Passwords with Hashcat](https://academy.hackthebox.com/course/preview/cracking-passwords-with-hashcat) . Les gestionnaires de mots de passe sont une solution viable pour stocker et utiliser des mots de passe complexes. Parfois, vous pouvez rencontrer des exigences de mot de passe faibles. Cela se produit généralement lorsque des mesures de sécurité supplémentaires sont en place. Les guichets automatiques en sont un excellent exemple. Le mot de passe, ou mieux le code PIN, est une simple séquence de 4 ou 5 chiffres. Assez faible, mais le manque de complexité est contrebalancé par une limitation du nombre total de tentatives (pas plus de 3 codes PIN avant de perdre l'accès physique à l'appareil).

* * * * *

Inférence de politique
----------------------

Les chances d'exécuter une attaque par force brute réussie augmentent après une évaluation appropriée de la stratégie. Connaître les exigences minimales en matière de mot de passe permet à un attaquant de commencer à tester uniquement les mots de passe conformes. Une application Web qui implémente une politique de mot de passe fort pourrait rendre une attaque par force brute presque impossible. En tant que développeur, choisissez toujours des mots de passe longs plutôt que des mots de passe courts mais complexes. Sur pratiquement toutes les applications qui permettent l'auto-enregistrement, il est possible de déduire la politique de mot de passe en enregistrant un nouvel utilisateur. Essayer d'utiliser le nom d'utilisateur comme mot de passe, ou un mot de passe très faible comme `123456`, entraîne souvent une erreur qui révélera la politique (ou certaines parties de celle-ci) dans un format lisible par l'homme.

![](https://academy.hackthebox.com/storage/modules/80/07-password_policy_exposed-small.png)

Les exigences de la politique définissent le nombre de familles de caractères différentes nécessaires et la longueur du mot de passe lui-même.

Les familles sont :

-   caractères minuscules, comme`abcd..z`

-   caractères majuscules, comme`ABCD..Z`

-   chiffre, nombres de`0 to 9`

-   caractères spéciaux, comme `,./.?!`ou tout autre caractère imprimable ( `space`est un caractère !)

Il est possible qu'une application réponde `Password does not meet complexity requirements`d'abord par un message et révèle les conditions exactes de la police après un certain nombre d'échecs d'enregistrement. C'est pourquoi il est recommandé de tester trois ou quatre fois avant d'abandonner.

La même attaque pourrait être menée sur une page de réinitialisation de mot de passe. Lorsqu'un utilisateur peut réinitialiser son mot de passe, le formulaire de réinitialisation peut divulguer la politique de mot de passe (ou des parties de celle-ci). Lors d'un engagement réel, le processus d'inférence pourrait être un jeu de devinettes. Puisqu'il s'agit d'une étape critique, nous vous fournissons un autre exemple de base. Ayant une application Web qui nous permet d'enregistrer un nouveau compte, nous essayons d'utiliser `123456`comme mot de passe pour identifier la politique. L'application Web répond par un `Password does not match minimum requirements`message. Une politique est évidemment en place, mais elle n'est pas divulguée.

![](https://academy.hackthebox.com/storage/modules/80/password_policy_not_exposed.png)

Nous commençons ensuite à deviner les exigences en enregistrant un compte et en entrant une séquence de marche au clavier pour le mot de passe comme `Qwertyiop123!@#`, qui est en fait prévisible mais suffisamment longue et complexe pour correspondre aux politiques standard.

Supposons que l'application Web accepte ces mots de passe comme valides. Réduisons maintenant la complexité en supprimant les caractères spéciaux, puis les chiffres, puis les majuscules, et en diminuant la longueur d'un caractère à la fois. Plus précisément, nous essayons d'enregistrer un nouvel utilisateur en utilisant `Qwertyiop123`, puis `Qwertyiop!@#`, puis `qwertyiop123`, et ainsi de suite jusqu'à ce que nous ayons une matrice avec les exigences minimales. Lorsque vous testez des applications Web, gardez également à l'esprit que certaines limitent également la longueur du mot de passe en obligeant les utilisateurs à avoir un mot de passe entre 8 et 15 caractères. Ce processus est sujet aux erreurs et il est également possible que certaines combinaisons ne soient pas testées alors que d'autres le seront deux fois. Pour cette raison, il est recommandé d'utiliser un tableau comme celui-ci pour garder une trace de nos tests :

| Essayé | Mot de passe | Inférieur | Supérieur | Chiffre | Spécial | >=8caractères | >=20caractères |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Oui Non | `qwerty` | X |  |  |  |  |  |
| Oui Non | `Qwerty` | X | X |  |  |  |  |
| Oui Non | `Qwerty1` | X | X | X |  |  |  |
| Oui Non | `Qwertyu1` | X | X | X |  | X |  |
| Oui Non | `Qwert1!` | X | X | X | X |  |  |
| Oui Non | `Qwerty1!` | X | X | X | X | X |  |
| Oui Non | `QWERTY1` |  | X | X |  |  |  |
| Oui Non | `QWERT1!` |  | X | X | X |  |  |
| Oui Non | `QWERTY1!` |  | X | X | X | X |  |
| Oui Non | `Qwerty!` | X | X |  | X |  |  |
| Oui Non | `Qwertyuiop12345!@#$%` | X | X | X | X | X | X |

En quelques essais, nous devrions être en mesure de déduire la politique même si le message est générique. Supposons maintenant que cette application web nécessite une chaîne entre 8 et 12 caractères, avec au moins un caractère majuscule et minuscule. Nous prenons maintenant une liste de mots géante et extrayons uniquement les mots de passe qui correspondent à cette politique. Unix `grep`n'est pas l'outil le plus rapide mais nous permet de faire le travail rapidement en utilisant des expressions régulières POSIX. La commande ci-dessous fonctionnera contre rockyou-50.txt, un sous-ensemble de la `rockyou`fuite de mot de passe bien connue présente dans les SecLists. Cette commande trouve les lignes ayant au moins un caractère majuscule ( `'[[:upper:]]'`), puis uniquement les lignes qui ont également un caractère minuscule ( `'[[:lower:]]'`) et d'une longueur de 8 et 12 caractères ('^.{8,12}$') en utilisant des expressions régulières étendues (-E).

```
dsgsec@htb[/htb]$ grep '[[:upper:]]' rockyou.txt | grep '[[:lower:]]' | grep -E '^.{8,12}$'

416712

```

Nous voyons qu'à partir de la norme `rockyou.txt`, qui contient plus de 14 millions de lignes, nous l'avons réduite à environ 400 000. Si vous voulez vous entraîner, téléchargez le script PHP [ici](https://academy.hackthebox.com/storage/modules/80/password_policy_php.txt) et essayez de respecter la politique. Nous vous suggérons de garder le tableau que nous venons de fournir à portée de main pour cet exercice.

* * * * *

Effectuez une attaque par force brute réelle
--------------------------------------------

Maintenant que nous avons un nom d'utilisateur, que nous connaissons la politique de mot de passe et les mesures de sécurité en place, nous pouvons commencer à forcer brutalement l'application Web. N'oubliez pas que vous devez également vérifier si un jeton anti-CSRF protège le formulaire et modifier votre script pour envoyer un tel jeton.
