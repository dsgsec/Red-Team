# Enumération des services

## NFS

Network File System (NFS) est un système de fichiers réseau développé par Sun Microsystems et a le même objectif que SMB. Son but est d'accéder aux systèmes de fichiers sur un réseau comme s'ils étaient locaux. Cependant, il utilise un protocole entièrement différent. NFS est utilisé entre les systèmes Linux et Unix. Cela signifie que les clients NFS ne peuvent pas communiquer directement avec les serveurs SMB. NFS est une norme Internet qui régit les procédures dans un système de fichiers distribué. Alors que le protocole NFS version 3.0 (NFSv3), utilisé depuis de nombreuses années, authentifie l'ordinateur client, cela change avec NFSv4. Ici, comme pour le protocole Windows SMB, l'utilisateur doit s'authentifier.

| Version | Caractéristiques |
| --- | --- |
| `NFSv2` | Il est plus ancien mais est pris en charge par de nombreux systèmes et fonctionnait initialement entièrement sur UDP. |
| `NFSv3` | Il a plus de fonctionnalités, y compris une taille de fichier variable et un meilleur rapport d'erreurs, mais n'est pas entièrement compatible avec les clients NFSv2. |
| `NFSv4` | Il inclut Kerberos, fonctionne à travers les pare-feu et sur Internet, ne nécessite plus de portmappers, prend en charge les ACL, applique des opérations basées sur l'état et offre des améliorations de performances et une sécurité élevée. C'est aussi la première version à avoir un protocole avec état.

### Configuration par défaut
NFS n'est pas difficile à configurer car il n'y a pas autant d'options que FTP ou SMB. Le fichier /etc/exports contient une table des systèmes de fichiers physiques sur un serveur NFS accessible par les clients. Le tableau des exportations NFS montre les options qu'il accepte et indique ainsi les options qui s'offrent à nous.

```
dsgsec@htb[/htb]$ cat /etc/exports 

# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
```

| Option | Description |
| --- | --- |
| `rw` | Read and write permissions. |
| `ro` | Read only permissions. |
| `sync` | Synchronous data transfer. (A bit slower) |
| `async` | Asynchronous data transfer. (A bit faster) |
| `secure` | Ports above 1024 will not be used. |
| `insecure` | Ports above 1024 will be used. |
| `no_subtree_check` | This option disables the checking of subdirectory trees. |
| `root_squash` | Assigns all permissions to files of root UID/GID 0 to the UID/GID of anonymous, which prevents `root` from accessing files on an NFS mount. |

### Export FS
```
root@nfs:~# echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
root@nfs:~# systemctl restart nfs-kernel-server 
root@nfs:~# exportfs

/mnt/nfs      	10.129.14.0/24
```

Nous avons partagé le dossier /mnt/nfs avec le sous-réseau 10.129.14.0/24 avec le paramètre indiqué ci-dessus. Cela signifie que tous les hôtes du réseau pourront monter ce partage NFS et inspecter le contenu de ce dossier.

### Empreinte du service

Lors de l'empreinte NFS, les ports TCP 111 et 2049 sont essentiels. Nous pouvons également obtenir des informations sur le service NFS et l'hôte via RPC, comme indiqué ci-dessous dans l'exemple.

#### Nmap
```bash
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -p111,2049 -sV -sC

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:12 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00018s latency).

PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41982/udp6  mountd
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      47217/tcp6  mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      39542/udp   nlockmgr
|   100021  1,3,4      44629/tcp   nlockmgr
|   100021  1,3,4      45273/tcp6  nlockmgr
|   100021  1,3,4      47524/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.58 seconds
```

Le script rpcinfo NSE récupère une liste de tous les services RPC en cours d'exécution, leurs noms et descriptions, ainsi que les ports qu'ils utilisent. Cela nous permet de vérifier si le partage cible est connecté au réseau sur tous les ports requis. De plus, pour NFS, Nmap a des scripts NSE qui peuvent être utilisés pour les analyses. Ceux-ci peuvent alors nous montrer, par exemple, le contenu du partage et ses statistiques.

```
dsgsec@htb[/htb]$ sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 17:37 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00021s latency).

PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| ??????????  ?      ?      ?     ?                    ..
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0      0      0     2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount: 
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs: 
|   Filesystem  1K-blocks   Used       Available   Use%  Maxfilesize  Maxlink
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%   16.0T        32000
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  3           2049/udp   nfs
|   100003  3           2049/udp6  nfs
|   100003  3,4         2049/tcp   nfs
|   100003  3,4         2049/tcp6  nfs
|   100005  1,2,3      41982/udp6  mountd
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      47217/tcp6  mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      39542/udp   nlockmgr
|   100021  1,3,4      44629/tcp   nlockmgr
|   100021  1,3,4      45273/tcp6  nlockmgr
|   100021  1,3,4      47524/udp6  nlockmgr
|   100227  3           2049/tcp   nfs_acl
|   100227  3           2049/tcp6  nfs_acl
|   100227  3           2049/udp   nfs_acl
|_  100227  3           2049/udp6  nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```

Une fois que nous avons découvert un tel service NFS, nous pouvons le monter sur notre machine locale. Pour cela, nous pouvons créer un nouveau dossier vide dans lequel le partage NFS sera monté. Une fois monté, nous pouvons y naviguer et afficher le contenu comme notre système local.

### Afficher les partages NFS disponibles
```
dsgsec@htb[/htb]$ showmount -e 10.129.14.128

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

### Montage du partage NFS
```
dsgsec@htb[/htb]$ mkdir target-NFS
dsgsec@htb[/htb]$ sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
dsgsec@htb[/htb]$ cd target-NFS
dsgsec@htb[/htb]$ tree .

.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files
```

### Répertorier le contenu avec les noms d'utilisateur et les noms de groupe
```
dsgsec@htb[/htb]$ ls -l mnt/nfs/

total 16
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 cry0l1t3 cry0l1t3  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1 root     root        0 Sep 19 17:22 nfs.share
```

### Contenu de la liste avec UID et GUID
```
dsgsec@htb[/htb]$ ls -n mnt/nfs/

total 16
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 1000 1000  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1    0 1000 1221 Sep 19 18:21 backup.sh
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share
```

Il est important de noter que si l'option root_squash est définie, nous ne pouvons pas modifier le fichier backup.sh même en tant que root.

Nous pouvons également utiliser NFS pour une escalade ultérieure. Par exemple, si nous avons accès au système via SSH et que nous voulons lire les fichiers d'un autre dossier qu'un utilisateur spécifique peut lire, nous devons télécharger un shell sur le partage NFS qui a le SUID de cet utilisateur, puis exécuter le shell via l'utilisateur SSH.

Après avoir effectué toutes les étapes nécessaires et obtenu les informations dont nous avons besoin, nous pouvons démonter le partage NFS.