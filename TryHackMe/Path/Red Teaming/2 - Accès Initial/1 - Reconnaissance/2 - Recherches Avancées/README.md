Recherches avancées
===========

Savoir utiliser efficacement un moteur de recherche est une compétence cruciale. Le tableau suivant présente certains modificateurs de recherche populaires qui fonctionnent avec de nombreux moteurs de recherche populaires.

| Symbole / Syntaxe | Fonction |
| --- | --- |
| `"search phrase"` | Trouver des résultats avec la phrase de recherche exacte |
| `OSINT filetype:pdf` | Trouver des fichiers de type `PDF`liés à un certain terme. |
| `salary site:blog.tryhackme.com` | Limitez les résultats de recherche à un site spécifique. |
| `pentest -site:example.com` | Exclure un site spécifique des résultats |
| `walkthrough intitle:TryHackMe` | Trouvez des pages avec un terme spécifique dans le titre de la page. |
| `challenge inurl:tryhackme` | Rechercher des pages avec un terme spécifique dans l'URL de la page. |

Remarque : En plus de `pdf`, les autres types de fichiers à prendre en compte sont : `doc`, `docx`, `ppt`, `pptx`, `xls`et `xlsx`.

Chaque moteur de recherche peut avoir un ensemble de règles et de syntaxe légèrement varié. Pour en savoir plus sur la syntaxe spécifique des différents moteurs de recherche, vous devrez visiter leurs pages d'aide respectives. Certains moteurs de recherche, comme Google, proposent une interface web pour les recherches avancées :  [Google Advanced Search](https://www.google.com/advanced_search) . D'autres fois, il est préférable d'apprendre la syntaxe par cœur, comme [Google Refine Web Searches](https://support.google.com/websearch/answer/2466433) , [DuckDuckGo Search Syntax](https://help.duckduckgo.com/duckduckgo-help-pages/results/syntax/) et [Bing Advanced Search Options](https://help.bing.microsoft.com/apex/index/18/en-US/10002) .

Les moteurs de recherche parcourent le Web jour et nuit pour indexer de nouvelles pages Web et de nouveaux fichiers. Parfois, cela peut conduire à l'indexation d'informations confidentielles. Voici des exemples d'informations confidentielles :

-   Documents à usage interne à l'entreprise
-   Feuilles de calcul confidentielles avec noms d'utilisateur, adresses e-mail et même mots de passe
-   Fichiers contenant des noms d'utilisateur
-   Répertoires sensibles
-   Numéro de version du service (dont certains peuvent être vulnérables et non corrigés)
-   Messages d'erreur

