Introduction à la falsification des verbes HTTP
============================

* * * * *

Le protocole `HTTP` fonctionne en acceptant diverses méthodes HTTP en tant que `verbes` au début d'une requête HTTP. Selon la configuration du serveur Web, les applications Web peuvent être scriptées pour accepter certaines méthodes HTTP pour leurs diverses fonctionnalités et effectuer une action particulière en fonction du type de la demande.

Alors que les programmeurs considèrent principalement les deux méthodes HTTP les plus couramment utilisées, `GET` et `POST`, n'importe quel client peut envoyer n'importe quelle autre méthode dans ses requêtes HTTP, puis voir comment le serveur Web gère ces méthodes. Supposons que l'application Web et le serveur Web principal soient configurés uniquement pour accepter les requêtes `GET` et `POST` . Dans ce cas, l'envoi d'une requête différente entraînera l'affichage d'une page d'erreur du serveur Web, ce qui n'est pas une vulnérabilité grave en soi (autre que de fournir une mauvaise expérience utilisateur et potentiellement conduire à la divulgation d'informations). D'autre part, si les configurations du serveur Web ne sont pas limitées pour accepter uniquement les méthodes HTTP requises par le serveur Web (par exemple `GET`/`POST`), et que l'application Web n'est pas développée pour gérer d'autres types de requêtes HTTP ( par exemple `HEAD`, `PUT`), alors nous pourrons peut-être exploiter cette configuration non sécurisée pour accéder à des fonctionnalités auxquelles nous n'avons pas accès, ou même contourner certains contrôles de sécurité.

* * * * *

Falsification des verbes HTTP
-------------------

Pour comprendre `HTTP Verb Tampering`, nous devons d'abord connaître les différentes méthodes acceptées par le protocole HTTP. HTTP a [9 verbes différents](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) qui peuvent être acceptés comme méthodes HTTP par les serveurs Web. Outre `GET` et `POST`, voici quelques-uns des verbes HTTP couramment utilisés :

| Verb | Description |
| --- | --- |
| `HEAD` | Identical to a GET request, but its response only contains the `headers`, without the response body |
| `PUT` | Writes the request payload to the specified location |
| `DELETE` | Deletes the resource at the specified location |
| `OPTIONS` | Shows different options accepted by a web server, like accepted HTTP verbs |
| `PATCH` | Apply partial modifications to the resource at the specified location |

Comme vous pouvez l'imaginer, certaines des méthodes ci-dessus peuvent exécuter des fonctionnalités très sensibles, comme écrire (`PUT`) ou supprimer (`DELETE`) des fichiers dans le répertoire webroot sur le serveur principal. Comme indiqué dans le module [Requêtes Web](https://academy.hackthebox.com/course/preview/web-requests) , si un serveur Web n'est pas configuré de manière sécurisée pour gérer ces méthodes, nous pouvons les utiliser pour prendre le contrôle de le serveur principal. Cependant, ce qui rend les attaques HTTP Verb Tampering plus courantes (et donc plus critiques), c'est qu'elles sont causées par une mauvaise configuration du serveur Web principal ou de l'application Web, l'une ou l'autre pouvant entraîner la vulnérabilité.

* * * * *

Configurations non sécurisées
------------------------

Les configurations de serveur Web non sécurisées provoquent le premier type de vulnérabilités HTTP Verb Tampering. La configuration d'authentification d'un serveur Web peut être limitée à des méthodes HTTP spécifiques, ce qui laisserait certaines méthodes HTTP accessibles sans authentification. Par exemple, un administrateur système peut utiliser la configuration suivante pour exiger une authentification sur une page Web particulière :

Code : xml

```
<Limit GET POST>
    Require valid-user
</Limit>

```

Comme nous pouvons le voir, même si la configuration spécifie à la fois les requêtes `GET` et `POST` pour la méthode d'authentification, un attaquant peut toujours utiliser une méthode HTTP différente (comme `HEAD`) pour contourner complètement ce mécanisme d'authentification, comme nous le verrons dans la section suivante. Cela conduit finalement à un contournement d'authentification et permet aux attaquants d'accéder à des pages Web et à des domaines auxquels ils ne devraient pas avoir accès.

* * * * *

Codage non sécurisé
---------------

Les pratiques de codage non sécurisées provoquent l'autre type de vulnérabilités de falsification de verbe HTTP (bien que certains puissent ne pas considérer cette falsification de verbe). Cela peut se produire lorsqu'un développeur Web applique des filtres spécifiques pour atténuer des vulnérabilités particulières sans couvrir toutes les méthodes HTTP avec ce filtre. Par exemple, si une page Web s'est avérée vulnérable à une vulnérabilité d'injection SQL et que le développeur principal a atténué la vulnérabilité d'injection SQL en appliquant les filtres de nettoyage d'entrée suivants :

Code : php

```
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}

```

Nous pouvons voir que le filtre de nettoyage n'est testé que sur le paramètre `GET`. Si les requêtes GET ne contiennent aucun caractère incorrect, la requête sera exécutée. Cependant, lorsque la requête est exécutée, les paramètres `$_REQUEST["code"]` sont utilisés, qui peuvent également contenir des paramètres `POST` , `entraînant une incohérence dans l'utilisation des verbes HTTP`. Dans ce cas, un attaquant peut utiliser une requête `POST` pour effectuer une injection SQL, auquel cas les paramètres `GET` seraient vides.(n'inclura aucun mauvais caractère). La requête passerait le filtre de sécurité, ce qui rendrait la fonction toujours vulnérable à l'injection SQL.

Bien que les deux vulnérabilités ci-dessus soient découvertes en public, la seconde est beaucoup plus courante, car elle est due à des erreurs de codage, tandis que la première est généralement évitée par les configurations de serveur Web sécurisé, car la documentation le met souvent en garde. Dans les prochaines sections, nous verrons des exemples des deux types et comment les exploiter.
