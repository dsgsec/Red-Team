Détection
=========

* * * * *

Le processus de détection des vulnérabilités de base de l'injection de commande du système d'exploitation est le même que celui utilisé pour exploiter ces vulnérabilités. Nous essayons d'ajouter notre commande à travers diverses méthodes d'injection. Si la sortie de la commande change par rapport au résultat habituel prévu, nous avons exploité avec succès la vulnérabilité. Cela peut ne pas être vrai pour les vulnérabilités d'injection de commandes plus avancées, car nous pouvons utiliser diverses méthodes de fuzzing ou des revues de code pour identifier les vulnérabilités potentielles d'injection de commandes. Nous pouvons ensuite construire progressivement notre charge utile jusqu'à ce que nous obtenions l'injection de commande. Ce module se concentrera sur les injections de commandes de base, où nous contrôlons l'entrée utilisateur qui est directement utilisée dans une commande système exécutant une fonction sans aucune désinfection.

Pour le démontrer, nous utiliserons l'exercice qui se trouve à la fin de cette section.

* * * * *

Détection d'injection de commande
---------------------------

Lorsque nous visitons l'application Web dans l'exercice ci-dessous, nous voyons un utilitaire `Host Checker` qui semble nous demander une adresse IP pour vérifier si elle est active ou non : 
![Exercice de base](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_exercise_1.jpg)

Nous pouvons essayer d'entrer l'adresse IP de l'hôte local `127.0.0.1` pour vérifier la fonctionnalité et, comme prévu, la sortie de la commande `ping` nous indique que l'hôte local est bien actif : 
![Exercice de base](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_exercise_2.jpg)

Bien que nous n'ayons pas accès au code source de l'application Web, nous pouvons deviner avec certitude que l'adresse IP que nous avons saisie va dans une commande `ping` puisque la sortie que nous recevons le suggère. Comme le résultat montre un seul paquet transmis dans la commande ping, la commande utilisée peut être la suivante :

Code : bash

```
ping -c 1 NOTRE_ENTREE

```

Si notre entrée n'est pas filtrée et échappée avant d'être utilisée avec la commande `ping` , nous pourrons peut-être injecter une autre commande arbitraire. Essayons donc de voir si l'application Web est vulnérable à l'injection de commandes du système d'exploitation.

* * * * *

Méthodes d'injection de commandes
-------------------------

Pour injecter une commande supplémentaire à celle prévue, nous pouvons utiliser l'un des opérateurs suivants :

| Injection Operator | Injection Character | URL-Encoded Character | Executed Command |
| --- | --- | --- | --- |
| Semicolon | `;` | `%3b` | Both |
| New Line | `\n` | `%0a` | Both |
| Background | `&` | `%26` | Both (second output generally shown first) |
| Pipe | `|` | `%7c` | Both (only second output is shown) |
| AND | `&&` | `%26%26` | Both (only if first succeeds) |
| OR | `||` | `%7c%7c` | Second (only if first fails) |
| Sub-Shell | ```` | `%60%60` | Both (Linux-only) |
| Sub-Shell | `$()` | `%24%28%29` | Both (Linux-only) |

Nous pouvons utiliser n'importe lequel de ces opérateurs pour injecter une autre commande afin que `les deux` ou `l'une ou l'autre` des commandes soient exécutées. `Nous écrirons notre entrée attendue (par exemple, une adresse IP), puis utiliserons l'un des opérateurs ci-dessus, puis écrirons notre nouvelle commande.`

Astuce : En plus de ce qui précède, il existe quelques opérateurs UNIX uniquement, qui fonctionneraient sur Linux et macOS, mais ne fonctionneraient pas sur Windows, comme l'emballage de notre commande injectée avec des doubles backticks (````) ou avec un opérateur de sous-shell (`$()`).

En général, pour l'injection de commandes de base, tous ces opérateurs peuvent être utilisés pour les injections de commandes "quels que soient le langage, le framework ou le serveur principal de l'application Web". Ainsi, si nous injectons dans une application Web `PHP` s'exécutant sur un serveur `Linux` , ou une application Web `.Net` s'exécutant sur un serveur principal `Windows` , ou une application Web `NodeJS` s'exécutant sur un serveur back-end `macOS` , nos injections devraient fonctionner malgré tout.

Remarque : La seule exception peut être le point-virgule `;`, qui ne fonctionnera pas si la commande a été exécutée avec `Ligne de commande Windows (CMD)`, mais fonctionnera toujours si elle a été exécutée avec `Windows PowerShell`.