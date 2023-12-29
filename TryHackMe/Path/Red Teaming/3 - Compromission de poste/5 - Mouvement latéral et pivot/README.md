![e3b49368f7e5dc84be730e9bc334f5dd](https://github.com/dsgsec/Red-Team/assets/82456829/c6ef0dce-c871-4cc0-8e1d-7970c856e216)

Dans cette salle, nous examinerons le mouvement latéral, un groupe de techniques utilisées par les attaquants pour se déplacer sur le réseau tout en créant le moins d'alertes possible. Nous découvrirons plusieurs techniques courantes utilisées dans la nature à cette fin et les outils impliqués .

Il est recommandé de passer par les salles [Breaching AD](https://tryhackme.com/room/breachingad) et [Enumerating AD](https://tryhackme.com/room/adenumeration) avant celle-ci.

Objectifs d'apprentissage
-------------------------

-   Familiarisez-vous avec les techniques de mouvements latéraux utilisées par les attaquants.
-   Découvrez comment utiliser du matériel d'authentification alternatif pour vous déplacer latéralement.
-   Découvrez différentes méthodes pour utiliser des hôtes compromis comme pivots.

Connexion au réseau
-------------------

Boîte d'attaque

Si vous utilisez l'AttackBox basée sur le Web, vous serez automatiquement connecté au réseau si vous démarrez l'AttackBox à partir de la page de la salle. Vous pouvez le vérifier en exécutant la commande ping sur l'adresse IP de l'hôte THMDC.za.tryhackme.com. Nous devons cependant encore configurer le DNS. Les réseaux Windows utilisent le service de noms de domaine (DNS) pour résoudre les noms d'hôte en adresses IP. Tout au long de ce réseau, le DNS sera utilisé pour les tâches. Vous devrez configurer DNS sur l'hôte sur lequel vous exécutez la connexion VPN. Afin de configurer notre DNS, exécutez la commande suivante :

Terminal

```
[thm@thm]$ systemd-resolve --interface lateralmovement --set-dns $THMDCIP --set-domain za.tryhackme.com
```

N'oubliez pas de remplacer $THMDCIP par l'IP de THMDC dans votre schéma réseau.

Vous pouvez tester que DNS fonctionne en exécutant :

`nslookup thmdc.za.tryhackme.com`

Cela devrait résoudre l'adresse IP de votre contrôleur de domaine.

Remarque : le DNS peut être réinitialisé sur l'AttackBox environ toutes les 3 heures. Si cela se produit, vous devrez redémarrer le service résolu par systemd. Si votre AttackBox se termine et que vous continuez avec la salle plus tard, vous devrez refaire toutes les étapes DNS.

Vous devez également prendre le temps de noter votre IP VPN. À l'aide `ifconfig`de ou `ip a`, notez l'adresse IP de la carte réseau à mouvement latéral . Il s'agit de votre adresse IP et de l'interface associée que vous devez utiliser lors de l'exécution des attaques dans les tâches.

Autres hôtes

Si vous comptez utiliser votre propre machine d'attaque, un fichier de configuration OpenVPN aura été généré pour vous une fois que vous aurez rejoint la salle. Accédez à votre page [d'accès](https://tryhackme.com/access) . Sélectionnez `Lateralmovementandpivoting`parmi les serveurs VPN (sous l'onglet réseau) et téléchargez votre fichier de configuration.

![6830c917f28884eb07eb155be85caedb](https://github.com/dsgsec/Red-Team/assets/82456829/e9363706-8f4a-4c87-8107-eafcd07cb6e5)

Utilisez un client OpenVPN pour vous connecter. Cet exemple est affiché sur la machine [Linux](https://tryhackme.com/access#pills-linux) .

Terminal

```
[thm@thm]$ sudo openvpn user-lateralmovementandpivoting.ovpn
Fri Mar 11 15:06:20 2022 OpenVPN 2.4.9 x86_64-redhat-linux-gnu [SSL (OpenSSL)] [LZO] [LZ4] [EPOLL] [PKCS11] [MH/PKTINFO] [AEAD] built on Apr 19 2020
Fri Mar 11 15:06:20 2022 library versions: OpenSSL 1.1.1g FIPS  21 Apr 2020, LZO 2.08
[....]
Fri Mar 11 15:06:22 2022 /sbin/ip link set dev lateralmovement up mtu 1500
Fri Mar 11 15:06:22 2022 /sbin/ip addr add dev lateralmovement 10.50.2.3/24 broadcast 10.50.2.255
Fri Mar 11 15:06:22 2022 /sbin/ip route add 10.200.4.0/24 metric 1000 via 10.50.2.1
Fri Mar 11 15:06:22 2022 WARNING: this configuration may cache passwords in memory -- use the auth-nocache option to prevent this
Fri Mar 11 15:06:22 2022 Initialization Sequence Completed
```

Le message « Séquence d'initialisation terminée » vous indique que vous êtes désormais connecté au réseau. Revenez à votre page d'accès. Vous pouvez vérifier que vous êtes connecté en regardant sur votre page d'accès. Actualisez la page et vous devriez voir une coche verte à côté de Connecté. Il vous montrera également votre adresse IP interne.

![87cba97fafce7c5fddc089c3c89ddff2](https://github.com/dsgsec/Red-Team/assets/82456829/8264815f-ae5e-4496-bfde-7712efc95e6b)

Remarque : Vous devez toujours configurer le DNS de la même manière que ce qui a été indiqué ci-dessus. Il est important de noter que bien qu'il ne soit pas utilisé, le contrôleur de domaine enregistre les requêtes DNS. Si vous utilisez votre machine, ces journaux peuvent inclure le nom d'hôte de votre appareil.

Kali

Si vous utilisez une machine virtuelle Kali, Network Manager est probablement utilisé comme gestionnaire DNS. Vous pouvez utiliser le menu GUI pour configurer DNS :

-   Gestionnaire de réseau -> Configuration réseau avancée -> Votre connexion -> Paramètres IPv4
-   Définissez ici votre IP DNS sur l'IP de THMDC dans le schéma de réseau ci-dessus

-   Ajoutez un autre DNS tel que 1.1.1.1 ou similaire pour vous assurer que vous avez toujours accès à Internet
-   Exécutez `sudo systemctl restart NetworkManager`et testez votre DNS de la même manière que les étapes ci-dessus.

Remarque : lors de la configuration de votre DNS de cette manière, la `nslookup`commande ne fonctionnera pas comme prévu. Pour tester si vous avez correctement configuré votre DNS, accédez simplement à <http://distributor.za.tryhackme.com/creds> . Si vous voyez le site Web, vous êtes prêt pour le reste de la salle.

Demander vos informations d'identification
------------------------------------------

Pour simuler une violation AD, vous recevrez votre premier ensemble d'informations d'identification AD. Une fois la configuration de votre réseau terminée, sur votre Attack Box, accédez à <http://distributor.za.tryhackme.com/creds> pour demander votre paire d'informations d'identification. Cliquez sur le bouton « Obtenir les informations d'identification » pour recevoir votre paire d'informations d'identification qui peut être utilisée pour l'accès initial.

Cette paire d'informations d'identification vous fournira un accès SSH à THMJMP2.za.tryhackme.com. THMJMP2 peut être considéré comme un hôte de saut dans cet environnement, simulant un point d'ancrage que vous avez pris. 

Pour l'accès SSH, vous pouvez utiliser la commande suivante :

`ssh za\\<AD Username>@thmjmp2.za.tryhackme.com`

Une note sur les coques inversées
---------------------------------

Si vous utilisez l'AttackBox et avez déjà rejoint d'autres salles réseau, assurez-vous de sélectionner l'adresse IP attribuée à l'interface du tunnel faisant face au `lateralmovementandpivoting`réseau comme votre ATTACKER_IP, sinon vos shells/connexions inversées ne fonctionneront pas correctement. Pour votre commodité, l'interface attachée à ce réseau s'appelle `lateralmovement`, vous devriez donc pouvoir obtenir la bonne adresse IP en exécutant`ip add show lateralmovement` :

![b6b6f6f3ef517c092e52c4bca4941129](https://github.com/dsgsec/Red-Team/assets/82456829/c5751a61-d247-4a27-814a-fd1e0cce1624)

Cela sera utile chaque fois que vous devrez établir une connexion inverse vers la machine de votre attaquant dans toute la pièce.
