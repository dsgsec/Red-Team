Drupal - Découverte & Énumération
================================

* * * * *

[Drupal](https://www.drupal.org/), lancé en 2001, est le troisième et dernier CMS que nous aborderons lors de notre tour du monde des applications courantes. Drupal est un autre CMS open source populaire parmi les entreprises et les développeurs. Drupal est écrit en PHP et prend en charge l'utilisation de MySQL ou PostgreSQL pour le backend. De plus, SQLite peut être utilisé s'il n'y a pas de SGBD installé. Comme WordPress, Drupal permet aux utilisateurs d'améliorer leurs sites Web grâce à l'utilisation de thèmes et de modules. Au moment de la rédaction, le projet Drupal compte près de 43 000 modules et 2 900 thèmes et est le troisième CMS le plus populaire en termes de part de marché. Voici quelques [statistiques](https://websitebuilder.org/blog/drupal-statistics/) intéressantes sur Drupal recueillies auprès de diverses sources :

- Environ 1,5 % des sites sur Internet exécutent Drupal (plus de 1,1 million de sites !), 5 % des 1 millions de sites Web les plus populaires sur Internet et 7 % des 10 000 sites les plus importants.
- Drupal représente environ 2,4% du marché des CMS
- Il est disponible en 100 langues
- Drupal est axé sur la communauté et compte plus de 1,3 million de membres
- Drupal 8 a été construit par 3 290 contributeurs, 1 288 entreprises et l'aide de la communauté
- 33 des entreprises du Fortune 500 utilisent Drupal d'une manière ou d'une autre
- 56% des sites gouvernementaux à travers le monde utilisent Drupal
- 23,8 % des universités, collèges et écoles utilisent Drupal dans le monde
- Certaines grandes marques qui utilisent Drupal incluent : Tesla et Warner Bros Records

Selon le [site Web] Drupal (https://www.drupal.org/project/usage/drupal), il n'y a qu'environ 950 000 instances de Drupal utilisées au moment de la rédaction (distribuées de la version 5.x à la version 9.3. x, au 5 septembre 2021). Comme nous pouvons le voir sur ces statistiques, l'utilisation de Drupal s'est maintenue régulièrement entre 900 000 et 1,1 million d'instances entre juin 2013 et septembre 2021. Ces statistiques ne tiennent pas compte de `TOUTES` instances de Drupal utilisées dans le monde, mais plutôt des instances exécutant le [État de la mise à jour ](https://www.drupal.org/project/update_status) module, qui se connecte quotidiennement à drupal.org pour rechercher les nouvelles versions de Drupal ou les mises à jour des modules en cours d'utilisation.

* * * * *

Découverte/Empreinte
----------------------

Lors d'un test d'intrusion externe, nous rencontrons ce qui semble être un CMS, mais nous savons d'après un examen rapide que le site n'exécute pas WordPress ou Joomla. Nous savons que les CMS sont souvent des cibles "juteuses", alors approfondissons celle-ci et voyons ce que nous pouvons découvrir.

Un site Web Drupal peut être identifié de plusieurs manières, notamment par le message d'en-tête ou de pied de page `Powered by Drupal`, le logo standard Drupal, la présence d'un fichier `CHANGELOG.txt` ou d'un fichier `README.txt`, via la source de la page. , ou des indices dans le fichier robots.txt tels que des références à `/node`.

```
dsgsec@htb[/htb]$ curl -s http://drupal.inlanefreight.local | grep Drupal

<meta name="Générateur" content="Drupal 8 (https://www.drupal.org)" />
       <span>Propulsé par <a href="https://www.drupal.org">Drupal</a></span>

```

Une autre façon d'identifier Drupal CMS consiste à utiliser [nodes](https://www.drupal.org/docs/8/core/modules/node/about-nodes). Drupal indexe son contenu à l'aide de nœuds. Un nœud peut contenir n'importe quoi, comme un article de blog, un sondage, un article, etc. Les URI de page sont généralement de la forme `/node/<nodeid>`.

![](https://academy.hackthebox.com/storage/modules/113/drupal_node.png)

Par exemple, l'article de blog ci-dessus se trouve à `/node/1`. Cette représentation est utile pour identifier un site Web Drupal lorsqu'un thème personnalisé est utilisé.

Remarque : toutes les installations de Drupal ne se ressemblent pas, n'affichent pas la page de connexion ou ne permettent même pas aux utilisateurs d'accéder à la page de connexion à partir d'Internet.

Drupal prend en charge trois types d'utilisateurs par défaut :

1. "Administrateur" : cet utilisateur a un contrôle total sur le site Web Drupal.
2. "Utilisateur authentifié": Ces utilisateurs peuvent se connecter au site Web et effectuer des opérations telles que l'ajout et la modification d'articles en fonction de leurs autorisations.
3. « Anonyme » : tous les visiteurs du site Web sont désignés comme anonymes. Par défaut, ces utilisateurs ne sont autorisés qu'à lire les messages.

* * * * *

Énumération
-----------

Une fois que nous avons découvert une instance Drupal, nous pouvons combiner une énumération manuelle et basée sur des outils (automatisée) pour découvrir la version, les plugins installés, etc. En fonction de la version de Drupal et des mesures de renforcement mises en place, nous devrons peut-être essayer plusieurs façons d'identifier le numéro de version. Les nouvelles installations de Drupal bloquent par défaut l'accès aux fichiers `CHANGELOG.txt` et `README.txt` , nous devrons donc peut-être effectuer une énumération supplémentaire. Examinons un exemple d'énumération du numéro de version à l'aide du fichier `CHANGELOG.txt` . Pour ce faire, nous pouvons utiliser `cURL` avec `grep`, `sed`, `head`, etc.

```
dsgsec@htb[/htb]$ curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""

Drupal 7.57, 2018-02-21

```

Ici, nous avons identifié une ancienne version de Drupal en cours d'utilisation. En essayant cela par rapport à la dernière version de Drupal au moment de la rédaction, nous obtenons une réponse 404.```
dsgsec@htb[/htb]$ curl -s http://drupal.inlanefreight.local/CHANGELOG.txt

<!DOCTYPE html><html><head><title>404 Not Found</title></head><body><h1>Not Found</h1><p>L'URL demandée "http://drupal. inlanefreight.local/CHANGELOG.txt" n'a pas été trouvé sur ce serveur.</p></body></html>

```

Il y a plusieurs autres choses que nous pourrions vérifier dans ce cas pour identifier la version. Essayons une analyse avec `droopescan` comme indiqué dans la section d'énumération Joomla. `Dropescan` a beaucoup plus de fonctionnalités pour Drupal que pour Joomla.

Exécutons une analyse sur l'hôte `http://drupal.inlanefreight.local` .

```
dsgsec@htb[/htb]$ droopescan scan drupal -u http://drupal.inlanefreight.local

[+] Plugins trouvés :
     php http://drupal.inlanefreight.local/modules/php/
         http://drupal.inlanefreight.local/modules/php/LICENSE.txt

[+] Aucun thème trouvé.

[+] Version(s) possible(s) :
     8.9.0
     8.9.1

[+] Possibles URL intéressantes trouvées :
     Administrateur par défaut - http://drupal.inlanefreight.local/user/login

[+] Numérisation terminée (0:03:19.199526 écoulé)

```

Cette instance semble exécuter la version `8.9.1` de Drupal. Au moment de la rédaction de cet article, ce n'était pas le dernier car il a été publié en juin 2020. Une recherche rapide des [vulnérabilités] liées à Drupal(https://www.cvedetails.com/vulnerability-list/vendor_id-1367/product_id- 2387/Drupal-Drupal.html) ne montre rien d'apparent pour cette version principale de Drupal. Dans ce cas, nous voudrions ensuite examiner les plugins installés ou abuser des fonctionnalités intégrées.