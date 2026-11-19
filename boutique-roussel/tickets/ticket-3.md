# [SYMPTOME]

Quand on choisit l'occasion « Merci », le bouquet « Le petit mot » n'apparaît pas, alors que c'est le bouquet le plus vendu pour dire merci. Il apparaît bien quand on affiche tous les bouquets.

# [ETAPES]

1. Ouvrir la boutique.
2. Choisir le filtre d'occasion « Toutes » : « Le petit mot » est affiché.
3. Choisir le filtre d'occasion « Merci » : les autres bouquets « merci » sont affichés, mais pas « Le petit mot ».

Les autres bouquets « merci » s'affichent : le code de tri fonctionne, et c'est la donnée de ce bouquet qui pose problème.

# [ATTENDU]

« Le petit mot » apparaît avec l'occasion « Merci » et avec chacune de ses autres occasions.

| Action | Attendu | Résultat après correction |
|---|---|---|
| Choisir « Merci » | « Le petit mot » apparaît avec les autres bouquets « merci » | OK |
| Choisir « Pour le plaisir » | « Le petit mot » apparaît toujours | OK |
| Choisir « Toutes » | tous les bouquets sont affichés | OK |
| Recharger la page (Ctrl+F5) | le JSON modifié est pris en compte | OK |

# [CODE_CONCERNE]

- [data/catalogue.json](../data/catalogue.json), bouquet « Le petit mot » : son occasion était écrite `"Merci"` (avec une majuscule), alors que l'id de l'occasion est `"merci"`.
- [js/app.js](../js/app.js), fonction `preparerFiltres` : chaque bouton d'occasion envoie l'`id` de l'occasion (`merci`), et non le texte affiché (`Merci`).
- [js/affichage.js](../js/affichage.js), fonction `afficherCatalogue` : le tri garde un bouquet si sa liste `occasions` contient l'id choisi (`includes`). La comparaison est exacte, donc `Merci` ne correspond pas à `merci`.

**Correction retenue** : corriger la donnée dans `catalogue.json`, sans modifier le code.

```json
"occasions": ["merci", "plaisir"],
```

C'est une erreur de saisie de la boutique, donc on corrige la donnée elle-même. Rendre le tri insensible aux majuscules aurait masqué l'erreur. Au ticket 1, cette tolérance était justifiée parce que la saisie venait de la cliente. Ici, les données sont écrites par la boutique et doivent être exactes.

**Vérification complémentaire** : les listes `occasions` des six bouquets ont été relues. Toutes leurs valeurs correspondent désormais à un `id` existant (anniversaire, amour, merci, mariage, plaisir).
