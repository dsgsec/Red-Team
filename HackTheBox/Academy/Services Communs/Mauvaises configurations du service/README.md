# Mauvaises configurations du service

Les erreurs de configuration se produisent généralement lorsqu'un administrateur système, un support technique ou un développeur ne configure pas correctement le cadre de sécurité d'une application, d'un site Web, d'un ordinateur de bureau ou d'un serveur, ce qui ouvre des voies ouvertes dangereuses pour les utilisateurs non autorisés. Explorons quelques-unes des erreurs de configuration les plus courantes des services courants.

<hr>

## Authentication
Au cours des années précédentes (bien que nous le voyions encore parfois lors des évaluations), il était courant que les services incluent des informations d'identification par défaut (nom d'utilisateur et mot de passe). Cela pose un problème de sécurité car de nombreux administrateurs laissent les informations d'identification par défaut inchangées. De nos jours, la plupart des logiciels demandent aux utilisateurs de configurer des informations d'identification lors de l'installation, ce qui est mieux que les informations d'identification par défaut. Cependant, gardez à l'esprit que nous trouverons toujours des fournisseurs utilisant des informations d'identification par défaut, en particulier sur les applications plus anciennes.

Même lorsque le service ne dispose pas d'un ensemble d'informations d'identification par défaut, un administrateur peut utiliser des mots de passe faibles ou aucun mot de passe lors de la configuration des services avec l'idée qu'il changera le mot de passe une fois le service configuré et en cours d'exécution.

En tant qu'administrateurs, nous devons définir des politiques de mot de passe qui s'appliquent aux logiciels testés ou installés dans notre environnement. Les administrateurs doivent être tenus de respecter une complexité minimale des mots de passe pour éviter les combinaisons d'utilisateurs et de mots de passe telles que :

```
admin:admin
admin:password
admin:<blank>
root:12345678
administrator:Password
```

Une fois que nous avons saisi la bannière de service, l'étape suivante devrait consister à identifier les informations d'identification par défaut possibles. S'il n'y a pas d'informations d'identification par défaut, nous pouvons essayer les combinaisons de nom d'utilisateur et de mot de passe faibles répertoriées ci-dessus.

### Authentification anonyme
Une autre mauvaise configuration qui peut exister dans les services communs est l'authentification anonyme. Le service peut être configuré pour autoriser l'authentification anonyme, permettant à toute personne disposant d'une connectivité réseau au service sans être invitée à s'authentifier.

### Droits d'accès mal configurés
Imaginons que nous récupérions les informations d'identification d'un utilisateur dont le rôle est de télécharger des fichiers sur le serveur FTP mais qui a le droit de lire tous les documents FTP. La possibilité est infinie, selon ce qui se trouve dans le serveur FTP. Nous pouvons trouver des fichiers contenant des informations de configuration pour d'autres services, des informations d'identification en texte brut, des noms d'utilisateur, des informations exclusives et des informations personnellement identifiables (PII).

Les droits d'accès mal configurés surviennent lorsque les comptes d'utilisateurs ont des autorisations incorrectes. Le plus gros problème pourrait être de donner aux personnes situées en bas de la chaîne de commandement un accès à des informations privées que seuls les gestionnaires ou les administrateurs devraient avoir.

Les administrateurs doivent planifier leur stratégie de droits d'accès, et il existe des alternatives telles que le contrôle d'accès basé sur les rôles (RBAC), les listes de contrôle d'accès (ACL). Si nous voulons plus de détails sur les avantages et les inconvénients de chaque méthode, nous pouvons lire Choisir la meilleure stratégie de contrôle d'accès par Warren Parad de Authress.

## Valeurs par défaut inutiles
La configuration initiale des appareils et des logiciels peut inclure, mais sans s'y limiter, les paramètres, les fonctionnalités, les fichiers et les informations d'identification. Ces valeurs par défaut visent généralement la convivialité plutôt que la sécurité. Le laisser par défaut n'est pas une bonne pratique de sécurité pour un environnement de production. Les valeurs par défaut inutiles sont les paramètres que nous devons modifier pour sécuriser un système en réduisant sa surface d'attaque.

Nous pourrions tout aussi bien livrer les informations personnelles de notre entreprise sur un plateau d'argent si nous prenons la route facile et acceptons les paramètres par défaut lors de la configuration d'un logiciel ou d'un appareil pour la première fois. En réalité, les attaquants peuvent obtenir des identifiants d'accès pour un équipement spécifique ou abuser d'un paramètre faible en effectuant une courte recherche sur Internet.

Les erreurs de configuration de sécurité font partie de la liste OWASP Top 10. Examinons ceux liés aux valeurs par défaut :

+ Des fonctionnalités inutiles sont activées ou installées (par exemple, des ports, des services, des pages, des comptes ou des privilèges inutiles).
+ Les comptes par défaut et leurs mots de passe sont toujours activés et inchangés.
+ La gestion des erreurs révèle des traces de pile ou d'autres messages d'erreur trop informatifs pour les utilisateurs.
+ Pour les systèmes mis à niveau, les dernières fonctionnalités de sécurité sont désactivées ou ne sont pas configurées de manière sécurisée.

## Prévention des erreurs de configuration
Une fois que nous avons compris notre environnement, la stratégie la plus simple pour contrôler les risques consiste à verrouiller l'infrastructure la plus critique et à n'autoriser que le comportement souhaité. Toute communication qui n'est pas requise par le programme doit être désactivée. Cela peut inclure des éléments tels que :

+ Les interfaces d'administration doivent être désactivées.
Le débogage est désactivé.
Désactivez l'utilisation des noms d'utilisateur et des mots de passe par défaut.
+ Configurez le serveur pour empêcher l'accès non autorisé, la liste des répertoires et d'autres problèmes.
+ Exécutez régulièrement des analyses et des audits pour vous aider à découvrir les futures erreurs de configuration ou les correctifs manquants.

Le Top 10 OWASP fournit une section sur la façon de sécuriser les processus d'installation :

+ Un processus de durcissement reproductible permet de déployer rapidement et facilement un autre environnement correctement verrouillé. Les environnements de développement, d'assurance qualité et de production doivent tous être configurés de manière identique, avec des informations d'identification différentes utilisées dans chaque environnement. De plus, ce processus devrait être automatisé afin de minimiser l'effort requis pour mettre en place un nouvel environnement sécurisé.

+ Une plate-forme minimale sans fonctionnalités, composants, documentation et exemples inutiles. Supprimez ou n'installez pas les fonctionnalités et les frameworks inutilisés.

+ Une tâche pour examiner et mettre à jour les configurations appropriées à toutes les notes de sécurité, mises à jour et correctifs dans le cadre du processus de gestion des correctifs (voir A06:2021-Composants vulnérables et obsolètes). 

+ Vérifiez les autorisations de stockage dans le cloud (par exemple, les autorisations de compartiment S3).

+ Une architecture d'application segmentée fournit une séparation efficace et sécurisée entre les composants ou les locataires, avec la segmentation, la conteneurisation ou les groupes de sécurité cloud (ACL).

+ Envoi de directives de sécurité aux clients, par exemple, des en-têtes de sécurité.

+ Un processus automatisé pour vérifier l'efficacité des configurations et des paramètres dans tous les environnements.