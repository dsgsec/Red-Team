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
           Content-Length: ...

          {"method": "sum", "params": {"a":3, "b":4}, "id":0}

          <-- HTTP/1.1 200 OK
           ...
           Content-Type: application/json-rpc

           {"result": 7, "error": null, "id": 0}

        ```

    L' `{"method": "sum", "params": {"a":3, "b":4}, "id":0}`objet est sérialisé à l'aide de JSON. Notez les trois propriétés : `method`, `params`et `id`. `method`contient le nom de la méthode à invoquer. `params`contient un tableau portant les arguments à passer. `id`contient un identifiant établi par le client. Le serveur doit répondre avec la même valeur dans l'objet de réponse s'il est inclus.

-   `SOAP (Simple Object Access Protocol)`

    -   SOAP utilise également XML mais fournit plus de fonctionnalités que XML-RPC. SOAP définit à la fois une structure d'en-tête et une structure de charge utile. Le premier identifie les actions que les nœuds SOAP sont censés entreprendre sur le message, tandis que le second traite des informations transportées. Une déclaration WSDL (Web Services Definition Language) est facultative. WSDL spécifie comment un service SOAP peut être utilisé. Divers protocoles de niveau inférieur (HTTP inclus) peuvent être le transport.
    -   Anatomie d'un message SOAP
        -   `soap:Envelope`: (Bloc obligatoire) Balise pour différencier SOAP du XML normal. Cette balise nécessite un `namespace`attribut.
        -   `soap:Header`: (Bloc facultatif) Active l'extensibilité de SOAP via les modules SOAP.
        -   `soap:Body`: (Bloc obligatoire) Contient la procédure, les paramètres et les données.
        -   `soap:Fault`: (Bloc facultatif) Utilisé dans `soap:Body`pour les messages d'erreur lors d'un échec d'appel d'API.
    -   Code : http

        ```
          --> POST /Quotation HTTP/1.0
          Host: www.xyz.org
          Content-Type: text/xml; charset = utf-8
          Content-Length: nnn

          <?xml version = "1.0"?>
          <SOAP-ENV:Envelope
            xmlns:SOAP-ENV = "http://www.w3.org/2001/12/soap-envelope"
             SOAP-ENV:encodingStyle = "http://www.w3.org/2001/12/soap-encoding">

            <SOAP-ENV:Body xmlns:m = "http://www.xyz.org/quotations">
               <m:GetQuotation>
                 <m:QuotationsName>MiscroSoft</m:QuotationsName>
              </m:GetQuotation>
            </SOAP-ENV:Body>
          </SOAP-ENV:Envelope>

          <-- HTTP/1.0 200 OK
          Content-Type: text/xml; charset = utf-8
          Content-Length: nnn

          <?xml version = "1.0"?>
          <SOAP-ENV:Envelope
           xmlns:SOAP-ENV = "http://www.w3.org/2001/12/soap-envelope"
            SOAP-ENV:encodingStyle = "http://www.w3.org/2001/12/soap-encoding">

          <SOAP-ENV:Body xmlns:m = "http://www.xyz.org/quotation">
          	  <m:GetQuotationResponse>
          	     <m:Quotation>Here is the quotation</m:Quotation>
             </m:GetQuotationResponse>
           </SOAP-ENV:Body>
          </SOAP-ENV:Envelope>

        ```

Remarque : Vous pouvez rencontrer des enveloppes SOAP légèrement différentes. Leur anatomie sera la même, cependant.

-   `WS-BPEL (Web Services Business Process Execution Language)`
    -   Les services Web WS-BPEL sont essentiellement des services Web SOAP avec plus de fonctionnalités pour décrire et appeler des processus métier.
    -   Les services Web WS-BPEL ressemblent fortement aux services SOAP. Pour cette raison, ils ne seront pas inclus dans le périmètre de ce module.
-   `RESTful (Representational State Transfer)`
    -   Les services Web REST utilisent généralement XML ou JSON. Les déclarations WSDL sont prises en charge mais peu courantes. HTTP est le transport de choix, et les verbes HTTP sont utilisés pour accéder/modifier/supprimer des ressources et utiliser des données.
    -   Code : http

        ```
          --> POST /api/2.2/auth/signin HTTP/1.1
          HOST: my-server
          Content-Type:text/xml

          <tsRequest>
            <credentials name="administrator" password="passw0rd">
              <site contentUrl="" />
            </credentials>
          </tsRequest>

        ```

    -   Code : http

        ```
          --> POST /api/2.2/auth/signin HTTP/1.1
          HOST: my-server
          Content-Type:application/json
          Accept:application/json

          {
           "credentials": {
             "name": "administrator",
            "password": "passw0rd",
            "site": {
              "contentUrl": ""
             }
            }
          }

        ```

* * * * *

Des spécifications/protocoles API similaires existent, tels que Remote Procedure Call (RPC), SOAP, REST, gRPC, GraphQL, etc.

Ne vous sentez pas dépassé ! Dans les sections suivantes, vous aurez la possibilité d'interagir avec différents services Web et API.




Langage de description de services Web (WSDL)
=============================================

* * * * *

WSDL signifie Web Service Description Language. WSDL est un fichier XML exposé par les services Web qui informe les clients des services/méthodes fournis, y compris leur emplacement et la convention d'appel de méthode.

Le fichier WSDL d'un service Web ne doit pas toujours être accessible. Les développeurs peuvent ne pas vouloir exposer publiquement le fichier WSDL d'un service Web, ou ils peuvent l'exposer via un emplacement inhabituel, en suivant une approche de sécurité par l'obscurité. Dans ce dernier cas, le fuzzing de répertoire/paramètre peut révéler l'emplacement et le contenu d'un fichier WSDL.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre le service cible et suivez-le.

Supposons que nous évaluons un service SOAP résidant dans `http://<TARGET IP>:3002`. Nous n'avons pas été informés d'un fichier WSDL.

