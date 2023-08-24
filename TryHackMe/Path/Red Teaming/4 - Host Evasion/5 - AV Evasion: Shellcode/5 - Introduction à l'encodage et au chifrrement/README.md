## Qu'est-ce que l'encodage ?

Le codage est le processus de modification des données de leur état d'origine vers un format spécifique en fonction de l'algorithme ou du type de codage. Il peut être appliqué à de nombreux types de données tels que les vidéos, le HTML, les URL et les fichiers binaires (EXE, images, etc.).

L'encodage est un concept important qui  est couramment utilisé à diverses fins, notamment :

-   Compilation et exécution du programme
-   Stockage et transmission de données
-   Traitement des données tel que la conversion de fichiers

De même, lorsqu'il s'agit de techniques d'évasion AV , le codage est également utilisé pour masquer les chaînes de shellcode dans un binaire. Cependant, le codage ne suffit pas à des fins d'évasion. De nos jours, les logiciels audiovisuels sont plus intelligents et peuvent analyser un binaire, et une fois qu'une chaîne codée est trouvée, elle est décodée pour vérifier la forme originale du texte.

Vous pouvez également utiliser deux ou plusieurs algorithmes d'encodage en tandem pour rendre plus difficile la détection du contenu caché par l' AV . La figure suivante montre que nous avons converti la chaîne « THM » en représentation hexadécimale, puis l'avons codée en Base64. Dans ce cas, vous devez vous assurer que votre compte-gouttes gère désormais un tel encodage pour restaurer la chaîne à son état d'origine.

![6b3d06034f60f472a3c3620815d25be1](https://github.com/dsgsec/Red-Team/assets/82456829/61ba7f78-8e6b-4181-b2b8-8d3456ffac9e)

## Qu'est-ce que le chiffrement ?

Le cryptage est l'un des éléments essentiels de la sécurité des informations et des données qui vise à empêcher l'accès non autorisé et la manipulation des données. Le processus de cryptage implique la conversion du texte brut (contenu non crypté) en une version cryptée appelée Ciphertext. Le texte chiffré ne peut pas être lu ou déchiffré sans connaître l'algorithme utilisé pour le cryptage ainsi que la clé.

Tout comme le codage, les techniques de chiffrement sont utilisées à diverses fins, telles que le stockage et la transmission sécurisée de données, ainsi que le chiffrement de bout en bout. Le chiffrement peut être utilisé de deux manières : en ayant une clé partagée entre deux parties ou en utilisant des clés publiques et privées.

Pour plus d'informations sur le cryptage, nous vous encourageons à consulter la salle [Encryption - Crypto 101](https://tryhackme.com/room/encryptioncrypto101) .

![7cd2d23f048dd314a46910fb29c9836f](https://github.com/dsgsec/Red-Team/assets/82456829/c2e7b93a-e5bb-433f-9c49-d6fc8f8e3737)

Pourquoi devons-nous connaître le codage et le cryptage ?

Les fournisseurs audiovisuels implémentent leur logiciel audiovisuel pour bloquer la plupart des outils publics (tels que Metasploit et autres) à l'aide de techniques de détection statiques ou dynamiques. Par conséquent, sans modifier le shellcode  généré  par ces outils publics, le taux de détection de votre compte-gouttes est élevé.

Le codage et le cryptage peuvent être utilisés dans les techniques d'évasion AV où nous codons et/ou chiffrons le shellcode utilisé dans un compte-gouttes pour le cacher aux logiciels AV pendant l'exécution. De plus, les deux techniques peuvent être utilisées non seulement pour masquer le shellcode mais également les fonctions, variables, etc. Dans cette salle, nous nous concentrons principalement sur le chiffrement du shellcode pour échapper à Windows Defender.
