## RSYNC

Rsync est un outil rapide et efficace pour copier des fichiers localement et à distance. Il peut être utilisé pour copier des fichiers localement sur une machine donnée et vers/depuis des hôtes distants. Il est très polyvalent et bien connu pour son algorithme de transfert delta. Cet algorithme réduit la quantité de données transmises sur le réseau lorsqu'une version du fichier existe déjà sur l'hôte de destination. Pour ce faire, il envoie uniquement les différences entre les fichiers source et l'ancienne version des fichiers qui résident sur le serveur de destination. Il est souvent utilisé pour les sauvegardes et la mise en miroir. Il trouve les fichiers qui doivent être transférés en examinant les fichiers dont la taille a changé ou l'heure de la dernière modification. Par défaut, il utilise le port 873 et peut être configuré pour utiliser SSH pour des transferts de fichiers sécurisés en se superposant à une connexion de serveur SSH établie.

Ce guide couvre certaines des façons dont Rsync peut être abusé, notamment en répertoriant le contenu d'un dossier partagé sur un serveur cible et en récupérant des fichiers. Cela peut parfois se faire sans authentification. D'autres fois, nous aurons besoin d'informations d'identification. Si vous trouvez des informations d'identification lors d'un pentest et que vous rencontrez Rsync sur un hôte interne (ou externe), il est toujours utile de vérifier la réutilisation du mot de passe car vous pourrez peut-être supprimer certains fichiers sensibles qui pourraient être utilisés pour obtenir un accès à distance à la cible.

Faisons un peu d'empreinte rapide. Nous pouvons voir que Rsync est utilisé en utilisant le protocole 31.

### Scan
```
dsgsec@htb[/htb]$ sudo nmap -sV -p 873 127.0.0.1

Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-19 09:31 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0058s latency).

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.13 seconds
```

### Acceder aux répertoires partagés
```
dsgsec@htb[/htb]$ nc -nv 127.0.0.1 873

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev            	Dev Tools
@RSYNCD: EXIT
```
## Énumération d'un partage ouvert
Ici, nous pouvons voir un partage appelé dev, et nous pouvons l'énumérer davantage.

```
dsgsec@htb[/htb]$ rsync -av --list-only rsync://127.0.0.1/dev

receiving incremental file list
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh

sent 25 bytes  received 221 bytes  492.00 bytes/sec
total size is 0  speedup is 0.00
```

À partir de la sortie ci-dessus, nous pouvons voir quelques fichiers intéressants qui peuvent valoir la peine d'être extraits pour approfondir leurs recherches. Nous pouvons également voir qu'un répertoire contenant probablement des clés SSH est accessible. À partir de là, nous pourrions synchroniser tous les fichiers sur notre hôte d'attaque avec la commande rsync -av rsync://127.0.0.1/dev. Si Rsync est configuré pour utiliser SSH pour transférer des fichiers, nous pourrions modifier nos commandes pour inclure l'indicateur -e ssh, ou -e "ssh -p2222" si un port non standard est utilisé pour SSH. Ce guide est utile pour comprendre la syntaxe d'utilisation de Rsync sur SSH.