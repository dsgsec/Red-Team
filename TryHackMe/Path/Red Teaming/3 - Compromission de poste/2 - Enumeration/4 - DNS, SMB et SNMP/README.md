### DNS

Nous connaissons tous les requêtes du système de noms de domaine ( DNS ) dans lesquelles nous pouvons rechercher, entre autres, des enregistrements A, AAAA, CName et TXT. Si vous souhaitez parfaire vos connaissances DNS, nous vous suggérons de visiter la salle [DNS in Detail ](https://tryhackme.com/room/dnsindetail). Si nous pouvons obtenir une « copie » de tous les enregistrements auxquels un serveur DNS est chargé de répondre, nous pourrions découvrir des hôtes dont nous ignorions l'existence.

Un moyen simple d'essayer le transfert de zone DNS consiste à utiliser la`dig` commande. Si vous souhaitez en savoir plus sur `dig`les commandes similaires, nous vous suggérons de consulter la salle [de reconnaissance passive ](https://tryhackme.com/room/passiverecon). Selon la configuration du serveur DNS , le transfert de zone DNS peut être restreint. S'il n'est pas restreint, cela devrait être réalisable en utilisant`dig -t AXFR DOMAIN_NAME @DNS_SERVER` . Le `-t AXFR`indique que nous demandons un transfert de zone, tandis que `@`précède le `DNS_SERVER`que nous souhaitons interroger concernant les enregistrements liés au spécifié `DOMAIN_NAME`.

### PME

Server Message Block ( SMB ) est un protocole de communication qui fournit un accès partagé aux fichiers et aux imprimantes. Nous pouvons vérifier les dossiers partagés en utilisant`net share` . Voici un exemple de sortie. Nous pouvons voir que cela `C:\Internal Files`est partagé sous le nom *Internal* .

Terminal

```
user@TryHackMe$ net share

Share name   Resource                        Remark

-------------------------------------------------------------------------------
C$           C:\                             Default share
IPC$                                         Remote IPC
ADMIN$       C:\Windows                      Remote Admin
Internal     C:\Internal Files               Internal Documents
Users        C:\Users
The command completed successfully.
```

### SNMP

Le protocole SNMP (Simple Network Management Protocol) a été conçu pour aider à collecter des informations sur les différents appareils du réseau. Il vous informe de divers événements réseau, d'un serveur avec un disque défectueux à une imprimante à court d'encre. Par conséquent, SNMP peut contenir une mine d'informations pour l'attaquant. Un outil simple pour interroger les serveurs liés à SNMP est `snmpcheck`. Vous pouvez le trouver sur l'AttackBox dans le `/opt/snmpcheck/`répertoire ; la syntaxe est assez simple : `/opt/snmpcheck/snmpcheck.rb 10.10.83.5 -c COMMUNITY_STRING`.

Si vous souhaitez installer `snmpcheck`sur votre machine Linux locale , considérez les commandes suivantes.

Terminal

```
git clone https://gitlab.com/kalilinux/packages/snmpcheck.git
cd snmpcheck/
gem install snmp
chmod +x snmpcheck-1.9.rb
```

Assurez-vous que vous exécutez la machine MS Windows Server à partir de la tâche 4 et répondez aux questions suivantes.
