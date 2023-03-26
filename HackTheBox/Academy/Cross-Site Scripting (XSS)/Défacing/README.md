Défigurer
========

* * * * *

Maintenant que nous comprenons les différents types de XSS et les différentes méthodes de découverte des vulnérabilités XSS dans les pages Web, nous pouvons commencer à apprendre à exploiter ces vulnérabilités XSS. Comme mentionné précédemment, les dommages et la portée d'une attaque XSS dépendent du type de XSS, un XSS stocké étant le plus critique, tandis qu'un XSS basé sur DOM l'est moins.

L'une des attaques les plus courantes généralement utilisées avec les vulnérabilités XSS stockées est les attaques de défiguration de sites Web. "Dégrader" un site Web signifie modifier son apparence pour toute personne qui visite le site Web. Il est très courant que des groupes de pirates défigurent un site Web pour affirmer qu'ils l'ont piraté avec succès, comme lorsque des pirates ont défiguré le National Health Service (NHS) du Royaume-Uni [en 2018](https://www.bbc.co.uk/ actualités/technologie-43812539). De telles attaques peuvent avoir un grand écho dans les médias et peuvent affecter de manière significative les investissements et le cours des actions d'une entreprise, en particulier pour les banques et les entreprises technologiques.

Bien que de nombreuses autres vulnérabilités puissent être utilisées pour obtenir le même résultat, les vulnérabilités XSS stockées sont parmi les vulnérabilités les plus utilisées pour ce faire.

* * * * *

Éléments de dégradation
-------------------

