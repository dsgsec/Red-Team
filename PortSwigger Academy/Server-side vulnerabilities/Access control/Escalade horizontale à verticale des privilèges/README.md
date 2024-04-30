Escalade horizontale à verticale des privilèges
-----------------------------------------------

Souvent, une attaque d'escalade de privilèges horizontale peut être transformée en une escalade de privilèges verticale, en compromettant un utilisateur plus privilégié. Par exemple, une escalade horizontale pourrait permettre à un attaquant de réinitialiser ou de capturer le mot de passe appartenant à un autre utilisateur. Si l'attaquant cible un utilisateur administratif et compromet son compte, il peut obtenir un accès administratif et ainsi effectuer une escalade verticale des privilèges.

Un attaquant pourrait être en mesure d'accéder à la page de compte d'un autre utilisateur en utilisant la technique de falsification de paramètres déjà décrite pour l'escalade horizontale des privilèges:

`https://insecure-website.com/myaccount?id=456`

Si l'utilisateur cible est un administrateur d'application, l'attaquant aura accès à une page de compte d'administration. Cette page peut divulguer le mot de passe de l'administrateur ou fournir un moyen de le modifier, ou peut fournir un accès direct à des fonctionnalités privilégiées.
