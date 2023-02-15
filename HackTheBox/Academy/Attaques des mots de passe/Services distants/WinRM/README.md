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