Introduction à l'attaque d'applications courantes
============================================

* * * * *

Les applications Web sont répandues dans la plupart sinon tous les environnements que nous rencontrons en tant que testeurs d'intrusion. Au cours de nos évaluations, nous rencontrerons une grande variété d'applications Web telles que les systèmes de gestion de contenu (CMS), les applications Web personnalisées, les portails intranet utilisés par les développeurs et les administrateurs système, les référentiels de code, les outils de surveillance du réseau, les systèmes de billetterie, les wikis, les bases de connaissances, traqueurs de problèmes, applications de conteneur de servlets, etc. Il est courant de trouver les mêmes applications dans de nombreux environnements différents. Bien qu'une application puisse ne pas être vulnérable dans un environnement, elle peut être mal configurée ou non corrigée dans le suivant. Un évaluateur doit avoir une solide compréhension de l'énumération et de l'attaque des applications courantes couvertes dans ce module.

Les applications Web sont des applications interactives accessibles via des navigateurs Web. Les applications Web adoptent généralement une architecture client-serveur pour exécuter et gérer les interactions. Ils sont généralement constitués de composants frontaux (l'interface du site Web ou "ce que l'utilisateur voit") qui s'exécutent côté client (navigateur) et d'autres composants principaux (code source de l'application Web) qui s'exécutent sur le serveur. -côté (serveur principal/bases de données). Pour une étude approfondie de la structure et de la fonction des applications Web, consultez le module [Introduction aux applications Web](https://academy.hackthebox.com/course/preview/introduction-to-web-applications) .

Tous les types d'applications Web (commerciales, open source et personnalisées) peuvent souffrir des mêmes types de vulnérabilités et de mauvaises configurations, à savoir les 10 principaux risques liés aux applications Web couverts dans le [OWASP Top 10](https://owasp.org/ www-project-top-dix/). Bien que nous puissions rencontrer des versions vulnérables de nombreuses applications courantes qui souffrent de vulnérabilités connues (publiques) telles que l'injection SQL, XSS, les bogues d'exécution de code à distance, la lecture de fichiers locaux et le téléchargement de fichiers sans restriction, il est tout aussi important pour nous de comprendre comment nous pouvons abusent des fonctionnalités intégrées de bon nombre de ces applications pour réaliser l'exécution de code à distance.

Alors que les organisations continuent de renforcer leur périmètre externe et de limiter les services exposés, les applications Web deviennent une cible plus attrayante pour les acteurs malveillants et les testeurs d'intrusion. De plus en plus d'entreprises passent au travail à distance et exposent (intentionnellement ou non) des applications au monde extérieur. Les applications abordées dans ce module sont généralement tout aussi susceptibles d'être exposées sur le réseau externe que sur le réseau interne. Ces applications peuvent servir de prise de pied dans l'environnement interne lors d'une évaluation externe ou de prise de pied, de mouvement latéral ou de problème supplémentaire à signaler à notre client lors d'une évaluation interne.

[L'état de la sécurité des applications en 2021](https://blog.barracuda.com/2021/05/18/report-the-state-of-application-security-in-2021/) est une enquête commandée par Barracuda pour recueillir des informations auprès des décideurs liés à la sécurité des applications. L'enquête comprend les réponses de 750 décideurs d'entreprises de 500 employés ou plus à travers le monde. Les résultats de l'enquête étaient étonnants : 72 % des personnes interrogées ont déclaré que leur organisation avait subi au moins une violation en raison d'une vulnérabilité d'application, 32 % avaient subi deux violations et 14 % en avaient subi trois. Les organisations interrogées ont réparti leurs défis comme suit : attaques de robots (43 %), attaques de la chaîne d'approvisionnement logicielle (39 %), détection de vulnérabilités (38 %) et sécurisation des API (37 %). Ce module se concentrera sur les vulnérabilités connues et les erreurs de configuration dans les applications open source et commerciales (versions gratuites présentées dans ce module), qui représentent un pourcentage important des attaques réussies auxquelles les organisations sont régulièrement confrontées.

* * * * *

Application Data
----------------

Ce module étudiera en profondeur plusieurs applications courantes tout en couvrant brièvement certaines autres moins courantes (mais toujours vues souvent). Voici quelques-unes des catégories d'applications que nous pouvons rencontrer lors d'une évaluation donnée et que nous pouvons exploiter pour prendre pied ou accéder à des données sensibles :

| Catégorie | Candidatures |
| --- | --- |
| [Gestion de contenu Web](https://enlyft.com/tech/web-content-management) | Joomla, Drupal, WordPress, DotNetNuke, etc. |
| [Serveurs d'applications](https://enlyft.com/tech/application-servers) | Apache Tomcat, Phusion Passenger, Oracle WebLogic, IBM WebSphere, etc. |
| [Informations de sécurité et gestion des événements (SIEM)](https://enlyft.com/tech/security-information-and-event-management-siem) | Splunk, Trustwave, LogRhythm, etc. |
| [Gestion du réseau](https://enlyft.com/tech/network-management) | PRTG Network Monitor, ManageEngine Opmanger, etc. |
| [Gestion informatique](https://enlyft.com/tech/it-management-software) | Nagios, Puppet, Zabbix, ManageEngine ServiceDesk Plus, etc. |
| [Cadres logiciels](https://enlyft.com/tech/software-frameworks) | JBoss, Axis2, etc. |
| [Gestion du service client] (https://enlyft.com/tech/customer-service-management) | osTicket, Zendesk, etc. |
| [Moteurs de recherche](https://enlyft.com/tech/search-engines) | Elasticsearch, Apache Solr, etc. |
| [Gestion de la configuration logicielle](https://enlyft.com/tech/software-configuration-management) | Atlassian JIRA, GitHub, GitLab, Bugzilla, Bugsnag, Bitbucket, etc. |
| [Outils de développement de logiciels](https://enlyft.com/tech/software-development-tools) | Jenkins, Atlassian Confluence, phpMyAdmin, etc. |
| [Intégration d'applications d'entreprise](https://enlyft.com/tech/enterprise-application-integration) | Oracle Fusion Middleware, BizTalk Server, Apache ActiveMQ, etc. |

Comme vous pouvez le voir en parcourant les liens pour chaque catégorie ci-dessus, il y a [des milliers d'applications](https://enlyft.com/tech/) que nous pouvons rencontrer lors d'une évaluation donnée. Beaucoup d'entre eux souffrent d'exploits connus du public ou ont des fonctionnalités qui peuvent être utilisées de manière abusive pour obtenir l'exécution de code à distance, voler des informations d'identification ou accéder à des informations sensibles avec ou sans informations d'identification valides. Ce module couvrira les applications les plus répandues que nous voyons à plusieurs reprises lors des évaluations internes et externes.

Jetons un coup d'œil au site Web d'Enlyft. Nous pouvons voir, par exemple, qu'ils ont pu collecter des données sur plus de 3,7 millions d'entreprises qui utilisent [WordPress](https://enlyft.com/tech/products/wordpress) qui représente près de 70 % de la part de marché dans le monde. pour les applications de gestion de contenu Web pour toutes les entreprises interrogées. Pour l'outil SIEM [Splunk](https://enlyft.com/tech/products/splunk) était utilisé par 22 174 des entreprises interrogées et représentait près de 30 % de la part de marché des outils SIEM. Alors que les applications restantes que nous couvrirons représentent une part de marché beaucoup plus petite pour leur catégorie respective, je les vois encore souvent, et les compétences acquises ici peuvent être appliquées à de nombreuses situations différentes.

Tout en travaillant sur les exemples de section, les questions et les évaluations de compétences, faites un effort concerté pour apprendre comment ces applications fonctionnent et *pourquoi* des vulnérabilités et des erreurs de configuration spécifiques existent plutôt que de simplement reproduire les exemples pour parcourir rapidement le module. Ces compétences vous seront très utiles et pourraient probablement vous aider à identifier les chemins d'attaque dans différentes applications que vous rencontrez lors d'une évaluation pour la première fois. Je rencontre encore des applications que je n'ai vues que quelques fois ou jamais auparavant, et les aborder avec cet état d'esprit m'a souvent aidé à réussir des attaques ou à trouver un moyen d'abuser des fonctionnalités intégrées.

* * * * *

Une histoire rapide
--------------

Par exemple, lors d'un test d'intrusion externe, j'ai rencontré l'[application Nexus Repository OSS](https://www.sonatype.com/products/repository-oss) de Sonatype, que je n'avais jamais vue auparavant. J'ai rapidement constaté que les informations d'identification d'administrateur par défaut de `admin:admin123` pour cette version n'avaient pas été modifiées, et j'ai pu me connecter et parcourir la fonctionnalité d'administration. Dans cette version, j'ai exploité l'API en tant qu'utilisateur authentifié pour obtenir l'exécution de code à distance sur le système. J'ai rencontré cette application lors d'une autre évaluation, j'ai pu me connecter à nouveau avec les informations d'identification par défaut. Cette fois, j'ai pu abuser de la fonctionnalité [Tasks](https://help.sonatype.com/repomanager3/system-configuration/tasks#Tasks-Admin-Executescript) (qui était désactivée la première fois que j'ai rencontré cette application) et écrire un rapide [Groovy](https://groovy-lang.org/) [script](https://help.sonatype.com/repomanager3/rest-and-integration-api/script-api/writing-scripts) dans Syntaxe Java pour exécuter un script et obtenir l'exécution de code à distance. Ceci est similaire à la façon dont nous utiliserons Jenkins [script console](https://www.jenkins.io/doc/book/managing/script-console/) plus loin dans ce module. J'ai rencontré de nombreuses autres applications, telles que [OpManager](https://www.manageengine.com/products/applications_manager/me-opmanager-monitoring.html) de ManageEngine qui vous permettent d'exécuter un script en tant qu'utilisateur auquel l'application s'exécute sous (généralement le puissant compte NT AUTHORITY\SYSTEM) et prendre pied. Nous ne devons jamais négliger les candidatures lors d'une évaluation interne et externe car elles peuvent être notre seule voie "d'entrée" dans un environnement relativement bien entretenu.

* * * * *

Applications courantes
-------------------

Je rencontre généralement au moins une des applications ci-dessous, que nous couvrirons en profondeur tout au long des sections du module. Bien que nous ne puissions pas couvrir toutes les applications possibles que nous pourrions rencontrer, les compétences enseignées dans ce module nous prépareront à aborder toutes les applications avec un œil critique et à les évaluer pour les vulnérabilités publiques et les erreurs de configuration.

| Candidature | Descriptif |
| --- | --- |
| WordPress | [WordPress](https://wordpress.org/) est un système de gestion de contenu (CMS) open source qui peut être utilisé à plusieurs fins. Il est souvent utilisé pour héberger des blogs et des forums. WordPress est hautement personnalisable et convivial pour le référencement, ce qui le rend populaire parmi les entreprises. Cependant, sa personnalisationy et sa nature extensible le rendent vulnérable aux vulnérabilités via des thèmes et des plugins tiers. WordPress est écrit en PHP et fonctionne généralement sur Apache avec MySQL comme backend. |
| Drupal | [Drupal](https://www.drupal.org/) est un autre CMS open source très apprécié des entreprises et des développeurs. Drupal est écrit en PHP et prend en charge l'utilisation de MySQL ou PostgreSQL pour le backend. De plus, SQLite peut être utilisé s'il n'y a pas de SGBD installé. Comme WordPress, Drupal permet aux utilisateurs d'améliorer leurs sites Web grâce à l'utilisation de thèmes et de modules. |
| Joomla | [Joomla](https://www.joomla.org/) est un autre CMS open source écrit en PHP qui utilise généralement MySQL, mais qui peut être exécuté avec PostgreSQL ou SQLite. Joomla peut être utilisé pour les blogs, les forums de discussion, le commerce électronique, etc. Joomla peut être fortement personnalisé avec des thèmes et des extensions et est estimé être le troisième CMS le plus utilisé sur Internet après WordPress et Shopify. |
| chat | [Apache Tomcat](https://tomcat.apache.org/) est un serveur Web open source qui héberge des applications écrites en Java. Tomcat a été initialement conçu pour exécuter des scripts Java Servlets et Java Server Pages (JSP). Cependant, sa popularité a augmenté avec les frameworks basés sur Java et est maintenant largement utilisée par des frameworks tels que Spring et des outils tels que Gradle. |
| Jenkins | [Jenkins](https://jenkins.io/) est un serveur d'automatisation open source écrit en Java qui aide les développeurs à créer et à tester leurs projets logiciels en continu. Il s'agit d'un système basé sur un serveur qui s'exécute dans des conteneurs de servlets tels que Tomcat. Au fil des ans, les chercheurs ont découvert diverses vulnérabilités dans Jenkins, dont certaines permettent l'exécution de code à distance sans nécessiter d'authentification. |
| Splunk | Splunk est un outil d'analyse de journaux utilisé pour collecter, analyser et visualiser des données. Bien qu'il ne soit pas initialement destiné à être un outil SIEM, Splunk est souvent utilisé pour la surveillance de la sécurité et l'analyse commerciale. Les déploiements Splunk sont souvent utilisés pour héberger des données sensibles et pourraient fournir une mine d'informations à un attaquant en cas de compromission. Historiquement, Splunk n'a pas souffert d'un nombre considérable de vulnérabilités connues en dehors d'une vulnérabilité de divulgation d'informations ([CVE-2018-11409](https://nvd.nist.gov/vuln/detail/CVE-2018-11409)), et une vulnérabilité d'exécution de code à distance authentifiée dans les très anciennes versions ([CVE-2011-4642](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-4642)). |
| Moniteur réseau PRTG | [PRTG Network Monitor](https://www.paessler.com/prtg) est un système de surveillance réseau sans agent qui peut être utilisé pour surveiller des métriques telles que la disponibilité, l'utilisation de la bande passante, etc. à partir d'une variété d'appareils tels que des routeurs, des commutateurs , serveurs, etc. Il utilise un mode de découverte automatique pour analyser un réseau, puis exploite des protocoles tels que ICMP, WMI, SNMP et NetFlow pour communiquer avec et collecter des données à partir des périphériques découverts. PRTG est écrit en [Delphi](https://en.wikipedia.org/wiki/Delphi_(software)). |
| osTicket | [osTicket](https://osticket.com/) est un système de ticket d'assistance open source largement utilisé. Il peut être utilisé pour gérer les tickets de service client reçus par e-mail, téléphone et interface Web. osTicket est écrit en PHP et peut fonctionner sur Apache ou IIS avec MySQL comme backend. |
| GitLab | [GitLab](https://about.gitlab.com/) est une plate-forme de développement de logiciels open source avec un gestionnaire de référentiel Git, un contrôle des versions, un suivi des problèmes, une révision du code, une intégration et un déploiement continus, et bien plus encore. Il a été initialement écrit en Ruby mais utilise maintenant Ruby on Rails, Go et Vue.js. GitLab propose à la fois des versions communautaires (gratuites) et d'entreprise du logiciel. |

* * * * *

Cibles des modules
--------------

Tout au long des sections du module, nous ferons référence à des URL telles que `http://app.inlanefreight.local`. Pour simuler un environnement vaste et réaliste avec plusieurs serveurs Web, nous utilisons des Vhosts pour héberger les applications Web. Étant donné que ces Vhosts correspondent tous à un répertoire différent sur le même hôte, nous devons effectuer des entrées manuelles dans notre fichier `/etc/hosts` sur la Pwnbox ou la VM d'attaque locale pour interagir avec le laboratoire. Cela doit être fait pour tous les exemples qui montrent des analyses ou des captures d'écran utilisant un FQDN. Les sections telles que Splunk qui utilisent uniquement l'adresse IP de la cible générée ne nécessiteront pas d'entrée de fichier hosts, et vous pouvez simplement interagir avec l'adresse IP générée et le port associé.

Pour faire cela rapidement, nous pourrions exécuter ce qui suit :

```
dsgsec@htb[/htb]$ IP=10.129.42.195
dsgsec@htb[/htb]$ printf "%s\t%s\n\n" "$IP" "app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local" | sudo tee -a /etc/hosts

```

Après cette commande, notre fichier `/etc/hosts` ressemblera à ceci (sur une Pwnbox nouvellement générée) :

```
dsgsec@htb[/htb]$ cat /etc/hosts

# Votre système a configuré 'manage_etc_hosts' comme True.
# Par conséquent, si vous souhaitez que les modifications apportées à ce fichier persistent
# alors vous devrez soit
# a.) apporter des modifications au fichier maître dans /etc/cloud/templates/hosts.debian.tmpl
# b.) modifier ou supprimer la valeur de 'manage_etc_hosts' dans
# /etc/cloud/cloud.cfg ou cloud-config à partir des données utilisateur
#
127.0.1.1 htb-9zftpkslke.htb-cloud.com htb-9zftpkslke
127.0.0.1 hôte local

# Les lignes suivantes sont souhaitables pour les hôtes compatibles IPv6
::1 ip6-localhost ip6-loopback
fe00 :: 0 ip6-localnet
ff00 :: 0 ip6-mcastprefix
ff02 :: 1 ip6-tous les nœuds
ff02 :: 2 ip6-allrouters
ff02 :: 3 ip6-allhosts

10.129.42.195 app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local

```

Vous pouvez écrire votre propre script ou éditer le fichier hosts à la main, ce qui est bien.

Si vous générez une cible pendant une section et que vous ne pouvez pas y accéder directement via l'IP, assurez-vous de vérifier votre fichier hosts et de mettre à jour toutes les entrées !

Les exercices de module qui nécessitent des vhosts afficheront une liste que vous pouvez utiliser pour modifier votre fichier hosts après avoir généré la machine virtuelle cible au bas de la section respective.