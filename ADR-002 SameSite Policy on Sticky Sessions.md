# ADR-002 Sticky sessions issue

## Issue

Recently Chrome changed their default SameSite policy to Lax.

This broke the StickySessions for some applications since they were using the domain as an API.

Should we set it to None and say that the security risks should be handled by the user ? Or accept to break user cases and keep it to Lax or none ?


```sequence
Utilisateur -> app.domain.com: get page
app.domain.com -> Utilisateur: Page statique
Utilisateur -> api.domain.com: Requete XHR connect
api.domain.com -> Container 1: connect
Container 1 -> api.domain.com: Response
api.domain.com -> Utilisateur: Response + sticky session cookie

Note left of Utilisateur: Le sticky session n'est pas pris \nen compte car cross-domain\n (car SameSite=Strict|Lax)

Utilisateur -> api.domain.com: Requete XHR 2 (sans le cookie)
api.domain.com -> Container 2: Requete
Container 2 -> api.domain.com: Erreur: Utilisateur pas connecté
api.domain.com -> Utilisateur: Erreur: Pas connecté + Nouveau sticky session cookie
```



```sequence
Utilisateur -> app.domain.com: get page
app.domain.com -> Utilisateur: Page statique
Utilisateur -> api.domain.com: Requete XHR connect
api.domain.com -> Container 1: connect
Container 1 -> api.domain.com: Response
api.domain.com -> Utilisateur: Response + sticky session cookie

Note left of Utilisateur: Le sticky session est pris \nen compte\n (car SameSite=None)

Utilisateur -> api.domain.com: Requete XHR 2 (avec le cookie)
api.domain.com -> Container 1: Requete
Container 1 -> api.domain.com: Réponse OK
api.domain.com -> Utilisateur: Réponse OK

Container 2 -> Container 2: 
```

   
### Alternatives

#### 1. Do Nothing

Consider it's not our responsibility, but it would break our feature of Sticky Sessions for people using different domains

We could give them solutions to counter :

- Add a route from their app to the problematic app

Pros:
 * Easier for us

Cons:
 * Non-standard
 * Will break apps

TODO:

- [ ] check if its really possible -> cookie forwarding ?

#### 2. Always Samesite=None

Pros:
 * Simple to implement
 * Will not break any app

Cons:
 * We could do better security wise
 * It makes the net less secure :cry: 


#### 3. Let the user choose the SameSite policy

Pros:
 * Better solution security wise

Cons:
 * Might break some customer apps during the migration (en fonction de la valeur par défault choisie)
     * Old apps: None
     * New apps: Strict
 * Take longer to develop (api, dashboard, CLI, appsdeck-nginx, scalingo-openresty)


