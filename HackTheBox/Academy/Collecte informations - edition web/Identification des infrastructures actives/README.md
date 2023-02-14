# Identification des infrastructures actives

L'infrastructure d'une application Web est ce qui la fait fonctionner et lui permet de fonctionner. Les serveurs Web sont directement impliqués dans le fonctionnement de toute application Web. Certains des plus populaires sont Apache, Nginx et Microsoft IIS, entre autres.

Si nous découvrons le serveur Web derrière l'application cible, cela peut nous donner une bonne idée du système d'exploitation exécuté sur le serveur principal. Par exemple, si nous découvrons que la version IIS est en cours d'exécution, nous pouvons déduire la version du système d'exploitation Windows utilisée en mappant la version IIS à la version Windows sur laquelle elle est installée par défaut. Certaines installations par défaut sont :

+ IIS 6.0 : Windows Server 2003
+ IIS 7.0-8.5 : Windows Server 2008 / Windows Server 2008R2
+ IIS 10.0 (v1607-v1709) : Windows Server 2016
+ IIS 10.0 (v1809-) : Windows Server 2019

Bien que cela soit généralement correct lorsqu'il s'agit de Windows, nous ne pouvons pas être sûrs dans le cas des distributions basées sur Linux ou BSD car elles peuvent exécuter différentes versions de serveur Web dans le cas de Nginx ou Apache. Ce type de technologie pourrait interférer avec ou altérer nos futures activités de test. Néanmoins, si nous traitons avec un serveur Web, nous ne pourrons pas trouver d'empreintes digitales sur le serveur en HTML ou JS.

