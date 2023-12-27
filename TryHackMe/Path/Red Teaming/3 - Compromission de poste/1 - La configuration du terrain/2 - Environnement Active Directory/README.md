Qu'est-ce que l'environnement Active Directory ( AD ) ?

![a8e30315a65f3451771a2e038864fdee](https://github.com/dsgsec/Red-Team/assets/82456829/d5a2c8db-79e5-42ab-8cd8-ea5d751f71a7)

Il s'agit d'un service d'annuaire basé sur Windows qui stocke et fournit des objets de données à l'environnement réseau interne. Il permet une gestion centralisée de l'authentification et de l'autorisation. L' AD contient des informations essentielles sur le réseau et l'environnement, notamment les utilisateurs, les ordinateurs, les imprimantes, etc. Par exemple, AD peut contenir des informations sur les utilisateurs telles que l'intitulé du poste, le numéro de téléphone, l'adresse, les mots de passe, les groupes, l'autorisation, etc.

![59664b98a3a0b01cf6b7e83e039ddb84](https://github.com/dsgsec/Red-Team/assets/82456829/2b92959b-7a7c-4603-935e-61d1b661c305)

Le diagramme est un exemple possible de la façon dont Active Directory peut être conçu. Le contrôleur AD est placé dans un sous-réseau pour les serveurs (indiqué ci-dessus comme réseau de serveurs), puis les clients AD se trouvent sur un réseau distinct où ils peuvent rejoindre le domaine et utiliser les services AD via le pare-feu.

Voici une liste de composants Active Directory avec lesquels nous devons être familiers :

-   Contrôleurs de domaine
-   Unités organisationnelles
-   Objets AD
-   Domaines AD
-   Forêt
-   Comptes de service AD : utilisateurs locaux intégrés, utilisateurs de domaine, comptes de service gérés
-   Administrateurs de domaine

Un contrôleur de domaine est un serveur Windows qui fournit des services Active Directory et contrôle l'ensemble du domaine. Il s'agit d'une forme de gestion centralisée des utilisateurs qui permet  le cryptage des données utilisateur ainsi que le contrôle de l'accès à un réseau, y compris les utilisateurs, les groupes, les politiques et les ordinateurs. Il permet également l'accès et le partage des ressources.  Ce sont toutes les raisons pour lesquelles les attaquants ciblent un contrôleur de domaine dans un domaine, car il contient de nombreuses informations de grande valeur.

![c982e300552d540f0fc456cc05be21cd](https://github.com/dsgsec/Red-Team/assets/82456829/40adc59d-c806-4c2b-83df-c2852aa6d469)

Les unités organisationnelles (OU) sont des conteneurs au sein du domaine AD avec une structure hiérarchique.

Les objets Active Directory  peuvent être un utilisateur unique ou un groupe, ou un composant matériel, tel qu'un ordinateur ou une imprimante. Chaque domaine contient une base de données contenant des informations sur l'identité des objets qui créent un environnement AD , notamment :

-   Utilisateurs : un principal de sécurité autorisé à s'authentifier auprès des machines du domaine
-   Ordinateurs - Un type spécial de comptes d'utilisateurs
-   GPO - Collections de stratégies appliquées à d'autres objets AD

Les domaines AD  sont un ensemble de composants Microsoft au sein d'un réseau AD . 

AD Forest  est un ensemble de domaines qui se font confiance.\

![bb4bec81a78f745e8cbc38f7879002dd](https://github.com/dsgsec/Red-Team/assets/82456829/c7e3aa85-7f16-401d-af14-c5b61ce82083)

Pour plus d'informations sur les bases d'Active Directory, nous vous suggérons d'essayer la salle TryHackMe suivante : [ Bases d'Active Directory ](https://tryhackme.com/room/winadbasics).

*\
*

Une fois l'accès initial obtenu, il est important de trouver un environnement AD dans un réseau d'entreprise, car l'environnement Active Directory fournit de nombreuses informations sur l'environnement aux utilisateurs joints.  En tant que membre de l'équipe rouge, nous en profitons en énumérant l' environnement AD et en accédant à  divers détails, qui peuvent ensuite être utilisés lors de la phase de mouvement latéral.
