![7d8572294a70c545705aecc6c2453a3e](https://github.com/dsgsec/Red-Team/assets/82456829/80874275-3090-4f33-92a9-79ee0c6697ae)

Le shellcode est un ensemble d'instructions de code machine spécialement conçues qui indiquent au programme vulnérable d'exécuter des fonctions supplémentaires et, dans la plupart des cas, donnent accès à un shell système ou créent un shell de commande inverse.

Une fois que le shellcode est injecté dans un processus et exécuté par le logiciel ou programme vulnérable, il modifie le flux d'exécution du code pour mettre à jour les registres et les fonctions du programme afin d'exécuter le code de l'attaquant.

Il est généralement écrit en langage Assembly et traduit en opcodes hexadécimaux (codes opérationnels). L'écriture d'un shellcode unique et personnalisé permet d'éviter considérablement les logiciels audiovisuels. Mais écrire un shellcode personnalisé nécessite d'excellentes connaissances et compétences dans le langage Assembly, ce qui n'est pas une tâche facile ! 

## Un simple shellcode !

Afin de créer votre propre shellcode, un ensemble de compétences est requis :

-   Une bonne compréhension des architectures CPU x86 et x64.
-   Langage d'assemblage.
-   Solide connaissance des langages de programmation tels que C.
-   Familiarité avec les systèmes d'exploitation Linux et Windows.

Pour générer notre propre shellcode, nous devons écrire et extraire des octets du code machine assembleur. Pour cette tâche, nous utiliserons AttackBox pour créer un shellcode simple pour Linux  qui écrit la chaîne "THM, Rocks!". Le code assembleur suivant utilise deux fonctions principales :

-   Fonction System Write (sys_write) pour imprimer une chaîne que nous choisissons.
-   Fonction System Exit (sys_exit) pour terminer l'exécution du programme.

Pour appeler ces fonctions, nous utiliserons des appels système . Un appel système est la manière dont un programme demande au noyau de faire quelque chose. Dans ce cas, nous demanderons au noyau d'écrire une chaîne sur notre écran et de quitter le programme. Chaque système d'exploitation a une convention d'appel différente concernant les appels système, ce qui signifie que pour utiliser l'écriture sous Linux , vous utiliserez probablement un appel système différent de celui que vous utiliseriez sous Windows. Pour Linux 64 bits, vous pouvez appeler les fonctions nécessaires depuis le noyau en configurant les valeurs suivantes :

| rax  | Appel système | rdi              | rsi              | rdx           |
| ---- | ------------- | ---------------- | ---------------- | ------------- |
| 0x1  | sys_write     | non signé int fd | const char \*buf | nombre size_t |
| 0x3c | sys_exit      | int code_erreur  |                  |               |

Le tableau ci-dessus nous indique les valeurs que nous devons définir dans différents registres de processeur pour appeler les fonctions sys_write et sys_exit à l'aide d'appels système. Pour Linux 64 bits , le registre rax est utilisé pour indiquer la fonction du noyau que nous souhaitons appeler. Définir rax sur 0x1 oblige le noyau à exécuter sys_write, et définir rax sur 0x3c obligera le noyau à exécuter sys_exit. Chacune des deux fonctions nécessite certains paramètres pour fonctionner, qui peuvent être définis via les registres rdi, rsi et rdx. Vous pouvez trouver une référence complète des appels système Linux 64 bits disponibles [ici](https://blog.rchapman.org/posts/Linux_System_Call_Table_for_x86_64/) .

Pour `sys_write`, le premier paramètre envoyé `rdi`est le descripteur de fichier dans lequel écrire. Le deuxième paramètre  `rsi`est un pointeur vers la chaîne que nous voulons imprimer, et le troisième  `rdx`est la taille de la chaîne à imprimer.

Pour `sys_exit`, rdi doit être défini sur le code de sortie du programme. Nous utiliserons le code 0, ce qui signifie que le programme s'est terminé avec succès.

Copiez le code suivant sur votre AttackBox dans un fichier appelé`thm.asm` : 

```
global _start

section .text
_start:
    jmp MESSAGE      ; 1) let's jump to MESSAGE

GOBACK:
    mov rax, 0x1
    mov rdi, 0x1
    pop rsi          ; 3) we are popping into `rsi`; now we have the
                     ; address of "THM, Rocks!\r\n"
    mov rdx, 0xd
    syscall

    mov rax, 0x3c
    mov rdi, 0x0
    syscall

MESSAGE:
    call GOBACK       ; 2) we are going back, since we used `call`, that means
                      ; the return address, which is, in this case, the address
                      ; of "THM, Rocks!\r\n", is pushed into the stack.
    db "THM, Rocks!", 0dh, 0ah
```

Expliquons un peu plus le code ASM. Tout d'abord, notre chaîne de message est stockée à la fin de la section .text. Puisque nous avons besoin d'un pointeur vers ce message pour l'imprimer, nous passerons à l'instruction d'appel avant le message lui-même. Lorsqu'est `call GOBACK`exécuté, l'adresse de la prochaine instruction après l'appel sera poussée dans la pile, ce qui correspond à l'endroit où se trouve notre message. Notez que les 0dh, 0ah à la fin du message sont l'équivalent binaire d'une nouvelle ligne (\r\n).

Ensuite, le programme démarre la routine GOBACK et prépare les registres requis pour notre première fonction sys_write().

-   Nous spécifions la fonction sys_write en stockant 1 dans le registre rax.
-   Nous définissons rdi sur 1 pour imprimer la chaîne sur la console de l'utilisateur (STDOUT).
-   Nous insérons un pointeur vers notre chaîne, qui a été poussée lorsque nous avons appelé GOBACK et la stockons dans rsi.
-   Avec l'instruction syscall, nous exécutons la fonction sys_write avec les valeurs que nous avons préparées.
-   Pour la partie suivante, nous faisons de même pour appeler la fonction sys_exit, nous définissons donc 0x3c dans le registre rax et appelons la fonction syscall pour quitter le programme.

Ensuite, nous compilons et lions le code ASM pour créer un fichier exécutable Linux x64 et enfin exécuter le programme.

Assembleur et liez notre code

```
user@AttackBox$ nasm -f elf64 thm.asm
user@AttackBox$ ld thm.o -o thm
user@AttackBox$ ./thm
THM,Rocks!
```

Nous avons utilisé la `nasm`commande pour compiler le fichier asm, en spécifiant l' `-f elf64`option pour indiquer que nous compilons pour Linux 64 bits . Notez qu'en conséquence, nous obtenons un fichier .o, qui contient du code objet, qui doit être lié pour être un fichier exécutable fonctionnel. La`ld` commande permet de lier l'objet et d'obtenir l'exécutable final. L' `-o`option est utilisée pour spécifier le nom du fichier exécutable de sortie.

Maintenant que nous avons le programme ASM compilé, extrayons le shellcode avec la `objdump`commande en vidant la section .text du binaire compilé.

Vider la section .text

```
user@AttackBox$ objdump -d thm

thm:     file format elf64-x86-64

Disassembly of section .text:

0000000000400080 <_start>:
  400080:	eb 1e                	jmp    4000a0

0000000000400082 :
  400082:	b8 01 00 00 00       	mov    $0x1,%eax
  400087:	bf 01 00 00 00       	mov    $0x1,%edi
  40008c:	5e                   	pop    %rsi
  40008d:	ba 0d 00 00 00       	mov    $0xd,%edx
  400092:	0f 05                	syscall
  400094:	b8 3c 00 00 00       	mov    $0x3c,%eax
  400099:	bf 00 00 00 00       	mov    $0x0,%edi
  40009e:	0f 05                	syscall

00000000004000a0 :
  4000a0:	e8 dd ff ff ff       	callq  400082
  4000a5:	54                   	push   %rsp
  4000a6:	48                   	rex.W
  4000a7:	4d 2c 20             	rex.WRB sub $0x20,%al
  4000aa:	52                   	push   %rdx
  4000ab:	6f                   	outsl  %ds:(%rsi),(%dx)
  4000ac:	63 6b 73             	movslq 0x73(%rbx),%ebp
  4000af:	21                   	.byte 0x21
  4000b0:	0d                   	.byte 0xd
  4000b1:	0a                   	.byte 0xa
```

Nous devons maintenant extraire la valeur hexadécimale de la sortie ci-dessus. Pour ce faire, nous pouvons utiliser `objcopy` pour vider la `.text`section dans un nouveau fichier appelé `thm.text`au format binaire comme suit :

Extraire la section .text

```
user@AttackBox$ objcopy -j .text -O binary thm thm.text
```

Le thm.text contient notre shellcode au format binaire, donc pour pouvoir l'utiliser, nous devrons d'abord le convertir en hexadécimal. La `xxd`commande a l' `-i`option qui affichera directement le fichier binaire dans une chaîne C :

Afficher l'équivalent hexadécimal de notre shellcode

```
user@AttackBox$ xxd -i thm.text
unsigned char new_text[] = {
  0xeb, 0x1e, 0xb8, 0x01, 0x00, 0x00, 0x00, 0xbf, 0x01, 0x00, 0x00, 0x00,
  0x5e, 0xba, 0x0d, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xb8, 0x3c, 0x00, 0x00,
  0x00, 0xbf, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xe8, 0xdd, 0xff, 0xff,
  0xff, 0x54, 0x48, 0x4d, 0x2c, 0x20, 0x52, 0x6f, 0x63, 0x6b, 0x73, 0x21,
  0x0d, 0x0a
};
unsigned int new_text_len = 50;
```

Enfin, nous l'avons, un  shellcode formaté  issu de notre assemblage ASM. C'était amusant! Comme nous le voyons, du dévouement et des compétences sont nécessaires pour générer du shellcode pour votre travail !

Pour confirmer que le shellcode extrait fonctionne comme prévu, nous pouvons exécuter notre shellcode et l'injecter dans un programme C.

```
#include <stdio.h>

int main(int argc, char **argv) {
    unsigned char message[] = {
        0xeb, 0x1e, 0xb8, 0x01, 0x00, 0x00, 0x00, 0xbf, 0x01, 0x00, 0x00, 0x00,
        0x5e, 0xba, 0x0d, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xb8, 0x3c, 0x00, 0x00,
        0x00, 0xbf, 0x00, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xe8, 0xdd, 0xff, 0xff,
        0xff, 0x54, 0x48, 0x4d, 0x2c, 0x20, 0x52, 0x6f, 0x63, 0x6b, 0x73, 0x21,
        0x0d, 0x0a
    };

    (*(void(*)())message)();
    return 0;
}
```

Ensuite, nous le compilons et l'exécutons comme suit,

Compiler notre programme C

```
user@AttackBox$ gcc -g -Wall -z execstack thm.c -o thmx
user@AttackBox$ ./thmx
THM,Rocks!
```

Bon! Ça marche. Notez que nous compilons le programme C en désactivant la protection NX, ce qui peut nous empêcher d'exécuter correctement le code dans le segment de données ou la pile.

Comprendre les shellcodes et la manière dont ils sont créés est essentiel pour les tâches suivantes, en particulier lorsqu'il s'agit de chiffrer et d'encoder le shellcode.

Répondre aux questions ci-dessous

Modifiez votre programme C pour exécuter le shellcode suivant. Qu'est-ce que le drapeau ?

```
unsigned char message[] = {
  0xeb, 0x34, 0xb9, 0x00, 0x00, 0x00, 0x00, 0x5e, 0x48, 0x89, 0xf0, 0x80,
  0x34, 0x08, 0x01, 0x48, 0x83, 0xc1, 0x01, 0x48, 0x83, 0xf9, 0x19, 0x75,
  0xf2, 0xb8, 0x01, 0x00, 0x00, 0x00, 0xbf, 0x01, 0x00, 0x00, 0x00, 0xba,
  0x19, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xb8, 0x3c, 0x00, 0x00, 0x00, 0xbf,
  0x00, 0x00, 0x00, 0x00, 0x0f, 0x05, 0xe8, 0xc7, 0xff, 0xff, 0xff, 0x55,
  0x49, 0x4c, 0x7a, 0x78, 0x31, 0x74, 0x73, 0x2c, 0x30, 0x72, 0x36, 0x2c,
  0x34, 0x69, 0x32, 0x30, 0x30, 0x62, 0x31, 0x65, 0x32, 0x7c, 0x0d, 0x0a
};
```
