Lecteurs de journaux d'événements
=================

* * * * *

Supposons que [l'audit de la création de processus](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation) les événements et les valeurs de ligne de commande correspondantes soient activés. Dans ce cas, ces informations sont enregistrées dans le journal des événements de sécurité Windows en tant qu'ID d'événement [4688 : un nouveau processus a été créé](https://docs.microsoft.com/en-us/windows/security/threat-protection/audit/événement-4688). Les organisations peuvent activer la journalisation des lignes de commande de processus pour aider les défenseurs à surveiller et à identifier les comportements potentiellement malveillants et à identifier les fichiers binaires qui ne devraient pas être présents sur un système. Ces données peuvent être envoyées à un outil SIEM ou ingérées dans un outil de recherche, tel qu'ElasticSearch, pour donner aux défenseurs une visibilité sur les fichiers binaires exécutés sur les systèmes du réseau. Les outils signaleraient ensuite toute activité potentiellement malveillante, telle que les commandes `whoami`, `netstat` et `tasklist` exécutées à partir du poste de travail d'un responsable marketing.

Cette [étude](https://blogs.jpcert.or.jp/en/2016/01/windows-commands-abused-by-attackers.html) montre certaines des commandes les plus exécutées par les attaquants après l'accès initial (`tasklist `, `ver`, `ipconfig`, `systeminfo`, etc.), pour la reconnaissance (`dir`, `net view`, `ping`, `net use`, `type`, etc.) et pour la diffusion malware au sein d'un réseau (`at`, `reg`, `wmic`, `wusa`, etc.). Outre la surveillance de l'exécution de ces commandes, une organisation peut aller plus loin et restreindre l'exécution de commandes spécifiques à l'aide de règles AppLocker affinées. Pour une organisation disposant d'un budget de sécurité serré, l'utilisation de ces outils intégrés de Microsoft peut offrir une excellente visibilité sur les activités du réseau au niveau de l'hôte. La plupart des outils EDR d'entreprise modernes effectuent la détection/le blocage, mais peuvent être hors de portée de nombreuses organisations en raison de contraintes budgétaires et de personnel. Ce petit exemple montre que les améliorations de la sécurité, telles que la visibilité au niveau du réseau et de l'hôte, peuvent être réalisées avec un minimum d'effort, de coût et un impact massif.

J'ai effectué un test d'intrusion contre une organisation de taille moyenne il y a quelques années avec une petite équipe de sécurité, pas d'EDR d'entreprise, mais j'utilisais une configuration similaire à ce qui a été détaillé ci-dessus (création de processus d'audit et valeurs de ligne de commande). Ils ont attrapé et contenu l'un des membres de mon équipe lorsqu'il a exécuté la commande `tasklist` à partir du poste de travail d'un membre du service financier (après avoir capturé les informations d'identification à l'aide de `Responder` et les avoir piratées hors ligne).

Administrateurs ou membres des [Event Log Readers](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255(v =ws.11)?redirectedfrom=MSDN#event-log-readers) groupe sont autorisés à accéder à ce journal. Il est concevable que les administrateurs système veuillent ajouter des utilisateurs avancés ou des développeurs dans ce groupe pour effectuer certaines tâches sans avoir à leur accorder un accès administratif.

#### Confirmation de l'appartenance au groupe

Confirmation de l'appartenance au groupe

```
C:\htb> groupe local net "Lecteurs de journaux d'événements"

Nom d'alias Lecteurs du journal des événements
Commentaire Les membres de ce groupe peuvent lire les journaux d'événements à partir de la machine locale

Membres

-------------------------------------------------- -----------------------------
enregistreur
La commande s'est terminée avec succès.

```

Microsoft a publié une référence [guide](https://download.microsoft.com/download/5/8/9/58911986-D4AD-4695-BF63-F734CD4DF8F2/ws-commands.pdf) pour toutes les commandes Windows intégrées , y compris la syntaxe, les paramètres et des exemples. De nombreuses commandes Windows prennent en charge la transmission d'un mot de passe en tant que paramètre, et si l'audit des lignes de commande de processus est activé, ces informations sensibles seront capturées.

Nous pouvons interroger les événements Windows à partir de la ligne de commande à l'aide de l'utilitaire [wevtutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) et de [Get-WinEvent]( https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1) Applet de commande PowerShell.

#### Recherche des journaux de sécurité à l'aide de wevtutil

Recherche des journaux de sécurité à l'aide de wevtutil

```
PS C:\htb> wevtutil qe Sécurité /rd:true /f:text | Select-String "/user"

         Ligne de commande de processus : net use T : \\fs01\backups /user:tim MyStr0ngP@ssword

```

Nous pouvons également spécifier d'autres informations d'identification pour `wevtutil` à l'aide des paramètres `/u` et `/p`.

#### Transmission des informations d'identification à wevtutil

Transmission des informations d'identification à wevtutil

```
C:\htb> wevtutil qe Sécurité /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"

```

Pour `Get-WinEvent`, la syntaxe est la suivante. Dans cet exemple, nous filtrons les événements de création de processus (4688), qui contiennent `/user` dans la ligne de commande du processus.

Remarque : La recherche dans le journal des événements `Security` avec `Get-WInEvent` nécessite un accès administrateur ou des autorisations ajustées sur la clé de registre `HKLM\System\CurrentControlSet\Services\Eventlog\Security`. L'appartenance au seul groupe `Event Log Readers` n'est pas suffisante.

#### Recherche des journaux de sécurité Using Get-WinEvent

Recherche de journaux de sécurité à l'aide de Get-WinEvent

```
PS C:\htb> Get-WinEvent -LogName security | où { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}

Ligne de commande
-----------
net use T: \\fs01\backups /user:tim MyStr0ngP@ssword

```

L'applet de commande peut également être exécutée en tant qu'autre utilisateur avec le paramètre `-Credential` .

Les autres journaux incluent le journal [PowerShell Operational](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.1) , qui peut également contenir des informations sensibles ou informations d'identification si la journalisation du bloc de script ou du module est activée. Ce journal est accessible aux utilisateurs non privilégiés.