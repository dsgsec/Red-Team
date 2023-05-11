Tomcat - Découverte et énumération
================================

* * * * *

[Apache Tomcat](https://tomcat.apache.org/) est un serveur Web open source qui héberge des applications écrites en Java. Tomcat a été initialement conçu pour exécuter des scripts Java Servlets et Java Server Pages (JSP). Cependant, sa popularité a augmenté dans les frameworks basés sur Java et est maintenant largement utilisée par des frameworks tels que Spring et des outils tels que Gradle. Selon les données recueillies par [BuiltWith](https://trends.builtwith.com/Web-Server/Apache-Tomcat-Coyote) il existe actuellement plus de 220 000 sites Web Tomcat actifs. Voici quelques statistiques plus intéressantes :

- BuiltWith a rassemblé des données qui montrent que plus de 904 000 sites Web ont à un moment donné utilisé Tomcat
- 1,22 % des 1 millions de sites Web les plus importants utilisent Tomcat, tandis que 3,8 % des 100 000 sites Web les plus importants utilisent
- Tomcat occupe la position [# 13](https://webtechsurvey.com/technology/apache-tomcat) pour les serveurs Web par part de marché
- Certaines organisations qui utilisent Tomcat incluent Alibaba, l'Office américain des brevets et des marques (USPTO), la Croix-Rouge américaine et le LA Times

Tomcat est souvent moins susceptible d'être exposé à Internet (cependant). Nous le voyons de temps en temps sur des pentests externes et pouvons constituer un excellent point d'ancrage dans le réseau interne. Il est beaucoup plus courant de voir Tomcat (et plusieurs instances, d'ailleurs) lors de pentests internes. Il occupera généralement la première place sous "Cibles à haute valeur" dans un rapport EyeWitness, et le plus souvent, au moins une instance interne est configurée avec des informations d'identification faibles ou par défaut. Plus sur cela plus tard.

* * * * *

Découverte/Empreinte
----------------------

Au cours de notre test d'intrusion externe, nous exécutons EyeWitness et voyons un hôte répertorié sous "Cibles à haute valeur ajoutée". L'outil pense que l'hôte exécute Tomcat, mais nous devons confirmer pour planifier nos attaques. Si nous traitons avec Tomcat sur le réseau externe, cela pourrait être un pied facile dans l'environnement réseau interne.

Les serveurs Tomcat peuvent être identifiés par l'en-tête Server dans la réponse HTTP. Si le serveur fonctionne derrière un proxy inverse, la demande d'une page invalide devrait révéler le serveur et la version. Ici, nous pouvons voir que la version Tomcat `9.0.30` est utilisée.

![](https://academy.hackthebox.com/storage/modules/113/tomcat_invalid.png)

Des pages d'erreur personnalisées peuvent être utilisées sans divulguer ces informations de version. Dans ce cas, une autre méthode de détection d'un serveur Tomcat et de sa version consiste à utiliser la page `/docs` .

```
dsgsec@htb[/htb]$ curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat

<html lang="fr"><head><META http-equiv="Content-Type" content="text/html; charset=UTF-8"><link href="./images/docs-stylesheet.css " rel="stylesheet" type="text/css"><title>Apache Tomcat 9 (9.0.30) - Index de la documentation</title><meta name="author"

<SNIP>

```

Il s'agit de la page de documentation par défaut, qui ne peut pas être supprimée par les administrateurs. Voici la structure générale des dossiers d'une installation Tomcat.

```
├── poubelle
├── conf
│ ├── catalina.policy
│ ├── catalina.propriétés
│ ├── contexte.xml
│ ├── tomcat-users.xml
│ ├── tomcat-users.xsd
│ └── web.xml
├── lib
├── journaux
├── température
├── applications Web
│ ├── gestionnaire
│ │ ├── images
│ │ ├── META-INF
│ │ └── WEB-INF
| | └── web.xml
│ └── RACINE
│ └── WEB-INF
└── travailler
     └── Catalina
         └── hôte local

```

Le dossier `bin` stocke les scripts et les fichiers binaires nécessaires au démarrage et à l'exécution d'un serveur Tomcat. Le dossier `conf` stocke divers fichiers de configuration utilisés par Tomcat. Le fichier `tomcat-users.xml` stocke les informations d'identification des utilisateurs et les rôles qui leur sont attribués. Le dossier `lib` contient les différents fichiers JAR nécessaires au bon fonctionnement de Tomcat. Les dossiers `logs` et `temp` stockent des fichiers journaux temporaires. Le dossier `webapps` est la racine Web par défaut de Tomcat et héberge toutes les applications. Le dossier `work` agit comme un cache et est utilisé pour stocker des données pendant l'exécution.

Chaque dossier à l'intérieur de `webapps` doit avoir la structure suivante.

```
applications Web/application personnalisée
├── images
├── index.jsp
├── META-INF
│ └── contexte.xml
├── status.xsd
└── WEB-INF
     ├── jsp
     | └── admin.jsp
     └── web.xml
     └── lib
     | └── jdbc_drivers.jar
     └── cours
         └── AdminServlet.class

```

Le fichier le plus important parmi ceux-ci est `WEB-INF/web.xml`, connu sous le nom de descripteur de déploiement. Ce fichier stocke des informations sur les routes utilisées par l'application et les classes gérant ces routes. Toutes les classes compilées utilisées par l'application doivent être stockées dans le dossier `WEB-INF/classes` . Ces classes peuvent contenir une logique métier importante ainsi que des informations sensibles. Toute vulnérabilité dans ces fichiers peut entraîner une compromission totale du site Web. Le dossier `lib` stocke les bibliothèques nécessaires à cette application particulière. Le dossier `jsp` stocke les [Jakarta Server Pages (JSP)](https://en.wikipedia.org/wiki/Jakarta_Server_Pages), anciennement appelées `JavaServer Pages`, qui peuvent être comparées aux fichiers PHP sur un serveur Apache.

Voici un exemplefichier web.xml.

Code : xml

```
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE application Web PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<application-web>
   <servlet>
     <servlet-name>AdminServlet</servlet-name>
     <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
   </servlet>

   <mappage-servlet>
     <servlet-name>AdminServlet</servlet-name>
     <url-pattern>/admin</url-pattern>
   </servlet-mapping>
</web-app>

```

La configuration `web.xml` ci-dessus définit un nouveau servlet nommé `AdminServlet` qui est mappé à la classe `com.inlanefreight.api.AdminServlet`. Java utilise la notation par points pour créer des noms de packages, ce qui signifie que le chemin sur le disque pour la classe définie ci-dessus serait :

- `classes/com/inlanefreight/api/AdminServlet.class`

Ensuite, un nouveau mappage de servlet est créé pour mapper les requêtes vers `/admin` avec `AdminServlet`. Cette configuration enverra toute requête reçue pour `/admin` à la classe `AdminServlet.class` pour traitement. Le descripteur `web.xml` contient de nombreuses informations sensibles et constitue un fichier important à vérifier lors de l'exploitation d'une vulnérabilité d'inclusion de fichier local (LFI).

Le fichier `tomcat-users.xml` est utilisé pour autoriser ou interdire l'accès aux pages d'administration `/manager` et `host-manager` .

Code : xml

```
<?xml version="1.0" encoding="UTF-8" ?>

<SNIP>

<tomcat-users xmlns="http://tomcat.apache.org/xml"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
               version="1.0">
<!--
   Par défaut, aucun utilisateur n'est inclus dans le rôle "manager-gui" requis
   pour faire fonctionner l'application Web "/manager/html". Si vous souhaitez utiliser cette application,
   vous devez définir un tel utilisateur - le nom d'utilisateur et le mot de passe sont arbitraires.

   Rôles de gestionnaire Tomcat intégrés :
     - manager-gui - permet d'accéder à l'interface graphique HTML et aux pages d'état
     - manager-script - permet d'accéder à l'API HTTP et aux pages d'état
     - manager-jmx - permet d'accéder au proxy JMX et aux pages d'état
     - manager-status - autorise l'accès aux pages de statut uniquement

   Les utilisateurs ci-dessous sont entourés d'un commentaire et sont donc ignorés. Si tu
   souhaitez configurer un ou plusieurs de ces utilisateurs pour une utilisation avec le gestionnaire web
   application, n'oubliez pas de supprimer les <!.. ..> qui les entourent. Toi
   devra également définir les mots de passe sur quelque chose de approprié.
-->

  <SNIP>

!-- le gestionnaire d'utilisateurs ne peut accéder qu'à la section du gestionnaire -->
<role rolename="manager-gui" />
<user username="tomcat" password="tomcat" roles="manager-gui" />

<!-- l'utilisateur admin peut accéder à la fois aux sections manager et admin -->
<role rolename="admin-gui" />
<user username="admin" password="admin" roles="manager-gui,admin-gui" />

</tomcat-users>

```

Le fichier nous montre à quoi chacun des rôles `manager-gui`, `manager-script`, `manager-jmx` et `manager-status` donne accès. Dans cet exemple, nous pouvons voir qu'un utilisateur `tomcat` avec le mot de passe `tomcat` a le rôle `manager-gui` et qu'un deuxième mot de passe faible `admin` est défini pour le compte d'utilisateur `admin`.

* * * * *

Énumération
-----------

Après avoir pris l'empreinte de l'instance Tomcat, à moins qu'elle ne présente une vulnérabilité connue, nous souhaitons généralement rechercher les pages `/manager` et `/host-manager` . Nous pouvons tenter de les localiser à l'aide d'un outil tel que `Gobuster` ou simplement y accéder directement.

```
dsgsec@htb[/htb]$ gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt

================================================= =============
Gobuster v3.0.1
par OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
================================================= =============
[+] URL : http://web01.inlanefreight.local:8180/
[+] Fils : 10
[+] Liste de mots : /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
[+] Codes d'état : 200 204 301 302 307 401 403
[+] Agent utilisateur : gobuster/3.0.1
[+] Délai d'attente : 10 s
================================================= =============
2021/09/21 17:34:54 Démarrage du gobuster
================================================= =============
/docs (Statut : 302)
/exemples (Statut : 302)
/gérant (Statut : 302)
Progression : 49 959 / 87 665 (56,99 %)^C
[!] Interruption du clavier détectée, se terminant.
================================================= =============
2021/09/21 17:44:29 Terminé
================================================= =============

```

Nous pourrons peut-être nous connecter à l'un d'entre eux en utilisant des informations d'identification faibles telles que `tomcat: tomcat`, `admin:admin`, etc. Si ces premiers essais ne fonctionnent pas, nous pouvons essayer une attaque par force brute de mot de passe contre la page de connexion, abordée dans la section suivante. Si nous parvenons à nous connecter, nous pouvons télécharger une fichier contenant un shell Web JSP et obtenire une exécution du code mote sur le serveur Tomcat.

Maintenant que nous avons appris la structure et la fonction de Tomcat, attaquons-le en abusant des fonctionnalités intégrées et en exploitant une vulnérabilité bien connue qui affectait des versions spécifiques de Tomcat.