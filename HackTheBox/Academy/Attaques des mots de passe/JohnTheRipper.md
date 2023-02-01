# JohnTheRipper

## Type d'attaques

Il existe différents type d'attaques réalisables avec `John`.

### Dictionnaire
Les attaques par dictionnaire impliquent l'utilisation d'une liste pré-générée de mots et de phrases (appelée dictionnaire) pour tenter de déchiffrer un mot de passe. Cette liste de mots et d'expressions est souvent acquise à partir de diverses sources, telles que des dictionnaires accessibles au public, des mots de passe ayant fait l'objet d'une fuite ou même achetée auprès d'entreprises spécialisées. Le dictionnaire est ensuite utilisé pour générer une série de chaînes qui sont ensuite utilisées pour comparer avec les mots de passe hachés. Si une correspondance est trouvée, le mot de passe est déchiffré, permettant à un attaquant d'accéder au système et aux données qui y sont stockées. Ce type d'attaque est très efficace. Par conséquent, il est essentiel de prendre les mesures nécessaires pour garantir la sécurité des mots de passe, comme l'utilisation de mots de passe complexes et uniques, leur modification régulière et l'utilisation d'une authentification à deux facteurs.

### Bruteforce
Les attaques par force brute impliquent de tenter toutes les combinaisons imaginables de caractères pouvant former un mot de passe. Il s'agit d'un processus extrêmement lent, et l'utilisation de cette méthode n'est généralement conseillée que s'il n'y a pas d'autres alternatives. Il est également important de noter que plus le mot de passe est long et complexe, plus il est difficile à déchiffrer et plus il faudra de temps pour épuiser chaque combinaison. Pour cette raison, il est fortement recommandé que les mots de passe comportent au moins 8 caractères, avec une combinaison de lettres, de chiffres et de symboles.

### Rainbow Table Attacks
Les attaques par table arc-en-ciel impliquent l'utilisation d'une table pré-calculée de hachages et de leurs mots de passe en clair correspondants, ce qui est une méthode beaucoup plus rapide qu'une attaque par force brute. Cependant, cette méthode est limitée par la taille de la table arc-en-ciel - plus la table est grande, plus elle peut stocker de mots de passe et de hachages. De plus, en raison de la nature de l'attaque, il est impossible d'utiliser des tables arc-en-ciel pour déterminer le texte en clair des hachages qui ne sont pas déjà inclus dans la table. En conséquence, les attaques de table arc-en-ciel ne sont efficaces que contre les hachages déjà présents dans la table, ce qui fait que plus la table est grande, plus l'attaque est réussie.

## Cracking

### Single Crack
Le mode Single Crack est l'un des modes John les plus couramment utilisés pour tenter de déchiffrer des mots de passe à l'aide d'une seule liste de mots de passe. Il s'agit d'une attaque par force brute, ce qui signifie que tous les mots de passe de la liste sont essayés, un par un, jusqu'à ce que le bon soit trouvé. Cette méthode est le moyen le plus simple et le plus simple de déchiffrer les mots de passe et constitue donc un choix populaire pour ceux qui souhaitent accéder à un système sécurisé. C'est cependant loin d'être la méthode la plus efficace car cela peut prendre un temps indéfini pour déchiffrer un mot de passe, selon la longueur et la complexité du mot de passe en question. La syntaxe de base de la commande est :
```
dsgsec@htb[/htb]$ john --format=<hash_type> <hash or hash_file>
```

Par exemple, si nous avons un fichier nommé hashes_to_crack.txt qui contient des hachages SHA-256, la commande pour les craquer serait :
```
dsgsec@htb[/htb]$ john --format=sha256 hashes_to_crack.txt
```

1. **john** est la commande pour exécuter le programme John the Ripper
2. **--format=sha256** spécifie que le format de hachage est SHA-256
3. **hashes.txt** est le nom du fichier contenant les hachages à cracker

