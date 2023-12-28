Cette salle se concentre sur le dénombrement post-exploitation. En d'autres termes, nous supposons que nous avons réussi à obtenir une certaine forme d'accès à un système. De plus, nous avons peut-être procédé à une élévation de privilèges ; en d'autres termes, nous pourrions avoir des privilèges d'administrateur ou de root sur le système cible. Certaines des techniques et outils discutés dans cette salle fourniraient toujours des résultats utiles même avec un compte non privilégié, c'est-à-dire sans root ni administrateur.

Si vous êtes intéressé par l'élévation des privilèges, vous pouvez consulter la salle [Windows Privilege Escalation](https://tryhackme.com/room/windowsprivesc20) et la salle [Linux PrivEsc](https://tryhackme.com/room/linprivesc) . De plus, il existe deux scripts pratiques, [WinPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) et [LinPEAS ](https://grimbins.github.io/grimbins/linpeas/)respectivement pour l'élévation des privilèges MS Windows et Linux .

Notre objectif est de collecter plus d'informations qui nous aideront à obtenir un meilleur accès au réseau cible. Par exemple, nous pourrions trouver les informations de connexion pour accorder l'accès à un autre système. Nous nous concentrons sur les outils couramment disponibles sur les systèmes standards pour collecter plus d'informations sur la cible. Faisant partie du système, ces outils semblent inoffensifs et provoquent le moins de « bruit ».

Nous supposons que vous avez accès à une interface de ligne de commande sur la cible, comme `bash`sur un système Linux ou `cmd.exe`sur un système MS Windows. En commençant par un type de shell sur un système Linux , il est généralement facile de passer à un autre. De même, à partir de `cmd.exe`, vous pouvez passer à PowerShell si disponible. Nous venons d'émettre la commande `powershell.exe`pour démarrer la ligne de commande interactive PowerShell dans le terminal ci-dessous.

Terminal

```
user@TryHackMe$ Microsoft Windows [Version 10.0.17763.2928]
(c) 2018 Microsoft Corporation. All rights reserved.

strategos@RED-WIN-ENUM C:\Users\strategos>powershell.exe
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\Users\strategos>
```

Cette salle est organisée comme suit :

-   Objectif du dénombrement
-   Énumération Linux avec les outils couramment installés : système, utilisateurs, réseau et services en cours d'exécution
-   Énumération MS Windows avec outils intégrés : système, utilisateurs, réseau et services en cours d'exécution
-   Exemples d'outils supplémentaires : Ceinture de sécurité

Bien que cela ne soit pas strictement nécessaire, nous vous conseillons de compléter la salle [La configuration du terrain](https://tryhackme.com/room/thelayoftheland) avant de parcourir celle-ci.
