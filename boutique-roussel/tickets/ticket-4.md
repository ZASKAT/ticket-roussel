# [SYMPTOME]

Une cliente voulait deux « Pivoine poudrée » : une petite et une grande. Elle a composé la petite et l'a ajoutée au panier, puis elle a choisi « Grand » et a cliqué de nouveau sur « Ajouter au panier ». Son panier contenait deux grandes.

# [ETAPES]

1. Ouvrir « Pivoine poudrée », laisser la taille « Petit » et cliquer sur « Ajouter au panier » : le panier contient une petite.
2. Choisir « Grand » **sans cliquer** sur « Ajouter au panier » : la ligne déjà dans le panier passe en « Grand ».
3. Cliquer sur « Ajouter au panier » : le panier contient une seule ligne « Grand » avec une quantité de 2.

Si on recharge la page entre l'étape 1 et l'étape 2, le bug ne se produit pas.

# [ATTENDU]

Une fois ajouté au panier, un bouquet garde la composition choisie au moment de l'ajout, même si la cliente modifie ensuite le formulaire.

| Action | Attendu | Résultat après correction |
|---|---|---|
| Ajouter une petite Pivoine poudrée, puis choisir « Grand » sans ajouter | la ligne du panier reste « Petit » | OK |
| Cliquer ensuite sur « Ajouter au panier » | deux lignes : une petite et une grande | OK |
| Ajouter deux fois exactement la même composition | une seule ligne avec une quantité de 2 (non-régression) | OK |
| Cocher le vase après avoir ajouté un bouquet | la ligne du panier ne gagne pas de vase | OK |
| Recharger la page | les deux lignes sont toujours là | OK |

# [CODE_CONCERNE]

- [js/composition.js](../js/composition.js) : un **seul** objet `options` (taille, vase, carte, message) sert à tout le formulaire. `lireFormulaire` le modifie à chaque changement, et `ajouter` l'envoie au panier.
- [js/panier.js](../js/panier.js), fonction `ajouterAuPanier` : elle rangeait cet objet tel quel dans la nouvelle ligne (`{ bouquetId, options, quantite: 1 }`).
- [js/stockage.js](../js/stockage.js) : au rechargement, le panier est relu depuis une sauvegarde texte (JSON), ce qui recrée des objets neufs.

**Mécanisme** : en JavaScript, ranger un objet dans une liste ne le copie pas. La ligne du panier et le formulaire partageaient donc le même objet `options`.

1. Choisir « Grand » modifiait aussi la ligne déjà dans le panier.
2. Au deuxième ajout, `memesOptions` comparait cet objet avec lui-même. Les options étaient forcément identiques, et la quantité passait à 2.
3. Au rechargement, les objets sont recréés depuis la sauvegarde, ce qui coupe le lien : le bug disparaît.

**Correction retenue** : le panier range une **copie** des options dans chaque nouvelle ligne ([panier.js:26](../js/panier.js#L26)).

```js
lignes.push({ bouquetId, options: { ...options }, quantite: 1 });
```

Une autre option était de faire la copie dans `ajouter` (composition.js). Elle a été écartée : avec la copie dans le panier, une ligne ne peut plus être modifiée de l'extérieur, quelle que soit la partie du site qui ajoute un bouquet.
