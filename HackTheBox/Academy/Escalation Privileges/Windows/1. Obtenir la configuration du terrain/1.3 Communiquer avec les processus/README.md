Communication avec les processus
============================

* * * * *

L'un des meilleurs endroits pour rechercher une élévation des privilèges est les processus en cours d'exécution sur le système. Même si un processus ne s'exécute pas en tant qu'administrateur, il peut conduire à des privilèges supplémentaires. L'exemple le plus courant consiste à découvrir un serveur Web tel qu'IIS ou XAMPP exécuté sur la boîte, à placer un shell "aspx/php" sur la boîte et à obtenir un shell en tant qu'utilisateur exécutant le serveur Web. Généralement, il ne s'agit pas d'un administrateur, mais il aura souvent le jeton `SeImpersonate` , permettant à `Rogue/Juicy/Lonely Potato` de fournir des autorisations SYSTEM.

* * * * *

Jetons d'accès
--------------

Sous Windows, les [access tokens](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens) sont utilisés pour décrire le contexte de sécurité (attributs ou règles de sécurité) d'un processus ou d'un thread. . Le jeton comprend des informations sur l'identité et les privilèges du compte d'utilisateur liés à un processus ou à un thread spécifique. Lorsqu'un utilisateur s'authentifie auprès d'un système, son mot de passe est vérifié par rapport à une base de données de sécurité et, s'il est correctement authentifié, un jeton d'accès lui est attribué. Chaque fois qu'un utilisateur interagit avec un processus, une copie de ce jeton sera présentée pour déterminer son niveau de privilège.

* * * * *

Énumération des services réseau
-------------------------------------

La manière la plus courante d'interagir avec les processus consiste à utiliser un socket réseau (DNS, HTTP, SMB, etc.). La commande [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) affiche les connexions TCP et UDP actives, ce qui nous donne une meilleure idée des services écoute sur quel(s) port(s) à la fois localement et accessible de l'extérieur. Nous pouvons trouver un service vulnérable accessible uniquement à l'hôte local (lorsqu'il est connecté à l'hôte) que nous pouvons exploiter pour élever les privilèges.

#### Afficher les connexions réseau actives

Afficher les connexions réseau actives

```
C:\htb> netstat -ano

Connexions actives

   Proto Adresse locale Adresse étrangère État PID
   TCP 0.0.0.0:21 0.0.0.0:0 ÉCOUTE 3812
   TCP 0.0.0.0:80 0.0.0.0:0 ÉCOUTE 4
   TCP 0.0.0.0:135 0.0.0.0:0 ÉCOUTE 836
   TCP 0.0.0.0:445 0.0.0.0:0 ÉCOUTE 4
   TCP 0.0.0.0:3389 0.0.0.0:0 ÉCOUTE 936
   TCP 0.0.0.0:5985 0.0.0.0:0 ÉCOUTE 4
   TCP 0.0.0.0:8080 0.0.0.0:0 ÉCOUTE 5044
   TCP 0.0.0.0:47001 0.0.0.0:0 ÉCOUTE 4
   TCP 0.0.0.0:49664 0.0.0.0:0 ÉCOUTE 528
   TCP 0.0.0.0:49665 0.0.0.0:0 ÉCOUTE 996
   TCP 0.0.0.0:49666 0.0.0.0:0 ÉCOUTE 1260
   TCP 0.0.0.0:49668 0.0.0.0:0 ÉCOUTE 2008
   TCP 0.0.0.0:49669 0.0.0.0:0 ÉCOUTE 600
   TCP 0.0.0.0:49670 0.0.0.0:0 ÉCOUTE 1888
   TCP 0.0.0.0:49674 0.0.0.0:0 ÉCOUTE 616
   TCP 10.129.43.8:139 0.0.0.0:0 ÉCOUTE 4
   TCP 10.129.43.8:3389 10.10.14.3:63191 ÉTABLI 936
   TCP 10.129.43.8:49671 40.67.251.132:443 ÉTABLI 1260
   TCP 10.129.43.8:49773 52.37.190.150:443 ÉTABLI 2608
   TCP 10.129.43.8:51580 40.67.251.132:443 ÉTABLI 3808
   TCP 10.129.43.8:54267 40.67.254.36:443 ÉTABLI 3808
   TCP 10.129.43.8:54268 40.67.254.36:443 ÉTABLI 1260
   TCP 10.129.43.8:54269 64.233.184.189:443 ÉTABLI 2608
   TCP 10.129.43.8:54273 216.58.210.195:443 ÉTABLI 2608
   TCP 127.0.0.1:14147 0.0.0.0:0 ÉCOUTE 3812

<SNIP>

   TCP 192.168.20.56:139 0.0.0.0:0 ÉCOUTE 4
   TCP [::]:21 [::]:0 ÉCOUTE 3812
   TCP [::]:80 [::]:0 ÉCOUTE 4
   TCP [::]:135 [::]:0 ÉCOUTE 836
   TCP [::]:445 [::]:0 ÉCOUTE 4
   TCP [::]:3389 [::]:0 ÉCOUTE 936
   TCP [::]:5985 [::]:0 ÉCOUTE 4
   TCP [::]:8080 [::]:0 ÉCOUTE 5044
   TCP [::]:47001 [::]:0 ÉCOUTE 4
   TCP [::]:49664 [::]:0 ÉCOUTE 528
   TCP [::]:49665 [::]:0 ÉCOUTE 996
   TCP [::]:49666 [::]:0 ÉCOUTE 1260
   TCP [::]:49668 [::]:0 ÉCOUTE 2008
   TCP [::]:49669 [::]:0 ÉCOUTE 600
   TCP [::]:49670[::]:0 ÉCOUTE 1888
   TCP [::]:49674 [::]:0 ÉCOUTE 616
   TCP [::1]:14147 [::]:0 ÉCOUTE 3812
   UDP 0.0.0.0:123 *:* 1104
   UDP 0.0.0.0:500 *:* 1260s
   UDP 0.0.0.0:3389 *:* 936

<SNIP>

```

