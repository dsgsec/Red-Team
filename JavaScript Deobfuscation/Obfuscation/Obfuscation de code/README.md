Obfuscation de code
===================

* * * * *

Avant de commencer à apprendre sur `deobfuscation`, nous devons d'abord apprendre sur `code obfuscation`. Sans comprendre comment le code est obscurci, nous ne pourrons peut-être pas désobscurcir le code avec succès, surtout s'il a été obscurci à l'aide d'un obfuscateur personnalisé.

* * * * *

Qu'est-ce que l'obscurcissement
-------------------------------

L'obscurcissement est une technique utilisée pour rendre un script plus difficile à lire par les humains, mais lui permet de fonctionner de la même manière d'un point de vue technique, bien que les performances puissent être plus lentes. Ceci est généralement réalisé automatiquement en utilisant un outil d'obscurcissement, qui prend le code comme entrée et tente de réécrire le code d'une manière beaucoup plus difficile à lire, en fonction de sa conception.

Par exemple, les obfuscateurs de code transforment souvent le code en un dictionnaire de tous les mots et symboles utilisés dans le code, puis tentent de reconstruire le code d'origine lors de l'exécution en se référant à chaque mot et symbole du dictionnaire. Voici un exemple de code JavaScript simple masqué :

![](https://academy.hackthebox.com/storage/modules/41/obfuscation_example.jpg)

Les codes écrits dans de nombreuses langues sont publiés et exécutés sans être compilés dans `interpreted`des langues telles que `Python`, `PHP`et `JavaScript`. Alors que `Python`et `PHP`résident généralement côté serveur et sont donc masqués pour les utilisateurs finaux, `JavaScript`il est généralement utilisé dans les navigateurs au niveau du `client-side`, et le code est envoyé à l'utilisateur et exécuté en texte clair. C'est pourquoi l'obscurcissement est très souvent utilisé avec `JavaScript`.

* * * * *

Cas d'utilisation
-----------------

Il existe de nombreuses raisons pour lesquelles les développeurs peuvent envisager d'obscurcir leur code. Une raison courante est de masquer le code d'origine et ses fonctions pour éviter qu'il ne soit réutilisé ou copié sans l'autorisation du développeur, ce qui rend plus difficile la rétro-ingénierie de la fonctionnalité d'origine du code. Une autre raison est de fournir une couche de sécurité lorsqu'il s'agit d'authentification ou de cryptage pour empêcher les attaques sur les vulnérabilités qui peuvent être trouvées dans le code.

`It must be noted that doing authentication or encryption on the client-side is not recommended, as code is more prone to attacks this way.`

Cependant, l'utilisation la plus courante de l'obscurcissement concerne les actions malveillantes. Il est courant que les attaquants et les acteurs malveillants masquent leurs scripts malveillants pour empêcher les systèmes de détection et de prévention des intrusions de détecter leurs scripts. Dans la section suivante, nous apprendrons comment obscurcir un code JavaScript simple et tenterons de l'exécuter avant et après l'obscurcissement pour noter les différences.

[Précédent](https://academy.hackthebox.com/module/41/section/441)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/41/section/447)

Aide-mémoire
