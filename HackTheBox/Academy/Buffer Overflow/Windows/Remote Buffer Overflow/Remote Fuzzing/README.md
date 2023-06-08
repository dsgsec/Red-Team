Fuzz à distance
==============

* * * * *

Jusqu'à présent, nous avons terminé un exercice de débordement de tampon local, qui couvrait toutes les parties essentielles d'un exercice de débordement de tampon basé sur la pile. En ce qui concerne l'exploitation à distance, la principale différence résiderait dans les scripts d'exploitation, tandis que les parties centrales de l'exploitation du débordement de mémoire tampon sont les mêmes.

* * * * *

Débogage d'un programme distant
--------------------------

Que nous déboguions un programme local ou un programme qui écoute les connexions distantes, nous devrons l'installer et le déboguer localement sur notre machine virtuelle Windows. Une fois que notre exploit est entièrement développé, nous pouvons alors l'exécuter sur le service distant sans avoir besoin d'un accès local. S'il est fait correctement, l'exploit devrait fonctionner, comme nous le verrons plus tard.

Cette fois, nous allons déboguer un programme appelé `CloudMe`, un outil d'utilisateur final pour un service de partage de fichiers, disponible sur le bureau de la machine virtuelle Windows ci-dessous. En tant que service de partage de fichiers, cet outil écoute sur un port toutes les mises à jour du serveur de fichiers. Nous pouvons le voir si l'outil est en cours d'exécution et nous répertorions les ports d'écoute dans `Powershell` :

```
PS C:\htb> netstat -a

...SNIP...
TCP    0.0.0.0:8888           0.0.0.0:0              LISTENING
[CloudMe.exe]

```

Comme nous pouvons le voir, le service écoute sur le port `8888`, et il a également établi une connexion à un serveur distant. Nous pouvons utiliser le programme `netcat` sur le bureau pour interagir avec ce port et voir s'il accepte des paramètres :

```
PS C:\Users\htb-student\Desktop> .\nc.exe 127.0.0.1 8888
?
PS C:\Users\htb-student\Desktop> .\nc.exe 127.0.0.1 8888
help
```

Nous essayons d'envoyer quelques paramètres, et cela ferme les connexions sans nous fournir de sortie. Essayons donc de le déboguer et de le fuzzer avec de grandes chaînes pour voir comment il les gérerait.

Pour déboguer un programme qui écoute sur un port distant, nous suivrons le même processus que nous avons suivi précédemment dans le module, exécuterons le programme et l'attacherons ou l'ouvrirons directement dans `x32dbg`. Si vous n'avez pas encore désactivé tous les points d'arrêt dans `x32dbg`, vous devriez le faire, car ce programme contient de nombreux points d'arrêt. Reportez-vous à la section `Fuzzing` pour savoir comment procéder.

* * * * *

Port distant de fuzzing
-------------------

Une fois que notre programme est en cours d'exécution et que nous y sommes attachés via `x32dbg`, nous pouvons commencer à le fuzzer et essayer de le planter. Contrairement au fuzzing local, où nous avons écrit nos charges utiles dans un fichier, puis ouvert le fichier dans notre application ou copié manuellement notre charge utile dans un champ de texte du programme, avec le fuzzing à distance, nous pouvons automatiser ce processus grâce à notre exploit Python.

Nous allons créer un nouveau script appelé `win32bof_exploit_remote.py` et commencer par ajouter quelques variables pour `IP` et `port`, de sorte que nous puissions facilement les modifier si nous voulons utiliser le script sur un autre serveur. Ensuite, nous écrirons notre fonction de fuzzing `def fuzz():`. Nous souhaitons envoyer des incréments de chaînes volumineuses, en commençant par `500` octets de long et en incrémentant de `500` à chaque itération, jusqu'à ce que nous envoyions une chaîne suffisamment longue qui plante le programme. Pour ce faire, nous allons boucler dans une plage de `0` à `10 000` avec des incréments de `500`, comme suit :

Code : python

```
import socket
from struct import pack

IP = "127.0.0.1"
port = 8888

def fuzz():
    for i in range(0,10000,500):
        buffer = b"A"*i
        print("Fuzzing %s bytes" % i)

```

L'instruction print nous aide à connaître la taille actuelle du tampon de fuzzing afin que, lorsque le programme plante finalement, nous sachions quelle longueur l'a fait planter.

Ensuite, nous devons nous connecter au port à chaque fois et lui envoyer notre charge utile. Pour ce faire, nous devons importer la bibliothèque `socket` comme nous l'avons fait au début de notre code ci-dessus, puis établir une connexion au port avec la fonction `connect` comme suit :

Code : python

```
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((IP, port))

```

