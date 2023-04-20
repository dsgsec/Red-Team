Principes de reconnaissance externe et de dénombrement
========================================

* * * * *

Avant de lancer un pentest, il peut être avantageux d'effectuer une `reconnaissance externe` de votre cible. Cela peut remplir de nombreuses fonctions différentes, telles que :

- Valider les informations qui vous sont fournies dans le document de cadrage du client
- S'assurer que vous prenez des mesures contre la portée appropriée lorsque vous travaillez à distance
- Recherche de toute information accessible au public pouvant affecter le résultat de votre test, telle que des informations d'identification divulguées

Pensez-y comme ceci; nous essayons d'obtenir la `configuration du terrain` pour nous assurer que nous fournissons le test le plus complet possible à notre client. Cela signifie également identifier toute fuite d'informations potentielle et toute violation de données dans le monde. Cela peut être aussi simple que de glaner un format de nom d'utilisateur sur le site Web principal ou les médias sociaux du client. Nous pouvons également plonger aussi profondément que l'analyse des référentiels GitHub à la recherche d'informations d'identification laissées dans les poussées de code, rechercher dans des documents des liens vers un intranet ou des sites accessibles à distance, et rechercher simplement toute information pouvant nous indiquer comment l'environnement de l'entreprise est configuré.

* * * * *

Que cherchons-nous?
------------------------

Lors de notre reconnaissance externe, il y a plusieurs éléments clés que nous devrions rechercher. Ces informations ne sont peut-être pas toujours accessibles au public, mais il serait prudent de voir ce qui existe. Si nous sommes bloqués lors d'un test d'intrusion, regarder en arrière ce qui pourrait être obtenu grâce à la reconnaissance passive peut nous donner le coup de pouce nécessaire pour aller de l'avant, comme les données de violation de mot de passe qui pourraient être utilisées pour accéder à un VPN ou à un autre service externe. Le tableau ci-dessous met en évidence le « Quoi » dans ce que nous recherchons au cours de cette phase de notre engagement.

