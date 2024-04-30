Quelles sont les vulnérabilités d'upload de fichiers?
--------------------------------------------------------------

Les vulnérabilités de téléchargement de fichiers sont lorsqu'un serveur Web permet aux utilisateurs de télécharger des fichiers sur son système de fichiers sans valider suffisamment leur nom, leur type, leur contenu ou leur taille. Ne pas appliquer correctement les restrictions sur celles-ci pourrait signifier que même une fonction de téléchargement d'image de base peut être utilisée pour télécharger des fichiers arbitraires et potentiellement dangereux à la place. Cela pourrait même inclure des fichiers de script côté serveur qui permettent l'exécution de code à distance.

Dans certains cas, le fait de télécharger le fichier est en soi suffisant pour causer des dommages. D'autres attaques peuvent impliquer une requête HTTP de suivi pour le fichier, généralement pour déclencher son exécution par le serveur.

Comment les vulnérabilités d'upload de fichiers surviennent-elles?
---------------------------------------------------------------------------

Compte tenu des dangers assez évidents, il est rare que les sites Web dans la nature n'aient aucune restriction sur les fichiers que les utilisateurs sont autorisés à télécharger. Plus communément, les développeurs implémentent ce qu'ils croient être une validation robuste qui est soit intrinsèquement défectueuse, soit facilement contournable.

Par exemple, ils peuvent tenter de mettre en liste noire les types de fichiers dangereux, mais ne tiennent pas compte des écarts d'analyse lors de la vérification des extensions de fichier. Comme pour toute liste noire, il est également facile d'omettre accidentellement des types de fichiers plus obscurs qui peuvent encore être dangereux.
