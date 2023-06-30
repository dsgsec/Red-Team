Infrastructure de phishing
==========================

Une certaine infrastructure devra être mise en place pour lancer une campagne de phishing réussie.

Nom de domaine:

Vous devrez enregistrer soit un nom de domaine authentique, soit un nom qui imite l'identité d'un autre domaine. Voir la tâche 5 pour plus de détails sur la façon de créer le nom de domaine parfait.

Certificats SSL/TLS :

La création de certificats SSL/TLS pour le nom de domaine que vous avez choisi ajoutera une couche supplémentaire d'authenticité à l'attaque.

Serveur de messagerie/compte :

Vous devrez soit configurer un serveur de messagerie, soit vous inscrire auprès d'un fournisseur de messagerie SMTP .

Enregistrements DNS :

La configuration d'enregistrements DNS tels que SPF, DKIM, DMARC améliorera la délivrabilité de vos e-mails et garantira qu'ils arrivent dans la boîte de réception plutôt que dans le dossier spam.

Serveur Web:

Vous devrez configurer des serveurs Web ou acheter un hébergement Web auprès d'une entreprise pour héberger vos sites Web de phishing. L'ajout de SSL/TLS aux sites Web leur donnera une couche supplémentaire d'authenticité.

Analytique:

Lorsqu'une campagne de phishing fait partie d'un engagement d'équipe rouge, il est plus important de conserver les informations d'analyse. Vous aurez besoin de quelque chose pour garder une trace des e-mails qui ont été envoyés, ouverts ou cliqués. Vous devrez également les combiner avec les informations de vos sites Web de phishing pour lesquels les utilisateurs ont fourni des informations personnelles ou téléchargé des logiciels. 

Automatisation et logiciels utiles :

Certaines des infrastructures ci-dessus peuvent être rapidement automatisées en utilisant les outils ci-dessous.

GoPhish - ( Cadre de phishing open-source ) - [getgophish.com](https://getgophish.com/)

GoPhish est un cadre basé sur le Web pour faciliter la mise en place de campagnes de phishing. GoPhish vous permet de stocker les paramètres de votre serveur SMTP pour l'envoi d'e-mails, dispose d'un outil Web pour créer des modèles d'e-mails à l'aide d'un simple éditeur WYSIWYG (What You See Is What You Get). Vous pouvez également planifier l'envoi des e-mails et disposer d'un tableau de bord d'analyse indiquant le nombre d'e-mails envoyés, ouverts ou cliqués.

La tâche suivante vous expliquera comment lancer une campagne de phishing à l'aide de ce logiciel.

SET - (boîte à outils d'ingénierie sociale) - [trustsec.com](https://www.trustedsec.com/tools/the-social-engineer-toolkit-set/)

La boîte à outils d'ingénierie sociale contient une multitude d'outils, mais certains des plus importants pour le phishing sont la capacité de créer des attaques de harponnage et de déployer de fausses versions de sites Web courants pour inciter les victimes à entrer leurs informations d'identification.

![9f0f844ee415917d739effc9e0234cc4](https://github.com/dsgsec/Red-Team/assets/82456829/98d8460b-7e46-45e2-bfa9-cc65e8ba71a7)
