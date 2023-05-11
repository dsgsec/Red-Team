Splunk - Découverte et énumération
================================

* * * * *

Splunk est un outil d'analyse de journaux utilisé pour collecter, analyser et visualiser des données. Bien qu'il ne soit pas initialement destiné à être un outil SIEM, Splunk est souvent utilisé pour la surveillance de la sécurité et l'analyse commerciale. Les déploiements Splunk sont souvent utilisés pour héberger des données sensibles et pourraient fournir une mine d'informations à un attaquant en cas de compromission. Historiquement, Splunk n'a pas souffert de nombreuses vulnérabilités connues à part une vulnérabilité de divulgation d'informations (CVE-2018-11409) et une vulnérabilité d'exécution de code à distance authentifiée dans de très anciennes versions (CVE-2011-4642). Voici quelques [détails](https://www.splunk.com/en_us/customers.html) sur Splunk :

- Splunk a été fondée en 2003, est devenue rentable pour la première fois en 2009 et a fait son introduction en bourse (IPO) en 2012 sur le NASDAQ sous le symbole SPLK
- Splunk compte plus de 7 500 employés et un chiffre d'affaires annuel de près de 2,4 milliards de dollars
- En 2020, Splunk a été nommé sur la liste Fortune 1000
- Les clients de Splunk comprennent 92 entreprises figurant sur la liste Fortune 100
- [Splunkbase](https://splunkbase.splunk.com/) permet aux utilisateurs de Splunk de télécharger des applications et des modules complémentaires pour Splunk. En 2021, il y a plus de 2 000 applications disponibles

Nous verrons le plus souvent Splunk lors de nos évaluations, en particulier dans les environnements de grandes entreprises lors de tests d'intrusion internes. Nous l'avons vu exposé à l'extérieur, mais c'est plus rare. Splunk ne souffre pas de nombreuses vulnérabilités exploitables et corrige rapidement tous les problèmes. Le principal objectif de Splunk lors d'une évaluation serait l'authentification faible ou nulle, car l'accès administrateur à Splunk nous donne la possibilité de déployer des applications personnalisées qui peuvent être utilisées pour compromettre rapidement un serveur Splunk et éventuellement d'autres hôtes du réseau en fonction de la façon dont Splunk est installation.

* * * * *

Découverte/Empreinte
----------------------

Splunk est répandu dans les réseaux internes et s'exécute souvent en tant que root sur Linux ou SYSTEM sur les systèmes Windows. Bien que cela soit rare, nous pouvons parfois rencontrer Splunk vers l'extérieur. Imaginons que nous découvrions une instance oubliée de Splunk dans notre rapport Aquatone qui a depuis été automatiquement convertie en version gratuite, qui ne nécessite pas d'authentification. Puisque nous n'avons pas encore pris pied dans le réseau interne, concentrons notre attention sur Splunk et voyons si nous pouvons transformer cet accès en RCE.

Le serveur Web Splunk s'exécute par défaut sur le port 8000. Sur les anciennes versions de Splunk, les informations d'identification par défaut sont `admin:changeme`, qui s'affichent de manière pratique sur la page de connexion.

![image](https://academy.hackthebox.com/storage/modules/113/changme.png)

La dernière version de Splunk définit les informations d'identification lors du processus d'installation. Si les informations d'identification par défaut ne fonctionnent pas, il est utile de vérifier les mots de passe faibles courants tels que `admin`, `Welcome`, `Welcome1`, `Password123`, etc.

![image](https://academy.hackthebox.com/storage/modules/113/splunk_login.png)

Nous pouvons découvrir Splunk avec une analyse rapide du service Nmap. Ici, nous pouvons voir que Nmap a identifié le service `Splunkd httpd` sur le port 8000 et le port 8089, le port de gestion Splunk pour la communication avec l'API REST Splunk.

```
dsgsec@htb[/htb]$ sudo nmap -sV 10.129.201.50

À partir de Nmap 7.80 ( https://nmap.org ) au 22/09/2021 à 08h43 HAE
Rapport d'analyse Nmap pour 10.129.201.50
L'hôte est actif (latence de 0,11 s).
Non illustré : 991 ports fermés
VERSION SERVICE À L'ÉTAT DU PORT
80/tcp ouvert http Microsoft IIS httpd 10.0
135/tcp ouvert msrpc Microsoft Windows RPC
139/tcp ouvert netbios-ssn Microsoft Windows netbios-ssn
445/tcp ouvrir microsoft-ds ?
3389/tcp open ms-wbt-server Microsoft Terminal Services
5357/tcp ouvert http Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp ouvrir ssl/http Splunkd httpd
8080/tcp open http Indy httpd 17.3.33.2830 (moniteur de bande passante Paessler PRTG)
8089/tcp ouvrir ssl/http Splunkd httpd
Informations sur le service : système d'exploitation : Windows ; CPE : cpe:/o:microsoft:windows

Détection de service effectuée. Veuillez signaler tout résultat incorrect sur https://nmap.org/submit/ .
Nmap terminé : 1 adresse IP (1 hôte actif) scannée en 39,22 secondes

```

* * * * *

Énumération
-----------

L'essai Splunk Enterprise se convertit en une version gratuite après 60 jours, qui ne nécessite pas d'authentification. Il n'est pas rare que les administrateurs système installent une version d'essai de Splunk pour le tester, ce qui est ensuite oublié. Cela se convertira automatiquement en version gratuite qui n'a aucune forme d'authentification, introduisant une faille de sécurité dans l'environnement. Certaines organisations peuvent opter pour la version gratuite en raison de contraintes budgétaires, ne comprenant pas pleinement les implications de l'absence de gestion des utilisateurs/rôles.

![image](https://academy.hackthebox.com/storage/modules/113/license_group.png)

Une fois connecté à Splunk (ou après avoir accédé à une instance de Splunk Free), nous pouvons parcourir les données, exécuter des rapports, créer des tableaux de bord, installer des applications à partir de la bibliothèque Splunkbase et installer des applications personnalisées.

![](https://academy.hackthebox.com/storage/modules/113/splunk_home.png)

Splunk a plusieurs façons d'exécuter du code, comme les applications Django côté serveur, les points de terminaison REST, les entrées scriptées et les scripts d'alerte. Une méthode courante pour obtenir l'exécution de code à distance sur un serveur Splunk consiste à utiliser une entrée scriptée. Ceux-ci sont conçus pour aider à intégrer Splunk avec des sources de données telles que des API ou des serveurs de fichiers qui nécessitent des méthodes personnalisées pour y accéder. Les entrées scriptées sont destinées à exécuter ces scripts, avec STDOUT fourni comme entrée à Splunk.

Comme Splunk peut être installé sur des hôtes Windows ou Linux, des entrées de script peuvent être créées pour exécuter des scripts Bash, PowerShell ou Batch. De plus, chaque installation Splunk est livrée avec Python installé, de sorte que les scripts Python peuvent être exécutés sur n'importe quel système Splunk. Un moyen rapide d'obtenir RCE consiste à créer une entrée scriptée qui indique à Splunk d'exécuter un script shell inverse Python. Nous aborderons cela dans la section suivante.

Outre cette fonctionnalité intégrée, Splunk a souffert de diverses vulnérabilités publiques au fil des ans, telles que cette [SSRF](https://www.exploit-db.com/exploits/40895) qui pourraient être utilisées pour obtenir un accès non autorisé. à l'API REST Splunk. Au moment de la rédaction, Splunk a [47](https://www.cvedetails.com/vulnerability-list/vendor_id-10963/Splunk.html) CVE. Si nous effectuons un scan de vulnérabilité contre Splunk lors d'un test d'intrusion, nous verrons souvent de nombreuses vulnérabilités non exploitables renvoyées. C'est pourquoi il est important de comprendre comment abuser des fonctionnalités intégrées.