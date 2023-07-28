Bruteforce Username
================================

* * * * *

`Username enumeration`est souvent négligé, probablement parce qu'il est supposé qu'un nom d'utilisateur n'est pas une information privée. Lorsque vous écrivez un message à un autre utilisateur, nous supposons généralement que nous connaissons son nom d'utilisateur, son adresse e-mail, etc. Le même nom d'utilisateur est souvent réutilisé pour accéder à d'autres services tels que , `FTP`et `RDP`, `SSH`entre autres. Étant donné que de nombreuses applications Web nous permettent d'identifier les noms d'utilisateur, nous devrions tirer parti de cette fonctionnalité et les utiliser pour des attaques ultérieures.

![](https://academy.hackthebox.com/storage/modules/80/05-user_search.png)

Par exemple, sur [Hack The Box](https://hackthebox.eu/) , `userid`et `username`sont différents. Par conséquent, l'énumération des utilisateurs n'est pas possible, mais un large éventail d'applications Web souffrent de cette vulnérabilité.

Les noms d'utilisateur sont souvent beaucoup moins compliqués que les mots de passe. Ils contiennent rarement des caractères spéciaux lorsqu'il ne s'agit pas d'adresses e-mail. Avoir une liste d'utilisateurs communs donne à un attaquant certains avantages. En plus d'obtenir une bonne expérience utilisateur (UX), il est rare de rencontrer des noms d'utilisateur aléatoires ou difficilement prévisibles. Un utilisateur se souviendra plus facilement de son adresse e-mail ou de son surnom qu'un nom d'utilisateur généré par ordinateur et (pseudo) aléatoire.

Disposant d'une liste de noms d'utilisateur valides, un attaquant peut réduire la portée d'une attaque par force brute ou mener des attaques ciblées (en tirant parti de l'OSINT) contre les employés du support ou les utilisateurs eux-mêmes. De plus, un mot de passe commun pourrait être facilement vaporisé sur des comptes valides, conduisant souvent à une compromission réussie du compte.

Il convient de noter que les noms d'utilisateur peuvent également être récoltés en crawlant une application Web ou en utilisant des informations publiques, par exemple des profils d'entreprise sur les réseaux sociaux.

La protection contre les attaques par énumération de noms d'utilisateur peut avoir un impact sur l'expérience utilisateur. Une application Web révélant qu'un nom d'utilisateur existe ou non peut aider un utilisateur légitime à identifier qu'il n'a pas saisi correctement son nom d'utilisateur, mais il en va de même pour un attaquant essayant de déterminer des noms d'utilisateur valides. Même les frameworks Web bien connus et matures, comme WordPress, souffrent de l'énumération des utilisateurs car l'équipe de développement a choisi d'avoir une UX plus fluide en abaissant un peu le niveau de sécurité du framework. Vous pouvez vous référer à ce [billet](https://core.trac.wordpress.org/ticket/3708) pour toute l'histoire

Nous pouvons voir le message de réponse après avoir soumis un nom d'utilisateur inexistant indiquant que le nom d'utilisateur saisi est inconnu.

![](https://academy.hackthebox.com/storage/modules/80/06-bruteforce_username/01-wordpress_wrong_username.png)

Dans le deuxième exemple, nous pouvons voir le message de réponse après avoir soumis un nom d'utilisateur valide (et un mot de passe erroné) indiquant que le nom d'utilisateur saisi existe, mais que le mot de passe est incorrect.

![](https://academy.hackthebox.com/storage/modules/80/06-bruteforce_username/02-wordpress_wrong_password.png)

La différence est claire. Lors du premier essai, lorsqu'un nom d'utilisateur inexistant est soumis, l'application affiche une entrée de connexion vide avec un message "Nom d'utilisateur inconnu". Lors de la deuxième tentative, lorsqu'un nom d'utilisateur existant est soumis (avec un mot de passe invalide), le champ du formulaire de nom d'utilisateur est pré-rempli avec le nom d'utilisateur valide. L'application affiche un message indiquant clairement que le mot de passe est erroné (pour ce nom d'utilisateur valide).

* * * * *

Attaque d'utilisateur inconnu
-----------------------------

Lorsqu'un échec de connexion se produit et que l'application répond par "Nom d'utilisateur inconnu" ou un message similaire, un attaquant peut effectuer une attaque par force brute contre la fonctionnalité de connexion à la recherche d'un, " " `The password you entered for the username X is incorrect`ou d'un message similaire. Lors d'un test d'intrusion, n'oubliez pas de vérifier également les noms d'utilisateur génériques tels que helpdesk, tech, admin, demo, guest, etc.

[SecLists](https://github.com/danielmiessler/SecLists/tree/master/Usernames) fournit une vaste collection de listes de mots qui peuvent être utilisées comme point de départ pour monter des attaques d'énumération d'utilisateurs.

Essayons de forcer brutalement une application Web. Nous avons deux façons de voir comment l'application Web attend les données. L'une consiste à inspecter le formulaire HTML et l'autre à utiliser un proxy d'interception pour capturer la demande POST réelle. Lorsque nous traitons d'une forme de base, il n'y a pas de différences significatives. Cependant, certaines applications utilisent du JavaScript obscurci ou artificiel pour masquer ou masquer des détails. Dans ces cas, l'utilisation d'un proxy d'interception est généralement préférée. En ouvrant la page de connexion et en essayant de se connecter, nous pouvons voir que l'application accepte le `userid`champ `Username`et le mot de passe comme `Password`.

![](https://academy.hackthebox.com/storage/modules/80/unknown_username-burp_request.png)

On remarque que l'application répond par un `Unknown username`message, et on devine qu'elle utilise un message différent lorsque le nom d'utilisateur est valide.

Nous pouvons effectuer l'attaque par force brute en utilisant `wfuzz`et une correspondance de chaîne inversée par rapport au texte de réponse ( `--hs`"Nom d'utilisateur inconnu", où " `hs`" doit être un mnémonique utilisé pour le masquage de chaîne), en utilisant une courte liste de mots de SecLists. Puisque nous n'essayons pas de trouver un mot de passe valide, nous ne nous soucions pas du `Password`champ, nous allons donc en utiliser un factice.

#### WFuzz - Nom d'utilisateur inconnu

  WFuzz - Nom d'utilisateur inconnu

```
dsgsec@htb[/htb]$ wfuzz -c -z file,/opt/useful/SecLists/Usernames/top-usernames-shortlist.txt -d "Username=FUZZ&Password=dummypass" --hs "Unknown username" http://brokenauthentication.hackthebox.eu/user_unknown.php

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://brokenauthentication.hackthebox.eu/user_unknown.php
Total requests: 17

===================================================================
ID           Response   Lines    Word     Chars       Payload
===================================================================

000000002:   200        56 L     143 W    1984 Ch     "admin"

Total time: 0.017432
Processed Requests: 17
Filtered Requests: 16
Requests/sec.: 975.1927

```

Bien que `wfuzz`masque automatiquement toute réponse contenant un message "Nom d'utilisateur inconnu", nous remarquons que "admin" est un utilisateur valide (les noms d'utilisateur restants sur la liste de mots top-username-shortlist.txt ne sont pas valides). Si une excellente UX n'est pas une exigence absolue, une application doit répondre avec un message générique comme "Identifiants invalides" pour les noms d'utilisateur inconnus et les mots de passe erronés.

![](https://academy.hackthebox.com/storage/modules/80/06-bruteforce_username/03-custom_invalid_credentials.png)

* * * * *

Inférence d'existence de nom d'utilisateur
------------------------------------------

Parfois, une application Web peut ne pas déclarer explicitement qu'elle ne connaît pas un nom d'utilisateur spécifique, mais permet à un attaquant de déduire cette information. Certaines applications Web préremplissent la valeur d'entrée du nom d'utilisateur si le nom d'utilisateur est valide et connu, mais laissent la valeur d'entrée vide ou avec une valeur par défaut lorsque le nom d'utilisateur est inconnu. C'est assez courant sur les versions mobiles des sites Web et c'était également le cas sur la page de connexion WordPress vulnérable que nous avons vue plus tôt. Lors du développement, essayez toujours de donner la même expérience pour les connexions échouées et acceptées : même une légère différence est plus que suffisante pour déduire une information.

En testant une application web en se connectant en tant qu'utilisateur inconnu, on remarque un message d'erreur générique et une page de connexion vide :

![](https://academy.hackthebox.com/storage/modules/80/06-bruteforce_username/04-inference_unknown.png)

Lorsque nous essayons de nous connecter en tant qu'utilisateur "admin", nous remarquons que le champ de saisie est pré-rempli avec le (probablement) un nom d'utilisateur valide, même si nous recevons le même message d'erreur générique :

![](https://academy.hackthebox.com/storage/modules/80/06-bruteforce_username/05-inference_valid.png)

Bien que cela soit rare, il est également possible que différents cookies soient définis lorsqu'un nom d'utilisateur est valide ou non. Par exemple, pour vérifier les tentatives de mot de passe à l'aide de contrôles côté client, une application Web peut définir puis vérifier un cookie nommé "failed_login" uniquement lorsque le nom d'utilisateur est valide. Inspectez soigneusement les réponses en surveillant les différences dans les en-têtes HTTP et le code source HTML.

* * * * *

Attaque de synchronisation
--------------------------

Certaines fonctions d'authentification peuvent contenir des défauts de conception. Un exemple est une fonction d'authentification où le nom d'utilisateur et le mot de passe sont vérifiés séquentiellement. Analysons la routine ci-dessous.

#### Code d'authentification vulnérable

Code : php

```
<?php
// connect to database
$db = mysqli_connect("localhost", "dbuser", "dbpass", "dbname");

// retrieve row data for user
$result = $db->query('SELECT * FROM users WHERE username="'.safesql($_POST['user']).'" AND active=1');

// $db->query() replies True if there are at least a row (so a user), and False if there are no rows (so no users)
if ($result) {
  // retrieve a row. don't use this code if multiple rows are expected
  $row = mysqli_fetch_row($result);

  // hash password using custom algorithm
  $cpass = hash_password($_POST['password']);

  // check if received password matches with one stored in the database
  if ($cpass === $row['cpassword']) {
	echo "Welcome $row['username']";
  } else {
    echo "Invalid credentials.";
  }
} else {
  echo "Invalid credentials.";
}
?>

```

L'extrait de code se connecte d'abord à la base de données, puis exécute une requête pour récupérer une ligne entière où le nom d'utilisateur correspond à celui demandé. S'il n'y a pas de résultat, la fonction se termine par un message générique. Lorsque `$result`est vrai (l'utilisateur existe et est actif), le mot de passe fourni est haché et comparé. Si l'algorithme de hachage utilisé est suffisamment puissant, les différences de synchronisation entre les deux branches seront perceptibles. En calculant `$cpass`à l'aide d'une fonction générique `hash_password()`, le temps de réponse sera plus élevé que dans l'autre cas. Cette petite erreur pourrait être évitée en vérifiant l'utilisateur et le mot de passe dans la même étape, en ayant un temps similaire pour les noms d'utilisateur valides et non valides.

Téléchargez le script [timing.py](https://academy.hackthebox.com/storage/modules/80/scripts/timing_py.txt) pour observer ces types de différences de temps et exécutez-le sur un exemple d'application Web ( [timing.php](https://academy.hackthebox.com/storage/modules/80/scripts/timing_php.txt) ) qui utilise `bcrypt`.

#### Attaque de synchronisation - Timing.py

  Attaque de synchronisation - Timing.py

```
dsgsec@htb[/htb]$ python3 timing.py /opt/useful/SecLists/Usernames/top-usernames-shortlist.txt

[+] user root took 0.003
[+] user admin took 0.263
[+] user test took 0.005
[+] user guest took 0.003
[+] user info took 0.001
[+] user adm took 0.001
[+] user mysql took 0.001
[+] user user took 0.001
[+] user administrator took 0.001
[+] user oracle took 0.001
[+] user ftp took 0.001
[+] user pi took 0.001
[+] user puppet took 0.001
[+] user ansible took 0.001
[+] user ec2-user took 0.001
[+] user vagrant took 0.001
[+] user azureuser took 0.001

```

Étant donné qu'il pourrait y avoir un problème de réseau, il est facile d'identifier "admin" comme un utilisateur valide car cela a pris beaucoup plus de temps que les autres utilisateurs testés. Si l'algorithme utilisé était rapide, les différences de temps seraient plus petites et un attaquant pourrait avoir un faux positif en raison d'un retard du réseau ou d'une charge CPU. Cependant, l'attaque est toujours possible en répétant un grand nombre de requêtes pour créer un modèle. Bien que nous puissions supposer qu'une application moderne hache les mots de passe à l'aide d'un algorithme robuste pour ralentir au maximum une potentielle attaque par force brute hors ligne, il est possible de déduire des informations même si elle utilise un algorithme rapide comme `MD5`ou `SHA1`.

Lorsque la base d'utilisateurs [de LinkedIn](https://en.wikipedia.org/wiki/2012_LinkedIn_hack) a été divulguée en 2012, les professionnels d'InfoSec ont lancé un débat sur `SHA1`son utilisation comme algorithme de hachage pour les mots de passe des utilisateurs. Bien qu'il `SHA1`ne se soit pas cassé pendant ces jours, il était connu comme une solution de hachage non sécurisée. Les professionnels de l'infosec ont commencé à se disputer sur le choix d'utiliser `SHA1`à la place des algorithmes de hachage plus robustes comme [scrypt](https://www.tarsnap.com/scrypt.html) , [bcrypt](https://en.wikipedia.org/wiki/Bcrypt) ou [PBKDF](https://en.wikipedia.org/wiki/Pbkdf2) (ou [argon2](https://en.wikipedia.org/wiki/Argon2) ).

S'il est toujours préférable d'utiliser un algorithme plus robuste qu'un algorithme plus faible, un ingénieur en architecture doit également garder à l'esprit le coût de calcul. Ce script Python très basique aide à faire la lumière sur le problème :

#### Python - Algorithmes de chiffrement

Code : Python

```
import scrypt
import bcrypt
import datetime
import hashlib

rounds = 100
salt = bcrypt.gensalt()

t0 = datetime.datetime.now()

for x in range(rounds):
    scrypt.hash(str(x).encode(), salt)

t1 = datetime.datetime.now()

for x in range(rounds):
    hashlib.sha1(str(x).encode())

t2 = datetime.datetime.now()

for x in range(rounds):
    bcrypt.hashpw(str(x).encode(), salt)

t3 = datetime.datetime.now()

print("sha1:   {}\nscrypt: {}\nbcrypt: {}".format(t2-t1,t1-t0,t3-t2))

```

Gardez à l'esprit que les meilleures pratiques modernes recommandent fortement d'utiliser des algorithmes plus robustes, ce qui entraîne une augmentation du temps CPU et de l'utilisation de la RAM. Si nous nous concentrons `bcrypt`pendant une minute, l'exécution du script ci-dessus sur un i5 de huitième génération à 8 cœurs donne les résultats suivants.

#### Python - Hashtime.py

  Python - Hashtime.py

```
dsgsec@htb[/htb]$ python3 hashtime.py

sha1:   0:00:00.000082
scrypt: 0:00:03.907575
bcrypt: 0:00:22.660548

```

Ajoutons un peu de contexte en passant en revue un exemple approximatif :

-   LinkedIn compte environ 200 millions d'utilisateurs quotidiens, ce qui signifie environ 24 connexions par seconde (nous n'excluons pas les utilisateurs avec un jeton souvenir).

S'ils utilisaient un algorithme robuste comme `bcrypt`, qui utilisait 0,23 seconde pour chaque tour sur notre machine de test, ils auraient besoin de six serveurs juste pour permettre aux gens de se connecter. Cela ne semble pas être un gros problème pour une entreprise qui gère des milliers de serveurs, mais cela nécessiterait une refonte de l'architecture.

* * * * *

Énumérer via la réinitialisation du mot de passe
------------------------------------------------

Les formulaires de réinitialisation sont souvent moins bien protégés que ceux de connexion. Par conséquent, ils divulguent très souvent des informations sur un nom d'utilisateur valide ou invalide. Comme nous en avons déjà discuté, une application qui répond par un " `You should receive a message shortly`" lorsqu'un nom d'utilisateur valide a été trouvé et " `Username unknown, check your data`" pour une entrée invalide fuit la présence d'utilisateurs enregistrés.

Cette attaque est bruyante car certains utilisateurs valides recevront probablement un e-mail demandant une réinitialisation du mot de passe. Cela étant dit, ces e-mails ne reçoivent souvent pas l'attention appropriée de la part des utilisateurs finaux.

* * * * *

Énumérer via le formulaire d'inscription
----------------------------------------

Par défaut, un formulaire d'inscription qui invite les utilisateurs à choisir leur nom d'utilisateur répond généralement par un message clair lorsque le nom d'utilisateur sélectionné existe déjà ou fournit d'autres « indications » si tel est le cas. En abusant de ce comportement, un attaquant pourrait enregistrer des noms d'utilisateur courants, comme admin, administrateur, tech, pour énumérer les noms valides. Un formulaire d'inscription sécurisé doit implémenter une certaine protection avant de vérifier si le nom d'utilisateur sélectionné existe, comme un CAPTCHA.

Une caractéristique intéressante des adresses e-mail que beaucoup de gens ne connaissent pas ou n'ont pas à l'esprit lors des tests est le sous-adressage. Cette extension, définie dans [RFC5233](https://tools.ietf.org/html/rfc5233) , indique que tout élément `+tag`situé dans la partie gauche d'une adresse e-mail doit être ignoré par l'agent de transport de courrier (MTA) et utilisé comme balise pour les filtres tamis. Cela signifie que l'écriture à une adresse e-mail comme `student+htb@hackthebox.eu`enverra l'e-mail à `student@hackthebox.eu`et, si les filtres sont pris en charge et correctement configurés, sera placé dans le dossier `htb`. Très peu d'applications web respectent cette RFC, ce qui conduit à la possibilité d'enregistrer une infinité d'utilisateurs en utilisant un tag et une seule adresse email réelle.

![](https://academy.hackthebox.com/storage/modules/80/username_registration.png)

Bien sûr, cette attaque est assez bruyante et doit être effectuée avec beaucoup de soin.

* * * * *

Noms d'utilisateur prévisibles
------------------------------

Dans les applications Web avec moins d'exigences UX comme, par exemple, la banque à domicile ou lorsqu'il est nécessaire de créer de nombreux utilisateurs dans un lot, nous pouvons voir des noms d'utilisateur créés de manière séquentielle.

Bien que cela soit rare, vous pouvez rencontrer des comptes tels que `user1000`, `user1001`. Il est également possible que les utilisateurs "administrateurs" aient une convention de nommage prévisible, comme `support.it`, `support.fr`, ou similaire. Un attaquant pourrait déduire l'algorithme utilisé pour créer des utilisateurs (quatre chiffres incrémentiels, code de pays, etc.) et deviner les comptes d'utilisateurs existants à partir de certains connus.
