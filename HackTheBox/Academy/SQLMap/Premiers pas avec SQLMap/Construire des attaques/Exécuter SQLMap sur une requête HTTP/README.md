Exécuter SQLMap sur une requête HTTP
====================================

* * * * *

SQLMap dispose de nombreuses options et commutateurs qui peuvent être utilisés pour configurer correctement la requête (HTTP) avant son utilisation.

Dans de nombreux cas, de simples erreurs telles que l'oubli de fournir des valeurs de cookie appropriées, une configuration trop compliquée avec une longue ligne de commande ou une déclaration incorrecte de données POST formatées empêcheront la détection et l'exploitation correctes de la vulnérabilité potentielle SQLi.

* * * * *

Commandes de boucle
-------------------

L'un des moyens les meilleurs et les plus simples de configurer correctement une requête SQLMap sur une cible spécifique (c'est-à-dire une requête Web avec des paramètres à l'intérieur) consiste à utiliser la `Copy as cURL`fonctionnalité du panneau Réseau (Moniteur) dans les outils de développement Chrome, Edge ou Firefox : ![copie_as_curl](https://academy.hackthebox.com/storage/modules/58/M5UVR6n.png)

En collant le contenu du presse-papiers ( `Ctrl-V`) dans la ligne de commande et en remplaçant la commande d'origine `curl`par `sqlmap`, nous pouvons utiliser SQLMap avec la `curl`commande identique :

```
dsgsec@htb[/htb]$ sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'

```

Lorsque vous fournissez des données à tester à SQLMap, il doit y avoir soit une valeur de paramètre pouvant être évaluée pour la vulnérabilité SQLi, soit des options/commutateurs spécialisés pour la recherche automatique des paramètres (par exemple, `--crawl`ou `--forms`) `-g`.

* * * * *

Requêtes GET/POST
-----------------

Dans le scénario le plus courant, `GET`les paramètres sont fournis à l'aide de l'option `-u`/ `--url`, comme dans l'exemple précédent. En ce qui concerne `POST`les données de test, le `--data`drapeau peut être utilisé comme suit :

```
dsgsec@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1&name=test'

```

Dans de tels cas, `POST`les paramètres `uid`et `name`seront testés pour la vulnérabilité SQLi. Par exemple, si nous avons une indication claire que le paramètre `uid`est sujet à une vulnérabilité SQLi, nous pourrions limiter les tests à ce paramètre uniquement en utilisant `-p uid`. Sinon, nous pourrions le marquer dans les données fournies à l'aide d'un marqueur spécial `*`comme suit :

```
dsgsec@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'

```

* * * * *

Requêtes HTTP complètes
-----------------------

Si nous devons spécifier une requête HTTP complexe avec de nombreuses valeurs d'en-tête différentes et un corps POST allongé, nous pouvons utiliser l' `-r`indicateur. Avec cette option, SQLMap est fourni avec le « fichier de requête », contenant l'intégralité de la requête HTTP dans un seul fichier texte. Dans un scénario courant, une telle requête HTTP peut être capturée depuis une application proxy spécialisée (par exemple `Burp`) et écrite dans le fichier de requête, comme suit :

