Désobscurcissement
==================

* * * * *

Maintenant que nous comprenons comment fonctionne l'obscurcissement du code, commençons notre apprentissage de la désobfuscation. Tout comme il existe des outils pour masquer automatiquement le code, il existe des outils pour embellir et désobscurcir le code automatiquement.

* * * * *

Embellir
--------

Nous voyons que le code actuel que nous avons est entièrement écrit sur une seule ligne. C'est ce qu'on appelle `Minified JavaScript`le code. Afin de formater correctement le code, nous avons besoin de `Beautify`notre code. La méthode la plus basique pour le faire est d'utiliser notre `Browser Dev Tools`.

Par exemple, si nous utilisions Firefox, nous pouvons ouvrir le débogueur du navigateur avec [ `CTRL+SHIFT+Z`], puis cliquer sur notre script `secret.js`. Cela affichera le script dans sa mise en forme d'origine, mais nous pouvons cliquer sur le `{ }`bouton ' ' en bas, ce qui mettra `Pretty Print`le script dans sa mise en forme JavaScript appropriée : ![](https://academy.hackthebox.com/storage/modules/41/js_deobf_pretty_print.jpg)

De plus, nous pouvons utiliser de nombreux outils en ligne ou plugins d'éditeur de code, comme [Prettier](https://prettier.io/playground/) ou [Beautifier](https://beautifier.io/) . Copions le `secret.js`script :

Code : javascript

```
eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('g 4(){0 5="6{7!}";0 1=8 a();0 2="/9.c";1.d("e",2,f);1.b(3)}', 17, 17, 'var|xhr|url|null|generateSerial|flag|HTB|flag|new|serial|XMLHttpRequest|send|php|open|POST|true|function'.split('|'), 0, {}))

```

Nous pouvons voir que les deux sites Web font un bon travail dans le formatage du code :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_prettier_1.jpg)

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_beautifier_1.jpg)

Cependant, le code n'est toujours pas très facile à lire. C'est parce que le code auquel nous avons affaire était non seulement minifié, mais également obscurci. Ainsi, simplement formater ou embellir le code ne suffira pas. Pour cela, nous aurons besoin d'outils pour désobscurcir le code.

* * * * *

Désobscurcir
------------

Nous pouvons trouver de nombreux bons outils en ligne pour désobscurcir le code JavaScript et le transformer en quelque chose que nous pouvons comprendre. Un bon outil est [JSNice](http://www.jsnice.org/) . Essayons de copier notre code obscurci ci-dessus et de l'exécuter dans JSNice en cliquant sur le `Nicify JavaScript`bouton.

Astuce : Nous devrions cliquer sur le bouton d'options à côté du bouton "Nicify JavaScript", et désélectionner "Infer types" pour réduire l'encombrement du code avec des commentaires.

Conseil : assurez-vous de ne laisser aucune ligne vide avant le script, car cela pourrait affecter le processus de désobscurcissement et donner des résultats inexacts.

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_jsnice_1.jpg)

Nous pouvons voir que cet outil fait un bien meilleur travail en désobscurcissant le code JavaScript et nous a donné une sortie que nous pouvons comprendre :

Code : javascript

```
'use strict';
function generateSerial() {
  ...SNIP...
  var xhr = new XMLHttpRequest;
  var url = "/serial.php";
  xhr.open("POST", url, true);
  xhr.send(null);
};

```

Comme mentionné précédemment, la méthode d'obscurcissement utilisée ci-dessus est `packing`. Une autre façon d'utiliser `unpacking`un tel code consiste à trouver la `return`valeur à la fin et à l'utiliser `console.log`pour l'imprimer au lieu de l'exécuter.

* * * * *

Ingénierie inverse
------------------

Bien que ces outils fassent du bon travail jusqu'à présent pour clarifier le code en quelque chose que nous pouvons comprendre, une fois que le code devient plus obscur et encodé, il deviendra beaucoup plus difficile pour les outils automatisés de le nettoyer. Cela est particulièrement vrai si le code a été obscurci à l'aide d'un outil d'obscurcissement personnalisé.

Nous aurions besoin de désosser manuellement le code pour comprendre comment il a été obscurci et sa fonctionnalité dans de tels cas. Si vous souhaitez en savoir plus sur la désobfuscation JavaScript avancée et l'ingénierie inverse, vous pouvez consulter le module [Secure Coding 101 , qui devrait couvrir ce sujet en détail.](https://academy.hackthebox.com/module/details/38)
