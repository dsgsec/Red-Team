Attaques hors ligne - Basé sur des règles
=========================================

Les attaques basées sur des règles sont également appelées  attaques hybrides . Les attaques basées sur des règles supposent que l'attaquant sait quelque chose sur la politique de mot de passe. Les règles sont appliquées pour créer des mots de passe conformément aux directives de la politique de mot de passe donnée et ne devraient, en théorie, générer que des mots de passe valides. L'utilisation de listes de mots préexistantes peut être utile lors de la génération de mots de passe qui correspondent à une politique - par exemple, manipuler ou "manger" un mot de passe tel que "password": p@ssword , Pa  $$ word  , Passw0rd , etc. 

Pour cette attaque, nous pouvons étendre notre liste de mots en utilisant hashcat ou John the ripper . Cependant, pour cette attaque, voyons comment fonctionne John l'éventreur . Habituellement, John the ripper a un fichier de configuration contenant des ensembles de règles, qui se trouve dans /etc/john/john.conf  ou  /opt/john/john.conf selon votre distribution ou la façon dont john a été installé. Vous pouvez lire /etc/john/john.conf et rechercher  List.Rules pour voir toutes les règles disponibles :

Attaque basée sur des règles

```
user@machine$ cat /etc/john/john.conf|grep "List.Rules:" | cut -d"." -f3 | cut -d":" -f2 | cut -d"]" -f1 | awk NF
JumboSingle
o1
o2
i1
i2
o1
i1
o2
i2
best64
d3ad0ne
dive
InsidePro
T0XlC
rockyou-30000
specific
ShiftToggle
Split
Single
Extra
OldOffice
Single-Extra
Wordlist
ShiftToggle
Multiword
best64
Jumbo
KoreLogic
T9
```

Nous pouvons voir que nous avons de nombreuses règles à notre disposition. Nous allons créer une liste de mots avec un seul mot de passe contenant la chaîne  tryhackme , pour voir comment nous pouvons étendre la liste de mots. Choisissons l'une des règles, la  règle best64  , qui contient les 64 meilleures règles John intégrées, et voyons ce qu'elle peut faire !

Attaque basée sur des règles

```
user@machine$ john --wordlist=/tmp/single-password-list.txt --rules=best64 --stdout | wc -l
Using default input encoding: UTF-8
Press 'q' or Ctrl-C to abort, almost any other key for status
76p 0:00:00:00 100.00% (2021-10-11 13:42) 1266p/s pordpo
76
```

--wordlist=  pour spécifier la liste de mots ou le fichier dictionnaire. 

--rules  pour spécifier la ou les règles à utiliser.

--stdout  pour imprimer la sortie sur le terminal.

|wc -l   pour compter le nombre de lignes produites par John .

En exécutant la commande précédente, nous élargissons notre liste de mots de passe de 1 à 76 mots de passe. Vérifions maintenant une autre règle, l'une des meilleures règles de John,  KoreLogic . KoreLogic utilise diverses règles intégrées et personnalisées pour générer des listes de mots de passe complexes. Pour plus d'informations, veuillez visiter ce site  [ici](https://contest-2010.korelogic.com/rules.html) . Utilisons maintenant cette règle et vérifions si le  Tryh@ckm3 est disponible dans notre liste !  [](https://contest-2010.korelogic.com/rules.html) 

Attaque basée sur des règles

```
user@machine$ john --wordlist=single-password-list.txt --rules=KoreLogic --stdout |grep "Tryh@ckm3"
Using default input encoding: UTF-8
Press 'q' or Ctrl-C to abort, almost any other key for status
Tryh@ckm3
7089833p 0:00:00:02 100.00% (2021-10-11 13:56) 3016Kp/s tryhackme999999
```

La sortie de la commande précédente montre que notre liste a la version complexe de tryhackme , qui est Tryh@ckm3 . Enfin, nous vous recommandons de vérifier toutes les règles et de trouver celle qui vous convient le mieux. De nombreuses règles appliquent des combinaisons à une liste de mots existante et élargissent la liste de mots pour augmenter les chances de trouver un mot de passe valide !  

### Règles personnalisées

John l'éventreur  a beaucoup à offrir. Par exemple, nous pouvons créer nos propres règles et les utiliser au moment de l'exécution pendant que John craque le hachage ou utiliser la règle pour créer une liste de mots personnalisée !

Disons que nous voulions créer une liste de mots personnalisée à partir d'un dictionnaire préexistant avec une modification personnalisée du dictionnaire d'origine. Le but est d'ajouter des caractères spéciaux (ex : !@#$*&) au début de chaque mot et d'ajouter les chiffres 0-9 à la fin. Le format sera le suivant :

[symboles]mot[0-9]

Nous pouvons ajouter notre règle à la fin de john.conf :

Jean Règles

```
user@machine$ sudo vi /etc/john/john.conf
[List.Rules:THM-Password-Attacks]
Az"[0-9]" ^[!@#$]
```

[List.Rules:THM-Password-Attacks]   spécifiez le nom de la règle THM-Password-Attacks.

Az  représente un seul mot de la liste de mots/dictionnaire d'origine en utilisant -p . 

"[0-9]"  ajoute un seul chiffre (de  0 à  9 ) à la fin du mot. Pour deux chiffres, nous pouvons ajouter  "[0-9][0-9]"  et ainsi de suite.    

^[!@#$]  ajoute un caractère spécial au début de chaque mot. ^  signifie le début de la ligne/du mot. Notez que changer  ^  en  $ ajoutera les caractères spéciaux à la fin de la ligne/du mot. 

Créons maintenant un fichier contenant un seul mot de  passe pour voir comment nous pouvons étendre notre liste de mots en utilisant cette règle. 

Jean Règles

```
user@machine$ echo "password" > /tmp/single.lst
```

Nous incluons le nom de la règle que nous avons créée dans la commande John en utilisant l'  option --rules  . Nous devons également afficher le résultat dans le terminal. Nous pouvons le faire en utilisant  --stdout  comme suit :

Jean Règles

```
user@machine$ john --wordlist=/tmp/single.lst --rules=THM-Password-Attacks --stdout
Using default input encoding: UTF-8
!password0
@password0
#password0
$password0
```

Il est maintenant temps de vous entraîner à créer votre propre règle.
