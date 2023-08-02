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

Requêtes HTTP
=============

* * * * *

Dans la section précédente, nous avons découvert que la `secret.js`fonction principale envoie une `POST`requête vide à `/serial.php`. Dans cette section, nous tenterons de faire de même en utilisant `cURL`pour envoyer une `POST`requête à `/serial.php`. Pour en savoir plus sur `cURL`les requêtes Web, vous pouvez consulter le module [Requêtes Web](https://academy.hackthebox.com/module/details/35) .

* * * * *

boucle
------

`cURL`est un puissant outil de ligne de commande utilisé dans les distributions Linux, macOS et même les dernières versions de Windows PowerShell. Nous pouvons demander n'importe quel site Web en fournissant simplement son URL, et nous l'obtiendrons au format texte, comme suit :

```
dsgsec@htb[/htb]$ curl http://SERVER_IP:PORT/

</html>
<!DOCTYPE html>

<head>
    <title>Secret Serial Generator</title>
    <style>
        *,
        html {
            margin: 0;
            padding: 0;
            border: 0;
...SNIP...
        <h1>Secret Serial Generator</h1>
        <p>This page generates secret serials!</p>
    </div>
</body>

</html>

```

C'est la même chose `HTML`que nous avons vécue lorsque nous avons vérifié le code source dans la première section.

* * * * *

Demande POST
------------

Pour envoyer une `POST`requête, nous devons ajouter le `-X POST`drapeau à notre commande, et elle doit envoyer une `POST`requête :

```
dsgsec@htb[/htb]$ curl -s http://SERVER_IP:PORT/ -X POST

```

Conseil : Nous ajoutons l'indicateur "-s" pour réduire l'encombrement de la réponse avec des données inutiles

Cependant, `POST`la demande contient généralement `POST`des données. Pour envoyer des données, nous pouvons utiliser le `-d "param1=sample"`drapeau " " et inclure nos données pour chaque paramètre, comme suit :

```
dsgsec@htb[/htb]$ curl -s http://SERVER_IP:PORT/ -X POST -d "param1=sample"

```

Maintenant que nous savons utiliser `cURL`pour envoyer `POST`des requêtes de base, dans la section suivante, nous l'utiliserons pour reproduire ce qui `server.js`se fait afin de mieux comprendre son objectif.

Décodage
========

* * * * *

Après avoir fait l'exercice de la section précédente, nous avons obtenu un étrange bloc de texte qui semble être encodé :

```
dsgsec@htb[/htb]$ curl http:/SERVER_IP:PORT/serial.php -X POST -d "param1=sample"

ZG8gdGhlIGV4ZXJjaXNlLCBkb24ndCBjb3B5IGFuZCBwYXN0ZSA7KQo=

```

C'est un autre aspect important de l'obscurcissement auquel nous avons fait référence `More Obfuscation`dans la `Advanced Obfuscation`section. De nombreuses techniques peuvent encore obscurcir le code et le rendre moins lisible par les humains et moins détectable par les systèmes. Pour cette raison, vous trouverez très souvent du code obscurci contenant des blocs de texte encodés qui sont décodés lors de l'exécution. Nous couvrirons 3 des méthodes d'encodage de texte les plus couramment utilisées :

-   `base64`
-   `hex`
-   `rot13`

* * * * *

Base64
------

`base64`Le codage est généralement utilisé pour réduire l'utilisation de caractères spéciaux, car tous les caractères codés dans `base64`seraient représentés en caractères alphanumériques, en plus de `+`et `/`uniquement. Quelle que soit l'entrée, même si elle est au format binaire, la chaîne encodée en base64 résultante ne les utiliserait que.

#### Base de repérage64

`base64`les chaînes encodées sont facilement repérables puisqu'elles ne contiennent que des caractères alphanumériques. Cependant, la caractéristique la plus distinctive de `base64`est son remplissage à l'aide `=`de caractères. La longueur des `base64`chaînes encodées doit être un multiple de 4. Si la sortie résultante ne contient que 3 caractères, par exemple, un supplément `=`est ajouté comme remplissage, et ainsi de suite.

#### Encodage Base64

Pour encoder n'importe quel texte sous `base64`Linux, nous pouvons lui faire un écho et le diriger avec ' `|`' vers `base64`:

  Encodage Base64

```
dsgsec@htb[/htb]$ echo https://www.hackthebox.eu/ | base64

aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K

```

#### Décodage Base64

Si nous voulons décoder n'importe quelle `base64`chaîne encodée, nous pouvons utiliser `base64 -d`, comme suit :

  Décodage Base64

```
dsgsec@htb[/htb]$ echo aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K | base64 -d

https://www.hackthebox.eu/

```

* * * * *

Hexagone
--------

Une autre méthode d'encodage courante est `hex`l'encodage, qui encode chaque caractère dans son `hex`ordre dans la `ASCII`table. Par exemple, `a`est `61`en hexadécimal, `b`est `62`, `c`est `63`, etc. Vous pouvez trouver la `ASCII`table complète sous Linux en utilisant la `man ascii`commande.

#### Hexagone de repérage

Toute chaîne encodée dans `hex`serait composée de caractères hexadécimaux uniquement, qui ne sont que de 16 caractères : 0-9 et af. Cela rend le repérage `hex`des chaînes encodées aussi simple que le repérage `base64`des chaînes encodées.

#### Encodage hexadécimal

Pour encoder n'importe quelle chaîne `hex`dans Linux, nous pouvons utiliser la `xxd -p`commande :

  Encodage hexadécimal

```
dsgsec@htb[/htb]$ echo https://www.hackthebox.eu/ | xxd -p

68747470733a2f2f7777772e6861636b746865626f782e65752f0a

```

#### Décodage hexadécimal

Pour décoder une `hex`chaîne encodée, nous pouvons utiliser la `xxd -p -r`commande :

  Décodage hexadécimal

```
dsgsec@htb[/htb]$ echo 68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r

https://www.hackthebox.eu/

```

* * * * *

César/Pourriture13
------------------

Une autre technique de codage courante - et très ancienne - est un chiffrement de César, qui décale chaque lettre d'un nombre fixe. Par exemple, un décalage d'un caractère fait `a`devenir `b`, et `b`devient `c`, et ainsi de suite. De nombreuses variantes du chiffrement de César utilisent un nombre différent de décalages, dont le plus courant est `rot13`, qui décale chaque caractère 13 fois vers l'avant.

#### Repérer César/Pourriture13

Même si cette méthode d'encodage rend tout texte aléatoire, il est toujours possible de le repérer car chaque caractère est mappé sur un caractère spécifique. Par exemple, dans `rot13`, `http://www`devient `uggc://jjj`, qui a encore quelques ressemblances et peut être reconnu comme tel.

#### Encodage Rot13

Il n'y a pas de commande spécifique sous Linux pour effectuer `rot13`l'encodage. Cependant, il est assez facile de créer notre propre commande pour effectuer le changement de caractère :

  Encodage Rot13

```
dsgsec@htb[/htb]$ echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'

uggcf://jjj.unpxgurobk.rh/

```

#### Décodage Rot13

Nous pouvons également utiliser la même commande précédente pour décoder rot13 :

  Décodage Rot13

```
dsgsec@htb[/htb]$ echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'

https://www.hackthebox.eu/

```

Une autre option pour encoder/décoder rot13 serait d'utiliser un outil en ligne, comme [rot13](https://rot13.com/) .

* * * * *

Autres types d'encodage
-----------------------

Il existe des centaines d'autres méthodes d'encodage que nous pouvons trouver en ligne. Même si ce sont les plus courantes, nous rencontrerons parfois d'autres méthodes d'encodage, qui peuvent nécessiter une certaine expérience pour les identifier et les décoder.

`If you face any similar types of encoding, first try to determine the type of encoding, and then look for online tools to decode it.`

Certains outils peuvent nous aider à déterminer automatiquement le type d'encodage, comme [Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier) . Essayez les chaînes encodées ci-dessus avec [Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier) , pour voir s'il peut identifier correctement la méthode d'encodage.

Outre l'encodage, de nombreux outils d'obscurcissement utilisent le chiffrement, qui encode une chaîne à l'aide d'une clé, ce qui peut rendre le code obscurci très difficile à désosser et à désobscurcir, surtout si la clé de déchiffrement n'est pas stockée dans le script lui-même.
