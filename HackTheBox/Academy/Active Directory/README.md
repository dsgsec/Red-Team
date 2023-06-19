Introduction à l'énumération et aux attaques Active Directory
======================================================

* * * * *

Active Directory expliqué
--------------------------

`Active Directory` (`AD`) est un service d'annuaire pour les environnements d'entreprise Windows qui a été officiellement mis en œuvre en 2000 avec la sortie de Windows Server 2000 et a été progressivement amélioré avec la sortie de chaque système d'exploitation de serveur ultérieur depuis. AD est basé sur les protocoles x.500 et LDAP qui l'ont précédé et utilise encore ces protocoles sous une forme ou une autre aujourd'hui. Il s'agit d'une structure hiérarchique distribuée qui permet une gestion centralisée des ressources d'une organisation, y compris les utilisateurs, les ordinateurs, les groupes, les périphériques réseau et les partages de fichiers, les stratégies de groupe, les périphériques et les approbations. AD fournit des fonctions d'authentification, de comptabilité et d'autorisation dans un environnement d'entreprise Windows. Si c'est la première fois que vous découvrez Active Directory ou que vous entendez ces termes, consultez le module [Introduction à Active Directory](https://academy.hackthebox.com/catalogue) pour un examen plus approfondi de la structure et de la fonction. d'AD, d'objets AD, etc.

* * * * *

Pourquoi devrions-nous nous soucier de la DA ?
-------------------------------------

Au moment de la rédaction de ce module, Microsoft Active Directory détient environ `43 %` de la [part de marché](https://www.slintel.com/tech/identity-and-access-management/microsoft-active-directory- market-share#faqs) pour les entreprises utilisant des solutions de gestion des identités et des accès. Il s'agit d'une énorme partie du marché, et il est peu probable qu'elle aille de sitôt puisque Microsoft améliore et fusionne les implémentations avec Azure AD. Une autre statistique intéressante à prendre en compte est qu'au cours des deux dernières années, Microsoft a signalé plus de `2 000 vulnérabilités liées à un [CVE](https://www.cvedetails.com/vendor/26/Microsoft.html). Les nombreux services d'AD et son objectif principal de faciliter la recherche et l'accès aux informations en font un peu un monstre à gérer et à renforcer correctement. Cela expose les entreprises aux vulnérabilités et à l'exploitation à partir de simples erreurs de configuration des services et des autorisations. Associez ces erreurs de configuration et la facilité d'accès aux vulnérabilités courantes des utilisateurs et du système d'exploitation, et vous avez une tempête parfaite dont un attaquant peut profiter. Avec tout cela à l'esprit, ce module explorera certains de ces problèmes communs et nous montrera comment identifier, énumérer et tirer parti de leur existence. Nous nous entraînerons à énumérer AD à l'aide d'outils et de langages natifs tels que `Sysinternals`, `WMI`, `DNS` et bien d'autres. Certaines attaques que nous allons également pratiquer incluent `Password spraying`, `Kerberoasting`, en utilisant des outils tels que `Responder`, `Kerbrute`, `Bloodhound`, et bien plus encore.

Nous pouvons souvent nous retrouver dans un réseau sans chemin clair pour prendre pied via un exploit à distance tel qu'une application ou un service vulnérable. Pourtant, nous sommes dans un environnement Active Directory, ce qui peut conduire à une prise de pied de plusieurs manières. L'objectif général de prendre pied dans l'environnement AD d'un client est d'augmenter les privilèges en se déplaçant latéralement ou verticalement sur le réseau jusqu'à ce que nous accomplissions l'objectif de l'évaluation. L'objectif peut varier d'un client à l'autre. Il peut s'agir d'accéder à un hôte spécifique, à la boîte de réception de l'utilisateur, à la base de données ou simplement de compromettre complètement le domaine et de rechercher tous les chemins possibles vers l'accès au niveau de l'administrateur de domaine pendant la période de test. De nombreux outils open source sont disponibles pour faciliter l'énumération et l'attaque d'Active Directory. Pour être plus efficace, nous devons comprendre comment effectuer autant de cette énumération manuellement que possible. Plus important encore, nous devons comprendre le "pourquoi" derrière certains défauts et erreurs de configuration. Cela nous rendra plus efficaces en tant qu'attaquants et nous permettra de donner des recommandations judicieuses à nos clients sur les principaux problèmes de leur environnement, ainsi que des conseils de remédiation clairs et exploitables.

Nous devons être à l'aise pour énumérer et attaquer AD depuis Windows et Linux, avec un ensemble d'outils limité ou des outils Windows intégrés, également connus sous le nom de "`vivre de la terre`". Il est courant de se retrouver dans des situations où nos outils échouent, sont bloqués ou nous effectuons une évaluation où le client nous fait travailler à partir d'un `poste de travail géré` ou d'une `instance VDI` au lieu de l'hôte d'attaque Linux ou Windows personnalisé que nous pouvons se sont habitués. Pour être efficace dans toutes les situations, nous devons être capables de nous adapter rapidement à la volée, comprendre les nombreuses nuances de la DA et savoir y accéder même lorsque nos options sont sévèrement limitées.

* * * * *

Exemples concrets
-------------------

Examinons quelques scénarios pour voir ce qui est possible dans un engagement centré sur l'AD dans le monde réel :

Scénario 1 - En attente d'un administrateur

Au cours de cet engagement, j'ai compromis un seul hôte et obtenu un accès de niveau `SYSTEM`. Comme il s'agissait d'un hôte joint à un domaine, j'ai pu utiliser cet accès pour énumérer le domaine. J'ai parcouru toute l'énumération standard, mais je n'ai pas trouvé grand-chose. Il y avait `Service Principal Names` (SPN) présents dans l'environnement, et j'ai pu effectuer une attaque Kerberoasting et récupérer des tickets TGS pour quelques comptes. J'ai essayé de les déchiffrer avec Hashcat et certaines de mes listes de mots et règles standard, mais sans succès au début. J'ai fini par laisser un travail de craquage en cours d'exécution du jour au lendemain avec une très grande liste de mots associée à la règle [d3ad0ne](https://github.com/hashcat/hashcat/blob/master/rules/d3ad0ne.rule) fournie avec Hashcat. Le lendemain matin, j'ai eu un coup sur un ticket et j'ai récupéré le mot de passe en clair pour un compte d'utilisateur. Ce compte ne m'a pas donné un accès significatif, mais il m'a donné un accès en écriture sur certains partages de fichiers. J'ai utilisé cet accès pour déposer des fichiers SCF autour des partages et laisser Responder en marche. Au bout d'un moment, j'ai reçu un seul résultat, le `hachage NetNTLMv2` d'un utilisateur. J'ai vérifié la sortie de BloodHound et j'ai remarqué que cet utilisateur était en fait un administrateur de domaine ! Journée facile à partir d'ici.

* * * * *

Scénario 2 - Pulvériser toute la nuit

La pulvérisation de mots de passe peut être un moyen extrêmement efficace de prendre pied dans un domaine, mais nous devons faire très attention à ne pas verrouiller les comptes d'utilisateurs dans le processus. Lors d'un engagement, j'ai trouvé une session SMB NULL à l'aide de l'outil [enum4linux](https://github.com/CiscoCXSecurity/enum4linux) et j'ai récupéré à la fois une liste de `tous` les utilisateurs du domaine et la `politique de mot de passe` du domaine. . Connaître la politique de mot de passe était crucial car je pouvais m'assurer que je restais dans les paramètres pour ne verrouiller aucun compte et je savais également que la politique était un mot de passe minimum de huit caractères et que la complexité du mot de passe était appliquée (ce qui signifie que le mot de passe d'un utilisateur nécessitait 3/ 4 de caractère spécial, chiffre, majuscule ou minuscule, c'est-à-dire Bienvenue1). J'ai essayé plusieurs mots de passe faibles courants tels que Welcome1, `Password1`, Password123, `Spring2018`, etc., mais je n'ai obtenu aucun résultat. Enfin, j'ai fait une tentative avec `Spring@18` et j'ai eu un succès ! En utilisant ce compte, j'ai exécuté BloodHound et trouvé plusieurs hôtes où cet utilisateur avait un accès administrateur local. J'ai remarqué qu'un compte d'administrateur de domaine avait une session active sur l'un de ces hôtes. J'ai pu utiliser l'outil Rubeus et extraire le ticket Kerberos TGT pour cet utilisateur de domaine. À partir de là, j'ai pu effectuer une attaque `pass-the-ticket` et m'authentifier en tant qu'administrateur de domaine. En prime, j'ai également pu prendre en charge le domaine d'approbation, car le groupe d'administrateurs de domaine du domaine que j'ai repris faisait partie du groupe d'administrateurs du domaine d'approbation via l'appartenance à un groupe imbriqué, ce qui signifie que je pouvais utiliser le même ensemble. d'informations d'identification pour s'authentifier auprès de l'autre domaine avec un accès complet au niveau administratif.

* * * * *

Scénario 3 - Se battre dans le noir

J'avais essayé tous mes moyens habituels pour prendre pied sur ce troisième engagement, et rien n'avait fonctionné. J'ai décidé d'utiliser l'outil [Kerbrute](https://github.com/ropnop/kerbrute) pour tenter d'énumérer les noms d'utilisateur valides, puis, si j'en trouvais, de tenter une attaque ciblée par pulvérisation de mot de passe, car je ne connaissais pas le politique de mot de passe et ne voulait verrouiller aucun compte. J'ai utilisé l'outil [linkedin2username](https://github.com/initstring/linkedin2username) pour d'abord mélanger les noms d'utilisateur potentiels de la page LinkedIn de l'entreprise. J'ai combiné cette liste avec plusieurs listes de noms d'utilisateurs du dépôt [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) GitHub et, après avoir utilisé la fonctionnalité `userenum` de Kerbrute, j'ai fini par avec 516 utilisateurs valides. Je savais que je devais être prudent avec la pulvérisation de mot de passe, alors j'ai essayé avec le mot de passe `Welcome2021` et j'ai obtenu un seul résultat ! À l'aide de ce compte, j'ai exécuté la version Python de BloodHound à partir de mon hôte d'attaque et j'ai constaté que tous les utilisateurs du domaine avaient un accès RDP à une seule boîte. Je me suis connecté à cet hôte et j'ai utilisé l'outil PowerShell [DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray) pour pulvériser à nouveau. J'étais plus confiant cette fois-ci parce que je pouvais a) voir la politique de mot de passe et b) l'outil DomainPasswordSpray supprimera les comptes proches du verrouillage de la liste cible. Étant donné que j'étais authentifié dans le domaine, je pouvais désormais pulvériser avec tous les utilisateurs du domaine, ce qui me donnait beaucoup plus de cibles. J'ai réessayé avec le mot de passe commun Fall2021 et j'ai obtenu plusieurs résultats, tous pour des utilisateurs ne figurant pas dans ma liste de mots initiale. J'ai vérifié les droits de chacun de ces comptes et j'ai découvert que l'un d'entre eux appartenait au groupe d'assistance, qui avait [GenericAll](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericall) droits sur le groupe [Enterprise Key Admins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#enterprise-key-admins)  . Le groupe Enterprise Key Admins avait des privilèges GenericAll sur un contrôleur de domaine, j'ai donc ajouté le compte que je contrôlais à ce groupe, je me suis à nouveau authentifié et j'ai hérité de ces privilèges. En utilisant ces droits, j'ai effectué les [Shadow Credentials](https://posts.specterops.io/shadow-credentials-abusing-key-trust-account-mapping-for-takeover-8ee1a53566ab) et a récupéré le hachage NT pour le compte de machine du contrôleur de domaine. Avec ce hachage NT, j'ai ensuite pu effectuer une attaque DCSync et récupérer les hachages de mots de passe NTLM pour tous les utilisateurs du domaine car un contrôleur de domaine peut effectuer la réplication, ce qui est requis pour DCSync.

* * * * *

Ceci est le chemin
---------------

Ces scénarios peuvent sembler accablants avec de nombreux concepts étrangers en ce moment, mais après avoir terminé ce module, vous serez familiarisé avec la plupart d'entre eux (certains concepts décrits dans ces scénarios sortent du cadre de ce module). Celles-ci montrent l'importance de l'énumération itérative, de la compréhension de notre cible, de l'adaptation et de la réflexion hors des sentiers battus lorsque nous progressons dans un environnement. Nous effectuerons de nombreuses parties des chaînes d'attaque décrites ci-dessus dans ces sections de module, puis vous pourrez mettre vos compétences à l'épreuve en attaquant deux environnements AD différents à la fin de ce module et en découvrant vos propres chaînes d'attaque. Accrochez-vous, car ce sera une balade amusante, mais cahoteuse, à travers le monde sauvage qui "énumère" et "attaque" Active Directory.

* * * * *

Exemples pratiques
------------------

Tout au long du module, nous couvrirons des exemples accompagnés de la sortie de commande. La plupart d'entre eux peuvent être reproduits sur les machines virtuelles cibles qui peuvent être générées dans les sections pertinentes. Vous recevrez des informations d'identification RDP pour interagir avec certaines des machines virtuelles cibles afin d'apprendre à énumérer et à attaquer à partir d'un hôte Windows (`MS01`) et un accès SSH à un hôte Parrot Linux préconfiguré (`ATTACK01`) pour effectuer des exemples d'énumération et d'attaque. depuis Linux. Vous pouvez vous connecter depuis la Pwnbox ou votre propre machine virtuelle (après avoir téléchargé une clé VPN une fois qu'une machine apparaît) via RDP en utilisant [FreeRDP](https://github.com/FreeRDP/FreeRDP/wiki/CommandLineInterface), [Remmina](https ://remmina.org/), ou le client RDP de votre choix le cas échéant ou le client SSH intégré à la Pwnbox ou votre propre VM.

* * * * *

#### Connexion via FreeRDP

Nous pouvons nous connecter via la ligne de commande en utilisant la commande :

Connexion via FreeRDP

```
dsgsec@htb[/htb]$ xfreerdp /v:<IP cible MS01> /u:htb-student /p:Academy_student_AD !

```

#### Connexion via SSH

Nous pouvons nous connecter à l'hôte d'attaque Parrot Linux fourni à l'aide de la commande, puis entrer le mot de passe fourni lorsque vous y êtes invité.

Connexion via SSH

```
dsgsec@htb[/htb]$ ssh htb-student@<IP cible ATTAQUE01>

```

#### Xfreerdp à l'hôte Parrot ATTACK01

Nous avons également installé un serveur `XRDP` sur l'hôte `ATTACK01` pour fournir un accès graphique à l'hôte d'attaque Parrot. Cela peut être utilisé pour interagir avec l'outil graphique BloodHound que nous aborderons plus loin dans cette section. Dans les sections où cet hôte apparaît (où vous disposez d'un accès SSH), vous pouvez également vous y connecter à l'aide de `xfreerdp` à l'aide de la même commande que vous le feriez avec l'hôte d'attaque Windows ci-dessus :

Xfreerdp à l'hôte Parrot ATTACK01

```
dsgsec@htb[/htb]$ xfreerdp /v:<ATTACK01 cible IP> /u:htb-student /p:HTB_@cademy_stdnt !

```

La plupart des sections fourniront des informations d'identification pour l'utilisateur `htb-student` sur `MS01` ou `ATTACK01`. En fonction du matériel et des défis, certaines sections vous demanderont de vous authentifier auprès d'une cible avec un utilisateur différent, et d'autres informations d'identification seront fournies.

Tout au long de ce module, plusieurs mini laboratoires Active Directory vous seront présentés. Certains de ces laboratoires peuvent prendre 3 à 5 minutes pour apparaître complètement et être accessibles via RDP. Nous vous recommandons de faire défiler jusqu'à la fin de chaque section, de cliquer pour faire apparaître le laboratoire, puis de commencer à lire le matériel, de sorte que l'environnement soit opérationnel au moment où vous atteignez les parties interactives de la section.

* * * * *

Boîte à outils
-------

Nous fournissons un hôte d'attaque Windows et Parrot Linux dans le laboratoire d'accompagnement de ce module. Tous les outils nécessaires pour réaliser tous les exemples et résoudre toutes les questions dans les sections du module sont présents sur les hôtes. Les outils nécessaires à l'hôte d'attaque Windows, `MS01` sont situés dans le répertoire `C:\Tools` . D'autres, tels que le module Active Directory PowerShell, se chargeront lors de l'ouverture d'une fenêtre de console PowerShell. Les outils sur l'hôte d'attaque Linux, `ATTACK01`, sont soit installés et ajoutés au PATH des utilisateurs `htb-student`, soit présents dans le répertoire `/opt` . Vous pouvez, bien sûr, (et c'est encouragé) compiler (si nécessaire) et télécharger vos propres outils et scripts sur les hôtes d'attaque pour prendre l'habitude de le faire ou les héberger sur un partage SMB à partir de la Pwnbox travaillant avec les outils de cette façon. Gardez à l'esprit que lors de l'exécution d'un test d'intrusion réel dans le réseau d'un client, il est toujours préférable de compiler les outils vous-même pour examiner le code au préalable et vous assurer qu'il n'y a rien de malveillant caché dans l'exécutable compilé. Nous ne voulons pas introduire des outils infectés dans le réseau d'un client et les exposer à une attaque extérieure.

* * * * *

Amusez-vous et n'oubliez pas de sortir des sentiers battus ! AD est immense. Vous ne le maîtriserez pas du jour au lendemain, mais continuez à y travailler, et bientôt le contenu de ce module sera
