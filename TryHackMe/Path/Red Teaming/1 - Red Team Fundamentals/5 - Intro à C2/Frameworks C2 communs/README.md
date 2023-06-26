### Framework C2 communs

Tout au long de votre parcours, vous pourrez rencontrer de nombreux Frameworks C2 différents ; nous discuterons de quelques Frameworks C2 populaires qui sont largement utilisés par les équipes rouges et les adversaires. Nous allons le diviser en deux sections :

-   Gratuit
-   Prime payée

Vous pouvez poser des questions comme "Pourquoi utiliser un framework C2 premium ou payant ?", et c'est une excellente question. Les frameworks C2 premium/payants sont généralement moins susceptibles d'être détectés par les fournisseurs d'antivirus. Cela ne veut pas dire qu'il est impossible d'être détecté, juste que les projets C2 open source sont généralement bien compris et que les signatures peuvent être facilement développées.

Habituellement, les frameworks C2 premium ont généralement des modules de post-exploitation plus avancés, des fonctionnalités pivotantes et même des demandes de fonctionnalités que les développeurs de logiciels open source peuvent parfois ne pas satisfaire. Par exemple, une fonctionnalité offerte par Cobalt Strike que la plupart des autres frameworks C2 n'offrent pas est la possibilité d'ouvrir un tunnel VPN à partir d'une balise. Cela peut être une fonctionnalité fantastique si un proxy ne fonctionne pas bien dans votre situation spécifique. Vous devez faire vos recherches pour savoir ce qui fonctionnera le mieux pour votre équipe.

### Frameworks C2 gratuits

Metasploit

