Guide d'utilisation abusive de la liste de contrôle d'accès (ACL)
=================================================================

* * * * *

Pour des raisons de sécurité, tous les utilisateurs et ordinateurs d'un environnement AD ne peuvent pas accéder à tous les objets et fichiers. Ces types d'autorisations sont contrôlés via des listes de contrôle d'accès (ACL). Présentant une menace sérieuse pour la posture de sécurité du domaine, une légère mauvaise configuration d'une liste de contrôle d'accès peut divulguer des autorisations à d'autres objets qui n'en ont pas besoin.

* * * * *

Présentation de la liste de contrôle d'accès (ACL)
--------------------------------------------------

Dans leur forme la plus simple, les ACL sont des listes qui définissent a) qui a accès à quel actif/ressource et b) le niveau d'accès qui leur est fourni. Les paramètres eux-mêmes dans une ACL sont appelés `Access Control Entities`( `ACEs`). Chaque ACE correspond à un utilisateur, un groupe ou un processus (également appelés principaux de sécurité) et définit les droits accordés à ce principal. Chaque objet a une ACL, mais peut avoir plusieurs ACE car plusieurs principaux de sécurité peuvent accéder aux objets dans AD. Les ACL peuvent également être utilisées pour auditer l'accès au sein d'AD.

Il existe deux types d'ACL :

1.  `Discretionary Access Control List`( `DACL`) - définit les entités de sécurité auxquelles l'accès à un objet est accordé ou refusé. Les DACL sont constituées d'ACE qui autorisent ou refusent l'accès. Lorsque quelqu'un tente d'accéder à un objet, le système vérifie la DACL pour le niveau d'accès autorisé. Si un DACL n'existe pas pour un objet, tous ceux qui tentent d'accéder à l'objet se voient accorder tous les droits. Si un DACL existe, mais n'a pas d'entrées ACE spécifiant des paramètres de sécurité spécifiques, le système refusera l'accès à tous les utilisateurs, groupes ou processus qui tentent d'y accéder.

2.  `System Access Control Lists`( `SACL`) - permet aux administrateurs de consigner les tentatives d'accès aux objets sécurisés.

Nous voyons l'ACL pour le compte d'utilisateur `forend`dans l'image ci-dessous. Chaque élément sous `Permission entries`constitue le `DACL`pour le compte d'utilisateur, tandis que les entrées individuelles (telles que `Full Control`ou `Change Password`) sont des entrées ACE indiquant les droits accordés sur cet objet utilisateur à divers utilisateurs et groupes.

#### Affichage de l'ACL de forend

