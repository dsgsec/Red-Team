IDOR dans les API non sécurisées
================================

* * * * *

Jusqu'à présent, nous n'avons utilisé les vulnérabilités IDOR que pour accéder aux fichiers et aux ressources auxquels nos utilisateurs n'ont pas accès. Cependant, des vulnérabilités IDOR peuvent également exister dans les appels de fonction et les API, et les exploiter nous permettrait d'effectuer diverses actions comme les autres utilisateurs.

Tout en `IDOR Information Disclosure Vulnerabilities`nous permettant de lire différents types de ressources, `IDOR Insecure Function Calls`nous permet d'appeler des API ou d'exécuter des fonctions en tant qu'autre utilisateur. Ces fonctions et API peuvent être utilisées pour modifier les informations privées d'un autre utilisateur, réinitialiser le mot de passe d'un autre utilisateur ou même acheter des articles en utilisant les informations de paiement d'un autre utilisateur. Dans de nombreux cas, nous pouvons obtenir certaines informations via une vulnérabilité IDOR de divulgation d'informations, puis utiliser ces informations avec des vulnérabilités d'appel de fonction non sécurisées IDOR, comme nous le verrons plus loin dans le module.

* * * * *

Identification des API non sécurisées
-------------------------------------

Pour en revenir à notre `Employee Manager`application Web, nous pouvons commencer à tester la `Edit Profile`page pour les vulnérabilités IDOR :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_employee_manager.jpg)

Lorsque nous cliquons sur le `Edit Profile`bouton, nous sommes redirigés vers une page pour modifier les informations de notre profil d'utilisateur, à savoir `Full Name`, `Email`, et `About Me`, qui est une caractéristique commune à de nombreuses applications Web :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_edit_profile.jpg)

Nous pouvons modifier n'importe lequel des détails de notre profil et cliquer sur `Update profile`, et nous verrons qu'ils sont mis à jour et persistent grâce aux actualisations, ce qui signifie qu'ils sont mis à jour dans une base de données quelque part. Interceptons la `Update`requête dans Burp et regardons-la :

![update_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_update_request.jpg)

Nous voyons que la page envoie une `PUT`requête au `/profile/api.php/profile/1`point de terminaison de l'API. `PUT`Les requêtes sont généralement utilisées dans les API pour mettre à jour les détails des éléments, tandis qu'elles `POST`sont utilisées pour créer de nouveaux éléments, `DELETE`supprimer des éléments et `GET`récupérer les détails des éléments. Ainsi, une `PUT`requête pour la `Update profile`fonction est attendue. Le bit intéressant est les paramètres JSON qu'il envoie :

Code: json

```
{
    "uid": 1,
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "employee",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}

```

Nous voyons que la `PUT`requête inclut quelques paramètres cachés, comme `uid`, `uuid`, et surtout `role`, qui est défini sur `employee`. L'application Web semble également définir les privilèges d'accès de l'utilisateur (par exemple `role`) côté client, sous la forme de notre `Cookie: role=employee`cookie, qui semble refléter ce qui `role`est spécifié pour notre utilisateur. Il s'agit d'un problème de sécurité courant. Les privilèges de contrôle d'accès sont envoyés dans le cadre de la requête HTTP du client, soit sous forme de cookie, soit dans le cadre de la requête JSON, le laissant sous le contrôle du client, qui pourrait être manipulé pour obtenir plus de privilèges.

Ainsi, à moins que l'application Web ne dispose d'un système de contrôle d'accès solide sur le back-end, `we should be able to set an arbitrary role for our user, which may grant us more privileges`. Cependant, comment saurions-nous quels autres rôles existent ?

* * * * *

Exploitation d'API non sécurisées
---------------------------------

Nous savons que nous pouvons modifier les paramètres `full_name`, `email`et `about`, car ce sont ceux sous notre contrôle dans le formulaire HTML de la `/profile`page Web. Essayons donc de manipuler les autres paramètres.

Il y a plusieurs choses que nous pourrions essayer dans ce cas :