En combinant des recherches Google avancées avec des termes spécifiques, des documents contenant des informations sensibles ou des serveurs Web vulnérables peuvent être trouvés. Des sites Web tels que [Google Hacking Database](https://www.exploit-db.com/google-hacking-database) (GHDB) collectent ces termes de recherche et sont accessibles au public. Jetons un coup d'œil à certaines des requêtes GHDB pour voir si notre client a des informations confidentielles exposées via les moteurs de recherche. GHDB contient des requêtes dans les catégories suivantes :

-   Considérez [GHDB-ID : 6364](https://www.exploit-db.com/ghdb/6364) car il utilise la requête pour découvrir\
    les journaux Nginx et peut révéler des erreurs de configuration du serveur qui peuvent être exploitées.[](https://www.exploit-db.com/ghdb/6364)`intitle:"index of" "nginx.log"`
-   Fichiers contenant des noms d'utilisateur\
    Par exemple, [GHDB-ID : 7047](https://www.exploit-db.com/ghdb/7047) utilise le terme de recherche `intitle:"index of" "contacts.txt"`pour découvrir les fichiers qui divulguent des informations juteuses.
-   Répertoires sensibles\
    Par exemple, considérez [GHDB-ID: 6768](https://www.exploit-db.com/ghdb/6768) , qui utilise le terme de recherche `inurl:/certs/server.key`pour savoir si une clé RSA privée est exposée.
-   Détection du serveur Web\
    Considérez [GHDB-ID: 6876](https://www.exploit-db.com/ghdb/6876) , qui détecte les informations du serveur GlassFish à l'aide de la requête `intitle:"GlassFish Server - Server Running"`.
-   Fichiers vulnérables\
    Par exemple, nous pouvons essayer de localiser les fichiers PHP en utilisant la requête`intitle:"index of" "*.php"` , comme fourni par [GHDB-ID: 7786](https://www.exploit-db.com/ghdb/7786) .
-   Serveurs vulnérables\
    Par exemple, pour découvrir les consoles Web SolarWinds Orion, [GHDB-ID : 6728](https://www.exploit-db.com/ghdb/6728) utilise la requête `intext:"user name" intext:"orion core" -solarwinds.com`.
-   Messages d'erreur\
    De nombreuses informations utiles peuvent être extraites des messages d'erreur. Un exemple est [GHDB-ID: 5963](https://www.exploit-db.com/ghdb/5963) , qui utilise la requête `intitle:"index of" errors.log`pour trouver les fichiers journaux liés aux erreurs.

Vous devrez peut-être adapter ces requêtes Google à vos besoins, car les requêtes renverront les résultats de tous les serveurs Web qui correspondent aux critères et ont été indexés. Pour éviter les problèmes juridiques, il est préférable de s'abstenir d'accéder à des fichiers en dehors du champ d'application de votre accord juridique.

Nous vous recommandons de rejoindre la salle [Google Dorking](https://tryhackme.com/room/googledorking) pour des informations plus détaillées.

Nous allons maintenant explorer deux sources supplémentaires qui peuvent fournir des informations précieuses sans interagir avec notre cible :

-   Réseaux sociaux
-   Annonces d'emploi
![f94dadbbcf2c644230d6eb310e159ed5](https://github.com/dsgsec/Red-Team/assets/82456829/ac92e43e-604b-45f4-824c-043ca2dfa24f)


### Réseaux sociaux

Les sites Web de médias sociaux sont devenus très populaires non seulement pour un usage personnel, mais aussi pour un usage professionnel. Certaines plateformes de médias sociaux peuvent révéler des tonnes d'informations sur la cible. Cela est d'autant plus vrai que de nombreux utilisateurs ont tendance à trop partager des détails sur eux-mêmes et sur leur travail. Pour n'en nommer que quelques-uns, il vaut la peine de vérifier les points suivants :

-   LinkedIn
-   Twitter
-   Facebook
-   Instagram

Les sites Web de médias sociaux facilitent la collecte des noms des employés d'une entreprise donnée; de plus, dans certains cas, vous pourriez apprendre des informations spécifiques qui peuvent révéler des réponses à des questions de récupération de mot de passe ou obtenir des idées à inclure dans une liste de mots ciblée. Les messages du personnel technique peuvent révéler des détails sur les systèmes et les fournisseurs d'une entreprise. Par exemple, un ingénieur réseau qui a récemment reçu des certifications Juniper peut faire allusion à l'infrastructure réseau Juniper utilisée dans l'environnement de son employeur.

![cf84f21108b6aae75e1fa73018bf12db](https://github.com/dsgsec/Red-Team/assets/82456829/b65cf937-5077-4b40-a0dd-aaae8557c10a)


### Offres d'emploi

Les offres d'emploi peuvent également vous en dire beaucoup sur une entreprise. En plus de révéler les noms et les adresses e-mail, les offres d'emploi pour les postes techniques pourraient donner un aperçu des systèmes et de l'infrastructure de l'entreprise cible. Les offres d'emploi populaires peuvent varier d'un pays à l'autre. Assurez-vous de vérifier les sites d'offres d'emploi dans les pays où votre client publierait ses annonces. De plus, il vaut toujours la peine de consulter leur site Web pour toute offre d'emploi et de voir si cela peut divulguer des informations intéressantes.

Notez que la  [Wayback Machine](https://archive.org/web/) peut être utile pour récupérer les versions précédentes d'une page d'offre d'emploi sur le site de votre client.
