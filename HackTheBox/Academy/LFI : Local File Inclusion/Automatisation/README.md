Numérisation automatisée
==================

Il est essentiel de comprendre comment fonctionnent les attaques par inclusion de fichiers et comment nous pouvons créer manuellement des charges utiles avancées et utiliser des techniques personnalisées pour atteindre l'exécution de code à distance. En effet, dans de nombreux cas, pour que nous puissions exploiter la vulnérabilité, elle peut nécessiter une charge utile personnalisée correspondant à ses configurations spécifiques. De plus, lorsqu'il s'agit de mesures de sécurité telles qu'un WAF ou un pare-feu, nous devons appliquer notre compréhension pour voir comment une charge utile / un personnage spécifique est bloqué et tenter de créer une charge utile personnalisée pour la contourner.

Nous n'avons peut-être pas besoin d'exploiter manuellement la vulnérabilité LFI dans de nombreux cas triviaux. Il existe de nombreuses méthodes automatisées qui peuvent nous aider à identifier et exploiter rapidement les vulnérabilités LFI triviales. Nous pouvons utiliser des outils de fuzzing pour tester une énorme liste de charges utiles LFI courantes et voir si l'une d'entre elles fonctionne, ou nous pouvons utiliser des outils LFI spécialisés pour tester ces vulnérabilités. C'est ce dont nous allons discuter dans cette section.

* * * * *

Paramètres de fuzzing
------------------

Les formulaires HTML que les utilisateurs peuvent utiliser sur le front-end de l'application Web ont tendance à être correctement testés et bien protégés contre les différentes attaques Web. Cependant, dans de nombreux cas, la page peut avoir d'autres paramètres exposés qui ne sont liés à aucun formulaire HTML, et par conséquent, les utilisateurs normaux n'y accéderaient jamais ou ne causeraient pas de dommages involontairement. C'est pourquoi il peut être important de fuzzer les paramètres exposés, car ils ont tendance à ne pas être aussi sûrs que les paramètres publics.

Le module [Attaquer les applications Web avec Ffuf](https://academy.hackthebox.com/module/details/54) explique en détail comment nous pouvons fuzzer les paramètres `GET`/`POST` . Par exemple, nous pouvons fuzzer la page pour les paramètres `GET` courants, comme suit :

```
dsgsec@htb[/htb]$ ffuf -w /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index. php?FUZZ=valeur' -fs 2287

...COUPER...

  :: Méthode : GET
  :: URL : http://<SERVER_IP>:<PORT>/index.php?FUZZ=value
  :: Liste de mots : FUZZ : /opt/useful/SecLists/Discovery/Web-Content/burp-parameter-names.txt
  :: Suivre les redirections : false
  :: Calibrage : faux
  :: Délai d'attente : 10
  :: Fils : 40
  :: Matcher : état de la réponse : 200 204 301 302 307 401 403
  :: Filtre : Taille de la réponse : xxx
________________________________________________

langue [Statut : xxx, Taille : xxx, Mots : xxx, Lignes : xxx]

```

Une fois que nous avons identifié un paramètre exposé qui n'est lié à aucun des formulaires que nous avons testés, nous pouvons effectuer tous les tests LFI abordés dans ce module. Ceci n'est pas propre aux vulnérabilités LFI, mais s'applique également à la plupart des vulnérabilités Web abordées dans d'autres modules, car les paramètres exposés peuvent également être vulnérables à toute autre vulnérabilité.

Astuce : Pour une analyse plus précise, nous pouvons limiter notre analyse aux paramètres LFI les plus populaires trouvés sur ce [lien](https://book.hacktricks.xyz/pentesting-web/file-inclusion#top-25-parameters) .

* * * * *

Listes de mots LFI
--------------

Jusqu'à présent, dans ce module, nous avons créé manuellement nos charges utiles LFI pour tester les vulnérabilités LFI. En effet, les tests manuels sont plus fiables et peuvent trouver des vulnérabilités LFI qui pourraient ne pas être identifiées autrement, comme indiqué précédemment. Cependant, dans de nombreux cas, nous pouvons souhaiter exécuter un test rapide sur un paramètre pour voir s'il est vulnérable à une charge utile LFI commune, ce qui peut nous faire gagner du temps dans les applications Web où nous devons tester diverses vulnérabilités.

Il existe un certain nombre de [LFI Wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) que nous pouvons utiliser pour cette analyse. Une bonne liste de mots est [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt), car elle contient divers contournements et fichiers communs, de sorte qu'il facilite l'exécution de plusieurs tests à la fois. Nous pouvons utiliser cette liste de mots pour flouter le paramètre `?language=` que nous avons testé tout au long du module, comme suit :

```
dsgsec@htb[/htb]$ ffuf -w /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language= FUZZ' -fs 2287

...COUPER...

  :: Méthode : GET
  :: URL : http://<SERVER_IP>:<PORT>/index.php?FUZZ=key
  :: Liste de mots : FUZZ: /opt/useful/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt
  :: Suivre les redirections : false
  :: Calibrage : faux
  :: Délai d'attente : 10
  :: Fils : 40
  :: Matcher : état de la réponse : 200 204 301 302 307 401 403
  :: Filtre : Taille de la réponse : xxx
________________________________________________

..%2F..%2F..%2F%2F..%2F..%2Fetc/passwd [Statut : 200, Taille : 3661, Mots : 645, Lignes : 91]
../../../../../../../../../../../../etc/hosts [Statut : 200, Taille : 2461, Mots : 636, lignes : 72]
...COUPER...
../../../../etc/passwd [Statut : 200, Taille : 3661, Mots : 645, Lignes : 91]
../../../../../etc/passwd [Statut : 200, Taille : 3661, Mots : 645, Lignes : 91]
../../../../../../etc/passwd&=%3C%3C%3C%3C [Statut : 200, Taille : 3661, Mots : 645, Lignes : 91]
..%2F..%2F..%2F..%2F..%2F.