Paramètres de fuzzing
==================

* * * * *

Pour l'exploitation du débordement de tampon basé sur la pile, nous suivons généralement quatre étapes principales pour identifier et exploiter la vulnérabilité de débordement de tampon :

1. Paramètres de fuzzing
2. Contrôler l'EIP
3. Identifier les mauvais personnages
4. Trouver une instruction de retour
5. Passer au Shellcode

Habituellement, la première étape de tout exercice de vulnérabilité binaire consiste à fuzzer divers paramètres et toute autre entrée que le programme accepte pour voir si notre entrée peut faire planter l'application. Si l'une de nos entrées réussit à faire planter le programme, nous examinons ce qui a causé le plantage du programme. Si nous constatons que le programme a planté parce que notre entrée a écrasé le registre `EIP` , nous avons probablement une vulnérabilité de dépassement de mémoire tampon basée sur la pile. Il ne reste plus qu'à exploiter cette vulnérabilité avec succès, dont la difficulté peut varier selon le système d'exploitation, l'architecture du programme et les protections.

Commençons par déboguer un programme local appelé `Free CD to MP3 Converter`, qui se trouve dans la machine virtuelle Windows ci-dessous.

* * * * *

Identification des champs d'entrée
------------------------

Comme indiqué dans la section précédente, nous pouvons soit ouvrir notre programme avec x32dbg, soit l'exécuter séparément et l'attacher. Il est toujours préférable de l'exécuter séparément et de s'y attacher pour s'assurer que nous le déboguerions exactement tel qu'il est lorsqu'il est exécuté normalement. Cela peut ne pas faire beaucoup de différence pour les programmes de base comme celui-ci, mais d'autres programmes qui s'appuient sur diverses bibliothèques peuvent être confrontés à certaines différences, c'est pourquoi nous préférons nous attacher à un processus. Une fois que notre débogueur est attaché à `Free CD to MP3 Converter`, nous pouvons commencer à fuzzer divers champs de saisie.

Selon la taille du programme, il peut y avoir différents champs de saisie à fuzzer. Voici des exemples de champs de saisie potentiels :

| Champ | Exemple |
| --- | --- |
| `Champs de saisie de texte` | - Champ "enregistrement de licence" du programme.\
- Divers champs de texte trouvés dans les préférences du programme. |
| `Fichiers ouverts` | Tout fichier que le programme peut ouvrir. |
| `Arguments du programme` | Divers arguments acceptés par le programme lors de l'exécution. |
| `Ressources distantes` | Tous les fichiers ou ressources chargés par le programme au moment de l'exécution ou dans certaines conditions. |

Ce sont les principaux paramètres que nous fuzzons habituellement, mais de nombreux autres paramètres peuvent être exploitables également.

Comme tout programme peut avoir plusieurs de ces types de paramètres, et chacun peut devoir être fuzzé avec différents types d'entrées, nous devrions essayer de sélectionner les paramètres avec les plus grandes possibilités de débordements et commencer à les fuzzer. Nous devrions rechercher un champ qui attend une entrée courte, comme un champ qui définit la date, car la date est généralement courte afin que les développeurs puissent s'attendre à une entrée courte uniquement.

Une autre chose courante que nous devrions rechercher est les champs qui devraient être traités d'une manière ou d'une autre, comme le champ d'enregistrement du numéro de licence, car il sera probablement exécuté sur une fonction spécifique pour tester s'il s'agit d'un numéro de licence correct. Les numéros de licence ont également tendance à avoir une longueur spécifique, de sorte que les développeurs peuvent s'attendre à une certaine longueur uniquement, et si nous fournissons une entrée suffisamment longue, elle peut déborder du champ de saisie.

Il en va de même pour les fichiers ouverts, car les fichiers ouverts ont tendance à être traités après avoir été ouverts. Alors que les développeurs peuvent conserver un tampon très long pour les fichiers ouverts, certains fichiers sont censés être plus courts, comme les fichiers de configuration, et si nous fournissons une longue entrée, cela peut déborder du tampon. Certains types de fichiers ont tendance à provoquer des vulnérabilités de débordement, comme les fichiers `.wav` ou les fichiers `.m3u` , en raison des vulnérabilités des bibliothèques qui traitent ces types de fichiers.

Dans cet esprit, commençons à fuzzer certains champs de notre programme.

* * * * *

Fuzzing des champs de texte
-------------------

Nous parcourons les différents éléments de menu du programme, et comme nous venons de le mentionner, les champs d'enregistrement de licence sont toujours un bon candidat pour les débordements, alors commençons à les fuzzer.

