Introduction à l'escalade de privilèges Windows
==========================================

* * * * *

Après avoir pris pied, l'élévation de nos privilèges offrira plus d'options de persistance et peut révéler des informations stockées localement qui peuvent favoriser notre accès à l'environnement. L'objectif général de l'élévation des privilèges Windows est d'étendre notre accès à un système donné à un membre du groupe `Local Administrators` ou du `NT AUTHORITY\SYSTEM` [LocalSystem](https://docs.microsoft.com/en- us/windows/win32/services/localsystem-account) compte. Il peut cependant y avoir des scénarios dans lesquels la transmission à un autre utilisateur du système peut suffire à atteindre notre objectif. L'escalade des privilèges est généralement une étape vitale lors de tout engagement. Nous devons utiliser l'accès obtenu, ou certaines données (telles que les informations d'identification) trouvées uniquement une fois que nous avons une session dans un contexte élevé. Dans certains cas, l'élévation des privilèges peut être l'objectif ultime de l'évaluation si notre client nous engage pour une évaluation de type « image dorée » ou « évasion du poste de travail ». L'escalade de privilèges est souvent vitale pour continuer à travers un réseau vers notre objectif ultime, ainsi que pour le mouvement latéral.

Cela étant dit, nous devrons peut-être augmenter les privilèges pour l'une des raisons suivantes :

| | |
| --- | --- |
| 1. | Lors du test d'un client [gold image](https://www.techopedia.com/definition/29456/golden-image) la version du poste de travail et du serveur Windows d'un client |
| 2. | Pour élever les privilèges localement afin d'accéder à certaines ressources locales telles qu'une base de données |
| 3. | Pour obtenir un accès de niveau [NT AUTHORITY\System](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) sur une machine jointe à un domaine afin de prendre pied dans l'Active Directory du client environnement |
| 4. | Pour obtenir des informations d'identification pour se déplacer latéralement ou élever les privilèges au sein du réseau du client |

Il existe de nombreux outils à notre disposition en tant que testeurs d'intrusion pour aider à l'escalade des privilèges. Néanmoins, il est également essentiel de comprendre comment effectuer des vérifications d'escalade de privilèges et exploiter les failles "manuellement" dans la mesure du possible dans un scénario donné. Nous pouvons rencontrer des situations où un client nous place sur un poste de travail géré sans accès Internet, fortement pare-feu et les ports USB désactivés, de sorte que nous ne pouvons pas charger d'outils/scripts d'assistance. Dans ce cas, il serait crucial de bien comprendre les vérifications d'escalade des privilèges Windows en utilisant à la fois PowerShell et la ligne de commande Windows.

Les systèmes Windows présentent une vaste surface d'attaque. Voici quelques-unes des façons dont nous pouvons augmenter les privilèges :

| | |
| --- | --- |
| Abus des privilèges de groupe Windows | Abus des privilèges d'utilisateur Windows |
| Contournement du contrôle de compte d'utilisateur | Abus d'autorisations de service/fichier faibles |
| Tirer parti des exploits du noyau non corrigés | Vol d'identifiants |
| Capture de trafic | et plus. |

* * * * *

Scénario 1 - Surmonter les restrictions réseau
--------------------------------------------

Une fois, on m'a confié la tâche d'augmenter les privilèges sur un système fourni par le client sans accès à Internet et avec des ports USB bloqués. En raison du contrôle d'accès au réseau en place, je ne pouvais pas brancher ma machine d'attaque directement sur le réseau de l'utilisateur pour m'aider. Au cours de l'évaluation, j'avais déjà trouvé une faille de réseau dans laquelle le VLAN de l'imprimante était configuré pour autoriser la communication sortante sur les ports 80, 443 et 445. J'ai utilisé des méthodes d'énumération manuelles pour trouver une faille liée aux autorisations qui m'a permis d'augmenter les privilèges et effectuer un vidage manuel de la mémoire du processus `LSASS` . À partir de là, j'ai pu monter un partage SMB hébergé sur ma machine d'attaque sur le VLAN de l'imprimante et exfiltrer le fichier `LSASS` DMP. Avec ce fichier en main, j'ai utilisé `Mimikatz` hors ligne pour récupérer le hachage du mot de passe NTLM d'un administrateur de domaine, que je pouvais craquer hors ligne et utiliser pour accéder à un contrôleur de domaine à partir du système fourni par le client.

* * * * *

Scénario 2 - Pillage des actions ouvertes
----------------------------------

Lors d'une autre évaluation, je me suis retrouvé dans un environnement plutôt verrouillé, bien surveillé et sans défauts de configuration évidents ni services/applications vulnérables en cours d'utilisation. J'ai trouvé un partage de fichiers largement ouvert, permettant à tous les utilisateurs de répertorier son contenu et de télécharger les fichiers qui y sont stockés. Ce partage hébergeait des sauvegardes de machines virtuelles dans l'environnement. J'étais explicitement intéressé par les fichiers de disque dur virtuel (fichiers `.VMDK` et `.VHDX` ). Je pouvais accéder à ce partage à partir d'une machine virtuelle Windows, monter le disque dur virtuel `.VHDX` en tant que lecteur local et parcourir le système de fichiers. À partir de là, j'ai récupéré les ruches de registre `SYSTEM`, `SAM` et `SECURITY` , les ai déplacées vers ma boîte d'attaque Linux et extrait le hachage du mot de passe de l'administrateur local à l'aide de [secretsdump.py](https://github. com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py). L'organisation utilisait une image dorée et le hachage de l'administrateur local pouvait être utilisé pour obtenir un accès administrateur à presque tous les systèmes Windows via une attaque pass-the-hash.

* * * * *

Scénario 3 - Chasse aux identifiants et abus des privilèges du compte
-------------------------------------------------------- -------

Dans ce dernier scénario, j'ai été placé dans un réseau plutôt verrouillé dans le but d'accéder à des serveurs de base de données critiques. Le client m'a fourni un ordinateur portable avec un compte d'utilisateur de domaine standard, et je pouvais y charger des outils. J'ai finalement exécuté l'outil [Snaffler](https://github.com/SnaffCon/Snaffler) pour rechercher des partages de fichiers à la recherche d'informations sensibles. Je suis tombé sur des fichiers `.sql` contenant des informations d'identification de base de données à faible privilège vers une base de données sur l'un de leurs serveurs de base de données. J'ai utilisé un client MSSQL localement pour me connecter à la base de données à l'aide des informations d'identification de la base de données, activer le [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp- cmdshell-transact-sql?view=sql-server-ver15) procédure stockée et obtenir l'exécution de la commande locale. En utilisant cet accès en tant que compte de service, j'ai confirmé que j'avais le [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege), qui peut être utilisé pour l'élévation des privilèges locaux. J'ai téléchargé une version compilée personnalisée de [Juicy Potato](https://github.com/ohpe/juicy-potato) sur l'hôte pour faciliter l'élévation des privilèges, et j'ai pu ajouter un utilisateur administrateur local. L'ajout d'un utilisateur n'était pas idéal, mais mes tentatives pour obtenir une balise/reverse shell n'ont pas fonctionné. Grâce à cet accès, j'ai pu accéder à distance à l'hôte de la base de données et prendre le contrôle complet de l'une des bases de données des clients de l'entreprise.

* * * * *

Pourquoi l'escalade de privilèges se produit-elle ?
-------------------------------------

Il n'y a pas de raison unique pour laquelle le ou les hôtes d'une entreprise peuvent être victimes d'une escalade de privilèges, mais il existe plusieurs causes sous-jacentes possibles. Certaines raisons typiques pour lesquelles des défauts sont introduits et passent inaperçus sont le personnel et le budget. De nombreuses organisations n'ont tout simplement pas le personnel nécessaire pour suivre correctement les correctifs, la gestion des vulnérabilités, les évaluations internes périodiques (auto-évaluations), la surveillance continue et les initiatives plus importantes et plus gourmandes en ressources. Ces initiatives peuvent inclure des mises à niveau de postes de travail et de serveurs, ainsi que des audits de partage de fichiers (pour verrouiller des répertoires et sécuriser/supprimer des fichiers sensibles tels que des scripts ou des fichiers de configuration contenant des informations d'identification).

* * * * *

Passer à autre chose
---------

Les scénarios ci-dessus montrent à quel point la compréhension de l'élévation des privilèges Windows est cruciale pour un testeur d'intrusion. Dans le monde réel, nous attaquerons rarement un seul hôte et nous devons être capables de réfléchir rapidement. Nous devrions être en mesure de trouver des moyens créatifs d'augmenter les privilèges et d'utiliser cet accès pour poursuivre notre progression vers l'objectif de l'évaluation.

* * * * *

Exemples pratiques
------------------

Tout au long du module, nous couvrirons des exemples accompagnés de la sortie de commande, dont la plupart peuvent être reproduits sur les machines virtuelles cibles qui peuvent être générées dans les sections pertinentes. Vous recevrez des informations d'identification RDP pour interagir avec les machines virtuelles cibles et effectuer les exercices de la section et les évaluations des compétences. Vous pouvez vous connecter depuis la Pwnbox ou votre propre machine virtuelle (après avoir téléchargé une clé VPN une fois qu'une machine apparaît) via RDP en utilisant [FreeRDP](https://github.com/FreeRDP/FreeRDP/wiki/CommandLineInterface), [Remmina](https ://remmina.org/), ou le client RDP de votre choix.

#### Connexion via FreeRDP

Nous pouvons nous connecter via la ligne de commande en utilisant la commande `xfreerdp /v:<target ip> /u:htb-student` et en tapant le mot de passe fourni lorsque vous y êtes invité. La plupart des sections fourniront des informations d'identification pour l'utilisateur `htb-student` , mais certaines, en fonction du matériel, vous feront passer RDP avec un utilisateur différent, et d'autres informations d'identification seront fournies.

Connexion via FreeRDP

```
dsgsec@htb[/htb]$ xfreerdp /v:10.129.43.36 /u:htb-student

[21:17:27:323] [28158:28159] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex réinitialisation de l'état d'erreur
[21:17:27:323] [28158:28159] [INFO][com.freerdp.client.common.cmdline] - chargement de channelEx rdpdr
[21:17:27:324] [28158:28159] [INFO][com.freerdp.client.common.cmdline] - chargement de channelEx rdpsnd
[21:17:27:324] [28158:28159] [INFO][com.freerdp.client.common.cmdline] - chargement de channelEx cliprdr
[21:17:27:648] [28158:28159] [INFO][com.freerdp.primitives] - détection automatique des primitives, en utilisant optimisé
[21:17:27:672] [28158:28159] [INFO][com.freerdp.core] - freerdp_tcp_is_hostname_resolvable:freerdp_set_last_error_ex état d'erreur de réinitialisation
[21:17:27:672] [28158:28159] [INFO][com.freerdp.core] - freerdp_tcp_connect:freerdp_set_last_error_ex réinitialisant l'état d'erreur
[21:17:28:770] [28158:28159] [INFO][com.freerdp.crypto] - création du répertoire /home/user2/.config/freerdp
[21:17:28:770] [28158:28159] [INFO][com.freerdp.crypto] - création du répertoire [/home/user2/.config/freerdp/certs]
[21:17:28:771] [28158:28159] [INFO][com.freerdp.crypto] - répertoire créé [/home/user2/.config/freerdp/server]
[21:17:28:794] [28158:28159] [WARN][com.freerdp.crypto] - Échec de la vérification du certificat « certificat auto-signé (18) » à la position de pile 0
[21:17:28:794] [28158:28159] [WARN][com.freerdp.crypto] - CN = WINLPE-SKILLS1-SRV
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@ @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - @ AVERTISSEMENT : NOM DU CERTIFICAT INCOMPATIBLE ! @
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@ @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - Le nom d'hôte utilisé pour cette connexion (10.129.43.36:3389)
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - ne correspond pas au nom donné dans le certificat :
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - Nom commun (CN) :
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - WINLPE-SKILLS1-SRV
[21:17:28:795] [28158:28159] [ERREUR][com.freerdp.crypto] - Un certificat valide pour le mauvais nom ne doit PAS être approuvé !
Détails du certificat pour 10.129.43.36:3389 (RDP-Server) :
Nom commun : WINLPE-SKILLS1-SRV
Objet : CN = WINLPE-SKILLS1-SRV
Émetteur : CN = WINLPE-SKILLS1-SRV
Empreinte digitale : 9f:f0:dd:28:f5:6f:83:db:5e:8c:5a:e9:5f:50:a4:50:2d:b3:e7:a7:af:f4:4a:8a : 1a:08:f3:cb:46:c3:c3:e8
Le certificat X.509 ci-dessus n'a pas pu être vérifié, peut-être parce que vous n'avez pas
le certificat CA dans votre magasin de certificats ou le certificat a expiré.
Veuillez consulter la documentation OpenSSL pour savoir comment ajouter une autorité de certification privée au magasin.
Faites-vous confiance au certificat ci-dessus ? (O/T/N) oui
Mot de passe:

```

De nombreuses sections du module nécessitent des outils tels que des scripts open source, des binaires précompilés et des PoC d'exploitation. Le cas échéant, ils se trouvent dans le répertoire `C:\Tools` sur l'hôte cible. Même si la plupart des outils sont fournis, mettez-vous au défi de télécharger des fichiers vers la cible (en utilisant les techniques présentées dans le [module de transfert de fichiers](https://academy.hackthebox.com/course/preview/file-transfers)) et même de compiler certains des outils par vous-même à l'aide de [Visual Studio](https://visualstudio.microsoft.com/downloads/).