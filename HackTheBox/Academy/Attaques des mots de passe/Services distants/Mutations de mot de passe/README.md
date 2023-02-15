# Mutations de mot de passe

Beaucoup de gens créent leurs mots de passe en fonction de la simplicité plutôt que de la sécurité. Pour éliminer cette faiblesse humaine qui compromet souvent les mesures de sécurité, des politiques de mot de passe peuvent être créées sur tous les systèmes qui déterminent à quoi un mot de passe doit ressembler. Cela signifie que le système reconnaît si le mot de passe contient des majuscules, des caractères spéciaux et des chiffres. De plus, la plupart des stratégies de mot de passe exigent une longueur minimale de huit caractères dans un mot de passe, y compris au moins une des spécifications ci-dessus.

Dans les sections précédentes, nous avons deviné des mots de passe très simples, mais il devient beaucoup plus difficile de l'adapter aux systèmes qui appliquent des politiques de mot de passe qui forcent la création de mots de passe plus complexes.

Malheureusement, la tendance des utilisateurs à créer des mots de passe faibles se produit également malgré l'existence de politiques de mot de passe. La plupart des personnes/employés suivent les mêmes règles lors de la création de mots de passe plus complexes. Les mots de passe sont souvent créés étroitement liés au service utilisé. Cela signifie que de nombreux employés sélectionnent souvent des mots de passe qui peuvent contenir le nom de l'entreprise dans les mots de passe. Les préférences et les intérêts d'une personne jouent également un rôle important. Ceux-ci peuvent être des animaux de compagnie, des amis, des sports, des passe-temps et de nombreux autres éléments de la vie. La collecte d'informations OSINT peut être très utile pour en savoir plus sur les préférences d'un utilisateur et peut aider à deviner le mot de passe. Plus d'informations sur OSINT peuvent être trouvées dans le module OSINT: Corporate Recon. Généralement, les utilisateurs utilisent les ajouts suivants pour que leur mot de passe corresponde aux stratégies de mot de passe les plus courantes :

| Descriptif | Syntaxe du mot de passe |
| --- | --- |
| La première lettre est en majuscule. | `Mot de passe` |
| Ajout de nombres. | `Mot de passe123` |
| Ajout d'année. | `Mot de passe2022` |
| Ajout du mois. | `Mot de passe02` |
| Le dernier caractère est un point d'exclamation. | `Mot de passe2022 !` |
| Ajout de caractères spéciaux. | `P@ssw0rd2022!` |

Étant donné que de nombreuses personnes souhaitent garder leurs mots de passe aussi simples que possible malgré les politiques de mot de passe, nous pouvons créer des règles pour générer des mots de passe faibles. D'après les statistiques fournies par WPengine, la plupart des mots de passe ne dépassent pas dix caractères. Donc, ce que nous pouvons faire, c'est choisir des termes spécifiques d'au moins cinq caractères et qui semblent être les plus familiers aux utilisateurs, tels que les noms de leurs animaux de compagnie, leurs passe-temps, leurs préférences et d'autres intérêts. Si l'utilisateur choisit un seul mot (comme le mois en cours), ajoute l'année en cours, suivie d'un caractère spécial, à la fin de son mot de passe, nous atteindrons l'exigence d'un mot de passe à dix caractères. Étant donné que la plupart des entreprises exigent des changements de mot de passe réguliers, un utilisateur peut modifier son mot de passe en changeant simplement le nom d'un mois ou d'un seul numéro, etc. Prenons un exemple simple pour créer une liste de mots de passe avec une seule entrée.

## Liste de mot de passe
```
dsgsec@htb[/htb]$ cat password.list

password
```

