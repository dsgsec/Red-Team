Présentation des dépassements de tampon
=========================

* * * * *

Les débordements de tampon sont devenus moins courants dans le monde d'aujourd'hui, car les compilateurs modernes ont intégré des protections de mémoire qui rendent difficile l'apparition accidentelle de bogues de corruption de mémoire. Cela étant dit, les langages comme C ne vont pas disparaître de sitôt et ils sont prédominants dans les logiciels embarqués et IOT (Internet of Things). L'un de mes dépassements de mémoire tampon quelque peu récents préférés était [CVE-2021-3156](https://blog.qualys.com/vulnerabilities-threat-research/2021/01/26/cve-2021-3156-heap-based-buffer -overflow-in-sudo-baron-samedit), qui était un débordement de tampon basé sur le tas dans sudo.

Ces attaques ne se limitent pas aux binaires, un grand nombre de dépassements de mémoire tampon se produisent dans les applications Web, en particulier les appareils intégrés qui utilisent des serveurs Web personnalisés. Un bon exemple est [CVE-2017-12542](https://www.bleepingcomputer.com/news/security/you-can-bypass-authentication-on-hpe-ilo4-servers-with-29-a-characters/ ) avec les appareils de gestion HP iLO (Integrated Lights Out). Le simple fait d'envoyer 29 caractères dans un paramètre d'en-tête HTTP a provoqué un débordement de tampon qui a contourné la connexion. J'aime cet exemple car il n'y a pas besoin d'une charge utile réelle sur laquelle vous en saurez plus plus tard puisque le système "a échoué à s'ouvrir" en atteignant une erreur.

En bref, les débordements de tampon sont causés par un code de programme incorrect, qui ne peut pas traiter correctement de trop grandes quantités de données par le CPU et peut donc manipuler le traitement du CPU. Supposons que trop de données soient écrites dans une mémoire réservée `buffer` ou `pile` qui n'est pas limitée, par exemple. Dans ce cas, des registres spécifiques seront écrasés, ce qui peut permettre l'exécution de code.

Un dépassement de mémoire tampon peut provoquer le plantage du programme, corrompre les données ou endommager les structures de données lors de l'exécution du programme. Le dernier d'entre eux peut écraser l' `adresse de retour` du programme spécifique avec des données arbitraires, permettant à un attaquant d'exécuter des commandes avec les `privilèges du processus` vulnérables au débordement de la mémoire tampon en transmettant du code machine arbitraire. Ce code est généralement destiné à nous donner un accès plus pratique au système pour l'utiliser à nos propres fins. De tels débordements de mémoire tampon dans les serveurs communs et les vers Internet exploitent également les logiciels clients.

Une cible particulièrement populaire sur les systèmes Unix est l'accès root, qui nous donne toutes les autorisations pour accéder au système. Cependant, comme cela est souvent mal compris, cela ne signifie pas qu'un débordement de buffer qui n'entraîne "que" les privilèges d'un utilisateur standard est inoffensif. Obtenir l'accès root convoité est souvent beaucoup plus facile si vous avez déjà des privilèges d'utilisateur.

Les débordements de buffer, en plus des imprudences de programmation, sont principalement rendus possibles par les systèmes informatiques basés sur l'architecture Von-Neumann.

La cause la plus importante des dépassements de mémoire tampon est l'utilisation de langages de programmation qui ne surveillent pas automatiquement les limites de la mémoire tampon ou de la pile pour empêcher le dépassement de mémoire tampon (basé sur la pile). Il s'agit notamment des langages `C` et `C++` , qui mettent l'accent sur les performances et ne nécessitent pas de surveillance.

Pour cette raison, les développeurs sont obligés de définir eux-mêmes ces zones dans le code de programmation, ce qui augmente considérablement la vulnérabilité. Ces zones sont souvent laissées indéfinies à des fins de test ou en raison d'une négligence. Même s'ils étaient utilisés à des fins de test, ils auraient pu être oubliés à la fin du processus de développement.

Cependant, tous les environnements d'application ne présenteront probablement pas une condition de débordement de la mémoire tampon. Par exemple, une application Java autonome est moins susceptible d'être comparée à d'autres en raison de la façon dont Java gère la gestion de la mémoire. Java utilise une technique de "garbage collection" pour gérer la mémoire, ce qui permet d'éviter les conditions de débordement de la mémoire tampon.

Développement d'exploit - Introduction
================================

* * * * *

Le développement d'exploits intervient dans la `phase d'exploitation` après l'identification d'un logiciel spécifique et même de ses versions. L'objectif de la phase d'exploitation est d'utiliser les informations trouvées et leur analyse pour exploiter les moyens potentiels d'obtenir une interaction et/ou un accès au système cible.

Développer nos propres exploits peut être très complexe et nécessite une compréhension approfondie des opérations du processeur et des fonctions du logiciel qui nous servent de cible. De nombreux exploits sont écrits dans différents langages de programmation. L'un des langages de programmation les plus populaires pour cela est `Python` car il est facile à comprendre et à écrire. Dans ce module, nous nous concentrerons sur les techniques de base pour le développement d'exploits, car une `compréhension fondamentale` doit être développée avant de pouvoir traiter les différents mécanismes de sécurité de la mémoire.

Avant d'exécuter des exploits, nous devons comprendre ce qu'est un exploit. Un exploit est un code qui amène le service à effectuer une opération souhaitée en abusant de la vulnérabilité trouvée. Ces codes servent souvent de `preuve de concept` (`POC`) dans nos rapports.

Il existe deux types d'exploits. L'un est inconnu (exploits "0 jours") et l'autre est connu (exploits "N-jours").

* * * * *

#### Exploits du jour 0

Un `0-day exploit` est un code qui exploite une vulnérabilité nouvellement identifiée dans une application spécifique. La vulnérabilité n'a pas besoin d'être publique dans l'application. Le danger avec de tels exploits est que si les développeurs de cette application ne sont pas informés de la vulnérabilité, ils persisteront probablement avec de nouvelles mises à jour.

* * * * *

#### Exploits N-Day

Si la vulnérabilité est publiée et informe les développeurs, ils auront encore besoin de temps pour écrire un correctif pour les prévenir au plus vite. Lorsqu'ils sont publiés, ils parlent d'"exploits N-day", comptant les jours entre la publication de l'exploit et une attaque sur les systèmes non corrigés.

De plus, ces exploits peuvent être divisés en quatre catégories différentes :

- "locale"
- 'Télécommande'
- "DoS"
- "Application Web"

* * * * *

#### Exploits locaux

Les exploits locaux / Privilege Escalation exploits peuvent être exécutés lors de l'ouverture d'un fichier. Cependant, la condition préalable à cela est que le logiciel local contienne une faille de sécurité. Souvent, un exploit local (par exemple, dans un document PDF ou en tant que macro dans un fichier Word ou Excel) essaie d'abord d'exploiter les failles de sécurité du programme avec lequel le fichier a été importé pour obtenir un niveau de privilège plus élevé et ainsi charger et exécuter `malveillants code` / `shellcode` dans le système d'exploitation. L'action réelle effectuée par l'exploit est appelée `charge utile`.

* * * * *

#### Exploits distants

Les exploits distants exploitent très souvent la vulnérabilité de débordement de la mémoire tampon pour faire fonctionner la charge utile sur le système. Ce type d'exploits diffère des exploits locaux car ils peuvent être exécutés sur le réseau pour effectuer l'opération souhaitée.

* * * * *

#### Exploits DoS

Les exploits DoS (`Denial of Service`) sont des codes qui empêchent d'autres systèmes de fonctionner, c'est-à-dire provoquent un plantage d'un logiciel individuel ou de l'ensemble du système.

* * * * *

#### Exploits d'applications Web

Un exploit d'application Web utilise une vulnérabilité dans un tel logiciel. De telles vulnérabilités peuvent, par exemple, permettre une injection de commande sur l'application elle-même ou la base de données sous-jacente.

