Technique Keyspace et CUPP
==================================================================

### Technique de l'espace clé

Une autre façon de préparer une liste de mots consiste à utiliser la technique de l'espace clé. Dans cette technique, nous spécifions une gamme de caractères, de chiffres et de symboles dans notre liste de mots. crunch  est l'un des nombreux outils puissants pour créer une liste de mots hors ligne. Avec  crunch , nous pouvons spécifier de nombreuses options, notamment min, max et options comme suit :

croquer

```
user@thm$ crunch -h
crunch version 3.6

Crunch can create a wordlist based on the criteria you specify.
The output from crunch can be sent to the screen, file, or to another program.

Usage: crunch   [options]
where min and max are numbers

Please refer to the man page for instructions and examples on how to use crunch.
```

L'exemple suivant crée une liste de mots contenant toutes les combinaisons possibles de 2 caractères, y compris 0-4 et ad. Nous pouvons utiliser l'  argument -o et spécifier un fichier pour enregistrer la sortie dans .  

croquer

```
user@thm$ crunch 2 2 01234abcd -o crunch.txt
Crunch will now generate the following amount of data: 243 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: xx
crunch: 100% completed generating output
```

Voici un extrait de la sortie :

croquer

```
user@thm$ cat crunch.txt
00
01
02
03
04
0a
0b
0c
0d
10
.
.
.
cb
cc
cd
d0
d1
d2
d3
d4
da
db
dc
dd

```

Il convient de noter que crunch peut générer un fichier texte très volumineux en fonction de la longueur du mot et des options de combinaison que vous spécifiez. La commande suivante crée une liste d'une longueur minimale et maximale de 8 caractères contenant des chiffres de 0 à 9, des lettres minuscules af et des lettres majuscules AF :

crunch 8 8 0123456789abcdefABCDEF -o crunch.txt  le fichier généré fait 459 Go  et contient 54875873536 mots.

crunch  nous permet également de spécifier un jeu de caractères en utilisant l'option -t pour combiner les mots de notre choix. Voici quelques-unes des autres options qui pourraient être utilisées pour vous aider à créer différentes combinaisons de votre choix :

@  - caractères alpha minuscules

,  - caractères alpha majuscules

%  - caractères numériques

^  - caractères spéciaux, y compris l'espace

Par exemple, si une partie du mot de passe nous est connue et que nous savons qu'il commence par  pass  et suit deux chiffres, nous pouvons utiliser le  symbole %  ci-dessus pour faire correspondre les chiffres. Ici, nous générons une liste de mots qui contient  pass  suivi de 2 chiffres :

croquer

```
user@thm$  crunch 6 6 -t pass%%
Crunch will now generate the following amount of data: 700 bytes
0 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 100
pass00
pass01
pass02
pass03
```

### CUPP - Profileur de mots de passe utilisateur communs