Commençons par créer une charge utile de texte très volumineuse, comme `10 000` caractères, et saisissons-les dans notre champ. Nous pouvons obtenir notre charge utile de texte avec python, comme suit :

```
PS C:\Users\htb-student\Desktop> python -c "print('A'*10000)"

AAAAA... SNIP....AAAA

```

Nous pouvons maintenant copier notre charge utile et la coller dans les deux champs de la fenêtre d'inscription, puis cliquer sur `Ok` :\
![](https://academy.hackthebox.com/storage/modules/89/win32bof_registration_fuzz.jpg)

Comme nous pouvons le voir, le programme ne plante pas et nous dit simplement `L'inscription n'est pas valide`.

`Essayez de fuzzer d'autres champs qui acceptent une entrée de texte en utilisant la même charge utile ci-dessus, et voyez si l'un d'entre eux provoque le plantage du programme.`

* * * * *

Fuzzing fichier ouvert
-------------------

Passons maintenant au fuzzing du programme avec les fichiers ouverts. Le menu `Fichier` du programme et le fait de cliquer sur le bouton `Encoder` semblent accepter les fichiers `.wav` , qui font partie des fichiers qui ont tendance à provoquer des débordements. Essayons donc de fuzzer le programme avec des fichiers `.wav` .

Tout d'abord, nous allons répéter ce que nous avons fait ci-dessus pour générer notre charge utile de texte et écrire le résultat dans un fichier `.wav` , comme suit :

```
PS C:\Users\htb-student\Desktop> python -c "print('A'*10000, file=open('fuzz.wav', 'w'))"

```

Remarque : Il existe des méthodes beaucoup plus avancées de fuzzing des paramètres, en envoyant automatiquement différents types de champs d'entrée et de paramètres pour tenter de planter le programme. Dans notre cas, nous commençons avec un exemple de base d'une chaîne longue simple.

Maintenant, tout en étant attaché à `x32dbg`, essayons d'ouvrir notre fichier de charge utile en cliquant sur l'icône `Encode` :\
![](https://academy.hackthebox.com/storage/modules/89/win32bof_converter_open_wav.jpg)

Le programme peut être `mis en pause` à certains moments du débogage en raison de points d'arrêt ou d'instructions `INT3` , nous pouvons donc simplement cliquer sur le bouton `Exécuter` situé dans la barre supérieure pour continuer l'exécution : ![Pause du débogueur](https ://academy.hackthebox.com/storage/modules/89/win32bof_x32dbg_pause.jpg) ![](https://academy.hackthebox.com/storage/modules/89/win32bof_x32dbg_run.jpg)

Astuce : Si nous voulons ignorer l'arrêt sur les points d'arrêt intégrés, nous pouvons sélectionner `Options > Préférences > Événements` et décocher tout sous `Pause sur`. Une fois que nous l'avons fait, le programme devrait cesser de se casser à chaque fois que nous l'exécutons, et ne se cassera que lorsque nous le plantons sur un débordement.

Une fois que nous avons ouvert le fichier, nous constatons que le programme se bloque et le débogueur s'interrompt avec un message disant `First chance exception on 41414141` :

![Message de plantage](https://academy.hackthebox.com/storage/modules/89/win32bof_crash_1.jpg)

Le message indique que le programme a tenté d'exécuter l'adresse `41414141`. En ASCII, la majuscule `A` a le code hexadécimal `0x41`, il semble donc que le programme ait essayé d'accéder à l'adresse `AAAA`, ce qui signifie que nous avons réussi à modifier l'adresse `EIP` .

Nous pouvons le confirmer en vérifiant la fenêtre des registres en haut à droite : ![Crash Registers](https://academy.hackthebox.com/storage/modules/89/win32bof_crash_registers.jpg)

Comme nous pouvons le voir, nous avons en effet écrasé `EBP` et `EIP`, puis le programme a essayé d'exécuter notre adresse `EIP` écrasée.

Nous pouvons même vérifier la pile dans la fenêtre en bas à droite et voir que notre tampon est rempli de `A` :

![Crash Stack](https://academy.hackthebox.com/storage/modules/89/win32bof_crash_stack.jpg)

Cela montre que nous contrôlons `EIP`, nous pouvons donc exploiter cette vulnérabilité pour exécuter le shellcode que nous écrivons en mémoire.

Dans la section suivante, nous verrons comment nous pouvons mettre une valeur spécifique dans `EIP`, en calculant jusqu'où elle se trouve dans la pile et en modifiant notre charge utile pour refléter cela.
