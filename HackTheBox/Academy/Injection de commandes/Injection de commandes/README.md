Injecter des commandes
==================

* * * * *

Jusqu'à présent, nous avons constaté que l'application Web `Host Checker` était potentiellement vulnérable aux injections de commandes et avons discuté des différentes méthodes d'injection que nous pouvons utiliser pour exploiter l'application Web. Alors, commençons nos tentatives d'injection de commande avec l'opérateur point-virgule (`;`).

* * * * *

Injecter notre commande
---------------------

Nous pouvons ajouter un point-virgule après notre entrée IP `127.0.0.1`, puis ajouter notre commande (par exemple `whoami`), de sorte que la charge utile finale que nous utiliserons est (`127.0.0.1; whoami`), et le la commande finale à exécuter serait :

Code : bash

```
ping -c 1 127.0.0.1; whoami

```

Essayons d'abord d'exécuter la commande ci-dessus sur notre machine virtuelle Linux pour nous assurer qu'elle fonctionne :

```
21y4d@htb[/htb]$ ping -c 1 127.0.0.1; whoami

PING 127.0.0.1 (127.0.0.1) 56(84) octets de données.
64 octets à partir de 127.0.0.1 : icmp_seq=1 ttl=64 time=1,03 ms

--- 127.0.0.1 statistiques de ping ---
1 paquets transmis, 1 reçu, 0% de perte de paquets, temps 0ms
rtt min/moy/max/mdev = 1,034/1,034/1,034/0,000 ms
21a4j

```

Comme nous pouvons le voir, la commande finale s'exécute avec succès et nous obtenons la sortie des deux commandes (comme mentionné dans le tableau précédent pour `;`). Maintenant, nous pouvons essayer d'utiliser notre charge utile précédente dans l'application Web `Host Checker`  : ![Basic Injection](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_injection.jpg)

Comme nous pouvons le voir, l'application Web a refusé notre entrée, car elle semble n'accepter que les entrées au format IP. Cependant, d'après l'apparence du message d'erreur, il semble provenir du front-end plutôt que du back-end. Nous pouvons revérifier cela avec `Firefox Developer Tools` en cliquant sur `[CTRL + SHIFT + E]` pour afficher l'onglet Réseau, puis en cliquant à nouveau sur le bouton `Vérifier`  :

![Injection de base](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_injection_network.jpg)

Comme nous pouvons le constater, aucune nouvelle demande de réseau n'a été effectuée lorsque nous avons cliqué sur le bouton `Vérifier` , mais nous avons reçu un message d'erreur. Cela indique que `la validation des entrées utilisateur se produit sur le front-end`.

Cela semble être une tentative pour nous empêcher d'envoyer des charges utiles malveillantes en autorisant uniquement la saisie de l'utilisateur dans un format IP. "Cependant, il est très courant que les développeurs n'effectuent la validation des entrées que sur le front-end sans valider ni nettoyer les entrées sur le back-end." Cela se produit pour diverses raisons, comme le fait que deux équipes différentes travaillent sur le front-end. /back-end ou faire confiance à la validation frontale pour empêcher les charges utiles malveillantes.

Cependant, comme nous le verrons, les validations frontales ne suffisent généralement pas à empêcher les injections, car elles peuvent être très facilement contournées en envoyant des requêtes HTTP personnalisées directement au back-end.

* * * * *

Contournement de la validation frontale
------------------------------

La méthode la plus simple pour personnaliser les requêtes HTTP envoyées au serveur principal consiste à utiliser un proxy Web capable d'intercepter les requêtes HTTP envoyées par l'application. Pour ce faire, nous pouvons démarrer `Burp Suite` ou `ZAP` et configurer Firefox pour qu'il transmette le trafic par proxy. Ensuite, nous pouvons activer la fonction d'interception de proxy, envoyer une requête standard à partir de l'application Web avec n'importe quelle adresse IP (par exemple `127.0.0.1`) et envoyer la requête HTTP interceptée à `repeater` en cliquant `[CTRL + R]`, et nous devrions avoir la requête HTTP pour la personnalisation :

#### Burp POST Demande

![Injection de base](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_repeater_1.jpg)

Nous pouvons maintenant personnaliser notre requête HTTP et l'envoyer pour voir comment l'application Web la gère. Nous allons commencer par utiliser la même charge utile précédente (`127.0.0.1; whoami`). Nous devons également coder en URL notre charge utile pour nous assurer qu'elle est envoyée comme nous l'avons prévu. Nous pouvons le faire en sélectionnant la charge utile, puis en cliquant `[CTRL + U]`. Enfin, nous pouvons cliquer sur `Envoyer` pour envoyer notre requête HTTP :

#### Burp POST Demande

![Injection de base](https://academy.hackthebox.com/storage/modules/109/cmdinj_basic_repeater_2.jpg)

Comme nous pouvons le voir, la réponse que nous avons cette fois-ci contient la sortie de la commande `ping` et le résultat de la commande `whoami` , `ce qui signifie que nous avons réussi à injecter notre nouvelle commande`.