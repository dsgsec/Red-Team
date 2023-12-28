![51cfb2e8bf86ff49d820dd45b78ad26c](https://github.com/dsgsec/Red-Team/assets/82456829/4b684b69-970b-4762-b5e5-65f51f400e02)
Lorsque vous obtenez un « shell » sur le système cible, vous avez généralement une connaissance très basique du système. S'il s'agit d'un serveur, vous savez déjà quel service vous avez exploité ; cependant, vous ne connaissez pas nécessairement d'autres détails, tels que les noms d'utilisateur ou les partages réseau. Par conséquent, la coquille ressemblera à une « pièce sombre » où vous aurez une connaissance incomplète et vague de ce qui vous entoure. En ce sens, le dénombrement vous aide à dresser un tableau plus complet et plus précis.

Le but du dénombrement post-exploitation est de recueillir autant d'informations sur le système et son réseau. Le système exploité peut être un ordinateur de bureau/ordinateur portable d'entreprise ou un serveur. Notre objectif est de collecter les informations qui nous permettraient de pivoter vers d'autres systèmes du réseau ou de piller le système actuel. Certaines des informations que nous souhaitons recueillir comprennent :

-   Utilisateurs et groupes
-   Noms d'hôtes
-   Tables de routage
-   Partages réseau
-   Services réseau
-   Applications et bannières
-   Configurations de pare-feu
-   Paramètres de service et configurations d'audit
-   Détails SNMP et DNS
-   Recherche d'informations d'identification (enregistrées sur les navigateurs Web ou les applications clientes)

Il n'y a aucun moyen de lister tout ce sur quoi nous pourrions tomber. Par exemple, nous pourrions trouver des clés SSH qui pourraient nous donner accès à d'autres systèmes. Dans l'authentification basée sur la clé SSH, nous générons une paire de clés SSH (clés publiques et privées) ; la clé publique est installée sur un serveur. Par conséquent, le serveur ferait confiance à tout système pouvant prouver qu'il connaît la clé privée associée.

De plus, nous pourrions tomber sur des données sensibles enregistrées parmi les documents ou les répertoires du bureau de l'utilisateur. Pensez que quelqu'un pourrait conserver un `passwords.txt`ou `passwords.xlsx`à la place d'un gestionnaire de mots de passe approprié. Le code source peut également contenir des clés et des mots de passe qui traînent, surtout si le code source n'est pas destiné à être rendu public.
