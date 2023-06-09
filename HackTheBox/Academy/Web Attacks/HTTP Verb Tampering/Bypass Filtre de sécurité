Contournement des filtres de sécurité
==========================

* * * * *

L'autre type de vulnérabilité HTTP Verb Tampering, le plus courant, est causé par des erreurs de "codage non sécurisé" commises lors du développement de l'application Web, qui conduisent à ce que l'application Web ne couvre pas toutes les méthodes HTTP dans certaines fonctionnalités. Cela se trouve couramment dans les filtres de sécurité qui détectent les requêtes malveillantes. Par exemple, si un filtre de sécurité était utilisé pour détecter les vulnérabilités d'injection et ne vérifiait les injections que dans les paramètres `POST` (par exemple `$_POST['parameter']`), il peut être possible de le contourner en modifiant simplement la méthode de requête. à `OBTENIR`.

* * * * *

Identité
--------

Dans l'application Web `File Manager` , si nous essayons de créer un nouveau nom de fichier avec des caractères spéciaux dans son nom (par exemple `test;`), nous obtenons le message suivant :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_malicious_request.jpg)

Ce message indique que l'application Web utilise certains filtres sur le back-end pour identifier les tentatives d'injection, puis bloque toutes les requêtes malveillantes. Peu importe ce que nous essayons, l'application Web bloque correctement nos requêtes et est sécurisée contre les tentatives d'injection. Cependant, nous pouvons essayer une attaque HTTP Verb Tampering pour voir si nous pouvons complètement contourner le filtre de sécurité.

* * * * *

Exploiter
-------

Pour essayer d'exploiter cette vulnérabilité, interceptons la requête dans Burp Suite (Burp), puis utilisons `Change Request Method` pour la remplacer par une autre méthode : ![unauthorized_request](https://academy.hackthebox.com/storage/modules /134/web_attacks_verb_tampering_GET_request.jpg)

Cette fois, nous n'avons pas reçu le message "Demande malveillante refusée !" et notre fichier a bien été créé :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_injected_request.jpg)

Pour confirmer si nous avons contourné le filtre de sécurité, nous devons tenter d'exploiter la vulnérabilité que le filtre protège : une vulnérabilité d'injection de commande, dans ce cas. Ainsi, nous pouvons injecter une commande qui crée deux fichiers, puis vérifier si les deux fichiers ont été créés. Pour ce faire, nous utiliserons le nom de fichier suivant dans notre attaque (`file1; touch file2;`):

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_filter_bypass.jpg)

Ensuite, nous pouvons à nouveau remplacer la méthode de requête par une requête `GET` : ![filter_bypass_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_filter_bypass_request.jpg)

Une fois que nous avons envoyé notre requête, nous constatons que cette fois, `file1` et `file2` ont été créés :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_after_filter_bypass.jpg)

Cela montre que nous avons réussi à contourner le filtre via une vulnérabilité HTTP Verb Tampering et à réaliser l'injection de commande. Sans la vulnérabilité HTTP Verb Tampering, l'application Web aurait peut-être été sécurisée contre les attaques par injection de commande, et cette vulnérabilité nous a permis de contourner complètement les filtres en place.
