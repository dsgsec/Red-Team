XSS stocké
==========

* * * * *

Avant d'apprendre à découvrir les vulnérabilités XSS et à les utiliser pour diverses attaques, nous devons d'abord comprendre les différents types de vulnérabilités XSS et leurs différences pour savoir lesquelles utiliser dans chaque type d'attaque.

Le premier type de vulnérabilité XSS, et le plus critique, est `Stored XSS` ou `Persistent XSS`. Si notre charge utile XSS injectée est stockée dans la base de données principale et récupérée lors de la visite de la page, cela signifie que notre attaque XSS est persistante et peut affecter tout utilisateur qui visite la page.

Cela fait de ce type de XSS le plus critique, car il affecte un public beaucoup plus large puisque tout utilisateur qui visite la page serait victime de cette attaque. De plus, Stored XSS peut ne pas être facilement amovible et la charge utile peut devoir être supprimée de la base de données principale.

Nous pouvons démarrer le serveur ci-dessous pour afficher et pratiquer un exemple de XSS stocké. Comme nous pouvons le constater, la page Web est une simple application `To-Do List` à laquelle nous pouvons ajouter des éléments. Nous pouvons essayer de saisir `test` et d'appuyer sur Entrée/Retour pour ajouter un nouvel élément et voir comment la page le gère :

![](https://academy.hackthebox.com/storage/modules/103/xss_stored_xss.jpg)

Comme nous pouvons le voir, notre entrée a été affichée sur la page. Si aucun nettoyage ou filtrage n'a été appliqué à notre entrée, la page pourrait être vulnérable à XSS.

* * * * *

Charges utiles de test XSS
--------------------

Nous pouvons tester si la page est vulnérable à XSS avec la charge utile XSS de base suivante :

Code : html

```
<script>alerte(window.origin)</script>

```

Nous utilisons cette charge utile car il s'agit d'une méthode très facile à repérer pour savoir quand notre charge utile XSS a été exécutée avec succès. Supposons que la page autorise n'importe quelle entrée et n'y effectue aucun nettoyage. Dans ce cas, l'alerte devrait apparaître avec l'URL de la page sur laquelle elle est exécutée, directement après avoir saisi notre charge utile ou lorsque nous actualisons la page :

![](https://academy.hackthebox.com/storage/modules/103/xss_stored_xss_alert.jpg)

Comme nous pouvons le voir, nous avons bien reçu l'alerte, ce qui signifie que la page est vulnérable à XSS, puisque notre charge utile s'est exécutée avec succès. Nous pouvons le confirmer davantage en consultant la source de la page en cliquant sur [`CTRL+U`] ou en cliquant avec le bouton droit de la souris et en sélectionnant `Afficher la source de la page`, et nous devrions voir notre charge utile dans la source de la page :

Code : html

```
<div></div><ul class="list-unstyled" id="todo"><ul><script>alerte(window.origin)</script>
</ul></ul>

```

Conseil : De nombreuses applications Web modernes utilisent des IFrames interdomaines pour gérer les entrées de l'utilisateur, de sorte que même si le formulaire Web est vulnérable à XSS, il ne s'agira pas d'une vulnérabilité sur l'application Web principale. C'est pourquoi nous affichons la valeur de `window.origin` dans la zone d'alerte, au lieu d'une valeur statique telle que `1`. Dans ce cas, la boîte d'alerte révélera l'URL sur laquelle elle est exécutée et confirmera quel formulaire est le plus vulnérable, au cas où un IFrame était utilisé.

Comme certains navigateurs modernes peuvent bloquer la fonction JavaScript `alert()` à des emplacements spécifiques, il peut être utile de connaître quelques autres charges utiles XSS de base pour vérifier l'existence de XSS. L'une de ces charges utiles XSS est `<plaintext>`, qui arrête d'afficher le code HTML qui le suit et l'affiche en texte brut. Une autre charge utile facile à repérer est `<script>print()</script>` qui fera apparaître la boîte de dialogue d'impression du navigateur, qui ne sera probablement pas bloquée par les navigateurs. Essayez d'utiliser ces charges utiles pour voir comment chacune fonctionne. Vous pouvez utiliser le bouton de réinitialisation pour supprimer toutes les charges utiles actuelles.

Pour voir si la charge utile est persistante et stockée sur le back-end, nous pouvons actualiser la page et voir si nous recevons à nouveau l'alerte. Si nous le faisions, nous verrions que nous continuons à recevoir l'alerte même tout au long de l'actualisation de la page, confirmant qu'il s'agit bien d'une vulnérabilité `Stored/Persistent XSS` . Cela ne nous est pas propre, car tout utilisateur qui visite la page déclenchera la charge utile XSS et recevra la même alerte.