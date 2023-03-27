Contournement des filtres d'espace
=======================

* * * * *

Il existe de nombreuses façons de détecter les tentatives d'injection, et il existe plusieurs méthodes pour contourner ces détections. Nous démontrerons le concept de détection et comment fonctionne le contournement en utilisant Linux comme exemple. Nous apprendrons à utiliser ces contournements et, éventuellement, à les empêcher. Une fois qu'on a bien compris leur fonctionnement, on peut passer par différentes sources sur internet pour découvrir d'autres types de contournements et apprendre comment les mitiger.

* * * * *

Contourner les opérateurs sur liste noire
-------------------------------------

Nous verrons que la plupart des opérateurs d'injection sont en effet blacklistés. Cependant, le caractère de nouvelle ligne n'est généralement pas mis sur liste noire, car il peut être nécessaire dans la charge utile elle-même. Nous savons que le caractère de nouvelle ligne fonctionne en ajoutant nos commandes à la fois sous Linux et sous Windows, alors essayons de l'utiliser comme opérateur d'injection : 
![Filter Operator](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_operator.jpg)

Comme nous pouvons le voir, même si notre charge utile incluait un caractère de nouvelle ligne, notre demande n'a pas été refusée et nous avons obtenu le résultat de la commande ping, ce qui signifie que ce caractère n'est pas sur la liste noire et que nous pouvons l'utiliser comme notre opérateur d'injection ». Commençons par discuter de la manière de contourner un caractère couramment mis sur liste noire - un caractère espace.

* * * * *

Contourner les espaces sur liste noire
-------------------------

Maintenant que nous avons un opérateur d'injection fonctionnel, modifions notre charge utile d'origine et renvoyons-la sous la forme (`127.0.0.1%0a whoami`): ![Filter Space](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_spaces_1.jpg)

Comme nous pouvons le constater, nous recevons toujours un message d'erreur "entrée non valide", ce qui signifie que nous avons encore d'autres filtres à contourner. Donc, comme nous l'avons fait précédemment, ajoutons uniquement le caractère suivant (qui est un espace) et voyons s'il a causé la demande refusée : 
![Filtrer l'espace](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_spaces_2.jpg)

Comme nous pouvons le voir, le personnage spatial est en effet également sur liste noire. Un espace est un caractère généralement mis sur liste noire, surtout si l'entrée ne doit contenir aucun espace, comme une adresse IP, par exemple. Pourtant, il existe de nombreuses façons d'ajouter un caractère d'espacement sans utiliser le caractère d'espacement !

#### Utilisation des onglets

L'utilisation de tabulations (%09) au lieu d'espaces est une technique qui peut fonctionner, car Linux et Windows acceptent les commandes avec des tabulations entre les arguments, et elles sont exécutées de la même manière. Alors, essayons d'utiliser une tabulation au lieu du caractère d'espace (`127.0.0.1%0a%09`) et voyons si notre demande est acceptée : ![Filter Space](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_spaces_3.jpg)

Comme nous pouvons le voir, nous avons réussi à contourner le filtre de caractères d'espace en utilisant une tabulation à la place. Voyons une autre méthode de remplacement des espaces.

#### Utilisation de $IFS

L'utilisation de la variable d'environnement Linux ($IFS) peut également fonctionner puisque sa valeur par défaut est un espace et une tabulation, ce qui fonctionnerait entre les arguments de commande. Donc, si nous utilisons `${IFS}` à la place des espaces, la variable devrait être automatiquement remplacée par un espace et notre commande devrait fonctionner.

Utilisons `${IFS}` et voyons si cela fonctionne (`127.0.0.1%0a${IFS}`) : ![Filter Space](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_spaces_4.jpg)

Nous voyons que notre demande n'a pas été refusée cette fois-ci, et nous avons à nouveau contourné le filtre spatial.

#### Utilisation de l'expansion des accolades

Il existe de nombreuses autres méthodes que nous pouvons utiliser pour contourner les filtres spatiaux. Par exemple, nous pouvons utiliser la fonctionnalité `Bash Brace Expansion` , qui ajoute automatiquement des espaces entre les arguments entourés d'accolades, comme suit :

Utilisation de l'expansion de l'accolade

```
dsgsec@htb[/htb]$ {ls,-la}

totale 0
drwxr-xr-x 1 21y4d 21y4d 0 juil 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d 0 Juil 13 13:01 ..

```

Comme nous pouvons le voir, la commande a été exécutée avec succès sans qu'il y ait d'espaces. Nous pouvons utiliser la même méthode dans les contournements du filtre d'injection de commande, en utilisant l'expansion des accolades sur nos arguments de commande, comme (`127.0.0.1%0a{ls,-la}`). Pour découvrir d'autres contournements de filtre d'espace, consultez la page [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space) page sur l'écriture de commandes sans espaces.