# Enumération des services

## SSH

Secure Shell (SSH) permet à deux ordinateurs d'établir une connexion cryptée et directe au sein d'un réseau éventuellement non sécurisé sur le port standard TCP 22. Cela est nécessaire pour empêcher des tiers d'intercepter le flux de données et donc d'intercepter des données sensibles. Le serveur SSH peut également être configuré pour n'autoriser que les connexions de clients spécifiques. Un avantage de SSH est que le protocole fonctionne sur tous les systèmes d'exploitation courants. Puisqu'il s'agit à l'origine d'une application Unix, elle est également implémentée nativement sur toutes les distributions Linux et MacOS. SSH peut également être utilisé sous Windows, à condition d'installer un programme approprié. Le serveur bien connu OpenBSD SSH (OpenSSH) sur les distributions Linux est un fork open-source du serveur SSH original et commercial de SSH Communication Security. En conséquence, il existe deux protocoles concurrents : SSH-1 et SSH-2.

SSH-2, également connu sous le nom de SSH version 2, est un protocole plus avancé que SSH version 1 en termes de cryptage, de vitesse, de stabilité et de sécurité. Par exemple, SSH-1 est vulnérable aux attaques MITM, alors que SSH-2 ne l'est pas.

Nous pouvons imaginer que nous voulons gérer un hôte distant. Cela peut être fait via la ligne de commande ou l'interface graphique. En outre, nous pouvons également utiliser le protocole SSH pour envoyer des commandes au système souhaité, transférer des fichiers ou effectuer une redirection de port. Par conséquent, nous devons nous y connecter en utilisant le protocole SSH et nous authentifier auprès de lui. Au total, OpenSSH dispose de six méthodes d'authentification différentes:

+ Authentification par mot de passe
+ Authentification par clé publique
+ Authentification basée sur l'hôte
+ Authentification du clavier
+ Authentification défi-réponse
+ Authentification GSSAPI

Nous allons examiner de plus près et discuter de l'une des méthodes d'authentification les plus couramment utilisées. De plus, nous pouvons en apprendre plus sur les autres méthodes d'authentification ici entre autres.

### Authentification par clé publique
Dans un premier temps, le serveur SSH et le client s'authentifient l'un auprès de l'autre. Le serveur envoie un certificat au client pour vérifier qu'il s'agit du bon serveur. Ce n'est qu'au moment de l'établissement du contact qu'il existe un risque qu'un tiers s'interpose entre les deux participants et intercepte ainsi la connexion. Étant donné que le certificat lui-même est également crypté, il ne peut pas être imité. Une fois que le client connaît le bon certificat, personne d'autre ne peut prétendre prendre contact via le serveur correspondant.

Cependant, après l'authentification du serveur, le client doit également prouver au serveur qu'il dispose d'une autorisation d'accès. Cependant, le serveur SSH est déjà en possession de la valeur de hachage cryptée du mot de passe défini pour l'utilisateur souhaité. Par conséquent, les utilisateurs doivent entrer le mot de passe chaque fois qu'ils se connectent à un autre serveur au cours de la même session. Pour cette raison, une option alternative pour l'authentification côté client est l'utilisation d'une clé publique et d'une paire de clés privées.

La clé privée est créée individuellement pour le propre ordinateur de l'utilisateur et sécurisée avec une phrase de passe qui doit être plus longue qu'un mot de passe typique. La clé privée est stockée exclusivement sur notre propre ordinateur et reste toujours secrète. Si nous voulons établir une connexion SSH, nous entrons d'abord la phrase secrète et ouvrons ainsi l'accès à la clé privée.

Les clés publiques sont également stockées sur le serveur. Le serveur crée un problème cryptographique avec la clé publique du client et l'envoie au client. Le client, à son tour, déchiffre le problème avec sa propre clé privée, renvoie la solution, et informe ainsi le serveur qu'il peut établir une connexion légitime. Au cours d'une session, les utilisateurs n'ont besoin d'entrer la phrase secrète qu'une seule fois pour se connecter à n'importe quel nombre de serveurs. À la fin de la session, les utilisateurs se déconnectent de leurs machines locales, garantissant qu'aucun tiers ayant accès physiquement à la machine locale ne peut se connecter au serveur.

