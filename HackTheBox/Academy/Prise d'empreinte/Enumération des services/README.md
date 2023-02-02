# Enumération des services

## FTP

### Connexion annonyme
```
dsgsec@htb[/htb]$ ftp 10.129.14.136

Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.


ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

### Empreinte du service

L'empreinte à l'aide de divers scanners de réseau est également une approche pratique et répandue. Ces outils nous permettent d'identifier plus facilement les différents services, même s'ils ne sont pas accessibles sur les ports standards. L'un des outils les plus utilisés à cette fin est Nmap. Nmap apporte également le Nmap Scripting Engine (NSE), un ensemble de nombreux scripts différents écrits pour des services spécifiques. Vous trouverez plus d'informations sur les fonctionnalités de Nmap et NSE dans le module Énumération de réseau avec Nmap. Nous pouvons mettre à jour cette base de données de scripts NSE avec la commande indiquée.

#### Scripts FTP Nmap
```
dsgsec@htb[/htb]$ sudo nmap --script-updatedb

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:49 CEST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.28 seconds
```
ou

```
dsgsec@htb[/htb]$ sudo nmap -sV -p21 -sC -A 10.129.14.136

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```
<hr>

## SMB

### Configuration par défaut
Comme nous pouvons l'imaginer, Samba propose une large gamme de paramètres que nous pouvons configurer. Encore une fois, nous définissons les paramètres via un fichier texte où nous pouvons obtenir un aperçu de certains paramètres. Ces paramètres ressemblent à ce qui suit lorsqu'ils sont filtrés :

```
dsgsec@htb[/htb]$ cat /etc/samba/smb.conf | grep -v "#\|\;" 

[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d

   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes

   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .

   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```
| Setting | Description |
| --- | --- |
| `[sharename]` | The name of the network share. |
| `workgroup = WORKGROUP/DOMAIN` | Workgroup that will appear when clients query. |
| `path = /path/here/` | The directory to which user is to be given access. |
| `server string = STRING` | The string that will show up when a connection is initiated. |
| `unix password sync = yes` | Synchronize the UNIX password with the SMB password? |
| `usershare allow guests = yes` | Allow non-authenticated users to access defined shared? |
| `map to guest = bad user` | What to do when a user login request doesn't match a valid UNIX user? |
| `browseable = yes` | Should this share be shown in the list of available shares? |
| `guest ok = yes` | Allow connecting to the service without using a password? |
| `read only = yes` | Allow users to read files only? |
| `create mask = 0700` | What permissions need to be set for newly created files? |

### SMBclient - Connexion au partage
```
dsgsec@htb[/htb]$ smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

Nous pouvons voir que nous avons maintenant cinq partages différents sur le serveur Samba à partir du résultat. Ainsi print$ et un IPC$ sont déjà inclus par défaut dans le réglage de base, comme nous l'avons déjà vu. Puisque nous nous occupons du partage [notes], connectons-nous et inspectons-le en utilisant le même programme client. Si nous ne sommes pas familiers avec le programme client, nous pouvons utiliser la commande help lors d'une connexion réussie, en répertoriant toutes les commandes possibles que nous pouvons exécuter.

```
dsgsec@htb[/htb]$ smbclient //10.129.14.128/notes

Enter WORKGROUP\<username>'s password: 
Anonymous login successful
Try "help" to get a list of possible commands.


smb: \> help

?              allinfo        altname        archive        backup         
blocksize      cancel         case_sensitive cd             chmod          
chown          close          del            deltree        dir            
du             echo           exit           get            getfacl        
geteas         hardlink       help           history        iosize         
lcd            link           lock           lowercase      ls             
l              mask           md             mget           mkdir          
more           mput           newer          notify         open           
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir    
posix_unlink   posix_whoami   print          prompt         put            
pwd            q              queue          quit           readlink       
rd             recurse        reget          rename         reput          
rm             rmdir          showacls       setea          setmode        
scopy          stat           symlink        tar            tarmode        
timeout        translate      unlock         volume         vuid           
wdel           logon          listconnect    showconnect    tcon           
tdis           tid            utimes         logoff         ..             
!            


smb: \> ls

  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

Du point de vue administratif, nous pouvons vérifier ces connexions en utilisant smbstatus. Outre la version Samba, nous pouvons également voir qui, depuis quel hôte et quel partage le client est connecté. Ceci est particulièrement important une fois que nous sommes entrés dans un sous-réseau (peut-être même isolé) auquel les autres peuvent toujours accéder.

Par exemple, avec la sécurité au niveau du domaine, le serveur Samba agit en tant que membre d'un domaine Windows. Chaque domaine possède au moins un contrôleur de domaine, généralement un serveur Windows NT fournissant une authentification par mot de passe. Ce contrôleur de domaine fournit au groupe de travail un serveur de mot de passe définitif. Les contrôleurs de domaine gardent une trace des utilisateurs et des mots de passe dans leur propre module d'authentification de sécurité (SAM) et authentifient chaque utilisateur lorsqu'il se connecte pour la première fois et souhaite accéder au partage d'une autre machine.

### Samba Status
```
root@samba:~# smbstatus

Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                                   Protocol Version  Encryption           Signing              
----------------------------------------------------------------------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)      SMB3_11           -                    -                    

