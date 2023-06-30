Introduction
=============

Cette salle est une introduction aux types et aux techniques utilisées dans les attaques par mot de passe. Nous discuterons des moyens d'obtenir et de générer des listes de mots de passe personnalisées. Voici quelques-uns des sujets dont nous discuterons :

-   Profilage de mot de passe
-   Techniques d'attaques par mot de passe
-   Attaques de mot de passe en ligne

### Qu'est-ce qu'un mot de passe ?

Les mots de passe sont utilisés comme méthode d'authentification pour permettre aux individus d'accéder à des systèmes informatiques ou à des applications.  L'utilisation de mots de passe garantit que le propriétaire du compte est le seul à y avoir accès. Cependant, si le mot de passe est partagé ou tombe entre de mauvaises mains, des modifications non autorisées d'un système donné peuvent se produire. Un accès non autorisé peut entraîner des modifications de l'état général et de la santé du système ou endommager le système de fichiers.  Les mots de passe sont généralement composés d'une combinaison de caractères tels que des lettres, des chiffres et des symboles. Ainsi, c'est à l'utilisateur de décider comment générer les mots de passe !

Une collection de mots de passe est souvent appelée dictionnaire ou liste de mots. Les mots de passe peu complexes et faciles à deviner se retrouvent couramment dans diverses violations de données de mot de passe divulguées publiquement. Par exemple, un mot de passe facile à deviner pourrait être  password ,  123456 ,  111111 et bien plus encore. Voici le  [top 100 des mots de passe les plus courants et les plus vus](https://techlabuzz.com/top-100-most-common-passwords/) pour votre référence. Ainsi, il ne faudra pas longtemps et il sera trop difficile pour l'attaquant d'exécuter des attaques par mot de passe contre la cible ou le service pour deviner le mot de passe. Choisir un mot de passe fort est une bonne pratique, ce qui le rend difficile à deviner ou à déchiffrer. Les mots de passe forts ne doivent pas être des mots courants ni trouvés dans les dictionnaires, et le mot de passe doit comporter au moins huit caractères. Il doit également contenir des lettres majuscules et minuscules, des chiffres et des chaînes de symboles (ex :  *&^%$#@ ).

Parfois, les entreprises ont leurs propres politiques de mot de passe et obligent les utilisateurs à suivre les directives lors de la création de mots de passe. Cela permet de s'assurer que les utilisateurs n'utilisent pas de mots de passe courants ou faibles au sein de leur organisation et pourrait limiter les vecteurs d'attaque tels que le forçage brutal. Par exemple, la longueur d'un mot de passe doit être de huit caractères et plus, y compris des caractères, quelques chiffres et au moins un symbole. Cependant, si l'attaquant découvre la politique de mot de passe, il pourrait générer une liste de mots de passe qui satisfait la politique de mot de passe du compte.

### À quel point les mots de passe sont-ils sécurisés ?

Les mots de passe sont une méthode de protection pour accéder aux comptes en ligne ou aux systèmes informatiques. Les méthodes d'authentification par mots de passe sont utilisées pour accéder aux systèmes personnels et privés, et son objectif principal d'utiliser le mot de passe est de le garder en sécurité et de ne pas le partager avec d'autres.

Pour répondre à la question :  Dans quelle mesure les mots de passe sont-ils sécurisés ?  dépend de divers facteurs. Les mots de passe sont généralement stockés dans le système de fichiers ou la base de données, et il est essentiel de les garder en sécurité. Nous avons vu des cas où des entreprises stockent des mots de passe dans des documents en clair, comme la  [violation de Sony ](https://www.techdirt.com/articles/20141204/12032329332/shocking-sony-learned-no-password-lessons-after-2011-psn-hack.shtml) en 2014. Par conséquent, une fois qu'un attaquant accède au système de fichiers, il peut facilement obtenir et réutiliser ces mots de passe. D'autre part, d'autres stockent les mots de passe dans le système en utilisant diverses techniques telles que des fonctions de hachage ou des algorithmes de cryptage pour les rendre plus sûrs. Même si l'attaquant doit accéder au système, il sera plus difficile à craquer. Nous couvrirons le craquage des hachages dans les tâches à venir.