![rot_request](https://academy.hackthebox.com/storage/modules/58/x7ND6VQ.png)

Un exemple de requête HTTP capturée avec `Burp`ressemblerait à :

Code : http

```
GET /?id=1 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947"
Cache-Control: max-age=0

```

Nous pouvons soit copier manuellement la requête HTTP depuis l'intérieur `Burp`et l'écrire dans un fichier, soit cliquer avec le bouton droit sur la requête `Burp`et choisir `Copy to file`. Une autre façon de capturer la requête HTTP complète consisterait à utiliser le navigateur, comme mentionné précédemment dans la section, et à choisir l'option `Copy`> `Copy Request Headers`, puis à coller la requête dans un fichier.

Pour exécuter SQLMap avec un fichier de requête HTTP, nous utilisons l' `-r`indicateur, comme suit :

```
dsgsec@htb[/htb]$ sqlmap -r req.txt
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.9}
|_ -| . [(]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 14:32:59 /2020-09-11/

[14:32:59] [INFO] parsing HTTP request from 'req.txt'
[14:32:59] [INFO] testing connection to the target URL
[14:32:59] [INFO] testing if the target URL content is stable
[14:33:00] [INFO] target URL content is stable

```

Astuce : de la même manière que pour l'option '--data', dans le fichier de requête enregistré, nous pouvons spécifier le paramètre que nous voulons injecter avec un astérisque (*), tel que '/?id=*'.

* * * * *

Requêtes SQLMap personnalisées
------------------------------

Si nous souhaitons créer manuellement des requêtes complexes, il existe de nombreux commutateurs et options pour affiner SQLMap.

Par exemple, s'il est nécessaire de spécifier la valeur du cookie (de session), l' `PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c`option `--cookie`sera utilisée comme suit :

```
dsgsec@htb[/htb]$ sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

```

Le même effet peut être obtenu en utilisant l'option`-H/--header` :

```
dsgsec@htb[/htb]$ sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'

```

Nous pouvons appliquer la même chose aux options telles que `--host`, `--referer`, et `-A/--user-agent`, qui sont utilisées pour spécifier les mêmes valeurs d'en-tête HTTP.

De plus, il existe un commutateur `--random-agent`conçu pour sélectionner aléatoirement une `User-agent`valeur d'en-tête dans la base de données incluse des valeurs normales du navigateur. Il s'agit d'un changement important à retenir, car de plus en plus de solutions de protection suppriment automatiquement tout le trafic HTTP contenant la valeur d'agent utilisateur par défaut reconnaissable de SQLMap (par exemple `User-agent: sqlmap/1.4.9.12#dev (http://sqlmap.org)`). Alternativement, le `--mobile`commutateur peut être utilisé pour imiter le smartphone en utilisant la même valeur d'en-tête.

Bien que SQLMap cible par défaut uniquement les paramètres HTTP, il est possible de tester les en-têtes pour la vulnérabilité SQLi. Le moyen le plus simple est de spécifier la marque d'injection "personnalisée" après la valeur de l'en-tête (par exemple `--cookie="id=1*"`). Le même principe s'applique à toute autre partie de la demande.

De plus, si nous souhaitons spécifier une méthode HTTP alternative, autre que `GET`et `POST`(par exemple, `PUT`), nous pouvons utiliser l'option `--method`, comme suit :

```
dsgsec@htb[/htb]$ sqlmap -u www.target.com --data='id=1' --method PUT

```

* * * * *

Requêtes HTTP personnalisées
----------------------------

Outre le `POST`style de corps de données de formulaire le plus courant (par exemple `id=1`), SQLMap prend également en charge les requêtes HTTP au format JSON (par exemple `{"id":1}`) et XML (par exemple `<element><id>1</id></element>`).

La prise en charge de ces formats est implémentée de manière « détendue » ; ainsi, il n'y a pas de contraintes strictes sur la façon dont les valeurs des paramètres sont stockées à l'intérieur. Dans le cas où le `POST`corps est relativement simple et court, cette option `--data`suffira.

Cependant, dans le cas d'un corps POST complexe ou long, on peut à nouveau utiliser l' `-r`option :

```
dsgsec@htb[/htb]$ cat req.txt
HTTP / HTTP/1.0
Host: www.example.com

{
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "Example JSON",
      "body": "Just an example",
      "created": "2020-05-22T14:56:29.000Z",
      "updated": "2020-05-22T14:56:28.000Z"
    },
    "relationships": {
      "author": {
        "data": {"id": "42", "type": "user"}
      }
    }
  }]
}

```

```
dsgsec@htb[/htb]$ sqlmap -r req.txt
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.4.9}
|_ -| . [)]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 00:03:44 /2020-09-15/

[00:03:44] [INFO] parsing HTTP request from 'req.txt'
JSON data found in HTTP body. Do you want to process it? [Y/n/q]
[00:03:45] [INFO] testing connection to the target URL
[00:03:45] [INFO] testing if the target URL content is stable
[00:03:46] [INFO] testing if HTTP parameter 'JSON type' is dynamic
[00:03:46] [WARNING] HTTP parameter 'JSON type' does not appear to be dynamic
[00:03:46] [WARNING] heuristic (basic) test shows that HTTP parameter 'JSON type' might not be injectable

```

Démarrer l'instance

 / 1 apparition restante
