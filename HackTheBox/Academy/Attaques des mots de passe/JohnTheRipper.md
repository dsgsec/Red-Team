# JohnTheRipper

## Type d'attaques

Il existe différents type d'attaques réalisables avec `John`.

### Dictionnaire
Les attaques par dictionnaire impliquent l'utilisation d'une liste pré-générée de mots et de phrases (appelée dictionnaire) pour tenter de déchiffrer un mot de passe. Cette liste de mots et d'expressions est souvent acquise à partir de diverses sources, telles que des dictionnaires accessibles au public, des mots de passe ayant fait l'objet d'une fuite ou même achetée auprès d'entreprises spécialisées. Le dictionnaire est ensuite utilisé pour générer une série de chaînes qui sont ensuite utilisées pour comparer avec les mots de passe hachés. Si une correspondance est trouvée, le mot de passe est déchiffré, permettant à un attaquant d'accéder au système et aux données qui y sont stockées. Ce type d'attaque est très efficace. Par conséquent, il est essentiel de prendre les mesures nécessaires pour garantir la sécurité des mots de passe, comme l'utilisation de mots de passe complexes et uniques, leur modification régulière et l'utilisation d'une authentification à deux facteurs.

### Bruteforce
Les attaques par force brute impliquent de tenter toutes les combinaisons imaginables de caractères pouvant former un mot de passe. Il s'agit d'un processus extrêmement lent, et l'utilisation de cette méthode n'est généralement conseillée que s'il n'y a pas d'autres alternatives. Il est également important de noter que plus le mot de passe est long et complexe, plus il est difficile à déchiffrer et plus il faudra de temps pour épuiser chaque combinaison. Pour cette raison, il est fortement recommandé que les mots de passe comportent au moins 8 caractères, avec une combinaison de lettres, de chiffres et de symboles.

### Rainbow Table Attacks
Les attaques par table arc-en-ciel impliquent l'utilisation d'une table pré-calculée de hachages et de leurs mots de passe en clair correspondants, ce qui est une méthode beaucoup plus rapide qu'une attaque par force brute. Cependant, cette méthode est limitée par la taille de la table arc-en-ciel - plus la table est grande, plus elle peut stocker de mots de passe et de hachages. De plus, en raison de la nature de l'attaque, il est impossible d'utiliser des tables arc-en-ciel pour déterminer le texte en clair des hachages qui ne sont pas déjà inclus dans la table. En conséquence, les attaques de table arc-en-ciel ne sont efficaces que contre les hachages déjà présents dans la table, ce qui fait que plus la table est grande, plus l'attaque est réussie.

## Cracking

### Single Crack
Le mode Single Crack est l'un des modes John les plus couramment utilisés pour tenter de déchiffrer des mots de passe à l'aide d'une seule liste de mots de passe. Il s'agit d'une attaque par force brute, ce qui signifie que tous les mots de passe de la liste sont essayés, un par un, jusqu'à ce que le bon soit trouvé. Cette méthode est le moyen le plus simple et le plus simple de déchiffrer les mots de passe et constitue donc un choix populaire pour ceux qui souhaitent accéder à un système sécurisé. C'est cependant loin d'être la méthode la plus efficace car cela peut prendre un temps indéfini pour déchiffrer un mot de passe, selon la longueur et la complexité du mot de passe en question. La syntaxe de base de la commande est :
```
dsgsec@htb[/htb]$ john --format=<hash_type> <hash or hash_file>
```

Par exemple, si nous avons un fichier nommé hashes_to_crack.txt qui contient des hachages SHA-256, la commande pour les craquer serait :
```
dsgsec@htb[/htb]$ john --format=sha256 hashes_to_crack.txt
```

1. **john** est la commande pour exécuter le programme John the Ripper
2. **--format=sha256** spécifie que le format de hachage est SHA-256
3. **hashes.txt** est le nom du fichier contenant les hachages à cracker