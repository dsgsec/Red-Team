Obtention d'identifiants de session sans interaction de l'utilisateur
=====================================================================

* * * * *

Il existe plusieurs techniques d'attaque grâce auxquelles un chasseur de primes de bogues ou un testeur d'intrusion peut obtenir des identifiants de session. Ces techniques d'attaque peuvent être divisées en deux catégories :

1.  Attaques d'obtention d'ID de session sans interaction de l'utilisateur
2.  Attaques d'obtention d'ID de session nécessitant une interaction de l'utilisateur

Cette section se concentrera sur les attaques d'obtention d'ID de session qui ne nécessitent pas d'interaction de l'utilisateur.

* * * * *

Obtention d'identifiants de session via le reniflage de trafic
--------------------------------------------------------------

Le reniflage de trafic est quelque chose que la plupart des testeurs d'intrusion font lorsqu'ils évaluent la sécurité d'un réseau de l'intérieur. Vous les verrez généralement brancher leurs ordinateurs portables ou Raspberry Pis sur les prises Ethernet disponibles. Cela leur permet de surveiller le trafic et leur donne une idée du trafic passant par le réseau (segment) et des services qu'ils peuvent attaquer. Le reniflage de trafic nécessite que l'attaquant et la victime soient sur le même réseau local. C'est alors et seulement alors que le trafic HTTP peut être inspecté par l'attaquant. Il est impossible pour un attaquant d'effectuer un sniffing de trafic à distance.

Vous avez peut-être remarqué que nous avons mentionné le trafic HTTP. En effet, HTTP est un protocole qui transfère des données non chiffrées. Ainsi, si un attaquant surveille le réseau, il peut capturer toutes sortes d'informations telles que les noms d'utilisateur, les mots de passe et même les identifiants de session. Ce type d'informations sera plus difficile et, la plupart du temps, impossible à obtenir si le trafic HTTP est crypté via SSL ou IPsec.

En résumé, l'obtention d'identifiants de session via le sniffing de trafic nécessite :

-   L'attaquant doit être positionné sur le même réseau local que la victime
-   Trafic HTTP non chiffré