Service      pid     Machine       Connected at                     Encryption   Signing     
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -           

No locked files
```

### Prise d'empreintes

Revenons à l'un de nos outils d'énumérations. Nmap propose également de nombreuses options et scripts NSE qui peuvent nous aider à examiner de plus près le service SMB de la cible et à obtenir plus d'informations. L'inconvénient, cependant, est que ces analyses peuvent prendre beaucoup de temps. Par conséquent, il est également recommandé de regarder le service manuellement, principalement parce que nous pouvons trouver beaucoup plus de détails que Nmap ne pourrait nous en montrer. Voyons d'abord ce que Nmap peut trouver sur notre serveur Samba cible, où nous avons créé le partage [notes] à des fins de test.

#### Nmap
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p139,445

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 15:15 CEST
Nmap scan report for sharing.inlanefreight.htb (10.129.14.128)
Host is up (0.00024s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 00:00:00:00:00:00 (VMware)

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-19T13:16:04
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.35 seconds
```

Nous pouvons voir d'après les résultats que ce n'est pas grand-chose que Nmap nous a fourni ici. Par conséquent, nous devrions recourir à d'autres outils qui nous permettent d'interagir manuellement avec la PME et d'envoyer des demandes spécifiques d'informations. L'un des outils pratiques pour cela est rpcclient. Il s'agit d'un outil permettant d'exécuter des fonctions MS-RPC.

L'appel de procédure à distance (RPC) est un concept et, par conséquent, également un outil central pour réaliser des structures opérationnelles et de partage du travail dans les réseaux et les architectures client-serveur. Le processus de communication via RPC comprend la transmission de paramètres et le retour d'une valeur de fonction.

#### RPClient
```
dsgsec@htb[/htb]$ rpcclient -U "" 10.129.14.128

Enter WORKGROUP\'s password:
rpcclient $> 
```

Le rpcclient nous propose de nombreuses requêtes différentes avec lesquelles nous pouvons exécuter des fonctions spécifiques sur le serveur SMB pour obtenir des informations. Une liste complète de toutes ces fonctions peut être trouvée sur la page de manuel de rpcclient.

| Query | Description |
| --- | --- |
| `srvinfo` | Server information. |
| `enumdomains` | Enumerate all domains that are deployed in the network. |
| `querydominfo` | Provides domain, server, and user information of deployed domains. |
| `netshareenumall` | Enumerates all available shares. |
| `netsharegetinfo <share>` | Provides information about a specific share. |
| `enumdomusers` | Enumerates all domain users. |
| `queryuser <RID>` | Provides information about a specific user. |

Cependant, il peut également arriver que toutes les commandes ne soient pas disponibles pour nous et que nous ayons certaines restrictions en fonction de l'utilisateur. Cependant, la requête queryuser <RID> est principalement autorisée en fonction du RID. Nous pouvons donc utiliser le rpcclient pour forcer brutalement les RID afin d'obtenir des informations. Étant donné que nous ne savons peut-être pas à qui a été attribué quel RID, nous savons que nous obtiendrons des informations à ce sujet dès que nous interrogerons un RID attribué. Il existe plusieurs façons et outils que nous pouvons utiliser pour cela. Pour rester avec l'outil, nous pouvons créer une boucle For en utilisant Bash où nous envoyons une commande au service en utilisant rpcclient et filtrer les résultats.

#### BruteForce User RID
```
dsgsec@htb[/htb]$ for i in $(seq 500 1100);do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201
		
        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201
		
        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```
<hr>

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