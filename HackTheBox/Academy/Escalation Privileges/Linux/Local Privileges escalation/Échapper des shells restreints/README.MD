Échapper des shell restreints
==================================

* * * * *

Un shell restreint est un type de shell qui limite la capacité de l'utilisateur à exécuter des commandes. Dans un shell restreint, l'utilisateur est uniquement autorisé à exécuter un ensemble spécifique de commandes ou uniquement autorisé à exécuter des commandes dans des répertoires spécifiques. Les shells restreints sont souvent utilisés pour fournir un environnement sûr aux utilisateurs susceptibles d'endommager accidentellement ou intentionnellement le système ou pour permettre aux utilisateurs d'accéder uniquement à certaines fonctionnalités du système. Certains exemples courants de shells restreints incluent le `rbash`shell sous Linux et le "shell à accès restreint" sous Windows.

#### RBASH

[Le shell Bourne restreint](https://www.gnu.org/software/bash/manual/html_node/The-Restricted-Shell.html) ( `rbash`) est une version restreinte du shell Bourne, un interpréteur de ligne de commande standard sous Linux qui limite la capacité de l'utilisateur à utiliser certaines fonctionnalités du shell Bourne, telles que le changement de répertoires, la définition ou la modification de variables d'environnement et l'exécution de commandes. dans d'autres répertoires. Il est souvent utilisé pour fournir un environnement sûr et contrôlé aux utilisateurs susceptibles d'endommager accidentellement ou intentionnellement le système.

#### RKSH

[Le shell Korn restreint](https://www.ibm.com/docs/en/aix/7.2?topic=r-rksh-command) ( `rksh`) est une version restreinte du shell Korn, un autre interpréteur de ligne de commande standard. Le `rksh`shell limite la capacité de l'utilisateur à utiliser certaines fonctionnalités du shell Korn, telles que l'exécution de commandes dans d'autres répertoires, la création ou la modification de fonctions shell et la modification de l'environnement shell.

#### RZSH

[Le shell Z restreint](https://manpages.debian.org/experimental/zsh/rzsh.1.en.html) ( `rzsh`) est une version restreinte du shell Z et est l'interpréteur de ligne de commande le plus puissant et le plus flexible. Le `rzsh`shell limite la capacité de l'utilisateur à utiliser certaines fonctionnalités du shell Z, telles que l'exécution de scripts shell, la définition d'alias et la modification de l'environnement shell.

Par exemple, les administrateurs utilisent souvent des shells restreints dans les réseaux d'entreprise pour fournir un environnement sûr et contrôlé aux utilisateurs susceptibles d'endommager accidentellement ou intentionnellement le système. En limitant la capacité de l'utilisateur à exécuter des commandes spécifiques ou à accéder à certains répertoires, les administrateurs peuvent s'assurer que les utilisateurs ne peuvent pas effectuer d'actions qui pourraient endommager le système ou compromettre la sécurité du réseau. De plus, les shells restreints peuvent donner aux utilisateurs un accès à certaines fonctionnalités du système uniquement, permettant aux administrateurs de contrôler les ressources et les fonctions disponibles pour chaque utilisateur.

Imaginez une entreprise avec un réseau de serveurs Linux hébergeant des applications et des services métier critiques. De nombreux utilisateurs, y compris des employés, des sous-traitants et des partenaires externes, accèdent au réseau. Pour protéger la sécurité et l'intégrité du réseau, l'équipe informatique de l'organisation a décidé de mettre en place des shells restreints pour tous les utilisateurs.

Pour cela, l'équipe informatique met en place plusieurs shells `rbash`, `rksh`et `rzsh`sur le réseau et affecte à chaque utilisateur un shell spécifique. Par exemple, les partenaires externes qui n'ont besoin d'accéder qu'à certaines fonctionnalités du réseau, telles que la messagerie électronique et le partage de fichiers, sont affectés à `rbash`des shells, ce qui limite leur capacité à exécuter des commandes spécifiques et à accéder à certains répertoires. Les sous-traitants qui ont besoin d'accéder à des fonctionnalités réseau plus avancées, telles que des serveurs de base de données et des serveurs Web, sont affectés à `rksh`des shells, ce qui leur offre plus de flexibilité mais limite toujours leurs capacités. Enfin, les employés qui ont besoin d'accéder au réseau à des fins spécifiques, telles que l'exécution d'applications ou de scripts spécifiques, sont affectés à`rzsh`shells, qui leur offrent le plus de flexibilité mais limitent toujours leur capacité à exécuter des commandes spécifiques et à accéder à certains répertoires.

Plusieurs méthodes peuvent être utilisées pour s'échapper d'un shell restreint. Certaines de ces méthodes impliquent l'exploitation de vulnérabilités dans le shell lui-même, tandis que d'autres impliquent l'utilisation de techniques créatives pour contourner les restrictions imposées par le shell. Voici quelques exemples de méthodes qui peuvent être utilisées pour s'échapper d'un shell restreint.

* * * * *

S'échapper
----------

Dans certains cas, il peut être possible de s'échapper d'un shell restreint en injectant des commandes dans la ligne de commande ou d'autres entrées acceptées par le shell. Par exemple, supposons que le shell autorise les utilisateurs à exécuter des commandes en les transmettant comme arguments à une commande intégrée. Dans ce cas, il peut être possible de s'échapper du shell en injectant des commandes supplémentaires dans l'argument.

#### Injection de commande

Imaginez que nous sommes dans un shell restreint qui nous permet d'exécuter des commandes en les passant comme arguments à la `ls`commande. Malheureusement, le shell ne nous permet d'exécuter la `ls`commande qu'avec un ensemble spécifique d'arguments, tels que `ls -l`ou `ls -a`, mais il ne nous permet pas d'exécuter d'autres commandes. Dans cette situation, nous pouvons utiliser l'injection de commande pour sortir du shell en injectant des commandes supplémentaires dans l'argument de la `ls`commande.

Par exemple, nous pourrions utiliser la commande suivante pour injecter une `pwd`commande dans l'argument de la `ls`commande :

  Injection de commande

```
dsgsec@htb[/htb]$ ls -l `pwd`

```

Cette commande entraînerait l' `ls`exécution de la commande avec l'argument `-l`, suivi de la sortie de la `pwd`commande. Étant donné que la `pwd`commande n'est pas limitée par le shell, cela nous permettrait d'exécuter la `pwd`commande et de voir le répertoire de travail actuel, même si le shell ne nous permet pas d'exécuter la `pwd`commande directement.

#### Substitution de commande

Une autre méthode pour s'échapper d'un shell restreint consiste à utiliser la substitution de commande. Cela implique d'utiliser la syntaxe de substitution de commande du shell pour exécuter une commande. Par exemple, imaginez que le shell permette aux utilisateurs d'exécuter des commandes en les entourant de backticks (`). Dans ce cas, il peut être possible de s'échapper du shell en exécutant une commande dans une substitution de backtick qui n'est pas restreinte par le shell.

#### Chaînage de commandes

Dans certains cas, il peut être possible de s'échapper d'un shell restreint en utilisant le chaînage de commandes. Nous aurions besoin d'utiliser plusieurs commandes dans une seule ligne de commande, séparées par un métacaractère shell, tel qu'un point-virgule ( `;`) ou une barre verticale ( `|`), pour exécuter une commande. Par exemple, si le shell permet aux utilisateurs d'exécuter des commandes séparées par des points-virgules, il peut être possible de s'échapper du shell en utilisant un point-virgule pour séparer deux commandes, dont l'une n'est pas restreinte par le shell.

#### Variables d'environnement

Pour s'échapper d'un shell restreint pour utiliser des variables d'environnement, il faut modifier ou créer des variables d'environnement que le shell utilise pour exécuter des commandes qui ne sont pas restreintes par le shell. Par exemple, si le shell utilise une variable d'environnement pour spécifier le répertoire dans lequel les commandes sont exécutées, il peut être possible de s'échapper du shell en modifiant la valeur de la variable d'environnement pour spécifier un répertoire différent.

#### Fonctions de l'interpréteur de commandes

Dans certains cas, il peut être possible de s'échapper d'un shell restreint en utilisant les fonctions du shell. Pour cela, nous pouvons définir et appeler des fonctions shell qui exécutent des commandes non restreintes par le shell. Disons que le shell permet aux utilisateurs de définir et d'appeler des fonctions shell, il peut être possible d'échapper au shell en définissant une fonction shell qui exécute une commande.
