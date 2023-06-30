Profilage de mot de passe #1 - Listes de mots par défaut, faibles, divulgués, combinés et de nom d'utilisateur
===========================================

Avoir une bonne liste de mots est essentiel pour mener à bien une attaque par mot de passe. Il est important de savoir comment générer des listes de noms d'utilisateur et des listes de mots de passe. Dans cette section, nous discuterons de la création de listes ciblées de noms d'utilisateur et de mots de passe. Nous couvrirons également divers sujets, y compris les mots de passe par défaut, faibles, divulgués et la création de listes de mots ciblées.

Mots de passe par défaut

Avant d'effectuer des attaques par mot de passe, il vaut la peine d'essayer quelques mots de passe par défaut contre le service ciblé. Les fabricants définissent des mots de passe par défaut avec des produits et des équipements tels que des commutateurs, des pare-feu, des routeurs. Il existe des scénarios où les clients ne changent pas le mot de passe par défaut, ce qui rend le système vulnérable. Ainsi, il est recommandé d'essayer  admin:admin , admin:123456 , etc. Si nous connaissons l'appareil cible, nous pouvons rechercher les mots de passe par défaut et les essayer.  Par exemple, supposons que le serveur cible est un Tomcat, un serveur d'applications Java léger et open source. Dans ce cas, nous pouvons essayer plusieurs mots de passe par défaut :  admin:admin ou tomcat:admin .

Voici quelques listes de sites Web qui fournissent des mots de passe par défaut pour divers produits.

