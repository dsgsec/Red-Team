Attaques SSRF contre d'autres systèmes back-end
-----------------------------------------------

Dans certains cas, le serveur d'applications est capable d'interagir avec des systèmes back-end qui ne sont pas directement accessibles par les utilisateurs. Ces systèmes ont souvent des adresses IP privées non routables. Les systèmes back-end sont normalement protégés par la topologie du réseau, de sorte qu'ils ont souvent une posture de sécurité plus faible. Dans de nombreux cas, les systèmes back-end internes contiennent des fonctionnalités sensibles accessibles sans authentification par toute personne capable d'interagir avec les systèmes.

Dans l'exemple précédent, imaginez qu'il y ait une interface d'administration à l'URL principale `https://192.168.0.68/admin`. Un attaquant peut soumettre la requête suivante pour exploiter la vulnérabilité SSRF et accéder à l'interface administrative:

`POST /product/stock HTTP/1.0 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 118 
stockApi=http://192.168.0.68/admin`
