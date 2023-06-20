Empoisonnement LLMNR/NBT-NS - à partir de Linux
===============================================

* * * * *

À ce stade, nous avons terminé notre énumération initiale du domaine. Nous avons obtenu des informations de base sur les utilisateurs et les groupes, énuméré les hôtes tout en recherchant des services et des rôles critiques comme un contrôleur de domaine, et déterminé certaines spécificités telles que le schéma de nommage utilisé pour le domaine. Dans cette phase, nous travaillerons côte à côte sur deux techniques différentes : l'empoisonnement du réseau et la pulvérisation de mots de passe. Nous effectuerons ces actions dans le but d'acquérir des informations d'identification valides en texte clair pour un compte d'utilisateur de domaine, nous accordant ainsi un pied dans le domaine pour commencer la prochaine phase d'énumération d'un point de vue accrédité.

Cette section et la suivante couvriront un moyen courant de recueillir des informations d'identification et de prendre un pied initial lors d'une évaluation : une attaque Man-in-the-Middle sur la résolution de noms de multidiffusion de liaison locale (LLMNR) et le service de noms NetBIOS (NBT-NS) émissions. Selon le réseau, cette attaque peut fournir des hachages de mot de passe à faible privilège ou de niveau administratif qui peuvent être piratés hors ligne ou même des informations d'identification en texte clair. Bien qu'ils ne soient pas couverts dans ce module, ces hachages peuvent également parfois être utilisés pour effectuer une attaque SMB Relay pour s'authentifier auprès d'un hôte ou de plusieurs hôtes du domaine avec des privilèges administratifs sans avoir à casser le hachage du mot de passe hors ligne. Plongeons-nous !

* * * * *

Apprêt LLMNR & NBT-NS
---------------------

