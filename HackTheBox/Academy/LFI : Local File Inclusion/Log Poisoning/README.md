Log Poisoning
=============

Nous avons vu dans les sections précédentes que si nous incluons un fichier contenant du code PHP, il sera exécuté, tant que la fonction vulnérable dispose des privilèges `Execute` . Les attaques dont nous parlerons dans cette section reposent toutes sur le même concept : écrire du code PHP dans un champ que nous contrôlons et qui est enregistré dans un fichier journal (c'est-à-dire `poison`/`contaminate` le fichier journal), puis inclure ce fichier journal pour exécuter le code PHP. Pour que cette attaque fonctionne, l'application Web PHP doit avoir des privilèges de lecture sur les fichiers journalisés, qui varient d'un serveur à l'autre.

Comme c'était le cas dans la section précédente, toutes les fonctions suivantes dotées des privilèges `Exécuter` devraient être vulnérables à ces attaques :

| Fonction | Lire le contenu | Exécuter | URL distante |
| --- | :-: | :-: | :-: |
| PHP | | | |
| `include()`/`include_once()` | ✅ | ✅ | ✅ |
| `require()`/`require_once()` | ✅ | ✅ | ❌ |
| NodeJS | | | |
| `res.render()` | ✅ | ✅ | ❌ |
| Java | | | |
| `importer` | ✅ | ✅ | ✅ |
| .NET | | | |
| `inclure` | ✅ | ✅ | ✅ |

* * * * *

Empoisonnement de session PHP
---------------------

La plupart des applications Web PHP utilisent des cookies `PHPSESSID` , qui peuvent contenir des données spécifiques relatives à l'utilisateur sur le back-end, afin que l'application Web puisse suivre les détails de l'utilisateur via ses cookies. Ces détails sont stockés dans `session` fichiers sur le back-end, et enregistrés dans `/var/lib/php/sessions/` sous Linux et dans `C:\Windows\Temp\` sous Windows. Le nom du fichier qui contient les données de nos utilisateurs correspond au nom de notre cookie `PHPSESSID` avec le préfixe `sess_` . Par exemple, si le cookie `PHPSESSID` est défini sur `el4ukv0kqbvoirg7nkp4dncpk3`, son emplacement sur le disque sera `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3`.

La première chose que nous devons faire dans une attaque d'empoisonnement de session PHP est d'examiner notre fichier de session PHPSESSID et de voir s'il contient des données que nous pouvons contrôler et empoisonner. Alors, commençons par vérifier si nous avons un cookie `PHPSESSID` défini pour notre session : ![image](https://academy.hackthebox.com/storage/modules/23/rfi_cookies_storage.png)

Comme nous pouvons le voir, la valeur de notre cookie `PHPSESSID` est `nhhv8i0o6ua4g88bkdl9u1fdsd`, elle doit donc être stockée dans `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd`. Essayons d'inclure ce fichier de session via la vulnérabilité LFI et d'afficher son contenu :

![](https://academy.hackthebox.com/storage/modules/23/rfi_session_include.png)

Remarque : Comme vous pouvez facilement le deviner, la valeur du cookie diffère d'une session à l'autre. Vous devez donc utiliser la valeur du cookie que vous trouvez dans votre propre session pour effectuer la même attaque.

Nous pouvons voir que le fichier de session contient deux valeurs : `page`, qui affiche la page de langue sélectionnée, et `preference`, qui affiche la langue sélectionnée. La valeur `preference` n'est pas sous notre contrôle, car nous ne l'avons spécifiée nulle part et doit être spécifiée automatiquement. Cependant, la valeur `page` est sous notre contrôle, car nous pouvons la contrôler via le paramètre `?language=` .

Essayons de définir la valeur de `page` une valeur personnalisée (par exemple `paramètre de langue`) et voyons si cela change dans le fichier de session. Nous pouvons le faire en visitant simplement la page avec `?language=session_poisoning` spécifié, comme suit :

Code : URL

```
http://<SERVER_IP> :<PORT>/index.php?language=session_poisoning

```

Maintenant, incluons à nouveau le fichier de session pour en examiner le contenu :

![](https://academy.hackthebox.com/storage/modules/23/lfi_poisoned_sessid.png)

Cette fois, le fichier de session contient `session_poisoning` au lieu de `es.php`, ce qui confirme notre capacité à contrôler la valeur de `page` dans le fichier de session. Notre prochaine étape consiste à effectuer l'étape d'"empoisonnement" en écrivant du code PHP dans le fichier de session. Nous pouvons écrire un shell Web PHP de base en remplaçant le paramètre `?language=` par un shell Web codé en URL, comme suit :

Code : URL

```
http://<SERVER_IP> :<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E

```

Enfin, nous pouvons inclure le fichier de session et utiliser `&cmd=id` pour exécuter une commande :

![](https://academy.hackthebox.com/storage/modules/23/rfi_session_id.png)

Remarque : Pour exécuter une autre commande, le fichier de session doit à nouveau être empoisonné avec le shell Web, car il est remplacé par `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` après notre dernière inclusion. Idéalement, nous utiliserions le shell Web empoisonné pour écrire un shell Web permanent dans le répertoire Web, ou enverrions un shell inversé pour une interaction plus facile.

* * * * *

Empoisonnement du journal du serveur
--------------------

 `Apache` et `Nginx` conservent divers fichiers journaux, tels que `access.log` et `error.log`. Le fichier `access.log` contient diverses informations sur toutes les requêtes adressées au serveur, y compris l'en-tête `User-Agent` de chaque requête. Comme nous pouvons contrôler l'en-tête `User-Agent` dans nos requêtes, nous pouvons l'utiliser pour empoisonner les journaux du serveur comme nous l'avons fait ci-dessus.

Une fois empoisonné, nous devons inclure les journaux via la vulnérabilité LFI, et pour cela, nous devons avoir un accès en lecture sur les journaux. Les journaux `Nginx` sont lisibles par défaut par les utilisateurs à faibles privilèges