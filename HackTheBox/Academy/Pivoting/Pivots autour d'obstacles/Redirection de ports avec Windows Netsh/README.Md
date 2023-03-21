Transfert de port avec Windows Netsh
==================================

* * * * *

[Netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts) est un outil de ligne de commande Windows qui peut aider à la configuration réseau d'un système Windows particulier système. Voici quelques-unes des tâches liées au réseau pour lesquelles nous pouvons utiliser `Netsh` :

- `Trouver des itinéraires`
- `Affichage de la configuration du pare-feu`
- `Ajouter des procurations`
- `Créer des règles de redirection de port`

Prenons un exemple du scénario ci-dessous où notre hôte compromis est le poste de travail d'un administrateur informatique basé sur Windows 10 (`10.129.15.150`,` 172.16.5.25`). Gardez à l'esprit qu'il est possible, dans le cadre d'un engagement, que nous ayons accès au poste de travail d'un employé par le biais de méthodes telles que l'ingénierie sociale et le phishing. Cela nous permettrait de nous éloigner davantage du réseau dans lequel se trouve le poste de travail.

![](https://academy.hackthebox.com/storage/modules/158/88.png)

Nous pouvons utiliser `netsh.exe` pour transférer toutes les données reçues sur un port spécifique (disons 8080) vers un hôte distant sur un port distant. Cela peut être effectué à l'aide de la commande ci-dessous.

#### Utilisation de Netsh.exe pour transférer le port

Utilisation de Netsh.exe pour la redirection de port

```
C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25

```

#### Vérification de la redirection de port

Vérification de la redirection de port

```
C:\Windows\system32> netsh.exe interface portproxy show v4tov4

Écoutez sur ipv4 : Connectez-vous à ipv4 :

Port d'adresse Port d'adresse
---------- ---------- --------------- ----------
10.129.42.198 8080 172.16.5.25 3389

```

Après avoir configuré le `portproxy` sur notre hôte pivot basé sur Windows, nous essaierons de nous connecter au port 8080 de cet hôte à partir de notre hôte d'attaque à l'aide de xfreerdp. Une fois qu'une requête est envoyée par notre hôte d'attaque, l'hôte Windows acheminera notre trafic en fonction des paramètres de proxy configurés par netsh.exe.

#### Connexion à l'hôte interne via la redirection de port

![](https://academy.hackthebox.com/storage/modules/158/netsh_pivot.png)

* * * * *

Remarque : lors de la création de votre cible, nous vous demandons d'attendre 3 à 5 minutes jusqu'à ce que l'ensemble du laboratoire avec toutes les configurations soit configuré afin que la connexion à votre cible fonctionne parfaitement.