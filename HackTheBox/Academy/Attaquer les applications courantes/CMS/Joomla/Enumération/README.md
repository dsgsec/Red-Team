Joomla - Découverte et énumération
================================

* * * * *

[Joomla](https://www.joomla.org/), publié en août 2005, est un autre CMS gratuit et open source utilisé pour les forums de discussion, les galeries de photos, le commerce électronique, les communautés d'utilisateurs, etc. Il est écrit en PHP et utilise MySQL dans le backend. Comme WordPress, Joomla peut être amélioré avec plus de 7 000 extensions et plus de 1 000 modèles. Il existe jusqu'à 2,5 millions de sites sur Internet exécutant Joomla. Voici quelques [statistiques](https://websitebuilder.org/blog/joomla-statistics/) intéressantes à propos de Joomla :

- Joomla représente 3,5% de la part de marché des CMS
- Joomla est 100% gratuit et signifie "tous ensemble" en swahili (orthographe phonétique de "Jumla")
- La communauté Joomla compte près de 700 000 dans ses forums en ligne
- Joomla alimente 3 % de tous les sites Web sur Internet, près de 25 000 des 1 millions de sites les plus importants dans le monde (seulement 10 % de la portée de WordPress)
- Certaines organisations notables qui utilisent Joomla incluent eBay, Yamaha, l'Université de Harvard et le gouvernement britannique
- Au fil des ans, 770 développeurs différents ont contribué à Joomla

Joomla collecte des [statistiques d'utilisation](https://developer.joomla.org/about/stats.html) anonymes, telles que la répartition des versions de Joomla, PHP et des bases de données et des systèmes d'exploitation de serveur utilisés sur les installations Joomla. Ces données peuvent être interrogées via leur [API] publique (https://developer.joomla.org/about/stats/api.html).

En interrogeant cette API, nous pouvons voir plus de 2,7 millions d'installations Joomla !

```
dsgsec@htb[/htb]$ curl -s https://developer.joomla.org/stats/cms_version | python3 -m json.tool

{
     "données": {
         "version_cms": {
             "3.0": 0,
             "3.1": 0,
             "3.10": 3.49,
             "3.2": 0.01,
             "3.3": 0.02,
             "3.4": 0.05,
             "3.5": 13,
             "3.6": 24.29,
             "3.7": 8.5,
             "3.8": 18.84,
             "3.9": 30.28,
             "4.0": 1.52,
             "4.1": 0
         },
         "total": 2776276
     }
}

```

* * * * *

Découverte/Empreinte
----------------------

Supposons que nous tombions sur un site e-commerce lors d'un test d'intrusion externe. À première vue, nous ne savons pas exactement ce qui fonctionne, mais cela ne semble pas être entièrement personnalisé. Si nous pouvons identifier ce sur quoi le site s'exécute, nous pourrons peut-être découvrir des vulnérabilités ou des erreurs de configuration. Sur la base des informations limitées, nous supposons que le site exécute Joomla, mais nous devons confirmer ce fait, puis déterminer le numéro de version et d'autres informations telles que les thèmes et plugins installés.

Nous pouvons souvent identifier Joomla en regardant la source de la page, qui nous indique que nous avons affaire à un site Joomla.

```
dsgsec@htb[/htb]$ curl -s http://dev.inlanefreight.local/ | grep Joomla

<meta name="generator" content="Joomla! - Gestion de contenu open source" />

<SNIP>

```

Le fichier `robots.txt` d'un site Joomla ressemble souvent à ceci :

```
# Si le site Joomla est installé dans un dossier
# par exemple www.example.com/joomla/ puis le fichier robots.txt
# DOIT être déplacé à la racine du site
# par exemple www.example.com/robots.txt
# ET le nom du dossier joomla DOIT être préfixé à tous les
# chemins.
# par exemple, la règle Disallow pour le dossier /administrator/ DOIT
# être changé en lecture
# Interdire : /joomla/administrateur/
#
# Pour plus d'informations sur la norme robots.txt, consultez :
# https://www.robotstxt.org/orig.html

Agent utilisateur: *
Interdire : /administrateur/
Interdire : /bin/
Interdire : /cache/
Interdire : /cli/
Interdire : /composants/
Interdire : /inclut/
Interdire : /installation/
Interdire : /langue/
Interdire : /mises en page/
Interdire : /bibliothèques/
Interdire : /logs/
Interdire : /modules/
Interdire : /plugins/
Interdire : /tmp/

```

Nous pouvons aussi souvent voir le favicon révélateur de Joomla (mais pas toujours). Nous pouvons identifier la version de Joomla si le fichier `README.txt` est présent.

```
dsgsec@htb[/htb]$ curl -s http://dev.inlanefreight.local/README.txt | head -n 5

1- Qu'est-ce que c'est ?
* Ceci est un Joomla! package d'installation/de mise à niveau vers la version 3.x
* Joomla! Site officiel : https://www.joomla.org
* Joomla! Historique des versions 3.9 - https://docs.joomla.org/Special:MyLanguage/Joomla_3.9_version_history
* Modifications détaillées dans le journal des modifications : https://github.com/joomla/joomla-cms/commits/staging

```

Dans certaines installations de Joomla, nous pouvons être en mesure d'identifier la version à partir de fichiers JavaScript dans le répertoire `media/system/js/` ou en accédant à `administrator/manifests/files/joomla.xml`.

```
dsgsec@htb[/htb]$ curl -s http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -

<?xml version="1.0" encoding="UTF-8" ?>
<extension version="3.6" type="fichier" method="mise à niveau">
   <name>files_joomla</name>
   <auteur>Joomla! Projet</auteur>
   <authorEmail>admin@joomla.org</authorEmail>
   <authorUrl>www.joomla.org</authorUrl>
   <copyright>(C) 2005 - 2019 Open Source Matters. Tous droits réservés</copyright>
   <license>GNU General Public License version 2 ou ultérieure ; voir LICENSE.txt</license>
   <version>3.9.4</version>
   <creationDate>Mars 2019</creationDate>

  <SNIP>

```

Le fichier `cache.xml` peut nous aider à nous donner la version approximative. Il se trouve dans `plugins/system/cache/cache.xml`.

* * * * *

Énumération
-----------

Essayons [droopescan](https://github.com/droope/droopescan), un scanner basé sur un plug-in qui fonctionne pour SilverStripe, WordPress et Drupal avec des fonctionnalités limitées pour Joomla et Moodle.

Nous pouvons cloner le référentiel Git et l'installer manuellement ou l'installer via `pip`.

```
dsgsec@htb[/htb]$ sudo pip3 install droopescan

Collecte de droopescan
   Téléchargement de dropescan-1.45.1-py2.py3-none-any.whl (514 Ko)
      |████████████████████████████████| 514 Ko 5,8 Mo/s

<SNIP>

```

Une fois l'installation terminée, nous pouvons confirmer que l'outil fonctionne en exécutant `droopescan -h`.

```
dsgsec@htb[/htb]$ droopescan -h

usage : droopescan (sous-commandes ...) [options ...] {arguments ...}

     |
  ___| ___ ___ ___ ___ ___ ___ ___ ___ ___
| )| )| )| )| )|___)|___ | | )| )
|__/ | |__/ |__/ |__/ |__ __/ |__ |__/|| /
                     |
================================================

commandes :

   analyse
     fonctionnalité de numérisation cms.

   Statistiques
     affiche l'état et les capacités du scanner.

arguments facultatifs :
   -h, --help affiche ce message d'aide et quitte
   --debug bascule la sortie de débogage
   --quiet supprimer toutes les sorties

Exemples d'invocation :
   analyse droopescan drupal -u URL_HERE
   droopescan scan silverstripe -u URL_HERE

Plus d'informations:
   analyse dropescan --aide

Veuillez consulter le fichier README pour plus d'informations sur les proxys.

```

Nous pouvons accéder à un menu d'aide plus détaillé en tapant `droopescan scan --help`.

Faisons un scan et voyons ce que ça donne.

```
dsgsec@htb[/htb]$ droopescan analyse joomla --url http://dev.inlanefreight.local/

[+] Version(s) possible(s) :
     3.8.10
     3.8.11
     3.8.11-rc
     3.8.12
     3.8.12-rc
     3.8.13
     3.8.7
     3.8.7-rc
     3.8.8
     3.8.8-rc
     3.8.9
     3.8.9-rc

[+] Possibles URL intéressantes trouvées :
     Informations détaillées sur la version. - http://dev.inlanefreight.local/administrator/manifests/files/joomla.xml
     Page de connexion. - http://dev.inlanefreight.local/administrator/
     Fichier de licence. - http://dev.inlanefreight.local/LICENSE.txt
     L'attribut de version contient la version approximative - http://dev.inlanefreight.local/plugins/system/cache/cache.xml

[+] Numérisation terminée (0:00:01.523369 écoulé)

```

Comme nous pouvons le voir, il n'a pas fourni beaucoup d'informations à part le numéro de version possible. Nous pouvons également essayer [JoomlaScan](https://github.com/drego85/JoomlaScan), qui est un outil Python inspiré de l'ancien OWASP [joomscan](https://github.com/OWASP/joomscan) outil. `JoomlaScan` est un peu obsolète et nécessite Python 2.7 pour s'exécuter. Nous pouvons le faire fonctionner en nous assurant d'abord que certaines dépendances sont installées.

```
dsgsec@htb[/htb]$ sudo python2.7 -m pip install urllib3
dsgsec@htb[/htb]$ sudo python2.7 -m pip install certifi
dsgsec@htb[/htb]$ sudo python2.7 -m pip install bs4

```

Bien qu'un peu dépassé, il peut être utile dans notre énumération. Lançons un scan.

```
dsgsec@htb[/htb]$ python2.7 joomlascan.py -u http://dev.inlanefreight.local

------------------------------------------------
       Analyse Joomla
    Utilisation : python joomlascan.py <cible>
     Version 0.5beta - Entrées de la base de données 1233
          créé par Andrea Draghetti
------------------------------------------------
Fichier robots trouvé : > http://dev.inlanefreight.local/robots.txt
Aucun journal d'erreur trouvé

Commencez l'analyse... avec 10 threads simultanés !
Composant trouvé : com_actionlogs > http://dev.inlanefreight.local/index.php?option=com_actionlogs
Sur les composants administrateur
Composant trouvé : com_admin > http://dev.inlanefreight.local/index.php?option=com_admin
Sur les composants administrateur
Composant trouvé : com_ajax > http://dev.inlanefreight.local/index.php?option=com_ajax
Mais peut-être n'est-il pas actif ou protégé
Fichier de LICENCE trouvé > http://dev.inlanefreight.local/administrator/components/com_actionlogs/actionlogs.xml
Fichier LICENSE trouvé > http://dev.inlanefreight.local/administrator/components/com_admin/admin.xml
Fichier de LICENCE trouvé > http://dev.inlanefreight.local/administrator/components/com_ajax/ajax.xml
Répertoire explorable > http://dev.inlanefreight.local/components/com_actionlogs/
Répertoire explorable > http://dev.inlanefreight.local/administrator/components/com_actionlogs/
Répertoire explorable > http://dev.inlanefreight.local/components/com_admin/
Répertoire explorable > http://dev.inlanefreight.local/administrator/components/com_admin/
Composant trouvé : com_banners > http://dev.inlanefreight.local/index.php?option=com_banners
Mais peut-être n'est-il pas actif ou protégé
Répertoire explorable > http://dev.inlanefreight.local/components/com_ajax/
Répertoire explorable > http://dev.inlanefreight.local/administrator/components/com_ajax/
Fichier de LICENCE trouvé > http://dev.inlanefreight.local/administrator/components/com_banners/banners.xml

<SNIP>

```

Bien qu'il ne soit pas aussi précieux que droopescan, cet outil peut nous aider à trouver des répertoires et des fichiers accessibles et peut aider à identifier les extensions installées. À ce stade, nous savons que nous avons affaire à Joomla `3.9.4`. Le portail de connexion de l'administrateur se trouve à `http://dev.inlanefreight.local/administrator/index.php`. Les tentatives d'énumération des utilisateurs renvoient un message d'erreur générique.

```
Avertissement
Le nom d'utilisateur et le mot de passe ne correspondent pas ou vous n'avez pas encore de compte.

```

Le compte administrateur par défaut sur les installations de Joomla est `admin`, mais le mot de passe est défini au moment de l'installation, donc la seule façon dont nous pouvons espérer entrer dans le back-end d'administration est si le compte est défini avec un mot de passe très faible/commun et nous pouvons entrer avec quelques conjectures ou une légère force brute. Nous pouvons utiliser ce [script](https://github.com/ajnik/joomla-bruteforce) pour tenter de forcer brutalement la connexion.

```
dsgsec@htb[/htb]$ sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin

administrateur : administrateur

```

Et nous obtenons un résultat avec les identifiants `admin:admin`. Quelqu'un n'a pas suivi les meilleures pratiques !