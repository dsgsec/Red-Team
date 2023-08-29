Énumération des utilisateurs
============================

L'énumération d'une liste d'utilisateurs valides est une phase critique d'une évaluation de sécurité WordPress. Armés de cette liste, nous pourrons peut-être deviner les informations d'identification par défaut ou effectuer une attaque de mot de passe par force brute. En cas de succès, nous pourrons peut-être nous connecter au backend WordPress en tant qu'auteur ou même en tant qu'administrateur. Cet accès peut potentiellement être exploité pour modifier le site Web WordPress ou même interagir avec le serveur Web sous-jacent.

Il existe deux méthodes pour effectuer une énumération manuelle des noms d'utilisateur.

* * * * *

Première méthode
----------------

La première méthode consiste à examiner les publications pour découvrir l'identifiant attribué à l'utilisateur et son nom d'utilisateur correspondant. Si nous passons la souris sur le lien de l'auteur de la publication intitulé « par administrateur », comme le montre l'image ci-dessous, un lien vers le compte de l'utilisateur apparaît dans le coin inférieur gauche du navigateur Web.

![image](https://academy.hackthebox.com/storage/modules/17/by-admin.png)

L' `admin`utilisateur se voit généralement attribuer l'ID utilisateur `1`. Nous pouvons le confirmer en spécifiant l'ID utilisateur pour le `author`paramètre dans l'URL.

http://blog.inlanefreight.com/?author=1

Cela peut également être fait à `cURL`partir de la ligne de commande. La réponse HTTP dans la sortie ci-dessous montre l'auteur qui correspond à l'ID utilisateur. L'URL dans l' `Location`en-tête confirme que cet ID utilisateur appartient à l' `admin`utilisateur.

#### Utilisateur existant

  Utilisateur existant

```
dsgsec@htb[/htb]$ curl -s -I -X GET http://blog.inlanefreight.com/?author=1

HTTP/1.1 301 Moved Permanently
Date: Wed, 13 May 2020 20:47:08 GMT
Server: Apache/2.4.29 (Ubuntu)
X-Redirect-By: WordPress
Location: http://blog.inlanefreight.com/index.php/author/admin/
Content-Length: 0
Content-Type: text/html; charset=UTF-8

```

La `cURL`demande ci-dessus nous redirige ensuite vers la page de profil de l'utilisateur ou la page de connexion principale. Si l'utilisateur n'existe pas, nous recevons un `404 Not Found error`.

#### Utilisateur non existant

  Utilisateur non existant

```
dsgsec@htb[/htb]$ curl -s -I -X GET http://blog.inlanefreight.com/?author=100

HTTP/1.1 404 Not Found
Date: Wed, 13 May 2020 20:47:14 GMT
Server: Apache/2.4.29 (Ubuntu)
Expires: Wed, 11 Jan 1984 05:00:00 GMT
Cache-Control: no-cache, must-revalidate, max-age=0
Link: <http://blog.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/"
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8

```

* * * * *

Deuxième méthode
----------------

La deuxième méthode nécessite une interaction avec le `JSON`point final, ce qui nous permet d'obtenir une liste d'utilisateurs. Cela a été modifié dans le noyau de WordPress après la version 4.7.1, et les versions ultérieures indiquent uniquement si un utilisateur est configuré ou non. Avant cette version, tous les utilisateurs ayant publié un article étaient affichés par défaut.

#### Point de terminaison JSON

  Point de terminaison JSON

```
dsgsec@htb[/htb]$ curl http://blog.inlanefreight.com/wp-json/wp/v2/users | jq

[
  {
    "id": 1,
    "name": "admin",
    "url": "",
    "description": "",
    "link": "http://blog.inlanefreight.com/index.php/author/admin/",
    <SNIP>
  },
  {
    "id": 2,
    "name": "ch4p",
    "url": "",
    "description": "",
    "link": "http://blog.inlanefreight.com/index.php/author/ch4p/",
    <SNIP>
  },
<SNIP>
```
