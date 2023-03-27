Autres opérateurs d'injection
=========================

* * * * *

Avant de poursuivre, essayons quelques autres opérateurs d'injection et voyons à quel point l'application Web les gérerait différemment.

* * * * *

AND Opérateur
------------

Nous pouvons commencer par l'opérateur `AND` (`&&`), de sorte que notre charge utile finale serait (`127.0.0.1 && whoami`), et la commande finale exécutée serait la suivante :

Code : bash

```
ping -c 1 127.0.0.1 && whoami

```

Comme nous le devrions toujours, essayons d'abord d'exécuter la commande sur notre machine virtuelle Linux pour nous assurer qu'il s'agit d'une commande qui fonctionne :

```
21y4d@htb[/htb]$ ping -c 1 127.0.0.1 && whoami

PING 127.0.0.1 (127.0.0.1) 56(84) octets de données.
64 octets à partir de 127.0.0.1 : icmp_seq=1 ttl=64 time=1,03 ms

--- 127.0.0.1 statistiques de ping ---
1 paquets transmis, 1 reçu, 0% de perte de paquets, temps 0ms
rtt min/moy/max/mdev = 1,034/1,034/1,034/0,000 ms
21a4j

```

Comme nous pouvons le voir, la commande s'exécute et nous obtenons le même résultat que nous avons obtenu précédemment. Essayez de vous référer au tableau des opérateurs d'injection de la section précédente et voyez en quoi l'opérateur `&&` est différent (si nous n'écrivons pas d'adresse IP et commençons directement par `&&`, la commande fonctionnerait-elle toujours ?).

Maintenant, nous pouvons faire la même chose qu'avant en copiant notre charge utile, en la collant dans notre requête HTTP dans `Burp Suite`, en l'encodant en URL, puis en l'envoyant : ![Basic Attack](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_AND.jpg)

Comme nous pouvons le voir, nous avons injecté avec succès notre commande et reçu la sortie attendue des deux commandes.

* * * * *

OR Opérateur
-----------

Enfin, essayons l'opérateur d'injection `OR` (`||`). L'opérateur `OR` n'exécute la deuxième commande que si la première commande échoue. Cela peut nous être utile dans les cas où notre injection casserait la commande d'origine sans avoir un moyen solide de faire fonctionner les deux commandes. Ainsi, l'utilisation de l'opérateur `OR` exécutera notre nouvelle commande si la première échoue.

Si nous essayons d'utiliser notre charge utile habituelle avec l'opérateur `||` (`127.0.0.1 || whoami`), nous verrons que seule la première commande s'exécutera :

```
21y4d@htb[/htb]$ ping -c 1 127.0.0.1 || whoami

PING 127.0.0.1 (127.0.0.1) 56(84) octets de données.
64 octets à partir de 127.0.0.1 : icmp_seq=1 ttl=64 time=0,635 ms

--- 127.0.0.1 statistiques de ping ---
1 paquets transmis, 1 reçu, 0% de perte de paquets, temps 0ms
rtt min/moy/max/mdev = 0,635/0,635/0,635/0,000 ms

```

Cela est dû au fonctionnement des commandes `bash`. Comme la première commande renvoie le code de sortie `0` indiquant une exécution réussie, la commande `bash` s'arrête et n'essaie pas l'autre commande. Il ne tenterait d'exécuter l'autre commande que si la première échouait et renvoyait un code de sortie `1`.

`Essayez d'utiliser la charge utile ci-dessus dans la requête HTTP et voyez comment l'application Web la gère.`

Essayons de casser intentionnellement la première commande en ne fournissant pas d'adresse IP et en utilisant directement l'opérateur `||` (`|| whoami`), de sorte que la commande `ping` échoue et que notre commande injectée soit exécutée :

```
21y4d@htb[/htb]$ ping -c 1 || whoami

ping : erreur d'utilisation : adresse de destination requise
21a4j

```

Comme nous pouvons le voir, cette fois, la commande `whoami` s'est exécutée après l'échec de la commande `ping` et nous a donné un message d'erreur. Alors, essayons maintenant la charge utile (`|| whoami`) dans notre requête HTTP : ![Basic Attack](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_OR.jpg)

Nous voyons que cette fois, nous n'avons obtenu que la sortie de la deuxième commande comme prévu. Avec cela, nous utilisons une charge utile beaucoup plus simple et obtenons un résultat beaucoup plus propre.

Ces opérateurs peuvent être utilisés pour différents types d'injection, comme les injections SQL, les injections LDAP, XSS, SSRF, XML, etc. Nous avons créé une liste des opérateurs les plus courants pouvant être utilisés pour les injections :

| Type d'injection | Opérateurs |
| --- | --- |
| Injection SQL | `'` `,` `;` `--` `/* */` |
| Injection de commandes | `;` `&&` |
| Injection LDAP | `*` `(` `)` `&` `|` |
| Injection XPath | `'` `or` `and` `not` `substring` `concat` `count` |
| Injection de commande du système d'exploitation | `;` `&` `|` |
| Injection de code | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^` |
| Traversée de répertoire/traversée de chemin de fichier | `../` `..\\` `%00` |
| Injection d'objet | `;` `&` `|` |
| Injection XQuery | `'` `;` `--` `/* */` |
| Injection de shellcode | `\x` `\u` `%u` `%n` |
| Injection d'en-tête | `\n` `\r\n` `\t` `%0d` `%0a` `%09` |

Gardez à l'esprit que ce tableau est incomplet et que de nombreuses autres options et opérateurs sont possibles. Cela dépend également fortement de l'environnement avec lequel nous travaillons et testons.