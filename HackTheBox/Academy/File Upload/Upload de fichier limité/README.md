Upload de fichiers limités
====================

* * * * *

Jusqu'à présent, nous avons principalement traité des contournements de filtre pour obtenir des téléchargements de fichiers arbitraires via une application Web vulnérable, qui est le principal objectif de ce module à ce niveau. Bien que les formulaires d'upload de fichiers avec des filtres faibles puissent être exploités pour télécharger des fichiers arbitraires, certains formulaires d'upload  ont des filtres sécurisés qui peuvent ne pas être exploités avec les techniques dont nous avons discuté. Cependant, même si nous avons affaire à un formulaire d'upload  de fichiers limité (c'est-à-dire non arbitraire), qui nous permet uniquement de télécharger des types de fichiers spécifiques, nous pouvons toujours être en mesure d'effectuer des attaques sur l'application Web.

Certains types de fichiers, comme `SVG`,` HTML`, `XML`, et même certains fichiers d'image et de document, peuvent nous permettre d'introduire de nouvelles vulnérabilités à l'application Web en permettant d'upload  des versions malveillantes de ces fichiers. C'est pourquoi les extensions de fichiers permis de fuzzing sont un exercice important pour toute attaque de téléchargement de fichiers. Il nous permet d'explorer quelles attaques peuvent être réalisables sur le serveur Web. Alors, explorons certaines de ces attaques.

* * * * *

XSS
---

De nombreux types de fichiers peuvent nous permettre d'introduire une vulnérabilité «XSS» stockée à l'application Web en téléchargeant des versions avec malveillance.

L'exemple le plus élémentaire est lorsqu'une application Web nous permet d'upload  des fichiers `HTML`. Bien que les fichiers HTML ne nous permettent pas d'exécuter du code (par exemple, PHP), il serait toujours possible d'implémenter le code JavaScript en eux pour transporter une attaque XSS ou CSRF sur quiconque visite la page HTML upload. Si la cible voit un lien à partir d'un site Web en qui ils ont confiance et que le site Web est vulnérable à l'upload de documents HTML, il peut être possible de les inciter à visiter le lien et à porter l'attaque sur leurs machines.

Un autre exemple d'attaques XSS est les applications Web qui affichent les métadonnées d'une image après son téléchargement. Pour de telles applications Web, nous pouvons inclure une charge utile XSS dans l'un des paramètres de métadonnées qui acceptent le texte brut, comme les paramètres «comment» ou «artiste», comme suit:

```
dsgsec @ htb [/ htb] $ exiftool -comment = '"> <img src = 1 onError = alert (window.origin)>' htb.jpg
dsgsec @ htb [/ htb] $ exiftool htb.jpg
...COUPER...
Commentaire: "> <img src = 1 onError = alert (window.origin)>

```

Nous pouvons voir que le paramètre «comment» a été mis à jour sur notre charge utile XSS. Lorsque les métadonnées de l'image s'affichent, la charge utile XSS doit être déclenchée et le code JavaScript sera exécuté pour porter l'attaque XSS. De plus, si nous modifions le type MIME de l'image en «Text / HTML», certaines applications Web peuvent l'afficher comme un document HTML au lieu d'une image, auquel cas la charge utile XSS serait déclenchée même si les métadonnées n'étaient pas directement affichées directement affichées .

Enfin, les attaques XSS peuvent également être transportées avec des images «SVG», ainsi que plusieurs autres attaques. `Les images de graphiques vectoriels évolutifs (SVG) sont basés sur XML, et ils décrivent des graphiques vectoriels 2D, que le navigateur rend dans une image. Pour cette raison, nous pouvons modifier leurs données XML pour inclure une charge utile XSS. Par exemple, nous pouvons écrire ce qui suit sur `htb.svg`:

Code: XML

```
<? xml version = "1.0" Encoding = "utf-8"?>
<! Doctype svg public "- // w3c // dtd svg 1.1 // en" "http://www.w3.org/graphics/svg/1.1/dtd/svg11.dtd">
<svg xmlns = "http://www.w3.org/2000/svg" version = "1.1" width = "1" height = "1">
     <rect x = "1" y = "1" width = "1" height = "1" fill = "green" rroD = "Black" />
     <script type = "text / javascript"> alert ("window.origin"); </ script>
</svg>

```

Une fois que nous avons upload  l'image sur l'application Web, la charge utile XSS sera déclenchée chaque fois que l'image est affichée.

Pour en savoir plus sur XSS, vous pouvez vous référer au module [https://academy.hackthebox.com/module/details/103) (https://academy.hackthebox.com/module/details/103).

Exercice: essayez les attaques ci-dessus avec l'exercice à la fin de cette section et voyez si la charge utile XSS est déclenchée et affiche l'alerte.

* * * * *

Xxe
---

Des attaques similaires peuvent être effectuées pour entraîner une exploitation XXE. Avec les images SVG, nous pouvons également inclure des données XML malveillantes pour fuir le code source de l'application Web et d'autres documents internes du serveur. L'exemple suivant peut être utilisé pour une image SVG qui fuit le contenu de (`/ etc / passwd`):

Code: XML

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>

```

Une fois que l'image SVG ci-dessus est upload et affichée, le document XML serait traité et nous devrions obtenir les informations de (`/ etc / passwd`) imprimées sur la page ou indiqués dans la source de la page. De même, si l'application Web permet le téléchargement de documents «XML», la même charge utile peut effectuer la même attaque lorsque les données XML s'affichent sur l'application Web.

Bien que les fichiers de systèmes de lecture comme `/ etc / passwd` puissent être très utiles pour l'énumération du serveur, il peut avoir un avantage encore plus significatif pour les tests de pénétration Web, car il nous permet de lire les fichiers source de l'application Web. L'accès au code source nous permettra de trouver plus de vulnérabilités à exploiter dans l'application Web par le biais de tests de pénétration de Whitebox. Pour déposerExploitation de chargement, il peut nous permettre de «localiser le répertoire de téléchargement, d'identifier les extensions autorisées ou de trouver le schéma de dénomination du fichier», ce qui peut devenir pratique pour une exploitation ultérieure.

Pour utiliser XXE pour lire le code source dans les applications Web PHP, nous pouvons utiliser la charge utile suivante dans notre image SVG:

Code: XML

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>

```

Une fois l'image SVG affichée, nous devons obtenir le contenu codé de Base64 de `index.php`, que nous pouvons décoder pour lire le code source. Pour plus de xxe, vous pouvez vous référer au module [WEB Attacks] (https://academy.hackthebox.com/module/details/134).

L'utilisation de données XML n'est pas unique aux images SVG, car elle est également utilisée par de nombreux types de documents, comme «PDF», «Documents Word», «PowerPoint Documents», entre autres. Tous ces documents incluent les données XML pour spécifier leur format et leur structure. Supposons qu'une application Web ait utilisé une visionneuse de documents vulnérable à XXE et autorisé à télécharger l'un de ces documents. Dans ce cas, nous pouvons également modifier leurs données XML pour inclure les éléments XXE malveillants, et nous serions en mesure de porter une attaque XXE aveugle sur le serveur Web back-end.

Une autre attaque similaire qui est également réalisable via ces types de fichiers est une attaque SSRF. Nous pouvons utiliser la vulnérabilité XXE pour énumérer les services disponibles en interne ou même appeler des API privés pour effectuer des actions privées. Pour en savoir plus sur SSRF, vous pouvez vous référer aux [attaques côté serveur] (https://academy.hackthebox.com/module/details/145).

* * * * *

Dos
---

Enfin, de nombreuses vulnérabilités de téléchargement de fichiers peuvent conduire à un «DOS)« DOS) »attaquer sur le serveur Web. Par exemple, nous pouvons utiliser les charges utiles XXE précédentes pour réaliser des attaques DOS, comme discuté dans les [attaques Web] (https://academy.hackthebox.com/module/details/134).

De plus, nous pouvons utiliser une «bombe de décompression» avec des types de fichiers qui utilisent la compression des données, comme les archives «zip». Si une application Web décompresse automatiquement une archive zip, il est possible de télécharger une archive malveillante contenant des archives zip imbriquées, ce qui peut éventuellement conduire à de nombreux pétaoctets de données, ce qui entraîne un crash sur le serveur back-end.

Une autre attaque DOS possible est une attaque «Pixel Flood» avec certains fichiers d'image qui utilisent la compression d'image, comme «JPG» ou «PNG». Nous pouvons créer n'importe quel fichier image `jpg` avec n'importe quelle taille d'image (par exemple` 500x500`), puis modifier manuellement ses données de compression pour dire qu'elle a une taille de (`0xffff x 0xffff`), qui se traduit par une image avec une perçue perçue Taille de 4 gigapixels. Lorsque l'application Web tente d'afficher l'image, elle tentera d'allouer toute sa mémoire à cette image, ce qui entraîne un crash sur le serveur arrière.

En plus de ces attaques, nous pouvons essayer quelques autres méthodes pour provoquer un DOS sur le serveur back-end. Une façon consiste à télécharger un fichier trop grand, car certains formulaires de téléchargement peuvent ne pas limiter la taille du fichier de téléchargement ou le vérifier avant de le télécharger, ce qui peut remplir le disque dur du serveur et le faire s'écraser ou ralentir considérablement.

Si la fonction de téléchargement est vulnérable à la traversée du répertoire, nous pouvons également tenter de télécharger des fichiers dans un autre répertoire (par exemple `../../../ etc / passwd`), ce qui peut également provoquer le plantage du serveur. «Essayez de rechercher d'autres exemples d'attaques DOS via une fonctionnalité de téléchargement de fichiers vulnérables».

Instance de démarrage