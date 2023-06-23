Énumération ACL
===============

* * * * *

Passons à l'énumération des ACL à l'aide de PowerView et parcourons certaines représentations graphiques à l'aide de BloodHound. Nous couvrirons ensuite quelques scénarios/attaques où les ACE que nous énumérons peuvent être exploités pour nous permettre d'accéder davantage à l'environnement interne.

* * * * *

Énumération des ACL avec PowerView
----------------------------------

Nous pouvons utiliser PowerView pour énumérer les ACL, mais la tâche de fouiller dans *tous* les résultats sera extrêmement longue et probablement inexacte. Par exemple, si nous exécutons la fonction, `Find-InterestingDomainAcl`nous recevrons une quantité massive d'informations en retour que nous aurions besoin de creuser pour donner un sens à :

#### Utilisation de Find-InterestingDomainAcl

  Utilisation de Find-InterestingDomainAcl

```
PS C:\htb> Find-InterestingDomainAcl

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : ab721a53-1e2f-11d0-9819-00aa0040529b
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : 00299570-246d-11d0-a768-00aa006e0529
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

<SNIP>

```

Si nous essayons de parcourir toutes ces données lors d'une évaluation limitée dans le temps, nous ne pourrons probablement jamais tout parcourir ou trouver quoi que ce soit d'intéressant avant la fin de l'évaluation. Désormais, il existe un moyen d'utiliser plus efficacement un outil tel que PowerView - en effectuant une énumération ciblée en commençant par un utilisateur sur lequel nous avons le contrôle. Concentrons-nous sur le user `wley`, que nous avons obtenu après avoir résolu la dernière question de la `LLMNR/NBT-NS Poisoning - from Linux`section. Creusons et voyons si cet utilisateur a des droits ACL intéressants dont nous pourrions tirer parti. Nous devons d'abord obtenir le SID de notre utilisateur cible pour effectuer une recherche efficace.

  Utilisation de Find-InterestingDomainAcl

```
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley

```

Nous pouvons alors utiliser la `Get-DomainObjectACL`fonction pour effectuer notre recherche ciblée. Dans l'exemple ci-dessous, nous utilisons cette fonction pour trouver tous les objets de domaine sur lesquels notre utilisateur a des droits en mappant le SID de l'utilisateur à l'aide de la variable à la `$sid`propriété `SecurityIdentifier`qui nous indique *qui* a le droit donné sur un objet. Une chose importante à noter est que si nous recherchons sans le drapeau `ResolveGUIDs`, nous verrons des résultats comme ci-dessous, où la droite `ExtendedRight`ne nous donne pas une image claire de l'entrée ACE que l'utilisateur `wley`a sur `damundsen`. Cela est dû au fait que la `ObjectAceType`propriété renvoie une valeur GUID qui n'est pas lisible par l'homme.

Notez que cette commande prendra un certain temps à s'exécuter, en particulier dans un environnement de grande taille. Cela peut prendre 1 à 2 minutes pour obtenir un résultat dans notre laboratoire.

#### Utilisation de Get-DomainObjectACL

  Utilisation de Get-DomainObjectACL

```
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
ActiveDirectoryRights  : ExtendedRight
ObjectAceFlags         : ObjectAceTypePresent
ObjectAceType          : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectAceType : 00000000-0000-0000-0000-000000000000
BinaryLength           : 56
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 256
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AceType                : AccessAllowedObject
AceFlags               : ContainerInherit
IsInherited            : False
InheritanceFlags       : ContainerInherit
PropagationFlags       : None
AuditFlags             : None

```

