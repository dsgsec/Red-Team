Énumération IDOR de masse
=====================

* * * * *

L'exploitation des vulnérabilités IDOR est facile dans certains cas, mais peut être très difficile dans d'autres. Une fois que nous avons identifié un IDOR potentiel, nous pouvons commencer à le tester avec des techniques de base pour voir s'il exposerait d'autres données. En ce qui concerne les attaques IDOR avancées, nous devons mieux comprendre comment fonctionne l'application Web, comment elle calcule ses références d'objets et comment fonctionne son système de contrôle d'accès pour pouvoir effectuer des attaques avancées qui peuvent ne pas être exploitables avec des techniques de base.

Commençons par discuter des diverses techniques d'exploitation des vulnérabilités IDOR, de l'énumération de base à la collecte de masse de données, en passant par l'élévation des privilèges des utilisateurs.

* * * * *

Paramètres non sécurisés
-------------------

Commençons par un exemple de base qui présente une vulnérabilité IDOR typique. L'exercice ci-dessous est une application Web `Employee Manager` qui héberge les dossiers des employés :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_employee_manager.jpg)

Notre application Web suppose que nous sommes connectés en tant qu'employé avec l'identifiant `uid=1` pour simplifier les choses. Cela nous obligerait à nous connecter avec des informations d'identification dans une application Web réelle, mais le reste de l'attaque serait le même. Une fois que nous avons cliqué sur `Documents`, nous sommes redirigés vers

`/documents.php` :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_documents.jpg)

Lorsque nous arrivons à la page `Documents` , nous voyons plusieurs documents appartenant à notre utilisateur. Il peut s'agir de fichiers téléchargés par notre utilisateur ou de fichiers définis pour nous par un autre service (par exemple, le service des ressources humaines). En vérifiant les liens de fichiers, nous voyons qu'ils ont des noms individuels :

Code : html

```
/documents/Facture_1_09_2021.pdf
/documents/Rapport_1_10_2021.pdf

```

Nous constatons que les fichiers ont un modèle de dénomination prévisible, car les noms de fichiers semblent utiliser l'utilisateur `uid` et le mois/l'année dans le nom de fichier, ce qui peut nous permettre de fuzzer des fichiers pour d'autres utilisateurs. Il s'agit du type de vulnérabilité IDOR le plus élémentaire. Il s'appelle "IDOR de fichier statique". Cependant, pour fuzzer avec succès d'autres fichiers, nous supposons qu'ils commencent tous par `Invoice` ou `Report`, ce qui peut révéler certains fichiers, mais pas tous. Cherchons donc une vulnérabilité IDOR plus solide.

Nous voyons que la page définit notre `uid` avec un paramètre `GET` dans l'URL comme (`documents.php?uid=1`). Si l'application Web utilise ce paramètre GET `uid` comme référence directe aux enregistrements des employés qu'elle doit afficher, nous pourrons peut-être afficher les documents d'autres employés en modifiant simplement cette valeur. Si le back-end de l'application Web dispose d'un système de contrôle d'accès approprié, nous obtiendrons une forme d'"Accès refusé". Cependant, étant donné que l'application Web passe pour notre `uid` en texte clair comme référence directe, cela peut indiquer une mauvaise conception de l'application Web, entraînant un accès arbitraire aux dossiers des employés.

Lorsque nous essayons de changer le `uid` en `?uid=2`, nous ne remarquons aucune différence dans la sortie de la page, car nous obtenons toujours la même liste de documents et pouvons supposer qu'elle renvoie toujours nos propres documents :

![](https://academy.hackthebox.com/storage/modules/134/web_attacks_idor_documents.jpg)

Cependant, `nous devons être attentifs aux détails de la page lors de tout pentest Web` et toujours garder un œil sur le code source et la taille de la page. Si nous regardons les fichiers liés, ou si nous cliquons dessus pour les visualiser, nous remarquerons qu'il s'agit bien de fichiers différents, qui semblent être les documents appartenant à l'employé avec `uid=2` :

Code : html

```
/documents/Facture_2_08_2020.pdf
/documents/Rapport_2_12_2020.pdf

```

Il s'agit d'une erreur courante dans les applications Web souffrant de vulnérabilités IDOR, car elles placent le paramètre qui contrôle les documents utilisateur à afficher sous notre contrôle tout en n'ayant aucun système de contrôle d'accès sur le back-end. Un autre exemple consiste à utiliser un paramètre de filtre pour afficher uniquement les documents d'un utilisateur spécifique (par exemple `uid_filter=1`), qui peut également être manipulé pour afficher les documents d'autres utilisateurs ou même complètement supprimé pour afficher tous les documents à la fois.

* * * * *

Dénombrement de masse
----------------

Nous pouvons essayer d'accéder manuellement aux documents d'autres employés avec `uid=3`, `uid=4`, etc. Cependant, l'accès manuel aux fichiers n'est pas efficace dans un environnement de travail réel avec des centaines ou des milliers d'employés. Ainsi, nous pouvons soit utiliser un outil comme `Burp Intruder` ou `ZAP Fuzzer` pour récupérer tous les fichiers, soit écrire un petit script bash pour télécharger tous les fichiers, ce que nous ferons.

Nous pouvons cliquer sur [`CTRL+SHIFT+C`] dans Firefox pour activer `l'inspecteur d'éléments`, puis cliquer sur l'un des liens pour afficher leur code source HTML, et nous obtiendrons ce qui suit :

Code : html

```
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Facture</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Rapport</a></li>

```

Nous pouvons choisir n'importe quel mot unique pour pouvoir `grep` le lien du fichier. Dans notre cas, nous voyons que chaque lien commence par `<li class='pure-tree_link'> `, nous pouvons donc `curl` la page et `grep` pour cette ligne, comme suit :

```
dsgsec@htb[/htb]$ curl -s "http://SERVER_IP:PORT/documents.php?uid=1" | grep "<li class='pure-tree_link'>"

<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Facture</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Rapport</a></li>

```

Comme nous pouvons le voir, nous avons réussi à capturer les liens des documents avec succès. Nous pouvons maintenant utiliser des commandes bash spécifiques pour couper les parties supplémentaires et obtenir uniquement les liens de document dans la sortie. Cependant, il est préférable d'utiliser un modèle `Regex` qui fait correspondre les chaînes entre `/document` et `.pdf`, que nous pouvons utiliser avec `grep` pour obtenir uniquement les liens vers les documents, comme suit :

```
dsgsec@htb[/htb]$ curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"

/documents/Facture_3_06_2020.pdf
/documents/Rapport_3_01_2020.pdf

```

Maintenant, nous pouvons utiliser une simple boucle `for` pour parcourir le paramètre `uid` et renvoyer le document de tous les employés, puis utiliser `wget` pour télécharger chaque lien de document :

Code : bash

```
#!/bin/bash

url="http://SERVER_IP:PORT"

pour je dans {1..10} ; faire
         pour le lien dans $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); faire
                 wget -q $url/$lien
         fait
fait

```

Lorsque nous exécutons le script, il télécharge tous les documents de tous les employés avec `uid` entre 1 et 10, exploitant ainsi avec succès la vulnérabilité IDOR pour énumérer en masse les documents de tous les employés. Ce script est un exemple de la façon dont nous pouvons atteindre le même objectif. Essayez d'utiliser un outil comme Burp Intruder ou ZAP Fuzzer, ou écrivez un autre script Bash ou PowerShell pour télécharger tous les documents.