Le [framework Metasploit](https://www.metasploit.com/) , développé et maintenu par Rapid7, est l'un des frameworks d'exploitation et de post-exploitation (C2) les plus populaires qui est accessible au public et est installé sur la plupart des distributions de tests d'intrusion.

MSFConsole

```
root@kali$ msfconsole

      .:okOOOkdc'           'cdkOOOko:.
    .xOOOOOOOOOOOOc       cOOOOOOOOOOOOx.
   :OOOOOOOOOOOOOOOk,   ,kOOOOOOOOOOOOOOO:
  'OOOOOOOOOkkkkOOOOO: :OOOOOOOOOOOOOOOOOO'
  oOOOOOOOO.    .oOOOOoOOOOl.    ,OOOOOOOOo
  dOOOOOOOO.      .cOOOOOc.      ,OOOOOOOOx
  lOOOOOOOO.         ;d;         ,OOOOOOOOl
  .OOOOOOOO.   .;           ;    ,OOOOOOOO.
   cOOOOOOO.   .OOc.     'oOO.   ,OOOOOOOc
    oOOOOOO.   .OOOO.   :OOOO.   ,OOOOOOo
     lOOOOO.   .OOOO.   :OOOO.   ,OOOOOl
      ;OOOO'   .OOOO.   :OOOO.   ;OOOO;
       .dOOo   .OOOOocccxOOOO.   xOOd.
         ,kOl  .OOOOOOOOOOOOO. .dOk,
           :kk;.OOOOOOOOOOOOO.cOk:
             ;kOOOOOOOOOOOOOOOk:
               ,xOOOOOOOOOOOx,
                 .lOOOOOOOl.
                    ,dOd,
                      .

       =[ metasploit v6.1.12-dev                          ]
+ -- --=[ 2177 exploits - 1152 auxiliary - 399 post       ]
+ -- --=[ 596 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: View a module's description using
info, or the enhanced version in your browser with
info -d

msf6 >
```

Armitage

[Armitage](https://web.archive.org/web/20211006153158/http://www.fastandeasyhacking.com/)  est une extension du Metasploit Framework - il ajoute une interface utilisateur graphique et est écrit en Java, et est incroyablement similaire à Cobalt Strike. C'est parce qu'ils ont tous deux été développés par Raphael Mudge. Armitage offre un moyen simple d'énumérer et de visualiser l'ensemble de vos cibles. En plus de ressembler beaucoup à Cobalt Strike, il offre même des fonctionnalités uniques. L'un des plus populaires se trouve dans le menu "Attaques" ; Cette fonctionnalité est connue sous le nom d'attaque Hail Mary, qui tente d'exécuter tous les exploits pour les services exécutés sur un poste de travail spécifique. Armitage est vraiment "Piratage rapide et facile".
![6fc808991c2ea5a5faae2a271f9bf907](https://github.com/dsgsec/Red-Team/assets/82456829/e047ac7d-124f-4291-86c8-f4e578d96382)


*Une capture d'écran de l'interface utilisateur d'Armitage*

Powershell Empire/Starkiller

[Powershell Empire](https://bc-security.gitbook.io/empire-wiki/) et [Starkiller](https://github.com/BC-SECURITY/Starkiller) est un autre C2 incroyablement populaire créé à l'origine par Harmjoy, Sixdub et Enigma0x3 de Veris Group. Actuellement, le projet a été interrompu et a été repris par l'équipe BC Security (Cx01N, Hubbl3 et _Vinnybod). Empire propose des agents écrits dans différentes langues compatibles avec plusieurs plates-formes, ce qui en fait un C2 incroyablement polyvalent. Pour plus d'informations sur Empire, nous vous recommandons de jeter un œil à la salle [Powershell Empire](https://tryhackme.com/room/rppsempire) .

![7605d1d88a01604402dcde1878a8acd0](https://github.com/dsgsec/Red-Team/assets/82456829/3c848042-0ccc-4726-9707-33a9e707dd17)

*Une capture d'écran de l'interface utilisateur de Starkiller*

Covenant

[Covenant](https://github.com/cobbr/Covenant) de Ryan Cobb est le dernier framework C2 gratuit que nous couvrirons - De loin, c'est l'un des frameworks C2 les plus uniques écrits en C#. Contrairement à Metasploit/Armitage, il est principalement utilisé pour la post-exploitation et le mouvement latéral avec des écouteurs HTTP, HTTPS et SMB avec  des agents hautement personnalisables .

![55f84861830c1bc50d2b251bda883019](https://github.com/dsgsec/Red-Team/assets/82456829/404f8581-8078-478f-b1e8-ed7b4450f2d6)

*Une capture d'écran de l'interface utilisateur de Covenant*

###

Silver

[Sliver](https://github.com/BishopFox/sliver) de [Bishop Fox](https://bishopfox.com/) est un framework C2 multi-utilisateurs avancé, hautement personnalisable et basé sur CLI. Sliver est écrit en Go, ce qui rend la rétro-ingénierie des "implants" C2 incroyablement difficile. Il prend en charge divers protocoles pour les communications C2 comme WireGuard, mTLS, HTTP(S), DNS et bien plus encore. De plus, il prend en charge les fichiers BOF pour des fonctionnalités supplémentaires, les domaines DNS Canary pour masquer les communications C2, la génération automatique de certificats Let's Encrypt pour les balises HTTPS, et bien plus encore.  

![9447257966d025c8927541a7adab9113](https://github.com/dsgsec/Red-Team/assets/82456829/f58df58d-9f30-401a-a365-3beb25f3a6e0)

﻿*Une capture d'écran de l'interface utilisateur Sliver*

### Frameworks C2 payants

Cobalt Strike

[Cobalt Strike](https://www.cobaltstrike.com/) par Help Systems (anciennement créé par Raphael Mudge) est sans doute l'un des frameworks de commandement et de contrôle les plus célèbres à côté de Metasploit. Tout comme Artimage, il est écrit en Java et conçu pour être aussi flexible que possible. Pour plus d'informations, consultez [la page de formation vidéo](https://www.youtube.com/playlist?list=PLcjpg2ik7YT6H5l9Jx-1ooRYpfvznAInJ) de Cobalt Strike . Il offre un aperçu supplémentaire des opérations de l'équipe rouge et du Framework par Raphael Mudge lui-même.
![573e8581dd1be594cf674cbf06271cde](https://github.com/dsgsec/Red-Team/assets/82456829/6d70d227-e37b-4070-9eb7-7947c4579648)


*Une capture d'écran de l'interface utilisateur de Cobalt Strike*

Brute Ratel

[Brute Ratel](https://bruteratel.com/) de Chetan Nayak ou Paranoid Ninja est un framework de commandement et de contrôle commercialisé en tant que "centre de commandement et de contrôle personnalisable" ou "C4" qui offre une véritable expérience de simulation d'adversaire avec un Framework C2 unique. Pour plus d'informations sur le Framework, l'auteur a fourni une [page de formation vidéo](https://bruteratel.com/tabs/tutorials/) qui illustre de nombreuses fonctionnalités du Framework.

![4afab434ea1d36584a5f7d0d7b8c2ad1](https://github.com/dsgsec/Red-Team/assets/82456829/f0ea54a0-6cac-4bb7-a7cc-df23cdf4c626)

*Capture d'écran de l'interface utilisateur de Brute Ratel - Source :  *<https://bruteratel.com/>[](https://bruteratel.com/)

### Autres Frameworks C2

Pour une liste plus complète des frameworks C2 et de leurs capacités, consultez la « [Matrice C2](https://howto.thec2matrix.com/) », un projet maintenu par Jorge Orchilles et Bryson Bort . Il contient une liste beaucoup plus complète de presque tous les Frameworks C2 actuellement disponibles. Nous vous recommandons fortement d'aller le vérifier après cette salle et d'explorer certains des autres Frameworks C2 qui n'ont pas été abordés dans cette salle.
