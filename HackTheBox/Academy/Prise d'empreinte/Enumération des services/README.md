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

<hr>

## DNS

Le système de noms de domaine (DNS) fait partie intégrante d'Internet. Par exemple, via des noms de domaine, tels que academy.hackthebox.com ou www.hackthebox.com, nous pouvons atteindre les serveurs Web auxquels le fournisseur d'hébergement a attribué une ou plusieurs adresses IP spécifiques. Le DNS est un système de résolution des noms d'ordinateurs en adresses IP, et il n'a pas de base de données centrale. Simplifié, on peut l'imaginer comme une bibliothèque avec de nombreux annuaires téléphoniques différents. Les informations sont distribuées sur plusieurs milliers de serveurs de noms. Les serveurs DNS distribués à l'échelle mondiale traduisent les noms de domaine en adresses IP et contrôlent ainsi le serveur auquel un utilisateur peut accéder via un domaine particulier. Il existe plusieurs types de serveurs DNS utilisés dans le monde :

- Serveur racine DNS
- Serveur de noms faisant autorité
- Serveur de noms ne faisant pas autorité
- Serveur de cache
- Serveur de transfert
- Résolveur

| Type de serveur | Descriptif |
| --- | --- |
| `Serveur racine DNS` | Les serveurs racine du DNS sont responsables des domaines de premier niveau (`TLD`). En dernier lieu, ils ne sont demandés que si le serveur de noms ne répond pas. Ainsi, un serveur racine est une interface centrale entre les utilisateurs et le contenu sur Internet, car il relie le domaine et l'adresse IP. La [Internet Corporation for Assigned Names and Numbers](https://www.icann.org/)(`ICANN`) coordonne le travail des serveurs de noms racine. Il existe `13` de tels serveurs racine dans le monde. |
| `Serveur de noms faisant autorité` | Les serveurs de noms faisant autorité détiennent l'autorité pour une zone particulière. Ils ne répondent qu'aux questions relevant de leur domaine de responsabilité et leurs informations sont contraignantes. Si un serveur de noms faisant autorité ne peut pas répondre à la requête d'un client, le serveur de noms racine prend le relais à ce stade. |
| `Serveur de noms ne faisant pas autorité` | Les serveurs de noms ne faisant pas autorité ne sont pas responsables d'une zone DNS particulière. Au lieu de cela, ils collectent eux-mêmes des informations sur des zones DNS spécifiques, ce qui se fait à l'aide d'une requête DNS récursive ou itérative. |
| `Mise en cache du serveur DNS` | Mise en cache Les serveurs DNS mettent en cache les informations d'autres serveurs de noms pendant une période spécifiée. Le serveur de noms faisant autorité détermine la durée de ce stockage. |
| `Serveur de transfert` | Les serveurs de transfert n'exécutent qu'une seule fonction: ils transmettent les requêtes DNS à un autre serveur DNS. |
| `Résolveur` | Les résolveurs ne sont pas des serveurs DNS faisant autorité, mais effectuent la résolution de noms localement sur l'ordinateur ou le routeur. |

![image](./ressources/tooldev-dns.png)

Différents enregistrements DNS sont utilisés pour les requêtes DNS, qui ont toutes des tâches différentes. De plus, des entrées distinctes existent pour différentes fonctions puisque nous pouvons configurer des serveurs de messagerie et d'autres serveurs pour un domaine.

| Enregistrement DNS | Descriptif |
| --- | --- |
| `A` | Renvoie une adresse IPv4 du domaine demandé en conséquence. |
| `AAA` | Renvoie une adresse IPv6 du domaine demandé. |
| `MX` | Renvoie les serveurs de messagerie responsables en conséquence. |
| `NS` | Renvoie les serveurs DNS (serveurs de noms) du domaine. |
| `TXT` | Cet enregistrement peut contenir diverses informations. Le polyvalent peut être utilisé, par exemple, pour valider la Google Search Console ou valider les certificats SSL. De plus, les entrées SPF et DMARC sont définies pour valider le trafic de messagerie et le protéger du spam. |
| `CNAME` | Cet enregistrement sert d'alias. Si le domaine www.hackthebox.eu doit pointer vers la même IP, et nous créons un enregistrement A pour l'un et un enregistrement CNAME pour l'autre. |
| `PTR` | L'enregistrement PTR fonctionne dans l'autre sens (recherche inversée). Il convertit les adresses IP en noms de domaine valides. |
| `SOA` | Fournit des informations sur la zone DNS correspondante et l'adresse e-mail du contact administratif. |

