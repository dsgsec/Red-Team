De manière générale, la détection AV peut être classée en trois approches principales :

1.  Détection statique
2.  Détection dynamique

3.  Détection heuristique et comportementale 

Détection statique

Une technique de détection statique est le type de détection antivirus le plus simple, basée sur des signatures prédéfinies de fichiers malveillants. Simplement, il utilise des techniques de correspondance de modèles dans la détection, telles que la recherche d'une chaîne unique, le CRC (sommes de contrôle), la séquence de valeurs bytecode/Hex et les hachages cryptographiques (MD5, SHA1, etc.).

Il effectue ensuite un ensemble de comparaisons entre les fichiers existants dans le système d'exploitation et une base de données de signatures. Si la signature existe dans la base de données, elle est alors considérée comme malveillante. Cette méthode est efficace contre les logiciels malveillants statiques.

![06689aaadc7842fbe0cc1424e89e1b3c](https://github.com/dsgsec/Red-Team/assets/82456829/d4a839af-7b06-4ade-8437-637358625c13)

Dans cette tâche, nous utiliserons une méthode de détection basée sur les signatures pour voir comment les produits antivirus détectent les fichiers malveillants. Il est important de noter que cette technique fonctionne contre les fichiers malveillants connus uniquement avec des signatures pré-générées dans une base de données. La base de données doit donc être mise à jour de temps en temps.

Nous utiliserons le  logiciel antivirus ClamAV  pour démontrer comment la détection basée sur les signatures identifie les fichiers malveillants . Le logiciel ClamAV est préinstallé dans la VM fournie et nous pouvons y accéder par le chemin suivant :  c:\Program Files\ClamAV\clamscan.exe .  Nous analyserons également quelques échantillons de logiciels malveillants, qui peuvent être trouvés sur le bureau. Le dossier Exemples de logiciels malveillants contient les fichiers suivants :

1.  EICAR est un fichier de test contenant des chaînes ASCII utilisées pour tester l'efficacité des logiciels antivirus au lieu de véritables logiciels malveillants susceptibles d'endommager votre machine. Pour plus d'informations, vous pouvez visiter le site officiel de l'EICAR, [ ici ](https://www.eicar.org/?page_id=3950).
2.  Backdoor 1 est un programme C# qui utilise une technique bien connue pour établir une connexion inverse, notamment la création d'un processus et l'exécution d'un shellcode Metasploit Framework.
3.  Backdoor 2 est un programme C# qui utilise l'injection de processus et le cryptage pour établir une connexion inverse, notamment en injectant un shellcode Metasploit dans un processus existant et en cours d'exécution.
4.  AV-Check est un programme C# qui énumère les logiciels AV sur une machine cible. Notez que ce fichier n'est pas malveillant. Nous discuterons de cet outil plus en détail dans la tâche 6. 
5.  notes.txt est un fichier texte qui contient une ligne de commande. Notez que ce fichier n'est pas malveillant.

ClamAV est livré avec sa base de données, et lors de l'installation, nous devons télécharger la version récemment mise à jour. Essayons d'analyser le dossier d'exemples de logiciels malveillants à l'aide du  binaire clamscan.exe  et vérifions les performances de ClamAV par rapport à ces échantillons.

Invite de commande

```
c:\>"c:\Program Files\ClamAV\clamscan.exe" c:\Users\thm\Desktop\Samples
Loading:    22s, ETA:   0s [========================>]    8.61M/8.61M sigs
Compiling:   4s, ETA:   0s [========================>]       41/41 tasks

C:\Users\thm\Desktop\Samples\AV-Check.exe: OK
C:\Users\thm\Desktop\Samples\backdoor1.exe: Win.Malware.Swrort-9872015-0 FOUND
C:\Users\thm\Desktop\Samples\backdoor2.exe: OK
C:\Users\thm\Desktop\Samples\eicar.com: Win.Test.EICAR_HDB-1 FOUND
C:\Users\thm\Desktop\Samples\notes.txt: OK
```

Le résultat ci-dessus montre que le logiciel ClamAV a correctement analysé et signalé deux de nos fichiers testés (EICAR, backdoor1, AV-Check et notes.txt) comme malveillants. Cependant, il a identifié à tort la porte dérobée2 comme non malveillante alors qu'elle le fait.

Vous pouvez exécuter  clamscan.exe --debug <file_to_scan> et vous verrez tous les modules chargés et utilisés pendant l'analyse. Par exemple, il utilise la méthode de décompression pour diviser les fichiers et rechercher une séquence malveillante prédéfinie de valeurs de bytecode, et c'est ainsi qu'il a pu détecter la porte dérobée C# 1. La valeur de bytecode du shellcode Metasploit utilisé dans la porte dérobée 1 était auparavant identifié et ajouté à la base de données de ClamAV.

Cependant, backdoor 2 utilise une technique de cryptage (XOR) pour le shellcode Metasploit, ce qui entraîne différentes séquences de valeurs de bytecode qu'il ne trouve pas dans la base de données ClamAV. 

Alors que ClamAV a pu détecter le fichier de test EICAR.COM comme malveillant en utilisant la technique basée sur la signature md5. Pour confirmer cela, nous pouvons réanalyser le fichier de test EICAR.COM en mode débogage (--debug). À un moment donné dans la sortie, vous verrez le message suivant :

```
LibClamAV debug: FP SIGNATURE: 44d88612fea8a8f36de82e1278abb02f:68:Win.Test.EICAR_HDB-1  # Name: eicar.com, Type: CL_TYPE_TEXT_ASCII
```

﻿Générons maintenant la valeur md5 de EICAR.COM si elle correspond à ce que nous voyons dans le message précédent de la sortie. Nous utiliserons pour cela le sigtool :

Invite de commande

```
c:\>"c:\Program Files\ClamAV\sigtool.exe" --md5 c:\Users\thm\Desktop\Samples\eicar.com
44d88612fea8a8f36de82e1278abb02f:68:eicar.com
```

Si vous vérifiez attentivement la valeur MD5 générée,  44d88612fea8a8f36de82e1278abb02f , elle correspond.

Créez votre propre base de données de signatures 

L'une des fonctionnalités de ClamAV consiste à créer votre propre base de données, vous permettant d'inclure des éléments introuvables dans la base de données officielle de ClamAV. Essayons de créer une signature pour Backdoor 2, que ClamAV a déjà manquée, et de l'ajouter à une base de données. Voici les étapes requises :

1.  Générez une signature MD5 pour le fichier.
2.  Ajoutez la signature générée dans une base de données avec l'extension ".hdb".
3.  Analysez à nouveau le ClamAV par rapport au fichier en utilisant notre nouvelle base de données.

Tout d'abord, nous utiliserons l'  outil sigtool , inclus dans la suite ClamAV, pour générer un hachage MD5 de  backdoor2.exe  en utilisant l'  argument --md5 .

Générer un hachage MD5

```
C:\Users\thm\Desktop\Samples>"c:\Program Files\ClamAV\sigtool.exe" --md5 backdoor2.exe
75047189991b1d119fdb477fef333ceb:6144:backdoor2.exe
```

Comme indiqué dans le résultat, la chaîne de hachage générée contient la structure suivante : Hash:Size-in-byte:FileName . Notez que ClamAV utilise la valeur générée dans la comparaison lors de l'analyse.

Maintenant que nous avons le hachage MD5, créons maintenant notre propre base de données. Nous utiliserons l' outil sigtool et enregistrerons la sortie dans un fichier en utilisant > thm.hdb comme suit :

Générez notre nouvelle base de données

```
C:\Users\thm\Desktop\Samples>"c:\Program Files\ClamAV\sigtool.exe" --md5 backdoor2.exe > thm.hdb
```

En conséquence, un fichier thm.hdb sera créé dans le répertoire courant qui exécute la commande. 

Nous savons déjà que ClamAV n'a pas détecté le backdoor2.exe en utilisant la base de données officielle ! Maintenant, analysons-le à nouveau en utilisant la base de données que nous avons créée, thm.hdb , et voyons le résultat !

Nouvelle analyse de backdoor2.exe à l'aide de la nouvelle base de données !

```
C:\Users\thm\Desktop\Samples>"c:\Program Files\ClamAV\clamscan.exe" -d thm.hdb backdoor2.exe
Loading:     0s, ETA:   0s [========================>]        1/1 sigs
Compiling:   0s, ETA:   0s [========================>]       10/10 tasks

C:\Users\thm\Desktop\Samples\backdoor2.exe: backdoor2.exe.UNOFFICIAL FOUND
```

Comme nous nous y attendions,  l' outil ClamAV a signalé le binaire backdoor2.exe comme malveillant sur la base de la base de données que nous avons fournie. En tant que pratique, ajoutez la signature MD5 d'AV-Check.exe dans la même base de données que nous avons déjà créée, puis vérifiez si ClamAV peut signaler AV-Check.exe comme malveillant.

Règles Yara pour la détection statique

L'un des outils qui aident à la détection statique est [Yara](http://virustotal.github.io/yara/) . Yara est un outil qui permet aux ingénieurs en logiciels malveillants de classer et de détecter les logiciels malveillants. Yara utilise une détection basée sur des règles. Ainsi, pour détecter de nouveaux logiciels malveillants, nous devons créer une nouvelle règle. ClamAV peut également gérer les règles Yara pour détecter les fichiers malveillants. La règle sera la même que dans notre base de données dans la section précédente. 

Pour créer une règle, nous devons examiner et analyser le malware ; sur la base des résultats, nous écrivons une règle. Prenons AV-Check.exe comme exemple et écrivons une règle pour celui-ci. 

Tout d'abord, analysons le fichier et répertorions toutes les chaînes lisibles par l'homme dans le binaire à l'aide de l'outil de chaînes. En conséquence, nous verrons toutes les fonctions, variables et chaînes absurdes. Mais si vous regardez attentivement, nous pouvons utiliser certaines des chaînes uniques de nos règles pour détecter ce fichier à l'avenir. L'AV-Check utilise une base de données de programme (.pdb), qui contient un type et des informations de débogage symboliques du programme lors de la compilation.

Invite de commande

```
C:\Users\thm\Desktop\Samples>strings AV-Check.exe | findstr pdb
C:\Users\thm\source\repos\AV-Check\AV-Check\obj\Debug\AV-Check.pdb
```

Nous utiliserons le chemin dans la sortie de la commande précédente comme exemple de chaîne unique dans la règle Yara que nous allons créer. La signature peut être autre chose dans le monde réel, comme des clés de registre, des commandes, etc. Si vous n'êtes pas familier avec Yara, nous vous suggérons de consulter la [salle Yara THM](https://tryhackme.com/room/yara) . Voici la règle de Yara que nous utiliserons dans notre détection :

```
rule thm_demo_rule {
	meta:
		author = "THM: Intro-to-AV-Room"
		description = "Look at how the Yara rule works with ClamAV"
	strings:
		$a = "C:\\Users\\thm\\source\\repos\\AV-Check\\AV-Check\\obj\\Debug\\AV-Check.pdb"
	condition:
		$a
}
```

Expliquons un peu plus la règle de Yara.

-   La règle commence par  la règle thm_demo_rule ,  qui est le nom de notre règle. ClamAV utilise ce nom si une règle correspond.
-   La section des métadonnées, qui contient des informations générales, contient l'auteur et la description, que l'utilisateur peut remplir.
-   La section chaînes contient les chaînes ou le bytecode que nous recherchons. Nous utilisons dans ce cas le chemin de la base de données du programme C#. Notez que nous ajoutons un  \ supplémentaire  dans ce chemin pour échapper au caractère spécial, afin de ne pas enfreindre la règle. 
-   Dans la section condition, nous spécifions si la chaîne définie se trouve dans la section chaîne, puis marquons le fichier.

Notez que les règles Yara doivent être stockées dans un  fichier d'extension .yara  pour que ClamAV puisse les gérer. Analysons à nouveau le  dossier c:\Users\thm\Desktop\Samples  à l'aide de la règle Yara que nous avons créée. Vous pouvez trouver une copie de la règle Yara sur le bureau à l'adresse  c:\Users\thm\Desktop\Files\thm-demo-1.yara .

Numérisation à l'aide de la règle Yara

```
C:\Users\thm>"c:\Program Files\ClamAV\clamscan.exe" -d Desktop\Files\thm-demo-1.yara Desktop\Samples
Loading:     0s, ETA:   0s [========================>]        1/1 sigs
Compiling:   0s, ETA:   0s [========================>]       40/40 tasks

C:\Users\thm\Desktop\Samples\AV-Check.exe: YARA.thm_demo_rule.UNOFFICIAL FOUND
C:\Users\thm\Desktop\Samples\backdoor1.exe: OK
C:\Users\thm\Desktop\Samples\backdoor2.exe: OK
C:\Users\thm\Desktop\Samples\eicar.com: OK
C:\Users\thm\Desktop\Samples\notes.txt: YARA.thm_demo_rule.UNOFFICIAL FOUND
```

En conséquence, ClamAV peut détecter le  binaire AV-Check.exe  comme malveillant sur la base de la règle Yara que nous fournissons. Cependant, ClamAV a donné un résultat faussement positif en signalant le  fichier notes.txt  comme malveillant. Si nous ouvrons le  fichier notes.txt  , nous pouvons voir que le texte contient le même chemin que nous avons spécifié dans la règle.

Améliorons notre règle Yara pour réduire les résultats faussement positifs. Nous préciserons le type de fichier dans notre règle. Souvent, les types d'un fichier peuvent être identifiés à l'aide de nombres magiques, qui constituent les deux premiers octets du binaire. Par exemple, [les fichiers exécutables](https://en.wikipedia.org/wiki/DOS_MZ_executable) (.exe) commencent toujours par la valeur ASCII « MZ » ou « 4D 5A » en hexadécimal.

Pour confirmer cela, utilisons l' application [HxD](https://mh-nexus.de/en/hxd/) , qui est un éditeur hexadécimal gratuit, pour examiner le binaire AV-Check.exe et voir les deux premiers octets. Notez que le HxD est  déjà disponible dans la VM fournie .

![44dfc0fa904b0e4a9dfe983001d38a2e](https://github.com/dsgsec/Red-Team/assets/82456829/c06ea1a5-faa5-47bd-9f5d-7a05f40aa047)

Sachant que cela contribuera à améliorer la détection, incluons cela dans notre règle Yara pour signaler uniquement les fichiers .exe contenant notre chaîne de signature comme malveillants. Voici la règle Yara améliorée :

```
rule thm_demo_rule {
	meta:
		author = "THM: Intro-to-AV-Room"
		description = "Look at how the Yara rule works with ClamAV"
	strings:
		$a = "C:\\Users\\thm\\source\\repos\\AV-Check\\AV-Check\\obj\\Debug\\AV-Check.pdb"
		$b = "MZ"
	condition:
		$b at 0 and $a
}
```

Dans la nouvelle règle Yara, nous avons défini une chaîne unique ($b) égale au MZ comme identifiant du type de fichier .exe. Nous avons également mis à jour la section des conditions, qui comprend désormais les conditions suivantes :

1.  Si la chaîne "MZ" est trouvée à l'emplacement 0, c'est le début du fichier.
2.  Si la chaîne unique (le chemin) apparaît dans le binaire.
3.  Dans la section condition, nous avons utilisé l'  opérateur AND  pour que les deux  définitions en 1 et 2 soient trouvées, nous avons alors une correspondance.  

Vous pouvez trouver la règle mise à jour dans  Desktop\Files\thm-demo-2.yara . Maintenant que nous avons notre règle Yara mise à jour, essayons à nouveau.

Numérisation à l'aide de la règle Yara

```
C:\Users\thm>"c:\Program Files\ClamAV\clamscan.exe" -d Desktop\Files\thm-demo-2.yara Desktop\Samples
Loading:     0s, ETA:   0s [========================>]        1/1 sigs
Compiling:   0s, ETA:   0s [========================>]       40/40 tasks

C:\Users\thm\Desktop\Samples\AV-Check.exe: YARA.thm_demo_rule.UNOFFICIAL FOUND
C:\Users\thm\Desktop\Samples\backdoor1.exe: OK
C:\Users\thm\Desktop\Samples\backdoor2.exe: OK
C:\Users\thm\Desktop\Samples\eicar.com: OK
C:\Users\thm\Desktop\Samples\notes.txt: OK
```

Le résultat montre que nous avons amélioré notre règle Yara pour réduire les résultats faussement positifs. C'était un exemple simple du fonctionnement d'un logiciel audiovisuel . Ainsi, les fournisseurs de logiciels antivirus travaillent dur pour lutter contre les logiciels malveillants et améliorer leurs produits et leurs bases de données afin d'améliorer les performances et la précision des résultats.

L'inconvénient de la détection basée sur les signatures est que les fichiers  auront une valeur de hachage différente si le binaire est modifié. Par conséquent, il est facile pour quelqu'un de contourner les techniques de détection basées sur les signatures s'il sait ce que recherche un logiciel antivirus et comment analyser les binaires, comme indiqué dans les salles suivantes.
