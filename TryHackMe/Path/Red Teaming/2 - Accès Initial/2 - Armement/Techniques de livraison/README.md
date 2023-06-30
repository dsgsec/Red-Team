Techniques de livraison
=========================

Les techniques de livraison sont l'un des facteurs importants pour obtenir l'accès initial. Ils doivent avoir l'air professionnels, légitimes et convaincants pour la victime afin de donner suite au contenu.

![54108dbd9d1c3d64fb86f2ad04b5949e](https://github.com/dsgsec/Red-Team/assets/82456829/a72398f7-51b7-416d-97b8-5789f3443428)

Livraison par e-mail

Il s'agit d'une méthode courante à utiliser pour envoyer la charge utile en envoyant un e-mail de phishing avec un lien ou une pièce jointe. Pour plus d'informations, rendez-vous [ici](https://attack.mitre.org/techniques/T1566/001/) . Cette méthode attache un fichier malveillant qui pourrait être du type mentionné précédemment. L'objectif est de convaincre la victime de visiter un site Web malveillant ou de télécharger et d'exécuter le fichier malveillant pour obtenir un accès initial au réseau ou à l'hôte de la victime.

Les équipes rouges devraient avoir leur propre infrastructure à des fins de phishing. En fonction de l'exigence d'engagement de l'équipe rouge, cela nécessite la configuration de diverses options au sein du serveur de messagerie, notamment DomainKeys Identified Mail ( DKIM ), Sender Policy Framework (SPF) et l'enregistrement DNS Pointer (PTR).

Les équipes rouges pourraient également utiliser des services de messagerie tiers tels que Google Gmail, Outlook, Yahoo et d'autres ayant une bonne réputation.

Une autre méthode intéressante consisterait à utiliser un compte de messagerie compromis au sein d'une entreprise pour envoyer des e-mails de phishing au sein de l'entreprise ou à d'autres. L'e-mail compromis pourrait être piraté par hameçonnage ou par d'autres techniques telles que les attaques par pulvérisation de mot de passe.

![08a3f660501cf5171277534e40aa96b8](https://github.com/dsgsec/Red-Team/assets/82456829/754714e0-4818-413b-be90-c3797153be5b)

### Livraison Web

Une autre méthode consiste à héberger des charges utiles malveillantes sur un serveur Web contrôlé par les red teamers. Le serveur Web doit suivre les directives de sécurité telles qu'un enregistrement et une réputation propres de son nom de domaine et de son certificat TLS (Transport Layer Security). Pour plus d'informations, rendez-vous [ici](https://attack.mitre.org/techniques/T1189/) .

Cette méthode comprend d'autres techniques telles que l'ingénierie sociale permettant à la victime de visiter ou de télécharger le fichier malveillant. Un raccourcisseur d'URL peut être utile lors de l'utilisation de cette méthode.

Dans cette méthode, d'autres techniques peuvent être combinées et utilisées. L'attaquant peut profiter d'exploits zero-day tels que l'exploitation de logiciels vulnérables comme Java ou des navigateurs pour les utiliser dans des e-mails de phishing ou des techniques de diffusion Web pour accéder à la machine victime.

![ff8ca3c104fa32e30603ecf97ee0d72e](https://github.com/dsgsec/Red-Team/assets/82456829/e8e64409-d71f-4d75-a7b6-5212467d4b42)

Livraison USB

Cette méthode nécessite que la victime branche physiquement la clé USB malveillante. Cette méthode pourrait être efficace et utile lors de conférences ou d'événements où l'adversaire peut distribuer l'USB. Pour plus d'informations sur la livraison USB, rendez-vous [ici](https://attack.mitre.org/techniques/T1091/) .

Souvent, les organisations établissent des politiques strictes telles que la désactivation de l'utilisation de l'USB dans leur environnement d'organisation à des fins de sécurité. Alors que d'autres organisations l'autorisent dans l'environnement cible.

Les attaques USB courantes utilisées pour militariser les périphériques USB incluent [Rubber Ducky](https://shop.hak5.org/products/usb-rubber-ducky-deluxe) et [USBHarpoon](https://www.minitool.com/news/usbharpoon.html) , le câble USB de chargement, tel que  [le câble O.MG ](https://shop.hak5.org/products/omg-cable).
