# Firewall / IPS / IDS Evasion

## Introduction

Il est possible de rajouter des arguments à notre commande afin de passer au travers des éléments de sécurité d'un réseau.
C'est faisable en changeant :
- La fragmentation des paquets
- Le port source
- L'IP source


## Commandes

| **Nmap Option** | **Description** |
|---|----|
| `-f` | Fragmente les paquets |
| `-g` | Change le port source |
| `-D RND:12` | Change l'adresse IP source 12 fois|
| `--disable-arp-ping` | Désactive la détection d'hôte par ARP|
| `-Pn` | Désactive la découverte d'hôtes|
| `-D RND:12` | Change l'adresse IP source 12 fois|

<hr>

## Notes :
1. Par défaut Nmap scanne le réseau entier, cela permet donc d'identifier le scan car nous allons envoyer des requêtes en grand nombres, pour cela nous allons utiliser
```
nmap -Pn
```

2. Pour désactiver la découverte d'hôtes sur le réseau l'option "-Pn" ne suffira pas, car nmap tentera de faire de même à l'aide du protocole ARP, il faut donc désactiver la découverte par ARP avec la commande :
```
nmap --disable-arp-ping
```