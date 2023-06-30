Commandement et contrôle - (C2 ou C&C)
========================================

Cette tâche présente le concept de base des cadres de commandement et de contrôle ( C2 ) utilisés dans les opérations de l'équipe rouge.

![9671adc6cb778fa7b151921f753e2f96](https://github.com/dsgsec/Red-Team/assets/82456829/74bd4435-734d-4c00-b6d5-2e0f97e1b868)

Qu'est-ce que le commandement et le contrôle (C2) ?

Les frameworks C2 sont des frameworks de post-exploitation qui permettent aux red teamers de collaborer et de contrôler les machines compromises. Le C2 est considéré comme l'un des outils les plus importants pour les équipes rouges lors d'opérations cybernétiques offensives. Les frameworks C2 fournissent des approches simples et rapides pour :

-   Générer diverses charges utiles malveillantes
-   Énumérer la machine/les réseaux compromis
-   Effectuer une élévation et un pivotement des privilèges
-   Mouvement latéral 
-   Et plein d'autres

Certains frameworks C2 populaires que nous allons brièvement souligner sont Cobalt Strike, PowerShell Empire, Metasploit. La plupart de ces cadres visent à prendre en charge un environnement pratique pour partager et communiquer  entre les opérations de l'équipe rouge une fois l'accès initial obtenu à un système.

CobaltStrike

Cobalt Strike est un cadre commercial qui se concentre sur les simulations d'adversaires et les opérations de l'équipe rouge. Il s'agit d'une combinaison d'outils d'accès à distance, de capacités de post-exploitation et d'un système de rapport unique. Il fournit à un agent des techniques avancées pour établir des communications secrètes et effectuer diverses opérations, notamment l'enregistrement de frappe, le téléchargement et le téléchargement de fichiers, le déploiement de VPN, les techniques d'escalade de privilèges, le mimikatz, l'analyse des ports et les mouvements latéraux les plus avancés.

Empire PowerShell

PowerShell Empire est un framework open source qui aide les opérateurs de l'équipe rouge et les testeurs de stylo à collaborer sur plusieurs serveurs à l'aide de clés et de mots de passe partagés. Il s'agit d'un framework d'exploitation basé sur des agents PowerShell et Python. PowerShell Empire se concentre sur le côté client et la post-exploitation de l'environnement Windows et Active Directory. Si vous souhaitez en savoir plus sur PowerShell Empire, nous vous suggérons d'essayer cette salle : [Empire](https://tryhackme.com/room/rppsempire) .

Metasploit 

Metasploit est un cadre d'exploitation largement utilisé qui offre diverses techniques et outils pour effectuer facilement du piratage. Il s'agit d'un framework open-source et est considéré comme l'un des principaux outils pour les opérations de pentesting et d'équipe rouge. Metasploit est l'un des outils que nous utilisons dans cette salle pour générer une charge utile pour notre étape de militarisation. Si vous souhaitez en savoir plus sur le framework Metasploit, nous vous suggérons de consulter le [module Metasploit](https://tryhackme.com/module/metasploit) .

La plupart des cadres C2 utilisent les techniques mentionnées dans cette salle comme préparation à l'étape d'accès initial. Pour plus de détails sur le framework C2, nous vous invitons à consulter la salle [Intro to C2](https://tryhackme.com/room/introtoc2) .
