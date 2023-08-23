# Moteurs antivirus

Un moteur AV est chargé de rechercher et de supprimer le code et les fichiers malveillants. Un bon logiciel AV implémente un noyau AV efficace et solide qui analyse avec précision et rapidité  les fichiers malveillants . En outre, il doit gérer et prendre en charge différents types de fichiers, y compris les fichiers d'archives, où il peut auto-extraire et inspecter tous les fichiers compressés.

La plupart des produits audiovisuels partagent les mêmes fonctionnalités communes mais sont implémentés différemment, notamment :

-   Scanner
-   Techniques de détection
-   Compresseurs et archives
-   Déballeurs
-   Émulateurs

## Scanner

La fonction de scanner est incluse dans la plupart des produits audiovisuels : le logiciel audiovisuel s'exécute et analyse en temps réel ou à la demande. Cette fonctionnalité est disponible dans l'interface graphique ou via l'invite de commande. L'utilisateur peut l'utiliser chaque fois que nécessaire pour vérifier des fichiers ou des répertoires. La fonction d'analyse doit prendre en charge les types de fichiers malveillants les plus connus pour détecter et supprimer la menace. En outre, il peut également prendre en charge d'autres types d'analyse en fonction du logiciel AV, notamment les vulnérabilités, les e-mails, la mémoire Windows et le registre Windows.

## Techniques de détection

Une technique de détection AV recherche et détecte les fichiers malveillants ; différentes techniques de détection peuvent être utilisées dans le moteur AV, notamment :

-   La détection basée sur les signatures est la technique antivirus traditionnelle qui recherche des modèles et des signatures malveillants prédéfinis dans les fichiers.
-   La détection heuristique est une technique plus avancée qui inclut diverses méthodes comportementales pour analyser les fichiers suspects.
-   La détection dynamique est une technique qui comprend la surveillance des appels système et des API, ainsi que les tests et analyses dans un environnement isolé.

Nous aborderons ces techniques dans la tâche suivante. Un bon moteur antivirus est précis et détecte rapidement les fichiers malveillants avec moins de résultats faussement positifs. Nous présenterons plusieurs produits audiovisuels qui fournissent des résultats inexacts et classent mal un fichier.

## Compresseurs et archives

La fonctionnalité « Compresseurs et archives » doit être incluse dans tout logiciel AV . Il doit prendre en charge et être capable de gérer différents types de fichiers système, y compris les fichiers compressés ou archivés : ZIP, TGZ, 7z, XAR, RAR, etc. Le code malveillant tente souvent d'échapper aux solutions de sécurité basées sur l'hôte en se cachant dans les fichiers compressés.  Pour cette raison, le logiciel AV doit décompresser et analyser tous les fichiers avant qu'un utilisateur n'ouvre un fichier dans l'archive .

## Analyse et décompression PE  (Portable Executable) 

Les logiciels malveillants cachent et regroupent leur code malveillant en le compressant et en le chiffrant dans une charge utile. Il se décompresse et se décrypte pendant l'exécution pour rendre plus difficile l'analyse statique. Ainsi, le logiciel AV doit être capable de détecter et de décompresser la plupart des packers connus (UPX, Armadillo, ASPack, etc.) avant l'exécution de l'analyse statique.

Les développeurs de logiciels malveillants utilisent diverses techniques, telles que le Packing, pour réduire la taille et modifier la structure du fichier malveillant. L'emballage compresse le fichier exécutable d'origine pour le rendre plus difficile à analyser. Par conséquent, le logiciel AV doit disposer d'une fonction de décompression pour décompresser les fichiers exécutables protégés ou compressés dans le code d'origine.

Une autre fonctionnalité que le logiciel AV doit avoir est l'analyseur d'en-tête Windows Portable Executable (PE). L'analyse PE des fichiers exécutables permet de distinguer les logiciels malveillants et légitimes (fichiers .exe). Le format de fichier PE sous Windows (32 et 64 bits) contient diverses informations et ressources, telles que du code objet, des DLL, des fichiers d'icônes, des fichiers de polices et des core dumps.

## Émulateurs

Un émulateur est une fonctionnalité antivirus qui effectue une analyse plus approfondie des fichiers suspects. Une fois qu'un émulateur reçoit une demande, l'émulateur exécute les fichiers suspects (exe, DLL , PDF, etc.) dans un environnement virtualisé et contrôlé. Il surveille le comportement des fichiers exécutables pendant l'exécution, y compris les appels des API Windows, le registre et d'autres fichiers Windows. Voici des exemples d'artefacts que l'émulateur peut collecter :

-   Appels d'API
-   Vidages de mémoire
-   Modifications du système de fichiers
-   Journaliser les événements
-   Processus en cours d'exécution
-   Requêtes Web

Un émulateur arrête l'exécution d'un fichier lorsque suffisamment d'artefacts sont collectés pour détecter un logiciel malveillant.

## Autres caractéristiques communes

Voici quelques fonctionnalités courantes trouvées dans les produits audiovisuels :

-   Un pilote d'autoprotection pour se prémunir contre les logiciels malveillants attaquant le véritable AV .
-   Fonctionnalité de pare-feu et d'inspection du réseau.
-   Outils de ligne de commande et d'interface graphique.
-   Un démon ou un service.
-   Une console de gestion.
