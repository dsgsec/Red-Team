Introduction à XXE
==================

* * * * *

`XML External Entity (XXE) Injection`des vulnérabilités se produisent lorsque des données XML sont extraites d'une entrée contrôlée par l'utilisateur sans les nettoyer correctement ou les analyser en toute sécurité, ce qui peut nous permettre d'utiliser des fonctionnalités XML pour effectuer des actions malveillantes. Les vulnérabilités XXE peuvent causer des dommages considérables à une application Web et à son serveur principal, de la divulgation de fichiers sensibles à l'arrêt du serveur principal, c'est pourquoi elle est considérée comme l'un des 10 principaux risques de sécurité Web par l' [OWASP](https://owasp.org/www-project-top-ten/) .

* * * * *

XML
---

`Extensible Markup Language (XML)`est un langage de balisage commun (similaire à HTML et SGML) conçu pour le transfert et le stockage flexibles de données et de documents dans divers types d'applications. XML ne se concentre pas sur l'affichage des données mais principalement sur le stockage des données des documents et la représentation des structures de données. Les documents XML sont formés d'arborescences d'éléments, où chaque élément est essentiellement désigné par un `tag`, et le premier élément est appelé le `root element`, tandis que les autres éléments sont `child elements`.

Nous voyons ici un exemple de base d'un document XML représentant une structure de document de courrier électronique :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<email>
  <date>01-01-2022</date>
  <time>10:00 am UTC</time>
  <sender>john@inlanefreight.com</sender>
  <recipients>
    <to>HR@inlanefreight.com</to>
    <cc>
        <to>billing@inlanefreight.com</to>
        <to>payslips@inlanefreight.com</to>
    </cc>
  </recipients>
  <body>
  Hello,
      Kindly share with me the invoice for the payment made on January 1, 2022.
  Regards,
  John
  </body>
</email>

```

L'exemple ci-dessus montre certains des éléments clés d'un document XML, comme :

| Clé | Définition | Exemple |
| --- | --- | --- |
| `Tag` | Les clés d'un document XML, généralement entourées de caractères ( `<`/ `>`). | `<date>` |
| `Entity` | Variables XML, généralement entourées de caractères ( `&`/ `;`). | `&lt;` |
| `Element` | L'élément racine ou l'un de ses éléments enfants, et sa valeur est stockée entre une balise de début et une balise de fin. | `<date>01-01-2022</date>` |
| `Attribute` | Spécifications facultatives pour tout élément stocké dans les balises, qui peuvent être utilisées par l'analyseur XML. | `version="1.0"`/`encoding="UTF-8"` |
| `Declaration` | Généralement la première ligne d'un document XML, et définit la version XML et l'encodage à utiliser lors de son analyse. | `<?xml version="1.0" encoding="UTF-8"?>` |

De plus, certains caractères sont utilisés dans le cadre d'une structure de document XML, comme `<`, `>`, `&`ou `"`. Ainsi, si nous devons les utiliser dans un document XML, nous devons les remplacer par leurs références d'entité correspondantes (par exemple `&lt;`, `&gt;`, `&amp;`, `&quot;`). Enfin, nous pouvons écrire des commentaires dans des documents XML entre `<!--`et `-->`, similaires aux documents HTML.

* * * * *

DTD XML
-------

`XML Document Type Definition (DTD)`permet la validation d'un document XML par rapport à une structure de document prédéfinie. La structure de document prédéfinie peut être définie dans le document lui-même ou dans un fichier externe. Voici un exemple de DTD pour le document XML que nous avons vu précédemment :

Code : xml

```
<!DOCTYPE email [
  <!ELEMENT email (date, time, sender, recipients, body)>
  <!ELEMENT recipients (to, cc?)>
  <!ELEMENT cc (to*)>
  <!ELEMENT date (#PCDATA)>
  <!ELEMENT time (#PCDATA)>
  <!ELEMENT sender (#PCDATA)>
  <!ELEMENT to  (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
]>

```

Comme nous pouvons le voir, la DTD déclare l' `email`élément racine avec la `ELEMENT`déclaration de type, puis dénote ses éléments enfants. Après cela, chacun des éléments enfants est également déclaré, certains d'entre eux ayant également des éléments enfants, tandis que d'autres ne peuvent contenir que des données brutes (comme indiqué par `PCDATA`).

La DTD ci-dessus peut être placée dans le document XML lui-même, juste après le `XML Declaration`dans la première ligne. Sinon, il peut être stocké dans un fichier externe (par exemple `email.dtd`), puis référencé dans le document XML avec le `SYSTEM`mot-clé, comme suit :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "email.dtd">

```

Il est également possible de référencer une DTD via une URL, comme suit :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "http://inlanefreight.com/email.dtd">

```

Ceci est relativement similaire à la façon dont les documents HTML définissent et référencent les scripts JavaScript et CSS.

* * * * *

Entités XML
-----------

Nous pouvons également définir des entités personnalisées (c'est-à-dire des variables XML) dans les DTD XML, pour permettre la refactorisation des variables et réduire les données répétitives. Cela peut être fait avec l'utilisation du `ENTITY`mot-clé, qui est suivi du nom de l'entité et de sa valeur, comme suit :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>

```

Une fois que nous avons défini une entité, elle peut être référencée dans un document XML entre une esperluette `&`et un point-virgule `;`(par exemple `&company;`). Chaque fois qu'une entité est référencée, elle sera remplacée par sa valeur par l'analyseur XML. Plus intéressant, cependant, nous pouvons `reference External XML Entities`avec le `SYSTEM`mot-clé, qui est suivi du chemin de l'entité externe, comme suit :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "http://localhost/company.txt">
  <!ENTITY signature SYSTEM "file:///var/www/html/signature.txt">
]>

```

Remarque : nous pouvons également utiliser le `PUBLIC`mot-clé à la place de `SYSTEM`pour charger des ressources externes, qui est utilisé avec des entités et des normes déclarées publiquement, comme un code de langue ( `lang="en"`). Dans ce module, nous utiliserons `SYSTEM`, mais nous devrions pouvoir utiliser l'un ou l'autre dans la plupart des cas.

Cela fonctionne de la même manière que les entités XML internes définies dans les documents. Lorsque nous référençons une entité externe (par exemple `&signature;`), l'analyseur remplacera l'entité par sa valeur stockée dans le fichier externe (par exemple `signature.txt`). `When the XML file is parsed on the server-side, in cases like SOAP (XML) APIs or web forms, then an entity can reference a file stored on the back-end server, which may eventually be disclosed to us when we reference the entity`.

Dans la section suivante, nous verrons comment nous pouvons utiliser des entités XML externes pour lire des fichiers locaux ou même effectuer des actions plus malveillantes.
