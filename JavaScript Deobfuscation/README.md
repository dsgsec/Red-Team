Introduction
============

* * * * *

Bienvenue dans le module de désobscurcissement JavaScript !

La désobfuscation de code est une compétence importante à acquérir si nous voulons être compétents en analyse de code et en ingénierie inverse. Au cours des exercices de l'équipe rouge/bleue, nous rencontrons souvent du code obscurci qui souhaite masquer certaines fonctionnalités, comme des logiciels malveillants qui utilisent du code JavaScript obscurci pour récupérer sa charge utile principale. Sans comprendre ce que fait ce code, nous ne savons peut-être pas exactement ce que fait le code et, par conséquent, nous ne pourrons peut-être pas terminer l'exercice d'équipe rouge/bleu.

Dans ce module, nous commençons par apprendre la structure générale d'une page HTML, puis nous y localisons le code JavaScript. Une fois que nous aurons fait cela, nous apprendrons ce qu'est l'obscurcissement, comment il est fait et où il est utilisé et suivrons cela en apprenant comment désobscurcir un tel code. Une fois le code désobscurci, nous tenterons de comprendre son utilisation générale pour répliquer ses fonctionnalités et découvrir ce qu'il fait manuellement.

Les sujets suivants seront abordés :

-   Localisation du code JavaScript
-   Introduction à l'obscurcissement du code
-   Comment désobscurcir le code JavaScript
-   Comment décoder les messages codés
-   Analyse de base du code
-   Envoi de requêtes HTTP de base

Code source
===========

* * * * *

De nos jours, la plupart des sites Web utilisent JavaScript pour exécuter leurs fonctions. Tandis qu'il `HTML`est utilisé pour déterminer les principaux champs et paramètres du site Web, et `CSS`est utilisé pour déterminer sa conception, `JavaScript`est utilisé pour exécuter toutes les fonctions nécessaires au fonctionnement du site Web. Cela se produit en arrière-plan, et nous ne voyons que le joli front-end du site Web et interagissons avec lui.

Même si tout ce code source est disponible côté client, il est rendu par nos navigateurs, nous ne prêtons donc pas souvent attention au code source HTML. Cependant, si nous voulions comprendre les fonctionnalités côté client d'une certaine page, nous commençons généralement par examiner le code source de la page. Cette section montrera comment nous pouvons découvrir le code source qui contient tout cela et comprendre son utilisation générale.

* * * * *

HTML
----

Nous allons commencer par lancer l'exercice ci-dessous, ouvrir Firefox dans notre PwnBox, et visiter l'url indiquée dans la question :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_mainsite.jpg)

Comme nous pouvons le voir, le site Web indique `Secret Serial Generator`, sans avoir de champs de saisie ni montrer de fonctionnalité claire. Donc, notre prochaine étape consiste à atteindre son code source. Nous pouvons le faire en appuyant sur `[CTRL + U]`, ce qui devrait ouvrir la vue source du site Web :

![](https://academy.hackthebox.com/storage/modules/41/js_deobf_mainsite_source_1.jpg)

Comme nous pouvons le voir, nous pouvons voir le `HTML`code source du site Web.

* * * * *

CSS
---

`CSS`code est soit défini `internally`dans le même `HTML`fichier entre `<style>`les éléments, soit défini `externally`dans un `.css`fichier séparé et référencé dans le `HTML`code.

Dans ce cas, nous voyons que le `CSS`est défini en interne, comme le montre l'extrait de code ci-dessous :

Code : html

```
    <style>
        *,
        html {
            margin: 0;
            padding: 0;
            border: 0;
        }
        ...SNIP...
        h1 {
            font-size: 144px;
        }
        p {
            font-size: 64px;
        }
    </style>

```

Si un `CSS`style de page est défini en externe, le fichier externe `.css`est référencé avec la `<link>`balise dans l'en-tête HTML, comme suit :

Code : html

```
<head>
    <link rel="stylesheet" href="style.css">
</head>

```

* * * * *

Javascript
----------

Le même concept s'applique à `JavaScript`. Il peut être écrit en interne entre `<script>`les éléments ou écrit dans un `.js`fichier séparé et référencé dans le `HTML`code.

Nous pouvons voir dans notre `HTML`source que le `.js`fichier est référencé en externe :

Code : html

```
<script src="secret.js"></script>

```

Nous pouvons vérifier le script en cliquant sur `secret.js`, ce qui devrait nous emmener directement dans le script. Quand on le visite, on voit que le code est très compliqué et incompréhensible :

Code : javascript

```
eval(function (p, a, c, k, e, d) { e = function (c) { '...SNIP... |true|function'.split('|'), 0, {}))

```

La raison derrière cela est `code obfuscation`. Qu'est-ce que c'est? Comment est-il fait? Où est-il utilisé ?
