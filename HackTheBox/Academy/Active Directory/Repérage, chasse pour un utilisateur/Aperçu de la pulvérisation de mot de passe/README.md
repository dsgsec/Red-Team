Aperçu de la pulvérisation de mot de passe
==========================================

* * * * *

La pulvérisation de mots de passe peut entraîner l'accès aux systèmes et potentiellement prendre pied sur un réseau cible. L'attaque consiste à tenter de se connecter à un service exposé à l'aide d'un mot de passe commun et d'une liste plus longue de noms d'utilisateur ou d'adresses e-mail. Les noms d'utilisateur et les e-mails peuvent avoir été recueillis lors de la phase OSINT du test d'intrusion ou de nos premières tentatives d'énumération. N'oubliez pas qu'un test d'intrusion n'est pas statique, mais nous parcourons constamment plusieurs techniques et répétons les processus à mesure que nous découvrons de nouvelles données. Souvent, nous travaillerons en équipe ou exécuterons plusieurs TTP à la fois pour utiliser notre temps efficacement. Au fur et à mesure que nous progressons dans notre carrière, nous constaterons que bon nombre de nos tâches telles que la numérisation, la tentative de déchiffrage de hachages et d'autres prennent un peu de temps. Nous devons nous assurer que nous utilisons notre temps de manière efficace et créative, car la plupart des évaluations sont limitées dans le temps. Ainsi, pendant que nos tentatives d'empoisonnement sont en cours, nous pouvons également utiliser les informations dont nous disposons pour tenter d'accéder via la pulvérisation de mots de passe. Voyons maintenant certaines des considérations relatives à la pulvérisation de mots de passe et comment établir notre liste cible à partir des informations dont nous disposons.

* * * * *

L'heure du conte
----------------

La pulvérisation de mots de passe peut être un moyen très efficace de prendre pied en interne. Il y a plusieurs fois que cette technique m'a aidé à prendre pied lors de mes évaluations. Gardez à l'esprit que ces exemples proviennent d'évaluations non évasives de "boîte grise" où j'avais un accès au réseau interne avec une machine virtuelle Linux et une liste de plages d'adresses IP dans le champ d'application et rien d'autre.

### Scénario 1