### Empreinte du service
L'un des outils que nous pouvons utiliser pour identifier le serveur SSH est ssh-audit. Il vérifie la configuration côté client et côté serveur et affiche des informations générales et quels algorithmes de chiffrement sont encore utilisés par le client et le serveur. Bien sûr, cela pourrait être exploité en attaquant ultérieurement le serveur ou le client au niveau cryptique.

#### SSH-AUDIT
```
dsgsec@htb[/htb]$ git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
dsgsec@htb[/htb]$ ./ssh-audit.py 10.129.14.132

# general
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
(gen) compression: enabled (zlib@openssh.com)                                   

# key exchange algorithms
(kex) curve25519-sha256                     -- [info] available since OpenSSH 7.4, Dropbear SSH 2018.76                            
(kex) curve25519-sha256@libssh.org          -- [info] available since OpenSSH 6.5, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp256                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp384                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) ecdh-sha2-nistp521                    -- [fail] using weak elliptic curves
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(kex) diffie-hellman-group-exchange-sha256 (2048-bit) -- [info] available since OpenSSH 4.4
(kex) diffie-hellman-group16-sha512         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73
(kex) diffie-hellman-group18-sha512         -- [info] available since OpenSSH 7.3
(kex) diffie-hellman-group14-sha256         -- [info] available since OpenSSH 7.3, Dropbear SSH 2016.73

# host-key algorithms
(key) rsa-sha2-512 (3072-bit)               -- [info] available since OpenSSH 7.2
(key) rsa-sha2-256 (3072-bit)               -- [info] available since OpenSSH 7.2
(key) ssh-rsa (3072-bit)                    -- [fail] using weak hashing algorithm
                                            `- [info] available since OpenSSH 2.5.0, Dropbear SSH 0.28
                                            `- [info] a future deprecation notice has been issued in OpenSSH 8.2: https://www.openssh.com/txt/release-8.2
(key) ecdsa-sha2-nistp256                   -- [fail] using weak elliptic curves
                                            `- [warn] using weak random number generator could reveal the key
                                            `- [info] available since OpenSSH 5.7, Dropbear SSH 2013.62
(key) ssh-ed25519                           -- [info] available since OpenSSH 6.5
...SNIP...
```
La première chose que nous pouvons voir dans les premières lignes de la sortie est la bannière qui révèle la version du serveur OpenSSH. Les versions précédentes présentaient certaines vulnérabilités, telles que CVE-2020-14145, qui permettaient à l'attaquant d'effectuer un Man-In-The-Middle et d'attaquer la tentative de connexion initiale. La sortie détaillée de la configuration de la connexion avec le serveur OpenSSH peut également souvent fournir des informations importantes, telles que les méthodes d'authentification que le serveur peut utiliser.

### Changer la méthode d'authentification
```
dsgsec@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config 
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
```

Pour les attaques potentielles par force brute, nous pouvons spécifier la méthode d'authentification avec l'option client SSH PreferredAuthentications.

```
dsgsec@htb[/htb]$ ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

OpenSSH_8.2p1 Ubuntu-4ubuntu0.3, OpenSSL 1.1.1f  31 Mar 2020
debug1: Reading configuration data /etc/ssh/ssh_config
...SNIP...
debug1: Authentications that can continue: publickey,password,keyboard-interactive
debug1: Next authentication method: password

cry0l1t3@10.129.14.132's password:
```

Même avec ce service évident et sécurisé, nous vous recommandons de configurer notre propre serveur OpenSSH sur notre machine virtuelle, de l'expérimenter et de nous familiariser avec les différents paramètres et options.

Nous pouvons rencontrer diverses bannières pour le serveur SSH lors de nos tests de pénétration. Par défaut, les bannières commencent par la version du protocole qui peut être appliquée, puis la version du serveur lui-même. Par exemple, avec SSH-1.99-OpenSSH_3.9p1, nous savons que nous pouvons utiliser les deux versions de protocole SSH-1 et SSH-2, et nous avons affaire à la version 3.9p1 du serveur OpenSSH. En revanche, pour une bannière avec SSH-2.0-OpenSSH_8.2p1, on a affaire à une version OpenSSH 8.2p1 qui n'accepte que la version du protocole SSH-2.