L'enregistrement SOA se trouve dans le fichier de zone d'un domaine et spécifie qui est responsable du fonctionnement du domaine et comment les informations DNS du domaine sont gérées.

```
dsgsec@htb[/htb]$ dig soa www.inlanefreight.com

; <<>> DiG 9.16.27-Debian <<>> soa www.inlanefreight.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 15876
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;www.inlanefreight.com.         IN      SOA

;; AUTHORITY SECTION:
inlanefreight.com.      900     IN      SOA     ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

;; Query time: 16 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jan 05 12:56:10 GMT 2023
;; MSG SIZE  rcvd: 128
```

**Le point (.) est remplacé par un arobase (@) dans l'adresse e-mail. Dans cet exemple, l'adresse e-mail de l'administrateur est awsdns-hostmaster@amazon.com.**

## Configuration par défaut

Il existe de nombreux types de configuration différents pour le DNS. Nous n'aborderons donc que les plus importantes pour mieux illustrer le principe de fonctionnement d'un point de vue administratif. Tous les serveurs DNS fonctionnent avec trois types différents de fichiers de configuration:

- fichiers de configuration DNS locaux
- fichiers de zones
- fichiers de résolution de noms inversés

## Prise d'empreinte DNS

L'empreinte sur les serveurs DNS se fait à la suite des requêtes que nous envoyons. Ainsi, tout d'abord, le serveur DNS peut être interrogé pour savoir quels autres serveurs de noms sont connus. Nous le faisons en utilisant l'enregistrement NS et la spécification du serveur DNS que nous voulons interroger en utilisant le caractère @. En effet, s'il existe d'autres serveurs DNS, nous pouvons également les utiliser et interroger les enregistrements. Cependant, d'autres serveurs DNS peuvent être configurés différemment et, en plus, peuvent être permanents pour d'autres zones.

```
dsgsec@htb[/htb]$ dig ns inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> ns inlanefreight.htb @10.129.14.128
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 45010
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: ce4d8681b32abaea0100000061475f73842c401c391690c7 (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      NS

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:04:03 CEST 2021
;; MSG SIZE  rcvd: 107
```
## DIG - Version Query

```
dsgsec@htb[/htb]$ dig CH TXT version.bind 10.129.120.85

; <<>> DiG 9.10.6 <<>> CH TXT version.bind
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 47786
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; ANSWER SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1"

;; ADDITIONAL SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1-Debian"

;; Query time: 2 msec
;; SERVER: 10.129.120.85#53(10.129.120.85)
;; WHEN: Wed Jan 05 20:23:14 UTC 2023
;; MSG SIZE  rcvd: 101
```
Nous pouvons utiliser l'option ANY pour afficher tous les enregistrements disponibles. Cela amènera le serveur à nous montrer toutes les entrées disponibles qu'il est prêt à divulguer. Il est important de noter que toutes les entrées des zones ne seront pas affichées.

## DIG - ANY Query
```
dsgsec@htb[/htb]$ dig any inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> any inlanefreight.htb @10.129.14.128
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7649
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 5, AUTHORITY: 0, ADDITIONAL: 2

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: 064b7e1f091b95120100000061476865a6026d01f87d10ca (good)
;; QUESTION SECTION:
;inlanefreight.htb.             IN      ANY

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:42:13 CEST 2021
;; MSG SIZE  rcvd: 437
```

## Transfert de zone

