Attaques hors ligne -Dictionnaire et Brute-Force
=======================================

Cette section traite des attaques hors ligne, y compris les attaques par dictionnaire, par force brute et basées sur des règles.

### Attaque de dictionnaire

Une attaque par dictionnaire est une technique utilisée pour deviner des mots de passe en utilisant des mots ou des phrases bien connus. L'attaque par dictionnaire repose entièrement sur des listes de mots pré-collectées qui ont été précédemment générées ou trouvées. Il est important de choisir ou de créer la meilleure liste de mots candidats pour votre cible afin de réussir cette attaque. Explorons comment effectuer une attaque par dictionnaire en utilisant ce que vous avez appris dans les tâches précédentes sur la génération de listes de mots. Nous présenterons une attaque par dictionnaire hors ligne utilisant  hashcat , qui est un outil populaire pour casser les hachages.

Disons que nous obtenons le hachage suivant f806fc5a2a0d5ba2471600758452799c et  que nous voulons effectuer une attaque par dictionnaire pour le casser. Tout d'abord, nous devons connaître au minimum les éléments suivants :

1- De quel type de hachage s'agit-il ?\
2- Quelle liste de mots allons-nous utiliser ? Ou quel type de mode d'attaque pourrions-nous utiliser ?

Pour identifier le type de hachage, nous pourrions utiliser un outil tel que  hashid ou  hash-identifier .  Pour cet exemple,  hash-identifier pense que la méthode de hachage possible est  MD5 . Veuillez noter que le temps de casser un hachage dépendra du matériel que vous utilisez (CPU et/ou GPU).  

Attaque de dictionnaire

```
user@machine$ hashcat -a 0 -m 0 f806fc5a2a0d5ba2471600758452799c /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...
f806fc5a2a0d5ba2471600758452799c:rockyou

Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: f806fc5a2a0d5ba2471600758452799c
Time.Started.....: Mon Oct 11 08:20:50 2021 (0 secs)
Time.Estimated...: Mon Oct 11 08:20:50 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   114.1 kH/s (0.02ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 40/40 (100.00%)
Rejected.........: 0/40 (0.00%)
Restore.Point....: 0/40 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: 123456 -> 123123

Started: Mon Oct 11 08:20:49 2021
Stopped: Mon Oct 11 08:20:52 2021
```

-a 0   définit le mode d'attaque sur une attaque par dictionnaire

-m 0   définit le mode de hachage pour craquer les hachages MD5 ; pour les autres types, exécutez hashcat -h pour obtenir une liste des hachages pris en charge.

f806fc5a2a0d5ba2471600758452799c  cette option peut être un seul hachage comme dans notre exemple ou un fichier contenant un hachage ou plusieurs hachages.

/usr/share/wordlists/rockyou.txt  le fichier de liste de mots/dictionnaire pour notre attaque

Nous exécutons hashcat avec l'option --show pour afficher la valeur fissurée si le hachage a été fissuré :

Attaque de dictionnaire

```
user@machine$ hashcat -a 0 -m 0 F806FC5A2A0D5BA2471600758452799C /usr/share/wordlists/rockyou.txt --show
f806fc5a2a0d5ba2471600758452799c:rockyou
```

En conséquence, la valeur fissurée est rockyou .

### Attaque de force brute

Le forçage brutal est une attaque courante utilisée par l'attaquant pour obtenir un accès non autorisé à un compte personnel. Cette méthode est utilisée pour deviner le mot de passe de la victime en envoyant des combinaisons de mots de passe standard. La principale différence entre un dictionnaire et une attaque par force brute est qu'une attaque par dictionnaire utilise une liste de mots qui contient tous les mots de passe possibles.

En revanche, une attaque par force brute vise à essayer toutes les combinaisons d'un personnage ou de personnages. Par exemple, supposons que nous ayons un compte bancaire auquel nous avons besoin d'un accès non autorisé. Nous savons que le code PIN contient 4 chiffres comme mot de passe. Nous pouvons effectuer une attaque par force brute qui commence de  0000  à  9999  pour deviner le code PIN valide en fonction de cette connaissance. Dans d'autres cas, une séquence de chiffres ou de lettres peut être ajoutée à des mots existants dans une liste, comme  admin0 , admin1 , .. admin9999 .  

Par exemple, hashcat a des options de jeu de caractères qui pourraient être utilisées pour générer vos propres combinaisons. Les jeux de caractères peuvent être trouvés dans  les options d'aide de hashcat .

Attaque de force brute

```
user@machine$ hashcat --help
 ? | Charset
 ===+=========
  l | abcdefghijklmnopqrstuvwxyz
  u | ABCDEFGHIJKLMNOPQRSTUVWXYZ
  d | 0123456789
  h | 0123456789abcdef
  H | 0123456789ABCDEF
  s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
  a | ?l?u?d?s
  b | 0x00 - 0xff
```

L'exemple suivant montre comment nous pouvons utiliser  hashcat  avec le mode d'attaque par force brute avec une combinaison de notre choix . 

Attaque de force brute

```
user@machine$ hashcat -a 3 ?d?d?d?d --stdout
1234
0234
2234
3234
9234
4234
5234
8234
7234
6234
..
..
```

-a 3   définit le mode d'attaque comme une attaque par force brute

?d?d?d?d  le ?d indique au hashcat d'utiliser un chiffre. Dans notre cas, ?d?d?d?d pour quatre chiffres commençant par 0000 et se terminant par 9999

--stdout affiche le résultat sur le terminal 

Appliquons maintenant le même concept pour casser le  hachage MD5 suivant :  05A5CF06982BA7892ED2A6D38FE832D6  un code PIN à quatre chiffres.

Attaque de force brute

```
user@machine$ hashcat -a 3 -m 0 05A5CF06982BA7892ED2A6D38FE832D6 ?d?d?d?d
05a5cf06982ba7892ed2a6d38fe832d6:2021

Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: 05a5cf06982ba7892ed2a6d38fe832d6
Time.Started.....: Mon Oct 11 10:54:06 2021 (0 secs)
Time.Estimated...: Mon Oct 11 10:54:06 2021 (0 secs)
Guess.Mask.......: ?d?d?d?d [4]
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 16253.6 kH/s (0.10ms) @ Accel:1024 Loops:10 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10000/10000 (100.00%)
Rejected.........: 0/10000 (0.00%)
Restore.Point....: 0/1000 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-10 Iteration:0-10
Candidates.#1....: 1234 -> 6764

Started: Mon Oct 11 10:54:05 2021
Stopped: Mon Oct 11 10:54:08 2021
```
