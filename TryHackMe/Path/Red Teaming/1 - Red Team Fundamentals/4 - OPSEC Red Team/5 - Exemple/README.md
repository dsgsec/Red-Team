Exemple
========

Dans cette tâche, nous appliquons les cinq éléments du processus OPSEC en nous concentrant sur différents exemples d'informations critiques liées aux tâches de l'équipe rouge. Nous suivrons les étapes suivantes :

1.  Identifier les informations critiques
2.  Analyser les menaces
3.  Analyser les vulnérabilités
4.  Évaluer le risque
5.  Appliquer les contre-mesures appropriées

### Programmes/OS/VM utilisés par l'équipe rouge

-   Informations critiques : nous parlons ensemble des programmes, du système d'exploitation ( OS ) et de la machine virtuelle (VM).
-   Analyse des menaces : L'équipe bleue recherche toute activité malveillante ou anormale sur le réseau. Selon le service auquel nous nous connectons, il est possible que le nom et la version du programme que nous utilisons, ainsi que la version du système d'exploitation et le nom d'hôte de la machine virtuelle soient enregistrés.
-   Analyse de vulnérabilité : si le système d'exploitation choisi pour l'activité donnée est trop unique, il peut être plus facile de relier les activités à votre opération. Il en va de même pour les machines virtuelles avec des noms d'hôte qui se démarquent. Par exemple, sur un réseau d'ordinateurs portables et d'ordinateurs de bureau physiques, si un nouvel hôte se joint au nom d'hôte`kali2021vm` , il devrait être facile à repérer par l'équipe bleue. De même, si vous utilisez divers scanners de sécurité ou, par exemple, vous n'utilisez pas d'agent utilisateur commun pour les activités Web.
-   Évaluation des risques : Le risque dépend principalement des services auxquels nous nous connectons. Par exemple, si nous démarrons une connexion VPN, le serveur VPN enregistrera de nombreuses informations à notre sujet. Il en va de même pour les autres services auxquels nous pourrions nous connecter.
-   Contre-mesures : si le système d'exploitation que nous utilisons est peu courant, cela vaudrait la peine d'apporter les modifications nécessaires pour camoufler notre système d'exploitation en un système différent. Pour les machines virtuelles et les hôtes physiques, il vaut la peine de changer les noms d'hôte en quelque chose de discret ou cohérent avec la convention de dénomination du client, car vous ne voulez pas qu'un nom d'hôte apparaisse `AttackBox`dans les journaux du serveur DHCP . En ce qui concerne les programmes et les outils, il vaut la peine de connaître les signatures que chaque outil laisse sur les journaux du serveur.

Exemple : la figure ci-dessous montre l'agent utilisateur qui sera enregistré par le serveur Web distant lors de l'exécution d'analyses Nmap avec l' `-sC`option lorsque Nmap sonde le serveur Web. Si un agent utilisateur HTTP n'est pas défini au moment de l'exécution du script Nmap donné, les journaux sur le système cible peuvent enregistrer un agent utilisateur contenant`Nmap Scripting Engine` . Cela peut être atténué en utilisant l'option `--script-args http.useragent="CUSTOM_AGENT"`.

![685031520972adc1c1a02ceac499f62c](https://github.com/dsgsec/Red-Team/assets/82456829/4af1982e-9cf8-4c35-a9af-6d20836fb290)
