Proxy inversé Nginx et AJP
==========================

* * * * *

Lorsque nous rencontrons un port proxy AJP ouvert (8009 TCP), nous pouvons utiliser Nginx avec le `ajp_module`pour accéder au Tomcat Manager "caché". Cela peut être fait en compilant le code source Nginx et en ajoutant le module requis, comme suit :

-   Télécharger le code source Nginx
-   Télécharger le module requis
-   Compilez le code source Nginx avec le `ajp_module`.
-   Créer un fichier de configuration pointant vers le port AJP

#### Télécharger le code source Nginx

  Télécharger le code source Nginx

```
dsgsec@htb[/htb]$ wget https://nginx.org/download/nginx-1.21.3.tar.gz
dsgsec@htb[/htb]$ tar -xzvf nginx-1.21.3.tar.gz

```

#### Compiler le code source Nginx avec le module ajp

  Compiler le code source Nginx avec le module ajp

```
dsgsec@htb[/htb]$ git clone https://github.com/dvershinin/nginx_ajp_module.git
dsgsec@htb[/htb]$ cd nginx-1.21.3
dsgsec@htb[/htb]$ sudo apt install libpcre3-dev
dsgsec@htb[/htb]$ ./configure --add-module=`pwd`/../nginx_ajp_module --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules
dsgsec@htb[/htb]$ make
dsgsec@htb[/htb]$ sudo make install
dsgsec@htb[/htb]$ nginx -V

nginx version: nginx/1.21.3
built by gcc 10.2.1 20210110 (Debian 10.2.1-6)
configure arguments: --add-module=../nginx_ajp_module --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules

```

Remarque : Dans la configuration suivante, nous utilisons le port 8009, qui est le port par défaut de Tomcat pour AJP, et c'est ainsi que nous l'utiliserions dans un environnement réel. Cependant, pour terminer l'exercice à la fin de cette section, vous devez spécifier l'adresse IP et le port de la cible que vous allez générer (ils seront tous deux visibles juste à côté de "Cible :"). Le port que vous verrez est essentiellement mappé sur le port 8009 du conteneur Docker sous-jacent.

Commentez le `server`bloc entier et ajoutez les lignes suivantes à l'intérieur du `http`bloc dans `/etc/nginx/conf/nginx.conf`.

#### Pointant vers le port AJP

  Pointant vers le port AJP

```
upstream tomcats {
	server <TARGET_SERVER>:8009;
	keepalive 10;
	}
server {
	listen 80;
	location / {
		ajp_keep_conn on;
		ajp_pass tomcats;
	}
}

```

Remarque : Si vous utilisez Pwnbox, le port 80 sera déjà utilisé, donc, dans la configuration ci-dessus, changez le port 80 en 8080. Enfin, à l'étape suivante, utilisez le port 8080 avec cURL.

Démarrez Nginx et vérifiez si tout fonctionne correctement en envoyant une requête cURL à votre hébergeur local.

  Pointant vers le port AJP

```
dsgsec@htb[/htb]$ sudo nginx
dsgsec@htb[/htb]$ curl http://127.0.0.1:80

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Apache Tomcat/X.X.XX</title>
        <link href="favicon.ico" rel="icon" type="image/x-icon" />
        <link href="favicon.ico" rel="shortcut icon" type="image/x-icon" />
        <link href="tomcat.css" rel="stylesheet" type="text/css" />
    </head>

    <body>
        <div id="wrapper">
            <div id="navigation" class="curved container">
                <span id="nav-home"><a href="https://tomcat.apache.org/">Home</a></span>
                <span id="nav-hosts"><a href="/docs/">Documentation</a></span>
                <span id="nav-config"><a href="/docs/config/">Configuration</a></span>
                <span id="nav-examples"><a href="/examples/">Examples</a></span>
                <span id="nav-wiki"><a href="https://wiki.apache.org/tomcat/FrontPage">Wiki</a></span>
                <span id="nav-lists"><a href="https://tomcat.apache.org/lists.html">Mailing Lists</a></span>
                <span id="nav-help"><a href="https://tomcat.apache.org/findhelp.html">Find Help</a></span>
                <br class="separator" />
            </div>
            <div id="asf-box">
                <h1>Apache Tomcat/X.X.XX</h1>
            </div>
            <div id="upper" class="curved container">
                <div id="congrats" class="curved container">
                    <h2>If you're seeing this, you've successfully installed Tomcat. Congratulations!</h2>
<SNIP>

```

Proxy inverse Apache et AJP
===========================

* * * * *

Heureusement, Apache a le module AJP précompilé pour nous. Nous devrons cependant l'installer, car il ne fait pas partie des installations par défaut. La configuration de l'AJP-Proxy dans notre serveur Apache peut être effectuée comme suit :

-   Installez le paquet libapache2-mod-jk
-   Activer le module
-   Créez le fichier de configuration pointant vers le port AJP-Proxy cible

Remarque : comme mentionné dans la section précédente, le port 80 est utilisé dans Pwnbox et Apache l'utilise également comme port par défaut. Vous pouvez remplacer le port par défaut d'Apache sur "/etc/apache2/ports.conf" par n'importe quel autre port. Si vous utilisez le port 8080, n'oubliez pas d'arrêter nginx au préalable avec `sudo nginx -s stop.` Dans la configuration suivante, nous utilisons 8009, qui est le port par défaut de Tomcat pour AJP, et c'est ainsi que nous l'utiliserions dans un environnement réel. Cependant, pour terminer l'exercice à la fin de la section précédente, cette fois en utilisant Apache, vous devez spécifier l'adresse IP et le port de la cible que vous allez générer (ils seront tous deux visibles juste à côté de "Cible :"). Le port que vous verrez est essentiellement mappé sur le port 8009 du conteneur Docker sous-jacent.

