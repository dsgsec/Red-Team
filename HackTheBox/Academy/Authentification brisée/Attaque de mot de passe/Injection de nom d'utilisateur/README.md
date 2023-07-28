Injection de nom d'utilisateur
==============================

* * * * *

Lorsque vous essayez de comprendre la logique de haut niveau derrière un formulaire de réinitialisation, peu importe s'il envoie un jeton, un mot de passe temporaire ou nécessite la bonne réponse. À un niveau élevé, lorsqu'un utilisateur saisit la valeur attendue, la fonctionnalité de réinitialisation permet à l'utilisateur de modifier le mot de passe ou de passer la phase d'authentification. La fonction qui vérifie si un jeton de réinitialisation est valide et est également le bon pour un compte donné est généralement soigneusement développée et testée dans un souci de sécurité. Cependant, il est parfois vulnérable lors de la deuxième phase du processus, lorsque l'utilisateur réinitialise le mot de passe après que la première connexion a été accordée.

Imaginez le scénario suivant. Après avoir créé notre propre compte, nous demandons une réinitialisation du mot de passe. Supposons que nous rencontrions un formulaire qui se comporte comme suit.

![](https://academy.hackthebox.com/storage/modules/80/10-reset.png)

Nous pouvons essayer d'injecter un nom d'utilisateur et/ou une adresse e-mail différents, en recherchant une éventuelle valeur d'entrée cachée ou en devinant tout nom d'entrée valide. Il a été observé que certaines applications donnent la priorité aux informations reçues par rapport aux informations stockées dans une valeur de session.

Un exemple de code vulnérable ressemble à ceci (la `$_REQUEST`variable contient à la fois `$_GET`et `$_POST`):

Code : php

```
<?php
  if isset($_REQUEST['userid']) {
	$userid = $_REQUEST['userid'];
  } else if isset($_SESSION['userid']) {
	$userid = $_SESSION['userid'];
  } else {
	die("unknown userid");
  }

```

Cela peut sembler bizarre au premier abord, mais pensez à une application Web qui permet aux administrateurs ou aux employés du service d'assistance de réinitialiser les mots de passe des autres utilisateurs. Souvent, la fonction qui change le mot de passe est réutilisée et partage la même base de code avec celle utilisée par les utilisateurs standards pour changer leur mot de passe. Une application doit toujours vérifier l'autorisation avant tout changement. Dans ce cas, il doit vérifier si l'utilisateur a le droit de modifier le mot de passe de l'utilisateur cible. Dans cet esprit, nous devons énumérer l'application Web pour identifier comment elle attend le champ nom d'utilisateur ou e-mail lors de la phase de connexion, lorsqu'il y a des messages ou un échange de communication, ou lorsque nous voyons les profils d'autres utilisateurs. Après avoir collecté une liste de tous les noms de champs de saisie possibles, nous allons attaquer l'application.

Nous avons forcé brutalement le nom d'utilisateur et le mot de passe sur une application Web qui utilise `userid`comme nom de champ lors du processus de connexion dans les exercices précédents. Conservons ce champ comme identifiant de l'utilisateur et opérons dessus. Une requête standard se présente comme suit.

![](https://academy.hackthebox.com/storage/modules/80/username_injection_req1.png)

Si vous modifiez la demande en ajoutant le `userid`champ, vous pouvez modifier le mot de passe d'un autre utilisateur.

![](https://academy.hackthebox.com/storage/modules/80/username_injection_req2.png)

Comme nous pouvons le voir, l'application répond par un `success`message.

Lorsque nous avons un petit nombre de champs et de valeurs d'utilisateur/e-mail à tester, vous pouvez monter cette attaque à l'aide d'un proxy d'interception. Si vous en avez beaucoup, vous pouvez automatiser l'attaque à l'aide de n'importe quel fuzzer ou d'un script personnalisé. Nous avons préparé une petite aire de jeux pour vous permettre de tester cette attaque. Vous pouvez télécharger le script PHP [ici](https://academy.hackthebox.com/storage/modules/80/scripts/username_injection_php.txt) et le script Python [ici](https://academy.hackthebox.com/storage/modules/80/scripts/username_injection_py.txt) . Prenez votre temps pour étudier les deux fichiers, puis essayez de reproduire l'attaque que nous avons montrée.
