Analyse de code
===============

* * * * *

Maintenant que nous avons désobscurci le code, nous pouvons commencer à le parcourir :

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

On voit que le `secret.js`fichier ne contient qu'une seule fonction, `generateSerial`.

* * * * *

Requêtes HTTP
-------------

Examinons chaque ligne de la `generateSerial`fonction.

#### Variables codées

La fonction commence par définir une variable `xhr`, qui crée un objet de `XMLHttpRequest`. Comme nous ne savons peut-être pas exactement ce `XMLHttpRequest`que fait JavaScript, laissez-nous Google `XMLHttpRequest`pour voir à quoi il sert.\
Après avoir lu à ce sujet, nous voyons qu'il s'agit d'une fonction JavaScript qui gère les requêtes Web.

La deuxième variable définie est la `URL`variable qui contient une URL vers `/serial.php`, qui doit être sur le même domaine, car aucun domaine n'a été spécifié.

#### Fonctions des codes

Ensuite, nous voyons que `xhr.open`est utilisé avec `"POST"`et `URL`. Nous pouvons Google cette fonction une fois de plus, et nous voyons qu'elle ouvre la requête HTTP définie ' `GET`ou `POST`' au `URL`, puis la ligne suivante `xhr.send`enverrait la requête.

Ainsi, tout `generateSerial`ce qui se passe est d'envoyer simplement une `POST`requête à `/serial.php`, sans inclure aucune `POST`donnée ni rien récupérer en retour.

Les développeurs peuvent avoir implémenté cette fonction chaque fois qu'ils ont besoin de générer une série, comme en cliquant sur un certain `Generate Serial`bouton, par exemple. Cependant, comme nous n'avons pas vu d'éléments HTML similaires générant des publications en série, les développeurs ne doivent pas encore avoir utilisé cette fonction et l'ont conservée pour une utilisation future.

Grâce à l'utilisation de la désobfuscation et de l'analyse du code, nous avons pu découvrir cette fonction. Nous pouvons maintenant tenter de répliquer sa fonctionnalité pour voir si elle est gérée côté serveur lors de l'envoi d'une `POST`requête. Si la fonction est activée et gérée côté serveur, nous pouvons découvrir une fonctionnalité non publiée, qui a généralement tendance à contenir des bogues et des vulnérabilités.

[Précédent](https://academy.hackthebox.com/module/41/section/442)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/41/section/444)

Aide-mémoire
