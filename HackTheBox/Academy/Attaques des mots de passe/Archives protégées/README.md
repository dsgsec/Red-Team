# Archives protégées
Outre les fichiers autonomes, il existe également un autre format de fichiers pouvant contenir non seulement des données, telles qu'un document Office ou un PDF, mais également d'autres fichiers qu'ils contiennent. Ce format s'appelle une archive ou un fichier compressé qui peut être protégé par un mot de passe si nécessaire.

Prenons le rôle d'un employé dans une entreprise administrative et imaginons que notre client souhaite résumer l'analyse dans différents formats, tels que Excel, PDF, Word et une présentation correspondante. Une solution serait d'envoyer ces fichiers individuellement, mais si l'on étend cet exemple à une grande entreprise traitant plusieurs projets en simultané, ce type de transfert de fichiers peut devenir lourd et entraîner la perte de fichiers individuels. Dans ces cas, les employés s'appuient souvent sur des archives, qui leur permettent de diviser tous les fichiers nécessaires de manière structurée selon les projets (souvent dans des sous-dossiers), de les résumer et de les regrouper dans un seul fichier.

Il existe de nombreux types de fichiers d'archives. Certaines extensions de fichiers courantes incluent, mais ne sont pas limitées à:
|  |  |  |
| --- | --- | --- |
| `tar` | `gz` | `rar` | `zip` |
| `vmdb/vmx` | `cpt` | `truecrypt` | `bitlocker` |
| `kdbx` | `luks` | `deb` | `7z` |
| `pkg` | `rpm` | `war` | `gzip` |

Une liste complète des types d'archives peut être trouvée sur FileInfo.com. Cependant, au lieu de les saisir manuellement, nous pouvons également les interroger à l'aide d'une ligne, les filtrer et les enregistrer dans un fichier si nécessaire. Au moment de la rédaction de cet article, 337 types de fichiers d'archives sont répertoriés sur fileinfo.com.

Télécharger toutes les extensions de fichiers
```
dsgsec@htb[/htb]$ curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt

.mint
.htmi 
.tpsr
.mpkg  
.arduboy
.ice
.sifz 
.fzpz 
.rar     
.comppkg.hauptwerk.rar
...SNIP...
```

Il est important de noter que toutes les archives ci-dessus ne prennent pas en charge la protection par mot de passe. D'autres outils sont souvent utilisés pour protéger les archives correspondantes avec un mot de passe. Par exemple, avec tar, l'outil openssl ou gpg est utilisé pour chiffrer les archives.

## Craquer des archives
Étant donné le nombre d'archives différentes et la combinaison d'outils, nous ne montrerons que quelques-unes des façons possibles de casser des archives spécifiques dans cette section. En ce qui concerne les archives protégées par mot de passe, nous avons généralement besoin de certains scripts qui nous permettent d'extraire les hachages des fichiers protégés et de les utiliser pour déchiffrer le mot de passe de ceux-ci.

Le format .zip est souvent largement utilisé dans les environnements Windows pour compresser de nombreux fichiers en un seul fichier. La procédure que nous avons déjà vue reste la même à l'exception de l'utilisation d'un script différent pour extraire les hachages.

### Craquage ZIP
#### Utilisation de zip2john
```
dsgsec@htb[/htb]$ zip2john ZIP.zip > zip.hash

ver 2.0 efh 5455 efh 7875 ZIP.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=42, decmplen=30, crc=490E7510
```

En extrayant les hachages, nous verrons également quels fichiers se trouvent dans l'archive ZIP.

Affichage du contenu de zip.hash
```
dsgsec@htb[/htb]$ cat zip.hash 

ZIP.zip/customers.csv:$pkzip2$1*2*2*0*2a*1e*490e7510*0*42*0*2a*490e*409b*ef1e7feb7c1cf701a6ada7132e6a5c6c84c032401536faf7493df0294b0d5afc3464f14ec081cc0e18cb*$/pkzip2$:customers.csv:ZIP.zip::ZIP.zip
```

Une fois que nous avons extrait le hachage, nous pouvons à nouveau utiliser john pour le déchiffrer avec la liste de mots de passe souhaitée. Parce que si John le cracke avec succès, il nous montrera le mot de passe correspondant que nous pouvons utiliser pour ouvrir l'archive ZIP.

### Casser le hachage avec John
```
dsgsec@htb[/htb]$ john --wordlist=rockyou.txt zip.hash

Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1234             (ZIP.zip/customers.csv)
1g 0:00:00:00 DONE (2022-02-09 09:18) 100.0g/s 250600p/s 250600c/s 250600C/s 123456..1478963
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

### Voir le hash casser
```
dsgsec@htb[/htb]$ john zip.hash --show

ZIP.zip/customers.csv:1234:customers.csv:ZIP.zip::ZIP.zip

