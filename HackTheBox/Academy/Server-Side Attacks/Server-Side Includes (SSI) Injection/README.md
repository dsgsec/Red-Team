Vue d'ensemble des inclusions côté serveur
==========================================

* * * * *

Les inclusions côté serveur ( `SSI`) sont une technologie utilisée par les applications Web pour créer du contenu dynamique sur les pages HTML avant le chargement ou pendant le processus de rendu en évaluant les directives SSI. Certaines directives SSI sont :

Code : html

```
// Date
<!--#echo var="DATE_LOCAL" -->

// Modification date of a file
<!--#flastmod file="index.html" -->

// CGI Program results
<!--#include virtual="/cgi-bin/counter.pl" -->

// Including a footer
<!--#include virtual="/footer.html" -->

// Executing commands
<!--#exec cmd="ls" -->

// Setting variables
<!--#set var="name" value="Rich" -->

// Including virtual files (same directory)
<!--#include virtual="file_to_include.html" -->

// Including files (same directory)
<!--#include file="file_to_include.html" -->

// Print all variables
<!--#printenv -->

```

L'utilisation de SSI sur une application Web peut être identifiée en vérifiant les extensions telles que .shtml, .shtm ou .stm. Cela dit, il existe des configurations de serveur autres que celles par défaut qui pourraient permettre à d'autres extensions (telles que .html) de traiter les directives SSI.

Nous devons soumettre des charges utiles à l'application cible, telles que celles mentionnées ci-dessus, via des champs de saisie pour tester l'injection SSI. Le serveur Web analysera et exécutera les directives avant de rendre la page si une vulnérabilité est présente, mais sachez que ces vulnérabilités peuvent également exister en format aveugle. Une injection SSI réussie peut conduire à extraire des informations sensibles de fichiers locaux ou même à exécuter des commandes sur le serveur Web cible.
