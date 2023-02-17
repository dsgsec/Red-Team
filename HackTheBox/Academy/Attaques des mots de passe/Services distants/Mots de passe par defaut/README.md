# Mot de passe par défaut / Mot de passe réutilisé

Il est courant pour les utilisateurs et les administrateurs de laisser les valeurs par défaut en place. Les administrateurs doivent suivre l'ensemble de la technologie, de l'infrastructure et des applications, ainsi que les données auxquelles ils accèdent. Dans ce cas, le même mot de passe est souvent utilisé à des fins de configuration, puis le mot de passe est oublié pour être changé pour une interface ou une autre. De plus, de nombreuses applications qui fonctionnent avec des mécanismes d'authentification, pratiquement toutes, sont souvent fournies avec des informations d'identification par défaut après l'installation. Ces informations d'identification par défaut peuvent être oubliées pour être modifiées après la configuration, en particulier lorsqu'il s'agit d'applications internes où les administrateurs supposent que personne d'autre ne les trouvera et n'essaient même pas de les utiliser.

De plus, des mots de passe faciles à retenir qui peuvent être saisis rapidement au lieu de saisir des mots de passe longs de 15 caractères sont souvent utilisés à plusieurs reprises car l'authentification unique (SSO) n'est pas toujours immédiatement disponible lors de l'installation initiale et la configuration dans les réseaux internes nécessite changements importants. Lors de la configuration des réseaux, nous travaillons parfois avec de vastes infrastructures (selon la taille de l'entreprise) qui peuvent avoir plusieurs centaines d'interfaces. Souvent, un périphérique réseau, tel qu'un routeur, une imprimante ou un pare-feu, est ignoré et les informations d'identification par défaut sont utilisées ou le même mot de passe est réutilisé.

Bourrage d'informations d'identification
Il existe diverses bases de données qui conservent une liste courante des informations d'identification par défaut connues. L'un d'eux est le DefaultCreds-Cheat-Sheet. 

https://github.com/ihebski/DefaultCreds-cheat-sheet

## DefautCreds-cheat-sheet

Cet outil permet de lister les couples d'utilisateurs / mots de passe les plus utilisés sur un service ou application :
+ MySQL
+ SSH
+ Tomcat
+ Bizlogic
+ etc...

## Identifiants par défaut

Les informations d'identification par défaut peuvent également être trouvées dans la documentation du produit, car elles contiennent les étapes nécessaires pour configurer le service avec succès. Certains appareils/applications exigent que l'utilisateur configure un mot de passe lors de l'installation, mais d'autres utilisent un mot de passe faible par défaut. Attaquer ces services avec les informations d'identification par défaut ou obtenues s'appelle le Credential Stuffing. Il s'agit d'une variante simplifiée du brute-forcing car seuls les noms d'utilisateur composites et les mots de passe associés sont utilisés.

Nous pouvons imaginer que nous avons trouvé certaines applications utilisées dans le réseau par nos clients. Après avoir recherché sur Internet les informations d'identification par défaut, nous pouvons créer une nouvelle liste qui sépare ces informations d'identification composites par deux points (nom d'utilisateur : mot de passe). De plus, nous pouvons sélectionner les mots de passe et les faire muter selon nos règles pour augmenter la probabilité de succès.