| Point de données | Descriptif |
| --- | --- |
| `Espace IP` | ASN valide pour notre cible, netblocks utilisés pour l'infrastructure publique de l'organisation, présence dans le cloud et les fournisseurs d'hébergement, entrées d'enregistrement DNS, etc. |
| `Informations sur le domaine` | Basé sur les données IP, le DNS et les enregistrements de site. Qui administre le domaine ? Existe-t-il des sous-domaines liés à notre cible ? Existe-t-il des services de domaine accessibles au public ? (Serveurs de messagerie, DNS, sites Web, portails VPN, etc.) Pouvons-nous déterminer quel type de défenses sont en place ? (SIEM, AV, IPS/IDS en cours d'utilisation, etc.) |
| `Format de schéma` | Pouvons-nous découvrir les comptes de messagerie, les noms d'utilisateur AD et même les politiques de mot de passe de l'organisation ? Tout ce qui nous donnera des informations que nous pouvons utiliser pour créer une liste de noms d'utilisateur valides afin de tester les services externes pour la pulvérisation de mots de passe, le bourrage d'informations d'identification, le forçage brutal, etc. |
| `Divulgations de données` | Pour les divulgations de données, nous rechercherons des fichiers accessibles au public (.pdf, .ppt, .docx, .xlsx, etc.) pour toute information permettant de faire la lumière sur la cible. Par exemple, tous les fichiers publiés contenant des listes de sites "intranet", des métadonnées utilisateur, des partages ou d'autres logiciels ou matériels critiques dans l'environnement (informations d'identification transmises à un dépôt GitHub public, format de nom d'utilisateur AD interne dans les métadonnées d'un PDF, par exemple exemple.) |
| `Violer les données` | Tous les noms d'utilisateur, mots de passe ou autres informations critiques publiés publiquement qui peuvent aider un attaquant à prendre pied. |

Nous avons abordé le `pourquoi` et le `quoi` de la reconnaissance externe ; examinons `où` et `comment`.

* * * * *

Où cherchons-nous ?
---------------------

Notre liste de points de données ci-dessus peut être rassemblée de différentes manières. Il existe de nombreux sites Web et outils différents qui peuvent nous fournir tout ou partie des informations ci-dessus que nous pourrions utiliser pour obtenir des informations essentielles à notre évaluation. Le tableau ci-dessous répertorie quelques ressources potentielles et des exemples pouvant être utilisés.

| Ressource | Exemples |
| --- | --- |
| `ASN / bureaux d'enregistrement IP` | [IANA](https://www.iana.org/), [arin](https://www.arin.net/) pour rechercher les Amériques, [RIPE](https://www.ripe.net/ ) pour effectuer des recherches en Europe, [BGP Toolkit](https://bgp.he.net/) |
| `Registraires de domaine et DNS` | [Domaintools](https://www.domaintools.com/), [PTRArchive](http://ptrarchive.com/), [ICANN](https://lookup.icann.org/lookup), enregistrement DNS manuel requêtes contre le domaine en question ou contre des serveurs DNS bien connus, tels que `8.8.8.8`. |
| `Médias sociaux` | Recherche Linkedin, Twitter, Facebook, les principaux sites de médias sociaux de votre région, des articles de presse et toute information pertinente que vous pouvez trouver sur l'organisation. |
| `Sites Web d'entreprises accessibles au public` | Souvent, le site Web public d'une société contiendra des informations pertinentes intégrées. Les articles de presse, les documents intégrés et les pages "À propos de nous" et "Contactez-nous" peuvent également être des mines d'or. |
| `Espaces de stockage cloud et de développement` | [GitHub](https://github.com/), [Compartiments AWS S3 et conteneurs de stockage Azure Blog](https://grayhatwarfare.com/), [Recherches Google à l'aide de "Dorks"](https://www. exploit-db.com/google-hacking-database) |
| `Violation des sources de données` | [HaveIBeenPwned](https://haveibeenpwned.com/) pour déterminer si des comptes de messagerie d'entreprise apparaissent dansdonnées de violation publique, [Dehashed](https://www.dehashed.com/) pour rechercher des e-mails d'entreprise avec des mots de passe en clair ou des hachages que nous pouvons essayer de déchiffrer hors ligne. Nous pouvons ensuite essayer ces mots de passe sur tous les portails de connexion exposés (Citrix, RDS, OWA, 0365, VPN, VMware Horizon, applications personnalisées, etc.) susceptibles d'utiliser l'authentification AD. |

### Recherche d'espaces d'adressage

![image](https://academy.hackthebox.com/storage/modules/143/bgp-toolkit.png)

Le `BGP-Toolkit` hébergé par [Hurricane Electric](http://he.net/) est une ressource fantastique pour rechercher quels blocs d'adresses sont attribués à une organisation et dans quel ASN ils résident. Tapez simplement un domaine ou une adresse IP, et la boîte à outils recherchera tous les résultats possibles. Nous pouvons tirer beaucoup de ces informations. De nombreuses grandes entreprises hébergeront souvent elles-mêmes leur infrastructure et, comme elles ont une telle empreinte, elles auront leur propre ASN. Ce ne sera généralement pas le cas pour les petites organisations ou les entreprises naissantes. Lors de vos recherches, gardez cela à l'esprit car les petites organisations hébergent souvent leurs sites Web et d'autres infrastructures dans l'espace de quelqu'un d'autre (Cloudflare, Google Cloud, AWS ou Azure, par exemple). Comprendre où réside cette infrastructure est extrêmement important pour nos tests. Nous devons nous assurer que nous n'interagissons pas avec des infrastructures hors de notre portée. Si nous ne faisons pas attention lorsque nous testons une petite organisation, nous pourrions finir par nuire par inadvertance à une autre organisation partageant cette infrastructure. Vous avez un accord pour tester avec le client, pas avec d'autres sur le même serveur ou avec le fournisseur. Les questions concernant l'infrastructure auto-hébergée ou gérée par un tiers doivent être traitées pendant le processus de cadrage et être clairement répertoriées dans tous les documents de cadrage que vous recevez.

Dans certains cas, votre client devra peut-être obtenir l'approbation écrite d'un fournisseur d'hébergement tiers avant de pouvoir tester. D'autres, comme AWS, ont des [directives](https://aws.amazon.com/security/penetration-testing/) spécifiques pour effectuer des tests d'intrusion et ne nécessitent pas d'approbation préalable pour tester certains de leurs services. D'autres, comme Oracle, vous demandent de soumettre une [notification de test de sécurité cloud](https://docs.oracle.com/en-us/iaas/Content/Security/Concepts/security_testing-policy_notification.htm). Ces types d'étapes doivent être gérées par la direction de votre entreprise, l'équipe juridique, l'équipe des contrats, etc. Il est de notre responsabilité de nous assurer que nous avons l'autorisation explicite d'attaquer tout hôte (à la fois interne et externe), et arrêter et clarifier la portée par écrit ne fait jamais de mal.

### DNS

Le DNS est un excellent moyen de valider notre portée et de découvrir les hôtes accessibles que le client n'a pas divulgués dans son document d'orientation. Des sites comme [domaintools](https://whois.domaintools.com/) et [viewdns.info](https://viewdns.info/) sont d'excellents points de départ. Nous pouvons récupérer de nombreux enregistrements et autres données allant de la résolution DNS aux tests DNSSEC et si le site est accessible dans des pays plus restreints. Parfois, nous pouvons trouver des hôtes supplémentaires hors de portée, mais qui semblent intéressants. Dans ce cas, nous pourrions apporter cette liste à notre client pour voir si l'un d'entre eux devrait effectivement être inclus dans le champ d'application. Nous pouvons également trouver des sous-domaines intéressants qui n'étaient pas répertoriés dans les documents d'orientation, mais résident sur des adresses IP dans le champ d'application et sont donc un jeu équitable.

#### Viewdns.info

![image](https://academy.hackthebox.com/storage/modules/143/viewdnsinfo.png)

C'est également un excellent moyen de valider certaines des données trouvées à partir de nos recherches IP/ASN. Toutes les informations sur le domaine trouvé ne seront pas à jour, et effectuer des vérifications qui peuvent valider ce que nous voyons est toujours une bonne pratique.

### Données publiques

Les médias sociaux peuvent être un trésor de données intéressantes qui peuvent nous donner des indications sur la structure de l'organisation, le type d'équipement qu'elle exploite, les implémentations potentielles de logiciels et de sécurité, leur schéma, etc. En haut de cette liste se trouvent des sites liés à l'emploi comme LinkedIn, Indeed.com et Glassdoor. De simples offres d'emploi en disent souvent beaucoup sur une entreprise. Par exemple, jetez un œil à la liste des emplois ci-dessous. Il est destiné à un `administrateur SharePoint` et peut nous renseigner sur de nombreuses choses. D'après la liste, nous pouvons dire que l'entreprise utilise SharePoint depuis un certain temps et dispose d'un programme mature puisqu'elle parle de programmes de sécurité, de sauvegarde et de reprise après sinistre, etc. Ce qui nous intéresse dans cette publication, c'est que nous pouvons voir que l'entreprise utilise probablement SharePoint 2013 et SharePoint 2016. Cela signifie qu'ils ont peut-être mis à niveau en place, laissant potentiellement en jeu des vulnérabilités qui n'existent peut-être pas dans les versions plus récentes. Cela signifie également que nous pouvons rencontrer différentes versions de SharePoint au cours de nos missions.

### Liste des tâches d'administration Sharepoint

![image](https://academy.hackthebox.com/storage/modules/143/spjob2.png)

Ne négligez pas les informations publiques telles que les offres d'emploi ou les médias sociaux. Vous pouvez lgagnez beaucoup sur une organisation uniquement grâce à ce qu'elle publie, et une publication bien intentionnée pourrait divulguer des données nous concernant en tant que testeurs d'intrusion.

Les sites Web hébergés par l'organisation sont également d'excellents endroits pour rechercher des informations. Nous pouvons recueillir des e-mails de contact, des numéros de téléphone, des organigrammes, des documents publiés, etc. Ces sites, en particulier les documents intégrés, peuvent souvent avoir des liens vers des infrastructures internes ou des sites intranet que vous ne connaîtriez pas autrement. La vérification de toutes les informations accessibles au public pour ces types de détails peut être un gain rapide lorsque vous essayez de formuler une image de la structure du domaine. Avec l'utilisation croissante de sites tels que GitHub, le stockage en nuage AWS et d'autres plates-formes hébergées sur le Web, les données peuvent également être divulguées involontairement. Par exemple, un développeur travaillant sur un projet peut accidentellement laisser des informations d'identification ou des notes codées en dur dans une version de code. Si vous savez où chercher ces données, cela peut vous donner une victoire facile. Cela pourrait faire la différence entre devoir pulvériser des mots de passe et des informations d'identification par force brute pendant des heures ou des jours ou prendre rapidement pied avec des informations d'identification de développeur, qui peuvent également avoir des autorisations élevées. Des outils comme [Trufflehog](https://github.com/trufflesecurity/truffleHog) et des sites comme [Greyhat Warfare](https://buckets.grayhatwarfare.com/) sont des ressources fantastiques pour trouver ces fils d'Ariane.

Nous avons passé un certain temps à discuter du recensement externe et de la reconnaissance d'une organisation, mais ce n'est qu'une pièce du puzzle. Pour une introduction plus détaillée à l'OSINT et à l'énumération externe, consultez [Footprinting](https://academy.hackthebox.com/course/preview/footprinting) et [OSINT : Corporate Recon](https://academy.hackthebox. com/course/preview/osint-corporate-recon).

Jusqu'à présent, nous avons été essentiellement passifs dans nos discussions. Au fur et à mesure que vous avancerez dans le pentest, vous deviendrez plus pratique, validant les informations que vous avez trouvées et sondant le domaine pour plus d'informations. Prenons une minute pour discuter des principes de dénombrement et de la façon dont nous pouvons mettre en place un processus pour effectuer ces actions.

* * * * *

Principes généraux de dénombrement
----------------------------------

Gardant à l'esprit que notre objectif est de mieux comprendre notre cible, nous recherchons toutes les avenues possibles que nous pouvons trouver qui nous fourniront une voie potentielle vers l'intérieur. L'énumération elle-même est un processus itératif que nous répéterons plusieurs fois tout au long d'un test d'intrusion. Outre le document d'orientation du client, il s'agit de notre principale source d'informations, nous voulons donc nous assurer que nous ne négligeons aucun effort. Au début de notre énumération, nous utiliserons d'abord des ressources `` passives '', en commençant par une portée large et en nous rétrécissant. Une fois que nous aurons épuisé notre série initiale de dénombrement passif, nous devrons examiner les résultats, puis passer à notre phase de dénombrement actif.

* * * * *

Exemple de processus d'énumération
---------------------------

Nous avons déjà couvert pas mal de concepts relatifs au dénombrement. Commençons à tout mettre ensemble. Nous allons pratiquer nos tactiques d'énumération sur le domaine `inlanefreight.com` sans effectuer d'analyses lourdes (telles que Nmap ou des analyses de vulnérabilité qui sont hors de portée). Nous allons commencer par vérifier nos données Netblocks et voir ce que nous pouvons trouver.

#### Vérifier les données ASN/IP et de domaine

![image](https://academy.hackthebox.com/storage/modules/143/BGPhe-inlane.png)

De ce premier aperçu, nous avons déjà glané quelques infos intéressantes. BGP.il rapporte :

- Adresse IP : 134.209.24.248
- Serveur de messagerie : mail1.inlanefreight.com
- Serveurs de noms : NS1.inlanefreight.com & NS2.inlanefreight.com

Pour l'instant, c'est ce qui nous intéresse depuis sa sortie. Inlanefreight n'est pas une grande entreprise, nous ne nous attendions donc pas à ce qu'elle ait sa propre ASN. Validons maintenant certaines de ces informations.

#### Afficher les résultats DNS

![image](https://academy.hackthebox.com/storage/modules/143/viewdns-results.png)

Dans la requête ci-dessus, nous avons utilisé `viewdns.info` pour valider l'adresse IP de notre cible. Les deux résultats concordent, ce qui est bon signe. Essayons maintenant une autre route pour valider les deux serveurs de noms dans nos résultats.

Afficher les résultats DNS

```
dsgsec@htb[/htb]$ nslookup ns1.inlanefreight.com

Serveur : 192.168.186.1
Adresse : 192.168.186.1#53

Réponse sans autorité :
Nom : ns1.inlanefreight.com
Adresse : 178.128.39.165

nslookup ns2.inlanefreight.com
Serveur : 192.168.86.1
Adresse : 192.168.86.1#53

Réponse sans autorité :
Nom : ns2.inlanefreight.com
Adresse : 206.189.119.186

```

Nous avons maintenant `deux` nouvelles adresses IP à ajouter à notre liste pour validation et test. Avant d'entreprendre toute autre action avec eux, assurez-vous qu'ils sont dans le champ d'application de votre test. Pour nos besoins, les adresses IP réelles ne seraient pas susceptibles d'être analysées, mais nous pourrions parcourir passivement tous les sites Web pour rechercher des données intéressantes. Pour l'instant, c'est tout avec l'énumération des informations de domaine à partir du DNS. Jetons un coup d'œil aux informations accessibles au public.

Inlanefreight est une société fictive que nous utilisons pour tson module, il n'y a donc pas de réelle présence sur les réseaux sociaux. Cependant, nous vérifierions des sites comme LinkedIn, Twitter, Instagram et Facebook pour obtenir des informations utiles si elles étaient réelles. Au lieu de cela, nous allons passer à l'examen du site "inlanefreight.com".

La première vérification que nous avons effectuée visait à rechercher des documents. En utilisant `filetype:pdf inurl:inlanefreight.com` comme recherche, nous recherchons des PDF.

#### Chasse aux fichiers

![image](https://academy.hackthebox.com/storage/modules/143/google-dorks.png)

Un document est apparu, nous devons donc nous assurer de noter le document et son emplacement et de télécharger une copie localement pour fouiller. Il est toujours préférable d'enregistrer les fichiers, les captures d'écran, la sortie de numérisation, la sortie de l'outil, etc., dès que nous les rencontrons ou les générons. Cela nous aide à conserver un enregistrement aussi complet que possible et à ne pas risquer d'oublier où nous avons vu quelque chose ou de perdre des données critiques. Ensuite, recherchons toutes les adresses e-mail que nous pouvons trouver.

#### Chasse aux adresses e-mail

![image](https://academy.hackthebox.com/storage/modules/143/intext-dork.png)

En utilisant le dork `intext :"@inlanefreight.com" inurl:inlanefreight.com`, nous recherchons toute instance qui ressemble à la fin d'une adresse e-mail sur le site Web. Un résultat prometteur est venu avec une page de contact. Lorsque nous regardons la page (photo ci-dessous), nous pouvons voir une grande liste d'employés et leurs coordonnées. Ces informations peuvent être utiles car nous pouvons déterminer que ces personnes sont au moins très probablement actives et travaillent toujours avec l'entreprise.

#### Résultats Dork par e-mail

En parcourant la [page de contact](https://www.inlanefreight.com/index.php/contact/), nous pouvons voir plusieurs e-mails pour le personnel de différents bureaux à travers le monde. Nous avons maintenant une idée de leur convention de dénomination des e-mails (first.last) et de l'endroit où certaines personnes travaillent dans l'organisation. Cela pourrait être utile lors d'attaques ultérieures par pulvérisation de mot de passe ou si l'ingénierie sociale/le phishing faisaient partie de notre champ d'engagement.

![image](https://academy.hackthebox.com/storage/modules/143/ilfreightemails.png)

#### Récolte du nom d'utilisateur

Nous pouvons utiliser un outil tel que [linkedin2username](https://github.com/initstring/linkedin2username) pour extraire les données de la page LinkedIn d'une entreprise et créer divers mashups de noms d'utilisateur (flast, first.last, f.last, etc. ) qui peuvent être ajoutés à notre liste de cibles potentielles de pulvérisation de mot de passe.

#### Chasse aux identifiants

[Dehashed](http://dehashed.com/) est un excellent outil pour rechercher des informations d'identification en texte clair et des hachages de mot de passe dans les données de violation. Nous pouvons effectuer une recherche soit sur le site, soit à l'aide d'un script qui effectue des requêtes via l'API. En règle générale, nous trouverons de nombreux anciens mots de passe pour les utilisateurs qui ne fonctionnent pas sur des portails externes qui utilisent l'authentification AD (ou interne), mais nous pouvons avoir de la chance ! Il s'agit d'un autre outil qui peut être utile pour créer une liste d'utilisateurs pour la pulvérisation de mot de passe externe ou interne.

Remarque : Pour nos besoins, les exemples de données ci-dessous sont fictifs.

Chasse aux identifiants

```
dsgsec@htb[/htb]$ sudo python3 dehashed.py -q inlanefreight.local -p

identifiant : 5996447501
email : roger.grimes@inlanefreight.local
nom d'utilisateur : rgrimes
mot de passe : Ilovefishing!
hash_password :
nom : Roger Grimes
vin :
adresse :
téléphone :
nom_base de données : ModBSolutions

identifiant : 7344467234
email : jane.yu@inlanefreight.local
nom d'utilisateur : jyu
mot de passe : Starlight1982_ !
hash_password :
nom : Jane Yu
vin :
adresse :
téléphone :
nom_base de données : MyFitnessPal

<SNIP>

```

Maintenant que nous avons compris, essayez de rechercher d'autres résultats liés au domaine inlanefreight.com. Que pouvez-vous trouver ? Y a-t-il d'autres fichiers, pages ou informations utiles intégrés sur le site ? Cette section a démontré l'importance d'analyser en profondeur notre cible, à condition de rester dans le cadre et de ne pas tester tout ce que nous ne sommes pas autorisés à faire et de respecter les contraintes de temps de la mission. J'ai eu pas mal d'évaluations où j'avais du mal à m'implanter d'un point de vue anonyme sur le réseau interne et j'ai eu recours à la création d'une liste de mots en utilisant diverses sources externes (Google, LinkedIn scraping, Dehashed, etc.) puis j'ai effectué un mot de passe interne ciblé pulvérisation pour obtenir des informations d'identification valides pour un compte d'utilisateur de domaine standard. Comme nous le verrons dans les sections suivantes, nous pouvons effectuer la grande majorité de notre énumération AD interne avec juste un ensemble d'informations d'identification d'utilisateur de domaine à faible privilège et même de nombreuses attaques. Le plaisir commence une fois que nous avons un ensemble d'informations d'identification. Passons à l'énumération interne et commençons à analyser le domaine interne "INLANEFREIGHT.LOCAL" de manière passive et active conformément à la portée et aux règles d'engagement de notre évaluation.