Fonctionnalité non protégée
---------------------------

À sa base, l'escalade verticale des privilèges se produit lorsqu'une application n'applique aucune protection pour les fonctionnalités sensibles. Par exemple, les fonctions administratives peuvent être liées à partir de la page d'accueil d'un administrateur, mais pas à partir de la page d'accueil d'un utilisateur. Cependant, un utilisateur peut être en mesure d'accéder aux fonctions administratives en naviguant vers l'URL d'administration correspondante.

Par exemple, un site Web peut héberger des fonctionnalités sensibles à l'URL suivante:

`https://insecure-website.com/admin`

Cela peut être accessible par n'importe quel utilisateur, pas seulement les utilisateurs administratifs qui ont un lien vers la fonctionnalité dans leur interface utilisateur. Dans certains cas, l'URL administrative peut être divulguée à d'autres endroits, tels que `robots.txt` fichier:

`https://insecure-website.com/robots.txt`

Même si l'URL n'est divulguée nulle part, un attaquant peut être en mesure d'utiliser une liste de mots pour forcer brutalement l'emplacement de la fonctionnalité sensible.
