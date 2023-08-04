Identification SSTI
===================

* * * * *

Nous pouvons détecter les vulnérabilités SSTI en injectant différentes balises dans les entrées que nous contrôlons pour voir si elles sont évaluées dans la réponse. Nous n'avons pas nécessairement besoin de voir les données injectées reflétées dans la réponse que nous recevons. Parfois, il est juste évalué sur différentes pages (en aveugle).

Le moyen le plus simple de détecter les injections consiste à fournir des expressions mathématiques entre accolades, par exemple :

Code : html

```
{7*7}
${7*7}
#{7*7}
%{7*7}
{{7*7}}
...

```

Nous rechercherons "49" dans la réponse lors de l'injection de ces charges utiles pour identifier que l'évaluation côté serveur s'est produite.

Le moyen le plus difficile d'identifier SSTI consiste à fuzzer le modèle en injectant des combinaisons de caractères spéciaux utilisés dans les expressions de modèle. Ces caractères incluent `${{<%[%'"}}%\`. Si une exception est provoquée, cela signifie que nous avons un certain contrôle sur ce que le serveur interprète en termes d'expressions de modèle.

Nous pouvons utiliser des outils tels que [Tplmap](https://github.com/epinna/tplmap) ou J2EE Scan (Burp Pro) pour tester automatiquement les vulnérabilités SSTI ou créer une liste de données utiles à utiliser avec Burp Intruder ou ZAP.

Le diagramme ci-dessous de [PortsSwigger](https://portswigger.net/research/server-side-template-injection) peut nous aider à identifier si nous avons affaire à une vulnérabilité SSTI et également à identifier le moteur de modèle sous-jacent.

![image](https://academy.hackthebox.com/storage/modules/145/img/ssti_diagram.png)

En plus du diagramme ci-dessus, nous pouvons essayer les approches suivantes pour reconnaître la technologie à laquelle nous avons affaire :

-   Vérifiez les erreurs détaillées pour les noms de technologie. Parfois, le simple fait de copier l'erreur dans la recherche Google peut nous fournir une réponse directe concernant la technologie sous-jacente utilisée
-   Vérifiez les extensions. Par exemple, les extensions .jsp sont associées à Java. Lorsque nous traitons avec Java, nous pouvons être confrontés à une vulnérabilité d'injection de langage d'expression/OGNL au lieu du SSTI traditionnel
-   Envoyez des expressions avec des accolades non fermées pour voir si des erreurs détaillées sont générées. N'essayez pas cette approche sur les systèmes de production, car vous pourriez planter le serveur Web.
