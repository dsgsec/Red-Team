Jenkins - Découverte et énumération
=================================

* * * * *

[Jenkins](https://www.jenkins.io/) est un serveur d'automatisation open source écrit en Java qui aide les développeurs à créer et à tester leurs projets logiciels en continu. Il s'agit d'un système basé sur un serveur qui s'exécute dans des conteneurs de servlets tels que Tomcat. Au fil des ans, les chercheurs ont découvert diverses vulnérabilités dans Jenkins, dont certaines permettent l'exécution de code à distance sans nécessiter d'authentification. Jenkins est un serveur d'[intégration continue](https://en.wikipedia.org/wiki/Continuous_integration). Voici quelques points intéressants sur Jenkins :

- Jenkins s'appelait à l'origine Hudson (sorti en 2005) et a été renommé en 2011 après un différend avec Oracle
- [Données](https://discovery.hgdata.com/product/jenkins) montre que plus de 86 000 entreprises utilisent Jenkins
- Jenkins est utilisé par des sociétés bien connues telles que Facebook, Netflix, Udemy, Robinhood et LinkedIn
- Il dispose de plus de 300 plugins pour prendre en charge les projets de construction et de test

* * * * *

Découverte/Empreinte
----------------------

Supposons que nous travaillions sur un test d'intrusion interne et que nous ayons terminé nos analyses de découverte Web. Nous remarquons ce que nous croyons être une instance de Jenkins et savons qu'elle est souvent installée sur des serveurs Windows exécutés en tant que compte SYSTEM tout-puissant. Si nous pouvons accéder via Jenkins et obtenir l'exécution de code à distance en tant que compte SYSTEM, nous aurions un pied dans Active Directory pour commencer l'énumération de l'environnement de domaine.

Jenkins s'exécute sur le port Tomcat 8080 par défaut. Il utilise également le port 5000 pour connecter des serveurs esclaves. Ce port est utilisé pour communiquer entre les maîtres et les esclaves. Jenkins peut utiliser une base de données locale, LDAP, une base de données utilisateur Unix, déléguer la sécurité à un conteneur de servlet ou n'utiliser aucune authentification. Les administrateurs peuvent également autoriser ou interdire aux utilisateurs de créer des comptes.

* * * * *

Énumération
-----------

![](https://academy.hackthebox.com/storage/modules/113/jenkins_global_security.png)

L'installation par défaut utilise généralement la base de données de Jenkins pour stocker les informations d'identification et ne permet pas aux utilisateurs d'enregistrer un compte. Nous pouvons rapidement prendre les empreintes digitales de Jenkins grâce à la page de connexion révélatrice.

![](https://academy.hackthebox.com/storage/modules/113/jenkins_login.png)

Nous pouvons rencontrer une instance Jenkins qui utilise des informations d'identification faibles ou par défaut telles que `admin:admin` ou n'a aucun type d'authentification activé. Il n'est pas rare de trouver des instances Jenkins qui ne nécessitent aucune authentification lors d'un test d'intrusion interne. Bien que rare, nous avons rencontré Jenkins lors de tests d'intrusion externes que nous avons pu attaquer.