Il existe de nombreux outils de reniflage de paquets. Dans ce module, nous utiliserons [Wireshark](https://www.wireshark.org/) . Wireshark a une fonction de filtre intégrée qui permet aux utilisateurs de filtrer le trafic pour un protocole spécifique tel que HTTP, SSH, FTP et même pour une adresse IP source particulière.

Entraînons-nous au détournement de session via le reniflage de trafic contre une application Web. Cette application Web est la cible que nous pouvons générer lors de l'exercice à la fin de cette section.

Naviguez jusqu'à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la. N'oubliez pas de configurer le vhost ( `xss.htb.net`) spécifié pour accéder à l'application.

Un moyen rapide de spécifier ce vhost (et tout autre) dans votre système d'attaque est le suivant :

```
dsgsec@htb[/htb]$ IP=ENTER SPAWNED TARGET IP HERE
dsgsec@htb[/htb]$ printf "%s\t%s\n\n" "$IP" "xss.htb.net csrf.htb.net oredirect.htb.net minilab.htb.net" | sudo tee -a /etc/hosts

```

Partie 1 : Simuler l'attaquant

Accédez à `http://xss.htb.net`et, à l'aide des outils de développement Web (Maj + Ctrl + I dans le cas de Firefox), notez que l'application utilise un cookie nommé `auth-session`très probablement comme identifiant de session. ![image](https://academy.hackthebox.com/storage/modules/153/3.png)

Lancez maintenant Wireshark pour commencer à renifler le trafic sur le réseau local comme suit.

```
dsgsec@htb[/htb]$ sudo -E wireshark

```

Vous rencontrerez ci-dessous. ![image](https://academy.hackthebox.com/storage/modules/153/1.png)

Faites un clic droit sur "tun0" puis cliquez sur "Démarrer la capture"

Partie 2 : Simuler la victime

Accédez à `http://xss.htb.net`et `New Private Window`connectez-vous à l'application en utilisant les informations d'identification ci-dessous :

-   Courriel : heavycat106
-   Mot de passe : rocknrol

Il s'agit d'un compte que nous avons créé pour examiner l'application !

Partie 3 : Obtenir le cookie de la victime grâce à l'analyse des paquets

Dans Wireshark, appliquez d'abord un filtre pour voir uniquement le trafic HTTP. Cela peut être fait comme suit (n'oubliez pas d'appuyer sur Entrée après avoir spécifié le filtre). ![image](https://academy.hackthebox.com/storage/modules/153/2.png)

Recherchez maintenant dans les octets du paquet tous `auth-session`les cookies comme suit.

Accédez à `Edit`->`Find Packet` ![image](https://academy.hackthebox.com/storage/modules/153/4.png)

Faites un clic gauche sur `Packet list`puis cliquez sur`Packet bytes` ![image](https://academy.hackthebox.com/storage/modules/153/5.png)

Sélectionnez `String`dans le troisième menu déroulant et spécifiez `auth-session`dans le champ à côté. Enfin, cliquez sur `Find`. Wireshark vous présentera les paquets contenant une `auth-session`chaîne. ![image](https://academy.hackthebox.com/storage/modules/153/6.png)

Le cookie peut être copié en cliquant avec le bouton droit sur une ligne qui le contient, puis en cliquant sur `Copy`et enfin en cliquant sur `Value`. ![image](https://academy.hackthebox.com/storage/modules/153/8.png)

Partie 4 : détourner la session de la victime

Revenez à la fenêtre du navigateur à l'aide de laquelle vous avez parcouru l'application pour la première fois (et non la fenêtre privée), ouvrez les outils de développement Web, accédez à *stockage* et remplacez la valeur de votre cookie actuel par celle que vous avez obtenue via Wireshark (n'oubliez pas de supprimer la `auth-session=`partie). ![image](https://academy.hackthebox.com/storage/modules/153/9.png)

Si vous actualisez la page, vous verrez que vous êtes maintenant connecté en tant que victime ! ![image](https://academy.hackthebox.com/storage/modules/153/10.png)

* * * * *

Obtention d'identifiants de session après exploitation (accès au serveur Web)
-----------------------------------------------------------------------------

Remarque : Les exemples ci-dessous ne peuvent pas être reproduits dans l'exercice de laboratoire de cette section !

Pendant la phase de post-exploitation, les identifiants de session et les données de session peuvent être récupérés à partir du disque ou de la mémoire d'un serveur Web. Bien sûr, un attaquant qui a compromis un serveur Web peut faire plus qu'obtenir des données de session et des identifiants de session. Cela dit, un attaquant peut ne pas vouloir continuer à émettre des commandes qui augmentent les chances de se faire prendre.

* * * * *

### PHP

Regardons où les identifiants de session PHP sont généralement stockés.

L'entrée `session.save_path`dans `PHP.ini`spécifie où les données de session seront stockées.

```
dsgsec@htb[/htb]$ locate php.ini
dsgsec@htb[/htb]$ cat /etc/php/7.4/cli/php.ini | grep 'session.save_path'
dsgsec@htb[/htb]$ cat /etc/php/7.4/apache2/php.ini | grep 'session.save_path'

```

![image](https://academy.hackthebox.com/storage/modules/153/11.png)

Dans notre cas de configuration par défaut, c'est `/var/lib/php/sessions`. Maintenant, veuillez noter qu'une victime doit être authentifiée pour que nous puissions voir son identifiant de session. Les fichiers qu'un attaquant recherchera utilisent la convention de nom `sess_<sessionID>`.

A quoi ressemble un identifiant de session PHP sur notre configuration locale. ![image](https://academy.hackthebox.com/storage/modules/153/12.png)

Le même identifiant de session PHP mais côté serveur Web ressemble à ceci.

```
dsgsec@htb[/htb]$ ls /var/lib/php/sessions
dsgsec@htb[/htb]$ cat //var/lib/php/sessions/sess_s6kitq8d3071rmlvbfitpim9mm

```

![image](https://academy.hackthebox.com/storage/modules/153/13.png)

Comme déjà mentionné, pour qu'un pirate détourne la session utilisateur liée à l'identifiant de session ci-dessus, un nouveau cookie doit être créé dans le navigateur Web avec les valeurs suivantes :

-   nom du cookie : PHPSESSID
-   valeur du cookie : s6kitq8d3071rmlvbfitpim9mm

* * * * *

### Java

Voyons maintenant où sont stockés les identifiants de session Java.

Selon l'Apache Software Foundation :

"L' `Manager`élément représente le *gestionnaire de session* utilisé pour créer et gérer les sessions HTTP d'une application Web.

Tomcat fournit deux implémentations standard de `Manager`. L'implémentation par défaut stocke les sessions actives, tandis que l'optionnelle stocke les sessions actives qui ont été échangées (en plus d'enregistrer les sessions lors d'un redémarrage du serveur) dans un emplacement de stockage sélectionné via l'utilisation d'un élément imbriqué approprié `Store`. Le nom de fichier du fichier de données de session par défaut est `SESSIONS.ser`."

Vous pouvez trouver plus d'informations [ici](http://tomcat.apache.org/tomcat-6.0-doc/config/manager.html) .

* * * * *

### .FILET

Enfin, regardons où sont stockés les identifiants de session .NET.

Les données de session peuvent être trouvées dans :

-   Le processus de travail d'application (aspnet_wp.exe) - C'est le cas en *mode InProc Session*
-   StateServer (un service Windows résidant sur IIS ou un serveur séparé) - C'est le cas en *mode OutProc Session*
-   Un serveur SQL

Veuillez vous référer à la ressource suivante pour plus de détails : [Introduction aux sessions ASP.NET](https://www.c-sharpcorner.com/UploadFile/225740/introduction-of-session-in-Asp-Net/)

* * * * *

Obtention des identifiants de session après exploitation (accès à la base de données)
-------------------------------------------------------------------------------------

Dans les cas où vous avez un accès direct à une base de données via, par exemple, une injection SQL ou des informations d'identification identifiées, vous devez toujours rechercher les sessions utilisateur stockées. Voir un exemple ci-dessous.

Code: sql

```
show databases;
use project;
show tables;
select * from users;

```

![image](https://academy.hackthebox.com/storage/modules/153/14.png)

Ici, nous pouvons voir que les mots de passe des utilisateurs sont hachés. Nous pourrions passer du temps à essayer de les casser ; cependant, il existe également une table "all_sessions". Extrayons les données de cette table.

Code: sql

```
select * from all_sessions;
select * from all_sessions where id=3;

```

![image](https://academy.hackthebox.com/storage/modules/153/15.png)

Ici, nous avons réussi à extraire les sessions ! Vous pouvez maintenant vous authentifier en tant qu'utilisateur "Développeur".

Il est temps que nous couvrons les attaques d'obtention d'ID de session nécessitant une interaction de l'utilisateur. Dans les sections suivantes, nous aborderons :

-   XSS (Cross-Site Scripting) <-- En mettant l'accent sur les sessions utilisateur
-   CSRF (Cross-Site Request Forgery)
-   Redirections ouvertes <-- En mettant l'accent sur les sessions utilisateur
