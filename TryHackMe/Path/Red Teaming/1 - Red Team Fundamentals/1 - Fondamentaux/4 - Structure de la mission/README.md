Structure de la mission
========================

Une fonction essentielle de l'équipe rouge est l'émulation de l'adversaire. Bien qu'il ne soit pas obligatoire, il est couramment utilisé pour évaluer ce qu'un véritable adversaire ferait dans un environnement en utilisant ses outils et ses méthodologies. L'équipe rouge peut utiliser diverses chaînes de destruction cybernétique pour résumer et évaluer les étapes et les procédures d'un engagement.

L'équipe bleue utilise couramment des chaînes de destruction cybernétique pour cartographier les comportements et décomposer le mouvement d'un adversaire. L'équipe rouge peut adapter cette idée pour mapper les TTP de l'adversaire ( tactiques ,  techniques et  procédures ) aux composants d'un engagement.

De nombreux organismes de réglementation et de normalisation ont publié leur cyber kill chain. Chaque chaîne de mise à mort suit à peu près la même structure, certaines allant plus en profondeur ou définissant les objectifs différemment. Vous trouverez ci-dessous une petite liste de chaînes de cyber-kill standard.

-   [Lockheed Martin Cyber ​​Kill Chaîne](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html)
-   [Chaîne de mise à mort unifiée](https://unifiedkillchain.com/)
-   [Chaîne Cyber ​​Kill de Varonis](https://www.varonis.com/blog/cyber-kill-chain/)
-   [Cycle d'attaque Active Directory](https://github.com/infosecn1nja/AD-Attack-Defense)
-   [Cadre MITRE ATT&CK](https://attack.mitre.org/)

Dans cette salle, nous ferons couramment référence à la "Lockheed Martin Cyber ​​Kill Chain". Il s'agit d'une chaîne de mise à mort plus standardisée que les autres et est très couramment utilisée par les équipes rouges et bleues.

La chaîne de destruction de Lockheed Martin se concentre sur un périmètre ou une brèche externe. Contrairement à d'autres chaînes de mise à mort, il ne fournit pas une analyse approfondie des mouvements internes. Vous pouvez considérer cette chaîne de mise à mort comme un résumé de tous les comportements et opérations présents.

![33e4c2dc2ab851b11654ae61953a7df1](https://github.com/dsgsec/Red-Team/assets/82456829/cf851d8f-ec0b-4409-b4c2-7f08f4d78824)

Les composants de la chaîne de mise à mort sont décomposés dans le tableau ci-dessous.

| Technique | But | Exemples |
| --- | --- | --- |
| Reconnaissance | Obtenir des informations sur la cible | Collecte des e-mails, OSINT |
| Armement | Combinez l'objectif avec un exploit. Entraîne généralement une charge utile livrable. | Exploitation avec porte dérobée, document de bureau malveillant |
| Livraison | Comment la fonction militarisée sera-t-elle transmise à la cible ? | E-mail, Web, USB |
| Exploitation | Exploiter le système de la cible pour exécuter du code | MS17-010, Zero-Logon, etc. |
| Installation | Installer des logiciels malveillants ou d'autres outils | Mimikatz, Rubeus, etc. |
| Commandement et contrôle | Contrôlez l'actif compromis à partir d'un contrôleur central à distance | Empire, Frappe de Cobalt, etc. |
| Actions sur les objectifs | Tous les objectifs finaux : ransomware, exfiltration de données, etc. | Conti, LockBit2.0, etc. |
