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