Nous pourrions Google pour la valeur GUID `00299570-246d-11d0-a768-00aa006e0529`et découvrir [cette](https://docs.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password) page montrant que l'utilisateur a le droit de forcer le changement du mot de passe de l'autre utilisateur. Alternativement, nous pourrions effectuer une recherche inversée à l'aide de PowerShell pour mapper le bon nom sur la valeur GUID.

#### Exécution d'une recherche inversée et mappage à une valeur GUID

  Exécution d'une recherche inversée et mappage à une valeur GUID

```
PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529"
PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl

Name              : User-Force-Change-Password
DisplayName       : Reset Password
DistinguishedName : CN=User-Force-Change-Password,CN=Extended-Rights,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
rightsGuid        : 00299570-246d-11d0-a768-00aa006e0529

```

Cela nous a donné notre réponse, mais serait très inefficace lors d'une évaluation. PowerView a le `ResolveGUIDs`drapeau, qui fait exactement cela pour nous. Remarquez comment la sortie change lorsque nous incluons cet indicateur pour afficher le format lisible par l'homme de la `ObjectAceType`propriété en tant que `User-Force-Change-Password`.

#### Utilisation de l'indicateur -ResolveGUIDs

  Utilisation de l'indicateur -ResolveGUIDs

```
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}

AceQualifier           : AccessAllowed
ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : User-Force-Change-Password
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

```

`Why did we walk through this example when we could have just searched using ResolveGUIDs first?`

Il est essentiel que nous comprenions ce que font nos outils et que nous ayons des méthodes alternatives dans notre boîte à outils au cas où un outil échouerait ou serait bloqué. Avant de poursuivre, examinons rapidement comment nous pourrions procéder à l'aide des applets de commande [Get-Acl](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2) et [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps) que nous pouvons trouver à notre disposition sur un système client. Savoir effectuer ce type de recherche sans utiliser un outil tel que PowerView est grandement bénéfique et pourrait nous démarquer de nos pairs. Nous pouvons être en mesure d'utiliser ces connaissances pour obtenir des résultats lorsqu'un client nous fait travailler à partir de l'un de ses systèmes, et nous sommes limités aux outils qui sont facilement disponibles sur le système sans la possibilité d'utiliser l'un des nôtres.

Cet exemple n'est pas très efficace et la commande peut prendre beaucoup de temps à s'exécuter, en particulier dans un environnement de grande taille. Cela prendra beaucoup plus de temps que la commande équivalente utilisant PowerView. Dans cette commande, nous avons d'abord dressé une liste de tous les utilisateurs du domaine avec la commande suivante :

#### Création d'une liste d'utilisateurs de domaine

  Création d'une liste d'utilisateurs de domaine

```
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

```

Nous lisons ensuite chaque ligne du fichier à l'aide d'une `foreach`boucle et utilisons l' `Get-Acl`applet de commande pour récupérer les informations ACL pour chaque utilisateur de domaine en transmettant chaque ligne du `ad_users.txt`fichier à l' `Get-ADUser`applet de commande. Nous sélectionnons ensuite uniquement le `Access property`, qui nous donnera des informations sur les droits d'accès. Enfin, nous définissons la `IdentityReference`propriété sur l'utilisateur dont nous contrôlons (ou cherchons à voir quels droits il a), dans notre cas, `wley`.

#### Une boucle foreach utile

  Une boucle foreach utile

```
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}

Path                  : Microsoft.ActiveDirectory.Management.dll\ActiveDirectory:://RootDSE/CN=Dana
                        Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
InheritanceType       : All
ObjectType            : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : ObjectAceTypePresent
AccessControlType     : Allow
IdentityReference     : INLANEFREIGHT\wley
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None

```

Une fois que nous avons ces données, nous pouvons suivre les mêmes méthodes que celles présentées ci-dessus pour convertir le GUID en un format lisible par l'homme afin de comprendre quels droits nous avons sur l'utilisateur cible.

Donc, pour récapituler, nous avons commencé avec l'utilisateur `wley`et avons maintenant le contrôle sur l'utilisateur `damundsen`via le `User-Force-Change-Password`droit étendu. Utilisons Powerview pour rechercher où, le cas échéant, le contrôle du `damundsen`compte pourrait nous mener.

#### Énumération supplémentaire des droits à l'aide de damunsen

  Énumération supplémentaire des droits à l'aide de damunsen

```
PS C:\htb> $sid2 = Convert-NameToSid damundsen
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Help Desk Level 1,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-4022
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1176
AccessMask            : 131132
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed

```

Nous pouvons maintenant voir que notre utilisateur `damundsen`a `GenericWrite`des privilèges sur le `Help Desk Level 1`groupe. Cela signifie, entre autres, que nous pouvons ajouter n'importe quel utilisateur (ou nous-mêmes) à ce groupe et hériter de tous les droits que ce groupe lui a appliqués. Une recherche des droits conférés à ce groupe ne donne rien d'intéressant.

Regardons et voyons si ce groupe est imbriqué dans d'autres groupes, en se souvenant que l'appartenance à un groupe imbriqué signifie que tous les utilisateurs du groupe A hériteront de tous les droits de tout groupe dans lequel le groupe A est imbriqué (un membre). Une recherche rapide nous montre que le `Help Desk Level 1`groupe est imbriqué dans le `Information Technology`groupe, ce qui signifie que nous pouvons obtenir tous les droits que le `Information Technology`groupe accorde à ses membres si nous nous ajoutons simplement au `Help Desk Level 1`groupe où notre utilisateur `damundsen`a `GenericWrite`des privilèges.

#### Enquête sur le groupe d'assistance de niveau 1 avec Get-DomainGroup

  Enquête sur le groupe d'assistance de niveau 1 avec Get-DomainGroup

```
PS C:\htb> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

memberof
--------
CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL

```

C'est beaucoup à digérer ! Récapitulons où nous en sommes :

-   Nous contrôlons l'utilisateur `wley`dont nous avons récupéré le hachage plus tôt dans le module (évaluation) à l'aide de Responder et craqué hors ligne à l'aide de Hashcat pour révéler la valeur du mot de passe en clair
-   Nous avons énuméré les objets sur lesquels l'utilisateur `wley`a le contrôle et avons constaté que nous pouvions forcer le changement du mot de passe de l'utilisateur`damundsen`
-   À partir de là, nous avons constaté que l' `damundsen`utilisateur peut ajouter un membre au `Help Desk Level 1`groupe en utilisant `GenericWrite`des privilèges
-   Le `Help Desk Level 1`groupe est imbriqué dans le `Information Technology`groupe, ce qui accorde aux membres de ce groupe tous les droits provisionnés au `Information Technology`groupe

Maintenant regardons autour de nous et voyons si les membres de `Information Technology`peuvent faire quelque chose d'intéressant. Encore une fois, faire notre recherche en utilisant `Get-DomainObjectACL`nous montre que les membres du `Information Technology`groupe ont `GenericAll`des droits sur l'utilisateur `adunn`, ce qui signifie que nous pourrions :

-   Modifier l'appartenance au groupe
-   Forcer le changement d'un mot de passe
-   Effectuez une attaque Kerberoasting ciblée et tentez de déchiffrer le mot de passe de l'utilisateur s'il est faible

#### Enquête sur le groupe des technologies de l'information

  Enquête sur le groupe des technologies de l'information

```
PS C:\htb> $itgroupsid = Convert-NameToSid "Information Technology"
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Angela Dunn,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-1164
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-4016
AccessMask            : 983551
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed

```

Enfin, voyons si l' `adunn`utilisateur dispose d'un type d'accès intéressant que nous pourrions exploiter pour nous rapprocher de notre objectif.

#### À la recherche d'un accès intéressant

  À la recherche d'un accès intéressant

```
PS C:\htb> $adunnsid = Convert-NameToSid adunn
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

<SNIP>

```

La sortie ci-dessus montre que notre `adunn`utilisateur a `DS-Replication-Get-Changes`des `DS-Replication-Get-Changes-In-Filtered-Set`droits sur l'objet de domaine. Cela signifie que cet utilisateur peut être exploité pour effectuer une attaque DCSync. Nous couvrirons cette attaque en profondeur dans la `DCSync`section.

* * * * *

Énumérer les ACL avec BloodHound
--------------------------------

Maintenant que nous avons énuméré le chemin d'attaque à l'aide de méthodes plus manuelles telles que PowerView et les applets de commande PowerShell intégrées, voyons à quel point cela aurait été plus facile à identifier à l'aide de l'outil extrêmement puissant BloodHound. Prenons les données que nous avons recueillies plus tôt avec l'ingestor SharpHound et téléchargeons-les sur BloodHound. Ensuite, nous pouvons définir l' `wley`utilisateur comme nœud de départ, sélectionner l' `Node Info`onglet et faire défiler jusqu'à `Outbound Control Rights`. Cette option nous montrera les objets sur lesquels nous avons un contrôle direct, via l'appartenance à un groupe, et le nombre d'objets que notre utilisateur pourrait nous amener à contrôler via les chemins d'attaque ACL sous `Transitive Object Control`. Si nous cliquons sur `1`à côté de `First Degree Object Control`, nous voyons le premier ensemble de droits que nous avons énumérés, `ForceChangePassword`sur l' `damundsen`utilisateur.

#### Affichage des informations sur les nœuds via BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/wley_damundsen.png)

Si nous faisons un clic droit sur la ligne entre les deux objets, un menu apparaîtra. Si nous sélectionnons `Help`, nous bénéficierons d'une aide sur l'utilisation abusive de cet ACE, notamment :

-   Plus d'informations sur le droit spécifique, les outils et les commandes qui peuvent être utilisés pour réussir cette attaque
-   Considérations relatives à la sécurité opérationnelle (Opsec)
-   Références externes.

Nous approfondirons ce menu plus tard.

#### Enquêter davantage sur ForceChangePassword

![image](https://academy.hackthebox.com/storage/modules/143/help_edge.png)

Si nous cliquons sur le `16`côté de `Transitive Object Control`, nous verrons l'intégralité du chemin que nous avons minutieusement énuméré ci-dessus. À partir de là, nous pourrions tirer parti des menus d'aide de chaque bord pour trouver les meilleurs moyens de réussir chaque attaque.

#### Affichage des chemins d'attaque potentiels via BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/wley_path.png)

Enfin, nous pouvons utiliser les requêtes prédéfinies dans BloodHound pour confirmer que l' `adunn`utilisateur dispose des droits DCSync.

#### Affichage des requêtes de pré-construction via BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/adunn_dcsync.png)

Nous avons maintenant énuméré ces chemins d'attaque de plusieurs façons. La prochaine étape consistera à exécuter cette chaîne d'attaque du début à la fin. Allons creuser !
