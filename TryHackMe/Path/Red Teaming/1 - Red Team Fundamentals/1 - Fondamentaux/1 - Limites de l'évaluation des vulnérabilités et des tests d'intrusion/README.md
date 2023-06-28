Évaluations de la vulnérabilité

Il s'agit de la forme la plus simple d'évaluation de la sécurité, et son objectif principal est d'identifier autant de vulnérabilités dans autant de systèmes du réseau que possible. À cette fin, des concessions peuvent être faites pour atteindre efficacement cet objectif. Par exemple, la machine de l'attaquant peut être inscrite sur la liste blanche des solutions de sécurité disponibles pour éviter d'interférer avec le processus de découverte des vulnérabilités. Cela a du sens puisque l'objectif est d'examiner chaque hôte sur le réseau et d'évaluer sa sécurité individuellement tout en fournissant le plus d'informations à l'entreprise sur où concentrer ses efforts de remédiation.

Pour résumer, une évaluation des vulnérabilités se concentre sur l'analyse des vulnérabilités des hôtes en tant qu'entités individuelles afin que les failles de sécurité puissent être identifiées  et que des mesures de sécurité efficaces puissent être déployées pour protéger le réseau de manière prioritaire. La plupart des travaux peuvent être effectués avec des outils automatisés et effectués par des opérateurs sans nécessiter beaucoup de connaissances techniques.

Par exemple, si vous deviez exécuter une évaluation des vulnérabilités sur un réseau, vous essaieriez normalement d'analyser autant d'hôtes que possible, mais n'essaieriez pas d'exploiter les vulnérabilités du tout :

![ee93fb3de94655a02ae7a6b69676944b](https://github.com/dsgsec/Red-Team/assets/82456829/5dd34ead-4602-4408-9536-58a3d800a7a5)

### Essais de pénétration

En plus d'analyser chaque hôte à la recherche de vulnérabilités, nous devons souvent comprendre leur impact sur notre réseau dans son ensemble. Les tests d'intrusion s'ajoutent aux évaluations de vulnérabilité en permettant au pentester d'explorer l'impact d'un attaquant sur l'ensemble du réseau en effectuant des étapes supplémentaires qui incluent :

-   Essayez d'  exploiter  les vulnérabilités trouvées sur chaque système. Ceci est important car parfois une vulnérabilité peut exister dans un système, mais les contrôles compensatoires en place empêchent efficacement son exploitation. Cela nous permet également de tester si nous pouvons utiliser les vulnérabilités détectées pour compromettre un hôte donné.
-   Effectuez  des tâches de post-exploitation  sur n'importe quel hôte compromis, ce qui nous permet de déterminer si nous pouvons en extraire des informations utiles ou si nous pourrions les utiliser pour pivoter vers d'autres hôtes qui n'étaient pas auparavant accessibles d'où nous nous trouvons.

Les tests d'intrusion peuvent commencer par analyser les vulnérabilités comme une évaluation de vulnérabilité régulière, mais fournir des informations supplémentaires sur la façon dont un attaquant peut enchaîner les vulnérabilités pour atteindre des objectifs spécifiques. Bien que son objectif reste d' identifier les vulnérabilités et d'établir des mesures pour protéger le réseau, il considère également le réseau dans son ensemble et la manière dont un attaquant pourrait profiter des interactions entre ses composants.

Si nous devions effectuer un test de pénétration en utilisant le même exemple de réseau qu'auparavant, en plus d'analyser tous les hôtes du réseau à la recherche de vulnérabilités, nous essaierions de confirmer s'ils peuvent être exploités afin de montrer l'impact qu'un attaquant pourrait avoir sur le réseau: 

![da055515377a73c12f0dcef197c1ac8c](https://github.com/dsgsec/Red-Team/assets/82456829/a57c1d5a-487d-47b1-9d79-60f5a26074bb)


En analysant comment un attaquant pourrait se déplacer sur notre réseau, nous obtenons également un aperçu de base sur les contournements possibles des mesures de sécurité et notre capacité à  détecter  un véritable acteur de menace dans une certaine mesure, limitée car la portée d'un test d'intrusion est généralement étendue et les testeurs d'intrusion ne se soucient pas beaucoup d'être bruyants ou de générer de nombreuses alertes sur les dispositifs de sécurité, car les contraintes de temps sur de tels projets nous obligent souvent à vérifier le réseau en peu de temps.

### Menaces persistantes avancées et pourquoi les pentesting réguliers ne suffisent pas

Alors que les engagements de sécurité conventionnels que nous avons mentionnés couvrent la découverte de la plupart des vulnérabilités techniques, il existe des limites à ces processus et à la mesure dans laquelle ils peuvent préparer efficacement une entreprise contre un véritable attaquant. Ces limitations incluent :

![f9c16ff60c1412eefd77f12253643bab](https://github.com/dsgsec/Red-Team/assets/82456829/0f5ac7f9-d41d-4ed9-8f46-3129ad803e8c)

Par conséquent, certains aspects des tests d'intrusion peuvent différer considérablement d'une véritable attaque, comme :

-   Les tests de pénétration sont BRUYANTS :  Habituellement, les pentesters ne feront pas beaucoup d'efforts pour essayer de passer inaperçus. Contrairement aux vrais attaquants, cela ne les dérange pas d'être faciles à détecter, car ils ont été engagés pour trouver autant de vulnérabilités que possible dans autant d'hôtes que possible.
-   Les vecteurs d'attaque non techniques peuvent être négligés : les attaques basées sur l'ingénierie sociale ou les intrusions physiques ne sont généralement pas incluses dans ce qui est testé.
-   Assouplissement des mécanismes de sécurité :  lors d'un test d'intrusion régulier, certains mécanismes de sécurité peuvent être temporairement désactivés ou assouplis pour l'équipe de pentesting au profit de l'efficacité. Bien que cela puisse sembler contre-intuitif, il est essentiel de se rappeler que les pentesters ont un temps limité pour vérifier le réseau. Par conséquent, il est généralement souhaitable de ne pas perdre son temps à chercher des moyens exotiques de contourner IDS/IPS, WAF, la tromperie d'intrusion ou d'autres mesures de sécurité, mais plutôt de se concentrer sur l'examen des vulnérabilités de l'infrastructure technologique critique.

D'un autre côté, les vrais attaquants ne suivront pas de code éthique et sont pour la plupart sans restriction dans leurs actions. De nos jours, les acteurs les plus importants de la menace sont connus sous le nom de  Advanced Persistent Threats ( APT ) , qui sont des groupes d'attaquants hautement qualifiés, généralement parrainés par des nations ou des groupes criminels organisés. Ils ciblent principalement les infrastructures critiques, les organisations financières et les institutions gouvernementales. Ils sont appelés persistants car les opérations de ces groupes peuvent rester non détectées sur des réseaux compromis pendant de longues périodes.\
Si une entreprise est touchée par une APT , serait-elle prête à réagir efficacement? Pourraient-ils détecter les méthodes utilisées pour obtenir et maintenir l'accès à leurs réseaux si l'attaquant s'y trouve depuis plusieurs mois ? Que se passe-t-il si l'accès initial a été obtenu parce que John de la comptabilité a ouvert une pièce jointe suspecte ? Et si un exploit zero-day était impliqué ? Les précédents tests d'intrusion nous préparent-ils à cela ?\
Pour fournir une approche plus réaliste de la sécurité, les engagements de l'équipe rouge sont nés.
