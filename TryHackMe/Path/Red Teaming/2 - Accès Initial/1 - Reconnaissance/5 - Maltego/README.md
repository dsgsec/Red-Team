Maltego
=======

[Maltego ](https://www.maltego.com/)est une application qui mélange la cartographie mentale avec OSINT. En général, vous commencerez par un nom de domaine, un nom d'entreprise, un nom de personne, une adresse e-mail, etc. Ensuite, vous pourrez laisser cette information subir diverses transformations.

Les informations collectées dans Maltego peuvent être utilisées pour des étapes ultérieures. Par exemple, les informations sur l'entreprise, les noms de contact et les adresses e-mail collectées peuvent être utilisées pour créer des e-mails de phishing d'apparence très légitime.

Considérez chaque bloc sur un graphique Maltego comme une entité. Une entité peut avoir des valeurs pour la décrire. Dans la terminologie de Maltego, une transformation est un morceau de code qui interrogerait une API pour récupérer des informations relatives à une entité spécifique. La logique est illustrée dans la figure ci-dessous. *Les informations* relatives à une entité passent par une *transformation* pour renvoyer zéro ou plusieurs entités.

![2e8ac9b5c947e1af26b7b9e24c5f8361](https://github.com/dsgsec/Red-Team/assets/82456829/57ccd519-b89d-46ee-99d6-e42e89dd772f)

Il est crucial de mentionner que certaines des transformations disponibles dans Maltego peuvent se connecter activement au système cible. Par conséquent, il est préférable de savoir comment fonctionne la transformation avant de l'utiliser si vous souhaitez vous limiter à la reconnaissance passive.

Chaque transformation peut conduire à plusieurs nouvelles valeurs. Par exemple, si nous partons du "Nom DNS" `cafe.thmredteam.com`, nous nous attendons à obtenir de nouveaux types d'entités en fonction de la transformation que nous utilisons. Par exemple, "To IP Address" devrait renvoyer les adresses IP comme indiqué ci-dessous.

![948575bd71d1305e8505c854ac9b81e0](https://github.com/dsgsec/Red-Team/assets/82456829/6c2d37fb-6ae8-4973-b184-5b3572ca2ceb)

Une façon d'y parvenir sur Maltego est de cliquer avec le bouton droit sur le "Nom DNS" `cafe.thmredteam.com`et de choisir :

1.  Transformations standards
2.  Résoudre en IP
3.  Vers l'adresse IP (DNS)

Après avoir exécuté cette transformation, nous obtiendrions une ou plusieurs adresses IP, comme indiqué ci-dessous.

![956d258d9b72d88bb6e088c2f2b4690f](https://github.com/dsgsec/Red-Team/assets/82456829/6c6dc980-dcce-49ad-978a-6ca8ed0934ef)

Ensuite, nous pouvons choisir d'appliquer une autre transformation pour l'une des adresses IP. Considérez la transformation suivante :

1.  DNS depuis IP
2.  Vers le nom DNS du DNS passif (Robtex)

Cette transformation remplira notre graphique avec de nouveaux noms DNS. En quelques clics supplémentaires, vous pouvez obtenir l'emplacement de l'adresse IP, etc. Le résultat pourrait ressembler à l'image ci-dessous.

![3d05e45bd6021ca651892542b06f4230](https://github.com/dsgsec/Red-Team/assets/82456829/504cc4a0-9e29-4100-9e4f-cdba51073ab2)

Les deux exemples ci-dessus devraient vous donner une idée du flux de travail utilisant Maltego. Vous pouvez observer que tout le travail est basé sur des transformations, et Maltego vous aidera à garder votre graphique organisé. Vous obtiendriez les mêmes résultats en interrogeant les différents sites Web et bases de données en ligne ; cependant, Maltego vous aide à obtenir toutes les informations dont vous avez besoin en quelques clics.

Nous avons expérimenté avec `whois`et `nslookup`dans une tâche précédente. Vous obtenez de nombreuses informations, des noms et adresses e-mail aux adresses IP. Les résultats de `whois`et `nslookup`sont affichés visuellement dans le graphique Maltego suivant. Fait intéressant, les transformations Maltego ont pu extraire et organiser les informations renvoyées par la base de données WHOIS. Bien que les adresses e-mail renvoyées ne soient pas utiles en raison de la protection de la vie privée, il vaut la peine de voir comment Maltego peut extraire ces informations et comment elles sont présentées.

![c54a869cffca4d657f46dac618cc9135](https://github.com/dsgsec/Red-Team/assets/82456829/4c428de3-3858-4f52-8b52-85801f13fdf8)

Maintenant que nous avons appris comment le pouvoir de Maltego découle de ses transformations, la seule chose logique est de rendre Maltego plus puissant en ajoutant de nouvelles transformations. Les transformations sont généralement regroupées en différentes catégories en fonction du type de données, du prix et du public cible. Bien que de nombreuses transformations puissent être utilisées avec Maltego Community Edition et des transformations gratuites, d'autres transformations nécessitent un abonnement payant. Une capture d'écran est montrée ci-dessous pour donner une idée plus claire.

![adc8ab512edcf6fef5414d434d7577a1](https://github.com/dsgsec/Red-Team/assets/82456829/24278a15-79fb-4341-98db-0d4a336dc2c9)

L'utilisation de Maltego nécessite une activation, même si vous optez pour Maltego CE (Community Edition). Par conséquent, vous pouvez répondre aux questions suivantes en visitant [Maltego Transform Hub](https://www.maltego.com/transform-hub/) ou en installant et en activant Maltego CE sur votre propre système (pas sur l'AttackBox).
