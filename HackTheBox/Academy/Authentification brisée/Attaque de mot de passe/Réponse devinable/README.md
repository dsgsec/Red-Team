Réponses devinables
===================

* * * * *

Souvent, les applications Web authentifient les utilisateurs qui ont perdu leur mot de passe en leur demandant de répondre à une ou plusieurs questions. Ces questions, généralement présentées à l'utilisateur lors de la phase d'inscription, sont pour la plupart codées en dur et ne peuvent pas être choisies par lui. Ils sont donc assez génériques.

En supposant que nous ayons trouvé une telle fonctionnalité sur un site Web cible, nous devrions essayer d'en abuser pour contourner l'authentification. Dans ces cas, le problème, ou plutôt le point faible, n'est pas la fonction en soi mais la prévisibilité des questions et les utilisateurs ou employés eux-mêmes. Il est courant de trouver des questions comme celles ci-dessous.

-   " `What is your mother's maiden name?`"

-   " `What city were you born in?`"

Le premier pourrait être trouvé en utilisant `OSINT`, tandis que la réponse au second pourrait être identifiée à nouveau en utilisant `OSINT`ou via une attaque par force brute. Certes, répondre aux deux questions pourrait être effectué sans en savoir beaucoup sur l'utilisateur cible.

![](https://academy.hackthebox.com/storage/modules/80/10-registration_question.png)

Nous déconseillons l'utilisation de réponses de sécurité car même lorsqu'une application permet aux utilisateurs de choisir leurs questions, les réponses peuvent toujours être prévisibles en raison de la négligence des utilisateurs. Pour augmenter le niveau de sécurité, une application Web doit répéter la première question jusqu'à ce que l'utilisateur réponde correctement. De cette façon, un attaquant qui n'a pas la chance de connaître la première réponse ou qui tombe sur une question qui peut être facilement brutalisée au premier coup ne peut pas essayer le second. Lorsque nous trouvons une application Web qui continue de faire tourner les questions, nous devons les collecter pour identifier la force brute la plus facile, puis monter l'attaque.

Scraper un site Web peut être assez compliqué car certaines applications Web brouillent les données de formulaire ou utilisent JavaScript pour remplir les formulaires. Certains autres conservent tous les détails de la question stockés côté serveur. Par conséquent, nous devons créer un script de force brute utilisant un assistant, comme lorsqu'un jeton anti-CSRF est présent. Nous avons préparé une page Web de base qui tourne les questions et un modèle Python que vous pouvez utiliser pour expérimenter cette attaque. Vous pouvez télécharger le fichier PHP [ici](https://academy.hackthebox.com/storage/modules/80/scripts/predictable_questions_php.txt) et le code Python [ici](https://academy.hackthebox.com/storage/modules/80/scripts/predictable_questions_py.txt) . Prenez le temps de bien comprendre le fonctionnement de l'application web. Nous vous suggérons d'essayer manuellement, puis d'écrire votre propre script. N'utilisez le script de quelqu'un d'autre qu'en dernier recours.