[La résolution de noms de multidiffusion Link-Local](https://datatracker.ietf.org/doc/html/rfc4795) (LLMNR) et [le service de noms NetBIOS](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063(v=technet.10)?redirectedfrom=MSDN) (NBT-NS) sont des composants Microsoft Windows qui servent de méthodes alternatives d'identification de l'hôte pouvant être utilisées en cas d'échec du DNS. Si une machine tente de résoudre un hôte mais que la résolution DNS échoue, généralement, la machine essaiera de demander à toutes les autres machines du réseau local l'adresse d'hôte correcte via LLMNR. LLMNR est basé sur le format DNS (Domain Name System) et permet aux hôtes sur le même lien local d'effectuer la résolution de noms pour d'autres hôtes. Il utilise `5355`nativement le port sur UDP. Si LLMNR échoue, le NBT-NS sera utilisé. NBT-NS identifie les systèmes sur un réseau local par leur nom NetBIOS. NBT-NS utilise un port `137`sur UDP.

Le plus important ici est que lorsque LLMNR/NBT-NS sont utilisés pour la résolution de noms, N'IMPORTE QUEL hôte sur le réseau peut répondre. C'est là que nous intervenons avec`Responder`empoisonner ces demandes. Avec l'accès au réseau, nous pouvons usurper une source de résolution de nom faisant autorité (dans ce cas, un hôte censé appartenir au segment de réseau) dans le domaine de diffusion en répondant au trafic LLMNR et NBT-NS comme s'ils avaient une réponse pour le demandeur. héberger. Cet effort d'empoisonnement est fait pour amener les victimes à communiquer avec notre système en prétendant que notre système malveillant connaît l'emplacement de l'hôte demandé. Si l'hôte demandé nécessite une résolution de nom ou des actions d'authentification, nous pouvons capturer le hachage NetNTLM et le soumettre à une attaque par force brute hors ligne pour tenter de récupérer le mot de passe en clair. La demande d'authentification capturée peut également être relayée pour accéder à un autre hôte ou utilisée avec un protocole différent (tel que LDAP) sur le même hôte. L'usurpation d'identité LLMNR/NBNS combinée à un manque de signature SMB peut souvent conduire à un accès administratif sur les hôtes d'un domaine. Les attaques de relais SMB seront couvertes dans un module ultérieur sur le mouvement latéral.

* * * * *

Exemple rapide - Empoisonnement LLMNR/NBT-NS
--------------------------------------------

Passons en revue un exemple rapide du flux d'attaque à un niveau très élevé :

1.  Un hôte tente de se connecter au serveur d'impression sur \\print01.inlanefreight.local, mais saisit accidentellement \\printer01.inlanefreight.local.
2.  Le serveur DNS répond en indiquant que cet hôte est inconnu.
3.  L'hôte diffuse ensuite sur l'ensemble du réseau local en demandant si quelqu'un connaît l'emplacement de \\printer01.inlanefreight.local.
4.  L'attaquant (nous avec `Responder`running) répond à l'hôte en indiquant que c'est le \\printer01.inlanefreight.local que l'hôte recherche.
5.  L'hôte croit cette réponse et envoie une demande d'authentification à l'attaquant avec un nom d'utilisateur et un hachage de mot de passe NTLMv2.
6.  Ce hachage peut ensuite être craqué hors ligne ou utilisé dans une attaque SMB Relay si les bonnes conditions existent.

* * * * *

TTP
---

Nous effectuons ces actions pour collecter les informations d'authentification envoyées sur le réseau sous la forme de hachages de mots de passe NTLMv1 et NTLMv2. Comme indiqué dans le module [Introduction à Active Directory](https://academy.hackthebox.com/course/preview/introduction-to-active-directory) , NTLMv1 et NTLMv2 sont des protocoles d'authentification qui utilisent le hachage LM ou NT. Nous prendrons ensuite le hachage et tenterons de le déchiffrer hors ligne à l'aide d'outils tels que [Hashcat](https://hashcat.net/hashcat/) ou [John](https://www.openwall.com/john/) dans le but d'obtenir le mot de passe en texte clair du compte à utiliser pour prendre pied ou étendre notre accès au domaine si nous capturons un hachage de mot de passe pour un compte avec plus de privilèges qu'un compte que nous possédons actuellement.

Plusieurs outils peuvent être utilisés pour tenter une intoxication LLMNR & NBT-NS :

| Outil | Description |
| --- | --- |
| [Répondant](https://github.com/lgandx/Responder) | Responder est un outil spécialement conçu pour empoisonner LLMNR, NBT-NS et MDNS, avec de nombreuses fonctions différentes. |
| [Invectiver](https://github.com/Kevin-Robertson/Inveigh) | Inveigh est une plate-forme MITM multiplateforme qui peut être utilisée pour des attaques d'usurpation et d'empoisonnement. |
| [Metasploit](https://www.metasploit.com/) | Metasploit dispose de plusieurs scanners intégrés et de modules d'usurpation d'identité conçus pour faire face aux attaques d'empoisonnement. |

Cette section et la suivante montreront des exemples d'utilisation de Responder et Inveigh pour capturer des hachages de mots de passe et tenter de les déchiffrer hors ligne. Nous commençons généralement un test d'intrusion interne à partir d'une position anonyme sur le réseau interne du client avec un hôte d'attaque Linux. Des outils tels que Responder sont parfaits pour établir une position sur laquelle nous pourrons ensuite nous développer grâce à une énumération et à des attaques supplémentaires. Responder est écrit en Python et généralement utilisé sur un hôte d'attaque Linux, bien qu'il existe une version .exe qui fonctionne sous Windows. Inveigh est écrit à la fois en C # et PowerShell (considéré comme un héritage). Les deux outils peuvent être utilisés pour attaquer les protocoles suivants :

-   LLMNR
-   DNS
-   MDNS
-   NBNS
-   DHCP
-   ICMP
-   HTTP
-   HTTPS
-   PME
-   LDAP
-   WebDAV
-   Authentification proxy

Responder prend également en charge :

-   MSSQL
-   DCE-RPC
-   Authentification FTP, POP3, IMAP et SMTP

* * * * *

### Intervenant en action

Responder est un outil relativement simple, mais il est extrêmement puissant et possède de nombreuses fonctions différentes. Dans la `Initial Enumeration`section précédente, nous avons utilisé le répondeur en mode analyse (passif). Cela signifie qu'il a écouté toutes les demandes de résolution, mais n'y a pas répondu et n'a pas envoyé de paquets empoisonnés. Nous agissions comme une mouche sur le mur, écoutant juste. Maintenant, nous allons aller plus loin et laisser Responder faire ce qu'il fait le mieux. Examinons quelques options disponibles en tapant `responder -h`dans notre console.

```
dsgsec@htb[/htb]$ responder -h
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

Usage: responder -I eth0 -w -r -f
or:
responder -I eth0 -wrf

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -A, --analyze         Analyze mode. This option allows you to see NBT-NS,
                        BROWSER, LLMNR requests without responding.
  -I eth0, --interface=eth0
                        Network interface to use, you can use 'ALL' as a
                        wildcard for all interfaces
  -i 10.0.0.21, --ip=10.0.0.21
                        Local IP to use (only for OSX)
  -e 10.0.0.22, --externalip=10.0.0.22
                        Poison all requests with another IP address than
                        Responder's one.
  -b, --basic           Return a Basic HTTP authentication. Default: NTLM
  -r, --wredir          Enable answers for netbios wredir suffix queries.
                        Answering to wredir will likely break stuff on the
                        network. Default: False
  -d, --NBTNSdomain     Enable answers for netbios domain suffix queries.
                        Answering to domain suffixes will likely break stuff
                        on the network. Default: False
  -f, --fingerprint     This option allows you to fingerprint a host that
                        issued an NBT-NS or LLMNR query.
  -w, --wpad            Start the WPAD rogue proxy server. Default value is
                        False
  -u UPSTREAM_PROXY, --upstream-proxy=UPSTREAM_PROXY
                        Upstream HTTP proxy used by the rogue WPAD Proxy for
                        outgoing requests (format: host:port)
  -F, --ForceWpadAuth   Force NTLM/Basic authentication on wpad.dat file
                        retrieval. This may cause a login prompt. Default:
                        False
  -P, --ProxyAuth       Force NTLM (transparently)/Basic (prompt)
                        authentication for the proxy. WPAD doesn't need to be
                        ON. This option is highly effective when combined with
                        -r. Default: False
  --lm                  Force LM hashing downgrade for Windows XP/2003 and
                        earlier. Default: False
  -v, --verbose         Increase verbosity.

```

Comme indiqué précédemment dans le module, l' `-A`indicateur nous met en mode d'analyse, ce qui nous permet de voir les requêtes NBT-NS, BROWSER et LLMNR dans l'environnement sans empoisonner les réponses. Il faut toujours fournir soit une interface soit une IP. Certaines options courantes que nous voudrons généralement utiliser sont`-wf` ; cela démarrera le serveur proxy escroc WPAD, tout en `-f`essayant d'identifier le système d'exploitation et la version de l'hôte distant. Nous pouvons utiliser l' `-v`indicateur pour augmenter la verbosité si nous rencontrons des problèmes, mais cela entraînera l'impression de nombreuses données supplémentaires sur la console. D'autres options telles que `-F`et `-P`peuvent être utilisées pour forcer l'authentification NTLM ou de base et forcer l'authentification proxy, mais peuvent provoquer une invite de connexion, elles doivent donc être utilisées avec parcimonie. L'utilisation de la`-w`flag utilise le serveur proxy WPAD intégré. Cela peut être très efficace, en particulier dans les grandes organisations, car il capture toutes les requêtes HTTP de tous les utilisateurs qui lancent Internet Explorer si le navigateur a activé [les paramètres de détection automatique](https://docs.microsoft.com/en-us/internet-explorer/ie11-deploy-guide/auto-detect-settings-for-ie11) .

Avec cette configuration illustrée ci-dessus, Responder écoutera et répondra à toutes les demandes qu'il voit sur le fil. Si vous réussissez et parvenez à capturer un hachage, Responder l'imprimera à l'écran et l'écrira dans un fichier journal par hôte situé dans le `/usr/share/responder/logs`répertoire. Les hachages sont enregistrés au format `(MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt`, et un hachage est imprimé sur la console et stocké dans son fichier journal associé, sauf si `-v`le mode est activé. Par exemple, un fichier journal peut ressembler à `SMB-NTLMv2-SSP-172.16.5.25`. Les hachages sont également stockés dans une base de données SQLite qui peut être configurée dans le `Responder.conf`fichier de configuration, généralement situé dans `/usr/share/responder`sauf si nous clonons le référentiel Responder directement depuis GitHub.

Nous devons exécuter l'outil avec les privilèges sudo ou en tant que root et nous assurer que les ports suivants sont disponibles sur notre hôte d'attaque pour qu'il fonctionne au mieux :

```

UDP 137, UDP 138, UDP 53, UDP/TCP 389,TCP 1433, UDP 1434, TCP 80, TCP 135, TCP 139, TCP 445, TCP 21, TCP 3141,TCP 25, TCP 110, TCP 587, TCP 3128, Multicast UDP 5355 and 5353

```

N'importe lequel des serveurs malveillants (c'est-à-dire SMB) peut être désactivé dans le `Responder.conf`fichier.

#### Journaux des intervenants

  Journaux des intervenants

```
dsgsec@htb[/htb]$ ls

Analyzer-Session.log                Responder-Session.log
Config-Responder.log                SMB-NTLMv2-SSP-172.16.5.200.txt
HTTP-NTLMv2-172.16.5.200.txt        SMB-NTLMv2-SSP-172.16.5.25.txt
Poisoners-Session.log               SMB-NTLMv2-SSP-172.16.5.50.txt
Proxy-Auth-NTLMv2-172.16.5.200.txt

```

Si Responder a réussi à capturer les hachages, comme indiqué ci-dessus, nous pouvons trouver les hachages associés à chaque hôte/protocole dans leur propre fichier texte. L'animation ci-dessous nous montre un exemple de répondeur exécutant et capturant des hachages sur le réseau.

Nous pouvons lancer une session Responder assez rapidement :

#### Démarrage du répondeur avec les paramètres par défaut

Code : bash

```
sudo responder -I ens224

```

#### Capturer avec Responder

![image](https://academy.hackthebox.com/storage/modules/143/responder_hashes.png)

En règle générale, nous devrions démarrer Responder et le laisser s'exécuter pendant un certain temps dans une fenêtre tmux pendant que nous effectuons d'autres tâches d'énumération pour maximiser le nombre de hachages que nous pouvons obtenir. Une fois que nous sommes prêts, nous pouvons transmettre ces hachages à Hashcat en utilisant le mode de hachage `5600`pour les hachages NTLMv2 que nous obtenons généralement avec Responder. Nous pouvons parfois obtenir des hachages NTLMv1 et d'autres types de hachages et pouvons consulter la page [d'exemples de hachages Hashcat pour les identifier et trouver le mode de hachage approprié. ](https://hashcat.net/wiki/doku.php?id=example_hashes)Si jamais nous obtenons un hachage étrange ou inconnu, ce site est une excellente référence pour aider à l'identifier. Consultez le module [Cracking Passwords With Hashcat](https://academy.hackthebox.com/course/preview/cracking-passwords-with-hashcat) pour une étude approfondie des différents modes de Hashcat et comment attaquer une grande variété de types de hachage.

Une fois que nous en avons assez, nous devons obtenir ces hachages dans un format utilisable pour nous dès maintenant. Les hachages NetNTLMv2 sont très utiles une fois crackés, mais ne peuvent pas être utilisés pour des techniques telles que pash-the-hash, ce qui signifie que nous devons essayer de les cracker hors ligne. Nous pouvons le faire avec des outils tels que Hashcat et John.

#### Cracker un hachage NTLMv2 avec Hashcat

  Cracker un hachage NTLMv2 avec Hashcat

```
dsgsec@htb[/htb]$ hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

FOREND::INLANEFREIGHT:4af70a79938ddf8a:0f85ad1e80baa52d732719dbf62c34cc:010100000000000080f519d1432cd80136f3af14556f047800000000020008004900340046004e0001001e00570049004e002d0032004e004c005100420057004d00310054005000490004003400570049004e002d0032004e004c005100420057004d0031005400500049002e004900340046004e002e004c004f00430041004c00030014004900340046004e002e004c004f00430041004c00050014004900340046004e002e004c004f00430041004c000700080080f519d1432cd80106000400020000000800300030000000000000000000000000300000227f23c33f457eb40768939489f1d4f76e0e07a337ccfdd45a57d9b612691a800a001000000000000000000000000000000000000900220063006900660073002f003100370032002e00310036002e0035002e003200320035000000000000000000:Klmcargo2

Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: FOREND::INLANEFREIGHT:4af70a79938ddf8a:0f85ad1e80ba...000000
Time.Started.....: Mon Feb 28 15:20:30 2022 (11 secs)
Time.Estimated...: Mon Feb 28 15:20:41 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1086.9 kH/s (2.64ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 10967040/14344385 (76.46%)
Rejected.........: 0/10967040 (0.00%)
Restore.Point....: 10960896/14344385 (76.41%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: L0VEABLE -> Kittikat

Started: Mon Feb 28 15:20:29 2022
Stopped: Mon Feb 28 15:20:42 2022

```

En regardant les résultats ci-dessus, nous pouvons voir que nous avons craqué le hachage NET-NTLMv2 pour l'utilisateur `FOREND`dont le mot de passe est `Klmcargo2`. Heureusement pour nous, notre domaine cible autorise les mots de passe faibles à 8 caractères. Ce type de hachage peut être "lent" à craquer même sur une plate-forme de craquement GPU, de sorte que les mots de passe volumineux et complexes peuvent être plus difficiles ou impossibles à craquer dans un délai raisonnable.

* * * * *

Passer à autre chose
--------------------

À ce stade de notre évaluation, nous avons obtenu et craqué un hachage NetNTLMv2 pour l'utilisateur `FOREND`. Nous pouvons l'utiliser comme un pied dans le domaine pour commencer une énumération plus poussée. Il est préférable de collecter autant de données que possible lors d'une évaluation, nous devrions donc essayer de déchiffrer autant de hachages que possible (à condition que notre énumération ultérieure montre l'intérêt de les déchiffrer pour approfondir notre accès). Nous ne voulons pas perdre un temps d'évaluation précieux à tenter de casser des hachages pour les utilisateurs qui ne nous aideront pas à avancer vers notre objectif. Avant de passer à d'autres moyens d'obtenir un pied via la pulvérisation de mot de passe, passons en revue une méthode similaire pour obtenir des hachages à partir d'un hôte Windows à l'aide de l'outil Inveigh.
