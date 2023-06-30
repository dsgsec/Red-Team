GoPhish
========

Cette tâche vous guidera tout au long de la configuration de GoPhish, de l'envoi d'une campagne de phishing et de la capture des informations d'identification de l'utilisateur à partir d'un site Web frauduleux.

Lancez d'abord la machine virtuelle en cliquant sur le bouton vert Start Machine à droite; une fois chargé, cliquez sur l'URL suivante pour ouvrir la page de connexion GoPhish [https://LAB_WEB_URL.p.thmlabs.com:8443](https://lab_web_url.p.thmlabs.com:8443/)   ou si vous êtes connecté au VPN TryHackMe, vous pouvez aller [https://MACHINE_IP](https://machine_ip/)   (si vous recevez une erreur Nginx, attendez encore 30 secondes et réessayez).

![06640644e0ff4b3f12fd96bce6c7a9ca](https://github.com/dsgsec/Red-Team/assets/82456829/ef130eda-cf2c-4d3a-aacc-01f50d2bcd1b)

Vous devriez pouvoir vous connecter avec le nom d'utilisateur : admin et le mot de passe : tryhackme

Profils d'envoi :

Les profils d'envoi sont les détails de connexion nécessaires pour envoyer réellement vos e-mails de phishing ; il s'agit simplement d'un serveur SMTP auquel vous avez accès. Cliquez sur le lien Profils d'envoi dans le menu de gauche, puis cliquez sur le bouton "Nouveau profil".

Ensuite, ajoutez les informations suivantes selon la capture d'écran ci-dessous :

Nom : Serveur local

De : noreply@redteam.thm

Hôte : 127.0.0.1:25

![2552180eb55c121e0268614de1fd522d](https://github.com/dsgsec/Red-Team/assets/82456829/ff2ffd3a-3b55-472a-8ee0-b36e68c29299)

Cliquez ensuite sur Enregistrer le profil .

Pages de destination :

Ensuite, nous allons configurer la page de destination ; il s'agit du site Web vers lequel l' e-mail d'hameçonnage va diriger la victime ; cette page est généralement une contrefaçon d'un site Web que la victime connaît.

Cliquez sur le lien Pages de destination dans le menu de gauche, puis cliquez sur le bouton "Nouvelle page".

Donnez à la Landing Page le nom ACME Login , ensuite dans la zone HTML ; vous devrez appuyer sur le bouton Source pour nous permettre de saisir le code HTML comme indiqué ci-dessous :
```
<!DOCTYPE html >\
<html lang ="fr" >\
<head>\
<meta charset ="UTF-8" >\
    <title> ACME IT SUPPORT - Panneau d'administration </title>\
 <style>\
        body { font-family : "Ubuntu " , monospace ; text-align : center }\
        div . formulaire de connexion { margin : auto ; largeur : 300 px ; frontière :; rembourrage : 10 px ; aligner texte : gauche ; taille de police : 13 px ; }\
        div . login-form div input { margin-bottom : 7 px ; }\
        div . login-form input { largeur : 280 px ; }\
        div . login-form div : last-child { text-align : center ; }\
        div . div du formulaire de connexion: entrée du dernier enfant { largeur : 100 px ; }\
    </style>\
</head>\
<body>\
    <h2> ACME IT SUPPORT </h2>\
    <h3> Panneau d'administration </h3>\
<form method ="post" >\
<div class ="login-form" >\
            < div> Nom d'utilisateur : </div>\
<div><input name ="nom d'utilisateur" ></div>\
            <div> Mot de passe :\
></div>\
<div><input type ="submit" value ="Connexion" ></div>\
 </div>\
 </form>\
</body>\
</html>
```
Cliquez à nouveau sur le bouton Source , et vous devriez voir une boîte de connexion avec les champs de nom d'utilisateur et de mot de passe selon l'image ci-dessous, cliquez également sur la boîte Capturer les données soumises , puis sur la boîte Capturer les mots de passe , puis cliquez sur le bouton Enregistrer la page.

![c46147f57dbac7f8ae02d2b44e0ed088](https://github.com/dsgsec/Red-Team/assets/82456829/225eab53-3f61-4fde-adaf-4d5513d89fe1)

Modèles d'e-mail :

Il s'agit de la conception et du contenu de l'e-mail que vous allez réellement envoyer à la victime ; il devra être persuasif et contenir un lien vers votre page de destination pour nous permettre de capturer le nom d'utilisateur et le mot de passe de la victime. Cliquez sur le lien Modèles d'e-mail dans le menu de gauche, puis cliquez sur le bouton Nouveau modèle . Donnez au modèle le nom Email 1 , le sujet New Message Received , cliquez sur l'onglet HTML, puis sur le bouton Source pour activer le mode éditeur HTML. Dans le contenu, écrivez un e-mail convaincant qui convaincrait l'utilisateur de cliquer sur le lien, le texte du lien devra être défini sur [https://admin.acmeitsupport.thm](https://admin.acmeitsupport.thm/) , mais le lien réel devra être défini sur {{.URL }}qui sera remplacé par notre page de destination usurpée lors de l'envoi de l'e-mail, vous pouvez le faire en mettant en surbrillance le texte du lien, puis en cliquant sur le bouton du lien dans la rangée supérieure d'icônes, assurez-vous de définir le menu déroulant du protocole sur <autre> .

![5a41d611e4d01c56aa74ca5e7867138b](https://github.com/dsgsec/Red-Team/assets/82456829/2e7885e8-1413-40db-87ef-afc2357b40fc)

![84f89f412b27060f51575e861c8dd1e6](https://github.com/dsgsec/Red-Team/assets/82456829/1e5d3236-632e-4a5b-937a-c9098297c9a9)

Votre e-mail doit ressembler à la capture d'écran ci-dessous. Cliquez sur Enregistrer le modèle une fois terminé.

![66c60935fa89bf7517c4bdc194cce3d4](https://github.com/dsgsec/Red-Team/assets/82456829/776f9af6-dc6c-41f8-bae3-36038c04c9d0)

Utilisateurs et groupes

C'est là que nous pouvons stocker les adresses e-mail de nos cibles. Cliquez sur le lien Utilisateurs et groupes dans le menu de gauche, puis cliquez sur le bouton Nouveau groupe . Donnez au groupe le nom Targets , puis ajoutez les adresses e-mail suivantes :

martin@acmeitsupport.thm\
brian@acmeitsupport.thm\
accounts@acmeitsupport.thm

Cliquez sur le bouton Enregistrer le modèle ; une fois terminé, cela devrait ressembler à la capture d'écran ci-dessous :

![ea9aa63c0ba57459b05628da47f1a9a3](https://github.com/dsgsec/Red-Team/assets/82456829/b5521aea-6596-42d2-8c8b-53f22a77261f)

Campagnes

Il est maintenant temps d'envoyer vos premiers e-mails ; cliquez sur le lien Campagnes dans le menu de gauche, puis cliquez sur le bouton Nouvelle campagne . Définissez les valeurs suivantes pour les entrées, conformément à la capture d'écran ci-dessous :

Nom : Campagne 1

Modèle d'e-mail : E-mail 1

Page de destination : connexion ACME

URL : [http://IP_MACHINE](http://machine_ip/)[](http://machine_ip/)

Date de lancement : pour ce laboratoire, réglez-la sur 2 jours pour vous assurer qu'il n'y a pas de complication avec différents fuseaux horaires. Dans une opération réelle, cela serait correctement défini.

Profil d'envoi : serveur local

Groupes : Cibles

Une fois terminé, cliquez sur le bouton Lancer la campagne , ce qui produira une invite Êtes-vous sûr où vous pouvez simplement appuyer sur le bouton Lancer .

![e70ffbbe29b1d8cd4085e66350300e59](https://github.com/dsgsec/Red-Team/assets/82456829/cb53a1ef-118c-4933-8388-cc65c116e60d)

Vous serez alors redirigé vers la page de résultats de la campagne.

Résultats

La page de résultats nous donne une idée de la performance de la campagne de phishing en nous indiquant combien d'e-mails ont été livrés, ouverts, cliqués et combien d'utilisateurs ont soumis des données à notre faux site Web.

Vous verrez au bas de l'écran une répartition pour chaque adresse e-mail ; vous remarquerez que les e-mails de Martin et de Brian ont été envoyés avec succès, mais que l'e-mail du compte a généré une erreur.

![e70ffbbe29b1d8cd4085e66350300e59](https://github.com/dsgsec/Red-Team/assets/82456829/ddff3713-e49b-471e-afdd-bb36c940702b)

Nous pouvons creuser davantage l'erreur en cliquant sur la flèche déroulante à côté de la ligne du compte, et en affichant les détails ou l'erreur, nous pouvons voir un message d'erreur indiquant que l'utilisateur est inconnu.

![ca6b76d74516da1db95ccebebf7d9423](https://github.com/dsgsec/Red-Team/assets/82456829/b6ec4fe7-5e2b-4529-904f-4333c709c6cd)

Au bout d'une minute et à condition que vous ayez suivi les instructions correctement, vous devriez voir le statut de brian passer à Données soumises.

![282f9bc37ff6fac45a87348e75ea1205](https://github.com/dsgsec/Red-Team/assets/82456829/54df30a9-9701-44aa-8343-40602c9692b5)

En développant les détails de Brian, puis en affichant les détails des données soumises, vous devriez pouvoir voir le nom d'utilisateur et le mot de passe de Brian, ce qui vous aidera à répondre à la question ci-dessous.

![68c81327e142c55eece9d0c2ed02794d](https://github.com/dsgsec/Red-Team/assets/82456829/82b207a2-9266-42bd-ba6d-82fdf34c3c5f)
