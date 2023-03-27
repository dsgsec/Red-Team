Identification des filtres
===================

* * * * *

Comme nous l'avons vu dans la section précédente, même si les développeurs tentent de sécuriser l'application Web contre les injections, elle peut toujours être exploitable si elle n'a pas été codée de manière sécurisée. Un autre type d'atténuation de l'injection consiste à utiliser des caractères et des mots sur liste noire sur le back-end pour détecter les tentatives d'injection et refuser la demande si une demande les contenait. Une autre couche supplémentaire consiste à utiliser des pare-feu d'applications Web (WAF), qui peuvent avoir une portée plus large et diverses méthodes de détection d'injection et empêcher diverses autres attaques telles que les injections SQL ou les attaques XSS.

Cette section examinera quelques exemples de la façon dont les injections de commandes peuvent être détectées et bloquées et comment nous pouvons identifier ce qui est bloqué.

* * * * *

Détection de filtre/WAF
--------------------

Commençons par visiter l'application Web dans l'exercice à la fin de cette section. Nous voyons la même application Web "Host Checker" que nous avons exploitée, mais elle a maintenant quelques mesures d'atténuation dans sa manche. Nous pouvons voir que si nous essayons les opérateurs précédents que nous avons testés, comme (`;`, `&&`, `||`), nous obtenons le message d'erreur `invalid input` : ![Filter](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_1.jpg)

Cela indique que quelque chose que nous avons envoyé a déclenché un mécanisme de sécurité en place qui a refusé notre demande. Ce message d'erreur peut être affiché de différentes manières. Dans ce cas, nous le voyons dans le champ où la sortie est affichée, ce qui signifie qu'elle a été détectée et empêchée par l'application Web `PHP` elle-même. `Si le message d'erreur affichait une page différente, avec des informations comme notre adresse IP et notre demande, cela peut indiquer qu'elle a été refusée par un WAF`.

Vérifions la charge utile que nous avons envoyée :

Code : bash

```
127.0.0.1 ; whoami

```

Outre l'IP (dont nous savons qu'elle n'est pas sur liste noire), nous avons envoyé :

1. Un point-virgule `;`
2. Un caractère espace
3. Une commande `whoami` 

Ainsi, l'application Web `a détecté un caractère sur liste noire` ou `a détecté une commande sur liste noire`, ou les deux. Alors, voyons comment contourner chacun.

* * * * *

Personnages sur liste noire
----------------------

Une application Web peut avoir une liste de caractères sur liste noire, et si la commande les contient, elle refusera la demande. Le code `PHP` peut ressembler à ceci :

Code : php

```
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}

```

Si un caractère de la chaîne que nous avons envoyée correspond à un caractère de la liste noire, notre demande est refusée. Avant de commencer nos tentatives de contournement du filtre, nous devrions essayer d'identifier quel personnage a causé la demande refusée.

* * * * *

Identification d'un personnage sur liste noire
---------------------------------

Réduisons notre demande à un caractère à la fois et voyons quand il est bloqué. Nous savons que la charge utile (`127.0.0.1`) fonctionne, alors commençons par ajouter le point-virgule (`127.0.0.1;`): 

![Filter Character](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_2.jpg)


Nous obtenons toujours une `invalid input`, une erreur signifiant qu'un point-virgule est sur la liste noire. Voyons donc si tous les opérateurs d'injection dont nous avons parlé précédemment sont sur la liste noire.