1.  Remplacez notre `uid`par celui d'un autre utilisateur `uid`, de sorte que nous puissions prendre en charge leurs comptes
2.  Modifier les détails d'un autre utilisateur, ce qui peut nous permettre d'effectuer plusieurs attaques Web
3.  Créez de nouveaux utilisateurs avec des détails arbitraires ou supprimez des utilisateurs existants
4.  Changer notre rôle en un rôle plus privilégié (par exemple `admin`) pour pouvoir effectuer plus d'actions

Commençons par remplacer notre `uid`par celui d'un autre utilisateur `uid`(par exemple `"uid": 2`). Cependant, tout nombre que nous définissons autre que le nôtre `uid`nous donne une réponse de`uid mismatch` :

![uid_mismatch](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_uid_mismatch.jpg)

L'application Web semble comparer la demande `uid`au point de terminaison de l'API ( `/1`). Cela signifie qu'une forme de contrôle d'accès sur le back-end nous empêche de modifier arbitrairement certains paramètres JSON, ce qui pourrait être nécessaire pour empêcher l'application Web de planter ou de renvoyer des erreurs.

Peut-être pouvons-nous essayer de modifier les détails d'un autre utilisateur. Nous allons changer le point de terminaison de l'API en `/profile/api.php/profile/2`, et changer `"uid": 2`pour éviter le précédent `uid mismatch`:

![uuid_mismatch](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_uuid_mismatch.jpg)

Comme nous pouvons le voir, cette fois, nous recevons un message d'erreur indiquant `uuid mismatch`. L'application Web semble vérifier si la `uuid`valeur que nous envoyons correspond à celle de l'utilisateur `uuid`. Puisque nous envoyons le nôtre `uuid`, notre demande échoue. Cela semble être une autre forme de contrôle d'accès pour empêcher les utilisateurs de modifier les détails d'un autre utilisateur.

Voyons ensuite si nous pouvons créer un nouvel utilisateur avec une `POST`requête au point de terminaison de l'API. Nous pouvons changer la méthode de la requête en `POST`, changer le `uid`en un new `uid`et envoyer la requête au point de terminaison API du new `uid`:

![créer_nouvel_utilisateur_1](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_create_new_user_1.jpg)

Nous recevons un message d'erreur indiquant `Creating new employees is for admins only`. La même chose se produit lorsque nous envoyons une `Delete`requête, car nous obtenons `Deleting employees is for admins only`. L'application Web peut vérifier notre autorisation via le `role=employee`cookie, car cela semble être la seule forme d'autorisation dans la requête HTTP.

Enfin, essayons de changer notre `role`en `admin`/ `administrator`pour obtenir des privilèges plus élevés. Malheureusement, sans connaître de `role`nom valide, nous obtenons `Invalid role`dans la réponse HTTP, et notre `role`ne se met pas à jour : ![invalid_role](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_invalid_role.jpg)

Alors, `all of our attempts appear to have failed`. Nous ne pouvons pas créer ou supprimer des utilisateurs car nous ne pouvons pas modifier nos fichiers `role`. Nous ne pouvons pas changer les nôtres `uid`, car il existe des mesures préventives sur le back-end que nous ne pouvons pas contrôler, et nous ne pouvons pas non plus modifier les détails d'un autre utilisateur pour la même raison. `So, is the web application secure against IDOR attacks?`.

Jusqu'à présent, nous n'avons testé que le `IDOR Insecure Function Calls`. Cependant, nous n'avons pas testé la `GET`demande de l'API pour `IDOR Information Disclosure Vulnerabilities`. S'il n'y avait pas de système de contrôle d'accès robuste en place, nous pourrions être en mesure de lire les détails des autres utilisateurs, ce qui pourrait nous aider avec les attaques précédentes que nous avons tentées.

`Try to test the API against IDOR Information Disclosure vulnerabilities by attempting to get other users' details with GET requests`. Si l'API est vulnérable, nous pouvons être en mesure de divulguer les détails d'autres utilisateurs, puis d'utiliser ces informations pour mener à bien nos attaques IDOR sur les appels de fonction.