### Cracking avec JohnTR
| Hash Format          | Example Command                                     | Description                                                          |
| -------------------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| afs                  | `john --format=afs hashes_to_crack.txt`             | AFS (Andrew File System) password hashes                             |
| bfegg                | `john --format=bfegg hashes_to_crack.txt`           | bfegg hashes used in Eggdrop IRC bots                                |
| bf                   | `john --format=bf hashes_to_crack.txt`              | Blowfish-based crypt(3) hashes                                       |
| bsdi                 | `john --format=bsdi hashes_to_crack.txt`            | BSDi crypt(3) hashes                                                 |
| crypt(3)             | `john --format=crypt hashes_to_crack.txt`           | Traditional Unix crypt(3) hashes                                     |
| des                  | `john --format=des hashes_to_crack.txt`             | Traditional DES-based crypt(3) hashes                                |
| dmd5                 | `john --format=dmd5 hashes_to_crack.txt`            | DMD5 (Dragonfly BSD MD5) password hashes                             |
| dominosec            | `john --format=dominosec hashes_to_crack.txt`       | IBM Lotus Domino 6/7 password hashes                                 |
| EPiServer SID hashes | `john --format=episerver hashes_to_crack.txt`       | EPiServer SID (Security Identifier) password hashes                  |
| hdaa                 | `john --format=hdaa hashes_to_crack.txt`            | hdaa password hashes used in Openwall GNU/Linux                      |
| hmac-md5             | `john --format=hmac-md5 hashes_to_crack.txt`        | hmac-md5 password hashes                                             |
| hmailserver          | `john --format=hmailserver hashes_to_crack.txt`     | hmailserver password hashes                                          |
| ipb2                 | `john --format=ipb2 hashes_to_crack.txt`            | Invision Power Board 2 password hashes                               |
| krb4                 | `john --format=krb4 hashes_to_crack.txt`            | Kerberos 4 password hashes                                           |
| krb5                 | `john --format=krb5 hashes_to_crack.txt`            | Kerberos 5 password hashes                                           |
| LM                   | `john --format=LM hashes_to_crack.txt`              | LM (Lan Manager) password hashes                                     |
| lotus5               | `john --format=lotus5 hashes_to_crack.txt`          | Lotus Notes/Domino 5 password hashes                                 |
| md4-gen              | `john --format=md4-gen hashes_to_crack.txt`         | Generic MD4 password hashes                                          |
| md5                  | `john --format=md5 hashes_to_crack.txt`             | MD5 password hashes                                                  |
| md5-gen              | `john --format=md5-gen hashes_to_crack.txt`         | Generic MD5 password hashes                                          |
| mscash               | `john --format=mscash hashes_to_crack.txt`          | MS Cache password hashes                                             |
| mscash2              | `john --format=mscash2 hashes_to_crack.txt`         | MS Cache v2 password hashes                                          |
| mschapv2             | `john --format=mschapv2 hashes_to_crack.txt`        | MS CHAP v2 password hashes                                           |
| mskrb5               | `john --format=mskrb5 hashes_to_crack.txt`          | MS Kerberos 5 password hashes                                        |
| mssql05              | `john --format=mssql05 hashes_to_crack.txt`         | MS SQL 2005 password hashes                                          |
| mssql                | `john --format=mssql hashes_to_crack.txt`           | MS SQL password hashes                                               |
| mysql-fast           | `john --format=mysql-fast hashes_to_crack.txt`      | MySQL fast password hashes                                           |
| mysql                | `john --format=mysql hashes_to_crack.txt`           | MySQL password hashes                                                |
| mysql-sha1           | `john --format=mysql-sha1 hashes_to_crack.txt`      | MySQL SHA1 password hashes                                           |
| NETLM                | `john --format=netlm hashes_to_crack.txt`           | NETLM (NT LAN Manager) password hashes                               |
| NETLMv2              | `john --format=netlmv2 hashes_to_crack.txt`         | NETLMv2 (NT LAN Manager version 2) password hashes                   |
| NETNTLM              | `john --format=netntlm hashes_to_crack.txt`         | NETNTLM (NT LAN Manager) password hashes                             |
| NETNTLMv2            | `john --format=netntlmv2 hashes_to_crack.txt`       | NETNTLMv2 (NT LAN Manager version 2) password hashes                 |
| NEThalfLM            | `john --format=nethalflm hashes_to_crack.txt`       | NEThalfLM (NT LAN Manager) password hashes                           |
| md5ns                | `john --format=md5ns hashes_to_crack.txt`           | md5ns (MD5 namespace) password hashes                                |
| nsldap               | `john --format=nsldap hashes_to_crack.txt`          | nsldap (OpenLDAP SHA) password hashes                                |
| ssha                 | `john --format=ssha hashes_to_crack.txt`            | ssha (Salted SHA) password hashes                                    |
| NT                   | `john --format=nt hashes_to_crack.txt`              | NT (Windows NT) password hashes                                      |
| openssha             | `john --format=openssha hashes_to_crack.txt`        | OPENSSH private key password hashes                                  |
| oracle11             | `john --format=oracle11 hashes_to_crack.txt`        | Oracle 11 password hashes                                            |
| oracle               | `john --format=oracle hashes_to_crack.txt`          | Oracle password hashes                                               |
| pdf                  | `john --format=pdf hashes_to_crack.txt`             | PDF (Portable Document Format) password hashes                       |
| phpass-md5           | `john --format=phpass-md5 hashes_to_crack.txt`      | PHPass-MD5 (Portable PHP password hashing framework) password hashes |
| phps                 | `john --format=phps hashes_to_crack.txt`            | PHPS password hashes                                                 |
| pix-md5              | `john --format=pix-md5 hashes_to_crack.txt`         | Cisco PIX MD5 password hashes                                        |
| po                   | `john --format=po hashes_to_crack.txt`              | Po (Sybase SQL Anywhere) password hashes                             |
| rar                  | `john --format=rar hashes_to_crack.txt`             | RAR (WinRAR) password hashes                                         |
| raw-md4              | `john --format=raw-md4 hashes_to_crack.txt`         | Raw MD4 password hashes                                              |
| raw-md5              | `john --format=raw-md5 hashes_to_crack.txt`         | Raw MD5 password hashes                                              |
| raw-md5-unicode      | `john --format=raw-md5-unicode hashes_to_crack.txt` | Raw MD5 Unicode password hashes                                      |
| raw-sha1             | `john --format=raw-sha1 hashes_to_crack.txt`        | Raw SHA1 password hashes                                             |
| raw-sha224           | `john --format=raw-sha224 hashes_to_crack.txt`      | Raw SHA224 password hashes                                           |
| raw-sha256           | `john --format=raw-sha256 hashes_to_crack.txt`      | Raw SHA256 password hashes                                           |
| raw-sha384           | `john --format=raw-sha384 hashes_to_crack.txt`      | Raw SHA384 password hashes                                           |
| raw-sha512           | `john --format=raw-sha512 hashes_to_crack.txt`      | Raw SHA512 password hashes                                           |
| salted-sha           | `john --format=salted-sha hashes_to_crack.txt`      | Salted SHA password hashes                                           |
| sapb                 | `john --format=sapb hashes_to_crack.txt`            | SAP CODVN B (BCODE) password hashes                                  |
| sapg                 | `john --format=sapg hashes_to_crack.txt`            | SAP CODVN G (PASSCODE) password hashes                               |
| sha1-gen             | `john --format=sha1-gen hashes_to_crack.txt`        | Generic SHA1 password hashes                                         |
| skey                 | `john --format=skey hashes_to_crack.txt`            | S/Key (One-time password) hashes                                     |
| ssh                  | `john --format=ssh hashes_to_crack.txt`             | SSH (Secure Shell) password hashes                                   |
| sybasease            | `john --format=sybasease hashes_to_crack.txt`       | Sybase ASE password hashes                                           |
| xsha                 | `john --format=xsha hashes_to_crack.txt`            | xsha (Extended SHA) password hashes                                  |
| zip                  | `john --format=zip hashes_to_crack.txt`             | ZIP (WinZip) password hashes                                         |

