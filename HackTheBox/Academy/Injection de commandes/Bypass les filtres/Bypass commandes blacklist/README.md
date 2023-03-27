Passer les commandes sur liste noire
=============================

* * * * *

Nous avons discuté de diverses méthodes pour contourner les filtres à un seul caractère. Cependant, il existe différentes méthodes pour contourner les commandes sur liste noire. Une liste noire de commandes se compose généralement d'un ensemble de mots, et si nous pouvons obscurcir nos commandes et leur donner un aspect différent, nous pourrons peut-être contourner les filtres.

Il existe différentes méthodes d'obscurcissement des commandes dont la complexité varie, comme nous le verrons plus tard avec les outils d'obscurcissement des commandes. Nous couvrirons quelques techniques de base qui peuvent nous permettre de changer l'apparence de notre commande pour contourner les filtres manuellement.

* * * * *

Liste noire des commandes
------------------

Jusqu'à présent, nous avons réussi à contourner le filtre de caractères pour les espaces et les points-virgules dans notre charge utile. Revenons donc à notre toute première charge utile et rajoutons la commande `whoami` pour voir si elle est exécutée : ![Filter Commands](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_commands_1 .jpg)

Nous voyons que même si nous avons utilisé des caractères qui ne sont pas bloqués par l'application Web, la requête est à nouveau bloquée une fois que nous avons ajouté notre commande. Cela est probablement dû à un autre type de filtre, qui est un filtre de liste noire de commandes.

Un filtre de liste noire de commande de base dans `PHP` ressemblerait à ceci :

Code : php

```
$listenoire = ['whoami', 'cat', ...SNIP...];
foreach ($liste noire comme $mot) {
     if (strpos('$_POST['ip']', $word) !== false) {
         echo "Entrée invalide" ;
     }
}

```

Comme nous pouvons le voir, il vérifie chaque mot de l'entrée de l'utilisateur pour voir s'il correspond à l'un des mots de la liste noire. Cependant, ce code recherche une correspondance exacte de la commande fournie, donc si nous envoyons une commande légèrement différente, il se peut qu'elle ne soit pas bloquée. Heureusement, nous pouvons utiliser diverses techniques d'obscurcissement qui exécuteront notre commande sans utiliser le mot de commande exact.

* * * * *

Linux et Windows
---------------

Une technique d'obscurcissement très courante et facile consiste à insérer certains caractères dans notre commande qui sont généralement ignorés par les shells de commande tels que `Bash` ou `PowerShell` et exécuteront la même commande que s'ils n'étaient pas là. Certains de ces caractères sont un guillemet simple `'` et un guillemet double `"`, en plus de quelques autres.

Les guillemets sont les plus faciles à utiliser et fonctionnent à la fois sur les serveurs Linux et Windows. Par exemple, si nous voulons masquer la commande `whoami` , nous pouvons insérer des guillemets simples entre ses caractères, comme suit :

```
21y4d@htb[/htb]$ w'h'o'am'i

21a4j

```

La même chose fonctionne également avec les guillemets doubles :

```
21y4d@htb[/htb]$ w"h"o"am"i

21a4j

```

Les points importants à retenir sont que `nous ne pouvons pas mélanger les types de guillemets` et `le nombre de guillemets doit être pair`. Nous pouvons essayer l'une des solutions ci-dessus dans notre charge utile (`127.0.0.1%0aw'h'o'am'i`) et voir si cela fonctionne :

#### Burp POST Demande

![Commandes de filtrage](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_commands_2.jpg)

Comme nous pouvons le voir, cette méthode fonctionne en effet.

* * * * *

Linux uniquement
----------

Nous pouvons insérer quelques autres caractères Linux uniquement au milieu des commandes, et le shell "bash" les ignorera et exécutera la commande. Ces caractères incluent la barre oblique inverse `\` et le caractère de paramètre positionnel `$@`. Cela fonctionne exactement comme avec les guillemets, mais dans ce cas, "le nombre de caractères n'a pas besoin d'être pair", et nous pouvons en insérer un seul si nous le souhaitons :

Code : bash

```
who$@ami
w\ho\am\i
```

Exercice : Essayez les deux exemples ci-dessus dans votre charge utile et voyez s'ils fonctionnent en contournant le filtre de commande. Si ce n'est pas le cas, cela peut indiquer que vous avez peut-être utilisé un caractère filtré. Seriez-vous capable de contourner cela également, en utilisant les techniques que nous avons apprises dans la section précédente ?

* * * * *

Windows seulement
------------

Il existe également des caractères Windows uniquement que nous pouvons insérer au milieu des commandes qui n'affectent pas le résultat, comme un signe d'insertion (`^`), comme nous pouvons le voir dans l'exemple suivant :

Burp POST Demande

```
C:\htb> who^ami

21a4j

```