Obfuscation avancée
===================

* * * * *

Jusqu'à présent, nous avons réussi à rendre notre code obscurci et plus difficile à lire. Cependant, le code contient toujours des chaînes en texte clair, ce qui peut révéler sa fonctionnalité d'origine. Dans cette section, nous allons essayer quelques outils qui devraient complètement obscurcir le code et masquer toute rémanence de sa fonctionnalité d'origine.

* * * * *

Obfuscateur
-----------

Visitons [https://obfuscator.io](https://obfuscator.io/) . Avant de cliquer sur `obfuscate`, nous allons passer `String Array Encoding`à `Base64`, comme indiqué ci-dessous :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_obfuscator_2.jpg)

Maintenant, nous pouvons coller notre code et cliquer sur `obfuscate`:

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_obfuscator_1.jpg)

Nous obtenons le code suivant :

Code : javascript

```
var _0x1ec6=['Bg9N','sfrciePHDMfty3jPChqGrgvVyMz1C2nHDgLVBIbnB2r1Bgu='];(function(_0x13249d,_0x1ec6e5){var _0x14f83b=function(_0x3f720f){while(--_0x3f720f){_0x13249d['push'](_0x13249d['shift']());}};_0x14f83b(++_0x1ec6e5);}(_0x1ec6,0xb4));var _0x14f8=function(_0x13249d,_0x1ec6e5){_0x13249d=_0x13249d-0x0;var _0x14f83b=_0x1ec6[_0x13249d];if(_0x14f8['eOTqeL']===undefined){var _0x3f720f=function(_0x32fbfd){var _0x523045='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=',_0x4f8a49=String(_0x32fbfd)['replace'](/=+$/,'');var _0x1171d4='';for(var _0x44920a=0x0,_0x2a30c5,_0x443b2f,_0xcdf142=0x0;_0x443b2f=_0x4f8a49['charAt'](_0xcdf142++);~_0x443b2f&&(_0x2a30c5=_0x44920a%0x4?_0x2a30c5*0x40+_0x443b2f:_0x443b2f,_0x44920a++%0x4)?_0x1171d4+=String['fromCharCode'](0xff&_0x2a30c5>>(-0x2*_0x44920a&0x6)):0x0){_0x443b2f=_0x523045['indexOf'](_0x443b2f);}return _0x1171d4;};_0x14f8['oZlYBE']=function(_0x8f2071){var _0x49af5e=_0x3f720f(_0x8f2071);var _0x52e65f=[];for(var _0x1ed1cf=0x0,_0x79942e=_0x49af5e['length'];_0x1ed1cf<_0x79942e;_0x1ed1cf++){_0x52e65f+='%'+('00'+_0x49af5e['charCodeAt'](_0x1ed1cf)['toString'](0x10))['slice'](-0x2);}return decodeURIComponent(_0x52e65f);},_0x14f8['qHtbNC']={},_0x14f8['eOTqeL']=!![];}var _0x20247c=_0x14f8['qHtbNC'][_0x13249d];return _0x20247c===undefined?(_0x14f83b=_0x14f8['oZlYBE'](_0x14f83b),_0x14f8['qHtbNC'][_0x13249d]=_0x14f83b):_0x14f83b=_0x20247c,_0x14f83b;};console[_0x14f8('0x0')](_0x14f8('0x1'));

```

Ce code est évidemment plus obscur, et nous ne pouvons voir aucun vestige de notre code d'origine. Nous pouvons maintenant essayer de l'exécuter dans [https://jsconsole.com](https://jsconsole.com/) pour vérifier qu'il remplit toujours sa fonction d'origine. Essayez de jouer avec les paramètres d'obscurcissement dans [https://obfuscator.io](https://obfuscator.io/) pour générer encore plus de code obscurci, puis essayez de le réexécuter dans [https://jsconsole.com](https://jsconsole.com/) pour vérifier qu'il remplit toujours sa fonction d'origine.

* * * * *

Plus d'obscurcissement
----------------------

Nous devrions maintenant avoir une idée claire du fonctionnement de l'obscurcissement du code. Il existe encore de nombreuses variantes d'outils d'obscurcissement de code, chacun d'entre eux obscurcissant le code différemment. Prenez le code JavaScript suivant, par exemple :

Code : javascript

```
[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(!
...SNIP...
[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]](!+[]+!+[]+[+[]])))()

```

Nous pouvons toujours exécuter ce code, et il remplirait toujours sa fonction d'origine :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_jsf.jpg)

Remarque : Le code ci-dessus a été extrait car le code complet est trop long, mais le code complet devrait s'exécuter avec succès.

Nous pouvons essayer d'obscurcir le code en utilisant le même outil dans [JSF](http://www.jsfuck.com/) , puis en le réexécutant. Nous remarquerons que l'exécution du code peut prendre un certain temps, ce qui montre comment l'obscurcissement du code peut affecter les performances, comme mentionné précédemment.

Il existe de nombreux autres obfuscateurs JavaScript, comme [JJ Encode](https://utf-8.jp/public/jjencode.html) ou [AA Encode](https://utf-8.jp/public/aaencode.html) . Cependant, ces obfuscateurs rendent généralement l'exécution/la compilation du code très lentes, il n'est donc pas recommandé de les utiliser sauf pour une raison évidente, comme le contournement des filtres ou des restrictions Web.

[Précédent](https://academy.hackthebox.com/module/41/section/447)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/41/section/442)

Aide-mémoire
