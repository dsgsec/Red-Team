Qu'est-ce que l'injection de commande OS?
-----------------------------------------

L'injection de commande d'OS est également connue sous le nom d'injection de shell. Il permet à un attaquant d'exécuter des commandes du système d'exploitation (OS) sur le serveur qui exécute une application, et généralement de compromettre complètement l'application et ses données. Souvent, un attaquant peut exploiter une vulnérabilité d'injection de commande OS pour compromettre d'autres parties de l'infrastructure d'hébergement et exploiter les relations de confiance pour faire pivoter l'attaque vers d'autres systèmes de l'organisation.

Commandes utiles
----------------

Après avoir identifié une vulnérabilité d'injection de commande OS, il est utile d'exécuter quelques commandes initiales pour obtenir des informations sur le système. Vous trouverez ci-dessous un résumé de certaines commandes utiles sur les plateformes Linux et Windows:

| But du commandement | Linux | Windows |
| Nom de l'utilisateur actuel | `whoami` | `whoami` |
| Système d'exploitation | `uname -a` | `ver` |
| Configuration du réseau | `ifconfig` | `ipconfig /all` |
| Connexions réseau | `netstat -an` | `netstat -an` |
| Exécuter des processus | `ps -ef` | `tasklist` |


Injection de commandes OS
-------------------------

Dans cet exemple, une application d'achat permet à l'utilisateur de voir si un article est en stock dans un magasin particulier. Ces informations sont accessibles via une URL:

`https://insecure-website.com/stockStatus?productID=381&storeID=29`

Pour fournir les informations de stock, l'application doit interroger divers systèmes existants. Pour des raisons historiques, la fonctionnalité est implémentée en appelant une commande shell avec les ID de produit et de magasin comme arguments:

`stockreport.pl 381 29`

Cette commande affiche l'état du stock pour l'élément spécifié, qui est retourné à l'utilisateur.

Injection de commandes OS - Suite
L'application n'implémente aucune défense contre l'injection de commande OS, de sorte qu'un attaquant peut soumettre l'entrée suivante pour exécuter une commande arbitraire:

& echo aiwefwlguh &
Si cette entrée est soumise dans le productID paramètre, la commande exécutée par l'application est:

stockreport.pl & echo aiwefwlguh & 29
Le echo la commande provoque l'écho de la chaîne fournie dans la sortie. C'est un moyen utile de tester certains types d'injection de commande OS. Le & character est un séparateur de commandes shell. Dans cet exemple, il provoque l'exécution de trois commandes distinctes, l'une après l'autre. La sortie retournée à l'utilisateur est:

Error - productID was not provided
aiwefwlguh
29: command not found
Les trois lignes de sortie démontrent que:

L'original stockreport.pl la commande a été exécutée sans ses arguments attendus, et a donc retourné un message d'erreur.
Le injecté echo la commande a été exécutée, et la chaîne fournie a été reprise dans la sortie.
L'argument initial 29 a été exécuté en tant que commande, ce qui a provoqué une erreur.
Placement du séparateur de commandes supplémentaire & après la commande injectée est utile car elle sépare la commande injectée de tout ce qui suit le point d'injection. Cela réduit les chances que ce qui suit empêche la commande injectée de s'exécuter.