Dans ce premier exemple, j'ai effectué toutes mes vérifications standard et je n'ai rien trouvé d'utile comme une session SMB NULL ou une liaison anonyme LDAP qui pourrait me permettre de récupérer une liste d'utilisateurs valides. J'ai donc décidé d'utiliser l' `Kerbrute`outil pour créer une liste de noms d'utilisateurs cibles en énumérant les utilisateurs de domaine valides (une technique que nous aborderons plus loin dans cette section). Pour créer cette liste, j'ai pris la `jsmith.txt`liste des noms d'utilisateur du référentiel GitHub [des noms d'utilisateur statistiquement probables](https://github.com/insidetrust/statistically-likely-usernames) et l'ai combinée avec les résultats que j'ai obtenus en grattant LinkedIn. Avec cette liste combinée en main, j'ai énuméré les utilisateurs valides avec `Kerbrute`, puis j'ai utilisé le même outil pour pulvériser le mot de passe avec le mot de passe commun`Welcome1`. J'ai obtenu deux résultats avec ce mot de passe pour les utilisateurs à très faibles privilèges, mais cela m'a donné suffisamment d'accès au domaine pour exécuter BloodHound et éventuellement identifier les chemins d'attaque qui ont conduit à la compromission du domaine.

### Scénario 2

Lors de la deuxième évaluation, j'ai été confronté à une configuration similaire, mais l'énumération des utilisateurs de domaine valides avec des listes de noms d'utilisateur communes et les résultats de LinkedIn n'ont donné aucun résultat. Je me suis tourné vers Google et j'ai recherché des fichiers PDF publiés par l'organisation. Ma recherche a généré de nombreux résultats, et j'ai confirmé dans les propriétés du document de 4 d'entre eux que la structure interne du nom d'utilisateur était au format `F9L8`, des GUID générés aléatoirement en utilisant uniquement des lettres majuscules et des chiffres ( `A-Z and 0-9`). Ces informations ont été publiées avec le document sur le `Author`terrain et montrent l'importance de nettoyer les métadonnées du document avant de publier quoi que ce soit en ligne. À partir de là, un court script Bash pourrait être utilisé pour générer 16 079 616 combinaisons de noms d'utilisateur possibles.

Code : bash

```

#!/bin/bash

for x in {{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}
    do echo $x;
done

```

J'ai ensuite utilisé la liste de noms d'utilisateur générée `Kerbrute`pour énumérer chaque compte d'utilisateur du domaine. Cette tentative de rendre plus difficile l'énumération des noms d'utilisateur m'a permis d'énumérer chaque compte du domaine en raison du GUID prévisible utilisé combiné aux métadonnées PDF que j'ai pu localiser et a grandement facilité l'attaque. En règle générale, je ne peux identifier que 40 à 60 % des comptes valides à l'aide d'une liste telle que `jsmith.txt`. Dans cet exemple, j'ai considérablement augmenté mes chances de réussir une attaque par pulvérisation de mot de passe en démarrant l'attaque avec TOUS les comptes de domaine de ma liste cible. De là, j'ai obtenu des mots de passe valides pour quelques comptes. Finalement, j'ai pu suivre une chaîne d'attaque compliquée impliquant [la délégation contrainte basée sur les ressources (RBCD)](https://posts.specterops.io/another-word-on-delegation-10bdbe3cd94a) et le[Les Shadow Credentials](https://www.fortalicesolutions.com/posts/shadow-credentials-workstation-takeover-edition) attaquent pour finalement prendre le contrôle du domaine.

* * * * *

Considérations sur la pulvérisation de mot de passe
---------------------------------------------------

Bien que la pulvérisation de mot de passe soit utile pour un testeur d'intrusion ou un red teamer, une utilisation imprudente peut causer des dommages considérables, comme le verrouillage de centaines de comptes de production. Un exemple est les tentatives de force brute pour identifier le mot de passe d'un compte à l'aide d'une longue liste de mots de passe. En revanche, la pulvérisation de mots de passe est une attaque plus mesurée, utilisant des mots de passe très courants dans plusieurs secteurs. Le tableau ci-dessous visualise un spray de mot de passe.

#### Visualisation de pulvérisation de mot de passe

| Attaque | Nom d'utilisateur | Mot de passe |
| --- | --- | --- |
| 1 | bob.smith@inlanefreight.local | Bienvenue1 |
| 1 | john.doe@inlanefreight.local | Bienvenue1 |
| 1 | jane.doe@inlanefreight.local | Bienvenue1 |
| RETARD |  |  |
| 2 | bob.smith@inlanefreight.local | Mot de passe0rd |
| 2 | john.doe@inlanefreight.local | Mot de passe0rd |
| 2 | jane.doe@inlanefreight.local | Mot de passe0rd |
| RETARD |  |  |
| 3 | bob.smith@inlanefreight.local | Hiver2022 |
| 3 | john.doe@inlanefreight.local | Hiver2022 |
| 3 | jane.doe@inlanefreight.local | Hiver2022 |

Cela implique d'envoyer moins de demandes de connexion par nom d'utilisateur et est moins susceptible de verrouiller des comptes qu'une attaque par force brute. Cependant, la pulvérisation de mot de passe présente toujours un risque de verrouillage, il est donc essentiel d'introduire un délai entre les tentatives de connexion. La pulvérisation de mot de passe interne peut être utilisée pour se déplacer latéralement au sein d'un réseau, et les mêmes considérations concernant les verrouillages de compte s'appliquent. Cependant, il peut être possible d'obtenir la politique de mot de passe du domaine avec un accès interne, ce qui réduit considérablement ce risque.

Il est courant de trouver une politique de mot de passe qui autorise cinq mauvaises tentatives avant de verrouiller le compte, avec un seuil de déverrouillage automatique de 30 minutes. Certaines organisations configurent des seuils de verrouillage de compte plus étendus, exigeant même qu'un administrateur déverrouille les comptes manuellement. Si vous ne connaissez pas la politique de mot de passe, une bonne règle empirique consiste à attendre quelques heures entre les tentatives, ce qui devrait être suffisamment long pour que le seuil de verrouillage du compte soit réinitialisé. Il est préférable d'obtenir la politique de mot de passe avant de tenter l'attaque lors d'une évaluation interne, mais ce n'est pas toujours possible. Nous pouvons pécher par excès de prudence et choisir de ne faire qu'une seule tentative de pulvérisation de mot de passe ciblée en utilisant un mot de passe faible/commun comme "je vous salue Marie" si toutes les autres options pour prendre pied ou approfondir l'accès ont été épuisées. Selon le type d'évaluation, nous pouvons toujours demander au client de clarifier la politique de mot de passe. Si nous avons déjà un pied ou si nous avons reçu un compte d'utilisateur dans le cadre des tests, nous pouvons énumérer la politique de mot de passe de différentes manières. Pratiquons cela dans la section suivante.
