ype Filters
===========

* * * * *

So far, we have only been dealing with type filters that only consider the file extension in the file name. However, as we saw in the previous section, we may still be able to gain control over the back-end server even with image extensions (e.g. `shell.php.jpg`). Furthermore, we may utilize some allowed extensions (e.g., SVG) to perform other attacks. All of this indicates that only testing the file extension is not enough to prevent file upload attacks.

This is why many modern web servers and web applications also test the content of the uploaded file to ensure it matches the specified type. While extension filters may accept several extensions, content filters usually specify a single category (e.g., images, videos, documents), which is why they do not typically use blacklists or whitelists. This is because web servers provide functions to check for the file content type, and it usually falls under a specific category.

There are two common methods for validating the file content: `Content-Type Header` or `File Content`. Let's see how we can identify each filter and how to bypass both of them.

* * * * *

Content-Type
------------

Let's start the exercise at the end of this section and attempt to upload a PHP script:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_content_type_upload.jpg)

We see that we get a message saying `Only images are allowed`. The error message persists, and our file fails to upload even if we try some of the tricks we learned in the previous sections. If we change the file name to `shell.jpg.phtml` or `shell.php.jpg`, or even if we use `shell.jpg` with a web shell content, our upload will fail. As the file extension does not affect the error message, the web application must be testing the file content for type validation. As mentioned earlier, this can be either in the `Content-Type Header` or the `File Content`.

The following is an example of how a PHP web application tests the Content-Type header to validate the file type:

Code: php

```
$type = $_FILES['uploadFile']['type'];

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}

```

The code sets the (`$type`) variable from the uploaded file's `Content-Type` header. Our browsers automatically set the Content-Type header when selecting a file through the file selector dialog, usually derived from the file extension. However, since our browsers set this, this operation is a client-side operation, and we can manipulate it to change the perceived file type and potentially bypass the type filter.

We may start by fuzzing the Content-Type header with SecLists' [Content-Type Wordlist](https://github.com/danielmiessler/SecLists/blob/master/Miscellaneous/web/content-type.txt) through Burp Intruder, to see which types are allowed. However, the message tells us that only images are allowed, so we can limit our scan to image types, which reduces the wordlist to `45` types only (compared to around 700 originally). We can do so as follows:

```
dsgsec@htb[/htb]$ wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Miscellaneous/web/content-type.txt
dsgsec@htb[/htb]$ cat content-type.txt | grep 'image/' > image-content-types.txt

```

Exercise: Try to run the above scan to find what Content-Types are allowed.

For the sake of simplicity, let's just pick an image type (e.g. `image/jpg`), then intercept our upload request and change the Content-Type header to it:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_content_type_request.jpg)

This time we get `File successfully uploaded`, and if we visit our file, we see that it was successfully uploaded:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell.jpg)

Note: A file upload HTTP request has two Content-Type headers, one for the attached file (at the bottom), and one for the full request (at the top). We usually need to modify the file's Content-Type header, but in some cases the request will only contain the main Content-Type header (e.g. if the uploaded content was sent as `POST` data), in which case we will need to modify the main Content-Type header.

* * * * *

MIME-Type
---------

The second and more common type of file content validation is testing the uploaded file's `MIME-Type`. `Multipurpose Internet Mail Extensions (MIME)` is an internet standard that determines the type of a file through its general format and bytes structure.

This is usually done by inspecting the first few bytes of the file's content, which contain the [File Signature](https://en.wikipedia.org/wiki/List_of_file_signatures) or [Magic Bytes](https://opensource.apple.com/source/file/file-23/file/magic/magic.mime). For example, if a file starts with (`GIF87a` or `GIF89a`), this indicates that it is a `GIF` image, while a file starting with plaintext is usually considered a `Text` file. If we change the first bytes of any file to the GIF magic bytes, its MIME type would be changed to a GIF image, regardless of its remaining content or extension.

Tip: Many other image types have non-printable bytes for their file signatures, while a `GIF` image starts with ASCII printable bytes (as shown above), so it is the easiest to imitate. Furthermore, as the string `GIF8` is common between both GIF signatures, it is usually enough to imitate a GIF image.

Let's take a basic example to demonstrate this. The `file` command on Unix systems finds the file type through the MIME type. If we create a basic file with text in it, it would be considered as a text file, as follows:

```
dsgsec@htb[/htb]$ echo "this is a text file" > text.jpg
dsgsec@htb[/htb]$ file text.jpg
text.jpg: ASCII text

```

As we see, the file's MIME type is `ASCII text`, even though its extension is `.jpg`. However, if we write `GIF8` to the beginning of the file, it will be considered as a `GIF` image instead, even though its extension is still `.jpg`:

```
dsgsec@htb[/htb]$ echo "GIF8" > text.jpg
dsgsec@htb[/htb]$file text.jpg
text.jpg: GIF image data

```

