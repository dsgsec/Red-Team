Contourner l'authentification à deux facteurs
---------------------------------------------

Parfois, la mise en œuvre de l'authentification à deux facteurs est défectueuse au point où elle peut être contournée entièrement.

Si l'utilisateur est d'abord invité à entrer un mot de passe, puis invité à entrer un code de vérification sur une page séparée, l'utilisateur est effectivement dans un "logged in" indiquer avant d'avoir entré le code de vérification. Dans ce cas, il vaut la peine de tester si vous pouvez directement passer aux pages "connectées uniquement" après avoir terminé la première étape d'authentification. Parfois, vous constaterez qu'un site Web ne vérifie pas réellement si vous avez terminé la deuxième étape avant de charger la page.