Le transfert de zone fait référence au transfert de zones vers un autre serveur dans DNS, qui se produit généralement via le port TCP 53. Cette procédure est abrégée Zone de transfert complet asynchrone (AXFR). Étant donné qu'une panne DNS a généralement de graves conséquences pour une entreprise, le fichier de zone est presque invariablement conservé à l'identique sur plusieurs serveurs de noms. Lorsque des modifications sont apportées, il faut s'assurer que tous les serveurs ont les mêmes données. La synchronisation entre les serveurs concernés est réalisée par transfert de zone. A l'aide d'une clé secrète rndc-key, que nous avons vu initialement dans la configuration par défaut, les serveurs s'assurent qu'ils communiquent avec leur propre maître ou esclave. Le transfert de zone implique le simple transfert de fichiers ou d'enregistrements et la détection de divergences dans les ensembles de données des serveurs concernés.

Les données d'origine d'une zone se trouvent sur un serveur DNS, appelé serveur de noms primaire pour cette zone. Cependant, pour augmenter la fiabilité, réaliser une simple répartition de charge, ou protéger le primaire des attaques, on installe en pratique dans la quasi-totalité des cas un ou plusieurs serveurs supplémentaires, appelés serveurs de noms secondaires pour cette zone. Pour certains domaines de premier niveau (TLD), il est obligatoire de rendre les fichiers de zone des domaines de second niveau accessibles sur au moins deux serveurs.

Les entrées DNS ne sont généralement créées, modifiées ou supprimées que sur le serveur principal. Cela peut être fait en éditant manuellement le fichier de zone concerné ou automatiquement par une mise à jour dynamique à partir d'une base de données. Un serveur DNS qui sert de source directe pour synchroniser un fichier de zone est appelé un maître. Un serveur DNS qui obtient des données de zone d'un maître est appelé un esclave. Un primaire est toujours un maître, tandis qu'un secondaire peut être à la fois un esclave et un maître.

L'esclave récupère l'enregistrement SOA de la zone concernée auprès du maître à certains intervalles, le soi-disant temps de rafraîchissement, généralement une heure, et compare les numéros de série. Si le numéro de série de l'enregistrement SOA du maître est supérieur à celui de l'esclave, les jeux de données ne correspondent plus.

## DIG - AXFR Zone Transfer

```
dsgsec@htb[/htb]$ dig axfr inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr inlanefreight.htb @10.129.14.128
;; global options: +cmd
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 4 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:51:19 CEST 2021
;; XFR size: 9 records (messages 1, bytes 520)
```

Si l'administrateur utilisait un sous-réseau pour l'option d'autorisation de transfert à des fins de test ou comme solution de contournement ou le définissait sur n'importe lequel, tout le monde interrogerait l'intégralité du fichier de zone sur le serveur DNS. De plus, d'autres zones peuvent être interrogées, qui peuvent même afficher des adresses IP internes et des noms d'hôte.

## DIG - AXFR Zone Transfer - Internal
```
dsgsec@htb[/htb]$ dig axfr internal.inlanefreight.htb @10.129.14.128

; <<>> DiG 9.16.1-Ubuntu <<>> axfr internal.inlanefreight.htb @10.129.14.128
;; global options: +cmd
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
internal.inlanefreight.htb. 604800 IN   TXT     "MS=ms97310371"
internal.inlanefreight.htb. 604800 IN   TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
internal.inlanefreight.htb. 604800 IN   TXT     "v=spf1 include:mailgun.org include:_spf.google.com include:spf.protection.outlook.com include:_spf.atlassian.net ip4:10.129.124.8 ip4:10.129.127.2 ip4:10.129.42.106 ~all"
internal.inlanefreight.htb. 604800 IN   NS      ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb. 604800 IN A     10.129.34.16
dc2.internal.inlanefreight.htb. 604800 IN A     10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A   10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A      10.129.34.136
vpn.internal.inlanefreight.htb. 604800 IN A     10.129.1.6
ws1.internal.inlanefreight.htb. 604800 IN A     10.129.1.34
ws2.internal.inlanefreight.htb. 604800 IN A     10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A    10.129.18.2
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
;; Query time: 0 msec
;; SERVER: 10.129.14.128#53(10.129.14.128)
;; WHEN: So Sep 19 18:53:11 CEST 2021
;; XFR size: 15 records (messages 1, bytes 664)
```
Les enregistrements A individuels avec les noms d'hôte peuvent également être découverts à l'aide d'une attaque par force brute. Pour ce faire, nous avons besoin d'une liste de noms d'hôtes possibles, que nous utilisons pour envoyer les demandes dans l'ordre. De telles listes sont fournies, par exemple, par SecLists.

