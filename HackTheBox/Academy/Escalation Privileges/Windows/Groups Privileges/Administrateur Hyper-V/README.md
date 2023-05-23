Administrateurs Hyper-V
======================

* * * * *

Le groupe [Administrateurs Hyper-V](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators) a accès complet à toutes les [fonctionnalités Hyper-V](https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines). Si les contrôleurs de domaine ont été virtualisés, les administrateurs de virtualisation doivent être considérés comme des administrateurs de domaine. Ils pourraient facilement créer un clone du contrôleur de domaine actif et monter le disque virtuel hors ligne pour obtenir le fichier NTDS.dit et extraire les hachages de mot de passe NTLM pour tous les utilisateurs du domaine.

Il est également bien documenté sur ce [blog](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/), que lors de la suppression d'une machine virtuelle, `vmms.exe ` tente de restaurer les autorisations de fichier d'origine sur le fichier `.vhdx` correspondant et le fait en tant que `NT AUTHORITY\SYSTEM`, sans se faire passer pour l'utilisateur. Nous pouvons supprimer le fichier `.vhdx` et créer un lien dur natif pour faire pointer ce fichier vers un fichier SYSTEM protégé, pour lequel nous aurons toutes les autorisations.

Si le système d'exploitation est vulnérable à [CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) ou [CVE-2019-0841](https://www.tenable. com/cve/CVE-2019-0841), nous pouvons en tirer parti pour obtenir des privilèges SYSTEM. Sinon, nous pouvons essayer de profiter d'une application sur le serveur qui a installé un service s'exécutant dans le contexte de SYSTEM, qui peut être démarré par des utilisateurs non privilégiés.

#### Fichier cible

Un exemple de ceci est Firefox, qui installe le `Mozilla Maintenance Service`. Nous pouvons mettre à jour [cet exploit](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1) (une preuve de concept pour le lien physique NT) pour accorder à notre utilisateur actuel des autorisations complètes sur le fichier ci-dessous :

Fichier cible

```
C:\Program Files (x86)\Service de maintenance Mozilla\maintenanceservice.exe

```

#### Prendre possession du fichier

Après avoir exécuté le script PowerShell, nous devrions avoir le contrôle total de ce fichier et pouvons en prendre possession.

Prendre possession du dossier

```
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe

```

#### Démarrage du service de maintenance de Mozilla

Ensuite, nous pouvons remplacer ce fichier par un `maintenanceservice.exe` malveillant, démarrer le service de maintenance et obtenir l'exécution de la commande en tant que SYSTEM.

Démarrage du service de maintenance de Mozilla

```
C:\htb> sc.exe start MozillaMaintenance

```

Remarque : ce vecteur a été atténué par les mises à jour de sécurité Windows de mars 2020, qui ont modifié le comportement relatif aux liens physiques.