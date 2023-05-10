Découverte et énumération d'applications
===================================

* * * * *

Pour gérer efficacement son réseau, une organisation doit maintenir (et mettre à jour en permanence) un inventaire des actifs qui comprend tous les périphériques connectés au réseau (serveurs, postes de travail, appareils réseau, etc.), les logiciels installés et les applications utilisées dans l'environnement. Si une organisation n'est pas sûre de ce qui est présent sur son réseau, comment saura-t-elle ce qu'il faut protéger et quelles failles potentielles existent ? L'organisation doit savoir si les applications sont installées localement ou hébergées par un tiers, leur niveau de correctif actuel, si elles sont en fin de vie ou approchent de leur fin de vie, être en mesure de détecter toute application malveillante dans le réseau (ou « shadow IT ») , et disposent d'une visibilité suffisante sur chaque application pour s'assurer qu'elles sont correctement sécurisées avec des mots de passe forts (autres que ceux par défaut). Idéalement, l'authentification multifacteur est activée. Certaines applications ont des portails administratifs qui peuvent être limités à n'être accessibles qu'à partir d'adresses IP spécifiques ou de l'hôte lui-même (localhost).

La réalité est que de nombreuses organisations ne savent pas tout sur leur réseau, et certaines organisations ont très peu de visibilité, et nous pouvons les aider avec cela. L'énumération que nous effectuons peut être très bénéfique pour nos clients pour les aider à améliorer ou à commencer à construire un inventaire d'actifs. Nous pouvons très probablement identifier des applications qui ont été oubliées, des versions de démonstration de logiciels dont la licence d'essai a peut-être expiré et converties en une version ne nécessitant pas d'authentification (dans le cas de Splunk), des applications avec des informations d'identification par défaut/faibles, des applications non autorisées/ les applications mal configurées et les applications qui souffrent de vulnérabilités publiques. Nous pouvons fournir ces données à nos clients sous la forme d'une combinaison des résultats de nos rapports (c'est-à-dire une application avec des informations d'identification par défaut `admin:admin`, sous forme d'annexes telles qu'une liste de services identifiés mappés à des hôtes ou des données d'analyse supplémentaires). Nous pouvons même aller plus loin et éduquer nos clients sur certains des outils que nous utilisons quotidiennement afin qu'ils puissent commencer à effectuer une reconnaissance périodique et proactive de leurs réseaux et trouver des lacunes avant que les testeurs d'intrusion, ou pire, les attaquants, ne les trouvent en premier.

En tant que testeurs d'intrusion, nous devons avoir de solides compétences en matière d'énumération et être en mesure d'obtenir la "configuration du terrain" sur n'importe quel réseau en commençant par très peu ou pas d'informations (découverte de boîte noire ou juste un ensemble de plages CIDR). Généralement, lorsque nous nous connectons à un réseau, nous commençons par un balayage ping pour identifier les "hôtes en direct". À partir de là, nous commencerons généralement une analyse de port ciblée et, éventuellement, une analyse de port plus approfondie pour identifier les services en cours d'exécution. Dans un réseau avec des centaines ou des milliers d'hôtes, ces données d'énumération peuvent devenir difficiles à manier. Disons que nous effectuons une analyse de port Nmap pour identifier les services Web courants tels que :

#### Nmap - Découverte Web

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ nmap -p 80,443,8000,8080,8180,8888,1000 --open -oA web_discovery -iL scope_list

```

Nous pouvons trouver une énorme quantité d'hôtes avec des services fonctionnant uniquement sur les ports 80 et 443. Que fait-on de ces données ? Passer au crible manuellement les données de dénombrement dans un environnement étendu prendrait beaucoup trop de temps, d'autant plus que la plupart des évaluations sont soumises à des contraintes de temps strictes. La navigation vers chaque adresse IP/nom d'hôte + port serait également très inefficace.

Heureusement pour nous, il existe plusieurs excellents outils qui peuvent grandement aider dans ce processus. Deux outils phénoménaux que chaque testeur devrait avoir dans son arsenal sont [EyeWitness](https://github.com/FortyNorthSecurity/EyeWitness) et [Aquatone](https://github.com/michenriksen/aquatone). Ces deux outils peuvent être alimentés en sortie d'analyse XML Nmap brute (Aquatone peut également prendre Masscan XML ; EyeWitness peut prendre la sortie XML Nessus) et être utilisés pour inspecter rapidement tous les hôtes exécutant des applications Web et prendre des captures d'écran de chacun. Les captures d'écran sont ensuite assemblées dans un rapport sur lequel nous pouvons travailler dans le navigateur Web pour évaluer la surface d'attaque Web.

Ces captures d'écran peuvent nous aider à réduire potentiellement des centaines d'hôtes et à créer une liste plus ciblée d'applications que nous devrions passer plus de temps à énumérer et à attaquer. Ces outils sont disponibles pour Windows et Linux, nous pouvons donc les utiliser sur tout ce que nous choisissons pour notre boîte d'attaque dans un environnement donné. Passons en revue quelques exemples de chacun pour créer un inventaire des applications présentes dans le domaine `INLANEFREIGHT.LOCAL` cible.

* * * * *

S'organiser
-----------------

Bien que nous couvrions la prise de notes, les rapports et la documentation dans un module séparé, il vaut la peine de saisir l'occasion de sélectionner une application de prise de notes si nous ne l'avons pas encore fait et de commencer à la configurer pour mieux enregistrer les données que nous recueillons dans cette phase. Le module [Getting Started](https://academy.hackthebox.com/course/preview/getting-started) aborde plusieurs applications de prise de notes. Si vous n'en avez pas choisi un à ce stade, ce serait un excellent moment pour commencer. Des outils comme OneNote, Evernote, Notion, Cherrytree, etc., sont toutes de bonnes options, et cela dépend de vos préférences personnelles. Quel que soit l'outil que vous choisissez, nous devrions travailler sur notre méthodologie de prise de notes à ce stade et créer des modèles que nous pouvons utiliser dans notre outil de choix configuré pour chaque type d'évaluation.

Pour cette section, je décomposerais la section "Énumération et découverte" de mon bloc-notes dans une section distincte "Découverte d'applications". Ici, je créerais des sous-sections pour la portée, les analyses (Nmap, Nessus, Masscan, etc.), les captures d'écran d'applications et les hôtes intéressants/notables pour approfondir plus tard. Il est important d'horodater chaque analyse que nous effectuons et d'enregistrer toutes les sorties et la syntaxe d'analyse exacte qui a été effectuée et les hôtes ciblés. Cela peut être utile plus tard si le client a des questions sur l'activité qu'il a vue pendant l'évaluation. Être organisé dès le départ et tenir des journaux et des notes détaillés nous aidera grandement avec le rapport final. J'installe généralement le squelette du rapport au début de l'évaluation avec mon cahier afin de pouvoir commencer à remplir certaines sections du rapport en attendant la fin de la numérisation. Tout cela nous fera gagner du temps à la fin de l'engagement, nous laissera plus de temps pour les choses amusantes (tester les erreurs de configuration et les exploits !) et garantira que nous soyons aussi minutieux que possible.

Un exemple de structure OneNote (également applicable à d'autres outils) peut ressembler à ce qui suit pour la phase de découverte :

`Test de pénétration externe - <Nom du client>`

- "Portée" (y compris les adresses/plages d'adresses IP couvertes, les URL, les hôtes fragiles, les délais de test et toutes les limitations ou autres informations relatives dont nous avons besoin à portée de main)

- `Points de contact clients`

- "Identifiants"

- `Découverte/Énumération`

     - `Scans`

     - `Hôtes en direct`

- "Découverte d'applications"

     - `Scans`
     - `Hôtes intéressants/remarquables`
- "Exploitation"

     - `<Nom d'hôte ou IP>`

     - `<Nom d'hôte ou IP>`

- `Post-Exploitation`

     - `<Nom d'hôte ou IP>`

     - `<<Nom d'hôte ou IP>`

Nous nous référerons à cette structure tout au long du module, il serait donc très utile de la reproduire et d'enregistrer tout notre travail sur ce module comme si nous travaillions sur un engagement réel. Cela nous aidera à affiner notre méthodologie de documentation, une compétence essentielle pour un testeur d'intrusion performant. Avoir des notes auxquelles se référer de chaque section sera utile lorsque nous arriverons aux deux évaluations des compétences à la fin du module et sera extrêmement utile à mesure que nous progresserons dans le parcours `Penetration Tester` .

* * * * *

Énumération initiale
-------------------

Supposons que notre client nous ait fourni la portée suivante :

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ cat scope_list

app.inlanefreight.local
dev.inlanefreight.local
drupal-dev.inlanefreight.local
drupal-qa.inlanefreight.local
drupal-acc.inlanefreight.local
drupal.inlanefreight.local
blog-dev.inlanefreight.local
blog.inlanefreight.local
app-dev.inlanefreight.local
jenkins-dev.inlanefreight.local
jenkins.inlanefreight.local
web01.inlanefreight.local
gitlab-dev.inlanefreight.local
gitlab.inlanefreight.local
support-dev.inlanefreight.local
support.inlanefreight.local
inlanefreight.local
10.129.201.50

```

Nous pouvons commencer par une analyse Nmap des ports Web courants. Je fais généralement une analyse initiale avec les ports `80,443,8000,8080,8180,8888,10000` puis j'exécute EyeWitness ou Aquatone (ou les deux en fonction des résultats de la première) par rapport à cette analyse initiale. Lors de l'examen du rapport de capture d'écran des ports les plus courants, je peux exécuter une analyse Nmap plus approfondie sur les 10 000 premiers ports ou sur tous les ports TCP, en fonction de la taille de l'étendue. Étant donné que l'énumération est un processus itératif, nous exécuterons un outil de capture d'écran Web sur toutes les analyses Nmap ultérieures que nous effectuerons pour assurer une couverture maximale.

Lors d'un test d'intrusion complet non évasif, j'exécute généralement également une analyse Nessus pour donner au client le meilleur rapport qualité-prix, mais nous devons être en mesure d'effectuer des évaluations sans compter sur des outils d'analyse. Même si la plupart des évaluations sont limitées dans le temps (et ne sont souvent pas adaptées à la taille de l'environnement), nous pouvons fournir à nos clients une valeur maximale en établissant une méthodologie d'énumération reproductible et approfondie qui peut être appliquée à tous les environnements que nous couvrons. Nous devons être efficaces lors de la phase de collecte/découverte des informations sans prendre de raccourcis qui pourraient laisser des failles critiques non découvertes. La méthodologie et les outils préférés de chacun varieront un peu, et nous devrions nous efforcer d'en créer un qui fonctionne bien pour nous tout en atteignant toujours le même objectif final.

Toutes les analyses que nous effectuons au cours d'un engagement non évasif visent à recueillir des données en tant qu'entrées pour notre processus de validation manuelle et de test manuel. Nous ne devons pas compter uniquement sur les scanners car l'élément humain dans les tests d'intrusion est essentiel. Nous trouvons souvent les vulnérabilités et les erreurs de configuration les plus uniques et les plus graves uniquement grâce à des tests manuels approfondis.

Creusons dans la liste de portée des hommesmentionné ci-dessus avec une analyse Nmap qui découvrira généralement la plupart des applications Web dans un environnement. Bien sûr, nous effectuerons des analyses plus approfondies plus tard, mais cela nous donnera un bon point de départ.

Remarque : tous les hôtes de la liste de portée ci-dessus ne seront pas accessibles lors de la génération de la cible ci-dessous. Il y aura des exercices séparés, similaires, à la fin de cette section afin de reproduire une grande partie de ce qui est montré ici.

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ sudo nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list

À partir de Nmap 7.80 ( https://nmap.org ) au 2021-09-07 21:49 EDT
Statistiques : 0:00:07 écoulé ; 1 hôtes terminés (4 en place), 4 en cours d'analyse furtive SYN
Synchronisation de l'analyse furtive SYN : environ 81,24 % effectués ; ETC : 21:49 (0:00:01 restant)

Rapport d'analyse Nmap pour app.inlanefreight.local (10.129.42.195)
L'hôte est actif (latence de 0,12 s).
Non illustré : 998 ports fermés
SERVICE DE L'ÉTAT DU PORT
22/tcp ouvrir ssh
80/tcp ouvert http

Rapport d'analyse Nmap pour app-dev.inlanefreight.local (10.129.201.58)
L'hôte est actif (latence de 0,12 s).
Non illustré : 993 ports fermés
SERVICE DE L'ÉTAT DU PORT
22/tcp ouvrir ssh
80/tcp ouvert http
8000/tcp ouvert http-alt
8009/tcp ouvert ajp13
8080/tcp open http-proxy
8180/tcp ouvert inconnu
8888/tcp open sun-answerbook

Rapport d'analyse Nmap pour gitlab-dev.inlanefreight.local (10.129.201.88)
L'hôte est actif (latence de 0,12 s).
Non illustré : 997 ports fermés
SERVICE DE L'ÉTAT DU PORT
22/tcp ouvrir ssh
80/tcp ouvert http
8081/tcp ouvrir la calotte glaciaire noire

Rapport d'analyse Nmap pour 10.129.201.50
L'hôte est actif (latence de 0,13 s).
Non illustré : 991 ports fermés
SERVICE DE L'ÉTAT DU PORT
80/tcp ouvert http
135/tcp ouvert msrpc
139/tcp ouvert netbios-ssn
445/tcp ouvrir microsoft-ds
3389/tcp ouvrir le serveur ms-wbt
5357/tcp ouvrir wsdapi
8000/tcp ouvert http-alt
8080/tcp open http-proxy
8089/tcp ouvert inconnu

<SNIP>

```

Comme nous pouvons le constater, nous avons identifié plusieurs hôtes exécutant des serveurs Web sur différents ports. D'après les résultats, nous pouvons déduire que l'un des hôtes est Windows et que les autres sont Linux (mais ne peuvent pas être sûrs à 100 % à ce stade). Portez également une attention particulière aux noms d'hôte. Dans cet atelier, nous utilisons des Vhosts pour simuler les sous-domaines d'une entreprise. Les hôtes avec `dev` dans le cadre du FQDN méritent d'être notés car ils peuvent exécuter des fonctionnalités non testées ou avoir des choses comme le mode de débogage activé. Parfois, les noms d'hôte ne nous disent pas grand-chose, comme `app.inlanefreight.local`. Nous pouvons en déduire qu'il s'agit d'un serveur d'applications, mais nous aurions besoin d'effectuer une énumération supplémentaire pour identifier les applications qui y sont exécutées.

Nous voudrions également ajouter `gitlab-dev.inlanefreight.local` à notre liste "d'hôtes intéressants" à explorer une fois la phase de découverte terminée. Nous pouvons être en mesure d'accéder à des référentiels Git publics qui pourraient contenir des informations sensibles telles que des informations d'identification ou des indices pouvant nous conduire à d'autres sous-domaines/Vhosts. Il n'est pas rare de trouver des instances Gitlab qui nous permettent d'enregistrer un utilisateur sans nécessiter l'approbation de l'administrateur pour activer le compte. Nous pouvons trouver des dépôts supplémentaires après la connexion. Il serait également utile de vérifier les commits précédents pour des données telles que les informations d'identification que nous couvrirons plus en détail plus tard dans ce module lorsque nous approfondirons Gitlab.

L'énumération supplémentaire de l'un des hôtes à l'aide d'une analyse de service Nmap (`-sV`) par rapport aux 1 000 premiers ports par défaut peut nous en dire plus sur ce qui s'exécute sur le serveur Web.

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ sudo nmap --open -sV 10.129.201.50

À partir de Nmap 7.80 ( https://nmap.org ) au 2021-09-07 21:58 EDT
Rapport d'analyse Nmap pour 10.129.201.50
L'hôte est actif (latence de 0,13 s).
Non illustré : 991 ports fermés
VERSION SERVICE À L'ÉTAT DU PORT
80/tcp ouvert http Microsoft IIS httpd 10.0
135/tcp ouvert msrpc Microsoft Windows RPC
139/tcp ouvert netbios-ssn Microsoft Windows netbios-ssn
445/tcp ouvrir microsoft-ds ?
3389/tcp open ms-wbt-server Microsoft Terminal Services
5357/tcp ouvert http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp ouvrir http Splunkd httpd
8080/tcp open http Indy httpd 17.3.33.2830 (moniteur de bande passante Paessler PRTG)
8089/tcp open ssl/http Splunkd httpd (licence gratuite ; connexion à distance désactivée)
Informations sur le service : système d'exploitation : Windows ; CPE : cpe:/o:microsoft:windows

Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 38,63 secondes

```

À partir de la sortie ci-dessus, nous pouvons voir qu'un serveur Web IIS s'exécute sur le port 80 par défaut, et il semble que `Splunk` s'exécute sur le port 8000/8089, tandis que `PRTG Network Monitor` est présent sur le port 8080. Si nous se trouvaient dans un environnement de taille moyenne à grande, ce type de dénombrement serait inefficace. Cela pourrait nous faire manquer une application Web qui pourrait s'avérer essentielle au succès de la mission.

* * * * *

Utiliser EyeWitness
----------------

Le premier est EyeWitness. Comme mentionné précédemment, EyeWitness peut prendre la sortie XML de Nmap et de Nessus et créer un rapport avec des captures d'écran de chaque application Web présente.sur les différents ports utilisant Selenium. Il va également aller plus loin et classer les applications dans la mesure du possible, les empreintes digitales et suggérer des informations d'identification par défaut en fonction de l'application. Il peut également recevoir une liste d'adresses IP et d'URL et être invité à ajouter `http://` et `https://` au début de chacune. Il effectuera une résolution DNS pour les adresses IP et peut se voir attribuer un ensemble spécifique de ports pour tenter de se connecter et de faire une capture d'écran.

Nous pouvons installer EyeWitness via apt :

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ sudo apt install eyewitness

```

ou clonez le [référentiel](https://github.com/FortyNorthSecurity/EyeWitness), accédez au répertoire `Python/setup` et exécutez le script d'installation `setup.sh`. EyeWitness peut également être exécuté à partir d'un conteneur Docker, et une version Windows est disponible, qui peut être compilée à l'aide de Visual Studio.

Exécuter `eyewitness -h` nous montrera les options qui s'offrent à nous :

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ eyewitness -h

utilisation : EyeWitness.py [--web] [-f NomFichier] [-x NomFichier.xml]
                      [--single URL unique] [--no-dns] [--timeout Délai]
                      [--jitter # de secondes] [--delay # de secondes]
                      [--threads nombre de fils]
                      [--max-retries Nombre maximal de tentatives après expiration du délai]
                      [-d Nom du répertoire] [--results Hôtes par page]
                      [--no-prompt] [--user-agent Agent utilisateur]
                      [--seuil de différence de différence]
                      [--proxy-ip 127.0.0.1] [--proxy-port 8080]
                      [--proxy-type socks5] [--show-selenium] [--resolve]
                      [--add-http-ports ADD_HTTP_PORTS]
                      [--add-https-ports ADD_HTTPS_PORTS]
                      [--only-ports ONLY_PORTS] [--prepend-https]
                      [--selenium-log-path SELENIUM_LOG_PATH] [--resume ew.db]
                      [--ocr]

EyeWitness est un outil utilisé pour capturer des captures d'écran à partir d'une liste d'URL

Protocoles :
   --web Capture d'écran HTTP utilisant Selenium

Options d'entrée :
   -f Filename Fichier séparé par des lignes contenant les URL à capturer
   -x NomFichier.xml Nmap Fichier XML ou .Nessus
   --single URL unique URL/hôte unique à capturer
   --no-dns Ignorer la résolution DNS lors de la connexion à des sites Web

Options de minutage :
   --timeout Délai d'attente Nombre maximal de secondes à attendre lors de la demande d'un
                         page Web (par défaut : 7)
   --jitter # de secondes
                         Randomisez les URL et ajoutez un délai aléatoire entre les requêtes
   --delay # of Seconds Délai entre l'ouverture du navigateur et la prise
                         la capture d'écran
   --threads # de fils
                         Nombre de threads à utiliser lors de l'utilisation d'une entrée basée sur un fichier
   --max-retries Nombre maximal de tentatives après expiration du délai
                         Nombre maximal de tentatives après expiration du délai d'attente

<SNIP>

```

Exécutons l'option `--web` par défaut pour prendre des captures d'écran en utilisant la sortie XML Nmap de l'analyse de découverte comme entrée.

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness

################################################# ##############################
# eyewitness #
################################################# ##############################
# Sécurité FortyNorth - https://www.fortynorthsecurity.com #
################################################# ##############################

Démarrage des requêtes Web (26 hôtes)
Tentative de capture d'écran http://app.inlanefreight.local
Tentative de capture d'écran http://app-dev.inlanefreight.local
Tentative de capture d'écran http://app-dev.inlanefreight.local:8000
Tentative de capture d'écran http://app-dev.inlanefreight.local:8080
Tentative de capture d'écran http://gitlab-dev.inlanefreight.local
Tentative de capture d'écran http://10.129.201.50
Tentative de capture d'écran http://10.129.201.50:8000
Tentative de capture d'écran http://10.129.201.50:8080
Tentative de capture d'écran http://dev.inlanefreight.local
Tentative de capture d'écran http://jenkins-dev.inlanefreight.local
Tentative de capture d'écran http://jenkins-dev.inlanefreight.local:8000
Tentative de capture d'écran http://jenkins-dev.inlanefreight.local:8080
Tentative de capture d'écran http://support-dev.inlanefreight.local
Tentative de capture d'écran http://drupal-dev.inlanefreight.local
[*] Atteindre le délai d'expiration lors de la connexion à http://10.129.201.50:8000, réessayer
Tentative de capture d'écran http://jenkins.inlanefreight.local
Tentative de capture d'écran http://jenkins.inlanefreight.local:8000
Tentative de capture d'écran http://jenkins.inlanefreight.local:8080
Tentative de capture d'écran http://support.inlanefreight.local
[*] A effectué 15 services sur 26
Tentative de capture d'écran http://drupal-qa.inlanefreight.local
Tentative de capture d'écran http://web01.inlanefreight.local
Tentative de capture d'écran http://web01.inlanefreight.local:8000
Tentative de capture d'écran http://web01.inlanefreight.local:8080
Tentative de screenshot http://inlanefreight.local
Tentative de capture d'écran http://drupal-acc.inlanefreight.local
Tentative de capture d'écran http://drupal.inlanefreight.local
Tentative de capture d'écran http://blog-dev.inlanefreight.local
Terminé en 57,859838008880615 secondes

[*] Fait! Rapport rédigé dans le dossier /home/mrb3n/Projects/inlanfreight/inlanefreight_eyewitness !
Souhaitez-vous ouvrir le rapport maintenant ? [O/n]

```

* * * * *

Utiliser Aquatone
--------------

[Aquatone](https://github.com/michenriksen/aquatone), comme mentionné précédemment, est similaire à EyeWitness et peut prendre des captures d'écran lorsqu'un fichier `.txt` d'hôtes ou un fichier Nmap `.xml` avec le ` -nmap` indicateur. Nous pouvons compiler Aquatone nous-mêmes ou télécharger un binaire précompilé. Après avoir téléchargé le binaire, il nous suffit de l'extraire et nous sommes prêts à partir.

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip

```

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ décompressez aquatone_linux_amd64_1.7.0.zip

Archive : aquatone_linux_amd64_1.7.0.zip
   gonflant : aquatone
   gonflage : README.md
   gonflage : LICENSE.txt

```

Nous pouvons le déplacer vers un emplacement de notre `$PATH` tel que `/usr/local/bin` pour pouvoir appeler l'outil de n'importe où ou simplement déposer le binaire dans notre répertoire de travail (par exemple, scans). C'est une préférence personnelle mais généralement plus efficace de construire nos machines virtuelles d'attaque avec la plupart des outils disponibles sans avoir à changer constamment de répertoires ou à les appeler depuis d'autres répertoires.

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ echo $CHEMIN

/home/mrb3n/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr /share/games:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

```

Dans cet exemple, nous fournissons à l'outil la même sortie Nmap `web_discovery.xml` spécifiant l'indicateur `-nmap` , et nous sommes partis pour les courses.

Nmap - Découverte Web

```
dsgsec@htb[/htb]$ cat web_discovery.xml | ./aquatone -nmap

aquatone v1.7.0 a commencé le 2021-09-07T22:31:03-04:00

Cibles : 65
Fils : 6
Ports : 80, 443, 8000, 8080, 8443
Rép sortie : .

http://web01.inlanefreight.local:8000/ : 403 Interdit
http://app.inlanefreight.local/ : 200 OK
http://jenkins.inlanefreight.local/ : 403 Interdit
http://app-dev.inlanefreight.local/ : 200
http://app-dev.inlanefreight.local/ : 200
http://app-dev.inlanefreight.local:8000/ : 403 Interdit
http://jenkins.inlanefreight.local:8000/ : 403 Interdit
http://web01.inlanefreight.local:8080/ : 200
http://app-dev.inlanefreight.local:8000/ : 403 Interdit
http://10.129.201.50:8000/ : 200 OK

<SNIP>

http://web01.inlanefreight.local:8000/ : capture d'écran réussie
http://app.inlanefreight.local/ : capture d'écran réussie
http://app-dev.inlanefreight.local/ : capture d'écran réussie
http://jenkins.inlanefreight.local/ : capture d'écran réussie
http://app-dev.inlanefreight.local/ : capture d'écran réussie
http://app-dev.inlanefreight.local:8000/ : capture d'écran réussie
http://jenkins.inlanefreight.local:8000/ : capture d'écran réussie
http://app-dev.inlanefreight.local:8000/ : capture d'écran réussie
http://app-dev.inlanefreight.local:8080/ : capture d'écran réussie
http://app.inlanefreight.local/ : capture d'écran réussie

<SNIP>

Calcul des structures de page... terminé
Regroupement des pages similaires... terminé
Génération du rapport HTML... terminé

Ecriture du fichier de session...Heure :
  - Commencé le : 2021-09-07T22:31:03-04:00
  - Terminé le : 2021-09-07T22:31:36-04:00
  - Durée : 33s

Demandes :
  - Réussi : 65
  - Échec : 0

  - 2xx : 47
  - 3xx : 0
  - 4xx : 18
  - 5xx : 0

Captures d'écran:
  - Réussi : 65
  - Échec : 0

A écrit le rapport HTML à : aquatone_report.html

```

* * * * *

Interprétation des résultats
------------------------

Même avec les 26 hôtes ci-dessus, ce rapport nous fera gagner du temps. Imaginez maintenant un environnement avec 500 ou 5 000 hôtes ! Après avoir ouvert le rapport, nous constatons qu'il est organisé en catégories, les `Cibles à forte valeur ajoutée' étant les premiers et généralement les hôtes les plus "juicy" à rechercher. J'ai exécuté EyeWitness dans de très grands environnements et j'ai généré des rapports avec des centaines de pages qui prennent des heures à parcourir. Souvent, les très gros rapports contiennent des hôtes intéressants profondément enfouis, il vaut donc la peine de tout revoir et de fouiller/rechercher toutes les applications que nous ne connaissons pas. J'ai trouvé l'application `ManageEngine OpManager` mentionnée dans la section d'introduction enfouie dans un rapport très volumineux lors d'un test d'intrusion externe. Cette instance a été laissée configurée avec les informations d'identification par défaut `admin:admin` et laissée largement ouverte à Internet. J'ai pu me connecter et réaliser l'exécution du code en exécutant un script PowerShell. L'application OpManager s'exécutait dans le contexte d'un compte d'administrateur de domaine, ce qui a entraîné une compromission complète du réseau interne.

Dans le rapport ci-dessous, je serais immédiatement ravi de voir Tomcat sur n'importe quelle évaluation (mais surtout lors d'un test de pénétration externe) et je ne serais pasUtilisez les informations d'identification par défaut sur les points de terminaison `/manager` et `/host-manager` . Si nous pouvons accéder à l'un ou l'autre, nous pouvons télécharger un fichier WAR malveillant et exécuter du code à distance sur l'hôte sous-jacent à l'aide du [code JSP](https://en.wikipedia.org/wiki/Jakarta_Server_Pages). Plus d'informations à ce sujet plus loin dans le module.

![image](https://academy.hackthebox.com/storage/modules/113/eyewitness4.png)

En poursuivant dans le rapport, il semble que le site Web principal `http://inlanefreight.local` vienne ensuite. Les applications Web personnalisées valent toujours la peine d'être testées car elles peuvent contenir une grande variété de vulnérabilités. Ici, je serais également intéressé de voir si le site Web utilisait un CMS populaire tel que WordPress, Joomla ou Drupal. L'application suivante, `http://support-dev.inlanefreight.local`, est intéressante car elle semble exécuter [osTicket](https://osticket.com/), qui a souffert de diverses vulnérabilités graves au fil des ans. . Les systèmes de billetterie d'assistance présentent un intérêt particulier car nous pouvons être en mesure de nous connecter et d'accéder à des informations sensibles. Si l'ingénierie sociale est dans le champ d'application, nous pouvons être en mesure d'interagir avec le personnel d'assistance à la clientèle ou même de manipuler le système pour enregistrer une adresse e-mail valide pour le domaine de l'entreprise que nous pouvons exploiter pour accéder à d'autres services.

Cette dernière pièce a été démontrée dans la boîte de publication hebdomadaire HTB [Delivery](https://0xdf.gitlab.io/2021/05/22/htb-delivery.html) par [IppSec](https://www.youtube. com/watch?v=gbs43E71mFM). Cette boîte particulière mérite d'être étudiée car elle montre ce qui est possible en explorant les fonctionnalités intégrées de certaines applications courantes. Nous couvrirons osTicket plus en profondeur plus tard dans ce module.

![image](https://academy.hackthebox.com/storage/modules/113/eyewitness3.png)

Lors d'une évaluation, je continuerais à examiner le rapport, en notant les hôtes intéressants, y compris l'URL et le nom/version de l'application pour plus tard. Il est important à ce stade de se rappeler que nous sommes encore dans la phase de collecte d'informations et que chaque petit détail peut faire ou défaire notre évaluation. Nous ne devrions pas être négligents et commencer à attaquer les hôtes tout de suite, car nous pourrions nous retrouver dans un terrier de lapin et manquer quelque chose de crucial plus tard dans le rapport. Lors d'un test de pénétration externe, je m'attendrais à voir un mélange d'applications personnalisées, certains CMS, peut-être des applications telles que Tomcat, Jenkins et Splunk, des portails d'accès à distance tels que Remote Desktop Services (RDS), des points de terminaison VPN SSL, Outlook Web Access (OWA), O365, peut-être une sorte de page de connexion de périphérique réseau périphérique, etc.

Votre kilométrage peut varier, et parfois nous rencontrerons des applications qui ne devraient absolument pas être exposées, comme une seule page avec un bouton de téléchargement de fichier que j'ai rencontré une fois avec un message indiquant : "Veuillez télécharger uniquement les fichiers .zip et .tar.gz ". Bien sûr, je n'ai pas tenu compte de cet avertissement (car cela était dans le champ d'application lors d'un test d'intrusion sanctionné par le client) et j'ai procédé au téléchargement d'un fichier de test ".aspx". À ma grande surprise, il n'y avait aucune sorte de validation côté client ou back-end, et le fichier semblait être téléchargé. En faisant un forçage rapide des répertoires, j'ai pu localiser un répertoire `/files` dans lequel la liste des répertoires était activée, et mon fichier `test.aspx` s'y trouvait. À partir de là, j'ai procédé au téléchargement d'un shell Web ".aspx" et j'ai pris pied dans l'environnement interne. Cet exemple montre que nous ne devons négliger aucun effort et qu'il peut y avoir un trésor absolu de données pour nous dans nos données de découverte d'applications.

Au cours d'un test de pénétration interne, nous verrons une grande partie de la même chose, mais souvent aussi de nombreuses pages de connexion d'imprimante (que nous pouvons parfois exploiter pour obtenir des informations d'identification LDAP en texte clair), des portails de connexion ESXi et vCenter, des pages de connexion iLO et iDRAC, une pléthore de réseaux appareils, appareils IoT, téléphones IP, référentiels de code interne, portails SharePoint et intranet personnalisés, appliances de sécurité, et bien plus encore.

* * * * *

Passer à autre chose
---------

Maintenant que nous avons travaillé sur notre méthodologie de découverte d'applications et mis en place notre structure de prise de notes, explorons en profondeur certaines des applications les plus courantes que nous rencontrerons à maintes reprises. Veuillez noter que ce module ne peut pas couvrir toutes les applications auxquelles nous serons confrontés. Nous visons plutôt à couvrir les plus courantes et à en apprendre davantage sur les vulnérabilités courantes, les erreurs de configuration et l'abus de leurs fonctionnalités intégrées.

Je peux vous garantir que vous serez confronté à au moins quelques-unes, voire toutes, de ces applications au cours de votre carrière en tant que testeur d'intrusion. La méthodologie et l'état d'esprit d'exploration de ces applications sont encore plus importants, que nous développerons et améliorerons tout au long de ce module et testerons lors des évaluations de compétences à la fin. De nombreux testeurs ont de grandes compétences techniques, mais des compétences non techniques telles qu'une méthodologie solide et reproductible ainsi qu'une organisation, une attention aux détails, une communication solide et une prise de notes / documentation et des rapports approfondis peuvent nous distinguer et aider à renforcer la confiance dans nos compétences des deux nos employeurs comme nos clients.