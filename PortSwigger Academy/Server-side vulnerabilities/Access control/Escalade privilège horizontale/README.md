Escalade privilège horizontale
------------------------------

L'escalade horizontale des privilèges se produit si un utilisateur peut accéder à des ressources appartenant à un autre utilisateur, au lieu de ses propres ressources de ce type. Par exemple, si un employé peut accéder aux dossiers d'autres employés ainsi qu'aux siens, il s'agit d'une augmentation horizontale des privilèges.

Les attaques d'escalade de privilèges horizontale peuvent utiliser des types similaires de méthodes d'exploitation pour l'escalade de privilèges verticale. Par exemple, un utilisateur peut accéder à sa propre page de compte en utilisant l'URL suivante:

`https://insecure-website.com/myaccount?id=123`

Si un attaquant modifie le `id` valeur de paramètre à celle d'un autre utilisateur, ils peuvent accéder à la page de compte d'un autre utilisateur, ainsi qu'aux données et fonctions associées.

#### Note

Ceci est un exemple d'une vulnérabilité non sécurisée de référence d'objet direct (IDOR). Ce type de vulnérabilité se produit lorsque les valeurs des paramètres utilisateur-contrôleur sont utilisées pour accéder directement aux ressources ou aux fonctions.

Dans certaines applications, le paramètre exploitable n'a pas de valeur prévisible. Par exemple, au lieu d'un numéro d'incrémentation, une application peut utiliser des identificateurs uniques au monde (GUID) pour identifier les utilisateurs. Cela peut empêcher un attaquant de deviner ou de prédire l'identifiant d'un autre utilisateur. Cependant, les GUID appartenant à d'autres utilisateurs peuvent être divulgués ailleurs dans l'application où les utilisateurs sont référencés, tels que les messages utilisateur ou les avis.
