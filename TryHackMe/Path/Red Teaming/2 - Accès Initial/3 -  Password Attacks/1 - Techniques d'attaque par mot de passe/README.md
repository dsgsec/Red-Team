Techniques d'attaque par mot de passe
========================================

![767385d2697d4b3f8ff7371a04e4c138](https://github.com/dsgsec/Red-Team/assets/82456829/482015e8-68b4-4335-bfd3-b4105981ff6b)

### Techniques d'attaque par mot de passe

Dans cette salle, nous discuterons des techniques qui pourraient être utilisées pour effectuer des attaques par mot de passe. Nous couvrirons diverses techniques telles qu'un dictionnaire, la force brute, la base de règles et les attaques de devinettes. Toutes les techniques ci-dessus sont considérées comme des attaques "en ligne" actives où l'attaquant doit communiquer avec la machine cible pour obtenir le mot de passe afin d'obtenir un accès non autorisé à la machine.

### Craquage de mot de passe vs devinette de mot de passe

Cette section traite de la terminologie de craquage de mot de passe du point de vue de la cybersécurité. Nous discuterons également des différences significatives entre le craquage de mot de passe et la devinette de mot de passe. Enfin, nous présenterons divers outils utilisés pour le craquage de mots de passe, notamment  Hashcat  et  John the Ripper .

Le craquage de mot de passe est une technique utilisée pour découvrir des mots de passe à partir de données cryptées ou hachées vers des données en clair. Les attaquants peuvent obtenir les mots de passe cryptés ou hachés à partir d'un ordinateur compromis ou les capturer en transmettant des données sur le réseau. Une fois les mots de passe obtenus, l'attaquant peut utiliser des techniques d'attaque par mot de passe pour déchiffrer ces mots de passe hachés à l'aide de divers outils.

Le craquage de mot de passe est considéré comme l'une des techniques traditionnelles de test d'intrusion. L'objectif principal est de permettre à l'attaquant d'accéder à des privilèges plus élevés et d'accéder à un système informatique ou à un réseau. La devinette de mot de passe et le piratage de mot de passe sont souvent utilisés par les professionnels de la sécurité de l'information. Les deux ont des significations et des implications différentes. La devinette de mot de passe est une méthode permettant de deviner des mots de passe pour des protocoles et des services en ligne basés sur des dictionnaires. Voici les principales différences entre le craquage de mot de passe et la devinette de mot de passe :

-   La devinette de mot de passe est une technique utilisée pour cibler les protocoles et services en ligne. Par conséquent, cela prend du temps et ouvre la possibilité de générer des journaux pour les tentatives de connexion infructueuses. Une attaque par devinette de mot de passe menée sur un système basé sur le Web nécessite souvent l'envoi d'une nouvelle demande pour chaque tentative, ce qui peut être facilement détecté. Cela peut entraîner le verrouillage d'un compte si le système est conçu et configuré de manière sécurisée.
-   Le craquage de mot de passe est une technique effectuée localement ou sur des systèmes contrôlés par l'attaquant.
