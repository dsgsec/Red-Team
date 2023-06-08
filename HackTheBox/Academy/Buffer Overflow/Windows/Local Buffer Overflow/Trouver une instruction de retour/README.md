Trouver une instruction de retour
============================

* * * * *

Comme nous avons confirmé que nous pouvons contrôler l'adresse stockée dans `EIP` lorsque le programme exécute l'instruction de retour `ret`, nous savons que nous pouvons subvertir l'exécution du programme et lui faire exécuter n'importe quelle instruction que nous voulons en écrivant l'adresse de l'instruction à ` EIP`, qui serait exécuté après l'instruction de retour `ret`.

Mais de quelles instructions disposons-nous ? Et comment une seule instruction d'assemblage nous aiderait-elle à obtenir l'exécution du code ? C'est ce dont nous allons discuter dans cette section.

* * * * *

Subvertir le déroulement du programme
------------------------

Pour subvertir avec succès le flux d'exécution du programme, nous devons écrire une adresse de travail sur `EIP` qui mène à une instruction qui nous sera bénéfique. Actuellement, nous n'avons écrit que 4 `B` dans `EIP`, qui (évidemment) n'est pas une adresse de travail, et lorsque le programme tentera d'accéder à cette adresse, il échouera, ce qui entraînera le plantage de l'ensemble du programme .

Pour trouver une adresse que nous pouvons utiliser, nous devons examiner toutes les instructions utilisées ou chargées par notre programme, en choisir une et écrire son adresse dans `EIP`. Dans les systèmes modernes avec la randomisation de la disposition de l'espace d'adresse (ASLR), si nous choisissons une adresse, elle sera inutile, car elle changera la prochaine fois que notre programme s'exécutera puisqu'elle sera randomisée. Dans ce cas, nous devrions suivre une méthode de fuite de l'ensemble actuel d'adresses en temps réel et l'utiliser dans notre exploit. Cependant, nous ne traitons aucun de ces types de protections dans ce module, nous pouvons donc supposer que l'adresse que nous choisissons ne changera pas, et nous pouvons l'utiliser en toute sécurité dans notre programme.

Pour savoir quelle instruction utiliser, nous devons d'abord savoir ce que nous voulons que cette adresse fasse. Alors que des méthodes d'exploitation binaires plus avancées telles que `ROP` s'appuient sur l'utilisation et le mappage de diverses instructions locales pour effectuer l'attaque (comme l'envoi d'un shell inversé), nous n'avons pas encore besoin d'aller plus loin, car nous avons affaire à un programme avec le plus de mémoire protections désactivées.

Nous allons donc utiliser une méthode connue sous le nom `Jumping to Stack`.

* * * * *

Sauter pour empiler
----------------

Comme nous avons déjà des entrées dans la pile, que nous débordons de données, nous pouvons potentiellement écrire des instructions qui nous enverront un reverse shell lors de leur exécution (sous forme de code machine/shellcode). Une fois que nous avons écrit nos données sur la pile, nous pouvons alors diriger le flux d'exécution du programme vers la pile, de sorte qu'il commence à exécuter notre shellcode, auquel cas nous recevrons un shell inversé et prendrons le contrôle du serveur distant.

Pour diriger le flux d'exécution vers la pile, nous devons écrire une adresse `EIP` pour ce faire. Ceci peut être fait de deux façons:

1. Écrivez l'adresse `ESP` (en haut de la pile) dans `EIP`, afin qu'il commence à exécuter le code trouvé en haut de la pile
2. À l'aide d'une instruction `JMP ESP` qui dirige le flux d'exécution vers la pile

Avant de continuer, nous devons noter que cette méthode NE FONCTIONNE PAS avec les machines modernes, comme nous l'avons mentionné précédemment, et qu'elle est considérée comme une méthode d'exploitation héritée.\
![](https://academy.hackthebox.com/storage/modules/89/win32bof_stack_meme.png)

Les systèmes et programmes modernes sont compilés avec le bit `NX` sur la pile ou la protection de la mémoire `DEP` dans Windows, qui empêche l'exécution de tout code écrit sur la pile. Ainsi, même si nous écrivions le shellcode sur la pile, il ne serait pas exécutable, et nous ne trouverions pas non plus d'instruction `JMP ESP` que nous puissions utiliser dans le programme.

Cependant, comme nous l'avons mentionné au début du module, il est toujours très avantageux de commencer par apprendre de telles techniques, car elles forment des techniques plus avancées comme `SEH` ou `ROP`. Une fois que nous maîtrisons cette technique, notre prochaine étape dans l'exploitation binaire serait de contourner les protections et d'utiliser des méthodes plus avancées pour obtenir l'exécution de code sans avoir besoin d'écrire du shellcode sur la pile.

* * * * *

Utilisation de l'adresse ESP
-----------------

