Architecture du processeur
================

* * * * *

L'architecture du `Von-Neumann` a été développée par le mathématicien hongrois John von Neumann et se compose de quatre unités fonctionnelles :

- "Mémoire"
- `Unité de contrôle`
- `Unité arithmétique logique`
- `Unité d'entrée/sortie`

Dans l'architecture de Von-Neumann, les unités les plus importantes, l' `unité logique arithmétique` (`ALU`) et `l'unité de contrôle` (`CU`), sont combinées dans la véritable `unité centrale de traitement` (`CPU`). Le `CPU` est responsable de l'exécution des `instructions` et du `contrôle de flux`. Les instructions sont exécutées les unes après les autres, pas à pas. Les commandes et les données sont extraites de la mémoire par le `CU`. La connexion entre le processeur, la mémoire et l'unité d'entrée/sortie est appelée "système de bus", qui n'est pas mentionné dans l'architecture originale de Von-Neumann mais joue un rôle essentiel dans la pratique. Dans l'architecture Von-Neumann, toutes les instructions et données sont transférées via le "système de bus".

#### Architecture von Neumann

![image](https://academy.hackthebox.com/storage/modules/31/von_neumann3.png)

* * * * *

Mémoire
------

La mémoire peut être divisée en deux catégories différentes :

- `Mémoire primaire`
- `Mémoire secondaire`

* * * * *

#### Mémoire primaire

La `mémoire principale` est le `Cache` et la `mémoire à accès aléatoire` (`RAM`). Si nous y réfléchissons logiquement, la mémoire n'est rien de plus qu'un endroit pour stocker des informations. Nous pouvons considérer cela comme laisser quelque chose à l'un de nos amis pour le reprendre plus tard. Mais pour cela, il est nécessaire de connaître l'"adresse" de l'ami pour récupérer ce que nous avons laissé derrière lui. C'est la même chose que `RAM`. La RAM décrit un type de mémoire dont les allocations de mémoire sont accessibles directement et de manière aléatoire par leurs "adresses mémoire".

Le `cache` est intégré au processeur et sert de tampon, ce qui, dans le meilleur des cas, garantit que le processeur est toujours alimenté en données et en code de programme. Avant que le code de programme et les données n'entrent dans le processeur pour traitement, la RAM sert de stockage de données. La taille de la RAM détermine la quantité de données pouvant être stockées pour le processeur. Cependant, lorsque la mémoire principale perd de l'alimentation, tout le contenu stocké est perdu.

* * * * *

#### Mémoire secondaire

La `mémoire secondaire` est le stockage de données externe, tel que `HDD/SSD`, `lecteurs Flash` et `CD/DVD-ROM` d'un ordinateur, qui n'est `pas` directement accessible par le processeur, mais via le ` Interfaces d'E/S. En d'autres termes, il s'agit d'un périphérique de stockage de masse. Il est utilisé pour stocker de manière permanente des données qui n'ont pas besoin d'être traitées pour le moment. Par rapport à la `mémoire principale`, elle a une capacité de stockage plus élevée, peut stocker des données en permanence même sans alimentation électrique et fonctionne beaucoup plus lentement.

* * * * *

Unité de contrôle
------------

L' `unité de contrôle` (`CU`) est responsable de l'interfonctionnement correct des différentes parties du processeur. Une connexion de bus interne est utilisée pour les tâches de `CU`. Les tâches de la `CU` peuvent être résumées comme suit :

- Lecture des données de la RAM
- Sauvegarde des données en RAM
- Fournir, décoder et exécuter une instruction
- Traitement des entrées des périphériques
- Traitement des sorties vers les périphériques
- Contrôle d'interruption
- Surveillance de l'ensemble du système

Le `CU` contient le `Instruction Register` (`IR`), qui contient toutes les instructions que le processeur décode et exécute en conséquence. Le décodeur d'instructions traduit les instructions et les transmet à l'unité d'exécution, qui exécute ensuite l'instruction. L'unité d'exécution transfère les données à `ALU` pour le calcul et reçoit le résultat à partir de là. Les données utilisées lors de l'exécution sont temporairement stockées dans `registres`.

* * * * *

Unité centrale de traitement
------------------------

L' `Central Processing Unit` (`CPU`) est l'unité fonctionnelle d'un ordinateur qui fournit la puissance de traitement réelle. Il est responsable du traitement des informations et du contrôle des opérations de traitement. Pour ce faire, le `CPU` récupère les commandes de la mémoire les unes après les autres et lance le traitement des données.

Le processeur est également souvent appelé `Microprocesseur` lorsqu'il est placé dans un seul circuit électronique, comme dans nos PC.

Chaque `CPU` a une architecture sur laquelle il a été construit. Les "architectures de processeur" les plus connues sont :

- `x86`/`i386` - (AMD et Intel)
- `x86-64`/`amd64` - (Microsoft et Sun)
- `ARM` - (Gland)

Chacune de ces architectures de processeur est construite d'une manière spécifique, appelée `Architecture de jeu d'instructions` (`ISA`), que le processeur utilise pour exécuter ses processus. « ISA » décrit donc le comportement d'un CPU concernant le jeu d'instructions utilisé. Les jeux d'instructions sont définis de manière à être indépendants d'une implémentation spécifique. Surtout, ISA nous donne la possibilité de comprendre le comportement unifié du `code machine` en `langage d'assemblage` concernant les `registres`, les `types de données`, etc.

Il existe quatre types différents d'"ISA" :

- `CISC` - `Calcul de jeux d'instructions complexes`
- `RISC` - `Calcul du jeu d'instructions réduit`
- `VLIW` - `Mot d'instruction très long`
- `EPIC` - `Explicittly Parallel Instruction Computing`

* * * * *

#### RISQUE

`RISC` signifie `Reduced Instruction Set Computer`, une conception d'architecture de microprocesseurs qui visait à simplifier la complexité du jeu d'instructions pour la programmation d'assemblage à un cycle d'horloge. Cela conduit à des fréquences d'horloge plus élevées du CPU mais permet une exécution plus rapide car des jeux d'instructions plus petits sont utilisés. Par jeu d'instructions, on entend l'ensemble des instructions machine qu'un processeur donné peut exécuter. Nous pouvons trouver `RISC` dans la plupart des smartphones aujourd'hui, par exemple. Néanmoins, presque tous les processeurs contiennent une partie de `RISC`. Les architectures `RISC` ont une longueur fixe d'instructions définies comme `32 bits` et `64 bits`.

* * * * *

#### ICCA

Contrairement à RISC, le `Complex Instruction Set Computer` (`CISC`) est une architecture de processeur avec un jeu d'instructions étendu et complexe. En raison du développement historique des ordinateurs et de leur mémoire, des séquences récurrentes d'instructions ont été combinées en instructions compliquées dans les ordinateurs de deuxième génération. L'adressage dans les architectures `CISC` ne nécessite pas 32 bits ou 64 bits contrairement à RISC, mais peut être fait avec un mode `8 bits` .

* * * * *

#### Cycle d'instructions

Le jeu d'instructions décrit l'ensemble des instructions machine d'un processeur. La portée du jeu d'instructions varie considérablement selon le type de processeur. Chaque processeur peut avoir des cycles d'instructions et des jeux d'instructions différents, mais ils ont tous une structure similaire, que nous pouvons résumer comme suit :

| Instruction | Descriptif |
| --- | --- |
| `1\. RÉCUPÉRER` | L'adresse d'instruction machine suivante est lue à partir du `Instruction Address Register` (`IAR`). Il est ensuite chargé à partir du `Cache` ou `RAM` dans le `Instruction Register` (`IR`). |
| `2\. DÉCODER` | Le décodeur d'instructions convertit les instructions et démarre les circuits nécessaires pour exécuter l'instruction. |
| `3\. ALLER OPÉRANDES` | Si d'autres données doivent être chargées pour l'exécution, elles sont chargées à partir du cache ou `RAM` dans les registres de travail. |
| `4\. EXÉCUTER` | L'instruction est exécutée. Il peut s'agir, par exemple, d'opérations dans l' `ALU`, d'un saut dans le programme, de la réécriture des résultats dans les registres de travail ou du contrôle de périphériques. En fonction du résultat de certaines instructions, le registre d'état est défini, qui peut être évalué par des instructions ultérieures. |
| `5\. POINTEUR D'INSTRUCTIONS DE MISE À JOUR` | Si aucune instruction de saut n'a été exécutée dans la phase EXECUTE, `IAR` est maintenant augmenté de la longueur de l'instruction afin qu'il pointe vers l'instruction machine suivante. |
Ò