Commençons par effectuer un fuzzing de répertoire de base sur le service Web.

```
dsgsec@htb[/htb]$ dirb http://<TARGET IP>:3002

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Fri Mar 25 11:53:09 2022
URL_BASE: http://<TARGET IP>:3002/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://<TARGET IP>:3002/ ----
+ http://<TARGET IP>:3002/wsdl (CODE:200|SIZE:0)

-----------------
END_TIME: Fri Mar 25 11:53:24 2022
DOWNLOADED: 4612 - FOUND: 1

```

Il semble `http://<TARGET IP>:3002/wsdl`exister. Examinons son contenu comme suit.

```
dsgsec@htb[/htb]$ curl http://<TARGET IP>:3002/wsdl

```

La réponse est vide ! Il existe peut-être un paramètre qui nous permettra d'accéder au fichier WSDL du service Web SOAP. Effectuons le fuzzing des paramètres en utilisant *ffuf* et la liste [burp-parameter-names.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/burp-parameter-names.txt) , comme suit. *-fs 0* filtre les réponses vides (taille = 0) et *-mc 200* correspond aux réponses *HTTP 200* .

```
dsgsec@htb[/htb]$ ffuf -w "/home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/burp-parameter-names.txt" -u 'http://<TARGET IP>:3002/wsdl?FUZZ' -fs 0 -mc 200

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://<TARGET IP>:3002/wsdl?FUZZ
 :: Wordlist         : FUZZ: /home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
 :: Filter           : Response size: 0
________________________________________________

:: Progress: [40/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Error
:: Progress: [537/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Erro
wsdl [Status: 200, Size: 4461, Words: 967, Lines: 186]
:: Progress: [982/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Erro::
Progress: [1153/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Err::
Progress: [1780/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Err::
Progress: [2461/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Err::
Progress: [2588/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Err::
Progress: [2588/2588] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::

```

Il semble que *wsdl* soit un paramètre valide. Lançons maintenant une demande de`http://<TARGET IP>:3002/wsdl?wsdl`

