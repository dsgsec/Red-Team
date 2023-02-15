# Vhost

Un hôte virtuel (vHost) est une fonctionnalité qui permet d'héberger plusieurs sites Web sur un seul serveur. C'est une excellente solution si vous avez de nombreux sites Web et que vous ne voulez pas passer par le processus long (et coûteux) de configuration d'un nouveau serveur Web pour chacun d'entre eux. Imaginez que vous deviez configurer un serveur Web différent pour une version mobile et une version de bureau de la même page. Il existe deux manières de configurer les hôtes virtuels:

+ Hébergement virtuel basé sur IP
+ Hébergement virtuel basé sur le nom

## Hébergement virtuel basé sur IP
Pour ce type, un hôte peut avoir plusieurs interfaces réseau. Plusieurs adresses IP, ou alias d'interface, peuvent être configurées sur chaque interface réseau d'un hôte. Les serveurs ou les serveurs virtuels exécutés sur l'hôte peuvent se lier à une ou plusieurs adresses IP. Cela signifie que différents serveurs peuvent être adressés sous différentes adresses IP sur cet hôte. Du point de vue du client, les serveurs sont indépendants les uns des autres.

## Hébergement virtuel basé sur le nom
La distinction du domaine pour lequel le service a été demandé se fait au niveau de l'application. Par exemple, plusieurs noms de domaine, tels que admin.inlanefreight.htb et backup.inlanefreight.htb, peuvent faire référence à la même adresse IP. En interne sur le serveur, ceux-ci sont séparés et distingués à l'aide de différents dossiers. En utilisant cet exemple, sur un serveur Linux, le vHost admin.inlanefreight.htb pourrait pointer vers le dossier /var/www/admin. Pour backup.inlanefreight.htb, le nom du dossier serait alors adapté et pourrait ressembler à /var/www/backup.

Au cours de nos activités de découverte de sous-domaines, nous avons vu certains sous-domaines ayant la même adresse IP qui peuvent être des hôtes virtuels ou, dans certains cas, différents serveurs assis derrière un proxy.

Imaginez que nous ayons identifié un serveur Web à 192.168.10.10 lors d'un pentest interne, et qu'il affiche un site Web par défaut à l'aide de la commande suivante. Y a-t-il des hôtes virtuels présents ?

```
dsgsec@htb[/htb]$ curl -s http://192.168.10.10

<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Faisons une requête cURL en envoyant un domaine préalablement identifié lors de la collecte d'informations dans l'en-tête HOST. On peut faire ça comme ça :
```
dsgsec@htb[/htb]$ curl -s http://192.168.10.10 -H "Host: randomtarget.com"

<html>
    <head>
        <title>Welcome to randomtarget.com!</title>
    </head>
    <body>
        <h1>Success! The randomtarget.com server block is working!</h1>
    </body>
</html>
```

Nous pouvons maintenant automatiser cela en utilisant un fichier de dictionnaire des noms de vhost possibles (tels que /opt/useful/SecLists/Discovery/DNS/namelist.txt sur la Pwnbox) et en examinant l'en-tête Content-Length pour rechercher d'éventuelles différences.

## Liste des Vhost
```
app
blog
dev-admin
forum
help
m
my
shop
some
store
support
www
```
## vHost Fuzzing
```
dsgsec@htb[/htb]$ cat ./vhosts | while read vhost;do echo "\n********\nFUZZING: ${vhost}\n********";curl -s -I http://192.168.10.10 -H "HOST: ${vhost}.randomtarget.com" | grep "Content-Length: ";done


********
FUZZING: app
********
Content-Length: 612

********
FUZZING: blog
********
Content-Length: 612

********
FUZZING: dev-admin
********
Content-Length: 120

********
FUZZING: forum
********
Content-Length: 612

********
FUZZING: help
********
Content-Length: 612

********
FUZZING: m
********
Content-Length: 612

********
FUZZING: my
********
Content-Length: 612

********
FUZZING: shop
********
Content-Length: 612

********
FUZZING: some
********
Content-Length: 195

********
FUZZING: store
********
Content-Length: 612

********
FUZZING: support
********
Content-Length: 612

********
FUZZING: www
********
Content-Length: 185
```

Nous avons identifié avec succès un hôte virtuel appelé dev-admin, auquel nous pouvons accéder à l'aide d'une requête cURL.
```
dsgsec@htb[/htb]$ curl -s http://192.168.10.10 -H "Host: dev-admin.randomtarget.com"

<!DOCTYPE html>
<html>
<body>

<h1>Randomtarget.com Admin Website</h1>

<p>You shouldn't be here!</p>

</body>
</html>
```

## Automatisation de la découverte des Vhost
```
dsgsec@htb[/htb]$ ffuf -w ./vhosts -u http://192.168.10.10 -H "HOST: FUZZ.randomtarget.com" -fs 612

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://192.168.10.10
 :: Wordlist         : FUZZ: ./vhosts
 :: Header           : Host: FUZZ.randomtarget.com
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 612
________________________________________________

dev-admin               [Status: 200, Size: 120, Words: 7, Lines: 12]
www                     [Status: 200, Size: 185, Words: 41, Lines: 9]
some                    [Status: 200, Size: 195, Words: 41, Lines: 9]
:: Progress: [12/12] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