Nous pouvons utiliser un outil très puissant appelé Hashcat pour combiner des listes de noms et d'étiquettes potentiels avec des règles de mutation spécifiques pour créer des listes de mots personnalisées. Pour vous familiariser avec Hashcat et découvrir tout le potentiel de cet outil, nous vous recommandons le module Cracking Passwords with Hashcat. Hashcat utilise une syntaxe spécifique pour définir les caractères et les mots et comment ils peuvent être modifiés. La liste complète de cette syntaxe se trouve dans la documentation officielle de Hashcat. Cependant, ceux énumérés ci-dessous nous suffisent pour comprendre comment Hashcat fait muter les mots.

| Fonction | Descriptif |
| --- | --- |
| `:` | Ne fais rien. |
| `l` | Minuscule toutes les lettres. |
| `u` | Majuscule toutes les lettres. |
| `c` | Mettez la première lettre en majuscule et les autres en minuscules. |
| `sXY` | Remplacez toutes les instances de X par Y. |
| `$!` | Ajoutez le caractère d'exclamation à la fin. |


Chaque règle est écrite sur une nouvelle ligne qui détermine comment le mot doit être muté. Si nous écrivons les fonctions présentées ci-dessus dans un fichier et considérons les aspects mentionnés, ce fichier peut alors ressembler à ceci :

## Liste des règles fichier Hashcat
```
dsgsec@htb[/htb]$ cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

Hashcat appliquera les règles de custom.rule pour chaque mot dans password.list et stockera la version mutée dans notre mut_password.list en conséquence. Ainsi, un mot entraînera quinze mots mutés dans ce cas.

## Génération de mots de passes basée sur le fichier de règle
```
dsgsec@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
dsgsec@htb[/htb]$ cat mut_password.list

password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
Password!
passw0rd!
p@ssword!
Passw0rd!
P@ssword!
p@ssw0rd!
P@ssw0rd!
```

Hashcat et John sont livrés avec des listes de règles prédéfinies que nous pouvons utiliser à des fins de génération et de craquage de mots de passe. L'une des règles les plus utilisées est best64.rule, qui peut souvent donner de bons résultats. Il est important de noter que le craquage de mots de passe et la création de listes de mots personnalisées est un jeu de devinettes dans la plupart des cas. Nous pouvons réduire cela et effectuer des devinettes plus ciblées si nous disposons d'informations sur la politique de mot de passe et prenons en compte le nom de l'entreprise, la région géographique, l'industrie et d'autres sujets/mots que les utilisateurs peuvent sélectionner pour créer leurs mots de passe. Les exceptions sont, bien sûr, les cas où les mots de passe sont divulgués et trouvés.

## Règles Hashcat déjà existante
```
dsgsec@htb[/htb]$ ls /usr/share/hashcat/rules/

best64.rule                  specific.rule
combinator.rule              T0XlC-insert_00-99_1950-2050_toprules_0_F.rule
d3ad0ne.rule                 T0XlC-insert_space_and_special_0_F.rule
dive.rule                    T0XlC-insert_top_100_passwords_1_G.rule
generated2.rule              T0XlC.rule
generated.rule               T0XlCv1.rule
hybrid                       toggles1.rule
Incisive-leetspeak.rule      toggles2.rule
InsidePro-HashManager.rule   toggles3.rule
InsidePro-PasswordsPro.rule  toggles4.rule
leetspeak.rule               toggles5.rule
oscommerce.rule              unix-ninja-leetspeak.rule
rockyou-30000.rule
```

Nous pouvons maintenant utiliser un autre outil appelé CeWL pour analyser les mots potentiels du site Web de l'entreprise et les enregistrer dans une liste séparée. Nous pouvons ensuite combiner cette liste avec les règles souhaitées et créer une liste de mots de passe personnalisée qui a une plus grande probabilité de deviner un mot de passe correct. Nous spécifions certains paramètres, comme la profondeur d'araignée (-d), la longueur minimale du mot (-m), le stockage des mots trouvés en minuscules (--minuscules), ainsi que le fichier où nous voulons stocker les résultats (-w).

## Génération de wordlist avec cewl
```
dsgsec@htb[/htb]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
dsgsec@htb[/htb]$ wc -l inlane.wordlist

326
```
