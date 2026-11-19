# [SYMPTOME]

Une cliente avait choisi une livraison l'après-midi, parce qu'elle travaille le matin. Le message de confirmation disait « matin », et la commande reçue par la fleuriste aussi. Le livreur est passé le matin et n'a trouvé personne. La cliente a dû venir chercher son bouquet à la boutique.

# [ETAPES]

1. Mettre un bouquet dans le panier.
2. Saisir un code postal livré (par exemple 37000) et choisir un jour de livraison.
3. Cocher « Après-midi », puis cliquer sur « Commander » : la confirmation indique « matin ».

En faisant varier le choix, une règle apparaît : la confirmation dit **toujours** « matin », quel que soit le moment coché.

| Essai | Moment coché | Résultat avant correction |
|---|---|---|
| 1 | Après-midi | confirmation « matin » |
| 2 | Matin | confirmation « matin » (normal, d'où un bug facile à rater) |

Vérification dans la console du navigateur (`F12`), avec « Après-midi » coché :

| Saisie dans la console | Résultat |
|---|---|
| `document.querySelector("input[name=moment]").value` | `"matin"` |
| `document.querySelector("input[name=moment]:checked").value` | `"apres-midi"` |

# [ATTENDU]

La confirmation indique le moment réellement choisi par la cliente.

| Action | Attendu | Résultat après correction |
|---|---|---|
| Commander avec « Après-midi » coché | confirmation « après-midi » | OK |
| Commander avec « Matin » coché | confirmation « matin » (non-régression) | OK |
| Commander sans moment coché (simulé dans la console) | message « Choisissez un moment de livraison. », aucune erreur dans la console | OK |
| Commander avec un panier vide, sans code postal ou sans jour | messages habituels inchangés (non-régression) | OK |

Pour simuler l'absence de moment coché (un bouton radio ne se décoche pas au clic) :

```js
document.querySelectorAll("input[name=moment]").forEach((radio) => radio.checked = false)
```

# [CODE_CONCERNE]

- [js/app.js](../js/app.js), fonction `commander` : elle lisait le moment avec `document.querySelector("input[name=moment]")`. `querySelector` renvoie le **premier** élément de la page qui correspond au sélecteur, sans tenir compte de son état. Le premier bouton étant « Matin », la confirmation affichait toujours « matin ».
- [js/affichage.js](../js/affichage.js), fonction `afficherMoments` : elle crée les boutons dans l'ordre des données et coche « Matin » par défaut (`radio.checked = index === 0`). Elle n'est pas en cause.
- [data/livraison.json](../data/livraison.json), partie `moments` : les identifiants et libellés sont cohérents (`matin` / « Matin », `apres-midi` / « Après-midi »). Les données ne sont pas en cause.

**Correction retenue** (option D) : lire le bouton coché, et refuser la commande si aucun moment n'est choisi, sur le modèle de la vérification du jour ([app.js:123](../js/app.js#L123) et suivantes).

```js
const moment = document.querySelector("input[name=moment]:checked");
...
if (!moment) {
  message.textContent = "Choisissez un moment de livraison."
  return;
}
```

`:checked` limite la recherche aux boutons cochés. Si aucun ne l'est, `querySelector` renvoie `null` et la vérification évite une erreur sur la ligne de confirmation (`moment.nextSibling`).

**Options écartées** :

| Option | Raison |
|---|---|
| A. Ajouter `:checked` seul | Aucune protection si aucun bouton n'est coché : la confirmation provoquerait une erreur. |
| B. `querySelectorAll` puis chercher le bouton coché | Même résultat que A, en plus long. |
| C. Mémoriser le moment dans `etat` à chaque changement | Plus de code (un écouteur à ajouter) pour un besoin couvert par une ligne. |

**Incident pendant la correction** : la première version du bloc de vérification n'avait pas de `return;`. Le message « Choisissez un moment de livraison. » s'affichait, mais la fonction continuait et provoquait une erreur `TypeError: Cannot read properties of null` sur `moment.nextSibling`. Le test semblait réussi à l'écran ; l'erreur n'était visible que dans la console. Le `return;` a été ajouté par l'élève.

**Reste à traiter** :
- Style : le `;` manque à la fin de la ligne du message, un espace traîne après le `}`, et le bloc est entouré de lignes vides contrairement aux autres vérifications. Sans effet sur le fonctionnement.
- Hors code : un geste commercial éventuel pour la cliente, à l'appréciation de la fleuriste.
