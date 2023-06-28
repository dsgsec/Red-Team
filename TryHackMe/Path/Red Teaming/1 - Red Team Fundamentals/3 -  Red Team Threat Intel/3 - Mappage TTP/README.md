Mappage TTP
===========

Le mappage TTP  est utilisé par la cellule rouge pour mapper les TTP collectés par les adversaires sur une chaîne de cyber-attaque standard. Le mappage des TTP sur une chaîne de mise à mort aide l'équipe rouge à planifier un engagement pour imiter un adversaire.

Pour commencer le processus de cartographie des TTP, un adversaire doit être sélectionné comme cible. Un adversaire peut être choisi en fonction de,

1.  Industrie cible
2.  Vecteurs d'attaque employés
3.  Pays d'origine
4.  Autres facteurs

Comme exemple pour cette tâche, nous avons décidé d'utiliser  [APT 39](https://attack.mitre.org/groups/G0087/) , un groupe de cyber-espionnage dirigé par le ministère iranien, connu pour cibler une grande variété d'industries.

Nous utiliserons la cyber kill chain de Lockheed Martin comme cyber kill chain standard pour cartographier les TTP.

![f53c9707927cbfec71f813810387dd3c](https://github.com/dsgsec/Red-Team/assets/82456829/bcbda8f0-8e02-43e3-a2df-636ef162fb4d)

Le premier cyber-cadre à partir duquel nous collecterons les TTP est  [MITRE ATT&CK](https://attack.mitre.org/) . Si vous n'êtes pas familier avec MITRE ATT&CK, il fournit des identifiants et des descriptions de TTP classés. Pour plus d'informations sur MITRE et sur l'utilisation d'ATT&CK, consultez la  [salle MITRE](https://tryhackme.com/room/mitre) .

ATT&CK fournit un résumé de base des TTP collectés par un groupe. Nous pouvons utiliser  [ATT&CK Navigator](https://mitre-attack.github.io/attack-navigator/)  pour nous aider à visualiser chaque TTP et à catégoriser sa place dans la chaîne de destruction. Navigator visualise la chaîne ATT&CK avec les TTP désignés des adversaires mis en évidence dans la sous-section correspondante.

Pour utiliser le navigateur ATT&CK : accédez à la page de résumé des groupes, à côté de  *"Techniques utilisées",*  accédez à  *"Couches du navigateur ATT&CK",*  dans la liste déroulante, accédez à  *"afficher". * Une couche ATT&CK Navigator doit s'être ouverte avec les TTP du groupe sélectionné mis en surbrillance dans un nouvel onglet.

En passant par la couche Navigator, nous pouvons attribuer divers TTP que nous souhaitons utiliser pendant l'engagement. Vous trouverez ci-dessous une chaîne de mise à mort compilée avec des TTP mappés pour  APT39 .


1.  Reconnaissance:
    -   Aucun TTP identifié, utilisez la méthodologie de l'équipe interne
2.  Armement :
    -   Interprète de commandes et de scripts
        -   PowerShell
        -   Python
        -   VBA
    -   Pièces jointes malveillantes exécutées par l'utilisateur
3.  Livraison:
    -   Exploiter les applications publiques
    -   Hameçonnage
4.  Exploitation:
    -   Modification du registre
    -   Tâches planifiées
    -   Enregistrement de frappe
    -   Vidage des informations d'identification
5.  Installation:
    -   Transfert d'outil d'entrée
    -   Utilisation du proxy
6.  Commande et contrôle :
    -   Protocoles Web (HTTP/HTTPS)
    -   DNS
7.  Actions sur les objectifs
    -   Exfiltration sur C2
    
![ce9424c66a674856de4c97613cbde8c4](https://github.com/dsgsec/Red-Team/assets/82456829/9813e052-8d77-43b6-b4d6-8a2769356cef)
