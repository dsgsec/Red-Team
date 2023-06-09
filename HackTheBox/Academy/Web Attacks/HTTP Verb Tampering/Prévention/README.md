Prévention de la falsification des verbes
=========================

* * * * *

Après avoir vu quelques façons d'exploiter les vulnérabilités de Verb Tampering, voyons comment nous pouvons nous protéger contre ces types d'attaques en empêchant Verb Tampering. Les configurations non sécurisées et le codage non sécurisé introduisent généralement des vulnérabilités de falsification de verbe. Dans cette section, nous examinerons des exemples de code et de configurations vulnérables et discuterons de la manière dont nous pouvons les corriger.

* * * * *

Configuration non sécurisée
----------------------

Des vulnérabilités HTTP Verb Tampering peuvent se produire dans la plupart des serveurs Web modernes, y compris `Apache`, `Tomcat` et `ASP.NET`. La vulnérabilité se produit généralement lorsque nous limitons l'autorisation d'une page à un ensemble particulier de verbes/méthodes HTTP, ce qui laisse les autres méthodes restantes sans protection.

Voici un exemple de configuration vulnérable pour un serveur Web Apache, qui se trouve dans le fichier de configuration du site (par exemple `000-default.conf`) ou dans un fichier de configuration de page Web `.htaccess` :

Code : xml

```
<Directory "/var/www/html/admin">
    AuthType Basic
    AuthName "Admin Panel"
    AuthUserFile /etc/apache2/.htpasswd
    <Limit GET>
        Require valid-user
    </Limit>
</Directory>

```

Comme nous pouvons le voir, cette configuration définit les configurations d'autorisation pour le répertoire Web `admin`. Cependant, comme le mot-clé `<Limit GET>` est utilisé, le paramètre `Require valid-user` ne s'appliquera qu'aux requêtes `GET` , laissant la page accessible via les requêtes `POST` . Même si `GET` et `POST` étaient spécifiés, cela laisserait la page accessible par d'autres méthodes, comme `HEAD` ou `OPTIONS`.

L'exemple suivant montre la même vulnérabilité pour une configuration de serveur Web `Tomcat` , qui se trouve dans le fichier `web.xml` d'une certaine application Web Java :

Code : xml

```
<security-constraint>
    <web-resource-collection>
        <url-pattern>/admin/*</url-pattern>
        <http-method>GET</http-method>
    </web-resource-collection>
    <auth-constraint>
        <role-name>admin</role-name>
    </auth-constraint>
</security-constraint>

```

Nous pouvons voir que l'autorisation est limitée uniquement à la méthode `GET` avec `http-method`, qui laisse la page accessible via d'autres méthodes HTTP.

Enfin, voici un exemple de configuration `ASP.NET` trouvée dans le fichier `web.config` d'une application Web :

Code : xml

```
<system.web>
    <authorization>
        <allow verbs="GET" roles="admin">
            <deny verbs="GET" users="*">
        </deny>
        </allow>
    </authorization>
</system.web>
```

Encore une fois, la portée `allow` et `deny` se limite à la méthode `GET` , qui laisse l'application Web accessible via d'autres méthodes HTTP.

Les exemples ci-dessus montrent qu'il n'est pas sûr de limiter la configuration d'autorisation à un verbe HTTP spécifique. C'est pourquoi nous devons toujours éviter de restreindre l'autorisation à une méthode HTTP particulière et toujours autoriser/refuser tous les verbes et méthodes HTTP.

Si nous voulons spécifier une seule méthode, nous pouvons utiliser des mots clés sûrs, comme `LimitExcept` dans Apache, `http-method-omission `dans Tomcat et `add`/`remove` dans ASP.NET, qui couvrent tous les verbes sauf les spécifiés.

Enfin, pour éviter des attaques similaires, nous devrions généralement "envisager de désactiver/refuser toutes les requêtes HEAD", sauf si l'application Web l'exige spécifiquement.

* * * * *

Codage non sécurisé
---------------

S'il est relativement facile d'identifier et de corriger les configurations de serveur Web non sécurisées, il est beaucoup plus difficile d'en faire de même pour le code non sécurisé. En effet, pour identifier cette vulnérabilité dans le code, nous devons trouver des incohérences dans l'utilisation des paramètres HTTP entre les fonctions, car dans certains cas, cela peut conduire à des fonctionnalités et des filtres non protégés.

Considérons le code `PHP` suivant de notre exercice `Gestionnaire de fichiers`  :

Code : php

```
if (isset($_REQUEST['filename'])) {
    if (!preg_match('/[^A-Za-z0-9. _-]/', $_POST['filename'])) {
        system("touch " . $_REQUEST['filename']);
    } else {
        echo "Malicious Request Denied!";
    }
}

```

Si nous ne considérions que les vulnérabilités d'injection de commande, nous dirions que cela est codé de manière sécurisée. La fonction `preg_match` recherche correctement les caractères spéciaux indésirables et n'autorise pas l'entrée dans la commande si des caractères spéciaux sont trouvés. Cependant, l'erreur fatale commise dans ce cas n'est pas due aux injections de commandes, mais à l' `utilisation incohérente des méthodes HTTP`.

Nous constatons que le filtre `preg_match` vérifie uniquement les caractères spéciaux dans les paramètres `POST` avec `$_POST['filename']`. Cependant, la commande `system` finale utilise la variable `$_REQUEST['filename']` , qui couvre à la fois les paramètres `GET` et `POST` . Ainsi, dans la section précédente, lorsque nous envoyions notre entrée malveillante via une requête `GET` , elle n'était pas arrêtée par la fonction `preg_match` , car les paramètres `POST` étaient vides et ne contenaient donc aucun caractère spécial. Cependant, une fois que nous avons atteint la fonction `system` , tous les paramètres trouvés dans la requête ont été utilisés, et nos paramètres `GET` ont été utilisés dans la commande, ce qui a finalement conduit à l'injection de commande.

Cet exemple de base nous montre comment des incohérences mineures dans l'utilisation des méthodes HTTP peuvent conduire à des vulnérabilités critiques. Dans une application Web de production, ces types de vulnérabilités ne seront pas aussi évidents. Ils seraient probablement répartis sur l'application Web et ne seraient pas sur deux lignes consécutives comme nous l'avons ici. Au lieu de cela, l'application Web aura probablement une fonction spéciale pour vérifier les injections et une fonction différente pour créer des fichiers. Cette séparation du code rend difficile la détection de ces types d'incohérences et, par conséquent, elles peuvent survivre jusqu'à la production.

Pour éviter les vulnérabilités HTTP Verb Tampering dans notre code, `nous devons être cohérents avec notre utilisation des méthodes HTTP` et nous assurer que la même méthode est toujours utilisée pour toute fonctionnalité spécifique dans l'application Web. Il est toujours conseillé d' `étendre la portée des tests dans les filtres de sécurité` en testant tous les paramètres de requête. Cela peut être fait avec les fonctions et variables suivantes :

| Langue | Fonction |
| --- | --- |
| PHP | `$_REQUEST['param']` |
| Java | `request.getParameter('param')` |
| C# | `Demande['param']` |

Si notre portée dans les fonctions liées à la sécurité couvre toutes les méthodes, nous devons éviter de telles vulnérabilités ou contournements de filtres.
