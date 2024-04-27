Fonctionnalité non protégée - Suite
-----------------------------------

Dans certains cas, les fonctionnalités sensibles sont masquées en lui donnant une URL moins prévisible. C'est un exemple de ce qu'on appelle "la sécurité par l'obscurité". Cependant, cacher les fonctionnalités sensibles ne fournit pas un contrôle d'accès efficace, car les utilisateurs peuvent découvrir l'URL masquée de plusieurs façons.

Imaginez une application qui héberge des fonctions administratives à l'URL suivante:

`https://insecure-website.com/administrator-panel-yb556`

Cela pourrait ne pas être directement devinable par un attaquant. Cependant, l'application peut toujours divulguer l'URL aux utilisateurs. L'URL peut être divulguée en JavaScript qui construit l'interface utilisateur en fonction du rôle de l'utilisateur:

`<script> var isAdmin = false; if (isAdmin) { ... var adminPanelTag = document.createElement('a'); adminPanelTag.setAttribute('https://insecure-website.com/administrator-panel-yb556'); adminPanelTag.innerText = 'Admin panel'; ... } </script>`

Ce script ajoute un lien vers l'interface utilisateur de l'utilisateur s'il s'agit d'un utilisateur administrateur. Cependant, le script contenant l'URL est visible par tous les utilisateurs quel que soit leur rôle.
