Cette tâche met en évidence certains des éléments essentiels de haut niveau de la structure de données PE pour les binaires Windows.

## Qu'est-ce que l'PE ?

Le format de fichier Windows Executable, alias PE (Portable Executable), est une structure de données qui contient les informations nécessaires aux fichiers. C'est un moyen d'organiser le code d'un fichier exécutable sur un disque. Les composants du système d'exploitation Windows, tels que les chargeurs Windows et DOS, peuvent le charger en mémoire et l'exécuter en fonction des informations de fichier analysées trouvées dans le PE.

En général, la structure de fichiers par défaut des binaires Windows, tels que les fichiers  EXE, DLL et de code objet, a la même structure PE et fonctionne dans le système d'exploitation Windows pour les deux architectures CPU (x86 et x64).

Une structure PE contient diverses sections contenant des informations sur le binaire, telles que des métadonnées et des liens vers une adresse mémoire de bibliothèques externes. L'une de ces sections est le PE Header , qui contient des informations de métadonnées, des pointeurs et des liens vers des sections d'adresse en mémoire. Une autre section est la section Données , qui comprend  des conteneurs contenant les informations nécessaires au chargeur Windows pour exécuter un programme, telles que  le code exécutable,  les ressources,  les liens vers les bibliothèques, les variables de données, etc. 

![ad61ff4aa1d4f649c02348dfa32eb613](https://github.com/dsgsec/Red-Team/assets/82456829/c98be0f2-4b08-4fc6-a3e7-c0e9d48397a9)

Il existe différents types de conteneurs de données dans la structure PE, chacun contenant des données différentes.

1.  .text stocke le code réel du programme
2.  .data contient les variables initialisées et définies
3.  .bss contient les données non initialisées (variables déclarées sans valeurs attribuées)
4.  .rdata contient les données en lecture seule
5.  .edata : contient des objets exportables et des informations de table associées
6.  Objets importés .idata et informations de table associées
7.  Informations sur la relocalisation de l'image .reloc
8.  .rsrc  relie les ressources externes utilisées par le programme telles que les images, les icônes, les binaires intégrés et le fichier manifeste, qui contient toutes les informations sur les versions du programme, les auteurs, la société et les droits d'auteur !

La structure PE est un sujet vaste et complexe, et nous n'entrerons pas dans les détails concernant les en-têtes et les sections de données. Cette tâche fournit une vue d'ensemble de haut niveau de la structure PE. Si vous souhaitez obtenir plus d'informations sur le sujet, nous vous suggérons de consulter les salles THM suivantes où le sujet est expliqué plus en détail :

-   [Éléments internes de Windows](https://tryhackme.com/room/windowsinternals)
-   Dissection des en-têtes PE

Vous pouvez également obtenir des détails plus détaillés sur PE si vous consultez le  site Web Docs du  [format Windows PE .](https://docs.microsoft.com/en-us/windows/win32/debug/pe-format)

En examinant le contenu du PE, nous verrons qu'il contient un tas d'octets qui ne sont pas lisibles par l'homme. Cependant, il inclut tous les détails dont le chargeur a besoin pour exécuter le fichier. Voici un exemple d'étapes dans lesquelles le chargeur Windows lit un binaire exécutable et l'exécute en tant que processus.

1.  Sections d'en-tête : les en-têtes DOS, Windows et facultatifs sont analysés pour fournir des informations sur le fichier EXE. Par exemple,

-   Le nombre magique commence par « MZ », qui indique au chargeur qu'il s'agit d'un fichier EXE.
-   Signatures de fichiers
-   Si le fichier est compilé pour une architecture CPU x86 ou x64.
-   Horodatage de création. 

3.  Analyse des détails de la table de section, tels que 

-   Nombre de sections que contient le fichier.

5.  Mappage du contenu du fichier dans la mémoire en fonction de

-   L'adresse EntryPoint et le décalage de l'ImageBase.

-   RVA : Adresse Virtuelle Relative, Adresses liées à Imagebase.

7.  Les importations, DLL et autres objets sont chargés dans la mémoire.
8.  L'adresse EntryPoint est localisée et la fonction d'exécution principale s'exécute.

## Pourquoi avons-nous besoin de connaître les PE ?

Il y a plusieurs raisons pour lesquelles nous devons en apprendre davantage. Premièrement,  puisque nous traitons de sujets d'emballage et de déballage, la technique nécessite des détails sur la structure PE. 

L'autre raison est que  les analystes de logiciels antivirus et de logiciels malveillants analysent les fichiers EXE sur la base des informations contenues dans l'en-tête PE et dans d'autres sections PE. Ainsi, pour créer ou modifier des logiciels malveillants  dotés d' une capacité d'évasion  antivirus ciblant une machine Windows, nous devons comprendre la structure des fichiers exécutables portables Windows et l'endroit où le shellcode malveillant peut être stocké.

Nous pouvons contrôler dans quelle section Data stocker notre shellcode par la manière dont nous définissons et initialisons la variable shellcode. Voici quelques exemples qui montrent comment nous pouvons stocker le shellcode dans PE :

1.  Définir le shellcode en tant que variable locale dans la fonction principale le stockera dans la section .TEXT PE.

2.  Définir le shellcode en tant que variable globale le stockera dans la section .Data . 
3.  Une autre technique consiste à stocker le shellcode sous forme de binaire brut dans une image d'icône et à le lier dans le code, donc dans ce cas, il apparaît dans la section Données .rsrc . 
4.  Nous pouvons ajouter une section de données personnalisée pour stocker le shellcode.

### PE-Bear

La machine virtuelle ci-jointe est une machine de développement Windows dotée des outils nécessaires pour analyser les fichiers EXE et lire les détails dont nous avons discuté. Pour votre commodité, nous avons fourni une copie du logiciel PE-Bear sur le bureau, qui permet de vérifier la structure du PE : en-têtes, sections, etc. PE-Bear fournit une interface utilisateur graphique pour afficher tous les détails pertinents de l'EXE. Pour charger un fichier EXE pour analyse, sélectionnez Fichier -> Charger les PE (Ctrl + O).
![c51856efd63b36680857498bac814469](https://github.com/dsgsec/Red-Team/assets/82456829/0a0d2130-5f7f-435b-897e-2443bcb4f08f)


Une fois qu'un fichier est chargé, nous pouvons voir tous les détails du PE. La capture d'écran suivante montre les détails PE du fichier chargé, y compris les en-têtes et les sections dont nous avons parlé plus tôt dans cette tâche.

![78dca06d1d1e4249f25734af8082b8be](https://github.com/dsgsec/Red-Team/assets/82456829/918ca241-de56-4e8b-991f-a7c5dc1ae29f)


Il est maintenant temps de l'essayer ! Chargez le fichier thm-intro2PE.exe pour répondre aux questions ci-dessous. Le fichier se trouve à l'emplacement suivant :  `c:\Tools\PE files\thm-intro2PE.exe`