Une option serait d'exécuter une boucle for dans Bash qui répertorie ces entrées et envoie la requête correspondante au serveur DNS souhaité.

## Bruteforce de sous-domaines
```
dsgsec@htb[/htb]$ for sub in $(cat /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done

ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
```

De nombreux outils différents peuvent être utilisés pour cela, et la plupart d'entre eux fonctionnent de la même manière. L'un de ces outils est par exemple DNSenum.

```
dsgsec@htb[/htb]$ dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb

dnsenum VERSION:1.2.6

-----   inlanefreight.htb   -----


Host's addresses:
__________________



Name Servers:
______________

ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136


Mail (MX) Servers:
___________________



Trying Zone Transfers and getting Bind Versions:
_________________________________________________

unresolvable name: ns.inlanefreight.htb at /usr/bin/dnsenum line 900 thread 1.

Trying Zone Transfer for inlanefreight.htb on ns.inlanefreight.htb ...
AXFR record query failed: no nameservers


Brute forcing with /home/cry0l1t3/Pentesting/SecLists/Discovery/DNS/subdomains-top1million-110000.txt:
_______________________________________________________________________________________________________

ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136
mail1.inlanefreight.htb.                 604800   IN    A        10.129.18.201
app.inlanefreight.htb.                   604800   IN    A        10.129.18.15
ns.inlanefreight.htb.                    604800   IN    A        10.129.34.136

...SNIP...
done.
```
<hr>

## SMTP

### Introduction

Le protocole SMTP (Simple Mail Transfer Protocol) est un protocole d'envoi d'e-mails sur un réseau IP. Il peut être utilisé entre un client de messagerie et un serveur de messagerie sortant ou entre deux serveurs SMTP. SMTP est souvent combiné avec les protocoles IMAP ou POP3, qui peuvent récupérer des e-mails et envoyer des e-mails. En principe, il s'agit d'un protocole client-serveur, bien que SMTP puisse être utilisé entre un client et un serveur et entre deux serveurs SMTP. Dans ce cas, un serveur agit effectivement en tant que client.

Par défaut, les serveurs SMTP acceptent les demandes de connexion sur le port 25. Cependant, les nouveaux serveurs SMTP utilisent également d'autres ports tels que le port TCP 587. Ce port est utilisé pour recevoir le courrier des utilisateurs/serveurs authentifiés, en utilisant généralement la commande STARTTLS pour changer le texte en clair existant. connexion à une connexion cryptée. Les données d'authentification sont protégées et ne sont plus visibles en clair sur le réseau. Au début de la connexion, l'authentification se produit lorsque le client confirme son identité avec un nom d'utilisateur et un mot de passe. Les e-mails peuvent alors être transmis. À cette fin, le client envoie au serveur les adresses de l'expéditeur et du destinataire, le contenu de l'e-mail et d'autres informations et paramètres. Une fois l'e-mail transmis, la connexion est de nouveau interrompue. Le serveur de messagerie commence alors à envoyer l'e-mail à un autre serveur SMTP.

SMTP fonctionne sans cryptage sans autre mesure et transmet toutes les commandes, données ou informations d'authentification en texte brut. Pour empêcher la lecture non autorisée des données, le SMTP est utilisé en conjonction avec le cryptage SSL/TLS. Dans certaines circonstances, un serveur utilise un port autre que le port TCP standard 25 pour la connexion chiffrée, par exemple, le port TCP 465.

