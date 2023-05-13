osTicket
========

* * * * *

[osTicket](https://osticket.com/) est un système de ticket d'assistance open source. Il peut être comparé à des systèmes tels que Jira, OTRS, Request Tracker et Spiceworks. osTicket peut intégrer les demandes des utilisateurs à partir de formulaires électroniques, téléphoniques et Web dans une interface Web. osTicket est écrit en PHP et utilise un backend MySQL. Il peut être installé sur Windows ou Linux. Bien qu'il n'y ait pas une quantité considérable d'informations sur le marché facilement disponibles sur osTicket, une recherche rapide sur Google pour "logiciel Helpdesk - optimisé par osTicket" renvoie environ 44 000 résultats, dont beaucoup semblent être des entreprises, des systèmes scolaires, des universités, des administrations locales, etc. ., en utilisant l'application. osTicket a même été montré brièvement dans l'émission [M. Robot] (https://forum.osticket.com/d/86225-osticket-on-usas-mr-robot-s01e08).

En plus d'apprendre à énumérer et à attaquer osTicket, le but de cette section est également de vous présenter le monde des systèmes de billetterie d'assistance et pourquoi ils ne doivent pas être négligés lors de nos évaluations.

* * * * *

Empreinte/Découverte/Énumération
----------------------------------

En regardant notre analyse EyeWitness précédente, nous remarquons une capture d'écran d'une instance osTicket qui montre également qu'un cookie nommé `OSTSESSID` a été défini lors de la visite de la page.

![image](https://academy.hackthebox.com/storage/modules/113/osticket_eyewitness.png)

De plus, la plupart des installations d'osTicket afficheront le logo d'osTicket avec la phrase `powered by` devant celui-ci dans le pied de page. Le pied de page peut également contenir les mots `Support Ticket System`.

![](https://academy.hackthebox.com/storage/modules/113/osticket_main.png)

Une analyse Nmap affichera simplement des informations sur le serveur Web, comme Apache ou IIS, et ne nous aidera pas à tracer l'application.

`osTicket` est une application Web hautement maintenue et entretenue. Si nous examinons les [CVE](https://www.cvedetails.com/vendor/2292/Osticket.html) découvertes au fil des décennies, nous ne trouverons pas beaucoup de vulnérabilités et d'exploits que pourrait avoir osTicket. C'est un excellent exemple pour montrer à quel point il est important de comprendre le fonctionnement d'une application web. Même si l'application n'est pas vulnérable, elle peut toujours être utilisée pour nos besoins. Ici, nous pouvons décomposer les fonctions principales en couches :

| `1\. Entrée utilisateur` | `2\. Traitement` | `3\. Solution` |
| --- | --- | --- |

#### Entrée utilisateur

La fonction principale d'osTicket est d'informer les employés de l'entreprise d'un problème afin qu'un problème puisse être résolu avec le service ou d'autres composants. Un avantage significatif que nous avons ici est que l'application est open-source. Par conséquent, nous avons de nombreux tutoriels et exemples disponibles pour examiner de plus près l'application. Par exemple, à partir de l'osTicket [documentation](https://docs.osticket.com/en/latest/Getting%20Started/Post-Installation.html), nous pouvons voir que seuls le personnel et les utilisateurs disposant de privilèges d'administrateur peuvent accéder à l'administrateur panneau. Donc, si notre entreprise cible utilise cette application ou une application similaire, nous pouvons causer un problème et "faire l'idiot" et contacter le personnel de l'entreprise. Le "manque de" connaissance simulé sur les services proposés par l'entreprise en combinaison avec un problème technique est une approche d'ingénierie sociale largement répandue pour obtenir plus d'informations de l'entreprise.

#### Traitement

En tant que membres du personnel ou administrateurs, ils essaient de reproduire des erreurs importantes pour trouver le cœur du problème. Le traitement est finalement effectué en interne dans un environnement isolé qui aura des paramètres très similaires aux systèmes en production. Supposons que le personnel et les administrateurs soupçonnent qu'il existe un bogue interne susceptible d'affecter l'entreprise. Dans ce cas, ils entreront plus en détail pour découvrir d'éventuelles erreurs de code et résoudre des problèmes plus importants.

#### Solution

Selon l'ampleur du problème, il est très probable que d'autres membres du personnel des services techniques seront impliqués dans la correspondance par courrier électronique. Cela nous donnera de nouvelles adresses e-mail à utiliser contre le panneau d'administration osTicket (dans le pire des cas) et des noms d'utilisateur potentiels avec lesquels nous pouvons effectuer OSINT ou essayer d'appliquer à d'autres services de l'entreprise.

* * * * *

Attaquer osTicket
------------------

Une recherche d'osTicket sur exploit-db montre divers problèmes, notamment l'inclusion de fichiers à distance, l'injection SQL, le téléchargement de fichiers arbitraires, XSS, etc. .gov/vuln/detail/CVE-2020-24881) qui était une vulnérabilité SSRF. S'il est exploité, ce type de faille peut être exploité pour accéder aux ressources internes ou effectuer une analyse des ports internes.

Outre les vulnérabilités liées aux applications Web, les portails de support peuvent parfois être utilisés pour obtenir une adresse e-mail pour un domaine d'entreprise, qui peut être utilisée pour s'inscrire à d'autres applications exposées nécessitant l'envoi d'une vérification par e-mail. Comme mentionné précédemment dans le module, cela est illustré dans la boîte de publication hebdomadaire HTB [Delivery](https://0xdf.gitlab.io/2021/05/22/htb-delivery.html) avec une vidéo de présentation [ici]( https://www.youtube.com/watch?v=gbs43E71mFM).

Passons en revue un exemple rapide, lié à cet [excellent article de blog](https://medium.com/intigriti/how-i-hacked-hundreds-of-companies-through-their-helpdesk-b7680ddc2d4c) qui [ @ippsec](https://twitter.com/ippsec) également mentionné était une source d'inspiration pour sa livraison de boîte que je recommande fortement de vérifier après avoir lu cette section.

Supposons que nous trouvions un service exposé tel que le serveur Slack d'une entreprise ou GitLab, qui nécessite une adresse e-mail d'entreprise valide pour se joindre. De nombreuses entreprises ont un e-mail d'assistance tel que `support@inlanefreight.local`, et les e-mails qui lui sont envoyés sont disponibles sur des portails d'assistance en ligne qui peuvent aller de Zendesk à un outil interne personnalisé. De plus, un portail d'assistance peut attribuer une adresse e-mail interne temporaire à un nouveau ticket afin que les utilisateurs puissent rapidement vérifier son statut.

Si nous rencontrons un portail d'assistance client lors de notre évaluation et que nous pouvons soumettre un nouveau ticket, nous pourrons peut-être obtenir une adresse e-mail d'entreprise valide.

![](https://academy.hackthebox.com/storage/modules/113/new_ticket.png)

Il s'agit d'une version modifiée d'osTicket à titre d'exemple, mais nous pouvons voir qu'une adresse e-mail a été fournie.

![](https://academy.hackthebox.com/storage/modules/113/ticket_email.png)

Maintenant, si nous nous connectons, nous pouvons voir des informations sur le ticket et les moyens de poster une réponse. Si l'entreprise configure son logiciel d'assistance pour associer les numéros de ticket aux e-mails, tout e-mail envoyé à l'adresse e-mail que nous avons reçue lors de l'inscription, `940288@inlanefreight.local`, s'affichera ici. Avec cette configuration, si nous pouvons trouver un portail externe tel qu'un Wiki, un service de chat (Slack, Mattermost, Rocket.chat) ou un référentiel Git tel que GitLab ou Bitbucket, nous pourrons peut-être utiliser cet e-mail pour enregistrer un compte et le portail d'assistance du service d'assistance pour recevoir un e-mail de confirmation d'inscription.

![](https://academy.hackthebox.com/storage/modules/113/ost_tickets.png)

* * * * *

osTicket - Exposition de données sensibles
----------------------------------

Disons que nous sommes sur un test d'intrusion externe. Au cours de notre OSINT et de la collecte d'informations, nous découvrons plusieurs identifiants d'utilisateur à l'aide de l'outil [Dehashed](http://dehashed.com/) (pour nos besoins, les exemples de données ci-dessous sont fictifs).

Solution

```
dsgsec@htb[/htb]$ sudo python3 dehashed.py -q inlanefreight.local -p

identifiant : 5996447501
email : julie.clayton@inlanefreight.local
nom d'utilisateur : jclayton
mot de passe : JulieC8765!
hash_password :
nom : Julie Clayton
vin :
adresse :
téléphone :
nom_base de données : ModBSolutions

identifiant : 7344467234
email : kevin@inlanefreight.local
nom d'utilisateur : kgrimes
mot de passe : Fish1ng_s3ason !
hash_password :
nom : Kevin Grimes
vin :
adresse :
téléphone :
nom_base de données : MyFitnessPal

<SNIP>

```

Ce vidage affiche les mots de passe en clair pour deux utilisateurs différents : `jclayton` et `kgrimes`. À ce stade, nous avons également effectué une énumération de sous-domaines et en avons rencontré plusieurs intéressants.

Solution

```
dsgsec@htb[/htb]$ cat ilfreight_subdomains

vpn.inlanefreight.local
support.inlanefreight.local
ns1.inlanefreight.local
mail.inlanefreight.local
apps.inlanefreight.local
ftp.inlanefreight.local
dev.inlanefreight.local
ir.inlanefreight.local
auth.inlanefreight.local
carrieres.inlanefreight.local
portal-stage.inlanefreight.local
dns1.inlanefreight.local
dns2.inlanefreight.local
meet.inlanefreight.local
portail-test.inlanefreight.local
home.inlanefreight.local
legacy.inlanefreight.local

```

Nous parcourons chaque sous-domaine et constatons que beaucoup sont obsolètes, mais `support.inlanefreight.local` et `vpn.inlanefreight.local` sont actifs et très prometteurs. `Support.inlanefreight.local` héberge une instance osTicket, et `vpn.inlanefreight.local` est un portail Web VPN SSL Barracuda qui ne semble pas utiliser l'authentification multifacteur.

![](https://academy.hackthebox.com/storage/modules/113/osticket_admin.png)

Essayons les informations d'identification pour `jclayton`. Pas de chance. Nous essayons ensuite les informations d'identification pour `kgrimes` et n'avons pas de succès, mais remarquant que la page de connexion accepte également une adresse e-mail, nous essayons `kevin@inlanefreight.local` et obtenons une connexion réussie !

![](https://academy.hackthebox.com/storage/modules/113/osticket_kevin.png)

L'utilisateur `kevin` semble être un agent d'assistance, mais n'a aucun ticket ouvert. Peut-être ne sont-ils plus actifs ? Dans une entreprise occupée, on s'attendrait à voir des tickets ouverts. En fouillant un peu, nous trouvons un ticket fermé, une conversation entre un employé distant et l'agent de support.

![](https://academy.hackthebox.com/storage/modules/113/osticket_ticket.png)

L'employé déclare qu'il a été verrouillé sur son compte VPN et demande à l'agent de le réinitialiser. L'agent indique ensuite à l'utilisateur que le mot de passe a été réinitialisé sur le nouveau mot de passe de connexion standard. L'utilisateur n'a pas ce mot de passe et demande à l'agent de l'appeler pour lui fournir le mot de passe (solide sensibilisation à la sécurité !). L'agent commet alors une erreur et envoie le mot de passe à l'utilisateur directement via le portail. À partir de là, nous pourrions essayer ce mot de passe contre le portail VPN exposé car l'utilisationr ne l'a peut-être pas changé.

De plus, l'agent de support déclare qu'il s'agit du mot de passe standard donné aux nouveaux membres et définit le mot de passe de l'utilisateur sur cette valeur. Nous avons été dans de nombreuses organisations où le service d'assistance utilise un mot de passe standard pour les nouveaux utilisateurs et les réinitialisations de mot de passe. Souvent, la politique de mot de passe du domaine est laxiste et n'oblige pas l'utilisateur à changer lors de la prochaine connexion. Si tel est le cas, cela peut fonctionner pour d'autres utilisateurs. Bien que hors de la portée de ce module, dans ce scénario, il serait utile d'utiliser des outils tels que [linkedin2username](https://github.com/initstring/linkedin2username) pour créer une liste d'utilisateurs d'employés de l'entreprise et tenter une attaque par pulvérisation de mot de passe contre le point de terminaison VPN avec ce mot de passe standard.

De nombreuses applications telles que osTicket contiennent également un carnet d'adresses. Il serait également utile d'exporter tous les e-mails/noms d'utilisateur du carnet d'adresses dans le cadre de notre énumération, car ils pourraient également s'avérer utiles lors d'une attaque telle que la pulvérisation de mots de passe.

* * * * *

Réflexions finales
----------------

Bien que cette section présente certains scénarios fictifs, ils sont basés sur des choses que nous sommes susceptibles de voir dans le monde réel. Lorsque nous rencontrons des portails d'assistance (en particulier externes), nous devons tester la fonctionnalité et voir si nous pouvons faire des choses comme créer un ticket et avoir une adresse e-mail d'entreprise légitime qui nous est attribuée. À partir de là, nous pourrons peut-être utiliser l'adresse e-mail pour vous connecter à d'autres services de l'entreprise et accéder à des données sensibles.

Cette section montre également les dangers de la réutilisation des mots de passe et les types de données que nous pouvons très probablement trouver si nous pouvons accéder à la file d'attente de tickets d'assistance d'un agent du service d'assistance. Les organisations peuvent empêcher ce type de fuite d'informations en prenant quelques mesures relativement simples :

- Limitez les applications exposées à l'extérieur
- Appliquer l'authentification multifacteur sur tous les portails externes
- Fournir une formation de sensibilisation à la sécurité à tous les employés et leur conseiller de ne pas utiliser leurs e-mails d'entreprise pour s'inscrire à des services tiers
- Appliquez une politique de mots de passe forts dans Active Directory et sur toutes les applications, en interdisant les mots courants tels que les variantes de `welcome` et `password`, le nom de l'entreprise, les saisons et les mois
- Exiger qu'un utilisateur change son mot de passe après sa connexion initiale et expire périodiquement les mots de passe de l'utilisateur