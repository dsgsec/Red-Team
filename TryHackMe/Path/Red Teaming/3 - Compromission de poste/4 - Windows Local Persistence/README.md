![1a73b2609190f98fe926361dfe61b910](https://github.com/dsgsec/Red-Team/assets/82456829/fee0f616-4e45-45a3-b61c-0b5b022e32cb)

Après avoir pris pied sur le réseau interne de votre cible, vous devez vous assurer de ne pas en perdre l'accès avant d'accéder aux joyaux de la couronne. Établir la persistance est l'une des premières tâches que nous aurons en tant qu'attaquants lors de l'accès à un réseau. En termes simples, la persistance fait référence à la création de moyens alternatifs pour retrouver l'accès à un hôte sans repasser par la phase d'exploitation.

Il existe de nombreuses raisons pour lesquelles vous souhaitez établir la persistance le plus rapidement possible, notamment :

-   La réexploitation n'est pas toujours possible : certains exploits instables peuvent tuer le processus vulnérable pendant l'exploitation, vous permettant ainsi d'avoir une seule chance sur certains d'entre eux.
-   Prendre pied est difficile à reproduire : par exemple, si vous avez utilisé une campagne de phishing pour obtenir votre premier accès, la répéter pour retrouver l'accès à un hébergeur est tout simplement trop de travail. Votre deuxième campagne pourrait également ne pas être aussi efficace, vous laissant sans accès au réseau.
-   L'équipe bleue est à vos trousses : toute vulnérabilité utilisée pour obtenir votre premier accès peut être corrigée si vos actions sont détectées. Vous êtes dans une course contre la montre !

Bien que vous puissiez conserver le hachage de certains mots de passe d'administrateur et le réutiliser pour vous reconnecter, vous risquez toujours que ces informations d'identification soient modifiées à un moment donné. De plus, il existe des moyens plus sournois pour retrouver l'accès à une machine compromise, ce qui rend la vie plus difficile à l'équipe bleue.

Dans cette salle, nous examinerons les techniques les plus couramment utilisées par les attaquants pour établir la persistance dans les systèmes Windows. Avant d'entrer dans cette salle, il est recommandé de se familiariser avec les principes fondamentaux des systèmes Windows. Vous pouvez consulter les salles à ce sujet dans les liens suivants :

-   [Fondamentaux de Windows 1](https://tryhackme.com/room/windowsfundamentals1xbx)
-   [Fondamentaux de Windows 2](https://tryhackme.com/room/windowsfundamentals2x0x)

Powershell est également largement utilisé dans cette salle. Vous pouvez en apprendre davantage à ce sujet dans la  salle [Hacking with Powershell](https://tryhackme.com/room/powershell)  .
