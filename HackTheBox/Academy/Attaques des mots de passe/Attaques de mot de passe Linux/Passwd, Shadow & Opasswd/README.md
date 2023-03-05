# Passwd, Shadow & Opasswd
Les distributions basées sur Linux peuvent utiliser de nombreux mécanismes d'authentification différents. L'un des mécanismes les plus couramment utilisés et les plus standard est les modules d'authentification enfichables (PAM). Les modules utilisés pour cela sont appelés pam_unix.so ou pam_unix2.so et sont situés dans /usr/lib/x86_x64-linux-gnu/security/ dans les distributions basées sur Debian. Ces modules gèrent les informations utilisateur, l'authentification, les sessions, les mots de passe actuels et les anciens mots de passe. Par exemple, si nous voulons changer le mot de passe de notre compte sur le système Linux avec passwd, PAM est appelé, qui prend les précautions appropriées et stocke et gère les informations en conséquence.

Le module standard pam_unix.so pour la gestion utilise des appels d'API standardisés à partir des bibliothèques et fichiers système pour mettre à jour les informations de compte. Les fichiers standard qui sont lus, gérés et mis à jour sont /etc/passwd et /etc/shadow. PAM possède également de nombreux autres modules de service, tels que LDAP, mount ou Kerberos.

## Fichier passwd
Le fichier /etc/passwd contient des informations sur chaque utilisateur existant sur le système et peut être lu par tous les utilisateurs et services. Chaque entrée du fichier /etc/passwd identifie un utilisateur sur le système. Chaque entrée comporte sept champs contenant un formulaire de base de données avec des informations sur l'utilisateur particulier, où deux points ( : ) séparent les informations. En conséquence, une telle entrée peut ressembler à ceci :

| `cry0l1t3` | `:` | `x` | `:` | `1000` | `:` | `1000` | `:` | `cry0l1t3,,,` | `:` | `/home/cry0l1t3` | `:` | `/bin/bash` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Login name |  | Password info |  | UID |  | GUID |  | Full name/comments |  | Home directory |  | Shell |

Le champ le plus intéressant pour nous est le champ Informations sur le mot de passe dans cette section car il peut y avoir différentes entrées ici. L'un des cas les plus rares que l'on ne trouve que sur des systèmes très anciens est le hachage du mot de passe crypté dans ce champ. Les systèmes modernes ont les valeurs de hachage stockées dans le fichier /etc/shadow, sur lequel nous reviendrons plus tard. Néanmoins, /etc/passwd est lisible à l'échelle du système, ce qui donne aux attaquants la possibilité de déchiffrer les mots de passe si des hachages sont stockés ici.

Habituellement, nous trouvons la valeur x dans ce champ, ce qui signifie que les mots de passe sont stockés sous forme cryptée dans le fichier /etc/shadow. Cependant, il se peut aussi que le fichier /etc/passwd soit inscriptible par erreur. Cela nous permettrait d'effacer ce champ pour l'utilisateur root afin que le champ d'informations sur le mot de passe soit vide. Cela empêchera le système d'envoyer une invite de mot de passe lorsqu'un utilisateur essaie de se connecter en tant que root.

Modification de /etc/passwd - Avant
```
root:x:0:0:root:/root:/bin/bash
```


Modification de /etc/passwd - Après
```
root::0:0:root:/root:/bin/bash
```

Root sans mot de passe
```
[cry0l1t3@parrot]─[~]$ head -n 1 /etc/passwd

root::0:0:root:/root:/bin/bash


[cry0l1t3@parrot]─[~]$ su

[root@parrot]─[/home/cry0l1t3]#
```

Même si les cas présentés se produiront rarement, nous devons toujours faire attention et surveiller les failles de sécurité car certaines applications nous obligent à définir des autorisations spécifiques pour des dossiers entiers. Si l'administrateur a peu d'expérience avec Linux ou les applications et leurs dépendances, l'administrateur peut donner des permissions d'écriture au répertoire /etc et oublier de les corriger.

## Shadow File
Étant donné que la lecture des valeurs de hachage des mots de passe peut mettre tout le système en danger, le fichier /etc/shadow a été développé, qui a un format similaire à /etc/passwd mais n'est responsable que des mots de passe et de leur gestion. Il contient toutes les informations de mot de passe pour les utilisateurs créés. Par exemple, s'il n'y a pas d'entrée dans le fichier /etc/shadow pour un utilisateur dans /etc/passwd, l'utilisateur est considéré comme non valide. Le fichier /etc/shadow n'est également lisible que par les utilisateurs disposant de droits d'administrateur. Le format de ce fichier est divisé en neuf champs :

