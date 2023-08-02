Obfuscation de base
===================

* * * * *

L'obscurcissement du code n'est généralement pas effectué manuellement, car il existe de nombreux outils pour différents langages qui effectuent l'obscurcissement automatique du code. De nombreux outils en ligne peuvent être trouvés pour le faire, bien que de nombreux acteurs malveillants et développeurs professionnels développent leurs propres outils d'obscurcissement pour rendre plus difficile le désobscurcissement.

* * * * *

Exécuter du code JavaScript
---------------------------

Prenons la ligne de code suivante comme exemple et essayons de l'obscurcir :

Code : javascript

```
console.log('HTB JavaScript Deobfuscation Module');

```

Tout d'abord, testons l'exécution de ce code en texte clair, pour le voir fonctionner en action. Nous pouvons aller sur [JSConsole](https://jsconsole.com/) , coller le code et appuyer sur Entrée, et voir sa sortie :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_jsconsole_1_1.jpg)

Nous voyons que cette ligne de code imprime `HTB JavaScript Deobfuscation Module`, ce qui est fait à l'aide de la `console.log()`fonction.

* * * * *

Minification du code JavaScript
-------------------------------

Un moyen courant de réduire la lisibilité d'un extrait de code JavaScript tout en le gardant entièrement fonctionnel est la minification JavaScript. `Code minification`signifie avoir tout le code sur une seule ligne (souvent très longue). `Code minification`est plus utile pour un code plus long, comme si notre code ne consistait qu'en une seule ligne, il n'aurait pas l'air très différent une fois minifié.

De nombreux outils peuvent nous aider à minifier le code JavaScript, comme [javascript-minifier](https://javascript-minifier.com/) . Nous copions simplement notre code, et cliquons sur `Minify`, et nous obtenons la sortie minifiée à droite :

![](https://academy.hackthebox.com/storage/modules/41/js_minify_1.jpg)

Encore une fois, nous pouvons copier le code minifié dans [JSConsole](https://jsconsole.com/) et l'exécuter, et nous voyons qu'il s'exécute comme prévu. Habituellement, le code JavaScript minifié est enregistré avec l'extension `.min.js`.

Remarque : la minification de code n'est pas exclusive à JavaScript et peut être appliquée à de nombreux autres langages, comme on peut le voir sur [javascript-minifier](https://javascript-minifier.com/) .

* * * * *

Emballage du code JavaScript
----------------------------

Maintenant, obscurcissons notre ligne de code pour la rendre plus obscure et difficile à lire. Tout d'abord, nous allons essayer [BeautifyTools](http://beautifytools.com/javascript-obfuscator.php) pour obscurcir notre code :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_obfuscator.jpg)

Code : javascript

```
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5.4(\'3 2 1 0\');',6,6,'Module|Deobfuscation|JavaScript|HTB|log|console'.split('|'),0,{}))

```

Nous voyons que notre code est devenu beaucoup plus obscur et difficile à lire. Nous pouvons copier ce code dans [https://jsconsole.com](https://jsconsole.com/) , pour vérifier qu'il remplit toujours sa fonction principale :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_jsconsole_3_1.jpg)

Nous voyons que nous obtenons le même résultat.

Remarque : Le type d'obfuscation ci-dessus est connu sous le nom de "packing", qui est généralement reconnaissable à partir des six arguments de fonction utilisés dans la fonction initiale "function(p,a,c,k,e,d)".

Un `packer`outil d'obscurcissement tente généralement de convertir tous les mots et symboles du code en une liste ou un dictionnaire, puis de s'y référer à l'aide de la `(p,a,c,k,e,d)`fonction pour reconstruire le code d'origine lors de l'exécution. Cela `(p,a,c,k,e,d)`peut être différent d'un emballeur à l'autre. Cependant, il contient généralement un certain ordre dans lequel les mots et les symboles du code d'origine ont été emballés pour savoir comment les ordonner lors de l'exécution.

Bien qu'un packer fasse un excellent travail en réduisant la lisibilité du code, nous pouvons toujours voir ses chaînes principales écrites en texte clair, ce qui peut révéler certaines de ses fonctionnalités. C'est pourquoi nous voudrons peut-être chercher de meilleures façons d'obscurcir notre code.

[Précédent](https://academy.hackthebox.com/module/41/section/440)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/41/section/448)

Aide-mémoire
