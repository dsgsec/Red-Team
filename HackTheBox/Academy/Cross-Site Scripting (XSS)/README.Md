Introduction
============

À mesure que les applications Web deviennent plus avancées et plus courantes, les vulnérabilités des applications Web augmentent également. Parmi les types de vulnérabilités d'application Web les plus courants figurent les [Cross-Site Scripting (XSS)](https://owasp.org/www-community/attacks/xss/) vulnérabilités. Les vulnérabilités XSS tirent parti d'une faille dans la désinfection des entrées utilisateur pour "écrire" du code JavaScript sur la page et l'exécuter côté client, ce qui entraîne plusieurs types d'attaques.

* * * * *

Qu'est-ce que XSS
-----------

Une application Web typique fonctionne en recevant le code HTML du serveur principal et en le restituant sur le navigateur Internet côté client. Lorsqu'une application Web vulnérable ne nettoie pas correctement l'entrée de l'utilisateur, un utilisateur malveillant peut injecter du code JavaScript supplémentaire dans un champ de saisie (par exemple, commentaire/réponse), de sorte qu'une fois qu'un autre utilisateur affiche la même page, il exécute sans le savoir le code JavaScript malveillant.

Les vulnérabilités XSS sont uniquement exécutées côté client et n'affectent donc pas directement le serveur principal. Ils ne peuvent affecter que l'utilisateur exécutant la vulnérabilité. L'impact direct des vulnérabilités XSS sur le serveur back-end peut être relativement faible, mais on les trouve très couramment dans les applications Web, cela équivaut donc à un risque moyen ("impact faible + probabilité élevée = risque moyen"), ce que nous devrions essayez toujours de `réduire` les risques en détectant, corrigeant et prévenant de manière proactive ces types de vulnérabilités.

![risque xss](https://academy.hackthebox.com/storage/modules/103/xss_risk_chart_1.jpg)

* * * * *

Attaques XSS
-----------

Les vulnérabilités XSS peuvent faciliter un large éventail d'attaques, qui peuvent être tout ce qui peut être exécuté via le code JavaScript du navigateur. Un exemple de base d'une attaque XSS est que l'utilisateur cible envoie involontairement son cookie de session au serveur Web de l'attaquant. Un autre exemple est que le navigateur de la cible exécute des appels d'API qui conduisent à une action malveillante, comme changer le mot de passe de l'utilisateur en un mot de passe choisi par l'attaquant. Il existe de nombreux autres types d'attaques XSS, de l'extraction de Bitcoin à l'affichage de publicités.

Comme les attaques XSS exécutent du code JavaScript dans le navigateur, elles sont limitées au moteur JS du navigateur (c'est-à-dire V8 dans Chrome). Ils ne peuvent pas exécuter de code JavaScript à l'échelle du système pour faire quelque chose comme l'exécution de code au niveau du système. Dans les navigateurs modernes, ils sont également limités au même domaine du site Web vulnérable. Néanmoins, être capable d'exécuter JavaScript dans le navigateur d'un utilisateur peut toujours conduire à une grande variété d'attaques, comme mentionné ci-dessus. En plus de cela, si un chercheur qualifié identifie une vulnérabilité binaire dans un navigateur Web (par exemple, un débordement de tas dans Chrome), il peut utiliser une vulnérabilité XSS pour exécuter un exploit JavaScript sur le navigateur de la cible, qui finit par sortir du navigateur. sandbox et exécute le code sur la machine de l'utilisateur.

Les vulnérabilités XSS peuvent être trouvées dans presque toutes les applications Web modernes et ont été activement exploitées au cours des deux dernières décennies. Un exemple XSS bien connu est le [Samy Worm](https://en.wikipedia.org/wiki/Samy_(computer_worm)), qui était un ver basé sur un navigateur qui exploitait une vulnérabilité XSS stockée dans le site Web de réseau social MySpace. en 2005. Il s'exécutait lors de la consultation d'une page Web infectée en publiant un message sur la page MySpace de la victime qui disait : « Samy est mon héros ». Le message lui-même contenait également la même charge utile JavaScript pour republier le même message lorsqu'il était vu par d'autres. En une seule journée, plus d'un million d'utilisateurs de MySpace ont vu ce message affiché sur leurs pages. Même si cette charge utile spécifique n'a causé aucun dommage réel, la vulnérabilité aurait pu être utilisée à des fins beaucoup plus néfastes, comme voler les informations de carte de crédit des utilisateurs, installer des enregistreurs de frappe sur leurs navigateurs ou même exploiter une vulnérabilité binaire dans les navigateurs Web des utilisateurs ( ce qui était plus courant dans les navigateurs Web à l'époque).

En 2014, un chercheur en sécurité a accidentellement identifié une [vulnérabilité XSS](https://blog.sucuri.net/2014/06/serious-cross-site-scripting-vulnerability-in-tweetdeck-twitter.html) dans le tableau de bord TweetDeck de Twitter. . Cette vulnérabilité a été exploitée pour créer un [tweet auto-retweeté](https://twitter.com/derGeruhn/status/476764918763749376) sur Twitter, ce qui a conduit le tweet à être retweeté plus de 38 000 fois en moins de deux minutes. Finalement, cela a forcé Twitter à [fermer temporairement TweetDeck](https://www.theguardian.com/technology/2014/jun/11/twitter-tweetdeck-xss-flaw-users-vulnerable) pendant qu'ils corrigeaient la vulnérabilité.

À ce jour, même les applications Web les plus importantes présentent des vulnérabilités XSS qui peuvent être exploitées. Même la page du moteur de recherche de Google comportait plusieurs vulnérabilités XSS dans sa barre de recherche, la plus récente datant de [2019](https://www.acunetix.com/blog/web-security-zone/mutation-xss-in-google -search/) lorsqu'une vulnérabilité XSS a été trouvée dans la bibliothèque XML. De plus, le serveur Apache, le serveur Web le plus couramment utilisé sur Internet, a déjà signalé une [vulnérabilité XSS](https://blogs.apache.org/infra/entry/apache_org_04_09_2010) qui était activement exploité pour voler les mots de passe des utilisateurs de certaines entreprises. Tout cela nous indique que les vulnérabilités XSS doivent être prises au sérieux et que de nombreux efforts doivent être déployés pour les détecter et les prévenir.

* * * * *

Types de XSS
------------

Il existe trois principaux types de vulnérabilités XSS :

| Taper | Descriptif |
| --- | --- |
| `XSS stocké (persistant)` | Le type de XSS le plus critique, qui se produit lorsque l'entrée de l'utilisateur est stockée dans la base de données principale, puis affichée lors de la récupération (par exemple, des publications ou des commentaires) |
| `XSS réfléchi (non persistant)` | Se produit lorsque l'entrée de l'utilisateur est affichée sur la page après avoir été traitée par le serveur principal, mais sans être stockée (par exemple, résultat de recherche ou message d'erreur) |
| `XSS basé sur DOM` | Un autre type XSS non persistant qui se produit lorsque l'entrée de l'utilisateur est directement affichée dans le navigateur et est entièrement traitée côté client, sans atteindre le serveur principal (par exemple, via des paramètres HTTP côté client ou des balises d'ancrage) |

Nous couvrirons chacun de ces types dans les sections à venir et effectuerons des exercices pour voir comment chacun d'eux se produit, puis nous verrons également comment chacun d'eux peut être utilisé dans des attaques.

Marquer terminé et suivant

[Suivant ](https://academy.hackthebox.com/module/103/section/967)

Aide-mémoire