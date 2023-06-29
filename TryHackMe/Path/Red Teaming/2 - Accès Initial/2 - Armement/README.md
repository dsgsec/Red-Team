Intrduction
===========

Dans cette salle, nous discuterons des différentes techniques utilisées pour la militarisation.  

![126ff098c0efeeeb8ab694a09b3359b0](https://github.com/dsgsec/Red-Team/assets/82456829/6abd31bb-a69e-49de-a2f2-aa2496684f35)

Qu'est-ce que l'armement

La militarisation est la deuxième étape du modèle Cyber ​​Kill Chain. À ce stade, l'attaquant génère et développe son propre code malveillant à l'aide de charges utiles livrables telles que des documents Word, des PDF, etc. [ [1](https://www.lockheedmartin.com/en-us/capabilities/cyber/cyber-kill-chain.html) ]. L'étape de militarisation vise à utiliser l'arme malveillante pour exploiter la machine cible et obtenir un accès initial.

La plupart des organisations ont un système d'exploitation Windows en cours d'exécution, ce qui sera une cible probable. La politique d'environnement d'une organisation bloque souvent le téléchargement et l'exécution  de fichiers .exe pour éviter les violations de sécurité. Par conséquent, les équipes rouges s'appuient sur la création de charges utiles personnalisées envoyées via divers canaux tels que les campagnes de phishing, l'ingénierie sociale, l'exploitation de navigateurs ou de logiciels, l'USB ou les méthodes Web. 

Le graphique suivant est un exemple de militarisation, où un document PDF ou Microsoft Office personnalisé est utilisé pour fournir une charge utile malveillante. La charge utile personnalisée est configurée pour se reconnecter à l'environnement de commande et de contrôle de l'infrastructure de l'équipe rouge.

![734a353799fc9f3cd05bb7421ceedd00](https://github.com/dsgsec/Red-Team/assets/82456829/bf09ff0b-3f1d-4a7e-a3e8-bfaa081238bd)

Pour plus d'informations sur les boîtes à outils de l'équipe rouge, veuillez visiter ce qui suit : un [référentiel GitHub](https://github.com/infosecn1nja/Red-Teaming-Toolkit#Payload%20Development) qui a tout, y compris l'accès initial, le développement de la charge utile, les méthodes de livraison, etc.

La plupart des organisations bloquent ou surveillent l'exécution des  fichiers .exe dans leur environnement contrôlé. Pour cette raison, les équipes rouges s'appuient sur l'exécution de charges utiles à l'aide d'autres techniques, telles que les technologies de script Windows intégrées. Par conséquent, cette tâche se concentre sur diverses techniques de script populaires et efficaces, notamment : 

-   L'hôte de script Windows (WSH)
-   Une application HTML ( HTA )
-   Applications Visual Basic (VBA)
-   PowerShell (PSH)