Nous pouvons utiliser du code JavaScript injecté (via XSS) pour donner à une page Web l'apparence que nous souhaitons. Cependant, la dégradation d'un site Web est généralement utilisée pour envoyer un message simple (c'est-à-dire que nous vous avons piraté avec succès), donc donner à la page Web dégradée un bel aspect n'est pas vraiment la cible principale.

Trois éléments HTML sont généralement utilisés pour modifier l'apparence principale d'une page Web :

- Couleur d'arrière-plan `document.body.style.background`
- Arrière-plan `document.body.background`
- Titre de la page `document.title`
- Texte de la page `DOM.innerHTML`

Nous pouvons utiliser deux ou trois de ces éléments pour écrire un message de base sur la page Web et même supprimer l'élément vulnérable, de sorte qu'il serait plus difficile de réinitialiser rapidement la page Web, comme nous le verrons ensuite.

* * * * *

Modification de l'arrière-plan
-------------------

Revenons à notre exercice `Stored XSS` et utilisons-le comme base pour notre attaque. Vous pouvez revenir à la section `XSS stocké` pour générer le serveur et suivre les étapes suivantes.

Pour changer l'arrière-plan d'une page Web, nous pouvons choisir une certaine couleur ou utiliser une image. Nous utiliserons une couleur comme arrière-plan car la plupart des attaques de dégradation utilisent une couleur sombre pour l'arrière-plan. Pour ce faire, nous pouvons utiliser la charge utile suivante :

Code : html

```
<script>document.body.style.background = "#141d2b"</script>

```

Astuce : Ici, nous définissons la couleur d'arrière-plan sur la couleur d'arrière-plan par défaut de Hack The Box. Nous pouvons utiliser n'importe quelle autre valeur hexadécimale ou utiliser une couleur nommée comme `= "noir"`.

Une fois que nous aurons ajouté notre charge utile à la liste `To-Do` , nous verrons que la couleur d'arrière-plan a changé :

![](https://academy.hackthebox.com/storage/modules/103/xss_defacing_background_color.jpg)

Cela sera persistant lors des actualisations de page et apparaîtra pour tous ceux qui visitent la page, car nous utilisons une vulnérabilité XSS stockée.

Une autre option consisterait à définir une image en arrière-plan à l'aide de la charge utile suivante :

Code : html

```
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>

```

Essayez d'utiliser la charge utile ci-dessus pour voir à quoi le résultat final peut ressembler.

* * * * *

Changer le titre de la page
-------------------

Nous pouvons remplacer le titre de la page `2Do` par le titre de notre choix, à l'aide de la fonction JavaScript `document.title`  :

Code : html

```
<script>document.title = 'Académie HackTheBox'</script>

```

Nous pouvons voir à partir de la fenêtre/onglet de la page que notre nouveau titre a remplacé le précédent :

![](https://academy.hackthebox.com/storage/modules/103/xss_defacing_page_title.jpg)

* * * * *

Modification du texte de la page
------------------

Lorsque nous voulons modifier le texte affiché sur la page Web, nous pouvons utiliser diverses fonctions JavaScript pour le faire. Par exemple, nous pouvons modifier le texte d'un élément HTML/DOM spécifique à l'aide de la fonction `innerHTML` :

Code : javascript

```
document.getElementById("todo").innerHTML = "Nouveau texte"

```

Nous pouvons également utiliser les fonctions jQuery pour obtenir plus efficacement la même chose ou pour modifier le texte de plusieurs éléments sur une seule ligne (pour ce faire, la bibliothèque `jQuery` doit avoir été importée dans la source de la page) :

Code : javascript

```
$("#todo").html('Nouveau texte');

```

Cela nous donne diverses options pour personnaliser le texte sur la page Web et faire des ajustements mineurs pour répondre à nos besoins. Cependant, étant donné que les groupes de piratage laissent généralement un simple message sur la page Web et ne laissent rien d'autre dessus, nous modifierons l'intégralité du code HTML du principal `body`, en utilisant `innerHTML`, comme suit :

Code : javascript

```
document.getElementsByTagName('body')[0].innerHTML = "Nouveau texte"

```

Comme nous pouvons le voir, nous pouvons spécifier l'élément `body` avec `document.getElementsByTagName('body')`, et en spécifiant `[0]`, nous sélectionnons le premier élément `body` , qui devrait modifier l'intégralité du texte de la page Web. Nous pouvons également utiliser `jQuery` pour obtenir le même résultat. Cependant, avant d'envoyer notre charge utile et de faire un changement permanent, nous devons préparer notreCode HTML séparément, puis utilisez `innerHTML` pour définir notre code HTML sur la source de la page.

Pour notre exercice, nous emprunterons le code HTML de la page principale de `Hack The Box Academy` :

Code : html

```
<centre>
     <h1 style="color: white">Formation sur la cybersécurité</h1>
     <p style="color: white">par
         <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy">
     </p>
</center>

```

Conseil : il serait sage d'essayer d'exécuter notre code HTML localement pour voir à quoi il ressemble et pour s'assurer qu'il s'exécute comme prévu, avant de nous y engager dans notre charge utile finale.

Nous allons réduire le code HTML en une seule ligne et l'ajouter à notre charge utile XSS précédente. La charge utile finale devrait être la suivante :

Code : html

```
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Formation à la cybersécurité</h1><p style="color: white">par < img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>

```

Une fois que nous aurons ajouté notre charge utile à la liste `To-Do` vulnérable, nous verrons que notre code HTML fait désormais partie intégrante du code source de la page Web et affiche notre message pour toute personne visitant la page :

![](https://academy.hackthebox.com/storage/modules/103/xss_defacing_change_text.jpg)

En utilisant trois charges utiles XSS, nous avons réussi à défigurer notre page Web cible. Si nous regardons le code source de la page Web, nous verrons que le code source d'origine existe toujours, et nos charges utiles injectées apparaissent à la fin :

Code : html

```
<div></div><ul class="list-unstyled" id="todo"><ul>
<script>document.body.style.background = "#141d2b"</script>
</ul><ul><script>document.title = 'Académie HackTheBox'</script>
</ul><ul><script>document.getElementsByTagName('body')[0].innerHTML = '...SNIP...'</script>
</ul></ul>

```

En effet, notre code JavaScript injecté modifie l'apparence de la page lorsqu'il est exécuté, ce qui, dans ce cas, se trouve à la fin du code source. Si notre injection était dans un élément au milieu du code source, d'autres scripts ou éléments peuvent être ajoutés après, nous devrons donc en tenir compte pour obtenir l'aspect final dont nous avons besoin.

Cependant, pour les utilisateurs ordinaires, la page semble dégradée et montre notre nouveau look.