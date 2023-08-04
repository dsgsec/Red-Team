Introduction aux attaques côté serveur
======================================

* * * * *

Les attaques côté serveur ciblent l'application ou le service fourni par un serveur, tandis que le but d'une attaque côté client est d'attaquer le client. Comprendre et identifier les différences est essentiel pour les tests d'intrusion et la chasse aux primes de bogues.

Un excellent exemple de ceux-ci qui devraient aider à clarifier les différences entre les attaques côté serveur et les attaques côté client sont `Cross-Site Request Forgeries (CSRF)`et `Server-side Request Forgeries (SSRF)`. Ces deux attaques impliquent un serveur Web et la façon dont les serveurs traitent les URL. Cependant, CSRF et SSRF ont des objectifs et des objectifs différents.

Citation grossière de la section Cross-Site Request Forgery du module [Introduction aux applications Web](https://academy.hackthebox.com/course/preview/introduction-to-web-applications) :

Les attaques CSRF peuvent utiliser d'autres attaques côté client, telles que les vulnérabilités XSS, pour effectuer des requêtes vers une application Web auprès de laquelle une victime a déjà été authentifiée. Cela permet à l'attaquant d'effectuer des actions en tant qu'utilisateur autorisé, comme changer son mot de passe pour quelque chose que l'attaquant connaîtrait ou effectuer toute action injustifiée en tant que victime.

De la situation ci-dessus, nous devrions être en mesure de déduire que la cible est le client. Les attaques côté serveur ciblent l'application réelle, l'objectif étant de divulguer des données sensibles ou d'injecter des entrées injustifiées dans l'application et même de réaliser l'exécution de code à distance (RCE). Les cibles dans cette situation sont les services back-end.

* * * * *

Types d'attaques côté serveur
-----------------------------

Ce module couvrira différents types d'attaques côté serveur et comment les exploiter. Ceux-ci sont:

-   `Abusing Intermediary Applications`: Accéder à des applications internes non accessibles depuis notre réseau en exploitant des protocoles binaires exposés spécifiques.
-   `Server-Side Request Forgery (SSRF)`: faire en sorte que le serveur d'applications d'hébergement envoie des requêtes à des domaines externes arbitraires ou à des ressources internes dans le but d'identifier des données sensibles.
-   `Server-Side Includes Injection (SSI)`: injection d'une charge utile afin que les directives d'inclusion côté serveur malintentionnées soient analysées pour permettre l'exécution de code à distance ou la fuite de données sensibles. Cette vulnérabilité se produit lorsqu'une entrée utilisateur mal validée parvient à faire partie d'une réponse qui est analysée pour les directives Server-Side Include.
-   `Edge-Side Includes Injection (ESI)`: ESI est un langage de balisage basé sur XML utilisé pour résoudre les problèmes de performances en stockant temporairement le contenu Web dynamique que les protocoles de mise en cache Web habituels ne sauvegardent pas. Edge-Side Include Injection se produit lorsqu'un attaquant parvient à refléter des balises ESI mal intentionnées dans la réponse HTTP. La cause principale de cette vulnérabilité est que les substituts HTTP ne peuvent pas valider l'origine de la balise ESI. Ils se feront un plaisir d'analyser et d'évaluer les balises ESI légitimes par le serveur en amont et les balises ESI malveillantes fournies par un attaquant.
-   `Server-Side Template Injection (SSTI)`: Les moteurs de modèles facilitent la présentation dynamique des données via des pages Web ou des e-mails. L'injection de modèle côté serveur consiste essentiellement à injecter des directives de modèle malintentionnées (charge utile) dans un modèle, en tirant parti des moteurs de modèle qui mélangent de manière non sécurisée l'entrée de l'utilisateur avec un modèle donné.
-   `Extensible Stylesheet Language Transformations Server-Side Injection (XSLT)`: XSLT est un langage basé sur XML généralement utilisé lors de la transformation de documents XML en HTML, un autre document XML ou PDF. Transformations extensibles du langage de feuille de style L'injection côté serveur peut se produire lorsqu'un téléchargement de fichier XSLT arbitraire est possible ou lorsqu'une application génère dynamiquement le document XML de la transformation XSL à l'aide d'une entrée non validée de l'utilisateur.
