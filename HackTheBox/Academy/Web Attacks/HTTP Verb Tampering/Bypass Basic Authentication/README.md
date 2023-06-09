Contournement de l'authentification de base
==============================

* * * * *

L'exploitation des vulnérabilités HTTP Verb Tampering est généralement un processus relativement simple. Nous avons juste besoin d'essayer d'autres méthodes HTTP pour voir comment elles sont gérées par le serveur Web et l'application Web. Alors que de nombreux outils d'analyse de vulnérabilité automatisés peuvent identifier de manière cohérente les vulnérabilités HTTP Verb Tampering causées par des configurations de serveur non sécurisées, ils manquent généralement d'identifier les vulnérabilités HTTP Tampering causées par un codage non sécurisé. En effet, le premier type peut être facilement identifié une fois que nous contournons une page d'authentification, tandis que l'autre nécessite des tests actifs pour voir si nous pouvons contourner les filtres de sécurité en place.

Le premier type de vulnérabilité HTTP Verb Tampering est principalement causé par `Configurations de serveur Web non sécurisées`, et l'exploitation de cette vulnérabilité peut nous permettre de contourner l'invite HTTP Basic Authentication sur certaines pages.

* * * * *

Identité
--------

Lorsque nous commençons l'exercice à la fin de cette section, nous constatons que nous disposons d'une application Web `File Manager` de base, dans laquelle nous pouvons ajouter de nouveaux fichiers en saisissant leurs noms et en appuyant sur `enter` :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_add.jpg)

Cependant, supposons que nous essayions de supprimer tous les fichiers en cliquant sur le bouton rouge `Réinitialiser` . Dans ce cas, nous constatons que cette fonctionnalité semble être limitée aux utilisateurs authentifiés uniquement, car nous obtenons l'invite `HTTP Basic Auth` :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_reset.jpg)

Comme nous n'avons pas d'informations d'identification, nous obtiendrons une page `401 non autorisé`  :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_unauthorized.jpg)

Voyons donc si nous pouvons contourner cela avec une attaque HTTP Verb Tampering. Pour ce faire, nous devons identifier les pages qui sont restreintes par cette authentification. Si nous examinons la requête HTTP après avoir cliqué sur le bouton Réinitialiser ou si nous examinons l'URL vers laquelle le bouton navigue après avoir cliqué dessus, nous constatons qu'elle se trouve à `/admin/reset.php`. Ainsi, soit le répertoire `/admin` est réservé aux utilisateurs authentifiés uniquement, soit seule la page `/admin/reset.php` l'est. Nous pouvons le confirmer en visitant le répertoire `/admin` , et nous sommes en effet invités à nous reconnecter. Cela signifie que le répertoire `/admin` complet est restreint.

* * * * *

Exploiter
-------

Pour essayer d'exploiter la page, nous devons identifier la méthode de requête HTTP utilisée par l'application Web. Nous pouvons intercepter la demande dans Burp Suite et l'examiner : ![unauthorized_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_unauthorized_request.jpg)

Comme la page utilise une requête `GET` , nous pouvons envoyer une requête `POST` et voir si la page Web autorise les requêtes `POST` (c'est-à-dire si l'authentification couvre les requêtes `POST`). Pour ce faire, nous pouvons cliquer avec le bouton droit de la souris sur la demande interceptée dans Burp et sélectionner `Change Request Method`, et cela transformera automatiquement la demande en une demande `POST` : ![change_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_change_request.jpg)

Une fois cela fait, nous pouvons cliquer sur `Suivant` et examiner la page dans notre navigateur. Malheureusement, nous sommes toujours invités à nous connecter et nous recevrons une page `401 Non autorisé` si nous ne fournissons pas les informations d'identification :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_reset.jpg)

Il semble donc que les configurations de serveur Web couvrent à la fois les requêtes `GET` et `POST` . Cependant, comme nous l'avons appris précédemment, nous pouvons utiliser de nombreuses autres méthodes HTTP, notamment la méthode `HEAD` , qui est identique à une requête `GET` mais ne renvoie pas le corps dans la réponse HTTP. Si cela réussit, nous ne recevrons peut-être aucune sortie, mais la fonction `reset` doit toujours être exécutée, ce qui est notre cible principale.

Pour voir si le serveur accepte les requêtes `HEAD` , nous pouvons lui envoyer une requête `OPTIONS` et voir quelles méthodes HTTP sont acceptées, comme suit :

```
dsgsec@htb[/htb]$ curl -i -X OPTIONS http://SERVER_IP:PORT/

HTTP/1.1 200 OK
Date:
Server: Apache/2.4.41 (Ubuntu)
Allow: POST,OPTIONS,HEAD,GET
Content-Length: 0
Content-Type: httpd/unix-directory

```

Comme nous pouvons le voir, la réponse indique `Allow : POST,OPTIONS,HEAD,GET`, ce qui signifie que le serveur Web accepte effectivement les requêtes `HEAD` , qui est la configuration par défaut pour de nombreux serveurs Web. Essayons donc d'intercepter à nouveau la requête `reset` et, cette fois, utilisons une requête `HEAD` pour voir comment le serveur Web la gère :

![HEAD_request](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_HEAD_request.jpg)

Une fois que nous avons remplacé `POST` par `HEAD` et que nous avons transmis la requête, nous constatons que nous n'obtenons plus d'invite de connexion ni de page `401 Unauthorized` et que nous obtenons une sortie vide à la place, comme prévu avec une requête `HEAD` . Si nous revenons à l'application Web `File Manager` , nous verrons que tous les fichiers ont bien été supprimés, ce qui signifie que nous avons déclenché avec succès la fonction `Reset` nalité sans avoir d'accès administrateur ni d'informations d'identification :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_verb_tampering_after_reset.jpg)

Essayez de tester d'autres méthodes HTTP et voyez lesquelles peuvent contourner avec succès l'invite d'authentification.
