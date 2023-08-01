Introduction aux services Web et aux API
========================================

* * * * *

Comme décrit par le World Wide Web Consortium (W3C) : *les services Web fournissent un moyen standard d'interopérabilité entre différentes applications logicielles, s'exécutant sur une variété de plates-formes et/ou de cadres. Les services Web se caractérisent par leur grande interopérabilité et extensibilité, ainsi que leurs descriptions exploitables par machine grâce à l'utilisation de XML.*

Les services Web permettent aux applications de communiquer entre elles. Les applications peuvent être totalement différentes. Considérez le scénario suivant :

-   Une application écrite en Java s'exécute sur un hôte Linux et utilise une base de données Oracle
-   Une autre application écrite en C++ s'exécute sur un hôte Windows et utilise une base de données SQL Server

Ces deux applications peuvent communiquer entre elles via Internet à l'aide de services Web.

Une interface de programmation d'application (API) est un ensemble de règles qui permet la transmission de données entre différents logiciels. La spécification technique de chaque API dicte l'échange de données.

Prenons l'exemple suivant : Un logiciel a besoin d'accéder à des informations, telles que les prix des billets pour des dates spécifiques. Pour obtenir les informations requises, il fera un appel à l'API d'un autre logiciel (y compris comment les données/fonctionnalités doivent être retournées). L'autre logiciel renverra toutes les données/fonctionnalités demandées.

L'interface par laquelle ces deux logiciels ont échangé des données est ce que l'API spécifie.

Vous pensez peut-être que les services Web et les API sont assez similaires, et vous aurez raison. Voir leurs principales différences ci-dessous.

Service Web vs API
------------------

* * * * *

Les termes `web service`et `application programming interface (API)`ne doivent pas être utilisés de manière interchangeable dans tous les cas.

-   Les services Web sont un type d'interface de programmation d'application (API). L'inverse n'est pas toujours vrai !
-   Les services Web ont besoin d'un réseau pour atteindre leur objectif. Les API peuvent atteindre leur objectif même hors ligne.
-   Les services Web autorisent rarement l'accès des développeurs externes, et il existe de nombreuses API qui accueillent le bricolage des développeurs externes.
-   Les services Web utilisent généralement SOAP pour des raisons de sécurité. Les API peuvent être trouvées en utilisant différentes conceptions, telles que XML-RPC, JSON-RPC, SOAP et REST.
-   Les services Web utilisent généralement le format XML pour l'encodage des données. Les API peuvent être trouvées en utilisant différents formats pour stocker des données, le plus populaire étant JavaScript Object Notation (JSON).

Approches/technologies de services Web
--------------------------------------

* * * * *

Il existe plusieurs approches/technologies pour fournir et consommer des services Web :

-   `XML-RPC`

    -   [XML-RPC](http://xmlrpc.com/spec.md) utilise XML pour coder/décoder l'appel de procédure distante (RPC) et le(s) paramètre(s) respectif(s). HTTP est généralement le transport de choix.
    -   Code : http

        ```
          --> POST /RPC2 HTTP/1.0
          User-Agent: Frontier/5.1.2 (WinNT)
          Host: betty.userland.com
          Content-Type: text/xml
          Content-length: 181

          <?xml version="1.0"?>
          <methodCall>
            <methodName>examples.getStateName</methodName>
            <params>
               <param>
         		     <value><i4>41</i4></value>
         		     </param>
        		  </params>
            </methodCall>

          <-- HTTP/1.1 200 OK
          Connection: close
          Content-Length: 158
          Content-Type: text/xml
          Date: Fri, 17 Jul 1998 19:55:08 GMT
          Server: UserLand Frontier/5.1.2-WinNT

          <?xml version="1.0"?>
          <methodResponse>
             <params>
                <param>
        		      <value><string>South Dakota</string></value>
        		      </param>
          	    </params>
           </methodResponse>

        ```

    La charge utile en XML est essentiellement une `<methodCall>`structure unique. `<methodCall>`doit contenir un `<methodName>`sous-élément lié à la méthode à appeler. Si l'appel requiert des paramètres, il `<methodCall>`doit alors contenir un `<params>`sous-élément.

-   `JSON-RPC`

    -   [JSON-RPC](https://www.jsonrpc.org/specification) utilise JSON pour appeler la fonctionnalité. HTTP est généralement le transport de choix.
    -   Code : http

        ```
          --> POST /ENDPOINT HTTP/1.1
           Host: ...
           Content-Type: application/json-rpc

        ```

    -   `\
        `
