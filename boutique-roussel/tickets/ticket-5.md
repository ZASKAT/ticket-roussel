# [SYMPTOME]

Une cliente avait trois bouquets différents dans son panier. Elle a voulu retirer le premier avec le bouton « Supprimer », et les trois ont disparu. Elle a cru à une panne et est partie sans commander.

# [ETAPES]

1. Mettre trois bouquets différents dans le panier (A, B, C).
2. Cliquer sur « Supprimer » sur la ligne A : tout le panier est vidé.

En faisant varier la position, une règle apparaît : la ligne supprimée **et toutes celles qui suivent** disparaissent.

| Essai (panier A, B, C) | Résultat avant correction |
|---|---|
| Supprimer A (premier) | panier vide |
| Supprimer B (milieu) | il reste A |
| Supprimer C (dernier) | il reste A et B (comportement normal, d'où un bug facile à rater) |
| Descendre B à 0 avec « − » | il reste A |

# [ATTENDU]

« Supprimer », ou descendre une quantité à 0 avec « − », retire uniquement la ligne concernée.

| Action (panier A, B, C) | Attendu | Résultat après correction |
|---|---|---|
| Supprimer A | il reste B et C | OK |
| Supprimer B | il reste A et C | OK |
| Supprimer C | il reste A et B (non-régression) | OK |
| Descendre B à 0 avec « − » | il reste A et C | OK |
| Supprimer les trois un par un | panier vide, message « panier vide » affiché | OK |
| Recharger la page après une suppression | les lignes restantes sont toujours là | OK |

# [CODE_CONCERNE]

- [js/panier.js](../js/panier.js), fonction `supprimerLigne` : elle retirait la ligne avec `lignes.splice(index)`. Sans deuxième argument, `splice` retire tous les éléments à partir de `index` jusqu'à la fin de la liste.
- [js/panier.js](../js/panier.js), fonction `changerQuantite` : elle appelle `supprimerLigne` quand la quantité tombe à 0, ce qui donnait le même bug avec le bouton « − ».
- [js/app.js](../js/app.js), fonction `quandActionPanier` : elle transmet la position de la ligne cliquée (`index`). Elle n'est pas en cause, car la position envoyée est la bonne.

**Correction retenue** : indiquer à `splice` le nombre de lignes à retirer ([panier.js:41](../js/panier.js#L41)).

```js
lignes.splice(index, 1);
```

La correction est dans `supprimerLigne`, que le bouton « Supprimer » et le bouton « − » utilisent tous les deux. Les deux cas sont donc réparés.

**Vérification** : le comportement de `splice` a été confirmé dans la documentation MDN. Le deuxième argument (`deleteCount`) donne le nombre d'éléments à retirer. S'il est absent, tout ce qui suit `index` est retiré.