CUPP est un outil automatique et interactif écrit en Python pour créer des listes de mots personnalisées. Par exemple, si vous connaissez certains détails sur une cible spécifique, tels que sa date de naissance, le nom de son animal de compagnie, le nom de sa société, etc., cela pourrait être un outil utile pour générer des mots de passe basés sur ces informations connues. Le CUPP prendra les informations fournies et générera une liste de mots personnalisée basée sur ce qui est fourni. Il existe également un support pour un  mode 1337/leet , qui remplace les lettres  a ,  i , e ,  t ,  o ,  s ,  g ,  z   par des chiffres. Par exemple, remplacez  a   par  4   ou  i avec  1 . Pour plus d'informations sur l'outil, veuillez visiter le référentiel GitHub  [ici](https://github.com/Mebus/cupp) .

Pour exécuter CUPP, nous avons besoin que Python 3 soit installé. Ensuite, clonez le dépôt GitHub sur votre machine locale en utilisant git comme suit :

CUPP

```
user@thm$  git clone https://github.com/Mebus/cupp.git
Cloning into 'cupp'...
remote: Enumerating objects: 237, done.
remote: Total 237 (delta 0), reused 0 (delta 0), pack-reused 237
Receiving objects: 100% (237/237), 2.14 MiB | 1.32 MiB/s, done.
Resolving deltas: 100% (125/125), done.

```

Changez maintenant le répertoire courant en CUPP et exécutez  python3 cupp.py ou avec  -h pour voir les options disponibles.  

CUPP

```
user@thm$  python3 cupp.py
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

usage: cupp.py [-h] [-i | -w FILENAME | -l | -a | -v] [-q]

Common User Passwords Profiler

optional arguments:
  -h, --help         show this help message and exit
  -i, --interactive  Interactive questions for user password profiling
  -w FILENAME        Use this option to improve existing dictionary, or WyD.pl output to make some pwnsauce
  -l                 Download huge wordlists from repository
  -a                 Parse default usernames and passwords directly from Alecto DB. Project Alecto uses purified
                     databases of Phenoelit and CIRT which were merged and enhanced
  -v, --version      Show the version of this program.
  -q, --quiet        Quiet mode (don't print banner)
```

CUPP prend en charge un mode interactif où il pose des questions sur la cible et sur la base des réponses fournies, il crée une liste de mots personnalisée. Si vous n'avez pas de réponse pour le champ donné, ignorez-le en appuyant sur la  touche Entrée  .

CUPP

```
user@thm$  python3 cupp.py -i
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name:
> Surname:
> Nickname:
> Birthdate (DDMMYYYY):

> Partners) name:
> Partners) nickname:
> Partners) birthdate (DDMMYYYY):

> Child's name:
> Child's nickname:
> Child's birthdate (DDMMYYYY):

> Pet's name:
> Company name:

> Do you want to add some key words about the victim? Y/[N]:
> Do you want to add special chars at the end of words? Y/[N]:
> Do you want to add some random numbers at the end of words? Y/[N]:
> Leet mode? (i.e. leet = 1337) Y/[N]:

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to .....txt, counting ..... words.
> Hyperspeed Print? (Y/n)

```

ِEn conséquence, une liste de mots personnalisée contenant différents nombres de mots en fonction de vos entrées est générée. Les listes de mots pré-créées peuvent être téléchargées sur votre machine comme suit :

CUPP

```
user@thm$  python3 cupp.py -l
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

        Choose the section you want to download:

     1   Moby            14      french          27      places
     2   afrikaans       15      german          28      polish
     3   american        16      hindi           29      random
     4   aussie          17      hungarian       30      religion
     5   chinese         18      italian         31      russian
     6   computer        19      japanese        32      science
     7   croatian        20      latin           33      spanish
     8   czech           21      literature      34      swahili
     9   danish          22      movieTV         35      swedish
    10   databases       23      music           36      turkish
    11   dictionaries    24      names           37      yiddish
    12   dutch           25      net             38      exit program
    13   finnish         26      norwegian

        Files will be downloaded from http://ftp.funet.fi/pub/unix/security/passwd/crack/dictionaries/ repository

        Tip: After downloading wordlist, you can improve it with -w option

> Enter number:
```

En fonction de vos intérêts, vous pouvez choisir la liste de mots dans la liste ci-dessus pour vous aider à générer des listes de mots pour le forçage brutal !

Enfin, CUPP pourrait également fournir des noms d'utilisateur et des mots de passe par défaut à partir de la base de données Alecto en utilisant l'  option -a  . 

CUPP

```
user@thm$  python3 cupp.py -a
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]

[+] Checking if alectodb is not present...
[+] Downloading alectodb.csv.gz from https://github.com/yangbh/Hammer/raw/b0446396e8d67a7d4e53d6666026e078262e5bab/lib/cupp/alectodb.csv.gz ...

[+] Exporting to alectodb-usernames.txt and alectodb-passwords.txt
[+] Done.
```
