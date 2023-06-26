  Contournements de base
======================

Dans la section précédente, nous avons vu plusieurs types d'attaques que nous pouvons utiliser pour différents types de vulnérabilités LFI. Dans de nombreux cas, nous pouvons être confrontés à une application Web qui applique diverses protections contre l'inclusion de fichiers, de sorte que nos charges utiles LFI normales ne fonctionneraient pas. Néanmoins, à moins que l'application Web ne soit correctement sécurisée contre les entrées d'utilisateurs LFI malveillantes, nous pourrons peut-être contourner les protections en place et atteindre l'inclusion de fichiers.

* * * * *

Filtres de traversée de chemin non récursifs
--------------------------------------------

L'un des filtres les plus basiques contre LFI est un filtre de recherche et de remplacement, où il supprime simplement les sous-chaînes de ( `../`) pour éviter les traversées de chemin. Par exemple:

Code : php

```
$language = str_replace('../', '', $_GET['language']);

```

Le code ci-dessus est censé empêcher la traversée du chemin et rend donc LFI inutile. Si nous essayons les charges utiles LFI que nous avons essayées dans la section précédente, nous obtenons ce qui suit :

![](https://academy.hackthebox.com/storage/modules/23/lfi_blacklist.png)

Nous voyons que toutes `../`les sous-chaînes ont été supprimées, ce qui a abouti à un chemin final étant `./languages/etc/passwd`. Cependant, ce filtre est très peu sûr, car ce n'est pas `recursively removing`la `../`sous-chaîne, car il s'exécute une seule fois sur la chaîne d'entrée et n'applique pas le filtre sur la chaîne de sortie. Par exemple, si nous utilisons `....//`comme charge utile, le filtre serait supprimé `../`et la chaîne de sortie serait `../`, ce qui signifie que nous pouvons toujours effectuer une traversée de chemin. Essayons d'appliquer cette logique pour inclure `/etc/passwd`à nouveau :

![](https://academy.hackthebox.com/storage/modules/23/lfi_blacklist_passwd.png)

Comme nous pouvons le voir, l'inclusion a réussi cette fois, et nous sommes capables de lire `/etc/passwd`avec succès. La `....//`sous-chaîne n'est pas le seul contournement que nous pouvons utiliser, car nous pouvons utiliser `..././`ou `....\/`et plusieurs autres charges utiles LFI récursives. De plus, dans certains cas, échapper le caractère de barre oblique peut également fonctionner pour éviter les filtres de traversée de chemin (par exemple `....\/`) ou ajouter des barres obliques supplémentaires (par exemple `....////`)

* * * * *

Codage
------

Certains filtres Web peuvent empêcher les filtres d'entrée qui incluent certains caractères liés à LFI, comme un point `.`ou une barre oblique `/`utilisés pour les traversées de chemin. Cependant, certains de ces filtres peuvent être contournés par l'encodage URL de notre entrée, de sorte qu'elle n'inclura plus ces mauvais caractères, mais sera toujours décodée dans notre chaîne de traversée de chemin une fois qu'elle aura atteint la fonction vulnérable. Les filtres PHP principaux des versions 5.3.4 et antérieures étaient particulièrement vulnérables à ce contournement, mais même sur les versions plus récentes, nous pouvons trouver des filtres personnalisés qui peuvent être contournés via l'encodage d'URL.

Si l'application Web cible n'autorisait pas `.`et `/`dans notre entrée, nous pouvons coder l'URL `../`en `%2e%2e%2f`, ce qui peut contourner le filtre. Pour ce faire, nous pouvons utiliser n'importe quel utilitaire d'encodage d'URL en ligne ou utiliser l'outil Burp Suite Decoder, comme suit : ![burp_url_encode](https://academy.hackthebox.com/storage/modules/23/burp_url_encode.jpg)

Remarque : Pour que cela fonctionne, nous devons coder l'URL de tous les caractères, y compris les points. Certains encodeurs d'URL peuvent ne pas encoder les points car ils sont considérés comme faisant partie du schéma d'URL.

Essayons d'utiliser cette charge utile LFI codée par rapport à notre application Web vulnérable précédente qui filtre `../`les chaînes :

![](https://academy.hackthebox.com/storage/modules/23/lfi_blacklist_passwd_filter.png)

Comme nous pouvons le voir, nous avons également réussi à contourner le filtre et à utiliser la traversée de chemin pour lire les fichiers `/etc/passwd`. De plus, nous pouvons également utiliser Burp Decoder pour encoder à nouveau la chaîne encodée afin d'avoir une `double encoded`chaîne, qui peut également contourner d'autres types de filtres.

Vous pouvez vous référer au module [Command Injections](https://academy.hackthebox.com/module/details/109) pour en savoir plus sur le contournement de divers caractères sur liste noire, car les mêmes techniques peuvent également être utilisées avec LFI.

* * * * *

Chemins approuvés
-----------------

Certaines applications Web peuvent également utiliser des expressions régulières pour s'assurer que le fichier inclus se trouve sous un chemin spécifique. Par exemple, l'application Web avec laquelle nous avons travaillé ne peut accepter que les chemins qui se trouvent sous le `./languages`répertoire, comme suit :

Code : php

```
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}

```

Pour trouver le chemin approuvé, nous pouvons examiner les demandes envoyées par les formulaires existants et voir quel chemin ils utilisent pour la fonctionnalité Web normale. De plus, nous pouvons fuzzer des répertoires Web sous le même chemin et en essayer différents jusqu'à ce que nous obtenions une correspondance. Pour contourner cela, nous pouvons utiliser la traversée de chemin et démarrer notre charge utile avec le chemin approuvé, puis utiliser `../`pour revenir au répertoire racine et lire le fichier que nous spécifions, comme suit :

![](https://academy.hackthebox.com/storage/modules/23/lfi_blacklist_passwd_filter.png)

Certaines applications Web peuvent appliquer ce filtre avec l'un des filtres précédents, nous pouvons donc combiner les deux techniques en démarrant notre charge utile avec le chemin approuvé, puis encoder en URL notre charge utile ou utiliser une charge utile récursive.

Remarque : Toutes les techniques mentionnées jusqu'à présent devraient fonctionner avec n'importe quelle vulnérabilité LFI, quel que soit le langage ou le framework de développement back-end.

* * * * *

Extension ajoutée
-----------------

Comme indiqué dans la section précédente, certaines applications Web ajoutent une extension à notre chaîne d'entrée (par exemple `.php`), pour s'assurer que le fichier que nous incluons est dans l'extension attendue. Avec les versions modernes de PHP, nous ne pourrons peut-être pas contourner cela et nous serons limités à lire uniquement les fichiers de cette extension, ce qui peut toujours être utile, comme nous le verrons dans la section suivante (par exemple pour lire le code source).

Il existe quelques autres techniques que nous pouvons utiliser, mais elles le sont `obsolete with modern versions of PHP and only work with PHP versions before 5.3/5.4`. Cependant, il peut toujours être avantageux de les mentionner, car certaines applications Web peuvent encore fonctionner sur des serveurs plus anciens, et ces techniques peuvent être les seuls contournements possibles.

#### Troncature de chemin

Dans les versions antérieures de PHP, les chaînes définies ont une longueur maximale de 4096 caractères, probablement en raison de la limitation des systèmes 32 bits. Si une chaîne plus longue est passée, ce sera simplement `truncated`, et tous les caractères après la longueur maximale seront ignorés. De plus, PHP avait également l'habitude de supprimer les barres obliques et les points simples dans les noms de chemin, donc si nous appelons ( `/etc/passwd/.`) alors le `/.`serait également tronqué, et PHP appellerait ( `/etc/passwd`). PHP, et les systèmes Linux en général, ignorent également les multiples barres obliques dans le chemin (par exemple, `////etc/passwd`est identique à `/etc/passwd`). De même, un raccourci vers le répertoire courant ( `.`) au milieu du chemin serait également ignoré (par exemple `/etc/./passwd`).

Si nous combinons ces deux limitations PHP, nous pouvons créer de très longues chaînes qui correspondent à un chemin correct. Chaque fois que nous atteignons la limite de 4096 caractères, l'extension ajoutée ( `.php`) serait tronquée et nous aurions un chemin sans extension ajoutée. Enfin, il est également important de noter que nous aurions également besoin `start the path with a non-existing directory`pour que cette technique fonctionne.

Un exemple d'une telle charge utile serait le suivant :

Code : url

```
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]

```

Bien sûr, nous n'avons pas à taper manuellement `./`2048 fois (total de 4096 caractères), mais nous pouvons automatiser la création de cette chaîne avec la commande suivante :

  Troncature de chemin

```
dsgsec@htb[/htb]$ echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
non_existing_directory/../../../etc/passwd/./././<SNIP>././././

```

Nous pouvons également augmenter le nombre de `../`, car en ajouter davantage nous amènerait toujours dans le répertoire racine, comme expliqué dans la section précédente. Cependant, si nous utilisons cette méthode, nous devons calculer la longueur totale de la chaîne pour nous assurer que seul le `.php`fichier est tronqué et non notre fichier demandé à la fin de la chaîne ( `/etc/passwd`). C'est pourquoi il serait plus simple d'utiliser la première méthode.

#### Octets nuls

Les versions de PHP antérieures à la 5.5 étaient vulnérables à `null byte injection`, ce qui signifie que l'ajout d'un octet nul ( `%00`) à la fin de la chaîne terminerait la chaîne et ne considérerait rien après. Cela est dû à la façon dont les chaînes sont stockées dans la mémoire de bas niveau, où les chaînes en mémoire doivent utiliser un octet nul pour indiquer la fin de la chaîne, comme on le voit dans les langages Assembly, C ou C++.

Pour exploiter cette vulnérabilité, nous pouvons terminer notre charge utile avec un octet nul (par exemple `/etc/passwd%00`), de sorte que le chemin final transmis `include()`soit ( `/etc/passwd%00.php`). De cette façon, même si `.php`est ajouté à notre chaîne, tout ce qui suit l'octet nul serait tronqué, et donc le chemin utilisé serait en fait `/etc/passwd`, ce qui nous amènerait à contourner l'extension ajoutée.
