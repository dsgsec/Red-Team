Gestion des erreurs SQLMap
==========================

* * * * *

Nous pouvons rencontrer de nombreux problèmes lors de la configuration de SQLMap ou de son utilisation avec des requêtes HTTP. Dans cette section, nous discuterons des mécanismes recommandés pour trouver la cause et la corriger correctement.

* * * * *

Erreurs d'affichage
-------------------

La première étape consiste généralement à basculer le `--parse-errors`, pour analyser les erreurs du SGBD (le cas échéant) et les afficher dans le cadre de l'exécution du programme :

```
...SNIP...
[16:09:20] [INFO] testing if GET parameter 'id' is dynamic
[16:09:20] [INFO] GET parameter 'id' appears to be dynamic
[16:09:20] [WARNING] parsed DBMS error message: 'SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '))"',),)((' at line 1'"
[16:09:20] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[16:09:20] [WARNING] parsed DBMS error message: 'SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''YzDZJELylInm' at line 1'
...SNIP...

```

Avec cette option, SQLMap imprimera automatiquement l'erreur du SGBD, nous donnant ainsi plus de clarté sur le problème afin que nous puissions le résoudre correctement.

* * * * *

Stockez le trafic
-----------------

L' `-t`option stocke l'intégralité du contenu du trafic dans un fichier de sortie :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt

dsgsec@htb[/htb]$ cat /tmp/traffic.txt
HTTP request [#1]:
GET /?id=1 HTTP/1.1
Host: www.example.com
Cache-control: no-cache
Accept-encoding: gzip,deflate
Accept: */*
User-agent: sqlmap/1.4.9 (http://sqlmap.org)
Connection: close

HTTP response [#1] (200 OK):
Date: Thu, 24 Sep 2020 14:12:50 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 914
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://www.example.com:80/?id=1

<!DOCTYPE html>
<html lang="en">
...SNIP...

```

Comme nous pouvons le voir dans le résultat ci-dessus, le `/tmp/traffic.txt`fichier contient désormais toutes les requêtes HTTP envoyées et reçues. Nous pouvons donc désormais enquêter manuellement sur ces demandes pour voir où le problème se produit.

* * * * *

Sortie détaillée
----------------

Un autre indicateur utile est l' `-v`option, qui augmente le niveau de verbosité de la sortie de la console :

```
dsgsec@htb[/htb]$ sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.9}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 16:17:40 /2020-09-24/

[16:17:40] [DEBUG] cleaning up configuration parameters
[16:17:40] [DEBUG] setting the HTTP timeout
[16:17:40] [DEBUG] setting the HTTP User-Agent header
[16:17:40] [DEBUG] creating HTTP requests opener object
[16:17:40] [DEBUG] resolving hostname 'www.example.com'
[16:17:40] [INFO] testing connection to the target URL
[16:17:40] [TRAFFIC OUT] HTTP request [#1]:
GET /?id=1 HTTP/1.1
Host: www.example.com
Cache-control: no-cache
Accept-encoding: gzip,deflate
Accept: */*
User-agent: sqlmap/1.4.9 (http://sqlmap.org)
Connection: close

[16:17:40] [DEBUG] declared web page charset 'utf-8'
[16:17:40] [TRAFFIC IN] HTTP response [#1] (200 OK):
Date: Thu, 24 Sep 2020 14:17:40 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 914
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://www.example.com:80/?id=1

<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <meta name="description" content="">
  <meta name="author" content="">
  <link href="vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
  <title>SQLMap Essentials - Case1</title>
</head>

<body>
...SNIP...

```

Comme nous pouvons le voir, l' `-v 6`option imprimera directement toutes les erreurs et la requête HTTP complète sur le terminal afin que nous puissions suivre tout ce que fait SQLMap en temps réel.

* * * * *

Utiliser un proxy
-----------------

Enfin, nous pouvons utiliser l' `--proxy`option pour rediriger l'ensemble du trafic via un proxy (MiTM) (par exemple `Burp`). Cela acheminera tout le trafic SQLMap via `Burp`, afin que nous puissions ultérieurement examiner manuellement toutes les requêtes, les répéter et utiliser toutes les fonctionnalités de `Burp`avec ces requêtes :

![burp_proxy](https://academy.hackthebox.com/storage/modules/58/eIwJeV3.png)