```
dsgsec@htb[/htb]$ curl http://<TARGET IP>:3002/wsdl?wsdl

<?xml version="1.0" encoding="UTF-8"?>
<wsdl:definitions targetNamespace="http://tempuri.org/"
	xmlns:s="http://www.w3.org/2001/XMLSchema"
	xmlns:soap12="http://schemas.xmlsoap.org/wsdl/soap12/"
	xmlns:http="http://schemas.xmlsoap.org/wsdl/http/"
	xmlns:mime="http://schemas.xmlsoap.org/wsdl/mime/"
	xmlns:tns="http://tempuri.org/"
	xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
	xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"
	xmlns:soapenc="http://schemas.xmlsoap.org/soap/encoding/"
	xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/">
	<wsdl:types>
		<s:schema elementFormDefault="qualified" targetNamespace="http://tempuri.org/">
			<s:element name="LoginRequest">
				<s:complexType>
					<s:sequence>
						<s:element minOccurs="1" maxOccurs="1" name="username" type="s:string"/>
						<s:element minOccurs="1" maxOccurs="1" name="password" type="s:string"/>
					</s:sequence>
				</s:complexType>
			</s:element>
			<s:element name="LoginResponse">
				<s:complexType>
					<s:sequence>
						<s:element minOccurs="1" maxOccurs="unbounded" name="result" type="s:string"/>
					</s:sequence>
				</s:complexType>
			</s:element>
			<s:element name="ExecuteCommandRequest">
				<s:complexType>
					<s:sequence>
						<s:element minOccurs="1" maxOccurs="1" name="cmd" type="s:string"/>
					</s:sequence>
				</s:complexType>
			</s:element>
			<s:element name="ExecuteCommandResponse">
				<s:complexType>
					<s:sequence>
						<s:element minOccurs="1" maxOccurs="unbounded" name="result" type="s:string"/>
					</s:sequence>
				</s:complexType>
			</s:element>
		</s:schema>
	</wsdl:types>
	<!-- Login Messages -->
	<wsdl:message name="LoginSoapIn">
		<wsdl:part name="parameters" element="tns:LoginRequest"/>
	</wsdl:message>
	<wsdl:message name="LoginSoapOut">
		<wsdl:part name="parameters" element="tns:LoginResponse"/>
	</wsdl:message>
	<!-- ExecuteCommand Messages -->
	<wsdl:message name="ExecuteCommandSoapIn">
		<wsdl:part name="parameters" element="tns:ExecuteCommandRequest"/>
	</wsdl:message>
	<wsdl:message name="ExecuteCommandSoapOut">
		<wsdl:part name="parameters" element="tns:ExecuteCommandResponse"/>
	</wsdl:message>
	<wsdl:portType name="HacktheBoxSoapPort">
		<!-- Login Operaion | PORT -->
		<wsdl:operation name="Login">
			<wsdl:input message="tns:LoginSoapIn"/>
			<wsdl:output message="tns:LoginSoapOut"/>
		</wsdl:operation>
		<!-- ExecuteCommand Operation | PORT -->
		<wsdl:operation name="ExecuteCommand">
			<wsdl:input message="tns:ExecuteCommandSoapIn"/>
			<wsdl:output message="tns:ExecuteCommandSoapOut"/>
		</wsdl:operation>
	</wsdl:portType>
	<wsdl:binding name="HacktheboxServiceSoapBinding" type="tns:HacktheBoxSoapPort">
		<soap:binding transport="http://schemas.xmlsoap.org/soap/http"/>
		<!-- SOAP Login Action -->
		<wsdl:operation name="Login">
			<soap:operation soapAction="Login" style="document"/>
			<wsdl:input>
				<soap:body use="literal"/>
			</wsdl:input>
			<wsdl:output>
				<soap:body use="literal"/>
			</wsdl:output>
		</wsdl:operation>
		<!-- SOAP ExecuteCommand Action -->
		<wsdl:operation name="ExecuteCommand">
			<soap:operation soapAction="ExecuteCommand" style="document"/>
			<wsdl:input>
				<soap:body use="literal"/>
			</wsdl:input>
			<wsdl:output>
				<soap:body use="literal"/>
			</wsdl:output>
		</wsdl:operation>
	</wsdl:binding>
	<wsdl:service name="HacktheboxService">
		<wsdl:port name="HacktheboxServiceSoapPort" binding="tns:HacktheboxServiceSoapBinding">
			<soap:address location="http://localhost:80/wsdl"/>
		</wsdl:port>
	</wsdl:service>
</wsdl:definitions>

```

