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