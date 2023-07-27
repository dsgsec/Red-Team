Informations d'identification par défaut
========================================

* * * * *

Il est courant de trouver des appareils en `default credentials`raison d'une erreur humaine ou d'une panne ou d'un manque de processus approprié. Selon le rapport [Under the hoodie](https://www.rapid7.com/research/reports/under-the-hoodie-2020/) de Rapid7 pour 2020, Rapid7 a obtenu l'accès en utilisant des informations d'identification par défaut connues ou des comptes devinables pendant 21 % de leurs engagements.

Malheureusement, les informations d'identification par défaut sont également utilisées par les responsables travaillant sur des environnements de systèmes de contrôle industriels (ICS). Ils préfèrent avoir des informations d'identification d'accès bien connues lors de la maintenance plutôt que de compter sur un utilisateur de l'entreprise, qui n'est souvent pas averti en matière de cybersécurité, pour stocker en toute sécurité un ensemble complexe d'informations d'identification. Tous les testeurs d'intrusion ont été témoins de ces informations d'identification stockées sur une note Post-it. Cependant, cela ne justifie pas l'utilisation des informations d'identification par défaut. Une étape obligatoire du renforcement de la sécurité consiste à modifier les informations d'identification par défaut et à utiliser des mots de passe forts à un stade précoce du déploiement de l'application.

