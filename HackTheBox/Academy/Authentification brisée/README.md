Qu'est-ce que l'authentification
================================

* * * * *

L'authentification est définie comme `the act of proving an assertion`. Dans le contexte de ce module, qui s'articule autour de la sécurité des applications, l'authentification peut être définie comme le processus permettant de déterminer si une entité (un utilisateur ou une application automatisée) est celle qu'elle prétend être.

La méthode d'authentification la plus répandue utilisée dans les applications Web est `login forms`, où un utilisateur entre son nom d'utilisateur et son mot de passe pour prouver son identité. Les formulaires de connexion peuvent être trouvés sur des sites Web tels que HTB Academy et Hack the Box vers des fournisseurs de messagerie tels que Gmail, des services bancaires en ligne, des sites de récompenses pour les membres et la grande majorité des sites Web qui offrent certains services. Sur HTB Academy, le formulaire de connexion ressemble à ceci :

![image](https://academy.hackthebox.com/storage/modules/80/login-form.png)

L'authentification est probablement la mesure de sécurité la plus répandue et constitue la première ligne de défense contre les accès non autorisés. Bien qu'il soit communément appelé et abrégé par «`auth`  », cette version courte est trompeuse car elle pourrait être confondue avec un autre concept de sécurité essentiel, `Authorization`.

[L'autorisation](https://en.wikipedia.org/wiki/Authorization) est définie comme `the process of approving or disapproving a request from a given (authenticated) entity`. Ce module ne couvrira pas l'autorisation en profondeur. Comprendre la différence entre les deux concepts de sécurité est essentiel pour aborder ce module avec le bon état d'esprit.

Supposons que nous ayons rencontré un formulaire de connexion lors d'un test de pénétration pour notre client Inlanefreight. De nos jours, la plupart des entreprises proposent certains services pour lesquels leurs clients doivent s'enregistrer et s'authentifier.

Notre objectif en tant qu'évaluateurs tiers est de vérifier si ces formulaires de connexion sont mis en œuvre de manière sécurisée et si nous pouvons les contourner pour obtenir un accès non autorisé. Il existe de nombreuses méthodes et procédures différentes pour tester les formulaires de connexion. Nous aborderons en détail les plus efficaces d'entre eux tout au long de ce module.

Présentation des méthodes d'authentification
============================================

* * * * *

Au cours de la phase d'authentification, l'entité qui souhaite s'authentifier envoie un `identification string`qui peut être un identifiant, un nom d'utilisateur, un e-mail, ainsi que des données supplémentaires. Le type de données le plus courant qu'un processus d'authentification exige d'être envoyé avec la chaîne d'identification est une chaîne de mot de passe. Cela dit, le type de données supplémentaires peut varier d'une implémentation à l'autre.

* * * * *

Authentification multifacteur
-----------------------------

`Multi-Factor Authentication`, communément appelé `MFA`(ou `2FA`lorsqu'il n'y a que deux facteurs impliqués), peut entraîner un processus d'authentification beaucoup plus robuste.

Les facteurs sont séparés en trois domaines différents :

-   quelque chose que l'utilisateur `sait`, par exemple, un nom d'utilisateur ou un mot de passe
-   quelque chose de l'utilisateur `a`, comme un jeton matériel
-   quelque chose de l'utilisateur `est`, généralement une empreinte biométrique

Lorsqu'un processus d'authentification nécessite que l'entité envoie des données appartenant à plusieurs de ces domaines, il doit être considéré comme un processus MFA. L'authentification à facteur unique nécessite généralement quelque chose de l'utilisateur`knows` :

-   `Username`+`Password`

Il est également possible que l'exigence ne concerne que quelque chose de l'utilisateur `a`.

Pensez à un badge d'entreprise ou à un billet de train. Pour passer à travers un tourniquet, vous devez souvent glisser un badge qui vous donne accès. Dans ce cas, vous n'avez besoin d'aucun code PIN, d'aucune chaîne d'identification ou de quoi que ce soit d'autre qu'une carte. Il s'agit d'un cas marginal car les badges d'entreprise, ou les multi-cartes de train, sont souvent utilisés pour correspondre à un utilisateur spécifique en envoyant un identifiant. En glissant, une autorisation et une forme d'authentification sont effectuées.

* * * * *

Authentification basée sur un formulaire
----------------------------------------

La méthode d'authentification la plus courante pour les applications Web est `Form-Based Authentication`(FBA). L'application présente un formulaire HTML dans lequel l'utilisateur saisit son nom d'utilisateur et son mot de passe, puis l'accès est accordé après avoir comparé les données reçues à un backend. Après une tentative de connexion réussie, le serveur d'application crée une session liée à une clé unique (généralement stockée dans un cookie). Cette clé unique est transmise entre le client et l'application Web à chaque communication ultérieure pour que la session soit maintenue.

Certaines applications Web exigent que l'utilisateur passe par plusieurs étapes d'authentification. Par exemple, la première étape nécessite de saisir le nom d'utilisateur, la seconde le mot de passe et la troisième un jeton `One-time Password`( `OTP`). Un `OTP`jeton peut provenir d'un périphérique matériel ou d'une application mobile qui génère des mots de passe. Les mots de passe à usage unique ont généralement une durée de vie limitée, par exemple 30 secondes, et sont valables pour une seule tentative de connexion, d'où le nom à usage unique.

Il convient de noter que les procédures de connexion en plusieurs étapes peuvent souffrir de vulnérabilités [de logique métier](https://owasp.org/www-community/vulnerabilities/Business_logic_vulnerability) . Par exemple, l'Étape 3 peut tenir pour acquis que l'Étape 1 et l'Étape 2 ont été réalisées avec succès.

* * * * *

Authentification basée sur HTTP
-------------------------------

De nombreuses applications offrent `HTTP-based`une fonctionnalité de connexion. Dans ces cas, le serveur d'applications peut spécifier différents schémas d'authentification tels que Basic, Digest et NTLM. Tous les schémas d'authentification HTTP s'articulent autour du `401`code d'état et de l' `WWW-Authenticate`en-tête de réponse et sont utilisés par les serveurs d'applications pour contester une demande client et fournir des détails d'authentification (processus Challenge-Response).

Lors de l'utilisation de l'authentification basée sur HTTP, le `Authorization header`contient les données d'authentification et doit être présent dans chaque demande d'authentification de l'utilisateur.

D'un point de vue réseau, les méthodes d'authentification susmentionnées pourraient être moins sécurisées que FBA car chaque requête contient des données d'authentification. Par exemple, pour effectuer une connexion d'authentification de base HTTP, le navigateur encode le nom d'utilisateur et le mot de passe à l'aide de base64. Le `Authorization header`contiendra les informations d'identification encodées en base64 dans chaque demande. Par conséquent, un attaquant capable de capturer le trafic réseau en clair capturera également les informations d'identification. La même chose se produirait si FBA était en place, mais pas pour chaque demande.

Vous trouverez ci-dessous un exemple d'en-tête qu'un navigateur envoie pour effectuer l'authentification de base.

#### En-tête d'authentification HTTP

  En-tête d'authentification HTTP

```
GET /basic_auth.php HTTP/1.1
Host: brokenauth.hackthebox.eu
Cache-Control: max-age=0
Authorization: Basic YWRtaW46czNjdXIzcDQ1NQ==

```

L'en-tête d'autorisation spécifie la méthode d'authentification HTTP, Basic dans cet exemple, et le jeton : si nous décodons la chaîne :

  En-tête d'authentification HTTP

```
YWRtaW46czNjdXIzcDQ1NQ==

```

en tant que chaîne base64, nous verrons que le navigateur s'est authentifié avec les informations d'identification :`admin:s3cur3p455`

L'authentification Digest et NTLM sont plus robustes car les données transmises sont hachées et peuvent contenir un nonce, mais il est toujours possible de craquer ou de réutiliser un jeton capturé.

* * * * *

Autres formes d'authentification
--------------------------------

Bien que cela soit rare, il est également possible que l'authentification soit effectuée en vérifiant l'adresse IP source. Une demande de localhost ou de l'adresse IP d'un serveur bien connu/de confiance pourrait être considérée comme légitime et autorisée car les développeurs supposaient que personne d'autre que l'entité prévue n'utiliserait cette adresse IP.

Les applications modernes pourraient utiliser des tiers pour authentifier les utilisateurs, tels que [SAML](https://en.wikipedia.org/wiki/Security_Assertion_Markup_Language) . De plus, `APIs`ils nécessitent généralement un formulaire d'authentification spécifique, souvent basé sur une approche en plusieurs étapes.

Les attaques contre l'authentification et l'autorisation des API, l'authentification unique et OAuth partagent les mêmes fondements que les attaques contre les applications Web classiques. Néanmoins, ces sujets sont assez vastes et méritent leur propre module.

* * * * *

Exemple de connexion
--------------------

Un scénario typique d'authentification de banque à domicile commence lorsqu'une application Web de banque en ligne demande notre identifiant, qui peut être un numéro à sept chiffres généré par l'application Web de banque en ligne elle-même ou un nom d'utilisateur choisi par l'utilisateur. Puis, sur une deuxième page, l'application demande un mot de passe pour l'identifiant donné. Sur une troisième page, l'utilisateur doit fournir un OTP généré par un jeton matériel ou reçu par SMS sur son téléphone mobile. Après avoir fourni les détails d'authentification à partir des deux facteurs ci-dessus (cas 2FA), l'application Web de banque en ligne vérifie si l'identifiant, le mot de passe et l'OTP sont valides.

Présentation des attaques contre l'authentification
===================================================

* * * * *

Les attaques d'authentification peuvent avoir lieu contre un total de trois domaines. Ces trois domaines sont répartis dans les catégories suivantes :

-   Le `HAS`domaine
-   Le `IS`domaine
-   Le `KNOWS`domaine

* * * * *

Attaquer le domaine HAS
-----------------------

En parlant des trois domaines décrits tout en couvrant l'authentification multifacteur, le `has`domaine semble assez simple car nous possédons ou non un jeton matériel. Les choses sont cependant plus compliquées qu'elles n'y paraissent :

-   Un badge pourrait être `cloned`sans le prendre en charge
-   Un algorithme cryptographique utilisé pour générer des mots de passe à usage unique pourrait être`broken`
-   Tout appareil physique peut être`stolen`

Une antenne longue portée peut facilement atteindre une distance de travail de 50cm et cloner un badge NFC classique. Vous pouvez penser que l'attaquant devrait être extrêmement proche de la victime pour exécuter une telle attaque avec succès. Considérez à quel point nous sommes tous assis les uns par rapport aux autres lorsque nous utilisons les transports en commun ou que nous attendons dans une file d'attente dans un magasin, et vous changerez probablement d'avis. Plusieurs personnes sont à portée de main pour effectuer une telle attaque de clonage chaque jour.

Imaginez que vous déjeunez rapidement dans un bar près du bureau. Vous ne remarquez même pas un agresseur qui passe devant votre siège parce que vous êtes préoccupé par une tâche urgente. Ils viennent de cloner le badge que vous gardez dans votre poche !!! Quelques minutes plus tard, ils transfèrent les informations de votre badge dans un jeton propre et l'utilisent pour entrer dans le bâtiment de votre entreprise tout en déjeunant.

Il est clair que le clonage d'un badge d'entreprise n'est pas si difficile, et les conséquences pourraient être graves.

* * * * *

Attaquer le domaine IS
----------------------

Vous pouvez penser que le `is`domaine est le plus difficile à attaquer. Si une personne s'appuie sur "quelque chose" pour prouver son identité et que ce "quelque chose" est compromis, elle perd la façon unique de prouver son identité puisqu'il n'y a aucun moyen de changer sa façon d'être. Le scan de la rétine, les lecteurs d'empreintes digitales, la reconnaissance faciale se sont tous avérés cassables. Tous peuvent être brisés par une fuite de tiers, une image haute définition, une écumoire ou même une femme de ménage maléfique qui vole le bon verre.

Les entreprises qui vendent des mesures de sécurité basées sur le `is`domaine déclarent qu'elles sont incroyablement sécurisées. En août 2019, une entreprise qui construit des serrures intelligentes biométriques gérées via une application mobile ou web a été [piratée](https://www.vpnmentor.com/blog/report-biostar2-leak/) . L'entreprise a utilisé les empreintes digitales ou la reconnaissance faciale pour identifier les utilisateurs autorisés. La violation a révélé toutes les empreintes digitales et tous les modèles faciaux, y compris les noms d'utilisateur et les mots de passe, les autorisations et les adresses des utilisateurs enregistrés. Alors que les utilisateurs peuvent facilement changer leur mot de passe et atténuer le problème, toute personne capable de reproduire des empreintes digitales ou des motifs faciaux pourra toujours déverrouiller et gérer ces serrures intelligentes.

* * * * *

Attaquer le domaine SAIT
------------------------

Le `knows`domaine est celui que nous allons creuser dans ce module. C'est le plus simple à comprendre, mais nous devons nous plonger dans tous les aspects car c'est aussi le plus répandu. Ce domaine fait référence à des choses qu'un utilisateur connaît, comme un `username`ou un `password`. Dans ce module, nous travaillerons `FBA`uniquement contre. Gardez à l'esprit que la même approche pourrait être adaptée aux implémentations d'authentification HTTP.

[Précédent](https://academy.hackthebox.com/module/80/section/769)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/80/section/772)

Aide-mémoireRessources
