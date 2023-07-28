Jeton de réinitialisation prévisible
====================================

* * * * *

Les jetons de réinitialisation (sous la forme d'un code ou d'un mot de passe temporaire) sont des données secrètes générées principalement par l'application lorsqu'une réinitialisation de mot de passe est demandée. Un utilisateur doit le fournir pour prouver son identité avant de modifier ses informations d'identification. Parfois, les applications vous demandent de choisir une ou plusieurs questions de sécurité et de fournir une réponse au moment de l'inscription. Si vous avez oublié votre mot de passe, vous pouvez le réinitialiser en répondant à nouveau à ces questions. Nous pouvons également considérer ces réponses comme des jetons.

Cette fonction nous permet de réinitialiser le mot de passe réel de l'utilisateur sans connaître le mot de passe. Il existe plusieurs façons de procéder, dont nous parlerons bientôt.

Un flux de réinitialisation de mot de passe peut sembler compliqué car il se compose généralement de plusieurs étapes que nous devons comprendre. Ci-dessous, nous avons créé un flux de base qui récapitule ce qui se passe lorsqu'un utilisateur demande une réinitialisation et reçoit un jeton par e-mail. Certaines étapes peuvent mal tourner et un processus qui semble sûr peut être vulnérable.

![](https://academy.hackthebox.com/storage/modules/80/reset_flow2.png)

* * * * *

Réinitialiser le jeton par e-mail
---------------------------------

Si une application permet à l'utilisateur de réinitialiser son mot de passe à l'aide d'une URL ou d'un mot de passe temporaire envoyé par e-mail, elle doit contenir une fonction de génération de jeton robuste. Les frameworks ont souvent des fonctions dédiées à cet effet. Cependant, les développeurs implémentent souvent leurs propres fonctions qui peuvent introduire des failles logiques et un cryptage faible ou implémenter la sécurité par l'obscurité.

* * * * *

Génération de jeton faible
--------------------------

Certaines applications créent un jeton à l'aide de valeurs connues ou prévisibles, telles que l'heure locale ou le nom d'utilisateur qui a demandé l'action, puis hachent ou codent la valeur. Il s'agit d'une mauvaise pratique de sécurité car un jeton n'a pas besoin de contenir d'informations provenant de l'utilisateur réel pour être validé et doit être une valeur purement aléatoire. Dans le cas d'un encodage réversible, il pourrait suffire de décoder le jeton pour comprendre comment il est construit et en forger un valide.

En tant que testeurs d'intrusion, nous devons être conscients de ces types d'implémentations médiocres. Nous devrions essayer de forcer brutalement tout hachage faible en utilisant des combinaisons connues telles que heure + nom d'utilisateur ou heure + e-mail lorsqu'un jeton de réinitialisation est demandé pour un utilisateur donné. Prenons par exemple ce code PHP. C'est l'équivalent logique de la vulnérabilité signalée comme [CVE-2016-0783](https://www.cvedetails.com/cve/CVE-2016-0783/) sur Apache OpenMeeting :

Code : php

```
<?php
function generate_reset_token($username) {
  $time = intval(microtime(true) * 1000);
  $token = md5($username . $time);
  return $token;
}

```

Il est facile de repérer la vulnérabilité. Un attaquant connaissant un nom d'utilisateur valide peut obtenir l'heure du serveur en lisant le `Date header`(qui est presque toujours présent dans la réponse HTTP). L'attaquant peut alors forcer brutalement la `$time`valeur en quelques secondes et obtenir un jeton de réinitialisation valide. Dans cet exemple, nous pouvons voir qu'une requête commune perd la date et l'heure.

![](https://academy.hackthebox.com/storage/modules/80/07-http_header_date.png)

Prenons comme exemple le code PHP téléchargeable [ici](https://academy.hackthebox.com/storage/modules/80/scripts/reset_token_time_php.txt) . L'application génère un jeton en créant un hachage md5 du nombre de secondes écoulées depuis l'époque (à des fins de démonstration, nous utilisons simplement une valeur temporelle). En lisant le code, on peut facilement repérer une vulnérabilité similaire à celle d'OpenMeeting. En utilisant le script [reset_token_time.py](https://academy.hackthebox.com/storage/modules/80/scripts/reset_token_time_py.txt) , nous pourrions gagner une certaine confiance dans la création et le forçage brutal d'un jeton basé sur le temps. Téléchargez les deux scripts et essayez d'obtenir le message de bienvenue.

Veuillez garder à l'esprit que tout en-tête pourrait être supprimé ou modifié en plaçant un proxy inverse devant l'application. Cependant, nous avons souvent la possibilité de déduire le temps de différentes manières. Il s'agit de l'heure d'un message intégré envoyé ou reçu, d'un en-tête d'e-mail ou de la dernière heure de connexion, pour n'en nommer que quelques-uns. Certaines applications ne vérifient pas l'âge du jeton, ce qui donne à un attaquant suffisamment de temps pour une attaque par force brute. Il a également été observé que certaines applications n'invalident ou n'expirent jamais les jetons, même si le jeton a été utilisé. Maintenir un tel composant critique actif est assez risqué car un attaquant pourrait trouver un ancien jeton et l'utiliser.

* * * * *

Jetons courts
-------------

Une autre mauvaise pratique est l'utilisation de jetons courts. Probablement pour aider les utilisateurs mobiles, une application peut générer un jeton d'une longueur de 5/6 caractères numériques qui peut parfois être facilement forcé brutalement. En réalité, il n'est pas nécessaire d'en utiliser un court car les jetons sont reçus principalement par e-mail et pourraient être intégrés dans un lien HTTP qui peut être validé à l'aide d'un simple appel GET comme `https://127.0.0.1/reset.php?token=any_random_sequence`. Un jeton pourrait donc facilement être une séquence de 32 caractères, par exemple. Considérons une application qui génère des jetons composés de cinq chiffres par souci de simplicité. Les valeurs de jeton valides vont de 00000 à 99999. À un rythme de 10 vérifications par seconde, un attaquant peut forcer brutalement toute la plage en 3 heures environ.

Considérez également que la même application répond par un `Valid token`message si le jeton soumis est valide ; sinon, un `Invalid token`message est renvoyé. Si nous voulions effectuer une attaque par force brute contre les jetons de l'application susmentionnée, nous pourrions utiliser `wfuzz`. Plus précisément, nous pourrions utiliser une correspondance de chaîne pour la chaîne sensible à la casse Valid ( `--ss`"Valid"). Bien sûr, si nous ne savions pas comment l'application Web répond lorsqu'un jeton valide est soumis, nous pourrions utiliser une « correspondance inversée » en recherchant toute réponse qui ne contient pas l'utilisation de «`Invalid token` Invalide`--hs` ». Enfin, une plage d'entiers à cinq chiffres peut être spécifiée et créée à `wfuzz`l'aide de `-z range,00000-99999`. Vous pouvez voir la `wfuzz`commande entière ci-dessous.

```
dsgsec@htb[/htb]$ wfuzz -z range,00000-99999 --ss "Valid" "https://brokenauthentication.hackthebox.eu/token.php?user=admin&token=FUZZ"

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://brokenauthentication.hackthebox.eu/token.php?user=admin&token=FUZZ
Total requests: 100000
===================================================================
ID           Response   Lines    Word     Chars       Payload
===================================================================
00011112:   200        0 L      5 W      26 Ch       "11111"
00017665:   200        0 L      5 W      28 Ch       "17664"
^C
Finishing pending requests...

```

Un attaquant pourrait obtenir un accès en tant qu'utilisateur avant le café du matin en exécutant l'attaque par force brute ci-dessus la nuit. L'utilisateur et un administrateur système qui vérifie les journaux et le trafic réseau remarqueront très probablement une anomalie, mais il pourrait être trop tard. Ce cas marginal peut sembler irréaliste, mais vous serez surpris par le manque de mesures de sécurité dans la nature. Essayez toujours de forcer brutalement les jetons lors de vos tests, étant donné qu'une telle attaque est bruyante et peut également provoquer un déni de service, elle doit donc être exécutée avec beaucoup de soin et éventuellement uniquement après avoir consulté votre client.

* * * * *

Cryptographie faible
--------------------

Même les jetons générés par cryptographie pourraient être prévisibles. Il a été observé que certains développeurs tentent de créer leur propre routine de chiffrement, recourant souvent à la sécurité via des processus d'obscurité. Les deux cas conduisent généralement à un faible caractère aléatoire des jetons. Aussi, certaines fonctions cryptographiques se sont avérées moins sécurisées. Rouler votre propre cryptage n'est jamais une bonne idée. Pour rester du bon côté, nous devons toujours utiliser des algorithmes de cryptage modernes et bien connus qui ont été largement révisés. Un cas d'utilisation fascinant sur les attaques contre la cryptographie faible est la recherche effectuée par le laboratoire F-Secure sur OpenCart, publiée [ici](https://labs.f-secure.com/advisories/opencart-predictable-password-reset-tokens/) .

Les chercheurs ont découvert que l'application utilise la fonction PHP [mt_rand()](https://www.php.net/manual/en/function.mt-rand.php) , qui est connue pour être [vulnérable](https://phpsecurity.readthedocs.io/en/latest/Insufficient-Entropy-For-Random-Values.html) en raison d'un manque d'entropie suffisante lors du processus de génération de valeur aléatoire. OpenCart utilise cette fonction vulnérable pour générer toutes les valeurs aléatoires, de CAPTCHA à session_id pour réinitialiser les jetons. L'accès à certains jetons cryptographiquement non sécurisés permet d'identifier la graine, ce qui conduit à prédire tout jeton passé et futur.

Attaquer mt_rand() n'est en aucun cas une tâche facile, mais des attaques de preuve de concept ont été publiées [ici](https://github.com/GeorgeArgyros/Snowflake) et [ici](https://download.openwall.net/pub/projects/php_mt_seed/) . mt_rand() doit donc être utilisé avec prudence et en tenant compte des implications de sécurité. L'exemple d'OpenCart était un cas sérieux car un attaquant pouvait facilement obtenir certaines valeurs générées à l'aide de mt_rand() via CAPTCHA sans même avoir besoin d'un compte utilisateur valide.

* * * * *

Réinitialiser le jeton en tant que mot de passe temporaire
----------------------------------------------------------

Il convient de noter que certaines applications utilisent des jetons de réinitialisation comme véritables mots de passe temporaires. De par sa conception, tout mot de passe temporaire doit être invalidé dès que l'utilisateur se connecte et le modifie. Il est peu probable que ces mots de passe temporaires ne soient pas invalidés immédiatement après utilisation. Cela étant dit, essayez d'être aussi minutieux que possible et vérifiez si les jetons de réinitialisation utilisés comme mots de passe temporaires peuvent être réutilisés.

Il y a plus de chances que des mots de passe temporaires soient générés à l'aide d'un algorithme prévisible comme mt_rand(), md5(nom d'utilisateur), etc., alors assurez-vous de tester la sécurité de l'algorithme en analysant certains jetons capturés.