![image](https://academy.hackthebox.com/storage/modules/143/DACL_example.png)

Les SACL sont visibles dans l' `Auditing`onglet.

#### Affichage des SACL via l'onglet Audit

![image](https://academy.hackthebox.com/storage/modules/143/SACL_example.png)

* * * * *

Entités de contrôle d'accès (ACE)
---------------------------------

Comme indiqué précédemment, les listes de contrôle d'accès (ACL) contiennent des entrées ACE qui nomment un utilisateur ou un groupe et le niveau d'accès dont ils disposent sur un objet sécurisable donné. Il existe `three`les principaux types d'ACE qui peuvent être appliqués à tous les objets sécurisables dans AD :

| AS | Description |
| --- | --- |
| `Access denied ACE` | Utilisé dans une DACL pour montrer qu'un utilisateur ou un groupe se voit explicitement refuser l'accès à un objet |
| `Access allowed ACE` | Utilisé dans une DACL pour montrer qu'un utilisateur ou un groupe est explicitement autorisé à accéder à un objet |
| `System audit ACE` | Utilisé dans une SACL pour générer des journaux d'audit lorsqu'un utilisateur ou un groupe tente d'accéder à un objet. Il enregistre si l'accès a été accordé ou non et quel type d'accès s'est produit |

Chaque ACE est composé des `four`composants suivants :

1.  L'identifiant de sécurité (SID) de l'utilisateur/groupe qui a accès à l'objet (ou nom principal graphiquement)
2.  Un indicateur indiquant le type d'ACE (accès refusé, autorisé ou ACE d'audit système)
3.  Un ensemble d'indicateurs qui spécifient si les conteneurs/objets enfants peuvent ou non hériter de l'entrée ACE donnée de l'objet principal ou parent
4.  Un [masque d'accès](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN) qui est une valeur 32 bits qui définit les droits accordés à un objet

Nous pouvons voir cela graphiquement dans `Active Directory Users and Computers`( `ADUC`). Dans l'exemple d'image ci-dessous, nous pouvons voir ce qui suit pour l'entrée ACE pour l'utilisateur`forend` :

#### Affichage des autorisations via les utilisateurs et les ordinateurs Active Directory

![image](https://academy.hackthebox.com/storage/modules/143/ACE_example.png)

1.  Le responsable de la sécurité est Angela Dunn ( adunn@inlanefreight.local )
2.  Le type ACE est`Allow`
3.  L'héritage s'applique à "Cet objet et tous les objets descendants", ce qui signifie que tous les objets enfants de l' `forend`objet auraient les mêmes autorisations accordées
4.  Les droits accordés à l'objet, à nouveau représentés graphiquement dans cet exemple

Lorsque les listes de contrôle d'accès sont vérifiées pour déterminer les autorisations, elles sont vérifiées de haut en bas jusqu'à ce qu'un accès refusé soit trouvé dans la liste.

* * * * *

Pourquoi les ACE sont-ils importants ?
--------------------------------------

Les attaquants utilisent les entrées ACE pour accéder plus avant ou établir la persistance. Ceux-ci peuvent être formidables pour nous en tant que testeurs d'intrusion, car de nombreuses organisations ne sont pas conscientes des ACE appliqués à chaque objet ou de l'impact que ceux-ci peuvent avoir s'ils sont appliqués de manière incorrecte. Ils ne peuvent pas être détectés par les outils d'analyse des vulnérabilités et restent souvent incontrôlés pendant de nombreuses années, en particulier dans les environnements vastes et complexes. Lors d'une évaluation où le client s'est occupé de tous les défauts/mauvaises configurations AD « à portée de main », l'abus d'ACL peut être un excellent moyen pour nous de nous déplacer latéralement/verticalement et même d'obtenir un compromis complet sur le domaine. Voici quelques exemples d'autorisations de sécurité d'objet Active Directory. Celles-ci peuvent être énumérées (et visualisées) à l'aide d'un outil tel que BloodHound, et sont toutes exploitables avec PowerView, entre autres outils :

-   `ForceChangePassword`maltraité avec`Set-DomainUserPassword`
-   `Add Members`maltraité avec`Add-DomainGroupMember`
-   `GenericAll`maltraité avec `Set-DomainUserPassword`ou`Add-DomainGroupMember`
-   `GenericWrite`maltraité avec`Set-DomainObject`
-   `WriteOwner`maltraité avec`Set-DomainObjectOwner`
-   `WriteDACL`maltraité avec`Add-DomainObjectACL`
-   `AllExtendedRights`maltraité avec `Set-DomainUserPassword`ou`Add-DomainGroupMember`
-   `Addself`maltraité avec`Add-DomainGroupMember`

Dans ce module, nous couvrirons l'énumération et l'exploitation de quatre ACE spécifiques pour mettre en évidence la puissance des attaques ACL :

-   [ForceChangePassword](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#forcechangepassword) - nous donne le droit de réinitialiser le mot de passe d'un utilisateur sans connaître d'abord son mot de passe (doit être utilisé avec prudence et généralement préférable de consulter notre client avant de réinitialiser les mots de passe).
-   [GenericWrite](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericwrite) - nous donne le droit d'écrire dans n'importe quel attribut non protégé d'un objet. Si nous avons cet accès sur un utilisateur, nous pourrions lui attribuer un SPN et effectuer une attaque Kerberoasting (qui repose sur le compte cible ayant un mot de passe faible). Sur un groupe signifie que nous pourrions nous ajouter ou ajouter un autre principal de sécurité à un groupe donné. Enfin, si nous avons cet accès sur un objet informatique, nous pourrions effectuer une attaque par délégation contrainte basée sur les ressources qui sort du cadre de ce module.
-   `AddSelf`- affiche les groupes de sécurité auxquels un utilisateur peut s'ajouter.
-   [GenericAll](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericall) - cela nous accorde un contrôle total sur un objet cible. Encore une fois, selon que cela est accordé à un utilisateur ou à un groupe, nous pourrions modifier l'appartenance à un groupe, forcer le changement d'un mot de passe ou effectuer une attaque Kerberoasting ciblée. Si nous avons cet accès sur un objet informatique et que la [solution de mot de passe de l'administrateur local (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) est utilisée dans l'environnement, nous pouvons lire le mot de passe LAPS et obtenir un accès administrateur local à la machine, ce qui peut nous aider dans les déplacements latéraux ou l'élévation des privilèges dans le domaine si nous pouvons obtenir des contrôles privilégiés ou obtenir une sorte d'accès privilégié.

Ce graphique, adapté d'un graphique créé par [Charlie Bromberg (Shutdown)](https://twitter.com/_nwodtuhs) , montre une excellente répartition des différentes attaques ACE possibles et des outils pour effectuer ces attaques à partir de Windows et de Linux (le cas échéant). Dans les quelques sections suivantes, nous couvrirons principalement l'énumération et l'exécution de ces attaques à partir d'un hôte d'attaque Windows avec des mentions de la façon dont ces attaques pourraient être effectuées à partir de Linux. Un module ultérieur spécifiquement sur les attaques ACL ira beaucoup plus en profondeur sur chacune des attaques répertoriées dans ce graphique et comment les exécuter à partir de Windows et Linux.

![image](https://academy.hackthebox.com/storage/modules/143/ACL_attacks_graphic.png)

Nous rencontrerons de temps en temps de nombreux autres ACE (privilèges) intéressants dans Active Directory. La méthodologie d'énumération des attaques ACL possibles à l'aide d'outils tels que BloodHound et PowerView et même des outils de gestion AD intégrés devrait être suffisamment adaptable pour nous aider chaque fois que nous rencontrons de nouveaux privilèges dans la nature avec lesquels nous ne sommes peut-être pas encore familiers. Par exemple, nous pouvons importer des données dans BloodHound et voir qu'un utilisateur que nous contrôlons (ou que nous pouvons potentiellement prendre en charge) a le droit de lire le mot de passe d'un compte de service géré de groupe (gMSA) via la périphérie ReadGMSAPassword [. ](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#readgmsapassword)Dans ce cas, il existe des outils tels que [GMSAPasswordReader](https://github.com/rvazarkar/GMSAPasswordReader)que nous pourrions utiliser, ainsi que d'autres méthodes, pour obtenir le mot de passe du compte de service en question. D'autres fois, nous pouvons rencontrer des droits étendus tels que [Unexpire-Password](https://learn.microsoft.com/en-us/windows/win32/adschema/r-unexpire-password) ou [Reanimate-Tombstones](https://learn.microsoft.com/en-us/windows/win32/adschema/r-reanimate-tombstones) en utilisant PowerView et nous devons faire un peu de recherche pour comprendre comment les exploiter à notre avantage. Il vaut la peine de vous familiariser avec tous les [bords BloodHound](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html) et autant de [droits étendus](https://learn.microsoft.com/en-us/windows/win32/adschema/extended-rights) Active Directory que possible, car vous ne savez jamais quand vous pouvez en rencontrer un moins courant lors d'une évaluation.

* * * * *

Attaques ACL dans la nature
---------------------------

Nous pouvons utiliser les attaques ACL pour :

-   Mouvement latéral
-   Escalade des privilèges
-   Persistance

Certains scénarios d'attaque courants peuvent inclure :

| Attaque | Description |
| --- | --- |
| `Abusing forgot password permissions` | Le service d'assistance et d'autres utilisateurs informatiques reçoivent souvent des autorisations pour effectuer des réinitialisations de mot de passe et d'autres tâches privilégiées. Si nous pouvons reprendre un compte avec ces privilèges (ou un compte dans un groupe qui confère ces privilèges à ses utilisateurs), nous pourrons peut-être effectuer une réinitialisation du mot de passe pour un compte plus privilégié dans le domaine. |
| `Abusing group membership management` | Il est également courant de voir le service d'assistance et d'autres membres du personnel qui ont le droit d'ajouter/de supprimer des utilisateurs d'un groupe donné. Cela vaut toujours la peine d'énumérer cela plus en détail, car nous pouvons parfois ajouter un compte que nous contrôlons dans un groupe AD intégré privilégié ou un groupe qui nous accorde une sorte de privilège intéressant. |
| `Excessive user rights` | Nous voyons également couramment des objets utilisateur, ordinateur et groupe avec des droits excessifs dont un client n'est probablement pas conscient. Cela peut se produire après une sorte d'installation de logiciel (Exchange, par exemple, ajoute de nombreuses modifications ACL dans l'environnement au moment de l'installation) ou une sorte de configuration héritée ou accidentelle qui donne à un utilisateur des droits involontaires. Parfois, nous pouvons reprendre un compte auquel certains droits ont été accordés par commodité ou pour résoudre plus rapidement un problème lancinant. |

Il existe de nombreux autres scénarios d'attaque possibles dans le monde des ACL Active Directory, mais ces trois sont les plus courants. Nous couvrirons l'énumération de ces droits de différentes manières, l'exécution des attaques et le nettoyage après nous.

Remarque : certaines attaques ACL peuvent être considérées comme « destructrices », comme la modification du mot de passe d'un utilisateur ou l'exécution d'autres modifications dans le domaine AD d'un client. En cas de doute, il est toujours préférable d'exécuter une attaque donnée par notre client avant de l'exécuter pour avoir une documentation écrite de son approbation en cas de problème. Nous devons toujours soigneusement documenter nos attaques du début à la fin et annuler tout changement. Ces données doivent être incluses dans notre rapport, mais nous devons également souligner clairement toutes les modifications que nous apportons afin que le client puisse revenir en arrière et vérifier que nos modifications ont bien été annulées correctement.