Avec cela, nous devrions être prêts à envoyer notre tampon, ce que nous pouvons faire via `s.send(buffer)`. Nous devrons également envelopper notre boucle dans un bloc `try/except` , afin de pouvoir arrêter l'exécution lorsque le programme plante et n'accepte plus les connexions. Notre fonction `fuzz()` finale devrait ressembler à ceci :

Code : python

```
def fuzz():
    try:
        for i in range(0,10000,500):
            buffer = b"A"*i
            print("Fuzzing %s bytes" % i)
            s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            s.connect((IP, port))
            s.send(buffer)
            s.close()
    except:
        print("Could not establish a connection")

fuzz()

```

Remarque : Dans notre cas, le programme ferme la connexion après chaque entrée, comme nous l'avons vu précédemment, nous établissons donc une nouvelle connexion à chaque itération de la boucle. Si nous pouvions conserver la connexion, comme un client ftp ou de messagerie, il serait préférable d'établir la connexion avant la boucle, puis de boucler la fonction `s.send(buffer)` .

Astuce : Comme notre serveur est vulnérable au point d'entrée après l'établissement de la connexion, nous envoyons directement notre charge utile. Il est également possible d'interagir avec le serveur et de transmettre des données telles que les identifiants de connexion ou certains paramètres.rs pour atteindre la fonction vulnérable, en utilisant `send` et `recv`. Vous pouvez en savoir plus sur les fonctions `socket` dans la [Documentation officielle](https://docs.python.org/3/library/socket.html).

Nous exécutons notre script et voyons ce qui suit :

```
Fuzzing 0 bytes
Fuzzing 500 bytes
...SNIP...
Fuzzing 9000 bytes
Fuzzing 9500 bytes

```

Nous constatons que l'intégralité du script s'est exécutée sans planter les services d'écoute, car le port `8888` était toujours à l'écoute tout au long de notre fuzzing. Cependant, si nous vérifions notre débogueur `x32dbg` , nous constatons que le programme frontal `cloudme` a planté et que son `EIP` a été écrasé par notre tampon `A` :

![Remote Fuzz](https://academy.hackthebox.com/storage/modules/89/win32bof_remote_fuzz.jpg)

Cela indique que le service d'écoute réel n'est peut-être pas vulnérable puisque notre entrée ne le plante jamais. Cependant, le programme frontal doit également traiter cette entrée (par exemple, pour la synchronisation des fichiers), et il est vulnérable à un dépassement de mémoire tampon, que nous pouvons exploiter via le service d'écoute. Il s'agit d'un cas unique qui montre que si une entrée est traitée à plusieurs emplacements/programmes, nous devons nous assurer de tous les déboguer, car un seul d'entre eux peut être vulnérable.

* * * * *

Fuzzing progressif
---------------

Nous sommes confrontés au problème ici parce que notre programme n'arrête jamais d'envoyer des charges utiles puisque le service d'écoute ne plante jamais. Alors, comment pourrions-nous savoir à quelle longueur de tampon le programme s'est écrasé ?

Nous pouvons progressivement envoyer notre tampon en ajoutant un `breakpoint()` après `s.send(buffer)`, de sorte que lorsque nous pouvons continuer manuellement en appuyant sur `c`, nous pouvons voir si notre entrée a planté le programme et écrasé `EIP `.

Astuce : Vous pouvez avoir à la fois `x32dbg` et le python IDLE côte à côte, de sorte que vous puissiez immédiatement remarquer quand le programme se bloque.

Nous allons donc ajouter notre point d'arrêt à notre exploit, redémarrer le programme dans `x32dbg`, et commencer à fuzzer progressivement le programme :

```
Fuzzing 0 bytes
> c:\users\htb-student\desktop\win32bof_exploit_remote.py(13)fuzz()
-> s.send(buffer)
(Pdb) c
Fuzzing 500 bytes
> c:\users\htb-student\desktop\win32bof_exploit_remote.py(12)fuzz()
-> breakpoint()
(Pdb) c
Fuzzing 1000 bytes
> c:\users\htb-student\desktop\win32bof_exploit_remote.py(13)fuzz()
-> s.send(buffer)
(Pdb) c
Fuzzing 1500 bytes
> c:\users\htb-student\desktop\win32bof_exploit_remote.py(12)fuzz()
-> breakpoint()
(Pdb) c
...
```

Une fois que le programme plante et `EIP` est écrasé, nous savons que le dernier nombre d'octets que nous avons envoyés est ce qui a planté le programme et que le programme est vulnérable à un dépassement de mémoire tampon.

Dans la section suivante, nous continuerons avec les étapes restantes que nous avons suivies précédemment pour exploiter la vulnérabilité de dépassement de mémoire tampon.
