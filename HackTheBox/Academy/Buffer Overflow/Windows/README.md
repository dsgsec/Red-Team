Débordement de tampon
===============

* * * * *

L'exploitation binaire fait partie des compétences les plus essentielles pour tout pentester. C'est généralement le moyen de trouver les vulnérabilités les plus avancées dans les programmes et les systèmes d'exploitation et nécessite beaucoup de compétences. Au fil des ans, de nombreuses protections ont été ajoutées à la manière dont la mémoire est gérée par le noyau du système d'exploitation et à la manière dont les fichiers binaires sont compilés pour éviter de telles vulnérabilités. Pourtant, il existe toujours de nouvelles façons d'exploiter les erreurs mineures trouvées dans les fichiers binaires et de les utiliser pour prendre le contrôle d'une machine distante ou obtenir des privilèges plus élevés sur une machine locale.

Cependant, à mesure que les protections binaires et de la mémoire deviennent plus avancées, les méthodes d'exploitation binaires aussi. C'est pourquoi les méthodes d'exploitation binaire modernes nécessitent une compréhension approfondie du langage d'assemblage, de l'architecture informatique et des principes fondamentaux de l'exploitation binaire.

Le langage d'assemblage et l'architecture informatique ont été traités en détail dans le module [Introduction au langage d'assemblage](https://academy.hackthebox.com/course/preview/intro-to-assembly-language) et dans le module [Stack-Based Buffer Overflows sur Linux x86](https://academy.hackthebox.com/course/preview/stack-based-buffer-overflows-on-linux-x86) le module couvre également les bases de l'exploitation binaire sous Linux.

* * * * *

Débordements de tampon
----------------

Dans l'exploitation binaire, notre objectif principal est de subvertir l'exécution du binaire d'une manière qui nous profite. Les dépassements de mémoire tampon sont le type d'exploitation binaire le plus courant, mais d'autres types d'exploitation binaire existent, tels que [Format String](https://owasp.org/www-community/attacks/Format_string_attack) exploitation et [Heap Exploitation](https ://wiki.owasp.org/index.php/Buffer_Overflows#Heap_Overflow).

Un débordement de mémoire tampon se produit lorsqu'un programme reçoit des données plus longues que prévu, de sorte qu'il écrase tout l'espace mémoire de la mémoire tampon sur la [pile](https://en.wikipedia.org/wiki/Stack_(abstract_data_type)). Cela peut écraser le prochain pointeur d'instruction `EIP` (*ou `RIP` en x86_64*), ce qui provoque le blocage du programme car il tentera d'exécuter des instructions à une adresse mémoire non valide. En forçant le programme à planter, c'est l'exemple le plus basique d'exploitation des débordements de tampon - connu sous le nom d'attaque par déni de service (`DOS`).

Une autre attaque de base consiste à écraser une valeur sur la pile pour modifier le comportement du programme. Par exemple, si un programme d'examen avait une vulnérabilité de débordement de tampon, nous pouvons écraser suffisamment le tampon pour écraser notre score. Étant donné que notre score d'examen est stocké dans la pile dans cet exemple, nous pourrions profiter de cette faille pour modifier notre score.

Si nous sommes un peu plus sophistiqués, nous pouvons changer l'adresse `EIP` en une instruction qui exécutera notre shellcode. Cela nous permettrait d'exécuter n'importe quelle commande que nous voulons au lieu de simplement planter le programme, connu sous le nom de Jumping to Shellcode.

Avec des protections de mémoire plus avancées, il peut ne pas être possible de charger tout notre shellcode et de pointer dessus. Au lieu de cela, nous pouvons utiliser une combinaison d'instructions du binaire pour exécuter une fonction particulière et écraser divers pointeurs pour modifier le flux d'exécution du programme. Ceci est connu sous le nom d'attaques de programmation orientée retour (`ROP`).

Enfin, les programmes et systèmes d'exploitation modernes peuvent utiliser le tas au lieu de la pile pour stocker la mémoire tampon, ce qui nécessiterait des méthodes de dépassement de tas ou d'exploitation de tas.

* * * * *

Débordement de pile
--------------

Commençons par montrer comment la pile fonctionne dans le stockage des données. La pile a une conception Last-in, First-out (LIFO), ce qui signifie que nous ne pouvons extraire que le dernier élément "poussé" dans la pile. Si nous `poussons` un élément dans la pile, il se trouvera en haut de la pile. Si nous `déposons` quelque chose de la pile, l'élément situé en haut de la pile serait décollé.

Le tableau suivant illustre le fonctionnement de la pile. Nous pouvons cliquer sur `push` pour pousser une valeur de `eax` vers la pile, et `pop` pour faire apparaître la valeur supérieure de la pile vers `eax` :

| | |
| | |
| | |
| 0xabcdef | <-- Haut de la pile (`$esp`) |
| 0x12345678 | <-- Bas de la pile (`$ebp`) |

`eax : `\
pousser pop

Réinitialiser la pile

L'exemple ci-dessus reçoit correctement les données de la mémoire tampon, de sorte qu'elles ne soient jamais débordées vers l'élément suivant. Passons maintenant en revue un autre exemple qui ne stocke pas correctement les données sur la pile.

L'exemple suivant attend de nous une entrée de huit caractères. Mais que se passerait-il si nous envoyions quelque chose de plus long ? Essayons d'envoyer '`01234567890123456789`' :

| | |
| | |
| | |
| 0xabcdef | <-- Haut de la pile (`$esp`) |
| 0x401000 | <-- Adresse de retour (`$eip`) |
| 0x12345678 | <-- Bas de la pile (`$ebp`) |

`eax : `\
pousser pop

Réinitialiser la pile

Comme nous pouvons le voir, lorsque nous envoyons une chaîne plus longue que prévu, elle écrase les autres valeurs existantes sur la pile et écraserait même toute la pile si elle est suffisamment longue. Plus important encore, nous voyons qu'il a écrasé la valeur à `EIP`, et lorsque la fonction essaie de revenir à cette adresse, le programme plante car cette adresse '`0x6789`' n'existe pas en mémoire. Cela se produit en raison de la conception LIFO de la pile, qui grandit vers le haut, tandis qu'une longue chaîne déborde de valeurs vers le bas jusqu'à ce qu'elle écrase finalement l'adresse de retour `EIP` et le bas du pointeur de pile `EBP`. Cela a été expliqué dans le module [Introduction au langage d'assemblage](https://academy.hackthebox.com/module/details/85).

Chaque fois qu'une fonction est appelée, un nouveau cadre de pile est créé et l'ancienne adresse `EIP` est poussée en haut du nouveau cadre de pile, afin que le programme sache où revenir une fois la fonction terminée. Par exemple, si notre entrée de tampon écrase l'intégralité de la pile et l'adresse de retour `EIP`, l'adresse `EIP` écrasée sera appelée lors du retour de la fonction en raison d'une instruction `RET` .

Si nous calculons notre entrée avec précision, nous pouvons placer une adresse valide à l'emplacement où `EIP` est stocké. Cela conduirait le programme à se rendre à notre adresse écrasée lorsqu'il revient et à subvertir le flux d'exécution du programme vers une adresse de notre choix.

* * * * *

Exemples concrets
-------------------

Il y a eu de nombreux incidents où des exploits de débordement de pile ont été utilisés pour pénétrer dans des systèmes restreints, comme les téléphones mobiles ou les consoles de jeux.

En 2010, les iPhone fonctionnant sous iOS 4 ont été jailbreakés à l'aide du jailbreak [greenpois0n](https://www.theiphonewiki.com/wiki/Greenpois0n_(jailbreak)), qui utilisait deux exploits différents pour obtenir un accès au niveau du noyau sur l'iPhone et installer des logiciels et des applications non officiels/non signés. L'un de ces exploits était un dépassement de mémoire tampon basé sur la pile sur le [HFS Volume Name](https://www.theiphonewiki.com/wiki/HFS_Legacy_Volume_Name_Stack_Buffer_Overflow) de l'iPhone. À cette époque, les iPhones ne randomisaient pas automatiquement l'espace d'adressage, et iOS 4.3 a corrigé ces vulnérabilités et introduit des protections de mémoire comme la randomisation des espaces d'adressage avec la randomisation de la disposition de l'espace d'adresse ([ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization )).

Un exploit de dépassement de mémoire tampon basé sur la pile a également été utilisé pour obtenir un accès au niveau du noyau sur la PlayStation Portable (PSP) d'origine exécutant le micrologiciel v2.0. Cela permettait l'utilisation de jeux piratés ainsi que l'installation de logiciels non signés. L'[Exploit TIFF](https://en.wikibooks.org/wiki/PSP/Homebrew_History) exploite une vulnérabilité trouvée dans la bibliothèque d'images TIFF utilisée dans la visionneuse de photos de la PSP. Cela conduit à l'exécution de code en affichant simplement un fichier `.tiff` malveillant dans la visionneuse de photos après avoir défini l'arrière-plan sur une image `.png` corrompue. Un autre exploit de débordement de pile similaire a été découvert plus tard dans le jeu PSP "Grand Theft Auto: Liberty City Stories", qui présentait une vulnérabilité de débordement dans ses données de jeu sauvegardé et peut être exploité en chargeant un fichier de chargement malveillant.

Un autre exemple d'exploit de dépassement de mémoire tampon basé sur la pile a été utilisé pour obtenir un accès au niveau du noyau sur la Nintendo Wii d'origine, ce qui permettait également l'utilisation de jeux piratés et l'installation de logiciels non signés. Le [Twilight Hack](https://wiibrew.org/wiki/Twilight_Hack) exploite une vulnérabilité trouvée dans le jeu "The Legend of Zelda : Twilight Princess" et est également exploité en chargeant des données de jeu enregistrées malveillantes, en utilisant un nom long pour Cheval de Link "Epona".

Enfin, en 2020, une nouvelle vulnérabilité a été découverte pour la PlayStation 2, près de 20 ans après sa sortie initiale. Le [FreeDVDBoot](https://cturt.github.io/freedvdboot.html) exploite une vulnérabilité du lecteur DVD de la PS2 en plaçant un fichier "VIDEO_TS.IFO" malveillant. Celui-ci est lu par le lecteur DVD et provoque un débordement pouvant conduire à l'exécution de code. Il s'agissait du tout premier hack PS2 entièrement basé sur un logiciel, car tous les hacks plus anciens utilisaient une forme de matériel comme une carte mémoire malveillante pour charger et exécuter des logiciels non signés.

Bien entendu, les systèmes d'exploitation tels que Windows, Linux et macOS ont toujours été la première cible des exploits de dépassement de mémoire tampon basés sur la pile. De nombreuses vulnérabilités de ce type ont été trouvées dans tous ces systèmes et logiciels qui y sont exécutés. En détectant ces vulnérabilités avant que les produits n'entrent en production, nous réduirions l'apparition de pièges potentiellement catastrophiques.

* * * * *

Protections contre le débordement de pile
--------------------------

Comme nous pouvons le remarquer à partir des exemples ci-dessus, la plupart d'entre eux sont assez anciens et datent d'au moins une décennie. En effet, les systèmes d'exploitation modernes disposent de nombreuses protections pour la pile, comme empêcher l'exécution de code ou modifier de manière aléatoire les adresses mémoire. Ces protections font en sorte que nous ne pouvons pas facilement exécuter notre code placé sur la pile ou pré-calculer l'adresse mémoire vers laquelle sauter.

Cependant, même avec ces types de protections, si un programme est vulnérable à un Buffer Overflow, il existe des méthodes avancées pour contourner ces protections. Certains exemples incluent la programmation orientée retour (`ROP`) mentionnée précédemment ou des méthodes d'exploitation spécifiques à Windows comme l'exploitation Egg Hunting ou Structured Exception Handling (`SEH`).

De plus, les compilateurs modernes empêchent l'utilisation de fonctions vulnérables aux débordements de Stack, ce qui réduit considérablementl'apparition de débordements de tampon basés sur la pile. C'est pourquoi les débordements de tampon basés sur la pile sont moins courants de nos jours. Dans le même temps, d'autres types d'exploitation binaire plus avancés sont plus courants, car ils ne peuvent pas être atténués en activant simplement une méthode de protection.

* * * * *

Pourquoi apprendre les débordements de pile de base ?
--------------------------------

Dans ce module, nous apprendrons comment obtenir l'exécution de code grâce à des débordements de tampon de base basés sur la pile. Nous le ferons sur les applications et les systèmes qui ne disposent d'aucune protection de la mémoire. Sinon, nous aurions besoin de méthodes plus avancées pour obtenir l'exécution du code.

`Mais si les débordements de base basés sur la pile ne sont plus courants de nos jours, alors pourquoi devrions-nous les apprendre ?`\
Nous le faisons parce que les apprendre nous donne une bonne compréhension des bases de l'exploitation binaire et des principes fondamentaux du développement d'exploits.

De plus, une fois que nous aurons maîtrisé comment détecter et exploiter les dépassements de mémoire tampon de base basés sur la pile, il sera beaucoup plus facile d'apprendre [Structured Exception Handling (SEH)](https://docs.microsoft.com/en-us/cpp/cpp /structured-exception-handling-c-cpp?view=msvc-160#:~:text=Structured%20exception%20handling%20(SEH)%20is,code%20more%20portable%20and%20flexible.), ce qui est très courant dans les systèmes Windows modernes.

Enfin, une fois que nous aurons une bonne maîtrise des débordements de pile de base et des contournements d'atténuation de base, nous serions prêts à commencer à apprendre des méthodes avancées de contournement d'atténuation, comme (`ROP`) et d'autres méthodes avancées d'exploitation binaire.
