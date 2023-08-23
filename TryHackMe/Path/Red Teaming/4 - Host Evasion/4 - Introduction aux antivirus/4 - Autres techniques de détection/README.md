Le concept de détection statique est relativement simple. Dans cette section, nous aborderons les différents types de techniques de détection.

## Détection dynamique

L'approche de détection dynamique est avancée et plus compliquée que la détection statique. La détection dynamique se concentre davantage sur la vérification des fichiers au moment de l'exécution à l'aide de différentes méthodes. Le diagramme suivant montre le flux d'analyse de détection dynamique :

![5d0d8cd90a2c4b2ce035af1f3b735563](https://github.com/dsgsec/Red-Team/assets/82456829/4e0200b5-85c9-42d6-9d43-3180dd8ce5b0)

La première méthode consiste à surveiller les API Windows. Le moteur de détection inspecte les appels d'application Windows et surveille les appels d'API Windows à l'aide de Windows [Hooks](https://docs.microsoft.com/en-us/windows/win32/winmsg/about-hooks) .

Une autre méthode de détection dynamique est le Sandboxing. Un bac à sable est un environnement virtualisé utilisé pour exécuter des fichiers malveillants séparés de l'ordinateur hôte. Cela se fait généralement dans un environnement isolé et l'objectif principal est d'analyser la manière dont les logiciels malveillants agissent dans le système. Une fois le logiciel malveillant confirmé, une signature et une règle uniques seront créées en fonction des caractéristiques du binaire. Enfin, une nouvelle mise à jour sera intégrée à la base de données cloud pour une utilisation future.

Ce type de détection présente également des inconvénients car elle nécessite l'exécution et l'exécution du logiciel malveillant pendant une durée limitée dans l'environnement virtuel pour protéger les ressources du système. Comme pour les autres techniques de détection, la détection dynamique peut être contournée. Les développeurs de logiciels malveillants implémentent leur logiciel pour ne pas fonctionner dans l'environnement virtuel ou simulé afin d'éviter l'analyse dynamique. Par exemple, ils vérifient si le système génère un véritable processus d'exécution du logiciel avant d'exécuter des activités malveillantes ou laissent le logiciel attendre un certain temps avant son exécution.

Pour plus d'informations sur l'évasion sandbox, nous vous suggérons de consulter la salle THM : [Sandbox Evasion](https://tryhackme.com/room/sandboxevasion) !

## Détection heuristique et comportementale

La détection heuristique et comportementale est devenue essentielle dans les produits audiovisuels modernes d'aujourd'hui . Les logiciels audiovisuels modernes s'appuient sur ce type de détection pour détecter les logiciels malveillants. L'analyse heuristique utilise diverses techniques, notamment des méthodes heuristiques statiques et dynamiques :

1.  L'analyse heuristique statique est un processus de décompilation (si possible) et d'extraction du code source du logiciel malveillant. Ensuite, le code source extrait est comparé à d'autres codes sources de virus bien connus. Ces codes sources sont préalablement connus et prédéfinis dans une base de données heuristique. Si une correspondance atteint ou dépasse un pourcentage seuil, le code est signalé comme malveillant.
2.  L'analyse heuristique dynamique est basée sur des règles comportementales prédéfinies. Les chercheurs en sécurité ont analysé les logiciels suspects dans des environnements isolés et sécurisés. Sur la base de leurs conclusions, ils ont signalé le logiciel comme malveillant. Ensuite, des règles comportementales sont créées pour correspondre aux activités malveillantes du logiciel sur une machine cible.

Voici des exemples de règles comportementales : 

-   Si un processus tente d'interagir avec le processus LSASS.exe qui contient les hachages NTLM des utilisateurs, les tickets Kerberos, etc.
-   Si un processus ouvre un port d'écoute et attend de recevoir des commandes d'un serveur de commande et de contrôle (C2)

Le diagramme suivant montre le flux d'analyse de détection heuristique et comportementale :

![bbdeb1fd3e1140a20ebee8553b194054](https://github.com/dsgsec/Red-Team/assets/82456829/6c625a39-5583-4769-aca3-8acfef6efaa8)

## Résumer les méthodes de détection

Résumons comment les logiciels audiovisuels modernes fonctionnent comme une seule unité, comprenant tous les composants, et combinent diverses fonctionnalités et techniques de détection pour mettre en œuvre son moteur audiovisuel. Voici un exemple des composants d'un moteur antivirus :

![61b1c8a009202170cf3deee3cefe7ccf](https://github.com/dsgsec/Red-Team/assets/82456829/922efd73-4a20-494e-a034-1d934df7ee10)

Dans le diagramme, vous pouvez voir qu'un  fichier Foobar.zip suspect est transmis au logiciel AV pour analyse. Le logiciel AV reconnaît qu'il s'agit d'un fichier compressé (.zip). Étant donné que le logiciel prend en charge les fichiers .zip, il  applique une fonction de désarchivage pour extraire les fichiers ( Foobar.exe ). Ensuite, il identifie le type de fichier pour savoir avec quel module travailler, puis effectue une opération d'analyse PE pour extraire les informations du binaire et d'autres caractéristiques. Ensuite, il vérifie si le fichier est compressé ; si c'est le cas, il décompresse le code.  Enfin, il transmet les informations collectées et le binaire au moteur AV , où il tente de détecter s'il est malveillant et nous donne le résultat.