La principale chose à rechercher avec les connexions réseau actives sont les entrées qui écoutent sur les adresses de bouclage (`127.0.0.1` et `::1`) qui n'écoutent pas sur l'adresse IP (`10.129.43.8`) ou diffusées (`0.0. 0.0`, `::/0`). La raison en est que les sockets réseau sur localhost ne sont souvent pas sécurisés en raison de l'idée qu'"ils ne sont pas accessibles au réseau". Celui qui ressort immédiatement sera le port `14147`, qui est utilisé pour l'interface d'administration de FileZilla. En se connectant à ce port, il peut être possible d'extraire des mots de passe FTP en plus de créer un partage FTP sur c:\ en tant qu'utilisateur FileZilla Server (potentiellement administrateur).

#### Plus d'exemples

L'un des meilleurs exemples de ce type d'élévation de privilèges est le `Splunk Universal Forwarder`, installé sur les terminaux pour envoyer les journaux à Splunk. La configuration par défaut de Splunk n'avait aucune authentification sur le logiciel et permettait à n'importe qui de déployer des applications, ce qui pouvait conduire à l'exécution de code. Encore une fois, la configuration par défaut de Splunk consistait à l'exécuter en tant que SYSTEM$ et non en tant qu'utilisateur à faibles privilèges. Pour plus d'informations, consultez [Splunk Universal Forwarder Hijacking](https://airman604.medium.com/splunk-universal-forwarder-hijacking-5899c3e0e6b2) et [SplunkWhisperer2](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/).

Le `port Erlang` (25672) est un autre vecteur d'escalade de privilèges locaux négligé mais courant. Erlang est un langage de programmation conçu autour de l'informatique distribuée et disposera d'un port réseau permettant à d'autres nœuds Erlang de rejoindre le cluster. Le secret pour rejoindre ce cluster s'appelle un cookie. De nombreuses applications qui utilisent Erlang utilisent soit un cookie faible (RabbitMQ utilise `rabbit` par défaut) ou placent le cookie dans un fichier de configuration qui n'est pas bien protégé. Quelques exemples d'applications Erlang sont SolarWinds, RabbitMQ et CouchDB. Pour plus d'informations, consultez le [article de blog Erlang-arce de Mubix](https://malicious.link/post/2018/erlang-arce/)

* * * * *

Canaux nommés
-----------

L'autre façon dont les processus communiquent entre eux est par le biais de canaux nommés. Les tubes sont essentiellement des fichiers stockés en mémoire qui sont effacés après avoir été lus. Cobalt Strike utilise des canaux nommés pour chaque commande (à l'exception [BOF](https://www.cobaltstrike.com/help-beacon-object-files)). Essentiellement, le flux de travail ressemble à ceci :

1. Beacon démarre un canal nommé de \.\pipe\msagent_12
2. Beacon démarre un nouveau processus et injecte une commande dans ce processus en dirigeant la sortie vers \.\pipe\msagent_12
3. Le serveur affiche ce qui a été écrit dans \.\pipe\msagent_12

Cobalt Strike a fait cela parce que si la commande en cours d'exécution était signalée par un antivirus ou plantait, cela n'affecterait pas la balise (processus exécutant la commande). Souvent, les utilisateurs de Cobalt Strike changent leurs canaux nommés pour se faire passer pour un autre programme. L'un des exemples les plus courants est mojo au lieu de msagent. L'une de mes découvertes préférées a été de trouver un tuyau nommé commençant par mojo, mais Chrome n'était pas installé sur l'ordinateur lui-même. Heureusement, cela s'est avéré être l'équipe rouge interne de l'entreprise. Cela en dit long lorsqu'un consultant externe trouve l'équipe rouge, mais pas l'équipe bleue interne.

#### En savoir plus sur les canaux nommés

Les canaux sont utilisés pour la communication entre deux applications ou processus utilisant la mémoire partagée. Il existe deux types de canaux : [canaux nommés](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes) et canaux anonymes. Un exemple de canal nommé est `\\.\PipeName\\ExampleNamedPipeServer`. Les systèmes Windows utilisent une implémentation client-serveur pour la communication par canal. Dans ce type d'implémentation, le processus qui crée un canal nommé est le serveur et le processus communiquant avec le canal nommé est le client. Les canaux nommés peuvent communiquer en utilisant `half-duplex`, ou un canal unidirectionnel, le client ne pouvant écrire que des données sur le serveur, ou `duplex`, qui est un canal de communication bidirectionnel qui permet au client d'écrire des données sur le canal, et le serveur pour répondre avec des données sur ce canal. Chaque connexion active à un serveur de canal nommé entraîne la création d'un nouveau canal nommé. Ceux-ci partagent tous le même nom de canal mais communiquent à l'aide d'un tampon de données différent.

Nous pouvons utiliser l'outil [PipeList](https://docs.microsoft.com/en-us/sysinternals/downloads/pipelist) de la suite Sysinternals pour énumérer les instances de canaux nommés.

#### Liste des canaux nommés avec Pipelist

Liste des canaux nommés avec Pipelist

```
C:\htb> pipelist.exe /accepteula

PipeList v1.02 - Liste les canaux nommés ouverts
Droits d'auteur (C) 2005-2016Marc Russinovitch
Sysinternals - www.sysinternals.com

Instances de nom de canal Instances max.
--------- --------- -------------
InitShutdown 3 -1
lsas 4 -1
ntsvcs 3 -1
scrpc 3 -1
Winsock2\CatalogChangeListener-340-0 1 1
Winsock2\CatalogChangeListener-414-0 1 1
epmapper 3 -1
Winsock2\CatalogChangeListener-3ec-0 1 1
Winsock2\CatalogChangeListener-44c-0 1 1
LSM_API_service 3 -1
atsvc 3 -1
Winsock2\CatalogChangeListener-5e0-0 1 1
journal des événements 3 -1
Winsock2\CatalogChangeListener-6a8-0 1 1
bobines 3 -1
Winsock2\CatalogChangeListener-ec0-0 1 1
wkssvc 4 -1
trkwks 3 -1
vmware-usbarbpipe 5 -1
srvsvc 4 -1
ROUTEUR 3 -1
vmware-authdpipe 1 1

<SNIP>

```

De plus, nous pouvons utiliser PowerShell pour répertorier les canaux nommés à l'aide de `gci` (`Get-ChildItem`).

#### Liste des canaux nommés avec PowerShell

Liste des canaux nommés avec PowerShell

```
PS C:\htb> gci \\.\pipe

     Répertoire : \\.\pipe

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
------ 31/12/1600 16:00 3 InitShutdown
------ 31/12/1600 16:00 4 lsss
------ 31/12/1600 16:00 3 ntsvcs
------ 31/12/1600 16:00 3 scerpc

     Répertoire : \\.\pipe\Winsock2

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
------ 31/12/1600 16:00 1 Winsock2\CatalogChangeListener-34c-0

     Répertoire : \\.\pipe

Mode LastWriteTime Longueur Nom
---- ------------- ------ ----
------ 31/12/1600 16:00 3 epmapper

<SNIP>

```

Après avoir obtenu une liste de canaux nommés, nous pouvons utiliser [Accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) pour énumérer les autorisations attribuées à un canal nommé spécifique en examinant l'accès discrétionnaire List (DACL), qui nous montre qui a les autorisations pour modifier, écrire, lire ou exécuter une ressource. Examinons le processus `LSASS` . Nous pouvons également examiner les DACL de tous les canaux nommés à l'aide de la commande `.\accesschk.exe /accepteula \pipe\`.

#### Examen des autorisations de canal nommé LSASS

Examen des autorisations de canal nommé LSASS

```
C:\htb> accesschk.exe /accepteula \\.\Pipe\lsass -v

Accesschk v6.12 - Signale les autorisations effectives pour les objets sécurisables
Copyright (C) 2006-2017 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Tuyau\lsass
   Niveau obligatoire non fiable [Pas de rédaction]
   RW Tout le monde
         FILE_READ_ATTRIBUTES
         FILE_READ_DATA
         FILE_READ_EA
         FILE_WRITE_ATTRIBUTES
         FILE_WRITE_DATA
         FILE_WRITE_EA
         SYNCHRONISER
         READ_CONTROL
   RW NT AUTORITY\CONNEXION ANONYME
         FILE_READ_ATTRIBUTES
         FILE_READ_DATA
         FILE_READ_EA
         FILE_WRITE_ATTRIBUTES
         FILE_WRITE_DATA
         FILE_WRITE_EA
         SYNCHRONISER
         READ_CONTROL
   RW APPLICATION PACKAGE AUTHORITY\Vos identifiants Windows
         FILE_READ_ATTRIBUTES
         FILE_READ_DATA
         FILE_READ_EA
         FILE_WRITE_ATTRIBUTES
         FILE_WRITE_DATA
         FILE_WRITE_EA
         SYNCHRONISER
         READ_CONTROL
   RW BUILTIN\Administrateurs
         FILE_ALL_ACCESS

```

D'après la sortie ci-dessus, nous pouvons voir que seuls les administrateurs ont un accès complet au processus LSASS, comme prévu.

* * * * *

Exemple d'attaque par canaux nommés
--------------------------

Passons en revue un exemple d'exploitation d'un canal nommé exposé pour élever les privilèges. Ceci [WindscribeService Named Pipe Privilege Escalation](https://www.exploit-db.com/exploits/48021) est un excellent exemple. À l'aide de `accesschk` , nous pouvons rechercher tous les canaux nommés qui autorisent l'accès en écriture avec une commande telle que `accesschk.exe -w \pipe\* -v` et remarquer que le canal nommé `WindscribeService` permet `READ` et `WRITE` accès au groupe "Tout le monde", c'est-à-dire tous les utilisateurs authentifiés.

#### Vérification du nom du service Windscribe
Autorisations de canal ed

En confirmant avec `accesschk` nous voyons que le groupe Tout le monde a bien `FILE_ALL_ACCESS` (Tous les droits d'accès possibles) sur le canal.

Vérification des autorisations de canal nommé WindscribeService

```
C:\htb> accesschk.exe -accepteula -w \pipe\WindscribeService -v

Accesschk v6.13 - Signale les autorisations effectives pour les objets sécurisables
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Pipe\WindscribeService
   Niveau obligatoire moyen (par défaut) [Pas de rédaction]
   RW Tout le monde
         FILE_ALL_ACCESS

```

À partir de là, nous pourrions tirer parti de ces autorisations laxistes pour élever les privilèges sur l'hôte à SYSTEM.