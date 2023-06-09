Identification des IDOR
=================

* * * * *

Paramètres d'URL et API
---------------------

* * * * *

La toute première étape de l'exploitation des vulnérabilités IDOR consiste à identifier les références directes d'objets. Chaque fois que nous recevons un fichier ou une ressource spécifique, nous devons étudier les requêtes HTTP pour rechercher des paramètres d'URL ou des API avec une référence d'objet (par exemple `?uid=1` ou `?filename=file_1.pdf`). Ceux-ci se trouvent principalement dans les paramètres d'URL ou les API, mais peuvent également être trouvés dans d'autres en-têtes HTTP, comme les cookies.

Dans les cas les plus basiques, nous pouvons essayer d'incrémenter les valeurs des références d'objets pour récupérer d'autres données, comme (`?uid=2`) ou (`?filename=file_2.pdf`). Nous pouvons également utiliser une application de fuzzing pour essayer des milliers de variantes et voir si elles renvoient des données. Tout accès réussi à des fichiers qui ne nous appartiennent pas indiquerait une vulnérabilité IDOR.

* * * * *

Appels AJAX
----------

Nous pouvons également être en mesure d'identifier des paramètres ou des API inutilisés dans le code frontal sous la forme d'appels JavaScript AJAX. Certaines applications Web développées dans des frameworks JavaScript peuvent placer de manière non sécurisée tous les appels de fonction sur le front-end et utiliser ceux qui sont appropriés en fonction du rôle de l'utilisateur.

Par exemple, si nous n'avions pas de compte administrateur, seules les fonctions de niveau utilisateur seraient utilisées, tandis que les fonctions d'administration seraient désactivées. Cependant, nous pourrons peut-être toujours trouver les fonctions d'administration si nous examinons le code JavaScript frontal et pouvons identifier les appels AJAX vers des points de terminaison ou des API spécifiques contenant des références directes d'objets. Si nous identifions des références d'objets directes dans le code JavaScript, nous pouvons les tester pour les vulnérabilités IDOR.

Ce n'est bien sûr pas propre aux fonctions d'administration, mais il peut également s'agir de fonctions ou d'appels qui ne peuvent pas être trouvés via la surveillance des requêtes HTTP. L'exemple suivant montre un exemple de base d'un appel AJAX :

Code : javascript

```
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}


```

La fonction ci-dessus ne peut jamais être appelée lorsque nous utilisons l'application Web en tant qu'utilisateur non administrateur. Cependant, si nous le localisons dans le code frontal, nous pouvons le tester de différentes manières pour voir si nous pouvons l'appeler pour effectuer des modifications, ce qui indiquerait qu'il est vulnérable à IDOR. Nous pouvons faire de même avec le code back-end si nous y avons accès (par exemple, des applications Web open source).

* * * * *

Comprendre le hachage/encodage
---------------------------

Certaines applications Web peuvent ne pas utiliser de simples numéros séquentiels comme références d'objet, mais peuvent encoder la référence ou la hacher à la place. Si nous trouvons de tels paramètres en utilisant des valeurs codées ou hachées, nous pouvons toujours être en mesure de les exploiter s'il n'y a pas de système de contrôle d'accès sur le back-end.

Supposons que la référence ait été encodée avec un encodeur commun (par exemple `base64`). Dans ce cas, nous pourrions le décoder et afficher le texte en clair de la référence d'objet, modifier sa valeur, puis l'encoder à nouveau pour accéder à d'autres données. Par exemple, si nous voyons une référence comme (`?filename=ZmlsZV8xMjMucGRm`), nous pouvons immédiatement deviner que le nom du fichier est `base64` codé (à partir de son jeu de caractères), que nous pouvons décoder pour obtenir la référence d'objet d'origine de ( `fichier_123.pdf`). Ensuite, nous pouvons essayer d'encoder une référence d'objet différente (par exemple `file_124.pdf`) et essayer d'y accéder avec la référence d'objet encodée (`?filename=ZmlsZV8xMjQucGRm`), ce qui peut révéler une vulnérabilité IDOR si nous avons pu récupérer des données. .

D'autre part, la référence de l'objet peut être hachée, comme (`download.php?filename=c81e728d9d4c2f636f067f89cc14862c`). À première vue, on peut penser qu'il s'agit d'une référence d'objet sécurisée, car elle n'utilise pas de texte clair ni d'encodage facile. Cependant, si nous regardons le code source, nous pouvons voir ce qui est haché avant que l'appel API ne soit effectué :

Code : javascript

```
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});

```

Dans ce cas, nous pouvons voir que le code utilise le `filename` et le hache avec `CryptoJS.MD5`, ce qui nous permet de calculer facilement le `filename` pour d'autres fichiers potentiels. Sinon, nous pouvons essayer manuellement d'identifier l'algorithme de hachage utilisé (par exemple, avec des outils d'identification de hachage), puis hacher le nom de fichier pour voir s'il correspond au hachage utilisé. Une fois que nous pouvons calculer les hachages pour d'autres fichiers, nous pouvons essayer de les télécharger, ce qui peut révéler une vulnérabilité IDOR si nous pouvons télécharger des fichiers qui ne nous appartiennent pas.

* * * * *

Comparer les rôles d'utilisateur
------------------

Si nous voulons effectuer des attaques IDOR plus avancées, nous devrons peut-être enregistrer plusieurs utilisateurs et comparer leurs requêtes HTTP et leurs références d'objet. Cela peut nous permettre de comprendre comment les paramètres d'URL et les identifiants uniques sont calculés, puis de les calculer pour que d'autres utilisateurs collectent leurs données.

Par exemple, si nous avait accès à deux utilisateurs différents, dont l'un peut voir son salaire après avoir effectué l'appel d'API suivant :

Code : json

```
{
  "attributes" :
    {
      "type" : "salary",
      "url" : "/services/data/salaries/users/1"
    },
  "Id" : "1",
  "Name" : "User1"

}

```

Le deuxième utilisateur peut ne pas disposer de tous ces paramètres d'API pour répliquer l'appel et ne devrait pas être en mesure d'effectuer le même appel que `User1`. Cependant, avec ces détails à portée de main, nous pouvons essayer de répéter le même appel d'API en étant connecté en tant qu'`User2` pour voir si l'application Web renvoie quelque chose. De tels cas peuvent fonctionner si l'application Web ne nécessite qu'une session de connexion valide pour effectuer l'appel d'API, mais n'a aucun contrôle d'accès sur le back-end pour comparer la session de l'appelant avec les données appelées.

Si tel est le cas, et que nous pouvons calculer les paramètres de l'API pour d'autres utilisateurs, il s'agirait d'une vulnérabilité IDOR. Même si nous ne pouvions pas calculer les paramètres de l'API pour les autres utilisateurs, nous aurions quand même identifié une vulnérabilité dans le système de contrôle d'accès back-end et pourrions commencer à rechercher d'autres références d'objets à exploiter.
