XSS réfléchi
=============

* * * * *

Il existe deux types de vulnérabilités `Non-Persistent XSS` : `Reflected XSS`, qui est traité par le serveur principal, et ``basé sur DOM', qui est entièrement traité côté client et n'atteint jamais l'arrière. -serveur final. Contrairement à Persistent XSS, les vulnérabilités `Non-Persistent XSS` sont temporaires et ne persistent pas lors des actualisations de page. Par conséquent, nos attaques n'affectent que l'utilisateur ciblé et n'affecteront pas les autres utilisateurs qui visitent la page.

Les vulnérabilités `Reflected XSS` se produisent lorsque notre entrée atteint le serveur principal et nous est renvoyée sans être filtrée ni nettoyée. Il existe de nombreux cas dans lesquels l'intégralité de notre entrée peut nous être renvoyée, comme des messages d'erreur ou des messages de confirmation. Dans ces cas, nous pouvons essayer d'utiliser les charges utiles XSS pour voir si elles s'exécutent. Cependant, comme il s'agit généralement de messages temporaires, une fois que nous quittons la page, ils ne s'exécuteront plus, et ils sont donc "non persistants".

Nous pouvons démarrer le serveur ci-dessous pour nous entraîner sur une page Web vulnérable à une vulnérabilité XSS réfléchie. Il s'agit d'une application `To-Do List` similaire à celle avec laquelle nous nous sommes entraînés dans la section précédente. Nous pouvons essayer d'ajouter n'importe quelle chaîne `test` pour voir comment elle est gérée :

![](https://academy.hackthebox.com/storage/modules/103/xss_reflected_1.jpg)

Comme nous pouvons le voir, nous obtenons `La tâche 'test' n'a pas pu être ajoutée.`, qui inclut notre entrée `test` dans le cadre du message d'erreur. Si notre entrée n'a pas été filtrée ou nettoyée, la page pourrait être vulnérable à XSS. Nous pouvons essayer la même charge utile XSS que nous avons utilisée dans la section précédente et cliquer sur "Ajouter" :

![](https://academy.hackthebox.com/storage/modules/103/xss_reflected_2.jpg)

Une fois que nous avons cliqué sur `Ajouter`, nous obtenons la fenêtre d'alerte :

![](https://academy.hackthebox.com/storage/modules/103/xss_stored_xss_alert.jpg)

Dans ce cas, nous constatons que le message d'erreur indique désormais `La tâche '' n'a pas pu être ajoutée.`. Étant donné que notre charge utile est enveloppée d'une balise `<script>` , elle n'est pas rendue par le navigateur, nous obtenons donc des guillemets simples vides `''` à la place. Nous pouvons à nouveau afficher la source de la page pour confirmer que le message d'erreur inclut notre charge utile XSS :

Code : html

```
<div></div><ul class="list-unstyled" id="todo"><div style="padding-left:25px">Tâche '<script>alerte(window.origin)</script>' n'a pas pu être ajouté.</div></ul>

```

Comme nous pouvons le voir, les guillemets simples contiennent en effet notre charge utile XSS `'<script>alert(window.origin)</script>'`.

Si nous visitons à nouveau la page `Reflected` , le message d'erreur n'apparaît plus et notre payload XSS n'est pas exécuté, ce qui signifie que cette vulnérabilité XSS est en effet `Non-Persistent`.

`Mais si la vulnérabilité XSS est non persistante, comment pourrions-nous cibler les victimes ?`

Cela dépend de la requête HTTP utilisée pour envoyer notre entrée au serveur. Nous pouvons vérifier cela via les `Outils de développement` de Firefox en cliquant sur [`CTRL+I`] et en sélectionnant l'onglet `Réseau` . Ensuite, nous pouvons remettre notre payload `test` et cliquer sur `Add` pour l'envoyer :

![](https://academy.hackthebox.com/storage/modules/103/xss_reflected_network.jpg)

Comme nous pouvons le voir, la première ligne indique que notre requête était une requête `GET`. La requête `GET` envoie leurs paramètres et données dans le cadre de l'URL. Ainsi, `pour cibler un utilisateur, nous pouvons lui envoyer une URL contenant notre charge utile`. Pour obtenir l'URL, nous pouvons copier l'URL de la barre d'URL dans Firefox après avoir envoyé notre charge utile XSS, ou nous pouvons cliquer avec le bouton droit sur la requête `GET` dans l'onglet `Réseau` et sélectionner `Copier> Copier l'URL`. Une fois que la victime visite cette URL, la charge utile XSS s'exécute :

![](https://academy.hackthebox.com/storage/modules/103/xss_stored_xss_alert.jpg)

Plein écran  Terminer