-   [](https://cirt.net/passwords)<https://cirt.net/passwords>
-   [](https://default-password.info/)<https://default-password.info/>
-   [](https://datarecovery.com/rd/default-passwords/)<https://datarecovery.com/rd/default-passwords/>

Mots de passe faibles\
Les professionnels collectent et génèrent des listes de mots de passe faibles au fil du temps et les combinent souvent en une seule grande liste de mots. Les listes sont générées en fonction de leur expérience et de ce qu'ils voient dans les engagements de pentesting.  Ces listes peuvent également contenir des fuites de mots de passe qui ont été publiées publiquement. Voici quelques-unes des listes courantes de mots de passe faibles :

-   <https://wiki.skullsecurity.org/index.php?title=Passwords>[](https://wiki.skullsecurity.org/index.php?title=Passwords) - Cela inclut les collections de mots de passe les plus connues.
-   [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Passwords)  - Une énorme collection de toutes sortes de listes, pas seulement pour le craquage de mots de passe.

### Mots de passe divulgués

Les données sensibles telles que les mots de passe ou les hachages peuvent être divulguées publiquement ou vendues à la suite d'une violation. Ces fuites accessibles au public ou au privé sont souvent appelées « dépotoirs ». Selon le contenu du vidage, un attaquant peut avoir besoin d'extraire les mots de passe des données. Dans certains cas, le vidage peut ne contenir que des hachages des mots de passe et nécessiter un crackage afin d'obtenir les mots de passe en texte brut. Voici quelques-unes des listes de mots de passe courantes qui contiennent des mots de passe faibles et divulgués, y compris les fuites des entreprises webhost , elitehacker , hak5 ,  Hotmail ,  PhpBB :

-   [SecLists/Mots de passe/Bases de données ayant fui](https://github.com/danielmiessler/SecLists/tree/master/Passwords/Leaked-Databases)

### Listes de mots combinées

Disons que nous avons plus d'une liste de mots. Ensuite, nous pouvons combiner ces listes de mots en un seul gros fichier. Cela peut être fait comme suit en utilisant  cat :

plafond

```
cat file1.txt file2.txt file3.txt > combined_list.txt
```

Pour nettoyer la liste combinée générée afin de supprimer les mots en double, nous pouvons utiliser  sort  et  uniq  comme suit :

plafond

```
sort combined_list.txt | uniq -u > cleaned_combined_list.txt
```

### Listes de mots personnalisées

La personnalisation des listes de mots de passe est l'un des meilleurs moyens d'augmenter les chances de trouver des informations d'identification valides. Nous pouvons créer des listes de mots de passe personnalisées à partir du site Web cible. Souvent, le site Web d'une entreprise contient des informations précieuses sur l'entreprise et ses employés, y compris les e-mails et les noms des employés. De plus, le site Web peut contenir des mots-clés spécifiques à ce que l'entreprise propose, y compris des noms de produits et de services, qui peuvent être utilisés dans le mot de passe d'un employé !

Des outils tels que  Cewl peuvent être utilisés pour explorer efficacement un site Web et extraire des chaînes ou des mots-clés. Cewl est un outil puissant pour générer une liste de mots spécifique à une entreprise ou une cible donnée. Considérez l'exemple suivant ci-dessous : 

plafond

```
user@thm$ cewl -w list.txt -d 5 -m 5 http://thm.labs
```

-w  écrira le contenu dans un fichier. Dans ce cas, list.txt.

-m 5  regroupe les chaînes (mots) de 5 caractères ou plus

-d 5  est le niveau de profondeur du web crawling/spidering (défaut 2)

http://thm.labs  est l'URL qui sera utilisée

En conséquence, nous devrions maintenant avoir une liste de mots de taille décente basée sur des mots pertinents pour l'entreprise spécifique, comme les noms, les emplacements et une grande partie de leur jargon professionnel. De même, la liste de mots qui a été créée pourrait être utilisée pour rechercher des noms d'utilisateur . 

Appliquez ce dont nous discutons en utilisant  cewl  contre  https://clinic.thmredteam.com/ pour analyser tous les mots et générer une liste de mots d'une longueur minimale de 8. Notez que nous utiliserons cette liste de mots plus tard avec une autre tâche ! 

### Listes de mots de nom d'utilisateur

La collecte des noms des employés à l'étape du recensement est essentielle. Nous pouvons générer des listes de noms d'utilisateur à partir du site Web de la cible.  Pour l'exemple suivant, nous supposerons que nous avons un  {prénom} {nom de famille} (ex : John Smith) et une méthode de génération de noms d'utilisateur. 

-   {prénom} : john
-   {nom de famille} : forgeron
-   {prénom}{nom} :   johnsmith 
-   {nom}{prénom} :   smithjohn  
-   première lettre du {prénom}{nom} :  jsmith 
-   première lettre du {nom}{prénom} :  sjohn  
-   première lettre du {prénom}.{nom} :  j.smith 
-   première lettre du {prénom}-{nom} :  j-smith 
-   et ainsi de suite

Heureusement, il existe un outil  username_generator qui pourrait aider à créer une liste avec la plupart des combinaisons possibles  si nous avons un prénom et un nom de famille. 

Noms d'utilisateur

```
user@thm$ git clone https://github.com/therodri2/username_generator.git
Cloning into 'username_generator'...
remote: Enumerating objects: 9, done.
remote: Counting objects: 100% (9/9), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 9 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (9/9), done.

user@thm$ cd username_generator

```

L'utilisation  de python3 username_generator.py -h  affiche le message d'aide de l'outil et les arguments facultatifs.

Noms d'utilisateur

```
user@thm$ python3 username_generator.py -h
usage: username_generator.py [-h] -w wordlist [-u]

Python script to generate user lists for bruteforcing!

optional arguments:
  -h, --help            show this help message and exit
  -w wordlist, --wordlist wordlist
                        Specify path to the wordlist
  -u, --uppercase       Also produce uppercase permutations. Disabled by default
```

Créons maintenant une liste de mots contenant le nom complet John Smith dans un fichier texte. Ensuite, nous exécuterons l'outil pour générer les combinaisons possibles du nom complet donné.

Noms d'utilisateur

```
user@thm$ echo "John Smith" > users.lst
user@thm$ python3 username_generator.py -w users.lst
usage: username_generator.py [-h] -w wordlist [-u]
john
smith
j.smith
j-smith
j_smith
j+smith
jsmith
smithjohn
```

Ceci n'est qu'un exemple de générateur de nom d'utilisateur personnalisé. N'hésitez pas à explorer plus d'options ou même à créer les vôtres dans le langage de programmation de votre choix !