### Wordlist

```
dsgsec@htb[/htb]$ john --wordlist=<wordlist_file> --rules <hash_file>
```

### Incremental Mode
Le mode incrémentiel est un mode John avancé utilisé pour déchiffrer les mots de passe à l'aide d'un jeu de caractères. Il s'agit d'une attaque hybride, ce qui signifie qu'elle tentera de faire correspondre le mot de passe en essayant toutes les combinaisons possibles de caractères du jeu de caractères. Ce mode est le plus efficace et le plus chronophage de tous les modes John. Ce mode fonctionne mieux lorsque nous connaissons le mot de passe, car il essaiera toutes les combinaisons possibles en séquence, en commençant par la plus courte. Cela le rend beaucoup plus rapide que l'attaque par force brute, où toutes les combinaisons sont essayées au hasard. De plus, le mode incrémentiel peut également être utilisé pour cracker des mots de passe faibles, ce qui peut être difficile à cracker en utilisant les modes John standard. La principale différence entre le mode incrémentiel et le mode liste de mots est la source des suppositions de mot de passe. Le mode incrémental génère les suppositions à la volée, tandis que le mode liste de mots utilise une liste prédéfinie de mots. Dans le même temps, le mode de crack unique est utilisé pour vérifier un seul mot de passe par rapport à un hachage.

La syntaxe pour exécuter John the Ripper en mode incrémentiel est la suivante :
```
dsgsec@htb[/htb]$ john --incremental <hash_file>
```

## Cracker un fichier avec John
```
cry0l1t3@htb:~$ <tool> <file_to_crack> > file.hash
cry0l1t3@htb:~$ pdf2john server_doc.pdf > server_doc.hash
cry0l1t3@htb:~$ john server_doc.hash
                # OR
cry0l1t3@htb:~$ john --wordlist=<wordlist.txt> server_doc.hash 
```

### Autres outils
| Tool                     | Description                                   |
| ----------------------- | --------------------------------------------- |
| `pdf2john`              | Converts PDF documents for John               |
| `ssh2john`              | Converts SSH private keys for John            |
| `mscash2john`           | Converts MS Cash hashes for John              |
| `keychain2john`         | Converts OS X keychain files for John         |
| `rar2john`              | Converts RAR archives for John                |
| `pfx2john`              | Converts PKCS#12 files for John               |
| `truecrypt_volume2john` | Converts TrueCrypt volumes for John           |
| `keepass2john`          | Converts KeePass databases for John           |
| `vncpcap2john`          | Converts VNC PCAP files for John              |
| `putty2john`            | Converts PuTTY private keys for John          |
| `zip2john`              | Converts ZIP archives for John                |
| `hccap2john`            | Converts WPA/WPA2 handshake captures for John |
| `office2john`           | Converts MS Office documents for John         |
| `wpa2john`              | Converts WPA/WPA2 handshakes for John         |
