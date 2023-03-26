Découverte XSS
=============

* * * * *

À présent, nous devrions avoir une bonne compréhension de ce qu'est une vulnérabilité XSS, des trois types de XSS et de la façon dont chaque type diffère des autres. Nous devons également comprendre comment XSS fonctionne en injectant du code JavaScript dans la source de la page côté client, exécutant ainsi du code supplémentaire, que nous apprendrons plus tard à utiliser à notre avantage.

Dans cette section, nous allons passer en revue différentes manières de détecter les vulnérabilités XSS au sein d'une application Web. Dans les vulnérabilités des applications Web (et toutes les vulnérabilités en général), les détecter peut devenir aussi difficile que de les exploiter. Cependant, comme les vulnérabilités XSS sont répandues, de nombreux outils peuvent nous aider à les détecter et à les identifier.

* * * * *

Découverte automatisée
-------------------

Presque tous les scanners de vulnérabilité des applications Web (comme [Nessus](https://www.tenable.com/products/nessus), [Burp Pro](https://portswigger.net/burp/pro) ou [ZAP]( https://owasp.org/www-project-zap/)) ont diverses capacités pour détecter les trois types de vulnérabilités XSS. Ces scanners effectuent généralement deux types d'analyse : une analyse passive, qui examine le code côté client à la recherche de vulnérabilités potentielles basées sur DOM, et une analyse active, qui envoie différents types de charges utiles pour tenter de déclencher un XSS via l'injection de charges utiles dans la source de la page. .

Alors que les outils payants ont généralement un niveau de précision plus élevé dans la détection des vulnérabilités XSS (en particulier lorsque des contournements de sécurité sont nécessaires), nous pouvons toujours trouver des outils open source qui peuvent nous aider à identifier les vulnérabilités XSS potentielles. Ces outils fonctionnent généralement en identifiant les champs de saisie dans les pages Web, en envoyant divers types de charges utiles XSS, puis en comparant la source de la page rendue pour voir si la même charge utile peut s'y trouver, ce qui peut indiquer une injection XSS réussie. Pourtant, cela ne sera pas toujours précis, car parfois, même si la même charge utile a été injectée, cela peut ne pas conduire à une exécution réussie pour diverses raisons, nous devons donc toujours vérifier manuellement l'injection XSS.

Certains des outils open source courants qui peuvent nous aider dans la découverte XSS sont [XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS ), et [XSSer](https://github.com/epsylon/xsser). Nous pouvons essayer `XSS Strike` en le clonant sur notre VM avec `git clone` :

```
dsgsec@htb[/htb]$ git clone https://github.com/s0md3v/XSStrike.git
dsgsec@htb[/htb]$ cd XSStrike
dsgsec@htb[/htb]$ pip install -r requirements.txt
dsgsec@htb[/htb]$ python xsstrike.py

XSStrike v3.1.4
...COUPER...

```

Nous pouvons ensuite exécuter le script et lui fournir une URL avec un paramètre en utilisant `-u`. Essayons de l'utiliser avec notre exemple `Reflected XSS` de la section précédente :

```
dsgsec@htb[/htb]$ python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"

         XSStrike v3.1.4

[~] Vérification des vulnérabilités DOM
[+] Statut WAF : Hors ligne
[!] Paramètre de test : tâche
[!] Réflexions trouvées : 1
[~] Analyse des réflexions
[~] Génération de charges utiles
[!] Charges utiles générées : 3072
-------------------------------------------------- ----------
[+] Charge utile : <HtMl%09onPoIntERENTER+=+confirm()>
[!] Efficacité : 100
[!] Confiance : 10
[?] Souhaitez-vous continuer à scanner ? [o/N]

```

Comme nous pouvons le voir, l'outil a identifié le paramètre comme vulnérable à XSS dès la première charge utile. `Essayez de vérifier la charge utile ci-dessus en la testant sur l'un des exercices précédents. Vous pouvez également essayer de tester les autres outils et les exécuter sur les mêmes exercices pour voir dans quelle mesure ils sont capables de détecter les vulnérabilités XSS.

* * * * *

Découverte manuelle
----------------

En ce qui concerne la découverte XSS manuelle, la difficulté de trouver la vulnérabilité XSS dépend du niveau de sécurité de l'application Web. Les vulnérabilités XSS de base peuvent généralement être trouvées en testant diverses charges utiles XSS, mais l'identification des vulnérabilités XSS avancées nécessite des compétences avancées en révision de code.

* * * * *

#### Charges utiles XSS

La méthode la plus élémentaire de recherche de vulnérabilités XSS consiste à tester manuellement diverses charges utiles XSS par rapport à un champ de saisie dans une page Web donnée. Nous pouvons trouver d'énormes listes de charges utiles XSS en ligne, comme celle sur [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) ou celle dans [PayloadBox]( https://github.com/payloadbox/xss-payload-list). Nous pouvons ensuite commencer à tester ces charges utiles une par une en les copiant chacune et en l'ajoutant dans notre formulaire, et en voyant si une boîte d'alerte apparaît.

Remarque : XSS peut être injecté dans n'importe quelle entrée de la page HTML, qui n'est pas exclusive aux champs d'entrée HTML, mais peut également se trouver dans les en-têtes HTTP comme le cookie ou l'agent utilisateur (c'est-à-dire lorsque leurs valeurs sont affichées sur la page).

Vous remarquerez que la majorité des charges utiles ci-dessus ne fonctionnent pas avec nos exemples d'applications Web, même si nous traitons avec le type de vulnérabilités XSS le plus élémentaire. En effet, ces charges utiles sont écrites pour une grande variété de points d'injection (comme l'injection après un seul guillemet) ou sont conçues pour eignorer certaines mesures de sécurité (comme les filtres de désinfection). De plus, ces charges utiles utilisent une variété de vecteurs d'injection pour exécuter du code JavaScript, comme les balises `<script>` de base, d'autres `attributs HTML` comme `<img>`, ou même des attributs `Style CSS` . C'est pourquoi nous pouvons nous attendre à ce que bon nombre de ces charges utiles ne fonctionnent pas dans tous les cas de test, car elles sont conçues pour fonctionner avec certains types d'injections.

C'est pourquoi il n'est pas très efficace de recourir à un copier/coller manuel des payloads XSS, car même si une application web est vulnérable, cela peut nous prendre un certain temps pour identifier la vulnérabilité, surtout si nous avons de nombreux champs de saisie à tester. C'est pourquoi il peut être plus efficace d'écrire notre propre script Python pour automatiser l'envoi de ces charges utiles, puis de comparer la source de la page pour voir comment nos charges utiles ont été rendues. Cela peut nous aider dans les cas avancés où les outils XSS ne peuvent pas facilement envoyer et comparer les charges utiles. De cette façon, nous aurions l'avantage de personnaliser notre outil à notre application Web cible. Cependant, il s'agit d'une approche avancée de la découverte XSS, et elle ne fait pas partie de la portée de ce module.

* * * * *

Révision des codes
-----------

La méthode la plus fiable pour détecter les vulnérabilités XSS est la révision manuelle du code, qui doit couvrir à la fois le code back-end et le code front-end. Si nous comprenons précisément comment notre entrée est traitée jusqu'à ce qu'elle atteigne le navigateur Web, nous pouvons écrire une charge utile personnalisée qui devrait fonctionner avec une grande confiance.

Dans la section précédente, nous avons examiné un exemple de base de révision de code HTML lors de l'examen de `Source` et `Sink` pour les vulnérabilités XSS basées sur DOM. Cela nous a donné un aperçu rapide du fonctionnement de la revue de code frontale pour identifier les vulnérabilités XSS, bien que sur un exemple frontal très basique.

Il est peu probable que nous trouvions des vulnérabilités XSS via des listes de charge utile ou des outils XSS pour les applications Web les plus courantes. En effet, les développeurs de ces applications Web exécutent probablement leur application via des outils d'évaluation des vulnérabilités, puis corrigent toutes les vulnérabilités identifiées avant leur publication. Dans de tels cas, l'examen manuel du code peut révéler des vulnérabilités XSS non détectées, qui peuvent survivre aux versions publiques d'applications Web courantes. Ce sont également des techniques avancées qui sortent du cadre de ce module. Néanmoins, si vous souhaitez les apprendre, le [Secure Coding 101 : JavaScript](https://academy.hackthebox.com/course/preview/secure-coding-101-javascript) et le [Whitebox Pentesting 101 : Command Injection ](https://academy.hackthebox.com/course/preview/whitebox-pentesting-101-command-injection) les modules couvrent ce sujet en détail.