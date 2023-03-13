Introduction aux inclusions de fichiers
========================

De nombreux langages back-end modernes, tels que `PHP`, `Javascript` ou `Java`, utilisent des paramètres HTTP pour spécifier ce qui est affiché sur la page Web, ce qui permet de créer des pages Web dynamiques, réduit la taille globale du script et simplifie le code. Dans de tels cas, des paramètres sont utilisés pour spécifier quelle ressource est affichée sur la page. Si ces fonctionnalités ne sont pas codées de manière sécurisée, un attaquant peut manipuler ces paramètres pour afficher le contenu de n'importe quel fichier local sur le serveur d'hébergement, entraînant une [Local File Inclusion (LFI)](https://owasp.org/www-project -web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) vulnérabilité.

* * * * *

Inclusion de fichiers locaux (LFI)
--------------------------

L'endroit le plus courant dans lequel nous trouvons généralement LFI est les moteurs de modèles. Afin que la majeure partie de l'application Web ait la même apparence lors de la navigation entre les pages, un moteur de création de modèles affiche une page qui affiche les parties statiques communes, telles que `header`, `barre de navigation` et `footer`, puis dynamiquement charge un autre contenu qui change entre les pages. Sinon, chaque page du serveur devrait être modifiée lorsque des modifications sont apportées à l'une des parties statiques. C'est pourquoi nous voyons souvent un paramètre comme `/index.php?page=about`, où `index.php` définit le contenu statique (par exemple, en-tête/pied de page), puis extrait uniquement le contenu dynamique spécifié dans le paramètre, qui dans ce cas peut être lu à partir d'un fichier appelé `about.php`. Comme nous contrôlons la partie "à propos" de la requête, il est possible que l'application Web récupère d'autres fichiers et les affiche sur la page.

Les vulnérabilités LFI peuvent entraîner la divulgation du code source, l'exposition de données sensibles et même l'exécution de code à distance dans certaines conditions. Le code source qui fuit peut permettre aux attaquants de tester le code pour d'autres vulnérabilités, ce qui peut révéler des vulnérabilités jusque-là inconnues. De plus, la fuite de données sensibles peut permettre aux attaquants d'énumérer le serveur distant pour d'autres faiblesses ou même de divulguer des informations d'identification et des clés qui peuvent leur permettre d'accéder directement au serveur distant. Dans des conditions spécifiques, LFI peut également permettre aux attaquants d'exécuter du code sur le serveur distant, ce qui peut compromettre l'ensemble du serveur principal et tout autre serveur qui lui est connecté.

* * * * *

Exemples de code vulnérable
---------------------------

Examinons quelques exemples de code vulnérable à l'inclusion de fichiers pour comprendre comment de telles vulnérabilités se produisent. Comme mentionné précédemment, les vulnérabilités d'inclusion de fichiers peuvent survenir dans de nombreux serveurs Web et frameworks de développement les plus populaires, tels que `PHP`, `NodeJS`, `Java`, `.Net` et bien d'autres. Chacun d'eux a une approche légèrement différente pour inclure les fichiers locaux, mais ils partagent tous une chose en commun : charger un fichier à partir d'un chemin spécifié.

Un tel fichier peut être un en-tête dynamique ou un contenu différent basé sur la langue spécifiée par l'utilisateur. Par exemple, la page peut avoir un paramètre `?language` GET, et si un utilisateur modifie la langue à partir d'un menu déroulant, la même page sera renvoyée mais avec un paramètre `language` différent (par exemple `?language= es`). Dans ce cas, la modification de la langue peut modifier le répertoire à partir duquel l'application Web charge les pages (par exemple `/en/` ou `/es/`). Si nous contrôlons le chemin en cours de chargement, nous pourrons peut-être exploiter cette vulnérabilité pour lire d'autres fichiers et potentiellement atteindre l'exécution de code à distance.

####PHP

Dans `PHP`, nous pouvons utiliser la fonction `include()` pour charger un fichier local ou distant lorsque nous chargeons une page. Si le `chemin` passé à `include()` est extrait d'un paramètre contrôlé par l'utilisateur, tel qu'un paramètre `GET` , et `le code ne filtre pas et ne nettoie pas explicitement l'entrée de l'utilisateur`, le code devient vulnérable à Inclusion de fichier. L'extrait de code suivant en montre un exemple :

Code : php

```
si (isset($_GET['langue'])) {
     inclure($_GET['langue']);
}

```

Nous voyons que le paramètre `language` est directement passé à la fonction `include()` . Ainsi, tout chemin que nous transmettons dans le paramètre `language` sera chargé sur la page, y compris tous les fichiers locaux sur le serveur principal. Ce n'est pas exclusif à la fonction `include()` , car il existe de nombreuses autres fonctions PHP qui conduiraient à la même vulnérabilité si nous avions le contrôle sur le chemin qui leur est transmis. Ces fonctions incluent `include_once()`, `require()`, `require_once()`, `file_get_contents()` et plusieurs autres également.

Remarque : Dans ce module, nous nous concentrerons principalement sur les applications Web PHP exécutées sur un serveur back-end Linux. Cependant, la plupart des techniques et des attaques fonctionneraient sur la majorité des autres frameworks, donc nos exemples seraient les mêmes avec une application Web écrite dans n'importe quel autre langage.

#### NodeJS

Tout comme dans le cas de PHP, les serveurs Web NodeJS peuvent également charger du contenu basé sur des paramètres HTTP. Voici un exemple simple d'utilisation d'un paramètre GET `language` pour contrôler les données écrites sur une page :

Code : javascdéchirer

```
if(req.query.language) {
     fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
         res.write(données);
     });
}

```

Comme nous pouvons le voir, tout paramètre transmis depuis l'URL est utilisé par la fonction `readfile` , qui écrit ensuite le contenu du fichier dans la réponse HTTP. Un autre exemple est la fonction `render()` dans le cadre `Express.js` . L'exemple suivant utilise le paramètre `language` pour déterminer de quel répertoire il doit extraire la page `about.html` :

Code : js

```
app.get("/about/:language", function(req, res) {
     res.render(`/${req.params.language}/about.html`);
});

```

Contrairement à nos exemples précédents dans lesquels les paramètres GET étaient spécifiés après un caractère (`?`) dans l'URL, l'exemple ci-dessus extrait le paramètre du chemin de l'URL (par exemple `/about/en` ou `/about/es`). Comme le paramètre est directement utilisé dans la fonction `render()` pour spécifier le fichier rendu, nous pouvons modifier l'URL pour afficher un autre fichier à la place.

####Java

Le même concept s'applique à de nombreux autres serveurs Web. Les exemples suivants montrent comment les applications Web d'un serveur Web Java peuvent inclure des fichiers locaux en fonction du paramètre spécifié, à l'aide de la fonction `include` :

Code : jsp

```
<c:if test="${not empty param.language}">
     <jsp:include file="<%= request.getParameter('language') %>" />
</c:si>

```

La fonction `include` peut prendre un fichier ou une URL de page comme argument, puis restitue l'objet dans le modèle frontal, similaire à ceux que nous avons vus précédemment avec NodeJS. La fonction `import` peut également être utilisée pour afficher un fichier local ou une URL, comme dans l'exemple suivant :

Code : jsp

```
<c:import url= "<%= request.getParameter('language') %>"/>

```

#### .FILET

Enfin, prenons un exemple de la façon dont les vulnérabilités d'inclusion de fichiers peuvent se produire dans les applications Web .NET. La fonction `Response.WriteFile` fonctionne de manière très similaire à tous nos exemples précédents, car elle prend un chemin de fichier pour son entrée et écrit son contenu dans la réponse. Le chemin peut être récupéré à partir d'un paramètre GET pour le chargement de contenu dynamique, comme suit :

Code : cs

```
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
     <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %>
}

```

De plus, la fonction `@Html.Partial()` peut également être utilisée pour afficher le fichier spécifié dans le cadre du modèle frontal, de la même manière que ce que nous avons vu précédemment :

Code : cs

```
@Html.Partial(HttpContext.Request.Query['langue'])

```

Enfin, la fonction `include` peut être utilisée pour afficher des fichiers locaux ou des URL distantes, et peut également exécuter les fichiers spécifiés :

Code : cs

```
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->

```

Lire vs exécuter
---------------

À partir de tous les exemples ci-dessus, nous pouvons voir que des vulnérabilités d'inclusion de fichiers peuvent survenir dans n'importe quel serveur Web et n'importe quel framework de développement, car tous fournissent des fonctionnalités pour charger du contenu dynamique et gérer des modèles frontaux.

La chose la plus importante à garder à l'esprit est que "certaines des fonctions ci-dessus ne lisent que le contenu des fichiers spécifiés, tandis que d'autres exécutent également les fichiers spécifiés". De plus, certains d'entre eux permettent de spécifier des URL distantes, tandis que d'autres ne fonctionnent qu'avec des fichiers locaux sur le serveur principal.

Le tableau suivant montre quelles fonctions peuvent exécuter des fichiers et lesquelles lisent uniquement le contenu des fichiers :

| Fonction | Lire le contenu | Exécuter | URL distante |
| --- | :-: | :-: | :-: |
| PHP | | | |
| `include()`/`include_once()` | ✅ | ✅ | ✅ |
| `require()`/`require_once()` | ✅ | ✅ | ❌ |
| `file_get_contents()` | ✅ | ❌ | ✅ |
| `fopen()`/`file()` | ✅ | ❌ | ❌ |
| NodeJS | | | |
| `fs.readFile()` | ✅ | ❌ | ❌ |
| `fs.sendFile()` | ✅ | ❌ | ❌ |
| `res.render()` | ✅ | ✅ | ❌ |
| Java | | | |
| `inclure` | ✅ | ❌ | ❌ |
| `importer` | ✅ | ✅ | ✅ |
| .NET | | | |
| `@Html.Partial()` | ✅ | ❌ | ❌ |
| `@Html.RemotePartial()` | ✅ | ❌ | ✅ |
| `Reponse.WriteFile()` | ✅ | ❌ | ❌ |
| `inclure` | ✅ | ✅ | ✅ |

C'est une différence importante à noter, car l'exécution de fichiers peut nous permettre d'exécuter des fonctions et éventuellement conduire à l'exécution de code, tandis que la seule lecture du contenu du fichier nous permettrait uniquement de lire le code source sans exécution de code. De plus, si nous avions accès au code source dans un exercice de boîte blanche ou dans un audit de code, la connaissance de ces actions nous aide à identifier les vulnérabilités potentielles d'inclusion de fichiers, en particulier si elles contiennent des entrées contrôlées par l'utilisateur.

Dans tous les cas, les vulnérabilités d'inclusion de fichiers sont critiques et peuvent éventuellement conduire à compromettre l'ensemble du serveur principal. Même si nous ne pouvions lire que le code source de l'application Web, cela peut toujours nous permettre de compromettre l'application Web, car cela peut révéler d'autres vulnérabilités comme mentionné précédemment, et le code source peut également contenir des clés de base de données, des informations d'identification d'administrateur ou d'autres information sensible.