| `cry0l1t3` | `:` | `$6$wBRzy$...SNIP...x9cDWUxW1` | `:` | `18937` | `:` | `0` | `:` | `99999` | `:` | `7` | `:` | `:` | `:` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Username |  | Encrypted password |  | Last PW change |  | Min. PW age |  | Max. PW age |  | Warning period | Inactivity period | Expiration date | Unused |

### Fichier Shadow
```
[cry0l1t3@parrot]─[~]$ sudo cat /etc/shadow

root:*:18747:0:99999:7:::
sys:!:18747:0:99999:7:::
...SNIP...
cry0l1t3:$6$wBRzy$...SNIP...x9cDWUxW1:18937:0:99999:7:::
```

Si le champ du mot de passe contient un caractère, tel que ! ou *, l'utilisateur ne peut pas se connecter avec un mot de passe Unix. Cependant, d'autres méthodes d'authentification pour la connexion, telles que Kerberos ou l'authentification basée sur une clé, peuvent toujours être utilisées. Le même cas s'applique si le champ du mot de passe crypté est vide. Cela signifie qu'aucun mot de passe n'est requis pour la connexion. Cependant, cela peut conduire à des programmes spécifiques refusant l'accès aux fonctions. Le mot de passe crypté a également un format particulier par lequel nous pouvons également trouver certaines informations :

```
$<type>$<salt>$<hashed>
```

Comme nous pouvons le voir ici, les mots de passe cryptés sont divisés en trois parties. Les types de cryptage nous permettent de faire la distinction entre les éléments suivants :

Comme nous pouvons le voir ici, les mots de passe cryptés sont divisés en trois parties. Les types de cryptage nous permettent de faire la distinction entre les éléments suivants :

Types d'algorithmes
+ $1$ – MD5
+ $2a$ – Poisson-globe
+ $2y$ – Eksblowfish
+ $5$ – SHA-256
+ $6$ – SHA-512

Par défaut, la méthode de chiffrement SHA-512 ($6$) est utilisée sur les dernières distributions Linux. On retrouvera également les autres méthodes de chiffrement que l'on pourra ensuite essayer de cracker sur des systèmes plus anciens. Nous discuterons un peu du fonctionnement de la fissuration.

## Opasswd
La bibliothèque PAM (pam_unix.so) peut empêcher la réutilisation d'anciens mots de passe. Le fichier dans lequel les anciens mots de passe sont stockés est le fichier /etc/security/opasswd. Des autorisations administrateur/racine sont également requises pour lire le fichier si les autorisations pour ce fichier n'ont pas été modifiées manuellement.

### Lire /etc/security/opasswd
```
dsgsec@htb[/htb]$ sudo cat /etc/security/opasswd

cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1
```

En regardant le contenu de ce fichier, nous pouvons voir qu'il contient plusieurs entrées pour l'utilisateur cry0l1t3, séparées par une virgule (,). Un autre point critique auquel il faut prêter attention est le type de hachage qui a été utilisé. En effet, l'algorithme MD5 ($1$) est beaucoup plus facile à cracker que SHA-512. Ceci est particulièrement important pour identifier les anciens mots de passe et peut-être même leur modèle, car ils sont souvent utilisés dans plusieurs services ou applications. Nous augmentons plusieurs fois la probabilité de deviner le mot de passe correct en fonction de son modèle.

### Craquage des identifiants Linux

Une fois que nous avons collecté des hachages, nous pouvons essayer de les déchiffrer de différentes manières pour obtenir les mots de passe en texte clair.

### Unshadow
```
dsgsec@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
dsgsec@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
dsgsec@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

### Hashcat - Cracking Unshadowed Hashes
```
dsgsec@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

### Hashcat - Cracking MD5 Hashes
```
dsgsec@htb[/htb]$ cat md5-hashes.list

qNDkF0zJ3v8ylCOrKB0kt0
E9uMSmiQeRh4pAAgzuvkq1
```
  
### Hashcat - Cracking MD5 Hashes
```
dsgsec@htb[/htb]$ hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```
