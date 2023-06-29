Hôte de script Windows (WSH)
=========================

L'hôte de script Windows est un outil d'administration Windows intégré qui exécute des fichiers batch pour automatiser et gérer les tâches au sein du système d'exploitation.

Il s'agit d'un moteur natif Windows, cscript.exe  (pour les scripts de ligne de commande) et wscript.exe  (pour les scripts d'interface utilisateur), qui sont responsables de l'exécution de divers scripts Microsoft Visual Basic (VBScript), y compris vbs  et vbe . Pour plus d'informations sur VBScript, veuillez visiter [ici](https://en.wikipedia.org/wiki/VBScript) . Il est important de noter que le moteur VBScript sur un système d'exploitation Windows exécute et exécute des applications avec le même niveau d'accès et d'autorisation qu'un utilisateur normal ; par conséquent, il est utile pour les équipes rouges.

Écrivons maintenant un  code VBScript  simple pour créer une boîte de message Windows qui affiche le  message Welcome to THM  . Assurez-vous d'enregistrer le code suivant dans un fichier, par exemple,  hello.vbs .

```
Dim message
message = "Welcome to THM"
MsgBox message
```

Dans la première ligne, nous avons déclaré la   variable  message en utilisant Dim . Ensuite, nous stockons une valeur de chaîne  Welcome to THM  dans la  variable message . Dans la ligne suivante, nous utilisons la fonction MsgBox pour afficher le contenu de la variable. Pour plus d'informations sur la fonction MsgBox, veuillez visiter [ici](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/scripting-articles/sfw6660x(v=vs.84)?redirectedfrom=MSDN) . Ensuite, nous utilisons wscript  pour lancer et exécuter le contenu de  hello.vbs .  En conséquence, un message Windows apparaîtra avec  le  message Bienvenue dans THM  . [](https://docs.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/scripting-articles/sfw6660x(v=vs.84)?redirectedfrom=MSDN)

![f40a7711a408932981d827bfe6e522f3](https://github.com/dsgsec/Red-Team/assets/82456829/b57a1218-54e5-4b71-8983-ce33f14d97fa)

Utilisons maintenant le VBScript pour exécuter des fichiers exécutables. Le code vbs suivant consiste à invoquer la calculatrice Windows, preuve que nous pouvons exécuter des fichiers .exe en utilisant le moteur natif de Windows (WSH).

```
Set shell = WScript.CreateObject("Wscript.Shell")
shell.Run("C:\Windows\System32\calc.exe " & WScript.ScriptFullName),0,True
```

Nous créons un objet de la bibliothèque  WScript en utilisant CreateObject pour appeler la charge utile d'exécution. Ensuite, nous utilisons la  méthode Run  pour exécuter la charge utile. Pour cette tâche, nous allons  exécuter la  calculatrice Windows calc.exe .   

Pour exécuter le fichier exe , nous pouvons l'exécuter en utilisant le wscript comme suit, 

Terminal

```
c:\Windows\System32>wscript c:\Users\thm\Desktop\payload.vbs
```

Nous pouvons également l'exécuter via cscript comme suit,

Terminal

```
c:\Windows\System32>cscript.exe c:\Users\thm\Desktop\payload.vbs
```

En conséquence, la calculatrice Windows apparaîtra sur le bureau.

![8c7cbe29ee437b83a244994621cf6996](https://github.com/dsgsec/Red-Team/assets/82456829/a83ab01d-a2e6-4e06-9205-21ba9f9fbbe7)

Une autre astuce. Si les fichiers VBS sont sur la liste noire, nous pouvons renommer le fichier en fichier .txt et l'exécuter en utilisant wscript comme suit,

Terminal

```
c:\Windows\System32>wscript /e:VBScript c:\Users\thm\Desktop\payload.txt
```

Le résultat sera aussi exact que l'exécution des fichiers vbs , qui exécutent le binaire calc.exe .

![f6d6a5f824fa64750e8b15ce6ba07a7a](https://github.com/dsgsec/Red-Team/assets/82456829/53de3e51-a126-4733-9c74-b97836f6aaf3)
