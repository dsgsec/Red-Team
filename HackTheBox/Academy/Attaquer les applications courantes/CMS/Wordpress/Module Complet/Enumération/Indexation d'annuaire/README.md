Indexation d'annuaire
=====================

Les plugins actifs ne devraient pas être notre seul domaine d'intérêt lors de l'évaluation d'un site Web WordPress. Même si un plugin est désactivé, il peut toujours être accessible et nous pouvons donc accéder à ses scripts et fonctions associés. La désactivation d'un plugin vulnérable n'améliore pas la sécurité du site WordPress. Il est recommandé de supprimer ou de maintenir à jour tous les plugins inutilisés.

L'exemple suivant montre un plugin désactivé.

![image](https://academy.hackthebox.com/storage/modules/17/plugin-deactivated3.png)

Si nous parcourons le répertoire des plugins, nous pouvons voir que nous avons toujours accès au `Mail Masta`plugin.

![](https://academy.hackthebox.com/storage/modules/17/plugin-mailmasta2.png)

Nous pouvons également afficher la liste des répertoires en utilisant cURL et convertir la sortie HTML dans un format lisible en utilisant `html2text`.

```
dsgsec@htb[/htb]$ curl -s -X GET http://blog.inlanefreight.com/wp-content/plugins/mail-masta/ | html2text

****** Index of /wp-content/plugins/mail-masta ******
[[ICO]]       Name                 Last_modified    Size Description
===========================================================================
[[PARENTDIR]] Parent_Directory                         -
[[DIR]]       amazon_api/          2020-05-13 18:01    -
[[DIR]]       inc/                 2020-05-13 18:01    -
[[DIR]]       lib/                 2020-05-13 18:01    -
[[   ]]       plugin-interface.php 2020-05-13 18:01  88K
[[TXT]]       readme.txt           2020-05-13 18:01 2.2K
===========================================================================
     Apache/2.4.29 (Ubuntu) Server at blog.inlanefreight.com Port 80

```

Ce type d'accès est appelé `Directory Indexing`. Il nous permet de parcourir le dossier et d'accéder aux fichiers pouvant contenir des informations sensibles ou du code vulnérable. Il est recommandé de désactiver l'indexation des répertoires sur les serveurs Web afin qu'un attaquant potentiel ne puisse pas accéder directement à des fichiers ou des dossiers autres que ceux nécessaires au bon fonctionnement du site Web.
