Rédiger des e-mails de phishing convaincants
=============================================

Nous devons travailler avec trois éléments concernant les e-mails de phishing : l'adresse e-mail de l'expéditeur, le sujet et le contenu.

L'adresse des expéditeurs :

Idéalement, l'adresse de l'expéditeur proviendrait d'un nom de domaine qui usurpe une marque importante, un contact connu ou un collègue. Voir la tâche Choisir un domaine d'hameçonnage ci-dessous pour plus d'informations à ce sujet.

Pour trouver les marques ou les personnes avec lesquelles une victime interagit, vous pouvez utiliser les tactiques OSINT (Open Source Intelligence). Par exemple:

-   Observez leur compte de médias sociaux pour toutes les marques ou amis avec qui ils parlent.
-   Recherche sur Google du nom de la victime et de son emplacement approximatif pour tous les avis que la victime peut avoir laissés sur des entreprises ou des marques locales.
-   Consulter le site Web de l'entreprise de la victime pour trouver des fournisseurs.
-   En regardant LinkedIn pour trouver des collègues de la victime.

L'objet:

Vous devez définir le sujet sur quelque chose d'assez urgent, inquiétant ou qui pique la curiosité de la victime, afin qu'elle ne l'ignore pas et qu'elle agisse rapidement.

Des exemples de ceci pourraient être :

1.  Votre compte a été compromis.
2.  Votre colis a été expédié/expédié.
3.  Informations sur la paie du personnel (ne pas transmettre !)
4.  Vos photos ont été publiées.

Le contenu:

Si vous usurpez l'identité d'une marque ou d'un fournisseur, il serait pertinent de rechercher leurs modèles d'e-mails standard et leur image de marque (style, images de logo, approbations, etc.) et de faire en sorte que votre contenu ressemble au leur, afin que la victime n'attende rien. Si vous vous faites passer pour un contact ou un collègue, il pourrait être avantageux de le contacter ; d'abord, ils peuvent avoir une image de marque dans leur modèle, avoir une signature électronique particulière ou même quelque chose de petit comme la façon dont ils se réfèrent à eux-mêmes, par exemple, quelqu'un peut s'appeler Dorothy et son e-mail est dorothy@company.thm. Pourtant, dans leur signature, il pourrait être écrit "Best Regards, Dot". Apprendre ces petites choses peut parfois avoir des effets psychologiques assez dramatiques sur la victime et la convaincre davantage d'ouvrir et d'agir sur l'e-mail.

Si vous avez configuré un site Web frauduleux pour collecter des données ou distribuer des logiciels malveillants, les liens vers celui-ci doivent être déguisés en utilisant le [texte d'ancrage](https://en.wikipedia.org/wiki/Anchor_text) et en le remplaçant soit par un texte indiquant "Cliquez ici", soit par un lien d'apparence correcte qui reflète l'entreprise que vous usurpez, par exemple :

`<a href="http://spoofsite.thm">Click Here</a>`

`<a href="http://spoofsite.thm">https://onlinebank.thm</a>`
