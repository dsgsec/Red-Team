Vulnérabilités d'authentification
---------------------------------

Conceptuellement, les vulnérabilités d'authentification sont faciles à comprendre. Cependant, ils sont généralement critiques en raison de la relation claire entre l'authentification et la sécurité.

Les vulnérabilités d'authentification peuvent permettre aux attaquants d'accéder à des données et fonctionnalités sensibles. Ils exposent également une surface d'attaque supplémentaire pour d'autres exploits. Pour cette raison, il est important d'apprendre à identifier et à exploiter les vulnérabilités d'authentification et à contourner les mesures de protection courantes.

Dans cette section, nous expliquons:

-   Les mécanismes d'authentification les plus couramment utilisés par les sites Web.
-   Vulnérabilités potentielles dans ces mécanismes.
-   Vulnérabilités inhérentes aux différents mécanismes d'authentification.
-   Vulnérabilités typiques introduites par leur mauvaise mise en œuvre.
-   Comment rendre vos propres mécanismes d'authentification aussi robustes que possible.

![Vulnérabilités d'authentification: empoisonnement par réinitialisation de mot de passe](https://portswigger.net/web-security/images/password-reset-poisoning.svg)


Quelle est la différence entre authentification et autorisation?
----------------------------------------------------------------

L'authentification est le processus de vérification qu'un utilisateur est qui il prétend être. L'autorisation consiste à vérifier si un utilisateur est autorisé à faire quelque chose.

Par exemple, l'authentification détermine si quelqu'un tente d'accéder à un site Web avec le nom d'utilisateur `Carlos123` c'est vraiment la même personne qui a créé le compte.

Une fois `Carlos123` est authentifié, leurs autorisations déterminent ce qu'ils sont autorisés à faire. Par exemple, ils peuvent être autorisés à accéder à des informations personnelles sur d'autres utilisateurs ou à effectuer des actions telles que la suppression du compte d'un autre utilisateur.
