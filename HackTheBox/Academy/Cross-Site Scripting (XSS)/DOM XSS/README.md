DOM XSS
=======

* * * * *

Le troisième et dernier type de XSS est un autre type "non persistant" appelé "XSS basé sur DOM". Alors que `Reflected XSS` envoie les données d'entrée au serveur principal via des requêtes HTTP, DOM XSS est entièrement traité côté client via JavaScript. DOM XSS se produit lorsque JavaScript est utilisé pour modifier la source de la page via le `Document Object Model (DOM)`.

Nous pouvons exécuter le serveur ci-dessous pour voir un exemple d'application Web vulnérable à DOM XSS. Nous pouvons essayer d'ajouter un élément `test` et nous voyons que l'application Web est similaire aux applications Web `To-Do List` que nous utilisions précédemment :

![](https://academy.hackthebox.com/storage/modules/103/xss_dom_1.jpg)

Cependant, si nous ouvrons l'onglet `Réseau` dans les outils de développement de Firefox et que nous rajoutons l'élément `test` , nous remarquerons qu'aucune requête HTTP n'est effectuée :

![](https://academy.hackthebox.com/storage/modules/103/xss_dom_network.jpg)

Nous constatons que le paramètre d'entrée dans l'URL utilise un hashtag `#` pour l'élément que nous avons ajouté, ce qui signifie qu'il s'agit d'un paramètre côté client qui est entièrement traité sur le navigateur. Cela indique que l'entrée est en cours de traitement côté client via JavaScript et n'atteint jamais le back-end ; il s'agit donc d'un `XSS basé sur DOM`.

De plus, si nous regardons la source de la page en appuyant sur [`CTRL+I`], nous remarquerons que notre chaîne `test` est introuvable. En effet, le code JavaScript met à jour la page lorsque nous cliquons sur le bouton `Ajouter` , après que la source de la page a été récupérée par notre navigateur. Par conséquent, la source de la page de base n'affichera pas notre entrée, et si nous actualisons la page, elle ne seront pas conservés (c'est-à-dire "Non persistant"). Nous pouvons toujours afficher la source de la page rendue avec l'outil Web Inspector en cliquant sur [`CTRL+SHIFT+C`] :

![](https://academy.hackthebox.com/storage/modules/103/xss_dom_inspector.jpg)

* * * * *

Source et puits
--------------

Pour mieux comprendre la nature de la vulnérabilité XSS basée sur DOM, nous devons comprendre le concept de `Source` et `Sink` de l'objet affiché sur la page. La `Source` est l'objet JavaScript qui prend l'entrée de l'utilisateur, et il peut s'agir de n'importe quel paramètre d'entrée comme un paramètre d'URL ou un champ de saisie, comme nous l'avons vu ci-dessus.

D'autre part, le `Sink` est la fonction qui écrit l'entrée de l'utilisateur dans un objet DOM sur la page. Si la fonction `Sink` ne nettoie pas correctement l'entrée de l'utilisateur, elle serait vulnérable à une attaque XSS. Certaines des fonctions JavaScript couramment utilisées pour écrire dans des objets DOM sont :

- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`

De plus, certaines des fonctions de la bibliothèque `jQuery` qui écrivent dans les objets DOM sont :

- `add()`
- `after()`
- `append()`

Si une fonction `Sink` écrit l'entrée exacte sans aucun nettoyage (comme les fonctions ci-dessus) et qu'aucun autre moyen de nettoyage n'a été utilisé, nous savons que la page doit être vulnérable à XSS.

Nous pouvons regarder le code source de l'application Web `To-Do` et vérifier `script.js`, et nous verrons que `Source` est extrait du paramètre `task=`  :

Code : javascript

```
var pos = document.URL.indexOf("task=");
var tâche = document.URL.substring(pos + 5, document.URL.length);

```

Juste en dessous de ces lignes, nous voyons que la page utilise la fonction `innerHTML` pour écrire la variable `task` dans le DOM `todo` :

Code : javascript

```
document.getElementById("todo").innerHTML = "<b>Prochaine tâche :</b>" + decodeURIComponent(task);

```

Ainsi, nous pouvons voir que nous pouvons contrôler l'entrée et que la sortie n'est pas nettoyée, donc cette page devrait être vulnérable à DOM XSS.

* * * * *

Attaques DOM
-----------

Si nous essayons la charge utile XSS que nous avons utilisée précédemment, nous verrons qu'elle ne s'exécutera pas. En effet, la fonction `innerHTML` n'autorise pas l'utilisation des balises `<script>` qu'elle contient comme fonctionnalité de sécurité. Néanmoins, de nombreuses autres charges utiles XSS que nous utilisons ne contiennent pas de balises `<script>` , comme la charge utile XSS suivante :

Code : html

```
<img src="" onerror=alert(window.origin)>

```

La ligne ci-dessus crée un nouvel objet d'image HTML, qui possède un attribut `onerror` qui peut exécuter du code JavaScript lorsque l'image est introuvable. Ainsi, comme nous avons fourni un lien d'image vide (`""`), notre code doit toujours être exécuté sans avoir à utiliser les balises `<script>` :

![](https://academy.hackthebox.com/storage/modules/103/xss_dom_alert.jpg)

Pour cibler un utilisateur avec cette vulnérabilité DOM XSS, nous pouvons à nouveau copier l'URL du navigateur et la partager avec eux, et une fois qu'ils l'ont visitée, le code JavaScript devrait s'exécuter. Ces deux charges utiles font partie des charges utiles XSS les plus élémentaires. Il existe de nombreux cas où nous devrons peut-être utiliser diverses charges utiles en fonction de la sécurité de l'application Web et du navigateur, dont nous parlerons dans la section suivante.