=================================================

* * * * *

Même si plonger plus profondément dans les contournements de la protection CSRF n'entre pas dans le cadre de ce module, découvrez ci-dessous quelques approches qui peuvent s'avérer utiles lors d'engagements ou de chasse aux bugs.

Valeur nulle
------------

* * * * *

Vous pouvez essayer de donner au jeton CSRF une valeur nulle (vide), par exemple :

`CSRF-Token:`

Cela peut fonctionner car parfois, la vérification ne recherche que l'en-tête et ne valide pas la valeur du jeton. Dans de tels cas, nous pouvons créer nos requêtes intersites à l'aide d'un jeton CSRF nul, tant que l'en-tête est fourni dans la requête.

Jeton CSRF aléatoire
--------------------

* * * * *

Définir la valeur du jeton CSRF sur la même longueur que le jeton CSRF d'origine mais avec une valeur différente/aléatoire peut également contourner une protection anti-CSRF qui valide si le jeton a une valeur et la longueur de cette valeur. Par exemple, si le jeton CSRF avait une longueur de 32 octets, nous recréerions un jeton de 32 octets.

Réel:

`CSRF-Token: 9cfffd9e8e78bd68975e295d1b3d3331`

Faux:

`CSRF-Token: 9cfffl3dj3837dfkj3j387fjcxmfjfd3`

Utiliser le jeton CSRF d'une autre session
------------------------------------------

* * * * *

Un autre contournement de la protection anti-CSRF consiste à utiliser le même jeton CSRF sur tous les comptes. Cela peut fonctionner dans les applications qui ne valident pas si le jeton CSRF est lié à un compte spécifique ou non et ne vérifient que si le jeton est algorithmiquement correct.

Créez deux comptes et connectez-vous au premier compte. Générez une requête et capturez le jeton CSRF. Copiez la valeur du jeton, par exemple, `CSRF-Token=9cfffd9e8e78bd68975e295d1b3d3331`.

Connectez-vous au deuxième compte et modifiez la valeur de *CSRF-Token* en `9cfffd9e8e78bd68975e295d1b3d3331`tout en émettant la même demande (ou une demande différente). Si la demande est émise avec succès, nous pouvons exécuter avec succès des attaques CSRF en utilisant un jeton généré via notre compte qui est considéré comme valide sur plusieurs comptes.

Demander la falsification de la méthode
---------------------------------------

* * * * *

Pour contourner les protections anti-CSRF, on peut essayer de changer la méthode de requête. De *POST* à *​​GET* et vice versa.

Par exemple, si l'application utilise POST, essayez de le remplacer par GET :

Code : http

```
POST /change_password
POST body:
new_password=pwned&confirm_new=pwned

```

Code : http

```
GET /change_password?new_password=pwned&confirm_new=pwned

```

Des demandes inattendues peuvent être servies sans avoir besoin d'un jeton CSRF.

Supprimez le paramètre de jeton CSRF ou envoyez un jeton vide
-------------------------------------------------------------

* * * * *

Ne pas envoyer de jeton fonctionne assez souvent en raison de l'erreur de logique d'application courante suivante. Les applications ne vérifient parfois la validité du jeton que si le jeton existe ou si le paramètre du jeton n'est pas vide.

Demande réelle :

Code : http

```
POST /change_password
POST body:
new_password=qwerty&csrf_token=9cfffd9e8e78bd68975e295d1b3d3331

```

Essayer:

Code : http

```
POST /change_password
POST body:
new_password=qwerty

```

Ou:

Code : http

```
POST /change_password
POST body:
new_password=qwerty&csrf_token=

```

Fixation de session > CSRF
--------------------------

* * * * *

Parfois, les sites utilisent ce qu'on appelle un cookie de double soumission comme défense contre CSRF. Cela signifie que la requête envoyée contiendra le même jeton aléatoire à la fois comme cookie et comme paramètre de requête, et le serveur vérifie si les deux valeurs sont égales. Si les valeurs sont égales, la demande est considérée comme légitime.

Si le cookie de double soumission est utilisé comme mécanisme de défense, l'application ne conserve probablement pas le jeton valide côté serveur. Il n'a aucun moyen de savoir si un jeton qu'il reçoit est légitime et vérifie simplement que le jeton dans le cookie et le jeton dans le corps de la requête sont identiques.

Si tel est le cas et qu'une vulnérabilité de fixation de session existe, un attaquant pourrait réussir une attaque CSRF comme suit :

Pas:

1.  Fixation de session
2.  Exécutez CSRF avec la requête suivante :

Code : http

```
POST /change_password
Cookie: CSRF-Token=fixed_token;
POST body:
new_password=pwned&CSRF-Token=fixed_token

```

Protection anti-CSRF via le Referrer Header
-------------------------------------------

* * * * *

Si une application utilise l'en-tête de référence comme mécanisme anti-CSRF, vous pouvez essayer de supprimer l'en-tête de référence. Ajoutez la balise meta suivante à votre page hébergeant votre script CSRF.

`<meta name="referrer" content="no-referrer"`

Contourner la Regex
-------------------

* * * * *

Parfois, le référent a une regex de liste blanche ou une regex qui autorise un domaine spécifique.

Supposons que le Referrer Header recherche *google.com* . Nous pourrions essayer quelque chose comme `www.google.com.pwned.m3`, qui peut contourner la regex ! S'il utilise son propre domaine ( `target.com`) comme liste blanche, essayez d'utiliser le domaine cible comme suit `www.target.com.pwned.m3`.

Vous pouvez également essayer certaines des solutions suivantes :

`www.target.com?www.pwned.m3`ou`www.pwned.m3/www.target.com`

Dans la section suivante, nous aborderons les vulnérabilités d'Open Redirect en nous concentrant sur l'attaque de la session d'un utilisateur.

[Précédent](https://academy.hackthebox.com/module/153/section/1451)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/153/section/1453)
