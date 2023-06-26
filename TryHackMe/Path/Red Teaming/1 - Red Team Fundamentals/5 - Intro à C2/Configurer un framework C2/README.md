### Configurons un serveur C2

Afin de mieux comprendre ce qui est nécessaire pour configurer et administrer un serveur C2 , nous utiliserons Armitage. Pour rappel, Armitage est une interface graphique pour le framework Metasploit, et à cause de cela, il a presque tous les aspects d'un framework C2 standard.

*Remarque : Si vous utilisez l'AttackBox, vous pouvez passer à la section Préparer notre environnement.*

### Configuration d'Armitage

Télécharger, construire et installer Armitage

Tout d'abord, nous devons cloner le dépôt depuis Gitlab :

Installer Armitage

```
root@kali$ git clone https://gitlab.com/kalilinux/packages/armitage.git && cd armitage
Cloning into 'armitage'...
remote: Enumerating objects: 760, done.
remote: Counting objects: 100% (160/160), done.
remote: Compressing objects: 100% (100/100), done.
remote: Total 760 (delta 55), reused 152 (delta 54), pack-reused 600
Receiving objects: 100% (760/760), 11.81 MiB | 8.55 MiB/s, done.
Resolving deltas: 100% (244/244), done.

```

Ensuite, nous devons construire la version actuelle ; nous pouvons le faire avec la commande suivante :

Bâtiment Armitage

```
root@kali$ bash package.sh
+ ./gradlew assemble

> Task :armitage:compileJava
Note: Some input files use or override a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
Note: Some input files use unchecked or unsafe operations.
Note: Recompile with -Xlint:unchecked for details.

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.8/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 12s
6 actionable tasks: 6 executed
+ for i in unix windows mac
+ '[' unix == mac ']'
+ mkdir -p release/unix
+ cp build.txt license.txt readme.txt whatsnew.txt release/unix
+ cp build/armitage.jar build/cortana.jar release/unix
+ cp -r dist/unix/armitage dist/unix/armitage-logo.png dist/unix/teamserver release/unix
+ '[' unix == mac ']'
+ for i in unix windows mac
+ '[' windows == mac ']'
+ mkdir -p release/windows
+ cp build.txt license.txt readme.txt whatsnew.txt release/windows
+ cp build/armitage.jar build/cortana.jar release/windows
+ cp -r dist/windows/armitage.exe release/windows
+ '[' windows == mac ']'
+ for i in unix windows mac
+ '[' mac == mac ']'
++ uname
+ '[' Linux '!=' Darwin ']'
+ echo 'Skipping macOS build because this is not running on Darwin'
Skipping macOS build because this is not running on Darwin

```

Une fois le processus de construction terminé, la construction de la version sera dans le dossier ./releases/unix/. Vous devez vérifier et vérifier qu'Armitage a pu être construit avec succès.

Vérification de la construction

```
root@kali$ cd ./release/unix/ && ls -la
total 11000
drwxr-xr-x 2 root root    4096 Feb  6 20:20 .
drwxr-xr-x 4 root root    4096 Feb  6 20:20 ..
-rwxr-xr-x 1 root root      75 Feb  6 20:20 armitage
-rw-r--r-- 1 root root 4334705 Feb  6 20:20 armitage.jar
-rw-r--r-- 1 root root   25985 Feb  6 20:20 armitage-logo.png
-rw-r--r-- 1 root root     282 Feb  6 20:20 build.txt
-rw-r--r-- 1 root root 6778470 Feb  6 20:20 cortana.jar
-rw-r--r-- 1 root root    1491 Feb  6 20:20 license.txt
-rw-r--r-- 1 root root    4385 Feb  6 20:20 readme.txt
-rwxr-xr-x 1 root root    2665 Feb  6 20:20 teamserver
-rw-r--r-- 1 root root   85945 Feb  6 20:20 whatsnew.txt

```

Dans ce dossier, il y a deux fichiers clés que nous utiliserons :

Serveur d'équipe -

C'est le fichier qui démarrera le serveur Armitage auquel plusieurs utilisateurs pourront se connecter. Ce fichier prend deux arguments :

-   Adresse IP

Vos collègues opérateurs de l'équipe rouge utiliseront l'adresse IP  pour se connecter à votre serveur Armitage.

-   Mot de passe partagé

Vos collègues opérateurs de l'équipe rouge utiliseront le mot de passe partagé  pour accéder à votre serveur Armitage.

Armitage -\
C'est le fichier que vous utiliserez pour vous connecter au serveur d'équipe Armitage. Lors de l'exécution du binaire, une nouvelle invite s'ouvrira, affichant les informations de connexion et votre nom d'utilisateur (cela doit être traité comme un surnom, pas un nom d'utilisateur pour l'authentification) et mot de passe.

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/628edce404a27f170a34d3a9d9464d1c.png)

*Interface graphique des informations de connexion d'Armitage*

### Préparer notre environnement

Avant de pouvoir lancer Armitage, nous devons effectuer quelques vérifications avant le vol pour nous assurer que Metasploit est correctement configuré. Armitage s'appuie fortement sur la fonctionnalité de base de données de Metasploit, nous devons donc démarrer et initialiser la base de données avant de lancer Armitage. Pour ce faire, nous devons exécuter les commandes suivantes :

Démarrage de PostgreSQL

```
root@kali$ systemctl start postgresql && systemctl status postgresql
postgresql.service - PostgreSQL RDBMS
   Loaded: loaded (/lib/systemd/system/postgresql.service; enabled; vendor preset: enabled)
   Active: active (exited) since Sun 2022-02-06 20:16:03 GMT; 41min ago
  Process: 1587 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
 Main PID: 1587 (code=exited, status=0/SUCCESS)

Feb 06 20:16:03 ip-10-10-142-239 systemd[1]: Starting PostgreSQL RDBMS...
Feb 06 20:16:03 ip-10-10-142-239 systemd[1]: Started PostgreSQL RDBMS.

```

