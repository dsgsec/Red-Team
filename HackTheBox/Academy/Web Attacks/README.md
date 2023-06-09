Introduction aux attaques Web
===========================

* * * * *

Comme les applications Web deviennent très courantes et utilisées par la plupart des entreprises, l'importance de les protéger contre les attaques malveillantes devient également plus critique. Au fur et à mesure que les applications Web modernes deviennent plus complexes et avancées, les types d'attaques utilisées contre elles augmentent également. Cela conduit à une vaste surface d'attaque pour la plupart des entreprises aujourd'hui, c'est pourquoi les attaques Web sont les types d'attaques les plus courants contre les entreprises. La protection des applications Web devient l'une des principales priorités de tout service informatique.

L'attaque d'applications Web externes peut entraîner la compromission du réseau interne de l'entreprise, ce qui peut éventuellement conduire à des actifs volés ou à des services interrompus. Cela peut potentiellement causer un désastre financier pour l'entreprise. Même si une entreprise n'a pas d'applications Web externes, elle utilise probablement des applications Web internes ou des points de terminaison d'API externes, qui sont tous deux vulnérables aux mêmes types d'attaques et peuvent être exploités pour atteindre les mêmes objectifs.

Alors que d'autres modules de la HTB Academy couvraient divers sujets sur les applications Web et divers types de techniques d'exploitation Web, dans ce module, nous aborderons trois autres attaques Web qui peuvent être trouvées dans n'importe quelle application Web, ce qui peut conduire à un compromis. Nous verrons comment détecter, exploiter et prévenir chacune de ces trois attaques.

* * * * *

Attaques Web
-----------

#### Altération du verbe HTTP

La première attaque Web abordée dans ce module est [HTTP Verb Tampering](https://owasp.org/www-project-web-security-testing-guide/v41/4-Web_Application_Security_Testing/07-Input_Validation_Testing/03-Testing_for_HTTP_Verb_Tampering). Une attaque HTTP Verb Tampering exploite les serveurs Web qui acceptent de nombreux verbes et méthodes HTTP. Cela peut être exploité en envoyant des requêtes malveillantes à l'aide de méthodes inattendues, ce qui peut conduire à contourner le mécanisme d'autorisation de l'application Web ou même à contourner ses contrôles de sécurité contre d'autres attaques Web. Les attaques HTTP Verb Tampering sont l'une des nombreuses autres attaques HTTP qui peuvent être utilisées pour exploiter les configurations de serveur Web en envoyant des requêtes HTTP malveillantes.

#### Références directes d'objets non sécurisées (IDOR)

La deuxième attaque abordée dans ce module est [Références d'objets directes non sécurisées (IDOR)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/05-Authorization_Testing/04- Testing_for_Insecure_Direct_Object_References). IDOR fait partie des vulnérabilités Web les plus courantes et peut conduire à l'accès à des données qui ne devraient pas être accessibles aux attaquants. Ce qui rend cette attaque très courante est essentiellement l'absence d'un système de contrôle d'accès solide sur le back-end. Comme les applications Web stockent les fichiers et les informations des utilisateurs, elles peuvent utiliser des numéros séquentiels ou des identifiants d'utilisateur pour identifier chaque élément. Supposons que l'application Web ne dispose pas d'un mécanisme de contrôle d'accès robuste et expose des références directes aux fichiers et aux ressources. Dans ce cas, nous pouvons accéder aux fichiers et aux informations d'autres utilisateurs en devinant ou en calculant simplement leurs identifiants de fichiers.

#### Injection d'entité externe XML (XXE)

La troisième et dernière attaque Web dont nous parlerons est l'[injection d'entité externe XML (XXE)](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing). De nombreuses applications Web traitent des données XML dans le cadre de leurs fonctionnalités. Supposons qu'une application Web utilise des bibliothèques XML obsolètes pour analyser et traiter les données d'entrée XML de l'utilisateur frontal. Dans ce cas, il peut être possible d'envoyer des données XML malveillantes pour divulguer des fichiers locaux stockés sur le serveur principal. Ces fichiers peuvent être des fichiers de configuration pouvant contenir des informations sensibles telles que des mots de passe ou même le code source de l'application Web, ce qui nous permettrait d'effectuer un test d'intrusion Whitebox sur l'application Web pour identifier davantage de vulnérabilités. Les attaques XXE peuvent même être exploitées pour voler les informations d'identification du serveur d'hébergement, ce qui compromettrait l'ensemble du serveur et permettrait l'exécution de code à distance.

Commençons par discuter de la première de ces attaques dans la section suivante.
