Introduction aux injections de commandes
===========================

* * * * *

Une vulnérabilité d'injection de commande fait partie des types de vulnérabilités les plus critiques. Cela nous permet d'exécuter des commandes système directement sur le serveur d'hébergement principal, ce qui pourrait compromettre l'ensemble du réseau. Si une application Web utilise une entrée contrôlée par l'utilisateur pour exécuter une commande système sur le serveur principal afin de récupérer et de renvoyer une sortie spécifique, nous pouvons être en mesure d'injecter une charge utile malveillante pour subvertir la commande prévue et exécuter nos commandes.

* * * * *

Que sont les injections
-------------------

Les vulnérabilités d'injection sont considérées comme le risque numéro 3 dans [les 10 principaux risques des applications Web de l'OWASP](https://owasp.org/www-project-top-ten/), compte tenu de leur impact élevé et de leur fréquence. L'injection se produit lorsque l'entrée contrôlée par l'utilisateur est mal interprétée dans le cadre de la requête Web ou du code en cours d'exécution, ce qui peut conduire à subvertir le résultat prévu de la requête en un résultat différent utile à l'attaquant.

Il existe de nombreux types d'injections dans les applications Web, en fonction du type de requête Web en cours d'exécution. Voici quelques-uns des types d'injections les plus courants :

| Injection | Descriptif |
| --- | --- |
| Injection de commande du système d'exploitation | Se produit lorsque l'entrée de l'utilisateur est directement utilisée dans le cadre d'une commande du système d'exploitation. |
| Injection de code | Se produit lorsque l'entrée de l'utilisateur se trouve directement dans une fonction qui évalue le code. |
| Injections SQL | Se produit lorsque l'entrée utilisateur est directement utilisée dans le cadre d'une requête SQL. |
| Script intersite/Injection HTML | Se produit lorsque l'entrée exacte de l'utilisateur est affichée sur une page Web. |

Il existe de nombreux autres types d'injections autres que celles décrites ci-dessus, telles que `injection LDAP`, `injection NoSQL`, `injection d'en-tête HTTP`, `injection XPath`, `injection IMAP`, `injection ORM`, et autres. Chaque fois que l'entrée de l'utilisateur est utilisée dans une requête sans être correctement filtrée, il peut être possible d'échapper aux limites de la chaîne d'entrée de l'utilisateur vers la requête parente et de la manipuler pour modifier son objectif. C'est pourquoi, à mesure que de plus en plus de technologies Web sont introduites dans les applications Web, nous verrons de nouveaux types d'injections introduites dans les applications Web.

* * * * *

Injections de commandes du système d'exploitation
---------------------

En ce qui concerne les injections de commandes du système d'exploitation, l'entrée de l'utilisateur que nous contrôlons doit entrer directement ou indirectement dans (ou affecter d'une manière ou d'une autre) une requête Web qui exécute des commandes système. Tous les langages de programmation Web ont des fonctions différentes qui permettent au développeur d'exécuter des commandes du système d'exploitation directement sur le serveur principal chaque fois qu'il en a besoin. Cela peut être utilisé à diverses fins, comme l'installation de plugins ou l'exécution de certaines applications.

#### Exemple PHP

Par exemple, une application Web écrite en `PHP` peut utiliser les fonctions `exec`, `system`, `shell_exec`, `passthru` ou `popen` pour exécuter des commandes directement sur le serveur principal, chacune ayant un léger cas d'utilisation différent. Le code suivant est un exemple de code PHP vulnérable aux injections de commandes :

Code : php

```
<?php
si (isset($_GET['nomfichier'])) {
     system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>

```

Peut-être qu'une application Web particulière a une fonctionnalité qui permet aux utilisateurs de créer un nouveau document `.pdf` qui est créé dans le répertoire `/tmp` avec un nom de fichier fourni par l'utilisateur et peut ensuite être utilisé par l'application Web pour le traitement du document. fins. Cependant, comme l'entrée utilisateur du paramètre `filename` dans la requête `GET` est utilisée directement avec la commande `touch` (sans être préalablement nettoyée ou échappée), l'application Web devient vulnérable à l'injection de commande du système d'exploitation. Cette faille peut être exploitée pour exécuter des commandes système arbitraires sur le serveur principal.

#### Exemple NodeJS

Ce n'est pas propre à `PHP` , mais peut se produire dans n'importe quel framework ou langage de développement Web. Par exemple, si une application Web est développée dans `NodeJS`, un développeur peut utiliser `child_process.exec` ou `child_process.spawn` dans le même but. L'exemple suivant exécute une fonctionnalité similaire à celle dont nous avons parlé ci-dessus :

Code : javascript

```
app.get("/createfile", function(req, res){
     child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})

```

Le code ci-dessus est également vulnérable à une vulnérabilité d'injection de commande, car il utilise le paramètre `filename` de la requête `GET` dans le cadre de la commande sans le nettoyer au préalable. Les applications Web `PHP` et `NodeJS` peuvent être exploitées à l'aide des mêmes méthodes d'injection de commande.

De même, d'autres langages de programmation de développement Web ont des fonctions similaires utilisées aux mêmes fins et, s'ils sont vulnérables, peuvent être exploités en utilisant les mêmes méthodes d'injection de commandes. De plus, les vulnérabilités d'injection de commande ne sont pas uniques aux applications Web, mais peuvent également affecter d'autres fichiers binaires et clients lourds s'ils transmettent une entrée utilisateur non filtrée à une fonction qui exécute des commandes système, qui peuvent également être exploitées avec les mêmes méthodes d'injection de commande.

La section suivante traitera des différentes méthodes de détection et d'exploitation des injections de commandes.sur les vulnérabilités des applications Web.