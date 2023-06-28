Analyse des menances
====================
Après avoir identifié les informations critiques, nous devons analyser les menaces. *L'analyse des menaces consiste à identifier les adversaires potentiels ainsi que leurs intentions et leurs capacités* . Adaptée du [manuel du programme de sécurité des opérations (OPSEC) du département américain de la Défense (DoD)](https://www.esd.whs.mil/Portals/54/Documents/DD/issuances/dodm/520502m.pdf) , l'analyse des menaces vise à répondre aux questions suivantes :

1.  Qui est l'adversaire ?
2.  Quels sont les objectifs de l'adversaire ?
3.  Quelles tactiques, techniques et procédures l'adversaire utilise-t-il ?
4.  Quelles informations essentielles l'adversaire a-t-il obtenues, le cas échéant ?

![528fdefeb510a511d19fa0ba2496cc74](https://github.com/dsgsec/Red-Team/assets/82456829/9b45661b-476d-4dc3-b15d-30c504e68659)

La tâche de l'équipe rouge est d'imiter une attaque réelle afin que l'équipe bleue découvre ses lacunes, le cas échéant, et soit mieux préparée à faire face aux menaces entrantes. L'objectif principal de l'équipe bleue est d'assurer la sécurité du réseau et des systèmes de l'organisation. Les intentions de l'équipe bleue sont claires ; ils veulent garder l'équipe rouge hors de leur réseau. Par conséquent, compte tenu de la tâche de l'équipe rouge, l'équipe bleue est considérée comme notre adversaire car chaque équipe a des objectifs contradictoires. Notons que les capacités de l'équipe bleue ne sont pas toujours connues au départ.

Les joueurs tiers malveillants peuvent avoir des intentions et des capacités différentes et peuvent par conséquent suspendre une menace. Cette partie peut être quelqu'un avec des capacités modestes qui analysent les systèmes au hasard à la recherche de fruits à portée de main, comme un serveur exploitable non corrigé, ou il peut s'agir d'un adversaire capable ciblant votre entreprise ou vos systèmes clients. Par conséquent, les intentions et les capacités de ce tiers peuvent également en faire un adversaire.

| Adversaire | Intentions | Capacités |
| --- | --- | --- |
| L'équipe bleue | Éloignez les intrus | Pas toujours connu |
| Tiers malveillant | Varie | Varie |

Nous considérons comme une menace tout adversaire ayant l'intention et la capacité de prendre des mesures qui nous empêcheraient de mener à bien notre opération :

```
threat = adversary + intent + capability
```

En d'autres termes, un adversaire sans intention ni capacité ne constitue pas une menace pour nos objectifs.
