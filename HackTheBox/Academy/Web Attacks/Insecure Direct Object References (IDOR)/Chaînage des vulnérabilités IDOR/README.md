Chaînage des vulnérabilités IDOR
================================

* * * * *

Habituellement, une `GET`demande au point de terminaison de l'API doit renvoyer les détails de l'utilisateur demandé, nous pouvons donc essayer de l'appeler pour voir si nous pouvons récupérer les détails de notre utilisateur. Nous remarquons également qu'après le chargement de la page, elle récupère les détails de l'utilisateur avec une `GET`requête au même point de terminaison API : ![get_api](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_get_api.jpg)

Comme mentionné dans la section précédente, la seule forme d'autorisation dans nos requêtes HTTP est le `role=employee`cookie, car la requête HTTP ne contient aucune autre forme d'autorisation spécifique à l'utilisateur, comme un jeton JWT, par exemple. Même si un jeton existait, à moins qu'il ne soit activement comparé aux détails de l'objet demandé par un système de contrôle d'accès principal, nous pouvons toujours être en mesure de récupérer les détails d'autres utilisateurs.

* * * * *

Divulgation d'information
-------------------------

Envoyons une `GET`requête avec une autre `uid`:

![get_another_user](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_get_another_user.jpg)

Comme nous pouvons le voir, cela a renvoyé les détails d'un autre utilisateur, avec les leurs `uuid`et `role`, confirmant un `IDOR Information Disclosure vulnerability`:

Code: json

```
{
    "uid": "2",
    "uuid": "4a9bd19b3b8676199592a346051f950c",
    "role": "employee",
    "full_name": "Iona Franklyn",
    "email": "i_franklyn@employees.htb",
    "about": "It takes 20 years to build a reputation and few minutes of cyber-incident to ruin it."
}

```

Cela nous fournit de nouveaux détails, notamment le `uuid`, que nous ne pouvions pas calculer auparavant, et ne pouvions donc pas modifier les détails des autres utilisateurs.

* * * * *

Modification des détails d'autres utilisateurs
----------------------------------------------

Maintenant, avec l'utilisateur `uuid`à portée de main, nous pouvons modifier les détails de cet utilisateur en envoyant une `PUT`demande à `/profile/api.php/profile/2`avec les détails ci-dessus ainsi que toutes les modifications que nous avons apportées, comme suit :

![modifier_un_autre_utilisateur](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_modify_another_user.jpg)

Nous ne recevons pas de message d'erreur de contrôle d'accès cette fois-ci, et lorsque nous essayons `GET`à nouveau d'accéder aux détails de l'utilisateur, nous constatons que nous avons effectivement mis à jour ses détails :

![new_another_user_details](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_new_another_user_details.jpg)

En plus de nous permettre d'afficher des détails potentiellement sensibles, la possibilité de modifier les détails d'un autre utilisateur nous permet également d'effectuer plusieurs autres attaques. Un type d'attaque consiste `modifying a user's email address`à demander ensuite un lien de réinitialisation du mot de passe, qui sera envoyé à l'adresse e-mail que nous avons spécifiée, nous permettant ainsi de prendre le contrôle de leur compte. Une autre attaque potentielle est `placing an XSS payload in the 'about' field`, qui serait exécutée une fois que l'utilisateur visite sa `Edit profile`page, nous permettant d'attaquer l'utilisateur de différentes manières.

* * * * *

Chaînage de deux vulnérabilités IDOR
------------------------------------

Puisque nous avons identifié une vulnérabilité IDOR Information Disclosure, nous pouvons également énumérer tous les utilisateurs et rechercher d'autres `roles`, idéalement un rôle d'administrateur. `Try to write a script to enumerate all users, similarly to what we did previously`.

Une fois que nous aurons énuméré tous les utilisateurs, nous trouverons un utilisateur administrateur avec les détails suivants :

Code: json

```
{
    "uid": "X",
    "uuid": "a36fa9e66e85f2dd6f5e13cad45248ae",
    "role": "web_admin",
    "full_name": "administrator",
    "email": "webadmin@employees.htb",
    "about": "HTB{FLAG}"
}

```

Nous pouvons modifier les détails de l'administrateur, puis effectuer l'une des attaques ci-dessus pour prendre le contrôle de son compte. Cependant, comme nous connaissons maintenant le nom du rôle d'administrateur ( `web_admin`), nous pouvons le définir sur notre utilisateur afin de pouvoir créer de nouveaux utilisateurs ou supprimer des utilisateurs actuels. Pour ce faire, nous intercepterons la demande lorsque nous cliquerons sur le `Update profile`bouton et changerons notre rôle en `web_admin`:

![modifier_notre_rôle](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_modify_our_role.jpg)

Cette fois, nous ne recevons pas le `Invalid role`message d'erreur, ni aucun message d'erreur de contrôle d'accès, ce qui signifie qu'il n'y a pas de mesures de contrôle d'accès back-end sur les rôles que nous pouvons définir pour notre utilisateur. Si nous nous référons `GET`à nos coordonnées utilisateur, nous voyons que notre `role`a bien été paramétré sur `web_admin`:

Code: json

```
{
    "uid": "1",
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "web_admin",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}

```

Maintenant, nous pouvons actualiser la page pour mettre à jour notre cookie, ou le définir manuellement comme `Cookie: role=web_admin`, puis intercepter la `Update`demande de création d'un nouvel utilisateur et voir si nous serions autorisés à le faire :

![créer_nouvel_utilisateur_2](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_create_new_user_2.jpg)

Nous n'avons pas reçu de message d'erreur cette fois. Si nous envoyons une `GET`demande pour le nouvel utilisateur, nous voyons qu'il a été créé avec succès :

![créer_nouvel_utilisateur_2](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_get_new_user.jpg)

En combinant les informations que nous avons obtenues avec `IDOR Information Disclosure vulnerability`une `IDOR Insecure Function Calls`attaque sur un point de terminaison d'API, nous pourrions modifier les détails d'autres utilisateurs et créer/supprimer des utilisateurs tout en contournant les divers contrôles de contrôle d'accès en place. À de nombreuses occasions, les informations que nous divulguons via les vulnérabilités IDOR peuvent être utilisées dans d'autres attaques, comme IDOR ou XSS, conduisant à des attaques plus sophistiquées ou contournant les mécanismes de sécurité existants.

Avec notre nouveau `role`, nous pouvons également effectuer des affectations en masse pour modifier des champs spécifiques pour tous les utilisateurs, comme placer des charges utiles XSS dans leurs profils ou changer leur e-mail en un e-mail que nous spécifions. `Try to write a script that changes all users' email to an email you choose.`. Vous pouvez le faire en récupérant leur `uuids`, puis en envoyant une `PUT`demande pour chacun avec le nouvel e-mail.