Web servers can also utilize this standard to determine file types, which is usually more accurate than testing the file extension. The following example shows how a PHP web application can test the MIME type of an uploaded file:

Code: php

```
$type = mime_content_type($_FILES['uploadFile']['tmp_name']);

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}

```

As we can see, the MIME types are similar to the ones found in the Content-Type headers, but their source is different, as PHP uses the `mime_content_type()` function to get a file's MIME type. Let's try to repeat our last attack, but now with an exercise that tests both the Content-Type header and the MIME type:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_content_type_request.jpg)

Once we forward our request, we notice that we get the error message `Only images are allowed`. Now, let's try to add `GIF8` before our PHP code to try to imitate a GIF image while keeping our file extension as `.php`, so it would execute PHP code regardless:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_mime_type_request.jpg)

This time we get `File successfully uploaded`, and our file is successfully uploaded to the server:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_mime_type.jpg)

We can now visit our uploaded file, and we will see that we can successfully execute system commands:

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell_gif.jpg)

Conseil : De nombreux autres types d'images ont des octets non imprimables pour leurs signatures de fichier, tandis qu'une image `GIF` commence par des octets imprimables ASCII (comme indiqué ci-dessus). Elle est donc la plus facile à imiter. De plus, comme la chaîne `GIF8` est commune aux deux signatures GIF, il suffit généralement d'imiter une image GIF.

Prenons un exemple basique pour le démontrer. La commande `file` sur les systèmes Unix trouve le type de fichier via le type MIME. Si nous créons un fichier de base contenant du texte, il sera considéré comme un fichier texte, comme suit :

```
dsgsec@htb[/htb]$ echo "ceci est un fichier texte" > text.jpg
dsgsec@htb[/htb]$ fichier text.jpg
text.jpg : texte ASCII

```

Comme nous le voyons, le type MIME du fichier est `texte ASCII`, même si son extension est `.jpg`. Cependant, si nous écrivons `GIF8` au début du fichier, il sera considéré comme une image `GIF` à la place, même si son extension est toujours `.jpg` :

```
dsgsec@htb[/htb]$ echo "GIF8" > text.jpg
dsgsec@htb[/htb]$fichier texte.jpg
text.jpg : données d'image GIF

```

Les serveurs Web peuvent également utiliser cette norme pour déterminer les types de fichiers, ce qui est généralement plus précis que de tester l'extension de fichier. L'exemple suivant montre comment une application Web PHP peut tester le type MIME d'un fichier téléchargé :

Code : php

```
$type = mime_content_type($_FILES['uploadFile']['tmp_name']);

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
     echo "Seules les images sont autorisées" ;
     mourir();
}

```

Comme nous pouvons le voir, les types MIME sont similaires à ceux trouvés dans les en-têtes Content-Type, mais leur source est différente, car PHP utilise la fonction `mime_content_type()` pour obtenir le type MIME d'un fichier. Essayons de répéter notre dernière attaque, mais maintenant avec un exercice qui teste à la fois l'en-tête Content-Type et le type MIME :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_content_type_request.jpg)

Une fois que nous avons transmis notre demande, nous remarquons que nous recevons le message d'erreur "Seules les images sont autorisées". Maintenant, essayons d'ajouter `GIF8` avant notre code PHP pour essayer d'imiter une image GIF tout en gardant notre extension de fichier comme `.php`, afin qu'il exécute le code PHP malgré tout :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_mime_type_request.jpg)

Cette fois, nous obtenons "Fichier téléchargé avec succès", et notre fichier est téléchargé avec succès sur le serveur :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_bypass_mime_type.jpg)

Nous pouvons maintenant visiter notre fichier téléchargé et nous verrons que nous pouvons exécuter avec succès les commandes système :

![](https://academy.hackthebox.com/storage/modules/136/file_uploads_php_manual_shell_gif.jpg)

Remarque : Nous constatons que la sortie de la commande commence par `GIF8` , car il s'agissait de la première ligne de notre script PHP à imiter les octets magiques GIF. Elle est désormais générée sous forme de texte brut avant l'exécution de notre code PHP.

Nous pouvons utiliser une combinaison des deux méthodes décrites dans cette section, ce qui peut nous aider à contourner certains filtres de contenu plus robustes. Par exemple, nous pouvons essayer d'utiliser un `Type MIME autorisé avec un type de contenu non autorisé`, un `Type MIME/Content autorisé avec une extension non autorisée` ou un `Type MIME/Content non autorisé avec une extension autorisée`, et bientôt. De même, nous pouvons tenter d'autres combinaisons et permutations pour essayer de confondre le serveur Web, et selon le niveau de sécurité du code, nous pouvons être en mesure de contourner divers filtres.
