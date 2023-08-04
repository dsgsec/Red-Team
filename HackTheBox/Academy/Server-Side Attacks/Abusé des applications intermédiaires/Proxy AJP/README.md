Proxy AJP

=========

* * * * *

Selon Apache, [AJP](https://cwiki.apache.org/confluence/display/TOMCAT/Connectors) (ou JK) est un protocole filaire. Il s'agit d'une version optimisée du protocole HTTP pour permettre à un serveur Web autonome tel qu'Apache de communiquer avec Tomcat. Historiquement, Apache a été beaucoup plus rapide que Tomcat pour servir du contenu statique. L'idée est de laisser Apache servir le contenu statique lorsque cela est possible, mais de transmettre la demande à Tomcat pour le contenu lié à Tomcat.

Lorsque nous rencontrons des ports proxy AJP ouverts ( `8009 TCP`) lors de tests d'intrusion, nous pouvons être en mesure de les utiliser pour accéder au gestionnaire Apache Tomcat "caché" derrière lui. Bien qu'AJP-Proxy soit un protocole binaire, nous pouvons configurer notre propre serveur Web Nginx ou Apache avec des modules AJP pour interagir avec lui et accéder à l'application sous-jacente. De cette façon, nous pouvons découvrir des panneaux administratifs, des applications et des sites Web qui seraient autrement inaccessibles.

Pour voir comment nous pouvons configurer notre propre serveur Web Nginx ou Apache avec des modules AJP pour interagir avec un proxy AJP ouvert et accéder à l'application sous-jacente, passez à la section interactive suivante.

Remarque : Si vous souhaitez répliquer un tel environnement vulnérable sur une machine locale, vous pouvez démarrer un Apache Tomcat Docker en exposant uniquement le proxy AJP comme suit :

Tout d'abord, créez un fichier appelé `tomcat-users.xml`comprenant le fichier ci-dessous.

#### tomcat-users.xml

  tomcat-users.xml

```

<tomcat-users>

  <role rolename="manager-gui"/>

  <role rolename="manager-script"/>

  <user username="tomcat" password="s3cret" roles="manager-gui,manager-script"/>

</tomcat-users>

```

Une fois ce fichier créé, installez le package docker sur votre ordinateur local et démarrez le serveur Apache Tomcat en exécutant les commandes ci-dessous.

#### Installation Docker

  Installation Docker

```

dsgsec@htb[/htb]$ sudo apt install docker.io

dsgsec@htb[/htb]$ sudo docker run -it --rm -p 8009:8009 -v `pwd`/tomcat-users.xml:/usr/local/tomcat/conf/tomcat-users.

```
