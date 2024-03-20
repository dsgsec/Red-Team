ntroduction aux approbations de domaine
=======================================

* * * * *

Scénario
--------

De nombreuses grandes organisations acquerront de nouvelles entreprises au fil du temps et les intégreront. Une façon de procéder pour faciliter l'utilisation consiste à établir une relation de confiance avec le nouveau domaine. Ce faisant, vous pouvez éviter de migrer tous les objets établis, ce qui rend l'intégration beaucoup plus rapide. Cette confiance peut également introduire des faiblesses dans l'environnement du client s'il n'y prend pas garde. Un sous-domaine présentant une faille ou une vulnérabilité exploitable peut nous fournir un accès rapide au domaine cible. Les entreprises peuvent également établir des fiducies avec d'autres entreprises (telles qu'un MSP), un client ou d'autres unités commerciales de la même entreprise (telles qu'une division de l'entreprise dans une autre région géographique). Explorons davantage les approbations de domaine et comment nous pouvons abuser des fonctionnalités intégrées lors de nos évaluations.

* * * * *

Présentation des approbations de domaine
----------------------------------------

Une [approbation](https://social.technet.microsoft.com/wiki/contents/articles/50969.active-directory-forest-trust-attention-points.aspx) est utilisée pour établir une authentification forêt-forêt ou domaine-domaine (intra-domaine), qui permet aux utilisateurs d'accéder aux ressources (ou d'effectuer des tâches administratives) d'un autre domaine, en dehors du domaine principal où réside leur compte. Une confiance crée un lien entre les systèmes d'authentification de deux domaines et peut permettre une communication unidirectionnelle ou bidirectionnelle (bidirectionnelle). Une organisation peut créer différents types de fiducies :

-   `Parent-child`: Deux domaines ou plus au sein de la même forêt. Le domaine enfant a une approbation transitive bidirectionnelle avec le domaine parent, ce qui signifie que les utilisateurs du domaine enfant `corp.inlanefreight.local`peuvent s'authentifier auprès du domaine parent `inlanefreight.local`, et vice versa.
-   `Cross-link`: Une confiance entre domaines enfants pour accélérer l'authentification.
-   `External`: Une approbation non transitive entre deux domaines distincts dans des forêts distinctes qui ne sont pas déjà rejointes par une approbation de forêt. Ce type de confiance utilise [le filtrage SID](https://www.serverbrain.org/active-directory-2008/sid-history-and-sid-filtering.html) ou filtre les demandes d'authentification (par SID) ne provenant pas du domaine approuvé.
-   `Tree-root`: Une approbation transitive bidirectionnelle entre un domaine racine de forêt et un nouveau domaine racine d'arborescence. Ils sont créés par conception lorsque vous configurez un nouveau domaine racine d'arborescence au sein d'une forêt.
-   `Forest`: Une approbation transitive entre deux domaines racine de forêt.
-   [ESAE](https://docs.microsoft.com/en-us/security/compass/esae-retirement) : Une forêt bastion utilisée pour gérer Active Directory.

Lors de l'établissement d'une fiducie, certains éléments peuvent être modifiés en fonction du business case.

Les fiducies peuvent être transitives ou non transitives.

-   Une `transitive`approbation signifie que l'approbation est étendue aux objets auxquels le domaine enfant fait confiance. Par exemple, disons que nous avons trois domaines. Dans une relation transitive, si `Domain A`a une confiance avec `Domain B`, et `Domain B`a une `transitive`confiance avec `Domain C`, alors `Domain A`fera automatiquement confiance à `Domain C`.
-   Dans un `non-transitive trust`, le domaine enfant lui-même est le seul à être approuvé.

![image](https://academy.hackthebox.com/storage/modules/143/transitive-trusts.png)

Adapté d' [ici](https://zindagitech.com/wp-content/uploads/2021/09/Picture2-Deepak-4.png.webp)

#### Table de confiance côte à côte

| Transitif | Non transitif |
| --- | --- |
| Partagé, 1 à plusieurs | Confiance directe |
| La confiance est partagée avec n'importe qui dans la forêt | Non étendu aux domaines enfants de niveau suivant |
| Les approbations de forêt, de racine d'arbre, parent-enfant et de liens croisés sont transitives | Typique pour les configurations de confiance externes ou personnalisées |

Une comparaison facile à faire peut être la livraison de colis à votre domicile. Pour une `transitive`fiducie, vous avez accordé la permission à toute personne de votre foyer (forêt) d'accepter un colis en votre nom. Pour une `non-transitive`confiance, vous avez donné des ordres stricts avec le colis que personne d'autre que le service de livraison et vous pouvez gérer le colis, et vous seul pouvez le signer.

Les fiducies peuvent être créées dans deux directions : unidirectionnelles ou bidirectionnelles (bidirectionnelles).

-   `One-way trust`: Les utilisateurs d'un `trusted`domaine peuvent accéder aux ressources d'un domaine de confiance, et non l'inverse.
-   `Bidirectional trust`: les utilisateurs des deux domaines d'approbation peuvent accéder aux ressources de l'autre domaine. Par exemple, dans une approbation bidirectionnelle entre `INLANEFREIGHT.LOCAL`et `FREIGHTLOGISTICS.LOCAL`, les utilisateurs de `INLANEFREIGHT.LOCAL`pourraient accéder aux ressources de `FREIGHTLOGISTICS.LOCAL`et vice versa.

Les approbations de domaine sont souvent mal configurées et peuvent nous fournir des chemins d'attaque involontaires critiques. En outre, les approbations établies pour faciliter leur utilisation peuvent ne pas être examinées ultérieurement pour déterminer les implications potentielles en matière de sécurité si la sécurité n'est pas prise en compte avant d'établir la relation d'approbation. Une fusion et acquisition (M&A) entre deux sociétés peut donner lieu à des fiducies bidirectionnelles avec les sociétés acquises, ce qui peut, sans le savoir, introduire un risque dans l'environnement de la société acquéreuse si la posture de sécurité de la société acquise est inconnue et non testée. Si quelqu'un souhaitait cibler votre organisation, il pourrait également se tourner vers l'autre société que vous avez acquise pour trouver une cible potentiellement plus facile à attaquer, lui permettant ainsi d'accéder indirectement à votre organisation. Il n'est pas rare de pouvoir effectuer une attaque telle que Kerberoasting contre un domaine en dehors du domaine principal et d'obtenir un utilisateur disposant d'un accès administratif au sein du domaine principal. J'ai effectué de nombreux tests d'intrusion où c'était le cas : je n'ai pas réussi à trouver un pied dans le domaine principal, mais j'ai pu trouver une faille dans un domaine de confiance qui, à son tour, m'a donné un pied à terre, voire des droits d'administrateur complets. dans le domaine principal. Ce type d'attaque « de bout en bout » pourrait être évité si la sécurité était considérée comme primordiale avant d'établir tout type de confiance de domaine. Lorsque nous examinons les relations de confiance, gardez ces réflexions à l'esprit lors de vos rapports. Souvent, nous constatons que l'organisation dans son ensemble ignore l'existence d'une relation de confiance avec un ou plusieurs domaines.

Vous trouverez ci-dessous une représentation graphique des différents types de confiance.

![image](https://academy.hackthebox.com/storage/modules/143/trusts-diagram.png)

* * * * *

Énumération des relations de confiance
--------------------------------------

Nous pouvons utiliser l' applet de commande [Get-ADTrust](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps) pour énumérer les relations d'approbation de domaine. Ceci est particulièrement utile si nous nous limitons à utiliser uniquement des outils intégrés.

#### Utilisation de Get-ADTrust

  Introduction aux approbations de domaine

```
PS C:\htb> Import-Module activedirectory
PS C:\htb> Get-ADTrust -Filter *

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=LOGISTICS.INLANEFREIGHT.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : False
IntraForest             : True
IsTreeParent            : False
IsTreeRoot              : False
Name                    : LOGISTICS.INLANEFREIGHT.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : f48a1169-2e58-42c1-ba32-a6ccb10057ec
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : LOGISTICS.INLANEFREIGHT.LOCAL
TGTDelegation           : False
TrustAttributes         : 32
TrustedPolicy           :
TrustingPolicy          :
TrustType               : Uplevel
UplevelOnly             : False
UsesAESKeys             : False
UsesRC4Encryption       : False

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=FREIGHTLOGISTICS.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : True
IntraForest             : False
IsTreeParent            : False
IsTreeRoot              : False
Name                    : FREIGHTLOGISTICS.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : 1597717f-89b7-49b8-9cd9-0801d52475ca
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : FREIGHTLOGISTICS.LOCAL
TGTDelegation           : False
TrustAttributes         : 8
TrustedPolicy           :
TrustingPolicy          :
TrustType               : Uplevel
UplevelOnly             : False
UsesAESKeys             : False
UsesRC4Encryption       : False

```

Le résultat ci-dessus montre que notre domaine actuel `INLANEFREIGHT.LOCAL`dispose de deux approbations de domaine. Le premier est with `LOGISTICS.INLANEFREIGHT.LOCAL`, et la `IntraForest`propriété montre qu'il s'agit d'un domaine enfant, et nous sommes actuellement positionnés dans le domaine racine de la forêt. La deuxième approbation concerne le domaine `FREIGHTLOGISTICS.LOCAL,`et la `ForestTransitive`propriété est définie sur `True`, ce qui signifie qu'il s'agit d'une approbation de forêt ou d'une approbation externe. Nous pouvons voir que les deux fiducies sont configurées pour être bidirectionnelles, ce qui signifie que les utilisateurs peuvent s'authentifier dans les deux sens. Il est important de le noter lors d'une évaluation. Si nous ne pouvons pas nous authentifier au sein d'une confiance, nous ne pouvons effectuer aucune énumération ni attaque au sein de la confiance.

Outre l'utilisation d'outils AD intégrés tels que le module Active Directory PowerShell, PowerView et BloodHound peuvent être utilisés pour énumérer les relations de confiance, le type de confiance établie et le flux d'authentification. Après avoir importé PowerView, nous pouvons utiliser la fonction [Get-DomainTrust](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainTrust/) pour énumérer les approbations existantes, le cas échéant.

#### Vérification des approbations existantes à l'aide de Get-DomainTrust

  Introduction aux approbations de domaine

```
PS C:\htb> Get-DomainTrust

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM

```

PowerView peut être utilisé pour effectuer un mappage de confiance de domaine et fournir des informations telles que le type de confiance (parent/enfant, externe, forêt) et la direction de la confiance (unidirectionnelle ou bidirectionnelle). Ces informations sont bénéfiques une fois que nous avons pris pied, et nous prévoyons de compromettre davantage l'environnement.

#### Utilisation de Get-DomainTrustMapping

  Introduction aux approbations de domaine

```
PS C:\htb> Get-DomainTrustMapping

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM

SourceName      : FREIGHTLOGISTICS.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:08 PM
WhenChanged     : 2/27/2022 12:02:41 AM

SourceName      : LOGISTICS.INLANEFREIGHT.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

```

À partir de là, nous pourrions commencer à effectuer un dénombrement parmi les fiducies. Par exemple, nous pourrions examiner tous les utilisateurs du domaine enfant :

#### Vérification des utilisateurs dans le domaine enfant à l'aide de Get-DomainUser

  Introduction aux approbations de domaine

```
PS C:\htb> Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName

samaccountname
--------------
htb-student_adm
Administrator
Guest
lab_adm
krbtgt

```

Un autre outil que nous pouvons utiliser pour obtenir Domain Trust est `netdom`. La `netdom query`sous-commande de l' `netdom`outil de ligne de commande de Windows peut récupérer des informations sur le domaine, notamment une liste de postes de travail, de serveurs et d'approbations de domaine.

#### Utiliser Netdom pour interroger la confiance de domaine

  Introduction aux approbations de domaine

```
C:\htb> netdom query /domain:inlanefreight.local trust
Direction Trusted\Trusting domain                         Trust type
========= =======================                         ==========

<->       LOGISTICS.INLANEFREIGHT.LOCAL
Direct
 Not found

<->       FREIGHTLOGISTICS.LOCAL
Direct
 Not found

The command completed successfully.

```

#### Utiliser Netdom pour interroger les contrôleurs de domaine

  Introduction aux approbations de domaine

```
C:\htb> netdom query /domain:inlanefreight.local dc
List of domain controllers with accounts in the domain:

ACADEMY-EA-DC01
The command completed successfully.

```

#### Utiliser Netdom pour interroger les postes de travail et les serveurs

  Introduction aux approbations de domaine

```
C:\htb> netdom query /domain:inlanefreight.local workstation
List of workstations with accounts in the domain:

ACADEMY-EA-MS01
ACADEMY-EA-MX01      ( Workstation or Server )

SQL01      ( Workstation or Server )
ILF-XRG      ( Workstation or Server )
MAINLON      ( Workstation or Server )
CISERVER      ( Workstation or Server )
INDEX-DEV-LON      ( Workstation or Server )
...SNIP...

```

Nous pouvons également utiliser BloodHound pour visualiser ces relations de confiance en utilisant la `Map Domain Trusts`requête prédéfinie. Ici, nous pouvons facilement voir qu'il existe deux fiducies bidirectionnelles.

#### Visualiser les relations de confiance dans BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/BH_trusts.png)

* * * * *

À partir de
-----------

Dans les sections suivantes, nous aborderons les attaques courantes que nous pouvons effectuer contre les approbations de domaine enfants -> parent et entre les approbations de forêt bidirectionnelles. Ces types d'attaques ne doivent pas être négligés, mais nous devons toujours vérifier auprès de notre client pour nous assurer que toutes les fiducies que nous découvrons lors de notre énumération entrent dans le champ de l'évaluation et que nous ne sortons pas des règles d'engagement.
