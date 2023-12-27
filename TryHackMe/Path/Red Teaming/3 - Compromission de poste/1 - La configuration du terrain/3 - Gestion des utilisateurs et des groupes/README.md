Dans cette tâche, nous en apprendrons davantage sur les utilisateurs et les groupes, notamment au sein d'Active Directory. La collecte d'informations sur la machine compromise est essentielle  et pourrait être utilisée à l'étape suivante. La découverte de comptes est la première étape une fois que nous avons obtenu un premier accès à la machine compromise pour comprendre ce que nous avons et quels autres comptes se trouvent dans le système. 

![d981e1c73997b17ace78454c87b8a8c1](https://github.com/dsgsec/Red-Team/assets/82456829/6467020c-91c5-42f0-b1c1-a259a73dc2e9)

Un environnement Active Directory contient différents comptes avec les autorisations, accès et rôles nécessaires à différentes fins. Les comptes de service Active Directory courants incluent les comptes d'utilisateurs locaux intégrés, les comptes d'utilisateurs de domaine, les comptes de services gérés et les comptes virtuels. 

-   Les comptes d'utilisateurs locaux intégrés sont utilisés pour gérer le système localement, qui ne fait pas partie de l' environnement AD .
-   Les comptes d'utilisateurs de domaine ayant accès à un environnement Active Directory peuvent utiliser les services AD (gérés par AD).
-   Les comptes de services gérés AD sont des comptes d'utilisateurs de domaine limités dotés de privilèges plus élevés pour gérer les services AD.
-   Les administrateurs de domaine sont des comptes d'utilisateurs qui peuvent gérer les informations dans un environnement Active Directory, y compris les configurations AD , les utilisateurs, les groupes, les autorisations, les rôles, les services, etc. L'un des objectifs de l'équipe rouge en matière d'engagement est de rechercher des informations menant à un administrateur de domaine. avoir un contrôle total sur l'environnement AD.

Voici les comptes d'administrateurs Active Directory :

**CONSTRUIT\Administrateur**|**Accès administrateur local sur un contrôleur de domaine**
:-----:|:-----:
Administrateurs de domaine|Accès administratif à toutes les ressources du domaine
Administrateurs d'entreprise|Disponible uniquement dans la racine de la forêt
Administrateurs de schéma|Capable de modifier le domaine/la forêt ; utile pour les équipes rouges
Opérateurs de serveur|Peut gérer des serveurs de domaine
Opérateurs de comptes|Peut gérer les utilisateurs qui ne font pas partie de groupes

Maintenant que nous découvrons les différents types de comptes dans l' environnement AD . Énumérons la machine Windows  à laquelle nous avons accès lors de la phase d'accès initiale. En tant qu'utilisateur actuel, nous disposons d'autorisations spécifiques pour afficher ou gérer des éléments au sein de la machine et de l' environnement AD . 

Énumération Active Directory ( AD )

Désormais, l'énumération dans l' environnement AD nécessite différents outils et techniques.  Une fois que nous avons confirmé que la machine fait partie de l' environnement AD , nous pouvons commencer à rechercher toute information variable qui pourrait être utilisée ultérieurement. Dans cette étape, nous utilisons PowerShell pour énumérer les utilisateurs et les groupes.

La commande PowerShell suivante permet d'obtenir tous les comptes d'utilisateurs Active Directory. Notez que nous devons utiliser   l'argument -Filter .

PowerShell

```
PS C:\Users\thm> Get-ADUser  -Filter *
DistinguishedName : CN=Administrator,CN=Users,DC=thmredteam,DC=com
Enabled           : True
GivenName         :
Name              : Administrator
ObjectClass       : user
ObjectGUID        : 4094d220-fb71-4de1-b5b2-ba18f6583c65
SamAccountName    : Administrator
SID               : S-1-5-21-1966530601-3185510712-10604624-500
Surname           :
UserPrincipalName :
PS C:\Users\thm>
```

Nous pouvons également utiliser l' [arborescence hiérarchique LDAP ](http://www.ietf.org/rfc/rfc2253.txt)pour rechercher un utilisateur dans l' environnement AD . Le nom distinctif (DN) est une collection de paires clé-valeur séparées par des virgules utilisées pour identifier des enregistrements uniques dans le répertoire. Le DN se compose du composant de domaine (DC), du nom d'unité organisationnelle (OU), du nom commun (CN) et d'autres. Le  « CN=User1,CN=Users,DC=thmredteam,DC=com » suivant  est un exemple de DN, qui peut être visualisé comme suit :

![764c72d40ec3d823b05d6473702e00f5](https://github.com/dsgsec/Red-Team/assets/82456829/db810ebe-0f08-4cd0-8db0-ad34794ad9d6)

À l'aide de l'  option SearchBase , nous spécifions un  CN Common-Name spécifique dans le répertoire actif. Par exemple, nous pouvons spécifier de répertorier tous les utilisateurs faisant partie de  Users .

PowerShell

```
PS C:\Users\thm> Get-ADUser -Filter * -SearchBase "CN=Users,DC=THMREDTEAM,DC=COM"

DistinguishedName : CN=Administrator,CN=Users,DC=thmredteam,DC=com
Enabled           : True
GivenName         :
Name              : Administrator
ObjectClass       : user
ObjectGUID        : 4094d220-fb71-4de1-b5b2-ba18f6583c65
SamAccountName    : Administrator
SID               : S-1-5-21-1966530601-3185510712-10604624-500
Surname           :
UserPrincipalName :
```

Notez que le résultat peut contenir plusieurs utilisateurs selon la configuration du CN.