Enfin, nous devons initialiser la base de données pour que Metasploit puisse l'utiliser. Il est important de noter que vous ne pouvez pas être l'utilisateur root lorsque vous tentez d'initialiser la base de données Metasploit. Sur l' AttackBox,  vous devez utiliser l' utilisateur Ubuntu .

Initialisation de la base de données PostgreSQL

```
user@kali$ msfdb --use-defaults delete
Stopping database at /home/ubuntu/.msf4/db
Deleting all data at /home/ubuntu/.msf4/db
MSF web service is no longer running

user@kali$ msfdb --use-defaults init
Creating database at /home/ubuntu/.msf4/db
Starting database at /home/ubuntu/.msf4/db...success
Creating database users
Writing client authentication configuration file /home/ubuntu/.msf4/db/pg_hba.conf
Stopping database at /home/ubuntu/.msf4/db
Starting database at /home/ubuntu/.msf4/db...success
Creating initial database schema
Generating SSL key and certificate for MSF web service
Attempting to start MSF web service...failed
[!] MSF web service does not appear to be started.
Please see /home/ubuntu/.msf4/logs/msf-ws.log for additional details.

```

Une fois l'initialisation terminée, nous pouvons enfin démarrer Armitage Team Server. 

### Démarrage et connexion à Armitage

Démarrage du serveur d'équipe d'Armitage

```
root@kali$ cd /opt/armitage/release/unix && ./teamserver YourIP P@ssw0rd123
[*] Generating X509 certificate and keystore (for SSL)
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
[*] Starting RPC daemon
[*] MSGRPC starting on 127.0.0.1:55554 (NO SSL):Msg...
[*] MSGRPC backgrounding at 2022-02-06 17:47:08 -0500...
[*] MSGRPC background PID 2083
[*] sleeping for 20s (to let msfrpcd initialize)
[*] Starting Armitage team server
Picked up _JAVA_OPTIONS: -Dawt.useSystemAAFontSettings=on -Dswing.aatext=true
[*] Use the following connection details to connect your clients:
        Host: 127.0.0.2
        Port: 55553
        User: msf
        Pass: P@ssw0rd123

[*] Fingerprint (check for this string when you connect):
        d211e51c8886113433f63b588fd5ccfc9e323059
[+] hacking is such a lonely thing, until now

```

Une fois que votre Teamserver est opérationnel, nous pouvons maintenant démarrer le client Armitage. Ceci est utilisé pour se connecter au Teamserver et affiche l'interface graphique à l'utilisateur.

Armitage de départ

```
root@kali$ cd /opt/armitage/release/unix && ./armitage
[*] Used the incumbent: 10.10.69.193
[*] Starting Cortana on 10.10.69.193
[*] Starting Cortana on 10.10.69.193
[*] Creating a default reverse handler... 0.0.0.0:8836

```

Lorsque vous utilisez un framework C2 , vous ne souhaitez jamais exposer publiquement l'interface de gestion ; Vous devez toujours écouter sur une interface locale, jamais sur une interface publique. Cela complique l'accès pour les autres opérateurs. Heureusement, il existe une solution simple pour cela. Pour que les opérateurs puissent accéder au serveur, vous devez créer un nouveau compte utilisateur pour eux et activer l'accès SSH sur le serveur, et ils pourront transférer le port SSH TCP/55553. Armitage refuse explicitement aux utilisateurs d'écouter sur 127.0.0.1 ; c'est parce qu'il s'agit essentiellement d'un serveur Metasploit partagé avec un "serveur de déconfliction" que lorsque plusieurs utilisateurs se connectent au serveur, vous ne voyez pas tout ce que vos autres utilisateurs voient. Avec Armitage, vous devez écouter sur votre adresse IP tun0/eth0.

![250523e0ea93b10e42c3515c6b812609](https://github.com/dsgsec/Red-Team/assets/82456829/27d88f56-4330-4c81-a832-2bdc5432fdd2)

*Modifiez l'adresse IP de l'hôte en fonction de ce que vous avez défini à l'étape précédente, "Démarrage du serveur d'équipe d'Armitage".*

Après avoir cliqué sur "Se connecter", vous serez invité à entrer un surnom. Vous pouvez définir ce que vous voulez ; seuls vos collègues opérateurs de l'équipe rouge le verront.

![67465eb4ae28c1f7d57452b6b632314c](https://github.com/dsgsec/Red-Team/assets/82456829/e155e8c5-204e-4b69-a91d-e434430cab1f)

*L'interface utilisateur d'Armitage pour mettre un surnom personnalisé*

Après un moment ou deux, l'interface utilisateur d'Armitage devrait s'ouvrir, jusqu'à ce que nous commencions à interagir avec des systèmes distants ; il aura l'air nu. Dans la prochaine tâche à venir, nous exploiterons une machine virtuelle vulnérable pour vous familiariser davantage avec l'interface utilisateur d'Armitage et son utilisation.

![fca62d124845af393a71e38e9eb94edb](https://github.com/dsgsec/Red-Team/assets/82456829/c768fcb5-e54c-40a4-a48f-8571043900aa)

*L'interface utilisateur d'Armitage*

Maintenant qu'Armitage est configuré et fonctionne correctement, dans la tâche suivante, nous en apprendrons davantage sur l'accès sécurisé à Armitage (comme décrit ci-dessus), la création d'écouteurs, divers types d'écouteurs, la génération de charges utiles, et bien plus encore !