Nous avons identifié le fichier WSDL du service SOAP !

Remarque : Les fichiers WSDL peuvent être trouvés sous de nombreuses formes, telles que `/example.wsdl`, `?wsdl`, `/example.disco`, `?disco`etc. [DISCO](https://docs.microsoft.com/en-us/archive/msdn-magazine/2002/february/xml-files-publishing-and-discovering-web-services-with-disco-and-uddi) est une technologie Microsoft de publication et de découverte de services Web.

Répartition des fichiers WSDL
-----------------------------

* * * * *

Passons maintenant en revue ensemble le fichier WSDL identifié ci-dessus.

Le fichier WSDL ci-dessus suit la disposition [WSDL version 1.1](https://www.w3.org/TR/2001/NOTE-wsdl-20010315) et se compose des éléments suivants.

-   `Definition`
    -   L'élément racine de tous les fichiers WSDL. Dans la définition, le nom du service Web est spécifié, tous les espaces de noms utilisés dans le document WSDL sont déclarés et tous les autres éléments de service sont définis.
    -   Code : xml

        ```
        <wsdl:definitions targetNamespace="http://tempuri.org/"

            <wsdl:types></wsdl:types>
            <wsdl:message name="LoginSoapIn"></wsdl:message>
            <wsdl:portType name="HacktheBoxSoapPort">
          	  <wsdl:operation name="Login"></wsdl:operation>
            </wsdl:portType>
            <wsdl:binding name="HacktheboxServiceSoapBinding" type="tns:HacktheBoxSoapPort">
          	  <wsdl:operation name="Login">
          		  <soap:operation soapAction="Login" style="document"/>
          		  <wsdl:input></wsdl:input>
          		  <wsdl:output></wsdl:output>
          	  </wsdl:operation>
            </wsdl:binding>
            <wsdl:service name="HacktheboxService"></wsdl:service>
        </wsdl:definitions>

        ```

-   `Data Types`
    -   Les types de données à utiliser dans les messages échangés.
    -   Code : xml

        ```
        <wsdl:types>
            <s:schema elementFormDefault="qualified" targetNamespace="http://tempuri.org/">
          	  <s:element name="LoginRequest">
          		  <s:complexType>
          			  <s:sequence>
          				  <s:element minOccurs="1" maxOccurs="1" name="username" type="s:string"/>
          				  <s:element minOccurs="1" maxOccurs="1" name="password" type="s:string"/>
          			  </s:sequence>
          		  </s:complexType>
          	  </s:element>
          	  <s:element name="LoginResponse">
          		  <s:complexType>
          			  <s:sequence>
          				  <s:element minOccurs="1" maxOccurs="unbounded" name="result" type="s:string"/>
          			  </s:sequence>
          		  </s:complexType>
          	  </s:element>
          	  <s:element name="ExecuteCommandRequest">
          		  <s:complexType>
          			  <s:sequence>
          				  <s:element minOccurs="1" maxOccurs="1" name="cmd" type="s:string"/>
          			  </s:sequence>
          		  </s:complexType>
          	  </s:element>
          	  <s:element name="ExecuteCommandResponse">
          		  <s:complexType>
          			  <s:sequence>
          				  <s:element minOccurs="1" maxOccurs="unbounded" name="result" type="s:string"/>
          			  </s:sequence>
          		  </s:complexType>
          	  </s:element>
            </s:schema>
        </wsdl:types>

        ```

-   `Messages`
    -   Définit les opérations d'entrée et de sortie prises en charge par le service Web. En d'autres termes, à travers l' élément *messages* , les messages à échanger sont définis et présentés soit comme un document entier, soit comme des arguments à mapper à une invocation de méthode.
    -   Code : xml

        ```
        <!-- Login Messages -->
        <wsdl:message name="LoginSoapIn">
            <wsdl:part name="parameters" element="tns:LoginRequest"/>
        </wsdl:message>
        <wsdl:message name="LoginSoapOut">
            <wsdl:part name="parameters" element="tns:LoginResponse"/>
        </wsdl:message>
        <!-- ExecuteCommand Messages -->
        <wsdl:message name="ExecuteCommandSoapIn">
            <wsdl:part name="parameters" element="tns:ExecuteCommandRequest"/>
        </wsdl:message>
        <wsdl:message name="ExecuteCommandSoapOut">
            <wsdl:part name="parameters" element="tns:ExecuteCommandResponse"/>
        </wsdl:message>
          ```	

        ```

-   `Operation`
    -   Définit les actions SOAP disponibles parallèlement à l'encodage de chaque message.
-   `Port Type`
    -   Encapsule tous les messages d'entrée et de sortie possibles dans une opération. Plus précisément, il définit le service web, les opérations disponibles et les messages échangés. Veuillez noter que dans WSDL version 2.0, l' élément *d'interface* est chargé de définir les opérations disponibles et lorsqu'il s'agit de messages, l'élément de types (de données) gère leur définition.
    -   Code : xml

        ```
        <wsdl:portType name="HacktheBoxSoapPort">
            <!-- Login Operaion | PORT -->
            <wsdl:operation name="Login">
          	  <wsdl:input message="tns:LoginSoapIn"/>
          	  <wsdl:output message="tns:LoginSoapOut"/>
            </wsdl:operation>
            <!-- ExecuteCommand Operation | PORT -->
            <wsdl:operation name="ExecuteCommand">
          	  <wsdl:input message="tns:ExecuteCommandSoapIn"/>
          	  <wsdl:output message="tns:ExecuteCommandSoapOut"/>
            </wsdl:operation>
        </wsdl:portType>

        ```

-   `Binding`
    -   Lie l'opération à un type de port particulier. Considérez les liaisons comme des interfaces. Un client appellera le type de port concerné et, en utilisant les détails fournis par la liaison, pourra accéder aux opérations liées à ce type de port. En d'autres termes, les liaisons fournissent des détails d'accès au service Web, tels que le format de message, les opérations, les messages et les interfaces (dans le cas de WSDL version 2.0).
    -   Code : xml

        ```
        <wsdl:binding name="HacktheboxServiceSoapBinding" type="tns:HacktheBoxSoapPort">
            <soap:binding transport="http://schemas.xmlsoap.org/soap/http"/>
            <!-- SOAP Login Action -->
            <wsdl:operation name="Login">
          	  <soap:operation soapAction="Login" style="document"/>
          	  <wsdl:input>
          		  <soap:body use="literal"/>
          	  </wsdl:input>
          	  <wsdl:output>
          		  <soap:body use="literal"/>
          	  </wsdl:output>
            </wsdl:operation>
            <!-- SOAP ExecuteCommand Action -->
            <wsdl:operation name="ExecuteCommand">
          	  <soap:operation soapAction="ExecuteCommand" style="document"/>
          	  <wsdl:input>
          		  <soap:body use="literal"/>
          	  </wsdl:input>
          	  <wsdl:output>
          		  <soap:body use="literal"/>
          	  </wsdl:output>
            </wsdl:operation>
        </wsdl:binding>

        ```

-   `Service`
    -   Un client appelle le service Web via le nom du service spécifié dans le numéro de service. Grâce à cet élément, le client identifie l'emplacement du service Web.
    -   Code : xml

        ```
            <wsdl:service name="HacktheboxService">

              <wsdl:port name="HacktheboxServiceSoapPort" binding="tns:HacktheboxServiceSoapBinding">
                <soap:address location="http://localhost:80/wsdl"/>
              </wsdl:port>

            </wsdl:service>

        ```

Dans la `SOAP Action Spoofing`section, plus tard, nous verrons comment nous pouvons tirer parti du fichier WSDL identifié pour interagir avec le service Web.

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/160/section/1471)