Un autre fait bien connu est que certains fournisseurs introduisent des comptes cachés codés en dur dans leurs produits. Un exemple est [CVE-2020-29583](https://nvd.nist.gov/vuln/detail/CVE-2020-29583) sur Zyxel USG. Les chercheurs ont trouvé un compte codé en dur avec des privilèges d'administrateur et un mot de passe inchangeable. Comme vous pouvez le voir sur le site Web [du NIST](https://nvd.nist.gov/vuln/search/results?cwe_id=CWE-798) , Zyxel n'est pas seul. Une recherche rapide dans la liste CVE avec "CWE-798 - utilisation d'informations d'identification codées en dur" comme filtre renvoie plus de 500 résultats.

Il existe un ancien projet, toujours maintenu par CIRT.net et disponible en tant que [base de données Web](https://www.cirt.net/passwords) utilisée pour collecter les informations d'identification par défaut divisées par les fournisseurs. De plus, [SecLists](https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv) a une bonne liste basée sur CIRT.net. Les deux options ci-dessus ont des chevauchements mais aussi des différences. C'est une bonne idée de vérifier les deux listes. De retour à SCADA, en 2016, SCADA StrangeLove a publié une liste de mots de passe connus pour les systèmes industriels, à la fois par défaut et codés en dur, sur leur propre référentiel [GitHub .](https://github.com/scadastrangelove/SCADAPASS/blob/master/scadapass.csv)

Même selon les normes de sécurité actuelles, il est courant de rencontrer des ensembles d'informations d'identification bien connus/médiocres dans les appareils ou applications critiques et non critiques. Des ensembles d'exemples pourraient être :

-   `admin:admin`
-   `admin:password`

C'est toujours une bonne idée d'essayer des ensembles connus ou pauvres `user`à `password`l'aide des listes susmentionnées. A titre d'exemple, après avoir trouvé un appareil Cisco lors d'un test d'intrusion, on peut voir que [passdb](https://www.cirt.net/passwords?criteria=cisco) contient 65 entrées pour les appareils Cisco :

![image](https://academy.hackthebox.com/storage/modules/80/cirt_passdb_cisco.png)

En fonction de l'appareil que nous avons trouvé, par exemple un commutateur, un routeur ou un point d'accès, nous devrions essayer au moins :

-   `empty:Cisco`
-   `cisco:cisco`
-   `Cisco:Cisco`
-   `cisco:router`
-   `tech:router`

Il convient de noter que nous pouvons ne pas trouver d'informations d'identification par défaut ou connues pour chaque appareil ou application dans les listes mentionnées ci-dessus et les bases de données. Dans ce cas, une recherche sur Google pourrait mener à des découvertes très intéressantes. Il est également courant de rencontrer des comptes d'utilisateurs facilement devinables ou faibles dans les applications personnalisées. C'est pourquoi nous devrions toujours essayer des combinaisons telles que : ( `user:user`, `tech:tech`).

Lorsque nous essayons de trouver des informations d'identification par défaut ou faibles, nous préférons utiliser des outils automatisés tels que `ffuf`, `wfuzz`, ou des scripts Python personnalisés, mais nous pouvons également faire la même chose à la main ou en utilisant un proxy tel que Burp/ZAP. Nous vous encourageons à tester toutes les méthodes pour vous familiariser avec les outils automatisés et les scripts.

* * * * *

Exemple pratique
----------------

Pour commencer à vous échauffer, téléchargez ce [script Python](https://academy.hackthebox.com/storage/modules/80/scripts/basic_bruteforce_py.txt) , lisez le code source et essayez de comprendre les commentaires. Si vous souhaitez essayer ce script dans un environnement réel, téléchargez ce [code PHP](https://academy.hackthebox.com/storage/modules/80/scripts/basic_bruteforce_php.txt) de base et placez-le sur un serveur Web prenant en charge PHP. Il sera utile tout en essayant de résoudre la question suivante.

Avant de pouvoir commencer à attaquer une page Web, nous devons d'abord déterminer les paramètres d'URL acceptés par la page. Pour ce faire, nous pouvons utiliser `Burp Suite`et capturer la requête pour voir quels paramètres ont été utilisés. Une autre façon de le faire est d'utiliser les outils de développement intégrés de notre navigateur. Par exemple, nous pouvons ouvrir Firefox dans la Pwnbox, puis afficher les outils réseau avec `[CTRL + SHIFT + E]`.

Une fois cela fait, nous pouvons essayer de nous connecter avec n'importe quelles informations d'identification ( `test`: `test`) pour exécuter le formulaire, après quoi les outils réseau afficheront les requêtes HTTP envoyées. Une fois que nous avons la demande, nous pouvons faire un clic droit sur l'un d'eux et sélectionner `Copy`> `Copy POST data`:

![Outils de développement](https://academy.hackthebox.com/storage/modules/57/bruteforcing_firefox_network_1.jpg)

Cela nous donnerait les paramètres POST suivants :

Code : bash

```
username=test&password=test

```

Une autre option serait d'utiliser `Copy`> `Copy as cURL command`, qui copierait l'intégralité `cURL`de la commande, que nous pouvons utiliser dans le Terminal pour répéter la même requête HTTP :

```
dsgsec@htb[/htb]$ curl 'http://URL:PORT/login.php' -H 'User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:68.0) Gecko/20100101 Firefox/68.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: http://URL:PORT' -H 'DNT: 1' -H 'Connection: keep-alive' -H 'Referer: http://URL:PORT/login.php' -H 'Cookie: PHPSESSID=8iafr4t6c3s2nhkaj63df43v05' -H 'Upgrade-Insecure-Requests: 1' -H 'Sec-GPC: 1' --data-raw 'username=test&password=test'

```

Comme nous pouvons le voir, cette commande contient également les paramètres `--data-raw 'username=test&password=test'`.

Démarrer l'instance

 / 1 spawn restant

En attente de démarrage...

#### Des questions

Répondez aux questions ci-dessous pour compléter cette section et gagner des cubes !

Cible : cliquez ici pour faire apparaître le système cible !

Aide-mémoire

+ 1Inspectez la page de connexion et effectuez une attaque par force brute. Quel est le nom d'utilisateur valide ?

Soumettre

Indice

[Précédent](https://academy.hackthebox.com/module/80/section/771)[Suivant](https://academy.hackthebox.com/module/80/section/837)

Aide-mémoireRessources[Aller aux questions](https://academy.hackthebox.com/module/80/section/772#questionsDiv)

##### Table des matières

###### Authentification brisée

[Qu'est-ce que l'authentification](https://academy.hackthebox.com/module/80/section/768)[Présentation des méthodes d'authentification](https://academy.hackthebox.com/module/80/section/769)[Présentation des attaques contre l'authentification](https://academy.hackthebox.com/module/80/section/771)

###### Connexion Brute Forcing

[  Informations d'identification par défaut](https://academy.hackthebox.com/module/80/section/772)[  Faibles protections Bruteforce](https://academy.hackthebox.com/module/80/section/837)[  Brute Forcing Noms d'utilisateur](https://academy.hackthebox.com/module/80/section/767)[  Brute Forcer les mots de passe](https://academy.hackthebox.com/module/80/section/777)[  Jeton de réinitialisation prévisible](https://academy.hackthebox.com/module/80/section/779)

###### Attaques de mot de passe

[Gestion des identifiants d'authentification](https://academy.hackthebox.com/module/80/section/778)[  Réponses devinables](https://academy.hackthebox.com/module/80/section/780)[  Injection de nom d'utilisateur](https://academy.hackthebox.com/module/80/section/781)

###### Attaques de session

[  Biscuits de forçage brutal](https://academy.hackthebox.com/module/80/section/782)[Gestion non sécurisée des jetons](https://academy.hackthebox.com/module/80/section/784)

###### Évaluation des compétences

[  Évaluation des compétences - Authentification brisée](https://academy.hackthebox.com/module/80/section/848)

##### Mon poste de travail

HORS LIGNE

  Démarrer l'instance

 / 1 spawn restant