Une fonction essentielle d'un serveur SMTP est d'empêcher le spam en utilisant des mécanismes d'authentification qui permettent uniquement aux utilisateurs autorisés d'envoyer des e-mails. À cette fin, la plupart des serveurs SMTP modernes prennent en charge l'extension de protocole ESMTP avec SMTP-Auth. Après avoir envoyé son e-mail, le client SMTP, également appelé Mail User Agent (MUA), le convertit en un en-tête et un corps et les télécharge tous les deux sur le serveur SMTP. Celui-ci dispose d'un agent de transfert de courrier (MTA), la base logicielle pour l'envoi et la réception d'e-mails. Le MTA vérifie la taille et le spam de l'e-mail, puis le stocke. Pour soulager le MTA, il est parfois précédé d'un Mail Submission Agent (MSA), qui vérifie la validité, c'est-à-dire l'origine du courrier électronique. Ce MSA est également appelé serveur relais. Celles-ci sont très importantes plus tard, car la soi-disant attaque par relais ouvert peut être effectuée sur de nombreux serveurs SMTP en raison d'une configuration incorrecte. Nous discuterons de cette attaque et de la manière d'en identifier le point faible un peu plus tard. Le MTA recherche alors dans le DNS l'adresse IP du serveur de messagerie destinataire.

A leur arrivée sur le serveur SMTP de destination, les paquets de données sont réassemblés pour former un e-mail complet. De là, l'agent de distribution du courrier (MDA) le transfère dans la boîte aux lettres du destinataire.

Client (MUA)	➞	Submission Agent (MSA)	➞	Open Relay (MTA)	➞	Mail Delivery Agent (MDA)	➞	Mailbox (POP3/IMAP)

### Utilisation de SMTP

#### Telnet - HELO/EHLO
Pour interagir avec le serveur SMTP, nous pouvons utiliser l'outil telnet pour initialiser une connexion TCP avec le serveur SMTP. L'initialisation proprement dite de la session se fait avec la commande mentionnée ci-dessus, HELO ou EHLO.
```
dsgsec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 


HELO mail1.inlanefreight.htb

250 mail1.inlanefreight.htb


EHLO mail1

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```
#### Telnet - VRFY
La commande VRFY peut être utilisée pour énumérer les utilisateurs existants sur le système. Cependant, cela ne fonctionne pas toujours. Selon la configuration du serveur SMTP, le serveur SMTP peut émettre le code 252 et confirmer l'existence d'un utilisateur qui n'existe pas sur le système. Une liste de tous les codes de réponse SMTP peut être trouvée ici.

```
sgsec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server 

VRFY root

252 2.0.0 root


VRFY cry0l1t3

252 2.0.0 cry0l1t3


VRFY testuser

252 2.0.0 testuser


VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa

252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa
```


####  Envoyer un mail

```
dsgsec@htb[/htb]$ telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
220 ESMTP Server


EHLO inlanefreight.htb

250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING


MAIL FROM: <cry0l1t3@inlanefreight.htb>

250 2.1.0 Ok


RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure

250 2.1.5 Ok


DATA

354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200
Hey man, I am trying to access our XY-DB but the creds don't work. 
Did you make any changes there?
.

250 2.0.0 Ok: queued as 6E1CF1681AB


QUIT

221 2.0.0 Bye
Connection closed by foreign host.
```
### Prise d'empreinte

Les scripts Nmap par défaut incluent les commandes smtp, qui utilisent la commande EHLO pour répertorier toutes les commandes possibles pouvant être exécutées sur le serveur SMTP cible.

#### Nmap
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -sC -sV -p25

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-27 17:56 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00025s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING, 
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

#### Nmap open-relay
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-30 02:29 CEST
NSE: Loaded 1 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.00s elapsed
Initiating ARP Ping Scan at 02:29
Scanning 10.129.14.128 [1 port]
Completed ARP Ping Scan at 02:29, 0.06s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 02:29
Completed Parallel DNS resolution of 1 host. at 02:29, 0.03s elapsed
Initiating SYN Stealth Scan at 02:29
Scanning 10.129.14.128 [1 port]
Discovered open port 25/tcp on 10.129.14.128
Completed SYN Stealth Scan at 02:29, 0.06s elapsed (1 total ports)
NSE: Script scanning 10.129.14.128.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.07s elapsed
Nmap scan report for 10.129.14.128
Host is up (0.00020s latency).

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-open-relay: Server is an open relay (16/16 tests)
|  MAIL FROM:<> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@nmap.scanme.org> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@ESMTP> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest%nmap.scanme.org@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest%nmap.scanme.org@ESMTP>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org">
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest%nmap.scanme.org">
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<"relaytest@nmap.scanme.org"@[10.129.14.128]>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org@ESMTP>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<@[10.129.14.128]:relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<@ESMTP:relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@[10.129.14.128]>
|_ MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@ESMTP>
MAC Address: 00:00:00:00:00:00 (VMware)

