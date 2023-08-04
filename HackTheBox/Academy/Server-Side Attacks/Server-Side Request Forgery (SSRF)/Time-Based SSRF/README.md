SSRF basé sur le temps
======================

* * * * *

Nous pouvons également déterminer l'existence d'une vulnérabilité SSRF en observant les différences de temps dans les réponses. Cette méthode est également utile pour découvrir les services internes.

Soumettons le document suivant à l'application PDF de la section précédente et observons le temps de réponse.

Code : html

```
<html>
    <body>
        <b>Time-Based Blind SSRF</b>
        <img src="http://blah.nonexistent.com">
    </body>
</html>

```

![image](https://academy.hackthebox.com/storage/modules/145/img/blind_time.png)

Nous pouvons voir que le service a mis 10 secondes pour répondre à la demande. Si nous soumettons une URL valide dans le document HTML, la réponse prendra moins de temps. N'oubliez pas qu'il `internal.app.local`s'agissait d'une application interne valide (à laquelle nous pouvions accéder via SSRF dans la section précédente).

![image](https://academy.hackthebox.com/storage/modules/145/img/blind_time2.png)

Dans certaines situations, l'application peut échouer immédiatement au lieu de prendre plus de temps pour répondre. Pour cette raison, nous devons observer attentivement les différences de temps entre les demandes.
