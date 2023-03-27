Validation côté client
======================

* * * * *

De nombreuses applications Web s'appuient uniquement sur le code JavaScript frontal pour valider le format de fichier sélectionné avant son téléchargement et ne le téléchargeraient pas si le fichier n'est pas au format requis (par exemple, pas une image).

Cependant, comme la validation du format de fichier se produit côté client, nous pouvons facilement la contourner en interagissant directement avec le serveur, en sautant complètement les validations frontales. Nous pouvons également modifier le code frontal via les outils de développement de notre navigateur pour désactiver toute validation en place.

* * * * *

Validation côté client
----------------------

L'exercice à la fin de cette section présente une fonctionnalité de base d'"image de profil", fréquemment utilisée dans les applications Web qui utilisent des fonctionnalités de profil utilisateur, telles que les applications Web de médias sociaux :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_profile_image_upload.jpg)

Cependant, cette fois, lorsque nous obtenons la boîte de dialogue de sélection de fichier, nous ne pouvons pas voir nos scripts `PHP` (ou ils peuvent être grisés), car la boîte de dialogue semble être limitée aux formats d'image uniquement :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_select_file_types.jpg)

Nous pouvons quand même sélectionner l'option `Tous les fichiers` pour sélectionner notre script `PHP` , mais lorsque nous le faisons, nous recevons un message d'erreur indiquant ("Seules les images sont autorisées !"), et le bouton `Télécharger` est désactivé :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_select_denied.jpg)

Cela indique une certaine forme de validation de type de fichier, nous ne pouvons donc pas simplement télécharger un shell Web via le formulaire de téléchargement comme nous l'avons fait dans la section précédente. Heureusement, toutes les validations semblent se produire sur le front-end, car la page ne s'actualise jamais et n'envoie aucune requête HTTP après avoir sélectionné notre fichier. Nous devrions donc pouvoir avoir un contrôle total sur ces validations côté client.

Tout code qui s'exécute côté client est sous notre contrôle. Alors que le serveur Web est responsable de l'envoi du code frontal, le rendu et l'exécution du code frontal se produisent dans notre navigateur. Si l'application Web n'applique aucune de ces validations sur le back-end, nous devrions pouvoir télécharger n'importe quel type de fichier.

Comme mentionné précédemment, pour contourner ces protections, nous pouvons soit "modifier la demande de téléchargement vers le serveur principal", soit "manipuler le code frontal pour désactiver ces validations de type".

* * * * *

Modification de la demande principale
-----------------------------

Commençons par examiner une requête normale via `Burp`. Lorsque nous sélectionnons une image, nous voyons qu'elle est reflétée comme notre image de profil, et lorsque nous cliquons sur `Télécharger`, notre image de profil est mise à jour et persiste lors des actualisations. Cela indique que notre image a été téléchargée sur le serveur, qui nous l'affiche maintenant :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_normal_request.jpg)

Si nous capturons la demande d'importation avec `Burp`, nous voyons la demande suivante envoyée par l'application Web :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_image_upload_request.jpg)

L'application Web semble envoyer une demande de téléchargement HTTP standard à `/upload.php`. De cette façon, nous pouvons maintenant modifier cette requête pour répondre à nos besoins sans avoir les restrictions de validation de type frontal. Si le serveur principal ne valide pas le type de fichier téléchargé, nous devrions théoriquement être en mesure d'envoyer n'importe quel type de fichier/contenu, et il serait téléchargé sur le serveur.

Les deux parties importantes de la requête sont `filename="HTB.png"` et le contenu du fichier à la fin de la requête. Si nous modifions le `filename` en `shell.php` et modifions le contenu du shell Web que nous avons utilisé dans la section précédente ; nous importerions un shell Web `PHP` au lieu d'une image.

Alors, capturons une autre demande de téléchargement d'image, puis modifions-la en conséquence :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_modified_upload_request.jpg)

Remarque : Nous pouvons également modifier le `Content-Type` du fichier téléchargé, bien que cela ne devrait pas jouer un rôle important à ce stade, nous le garderons donc tel quel.

Comme nous pouvons le voir, notre demande de téléchargement a abouti, et nous avons reçu `Fichier téléchargé avec succès` dans la réponse. Ainsi, nous pouvons maintenant visiter notre fichier téléchargé et interagir avec lui et obtenir l'exécution de code à distance.

* * * * *

