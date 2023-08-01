XSS
======================

* * * * *

Les vulnérabilités de type Cross-Site Scripting (XSS) affectent aussi bien les applications Web que les API. Une vulnérabilité XSS peut permettre à un attaquant d'exécuter du code JavaScript arbitraire dans le navigateur de la cible et entraîner une compromission complète de l'application Web si elle est enchaînée avec d'autres vulnérabilités. Notre module [Cross-Site Scripting (XSS)](https://academy.hackthebox.com/module/details/103) couvre XSS en détail.

Allez à la fin de cette section et cliquez sur `Click here to spawn the target system!`ou sur l' `Reset Target`icône . Utilisez la Pwnbox fournie ou une machine virtuelle locale avec la clé VPN fournie pour atteindre l'API cible et suivez-la.

Supposons que nous examinions mieux l'API de la section précédente, `http://<TARGET IP>:3000/api/download`.

Interagissons d'abord avec lui via le navigateur en demandant ce qui suit.

![](https://academy.hackthebox.com/storage/modules/160/6.png)

`test_value`se reflète dans la réponse.

Voyons ce qui se passe lorsque nous entrons dans une charge utile telle que ci-dessous (au lieu de *test_value* ).

Code : javascript

```
<script>alert(document.domain)</script>

```

![image](https://academy.hackthebox.com/storage/modules/160/9.png)

Il semble que l'application encode la charge utile soumise. Nous pouvons essayer d'encoder notre charge utile une fois par URL et de la soumettre à nouveau, comme suit.

Code : javascript

```
%3Cscript%3Ealert%28document.domain%29%3C%2Fscript%3E

```

![image](https://academy.hackthebox.com/storage/modules/160/10.png)

Maintenant, notre charge utile JavaScript soumise est évaluée avec succès. Le point de terminaison de l'API est vulnérable à XSS !
