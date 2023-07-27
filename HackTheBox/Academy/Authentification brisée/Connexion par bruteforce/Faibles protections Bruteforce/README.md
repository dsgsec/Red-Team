Faibles protections Bruteforce
==============================

* * * * *

Avant de creuser dans les attaques, nous devons comprendre les protections possibles que nous pourrions rencontrer au cours de notre processus de test. De nos jours, il existe de nombreux mécanismes de sécurité différents conçus pour empêcher les attaques automatisées. Parmi les plus courantes, citons les suivantes.

-   `CAPTCHA`
-   `Rate Limits`

De plus, les développeurs Web créent souvent leurs propres mécanismes de sécurité qui rendent le processus de test plus « intéressant » pour nous, car ces mécanismes de sécurité personnalisés peuvent contenir des bogues que nous pouvons exploiter. Familiarisons-nous d'abord avec les mécanismes de sécurité courants contre les attaques automatisées pour comprendre leur fonction et préparer nos attaques contre elles.

* * * * *

CAPTCHA
-------

[CAPTCHA](https://en.wikipedia.org/wiki/CAPTCHA) , une mesure de sécurité largement utilisée nommée d'après la phrase "Completely Automated Public Turing test to tell Computers and Humans Apart", peut avoir de nombreuses formes différentes. Cela peut nécessiter, par exemple, de taper un mot présenté sur une image, d'entendre un court extrait audio et saisir ce que vous avez entendu dans un formulaire, faire correspondre une image à un motif donné ou effectuer des opérations mathématiques de base.

![](https://academy.hackthebox.com/storage/modules/80/07-captcha_math-small.png)

Même si CAPTCHA a été contourné avec succès dans le passé, il est toujours assez efficace contre les attaques automatisées. Une application devrait au moins demander à un utilisateur de résoudre un CAPTCHA après quelques tentatives infructueuses. Certains développeurs ignorent souvent complètement cette protection, et d'autres préfèrent présenter un CAPTCHA après quelques échecs de connexion pour conserver une bonne expérience utilisateur.

Il est également possible pour les développeurs d'utiliser une implémentation personnalisée ou faible de CAPTCHA, où, par exemple, le nom de l'image est composé des caractères contenus dans l'image. Avoir des protections faibles est souvent pire que de ne pas avoir de protection car cela donne un faux sentiment de sécurité. L'image ci-dessous montre une implémentation faible où le code PHP place le contenu de l'image dans le `id`champ. Ce type d'implémentation faible est rare mais pas improbable.

![](https://academy.hackthebox.com/storage/modules/80/06-captcha_id.png)

En tant qu'attaquant, nous pouvons simplement lire le code source de la page pour trouver la valeur du code CAPTCHA et contourner la protection. Nous devrions toujours lire la source.

En tant que développeurs, nous ne devons pas développer notre propre CAPTCHA, mais nous fier à un CAPTCHA bien testé et l'exiger après très peu d'échecs de connexion.

* * * * *

Limitation de débit
-------------------

Une autre protection standard est la limitation de débit. Avec un compteur qui s'incrémente après chaque tentative infructueuse, une application peut bloquer un utilisateur après trois tentatives infructueuses dans les 60 secondes et notifier l'utilisateur en conséquence.

![](https://academy.hackthebox.com/storage/modules/80/06-rate_limit.png)

Une attaque par force brute standard ne sera pas efficace lorsque la limitation de débit est en place. Lorsque l'outil utilisé n'a pas connaissance de cette protection, il essaiera des combinaisons de nom d'utilisateur et de mot de passe qui ne sont jamais réellement validées par l'application Web attaquée. Dans un tel cas, la majorité des identifiants tentés apparaîtront comme non valides (faux négatifs). Une solution de contournement simple consiste à apprendre à notre outil à comprendre les messages liés à la limitation du débit et aux tentatives de connexion réussies et infructueuses. Téléchargez [rate_limit_check.py](https://academy.hackthebox.com/storage/modules/80/scripts/rate_limit_check_py.txt) et parcourez le code. Les lignes pertinentes sont 10 et 13, où nous configurons un temps d'attente et un message de verrouillage, et la ligne 41, où nous effectuons la vérification proprement dite.

Après avoir été bloquée, l'application peut également nécessiter une opération manuelle avant de déverrouiller le compte. Par exemple, un code de confirmation envoyé par e-mail ou une tape sur un téléphone portable. La limitation des tarifs n'impose pas toujours une période de réflexion. L'application peut présenter à l'utilisateur des questions auxquelles il doit répondre correctement avant de réaccéder à la fonctionnalité de connexion au moment où la limitation du débit démarre.

La plupart des implémentations standard de limitation de débit que nous voyons aujourd'hui imposent un délai après `N`les tentatives infructueuses. Par exemple, un utilisateur peut essayer de se connecter trois fois, puis il doit attendre 1 minute avant de réessayer. Après trois tentatives infructueuses supplémentaires, ils doivent attendre 2 minutes et ainsi de suite.

D'une part, un utilisateur régulier pourrait être contrarié après qu'un délai soit imposé, mais d'autre part, la limitation du débit est une excellente forme de protection contre les attaques automatisées par force brute. Notez que la limitation du débit peut être rendue plus robuste en augmentant progressivement le délai et en regroupant les demandes par nom d'utilisateur, adresse IP source, agent utilisateur du navigateur et autres caractéristiques.

Nous pensons que chaque application Web a ses propres exigences en termes de convivialité et de sécurité qui doivent être parfaitement équilibrées lors de l'élaboration d'une limite de débit. L'application d'un verrouillage précoce sur une application Web encombrée et non critique entraînera sans aucun doute de nombreuses demandes au service d'assistance. D'un autre côté, utiliser une limite de débit trop tard pourrait être complètement inutile.

Les frameworks matures ont des protections contre la force brute intégrées ou utilisent des plugins/extensions externes dans le même but. En dernier recours, les principaux serveurs Web comme Apache httpd ou Nginx pourraient être utilisés pour limiter le débit sur une page de connexion donnée.

* * * * *

Protections insuffisantes
-------------------------

Lorsqu'un attaquant peut trafiquer des données prises en compte pour augmenter la sécurité, il peut contourner tout ou partie des protections. Par exemple, changer l' `User-Agent`en-tête est facile. Certaines applications Web ou pare-feu d'applications Web exploitent des en-têtes comme `X-Forwarded-For`pour deviner l'adresse IP source réelle. Cela est dû au fait que de nombreux fournisseurs d'accès Internet, opérateurs de téléphonie mobile ou grandes entreprises "cachent" généralement les utilisateurs derrière le NAT. Le blocage d'une adresse IP sans l'aide d'un en-tête comme `X-Forwarded-For`peut entraîner le blocage de tous les utilisateurs derrière le NAT spécifique.

Un exemple vulnérable simple pourrait être :

#### Exemple de script PHP vulnérable

Code : php

```
<?php
// get IP address
if (array_key_exists('HTTP_X_FORWARDED_FOR', $_SERVER)) {
	$realip = array_map('trim', explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']))[0];
} else if (array_key_exists('HTTP_CLIENT_IP', $_SERVER)) {
	$realip = array_map('trim', explode(',', $_SERVER['HTTP_CLIENT_IP']))[0];
} else if (array_key_exists('REMOTE_ADDR', $_SERVER)) {
	$realip = array_map('trim', explode(',', $_SERVER['REMOTE_ADDR']))[0];
}

echo "<div>Your real IP address is: " . htmlspecialchars($realip) . "</div>";
?>

```

[CVE-2020-35590](https://nvd.nist.gov/vuln/detail/CVE-2020-35590) est lié à une vulnérabilité de plugin WordPress similaire à celle présentée dans l'extrait ci-dessus. Les développeurs du plugin ont introduit une amélioration de la sécurité qui bloquerait une tentative de connexion à partir de la même adresse IP. Malheureusement, cette mesure de sécurité pourrait être contournée en créant un `X-Forwarded-For`en-tête.

À partir du script que nous avons fourni dans le chapitre précédent, nous pouvons modifier les en-têtes dans la définition du script [basic_bruteforce.py](https://academy.hackthebox.com/storage/modules/80/scripts/basic_bruteforce_py.txt)`dict` fourni à la ligne 9 comme ceci :

Code : Python

```
headers = {
  "User-Agent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36",
  "X-Forwarded-For": "1.2.3.4"
}

```

Le code PHP vulnérable pensera que la requête provient de l'adresse IP 1.2.3.4. Notez que nous avons utilisé une déclaration multiligne des en-têtes `dict`pour maintenir une bonne lisibilité.

Certaines applications Web peuvent accorder l'accès aux utilisateurs en fonction de leur adresse IP source. Le comportement dont nous venons de parler pourrait être abusé pour contourner ce type de protection.

Du point de vue d'un développeur, toutes les mesures de sécurité doivent être envisagées en tenant compte à la fois de l'expérience utilisateur et de la sécurité de l'entreprise. Une banque peut imposer un verrouillage d'utilisateur qui nécessite l'annulation d'un appel téléphonique. Une banque peut également éviter le CAPTCHA en raison de la nécessité d'un deuxième facteur d'authentification (OTP sur clé USB ou via SMS, par exemple). Cependant, un e-magazine doit examiner attentivement chaque protection de sécurité pour obtenir une bonne expérience utilisateur tout en conservant une posture de sécurité solide.

En aucun cas, une application Web ne doit s'appuyer sur un seul élément inviolable comme protection de sécurité. Il n'existe aucun moyen fiable d'identifier l'adresse IP réelle d'un utilisateur derrière un NAT, et chaque élément d'information utilisé pour distinguer les visiteurs peut être falsifié. Par conséquent, les développeurs doivent implémenter des protections contre les attaques par force brute qui ralentissent un attaquant autant que possible avant de recourir au verrouillage de l'utilisateur. Le ralentissement des choses peut être réalisé grâce à des mécanismes CAPTCHA plus difficiles, tels que CAPTCHA qui change de format à chaque chargement de page, ou CAPTCHA enchaîné avec une question personnelle à laquelle l'utilisateur a déjà répondu. Cela dit, la meilleure solution serait probablement d'utiliser MFA.
