# [SYMPTOME]

Quand une cliente clique sur le bouton « + » d'une ligne du panier pour ajouter un exemplaire, le panier en ajoute plusieurs d'un coup, et de plus en plus à chaque clic. Une cliente s'est retrouvée avec une quinzaine de bouquets et ne l'a vu qu'au total, avant de commander.

# [ETAPES]

1. Ajouter un bouquet au panier.
2. Cliquer plusieurs fois sur « + » sur la ligne du panier.
3. Noter la quantité après chaque clic : elle double au lieu d'augmenter de 1.

Pour confirmer la cause dans le navigateur :

1. Ouvrir les outils (F12), onglet **Éléments**, et sélectionner `<ul id="panier-liste">`.
2. Dans l'onglet **Écouteurs d'événements**, ouvrir la ligne `click`.
3. Compter les écouteurs, cliquer sur « + », puis recompter : leur nombre augmente à chaque clic.

# [ATTENDU]

Chaque clic sur « + » ajoute exactement un exemplaire, chaque clic sur « − » en retire exactement un, et « Supprimer » retire une seule ligne.

| Action | Attendu | Résultat après correction |
|---|---|---|
| Ajouter un bouquet, puis cliquer 5 fois sur « + » | 1 → 2 → 3 → 4 → 5 → 6 | OK |
| Cliquer sur « − » | −1 par clic ; à 0, la ligne disparaît | OK |
| Cliquer sur « Supprimer » | une seule ligne supprimée | OK |
| Deux bouquets différents, « + » sur le 2e | seul le 2e change | OK |
| Recharger la page, puis cliquer sur « + » | +1 | OK |
| Écouteurs d'événements sur `#panier-liste` | un seul écouteur `click`, même après plusieurs clics | OK |

# [CODE_CONCERNE]

- [js/affichage.js](../js/affichage.js), fonction `afficherPanier` : à chaque affichage, elle vidait la liste avec `replaceChildren()` puis ajoutait un écouteur de clic sur la liste `#panier-liste`. `replaceChildren()` retire les lignes **à l'intérieur** de la liste, mais pas les écouteurs posés **sur** la liste : ils s'accumulaient.
- [js/app.js](../js/app.js) : `quandActionPanier` applique l'action (+1, −1, supprimer), puis appelle `rafraichir()`, qui rappelle `afficherPanier`.
- [js/panier.js](../js/panier.js), fonction `changerQuantite` : elle fait correctement +1 ou −1. Elle n'est pas en cause, c'est le nombre de fois où elle est appelée qui augmentait.

**Mécanisme** : avec N écouteurs, un clic exécute N fois « +1, puis réaffichage ». Chaque réaffichage ajoute un écouteur, donc on passe à 2N écouteurs : la quantité ajoutée double à chaque clic.

**Correction retenue** : poser l'écouteur **une seule fois**, au démarrage, dans la fonction `demarrer()` d'`app.js`, et le retirer d'`afficherPanier`.

```js
document.querySelector("#panier-liste").addEventListener("click", (evenement) => {
  const cible = evenement.target.closest("button[data-action]");
  if (!cible) return;
  quandActionPanier(cible.dataset.action, Number(cible.dataset.index));
});
```

La liste `#panier-liste` existe dès le chargement de la page et n'est jamais recréée, donc un seul écouteur suffit. Deux autres options ont été écartées : un écouteur sur chaque bouton (il faudrait modifier la fonction `bouton`), ou retirer l'ancien écouteur avant d'en ajouter un nouveau (plus complexe).

**Incident pendant la correction** : une ligne mal recopiée (`.addEventListener(...)` effacé et `});` remplacé par `};`) a provoqué une erreur de syntaxe qui empêchait toute la page de se charger. L'erreur a été retrouvée en comparant avec la version d'origine dans le Contrôle de source de VS Code.

**Reste à traiter** : le paramètre `surAction` d'`afficherPanier` n'est plus utilisé. On peut le retirer, ainsi que l'argument `quandActionPanier` passé dans `rafraichir()`.