NSE: Script Post-scanning.
Initiating NSE at 02:29
Completed NSE at 02:29, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.48 seconds
           Raw packets sent: 2 (72B) | Rcvd: 2 (72B)
```

#### Scan users
```
smtp-user-enum -U footprinting-wordlist.txt -t 10.129.14.248 -w 25 -M VRFY
```

<hr>

## IMAP & POP3

### Définition
Avec l'aide du protocole d'accès aux messages Internet (IMAP), l'accès aux e-mails à partir d'un serveur de messagerie est possible. Contrairement au Post Office Protocol (POP3), IMAP permet la gestion en ligne des e-mails directement sur le serveur et prend en charge les structures de dossiers. Il s'agit donc d'un protocole réseau pour la gestion en ligne des emails sur un serveur distant. Le protocole est basé sur le client-serveur et permet la synchronisation d'un client de messagerie local avec la boîte aux lettres sur le serveur, fournissant une sorte de système de fichiers réseau pour les e-mails, permettant une synchronisation sans problème entre plusieurs clients indépendants. POP3, d'autre part, n'a pas la même fonctionnalité qu'IMAP, et il ne fournit que la liste, la récupération et la suppression des e-mails en tant que fonctions sur le serveur de messagerie. Par conséquent, des protocoles tels que IMAP doivent être utilisés pour des fonctionnalités supplémentaires telles que les boîtes aux lettres hiérarchiques directement sur le serveur de messagerie, l'accès à plusieurs boîtes aux lettres au cours d'une session et la présélection des e-mails.

Les clients accèdent à ces structures en ligne et peuvent créer des copies locales. Même sur plusieurs clients, cela se traduit par une base de données uniforme. Les e-mails restent sur le serveur jusqu'à ce qu'ils soient supprimés. IMAP est basé sur du texte et possède des fonctions étendues, telles que la navigation dans les e-mails directement sur le serveur. Il est également possible que plusieurs utilisateurs accèdent simultanément au serveur de messagerie. Sans une connexion active au serveur, la gestion des e-mails est impossible. Cependant, certains clients proposent un mode hors ligne avec une copie locale de la boîte aux lettres. Le client synchronise toutes les modifications locales hors ligne lorsqu'une connexion est rétablie.

Le client établit la connexion au serveur via le port 143. Pour la communication, il utilise des commandes textuelles au format ASCII. Plusieurs commandes peuvent être envoyées successivement sans attendre la confirmation du serveur. Des confirmations ultérieures du serveur peuvent être attribuées aux commandes individuelles à l'aide des identifiants envoyés avec les commandes. Immédiatement après l'établissement de la connexion, l'utilisateur est authentifié par son nom d'utilisateur et son mot de passe auprès du serveur. L'accès à la boîte aux lettres souhaitée n'est possible qu'après une authentification réussie.

### Commandes IMAP
```
IMAP Commands
Command	Description
1 LOGIN username password	User's login.
1 LIST "" *	Lists all directories.
1 CREATE "INBOX"	Creates a mailbox with a specified name.
1 DELETE "INBOX"	Deletes a mailbox.
1 RENAME "ToRead" "Important"	Renames a mailbox.
1 LSUB "" *	Returns a subset of names from the set of names that the User has declared as being active or subscribed.
1 SELECT INBOX	Selects a mailbox so that messages in the mailbox can be accessed.
1 UNSELECT INBOX	Exits the selected mailbox.
1 FETCH <ID> all	Retrieves data associated with a message in the mailbox.
1 CLOSE	Removes all messages with the Deleted flag set.
1 LOGOUT	Closes the connection with the IMAP server.
```

### Commandes POP3
```
Command	Description
USER username	Identifies the user.
PASS password	Authentication of the user using its password.
STAT	Requests the number of saved emails from the server.
LIST	Requests from the server the number and size of all emails.
RETR id	Requests the server to deliver the requested email by ID.
DELE id	Requests the server to delete the requested email by ID.
CAPA	Requests the server to display the server capabilities.
RSET	Requests the server to reset the transmitted information.
QUIT	Closes the connection with the POP3 server.
```

### Prise d'empreintes

#### Nmap
```
dsgsec@htb[/htb]$ sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 22:09 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00026s latency).

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE SASL STLS TOP UIDL RESP-CODES CAPA PIPELINING
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
143/tcp open  imap     Dovecot imapd
|_imap-capabilities: more have post-login STARTTLS Pre-login capabilities LITERAL+ LOGIN-REFERRALS OK LOGINDISABLEDA0001 SASL-IR ENABLE listed IDLE ID IMAP4rev1
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
993/tcp open  ssl/imap Dovecot imapd
|_imap-capabilities: more have post-login OK capabilities LITERAL+ LOGIN-REFERRALS Pre-login AUTH=PLAINA0001 SASL-IR ENABLE listed IDLE ID IMAP4rev1
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE USER SASL(PLAIN) TOP UIDL RESP-CODES CAPA PIPELINING
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight/stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58
MAC Address: 00:00:00:00:00:00 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.74 seconds
```

Par exemple, à partir de la sortie, nous pouvons voir que le nom commun est mail1.inlanefreight.htb et que le serveur de messagerie appartient à l'organisation Inlanefreight, située en Californie. Les capacités affichées nous montrent les commandes disponibles sur le serveur et pour le service sur le port correspondant.

Si nous réussissons à déterminer les informations d'identification d'accès pour l'un des employés, un attaquant pourrait se connecter au serveur de messagerie et lire ou même envoyer les messages individuels.

#### cURL
```
dsgsec@htb[/htb]$ curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

