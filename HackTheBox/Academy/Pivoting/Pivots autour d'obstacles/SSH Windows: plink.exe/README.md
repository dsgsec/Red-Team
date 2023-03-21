SSH pour Windows : plink.exe
==========================

* * * * *

[Plink](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), abréviation de PuTTY Link, est un outil SSH en ligne de commande Windows qui fait partie du package PuTTY une fois installé. Semblable à SSH, Plink peut également être utilisé pour créer des transferts de port dynamiques et des proxies SOCKS. Avant la chute de [2018](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview), Windows n'avait pas de client ssh natif inclus, les utilisateurs devaient donc installer leur posséder. L'outil de choix pour de nombreux administrateurs système qui avaient besoin de se connecter à d'autres hôtes était [PuTTY](https://www.putty.org/).

Imaginez que nous sommes sur un pentest et que nous accédons à une machine Windows. Nous énumérons rapidement l'hôte et sa posture de sécurité et déterminons qu'il est modérément verrouillé. Nous devons utiliser cet hôte comme point pivot, mais il est peu probable que nous puissions tirer nos propres outils sur l'hôte sans être exposés. Au lieu de cela, nous pouvons vivre de la terre et utiliser ce qui est déjà là. Si l'hôte est plus ancien et que PuTTY est présent (ou si nous pouvons trouver une copie sur un partage de fichiers), Plink peut être notre chemin vers la victoire. Nous pouvons l'utiliser pour créer notre pivot et potentiellement éviter la détection un peu plus longtemps.

Ce n'est qu'un scénario potentiel où Plink pourrait être bénéfique. Nous pourrions également utiliser Plink si nous utilisons un système Windows comme hôte d'attaque principal au lieu d'un système basé sur Linux.

* * * * *

Apprendre à connaître Plink
---------------------

Dans l'image ci-dessous, nous avons un hôte d'attaque basé sur Windows.

![](https://academy.hackthebox.com/storage/modules/158/66.png)

L'hôte d'attaque Windows démarre un processus plink.exe avec les arguments de ligne de commande ci-dessous pour démarrer une redirection de port dynamique sur le serveur Ubuntu. Cela démarre une session SSH entre l'hôte d'attaque Windows et le serveur Ubuntu, puis plink commence à écouter sur le port 9050.


#### Utilisation de Plink.exe

Utilisation de Plink.exe

```
plink -D 9050 ubuntu@10.129.15.50

```

Un autre outil basé sur Windows appelé [Proxifier](https://www.proxifier.com/) peut être utilisé pour démarrer un tunnel SOCKS via la session SSH que nous avons créée. Proxifier est un outil Windows qui crée un réseau tunnelisé pour les applications clientes de bureau et lui permet de fonctionner via un proxy SOCKS ou HTTPS et permet le chaînage de proxy. Il est possible de créer un profil où nous pouvons fournir la configuration de notre serveur SOCKS démarré par Plink sur le port 9050.

![](https://academy.hackthebox.com/storage/modules/158/reverse_shell_9.png)

Après avoir configuré le serveur SOCKS pour `127.0.0.1` et le port 9050, nous pouvons directement démarrer `mstsc.exe` pour démarrer une session RDP avec une cible Windows qui autorise les connexions RDP.

Remarque : nous pouvons essayer cette technique dans n'importe quelle section interactive de ce module à partir d'un hôte d'attaque personnel basé sur Windows. Une fois que vous avez terminé ce module à partir d'un hôte d'attaque basé sur Linux, n'hésitez pas à essayer de le parcourir à partir d'un hôte d'attaque personnel basé sur Windows. De plus, lors de la création de votre cible, nous vous demandons d'attendre 3 à 5 minutes jusqu'à ce que tout le laboratoire avec toutes les configurations soit configuré afin que la connexion à votre cible fonctionne parfaitement.