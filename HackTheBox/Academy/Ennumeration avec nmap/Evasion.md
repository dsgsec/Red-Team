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