* LIST (\HasNoChildren) "." Important
* LIST (\HasNoChildren) "." INBOX
```

Pour interagir avec le serveur IMAP ou POP3 via SSL, nous pouvons utiliser openssl, ainsi que ncat. Les commandes pour cela ressembleraient à ceci:

#### OpenSSL - TLS Encrypted Interaction POP3
```
dsgsec@htb[/htb]$ openssl s_client -connect 10.129.14.128:pop3s

CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Customer Support, CN = mail1.inlanefreight.htb, emailAddress = cry0l1t3@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Customer Support, CN = mail1.inlanefreight.htb, emailAddress = cry0l1t3@inlanefreight.htb
verify return:1
---
Certificate chain
 0 s:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Customer Support, CN = mail1.inlanefreight.htb, emailAddress = cry0l1t3@inlanefreight.htb

...SNIP...

---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 3CC39A7F2928B252EF2FFA5462140B1A0A74B29D4708AA8DE1515BB4033D92C2
    Session-ID-ctx:
    Resumption PSK: 68419D933B5FEBD878FF1BA399A926813BEA3652555E05F0EC75D65819A263AA25FA672F8974C37F6446446BB7EA83F9
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - d7 86 ac 7e f3 f4 95 35-88 40 a5 b5 d6 a6 41 e4   ...~...5.@....A.
    0010 - 96 6c e6 12 4f 50 ce 72-36 25 df e1 72 d9 23 94   .l..OP.r6%..r.#.
    0020 - cc 29 90 08 58 1b 57 ab-db a8 6b f7 8f 31 5b ad   .)..X.W...k..1[.
    0030 - 47 94 f4 67 58 1f 96 d9-ca ca 56 f9 7a 12 f6 6d   G..gX.....V.z..m
    0040 - 43 b9 b6 68 de db b2 47-4f 9f 48 14 40 45 8f 89   C..h...GO.H.@E..
    0050 - fa 19 35 9c 6d 3c a1 46-5c a2 65 ab 87 a4 fd 5e   ..5.m<.F\.e....^
    0060 - a2 95 25 d4 43 b8 71 70-40 6c fe 6f 0e d1 a0 38   ..%.C.qp@l.o...8
    0070 - 6e bd 73 91 ed 05 89 83-f5 3e d9 2a e0 2e 96 f8   n.s......>.*....
    0080 - 99 f0 50 15 e0 1b 66 db-7c 9f 10 80 4a a1 8b 24   ..P...f.|...J..$
    0090 - bb 00 03 d4 93 2b d9 95-64 44 5b c2 6b 2e 01 b5   .....+..dD[.k...
    00a0 - e8 1b f4 a4 98 a7 7a 7d-0a 80 cc 0a ad fe 6e b3   ......z}......n.
    00b0 - 0a d6 50 5d fd 9a b4 5c-28 a4 c9 36 e4 7d 2a 1e   ..P]...\(..6.}*.

    Start Time: 1632081313
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
+OK HTB-Academy POP3 Server
```

#### OpenSSL - TLS Encrypted Interaction IMAP
```
dsgsec@htb[/htb]$ openssl s_client -connect 10.129.14.128:imaps

CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Customer Support, CN = mail1.inlanefreight.htb, emailAddress = cry0l1t3@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Customer Support, CN = mail1.inlanefreight.htb, emailAddress = cry0l1t3@inlanefreight.htb
verify return:1
---
Certificate chain
 0 s:C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Customer Support, CN = mail1.inlanefreight.htb, emailAddress = cry0l1t3@inlanefreight.htb

...SNIP...

---
read R BLOCK
---
Post-Handshake New Session Ticket arrived:
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : TLS_AES_256_GCM_SHA384
    Session-ID: 2B7148CD1B7B92BA123E06E22831FCD3B365A5EA06B2CDEF1A5F397177130699
    Session-ID-ctx:
    Resumption PSK: 4D9F082C6660646C39135F9996DDA2C199C4F7E75D65FA5303F4A0B274D78CC5BD3416C8AF50B31A34EC022B619CC633
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - 68 3b b6 68 ff 85 95 7c-8a 8a 16 b2 97 1c 72 24   h;.h...|......r$
    0010 - 62 a7 84 ff c3 24 ab 99-de 45 60 26 e7 04 4a 7d   b....$...E`&..J}
    0020 - bc 6e 06 a0 ff f7 d7 41-b5 1b 49 9c 9f 36 40 8d   .n.....A..I..6@.
    0030 - 93 35 ed d9 eb 1f 14 d7-a5 f6 3f c8 52 fb 9f 29   .5........?.R..)
    0040 - 89 8d de e6 46 95 b3 32-48 80 19 bc 46 36 cb eb   ....F..2H...F6..
    0050 - 35 79 54 4c 57 f8 ee 55-06 e3 59 7f 5e 64 85 b0   5yTLW..U..Y.^d..
    0060 - f3 a4 8c a6 b6 47 e4 59-ee c9 ab 54 a4 ab 8c 01   .....G.Y...T....
    0070 - 56 bb b9 bb 3b f6 96 74-16 c9 66 e2 6c 28 c6 12   V...;..t..f.l(..
    0080 - 34 c7 63 6b ff 71 16 7f-91 69 dc 38 7a 47 46 ec   4.ck.q...i.8zGF.
    0090 - 67 b7 a2 90 8b 31 58 a0-4f 57 30 6a b6 2e 3a 21   g....1X.OW0j..:!
    00a0 - 54 c7 ba f0 a9 74 13 11-d5 d1 ec cc ea f9 54 7d   T....t........T}
    00b0 - 46 a6 33 ed 5d 24 ed b0-20 63 43 d8 8f 14 4d 62   F.3.]$.. cC...Mb

    Start Time: 1632081604
    Timeout   : 7200 (sec)
    Verify return code: 18 (self signed certificate)
    Extended master secret: no
    Max Early Data: 0
---
read R BLOCK
* OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LITERAL+ AUTH=PLAIN] HTB-Academy IMAP4 v.0.21.4
```
