Gestion des identifiants d'authentification
===========================================

* * * * *

Par gestion des identifiants d'authentification, nous entendons comment une application fonctionne sur les mots de passe (réinitialisation du mot de passe, récupération du mot de passe ou modification du mot de passe). Une réinitialisation de mot de passe, par exemple, pourrait être un moyen simple mais bruyant de contourner l'authentification.

En ce qui concerne les applications Web typiques, les utilisateurs qui oublient leur mot de passe peuvent en obtenir un nouveau de trois manières lorsqu'aucun facteur d'authentification externe n'est utilisé.

1.  En en demandant un nouveau qui sera envoyé par email par l'application
2.  En demandant une URL qui leur permettra d'en définir une nouvelle
3.  En répondant à des questions préremplies comme preuve d'identité, puis en en définissant une nouvelle

En tant que testeurs d'intrusion, nous devons toujours rechercher des failles logiques dans les fonctionnalités "mot de passe oublié" et "changement de mot de passe", car elles peuvent nous permettre de contourner l'authentification.