Essayons d'abord la méthode la plus basique d'écriture de l'adresse du haut de la pile `ESP`. Une fois que nous écrivons une adresse à `EIP` et que le programme se bloque sur l'instruction de retour `ret`, le débogueur s'arrête à ce point, et l'adresse `ESP` correspond au début de notre shellcode, de la même manière que nous vu nos personnages sur la pile lors de la recherche de mauvais personnages. Nous pouvons noter l'adresse `ESP` à ce stade, qui dans ce cas est `0014F974` :

![Pattern EIP](https://academy.hackthebox.com/storage/modules/89/win32bof_pattern_eip.jpg)

Conseil : Nous pouvons également consulter la pile dans le volet inférieur droit pour trouver les mêmes détails.

Cette méthode peut fonctionner pour ce programme particulier, mais ce n'est pas une méthode très fiable sur les machines Windows. Tout d'abord, l'entrée que nous attaquons ici est un fichier audio, nous voyons donc que tous les caractères sont autorisés sans caractères erronés. Cependant, dans de nombreux cas, nous pouvons attaquer une entrée de chaîne ou un argument de programme, auquel cas `0x00` serait un mauvais caractère, et nous n'utiliserions pas l'adresse de `ESP` puisqu'elle commence par `00`.

Une autre raison est que l'adresse `ESP` peutpas être le même sur toutes les machines. Ainsi, il peut fonctionner tout au long du processus de débogage et de développement de l'exploit, mais il peut ne pas s'agir de la même adresse lorsque nous lançons l'exploit sur une cible réelle, car il peut avoir une adresse "ESP" différente, auquel cas notre exploit échouerait.

Néanmoins, notons cette adresse et continuons, et nous testerons les deux méthodes dans la section suivante.

* * * * *

Utilisation de JMP ESP
--------------

La manière la plus fiable d'exécuter le shellcode chargé sur la pile est de trouver une instruction utilisée par le programme qui dirige le flux d'exécution du programme vers la pile. Nous pouvons utiliser plusieurs de ces instructions, mais nous utiliserons la plus basique, `JMP ESP`, qui saute au sommet de la pile et continue l'exécution.

#### Localisation des modules

Pour trouver cette instruction, nous devons parcourir les exécutables et les bibliothèques chargées par notre programme. Ceci comprend:

1. Le fichier `.exe` du programme
2. Les propres bibliothèques `.dll` du programme
3. Toutes les bibliothèques `.dll` Windows utilisées par le programme

Pour trouver une liste de tous les fichiers chargés par le programme, nous pouvons utiliser `ERC --ModuleInfo`, comme suit : ![Module Info](https://academy.hackthebox.com/storage/modules/89/win32bof_module_info.jpg )

On retrouve de nombreux modules chargés par le programme. Cependant, nous pouvons ignorer tous les fichiers avec :

- `NXCompat` : comme nous recherchons une instruction `JMP ESP` , le fichier ne doit donc pas avoir de protection contre l'exécution de la pile.
- `Rebase` ou `ASLR` : étant donné que ces protections entraîneraient un changement d'adresse entre les exécutions

En ce qui concerne `OS DLL`, si nous exécutons une version plus récente de Windows comme Windows 10, nous pouvons nous attendre à ce que tous les fichiers OS DLL aient toutes les protections de mémoire présentes, nous n'en utiliserons donc aucune. Si nous attaquions une ancienne version de Windows comme Windows XP, de nombreuses DLL de système d'exploitation chargées n'ont probablement aucune protection, de sorte que nous pouvons également y rechercher des instructions "JMP ESP".

Si nous ne considérons que les fichiers avec `False` défini sur toutes les protections, nous obtiendrons la liste suivante :

Localisation des modules

```
-------------------------------------------------- -------------------------------------------------- --------------------
  base | Point d'entrée | Taille | Rebase | SafeSEH | ASLR | NXCompat | DLL du système d'exploitation | Version, nom et chemin
-------------------------------------------------- -------------------------------------------------- --------------------
0x400000 0xd88fc 0x11c000 Faux Faux Faux Faux Faux C:\Program Files\CD to MP3 Freeware\cdextract.exe
0x672c0000 0x1000 0x13000 Faux Faux Faux Faux Faux 1.0rc1;AKRip32;C:\Program Files\CD en MP3 Freeware\akrip32.dll
0x10000000 0xa3e0 0xc000 Faux Faux Faux Faux Faux C:\Program Files\CD to MP3 Freeware\ogg.dll

```

Comme nous pouvons le voir, tous les fichiers appartiennent au programme lui-même, ce qui indique que le programme et tous ses fichiers ont été compilés sans aucune protection de la mémoire, ce qui signifie que nous pouvons y trouver des instructions "JMP ESP". `La meilleure option est d'utiliser une instruction du programme lui-même, car nous serons sûrs que cette adresse existera quelle que soit la version de Windows exécutant le programme`.

#### Recherche de JMP ESP

Maintenant que nous avons une liste de fichiers chargés qui peuvent inclure l'instruction que nous recherchons, nous pouvons les rechercher pour des instructions utilisables. Pour accéder à l'un de ces fichiers, nous pouvons accéder à l'onglet `Symboles` en cliquant dessus ou en appuyant sur `alt+e` :

![Symboles de modules](https://academy.hackthebox.com/storage/modules/89/win32bof_module_symbols.jpg)

Nous pouvons commencer par `cdextract.exe` et double-cliquer dessus pour ouvrir la vue et rechercher ses instructions. Pour rechercher l'instruction `JMP ESP` dans les instructions de ce fichier, nous pouvons cliquer `ctrl+f`, ce qui nous permet de rechercher n'importe quelle instruction dans le fichier ouvert `cdextract.exe` : ![Find Command](https://academy.hackthebox.com/storage/modules/89/win32bof_find_command.jpg)

Nous pouvons entrer `jmp esp`, et il devrait nous montrer si ce fichier contient l'une des instructions que nous avons recherchées : ![Find JMP ESP](https://academy.hackthebox.com/storage/modules/89/win32bof_find_jmp_esp.jpg)

Comme nous pouvons le voir, nous avons trouvé les correspondances suivantes :

Code : coque

```
Démontage d'adresse
00419D0B jmp esp
00463B91 esp jmp
00477A8B jmp esp
0047E58B jmp esp
004979F4 esp jmp

```

Remarque : Nous pouvons également rechercher `CALL ESP`, qui passera également à la pile.

Comme c'est le cas avec l'adresse lors de l'utilisation de l'adresse `ESP` , `nous devons nous assurer que l'adresse d'instruction ne contient pas de caractères erronés`. Sinon, notre charge utile serait tronquée et l'attaque échouerait. Cependant, dans notre cas, nous n'avons pas de mauvais caractères, nous pouvons donc choisir l'une des adresses ci-dessus.

Nous pouvons double-cliquer sur l'un des résultats pour voir l'instruction dans le démontage du fichier principal et revérifier qu'il s'agit bien d'une instruction `JMP ESP` .

Nous pouvons également vérifier les autres fichiers `.dll` chargés pour voir s'ils contiennent des instructions utiles, au cas où l'un des éléments ci-dessus ne fonctionnerait pas correctement. à dAinsi, nous pouvons revenir à l'onglet `Symboles` , en double-cliquant sur le fichier que nous voulons rechercher, puis effectuer le même processus pour rechercher l'instruction `JMP ESP` .

Si nous avions une longue liste de modules chargés, nous pourrions tous les parcourir en cliquant avec le bouton droit de la souris sur le volet principal en haut à droite `CPU` et en sélectionnant `Rechercher> Tous les modules> Commande`, puis en saisissant `jmp esp`. Cependant, cela peut renvoyer une longue liste de résultats, dont certains peuvent ne pas être utilisables. Cela dépend des protections présentes pour son binaire et s'il est directement accessible par notre programme. Pour cette raison, il est préférable d'essayer d'abord de rechercher dans des fichiers individuels.

#### Recherche de modèles

Un autre exemple de commande de base pour accéder à la pile est `PUSH ESP` suivi de `RET`. Puisque nous recherchons deux instructions, dans ce cas, nous devrions rechercher en utilisant le code machine plutôt que les instructions d'assemblage. Nous pouvons utiliser [Online Assemblers](https://defuse.ca/online-x86-assembler.htm) ou l'outil `msf-nasm_shell` qui se trouve dans `PwnBox` pour convertir toutes les instructions d'assemblage en code machine. Les deux prennent une instruction de montage et nous donnent le code machine correspondant.

Après avoir utilisé l'un d'entre eux, nous constaterions que le code machine pour `JMP ESP` est `FFE4`, et pour `PUSH ESP ; RET` est `54C3`. Nous pouvons maintenant effectuer une recherche à l'aide de ce modèle en cliquant `ctrl+b` dans le volet `CPU` et en saisissant le modèle `54C3` :

![Trouver un modèle](https://academy.hackthebox.com/storage/modules/89/win32bof_find_pattern.jpg)

Une fois que nous l'aurons fait, nous trouverons quelques autres adresses que nous pouvons également utiliser : ![Find Pattern PUSH ESP](https://academy.hackthebox.com/storage/modules/89/win32bof_find_pattern_push_esp.jpg)

Nous pouvons double-cliquer sur l'un d'eux et confirmer qu'il s'agit bien d'une instruction `PUSH ESP` suivie d'une instruction `RET` :

![PUSH ESP](https://academy.hackthebox.com/storage/modules/89/win32bof_pattern_push_esp.jpg)

* * * * *

Résumé
-------

Nous avons discuté de nombreuses méthodes pour trouver une instruction qui exécuterait le shellcode que nous chargeons sur la pile :

1. Nous pouvons utiliser l'adresse `ESP` 
2. Nous pouvons rechercher des modules chargés avec une sécurité désactivée pour les instructions `JMP ESP` 
3. Nous pouvons rechercher des instructions d'assemblage ou rechercher des modèles de code machine
4. Toute adresse que nous choisissons ne doit contenir aucun mauvais caractère