Les commandes et fichiers de configuration requis sont les suivants :

```
dsgsec@htb[/htb]$ sudo apt install libapache2-mod-jk
dsgsec@htb[/htb]$ sudo a2enmod proxy_ajp
dsgsec@htb[/htb]$ sudo a2enmod proxy_http
dsgsec@htb[/htb]$ export TARGET="<TARGET_IP>"
dsgsec@htb[/htb]$ echo -n """<Proxy *>
Order allow,deny
Allow from all
</Proxy>
ProxyPass / ajp://$TARGET:8009/
ProxyPassReverse / ajp://$TARGET:8009/""" | sudo tee /etc/apache2/sites-available/ajp-proxy.conf
dsgsec@htb[/htb]$ sudo ln -s /etc/apache2/sites-available/ajp-proxy.conf /etc/apache2/sites-enabled/ajp-proxy.conf
dsgsec@htb[/htb]$ sudo systemctl start apache2

```

Remarque : La commande cURL ci-dessous est celle que vous utiliseriez normalement, car Apache écoute sur le port 80 par défaut. N'oubliez pas que vous avez dû remplacer le port 80 par un autre de votre choix. Donc, pour terminer l'exercice de la section précédente, la prochaine étape serait de spécifier le port de votre choix lors de l'utilisation de cURL, "curl http://127.0.0.1:8080" par exemple.

#### Accéder à la page Tomcat "cachée"

  Accéder à la page Tomcat "cachée"

```
dsgsec@htb[/htb]$ curl http://127.0.0.1

<SNIP>
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <title>Apache Tomcat/X.X.XX</title>
        <link href="favicon.ico" rel="icon" type="image/x-icon" />
        <link href="favicon.ico" rel="shortcut icon" type="image/x-icon" />
        <link href="tomcat.css" rel="stylesheet" type="text/css" />
    </head>

    <body>
        <div id="wrapper">
            <div id="navigation" class="curved container">
                <span id="nav-home"><a href="https://tomcat.apache.org/">Home</a></span>
                <span id="nav-hosts"><a href="/docs/">Documentation</a></span>
                <span id="nav-config"><a href="/docs/config/">Configuration</a></span>
                <span id="nav-examples"><a href="/examples/">Examples</a></span>
                <span id="nav-wiki"><a href="https://wiki.apache.org/tomcat/FrontPage">Wiki</a></span>
                <span id="nav-lists"><a href="https://tomcat.apache.org/lists.html">Mailing Lists</a></span>
                <span id="nav-help"><a href="https://tomcat.apache.org/findhelp.html">Find Help</a></span>
                <br class="separator" />
            </div>
            <div id="asf-box">
                <h1>Apache Tomcat/X.X.XX</h1>
            </div>
            <div id="upper" class="curved container">
                <div id="congrats" class="curved container">
                    <h2>If you're seeing this, you've successfully installed Tomcat. Congratulations!</h2>
                </div>
<SNIP>

```

Si nous configurons tout correctement, nous pourrons accéder au gestionnaire Apache Tomcat en utilisant à la fois cURL et notre navigateur Web.

![image](https://academy.hackthebox.com/storage/modules/145/img/tomcat.png)

[Précédent](https://academy.hackthebox.com/module/145/section/1295)

Marquer terminé et suivant

[Suivant](https://academy.hackthebox.com/module/145/section/1297)

Aide-mémoire

##### Table des matières

[Introduction aux attaques côté serveur](https://academy.hackthebox.com/module/145/section/1293)

###### Abuser des candidatures intermédiaires

[Proxy AJP](https://academy.hackthebox.com/module/145/section/1294)[  Proxy inversé Nginx et AJP](https://academy.hackthebox.com/module/145/section/1295)[Proxy inverse Apache et AJP](https://academy.hackthebox.com/module/145/section/1296)

###### Contrefaçon de requête côté serveur (SSRF)

[Présentation de la falsification de requête côté serveur (SSRF)](https://academy.hackthebox.com/module/145/section/1297)[  Exemple d'exploitation SSRF](https://academy.hackthebox.com/module/145/section/1298)[SSRF aveugle](https://academy.hackthebox.com/module/145/section/1299)[  Exemple d'exploitation SSRF en aveugle](https://academy.hackthebox.com/module/145/section/1300)[SSRF basé sur le temps](https://academy.hackthebox.com/module/145/section/1301)

###### Injection d'inclusions côté serveur (SSI)

[Vue d'ensemble des inclusions côté serveur](https://academy.hackthebox.com/module/145/section/1302)[  Exemple d'exploitation d'injection SSI](https://academy.hackthebox.com/module/145/section/1303)

###### Injection Edge-Side (ESI)

[Edge-Side Comprend (ESI)](https://academy.hackthebox.com/module/145/section/1304)

###### Injections de modèles côté serveur

[Introduction aux moteurs de modèles](https://academy.hackthebox.com/module/145/section/1305)[Identification SSTI](https://academy.hackthebox.com/module/145/section/1306)[  Exemple d'exploitation SSTI 1](https://academy.hackthebox.com/module/145/section/1307)[  Exemple d'exploitation SSTI 2](https://academy.hackthebox.com/module/145/section/1344)[  Exemple d'exploitation SSTI 3](https://academy.hackthebox.com/module/145/section/1343)

###### Transformations extensibles du langage de feuille de style Injections côté serveur

[Attaquer XSLT](https://academy.hackthebox.com/module/145/section/1308)

###### Évaluation des compétences

[  Attaques côté serveur - Évaluation des compétences](https://academy.hackthebox.com/module/145/section/1346)

##### Mon poste de travail

  Interagir   Mettre fin   Réinitialiser