1 password hash cracked, 0 left
```

## Craquage des archives cryptées OpenSSL
De plus, il n'est pas toujours évident de savoir si l'archive trouvée est protégée par un mot de passe, en particulier si une extension de fichier est utilisée qui ne prend pas en charge la protection par mot de passe. Comme nous l'avons déjà discuté, openssl peut être utilisé pour chiffrer le format gzip à titre d'exemple. En utilisant le fichier outil, nous pouvons obtenir des informations sur le format du fichier spécifié. Cela pourrait ressembler à ceci, par exemple :

### Lister les fichiers
```
dsgsec@htb[/htb]$ ls

GZIP.gzip  rockyou.txt
```

### Utiliser le fichier
```
dsgsec@htb[/htb]$ file GZIP.gzip 

GZIP.gzip: openssl enc'd data with salted password
```

Lors du cracking de fichiers et d'archives cryptés OpenSSL, nous pouvons rencontrer de nombreuses difficultés différentes qui apporteront de nombreux faux positifs ou même ne parviendront pas à deviner le mot de passe correct. Par conséquent, le choix le plus sûr pour réussir est d'utiliser l'outil openssl dans une boucle for qui essaie d'extraire les fichiers de l'archive directement si le mot de passe est deviné correctement.

Le one-liner suivant montrera de nombreuses erreurs liées au format GZIP, que nous pouvons ignorer. Si nous avons utilisé la bonne liste de mots de passe, comme dans cet exemple, nous verrons que nous avons réussi à extraire un autre fichier de l'archive.

### Utilisation d'une boucle for pour afficher le contenu extrait

```
dsgsec@htb[/htb]$ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

<SNIP>
```

Une fois la boucle for terminée, nous pouvons regarder à nouveau dans le dossier actuel pour vérifier si le cracking de l'archive a réussi.

### Liste du contenu de l'archive fissurée
```
Liste du contenu de l'archive fissurée
```

## Craquage de lecteurs chiffrés BitLocker
BitLocker est un programme de chiffrement pour des partitions entières et des disques externes. Microsoft l'a développé pour le système d'exploitation Windows. Il est disponible depuis Windows Vista et utilise l'algorithme de cryptage AES avec une longueur de 128 bits ou 256 bits. Si le mot de passe ou le code PIN de BitLocker est oublié, nous pouvons utiliser la clé de récupération pour déchiffrer la partition ou le lecteur. La clé de récupération est une chaîne de nombres à 48 chiffres générée lors de la configuration de BitLocker qui peut également être forcée brutalement.

Des lecteurs virtuels sont souvent créés dans lesquels des informations personnelles, des notes et des documents sont stockés sur l'ordinateur ou l'ordinateur portable fourni par l'entreprise pour empêcher l'accès à ces informations par des tiers. Encore une fois, nous pouvons utiliser un script appelé bitlocker2john pour extraire le hachage que nous devons craquer. Quatre hachages différents seront extraits, qui peuvent être utilisés avec différents modes de hachage Hashcat. Pour notre exemple, nous allons travailler avec le premier, qui fait référence au mot de passe BitLocker.

### Using bitlocker2john
```
dsgsec@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes
dsgsec@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash
dsgsec@htb[/htb]$ cat backup.hash

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e...SNIP...70696f7eab6b
```

John et Hashcat peuvent être utilisés à cette fin. Cet exemple examinera la procédure avec Hashcat. Le mode Hashcat pour craquer les hachages BitLocker est -m 22100. Nous fournissons donc à Hashcat le fichier avec le seul hachage, spécifions notre liste de mots de passe et spécifions le mode de hachage. Comme il s'agit d'un cryptage robuste (AES), le piratage peut prendre un certain temps, selon le matériel utilisé. De plus, nous pouvons spécifier le nom du fichier dans lequel le résultat doit être stocké.

### Utiliser hashcat pour cracker backup.hash
```
dsgsec@htb[/htb]$ hashcat -m 22100 backup.hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt -o backup.cracked

hashcat (v6.1.1) starting...

<SNIP>

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f:1234qwer
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: BitLocker
Hash.Target......: $bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$10...8ec54f
Time.Started.....: Wed Feb  9 11:46:40 2022 (1 min, 42 secs)
Time.Estimated...: Wed Feb  9 11:48:22 2022 (0 secs)
Guess.Base.......: File (/opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       28 H/s (8.79ms) @ Accel:32 Loops:4096 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2880/6163 (46.73%)
Rejected.........: 0/2880 (0.00%)
Restore.Point....: 2816/6163 (45.69%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:1044480-1048576
Candidates.#1....: chemical -> secrets

Started: Wed Feb  9 11:46:35 2022
Stopped: Wed Feb  9 11:48:23 2022
```

### Voir le hash cassé
```
dsgsec@htb[/htb]$ cat backup.cracked 

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f:1234qwer
```

