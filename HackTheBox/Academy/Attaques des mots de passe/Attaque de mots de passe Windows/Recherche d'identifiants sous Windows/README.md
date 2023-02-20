# Recherche d'informations d'identification dans Windows
Une fois que nous avons accès à une machine Windows cible via l'interface graphique ou la CLI, nous pouvons bénéficier de manière significative de l'intégration de la recherche d'informations d'identification dans notre approche. Credential Hunting est le processus consistant à effectuer des recherches détaillées dans le système de fichiers et dans diverses applications pour découvrir les informations d'identification. Pour comprendre ce concept, plaçons-nous dans un scénario. Nous avons eu accès au poste de travail Windows 10 d'un administrateur informatique via RDP.

## Centré sur la recherche
De nombreux outils à notre disposition dans Windows ont une fonctionnalité de recherche. De nos jours, il existe des fonctionnalités centrées sur la recherche intégrées dans la plupart des applications et des systèmes d'exploitation, nous pouvons donc les utiliser à notre avantage lors d'un engagement. Un utilisateur peut avoir documenté ses mots de passe quelque part sur le système. Il peut même y avoir des informations d'identification par défaut qui pourraient être trouvées dans divers fichiers. Il serait sage de baser notre recherche d'informations d'identification sur ce que nous savons de la façon dont le système cible est utilisé. Dans ce cas, nous savons que nous avons accès au poste de travail d'un administrateur informatique.

Que peut faire un administrateur informatique au quotidien et lesquelles de ces tâches peuvent nécessiter des informations d'identification ?

Nous pouvons utiliser cette question et considération pour affiner notre recherche afin de réduire autant que possible le besoin de deviner au hasard.

## Termes clés à rechercher
Que nous nous retrouvions avec un accès à l'interface graphique ou à la CLI, nous savons que nous aurons des outils à utiliser pour la recherche, mais ce que nous recherchons exactement est tout aussi important. Voici quelques termes clés utiles que nous pouvons utiliser et qui peuvent nous aider à découvrir certaines informations d'identification :

|  |  |  |
| --- | --- | --- |
| Passwords | Passphrases | Keys |
| Username | User account | Creds |
| Users | Passkeys | Passphrases |
| configuration | dbcredential | dbpassword |
| pwd | Login | Credentials |

## Outils de recherche
Avec l'accès à l'interface graphique, il vaut la peine d'essayer d'utiliser Windows Search pour trouver des fichiers sur la cible en utilisant certains des mots-clés mentionnés ci-dessus.

Par défaut, il recherchera dans divers paramètres du système d'exploitation et dans le système de fichiers les fichiers et les applications contenant le terme clé saisi dans la barre de recherche.

Nous pouvons également tirer parti d'outils tiers tels que Lazagne pour découvrir rapidement les informations d'identification que les navigateurs Web ou d'autres applications installées peuvent stocker de manière non sécurisée. Il serait avantageux de conserver une copie autonome de Lazagne sur notre hôte d'attaque afin que nous puissions rapidement la transférer vers la cible. Lazagne.exe fera très bien l'affaire pour nous dans ce scénario. Nous pouvons utiliser notre client RDP pour copier le fichier sur la cible à partir de notre hôte d'attaque. Si nous utilisons xfreerdp, tout ce que nous devons faire est de copier et coller dans la session RDP que nous avons établie.

Une fois que Lazagne.exe est sur la cible, nous pouvons ouvrir l'invite de commande ou PowerShell, accéder au répertoire dans lequel le fichier a été téléchargé et exécuter la commande suivante :

### Running Lazagne All
```
C:\Users\bob\Desktop> start lazagne.exe all
```

Cela exécutera Lazagne et exécutera tous les modules inclus. Nous pouvons inclure l'option -vv pour étudier ce qu'il fait en arrière-plan. Une fois que nous avons appuyé sur Entrée, il ouvrira une autre invite et affichera les résultats.

```
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: bob ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: 10.129.202.51
Login: admin
Password: SteveisReallyCool123
Port: 22
```

Si nous utilisions l'option -vv, nous verrions des tentatives de collecte de mots de passe à partir de tous les logiciels pris en charge par Lazagne. Nous pouvons également consulter la page GitHub sous la section des logiciels pris en charge pour voir tous les logiciels à partir desquels Lazagne essaiera de recueillir des informations d'identification. Il peut être un peu choquant de voir à quel point il est facile d'obtenir des informations d'identification en texte clair. Cela peut être attribué en grande partie à la manière non sécurisée dont de nombreuses applications stockent les informations d'identification.

### Utilisation de findstr
Nous pouvons également utiliser findstr pour rechercher des modèles dans de nombreux types de fichiers. En gardant à l'esprit les termes clés courants, nous pouvons utiliser des variantes de cette commande pour découvrir les informations d'identification sur une cible Windows :

```
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

## Considérations supplémentaires
Il existe des milliers d'outils et de termes clés que nous pourrions utiliser pour rechercher des informations d'identification sur les systèmes d'exploitation Windows. Sachez que ceux que nous choisirons d'utiliser seront principalement basés sur la fonction de l'ordinateur. Si nous atterrissons sur un système d'exploitation Windows Server, nous pouvons utiliser une approche différente de celle si nous atterrissons sur un système d'exploitation Windows Desktop. Soyez toujours conscient de la façon dont le système est utilisé, et cela nous aidera à savoir où chercher. Parfois, nous pouvons même trouver des informations d'identification en naviguant et en répertoriant les répertoires sur le système de fichiers pendant l'exécution de nos outils.

Voici quelques autres endroits que nous devrions garder à l'esprit lors de la recherche d'informations d'identification :

+ Mots de passe dans la stratégie de groupe dans le partage SYSVOL
+ Mots de passe dans les scripts du partage SYSVOL
+ Mot de passe dans les scripts sur les partages informatiques
+ Mots de passe dans les fichiers web.config sur les machines de développement et les partages informatiques
unattend.xml
+ Mots de passe dans les champs de description de l'utilisateur AD ou de l'ordinateur
+ Bases de données KeePass -> extrayez le hachage, craquez et obtenez de nombreux accès.
+ Trouvé sur les systèmes utilisateur et les partages
+ Fichiers tels que pass.txt, passwords.docx, passwords.xlsx trouvés sur les systèmes des utilisateurs, partages, Sharepoint

Vous avez obtenu l'accès au poste de travail Windows 10 d'un administrateur informatique et commencez votre processus de recherche d'informations d'identification en recherchant des informations d'identification dans des emplacements de stockage communs.