Désactivation de la validation frontale
------------------------------

Une autre méthode pour contourner les validations côté client consiste à manipuler le code frontal. Comme ces fonctions sont entièrement traitées dans notre navigateur Web, nous avons un contrôle total sur elles. Ainsi, nous pouvons modifier ces scripts ou les désactiver entièrement. Ensuite, nous pouvons utiliser la fonctionnalité de téléchargement pour télécharger n'importe quel type de fichier sans avoir besoin d'utiliser `Burp` pour capturer et modifier nos demandes.

Pour commencer, nous pouvons cliquer sur [`CTRL+SHIFT+C`] pour basculer l'inspecteur de page du navigateur, puis cliquer sur l'image de profil, où nous déclenchons le sélecteur de fichier pour le formulaire de téléchargement :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_element_inspector.jpg)

Cela mettra en surbrillance l'entrée de fichier HTML suivante à la ligne `18` :

Code : html

```
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">

```

Ici, nous voyons que l'entrée de fichier spécifie (`.jpg,.jpeg,.png`) comme types de fichiers autorisés dans la boîte de dialogue de sélection de fichiers. Cependant, nous pouvons facilement modifier cela et sélectionner `Tous les fichiers` comme nous l'avons fait auparavant, il est donc inutile de modifier cette partie de la page.

La partie la plus intéressante est `onchange="checkFile(this)"`, qui semble exécuter un code JavaScript chaque fois que nous sélectionnons un fichier, qui semble effectuer la validation du type de fichier. Pour obtenir les détails de cette fonction, nous pouvons accéder à la `Console` du navigateur en cliquant sur [`CTRL+SHIFT+K`], puis saisir le nom de la fonction (`checkFile`) pour obtenir ses détails :

Code : javascript

```
fonction checkFichier(Fichier) {
...COUPER...
     si (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
         $('#error_message').text("Seules les images sont autorisées !");
         Fichier.formulaire.reset();
         $("#submit").attr("désactivé", vrai);
     ...COUPER...
     }
}

```

L'élément clé que nous retirons de cette fonction est l'endroit où elle vérifie si l'extension de fichier est une image, et si ce n'est pas le cas, elle imprime le message d'erreur que nous avons vu précédemment ("Seules les images sont autorisées !") et désactive le bouton "Télécharger". . Nous pouvons ajouter `PHP` comme l'une des extensions autorisées ou modifier la fonction pour supprimer la vérification des extensions.

Heureusement, nous n'avons pas besoin de nous lancer dans l'écriture et la modification du code JavaScript. Nous pouvons supprimer cette fonction du code HTML car son utilisation principale semble être la validation du type de fichier, et sa suppression ne devrait rien casser.

Pour ce faire, nous pouvons revenir à notre inspecteur, cliquer à nouveau sur l'image du profil, double-cliquer sur le nom de la fonction (`checkFile`) à la ligne `18`, et la supprimer :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_removed_js_function.jpg)

Astuce : Vous pouvez également faire de même pour supprimer `accept=".jpg,.jpeg,.png"`, ce qui devrait faciliter la sélection du shell `PHP` dans la boîte de dialogue de sélection de fichier, bien que ce ne soit pas obligatoire, comme mentionné précédemment. .

Avec la fonction `checkFile` supprimée de l'entrée du fichier, nous devrions être en mesure de sélectionner notre shell Web `PHP` via la boîte de dialogue de sélection de fichier et de le télécharger normalement sans validation, comme nous l'avons fait dans la section précédente.

Remarque : La modification que nous avons apportée au code source est temporaire et ne persistera pas lors des actualisations de page, car nous ne la modifions que côté client. Cependant, notre seul besoin est de contourner la validation côté client, cela devrait donc suffire à cet effet.

Une fois que nous avons téléchargé notre shell Web en utilisant l'une des méthodes ci-dessus, puis actualisé la page, nous pouvons utiliser `Page Inspector` une fois de plus avec [`CTRL + SHIFT + C`], cliquez sur l'image de profil, et nous devrions voir le URL de notre shell Web téléchargé :

Code : html

```
<img src="/profile_images/shell.php" class="profile-image" id="profile-image">

```

Si nous pouvons cliquer sur le lien ci-dessus, nous accéderons à notre shell Web téléchargé, avec lequel nous pouvons interagir pour exécuter des commandes sur le serveur principal :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell.jpg)