# [SYMPTOME]

Une cliente fidèle, qui commandait déjà sur le site l'an dernier avec l'ancienne version, voit « NaN € » à la place des prix dans son panier et ne parvient plus à commander. Chez la fleuriste et chez les autres clientes, tout fonctionne.

`NaN` (*Not a Number*) signifie qu'un calcul a reçu une valeur qui n'est pas un nombre.

# [ETAPES]

Le panier est conservé d'une visite à l'autre dans le `localStorage` du navigateur (clé `panier-roussel`), propre à chaque navigateur. Le bug ne se voit donc que dans le navigateur de la cliente. Il a été reproduit en imitant un panier enregistré par l'ancienne version.

1. Ouvrir la console du navigateur (`F12`).
2. Enregistrer un panier au format supposé de l'ancienne version : une simple liste de lignes, sans `version`, et des lignes sans `quantite`. Par exemple :
   ```js
   localStorage.setItem("panier-roussel", JSON.stringify([{ bouquetId: "blanc-pur", options: { taille: "petit", vase: false, carte: false, message: "" } }]))
   ```
3. Recharger la page : les prix du panier affichent « NaN € ».

Le format réel de l'ancienne version n'est pas connu. Le code (`donnees.lignes || donnees`) indique seulement qu'il s'agissait d'une liste. En faisant varier une propriété à la fois :

| Panier enregistré | Résultat avant correction |
|---|---|
| Format actuel (`{ version: 2, lignes, codePromo }`) | prix corrects |
| Liste seule, lignes avec `quantite` | prix corrects |
| Ligne sans `quantite` | « NaN € » |
| Ligne avec un bouquet inconnu (`bouquetId: "tulipes"`) | prix de 0, pas `NaN` |

Vérification dans la console : `38.9 * 2` donne `77.8`, `38.9 * undefined` donne `NaN`.

# [ATTENDU]

Un panier enregistré par une autre version du site est ignoré : la page démarre avec un panier vide, sans `NaN`, et la commande fonctionne.

| Cas | Attendu | Résultat après correction |
|---|---|---|
| Ancien panier (liste sans `version`), puis recharger | panier vide, plus de `NaN` | OK |
| Panier `{ version: 1, lignes: [...] }`, puis recharger | panier vide | OK |
| Panier actuel : ajouter 2 bouquets, puis recharger | les 2 bouquets sont conservés (non-régression) | OK |
| Aucune sauvegarde (`localStorage.removeItem("panier-roussel")`) | panier vide, aucune erreur | OK |
| Après l'ancien panier : ajouter un bouquet, recharger, commander | bouquet conservé, commande confirmée | OK |
| Contenu enregistré après rechargement | contient `version: 2` | OK |

Pour dépanner la cliente sans attendre la mise en ligne : effacer les données du site dans son navigateur.

# [CODE_CONCERNE]

- [js/stockage.js](../js/stockage.js), fonction `chargerPanier` : elle relisait le panier enregistré sans vérifier son format. La sauvegarde contient pourtant un champ `version` (constante `VERSION = 2`), qui n'était jamais contrôlé à la lecture.
- [js/prix.js](../js/prix.js), fonction `prixLigne` : elle calcule `prixUnitaire(ligne) * ligne.quantite`. Sans `quantite`, le résultat est `NaN`, qui se propage au sous-total, à la livraison et au total. Elle n'est pas en cause : elle reçoit des données qu'elle n'attend pas.
- [js/panier.js](../js/panier.js), fonction `changerQuantite` : sur une ligne sans `quantite`, « + » calcule `undefined + 1` (`NaN`) et « − » ne fait jamais tomber la quantité à 0. Conséquence du même problème.
- Hypothèses écartées : un bouquet disparu du catalogue donne un prix de 0 (`bouquet ? bouquet.prix : 0`), pas `NaN`. Un navigateur trop ancien ou une erreur dans le catalogue auraient touché d'autres clientes.

**Correction retenue** (option A) : ignorer une sauvegarde d'une autre version ([stockage.js:19](../js/stockage.js#L19)).

```js
const donnees = JSON.parse(brut);
if (donnees.version !== VERSION) {
  return { lignes: [], codePromo: "" };
}
```

Un ancien panier est une simple liste : il n'a pas de propriété `version`, donc `donnees.version` vaut `undefined`, différent de `2`. Au premier affichage, `rafraichir` (`app.js`) appelle `sauverPanier`, qui remplace l'ancienne sauvegarde par le format actuel.

**Options écartées** :

| Option | Raison |
|---|---|
| B. Convertir l'ancien panier (ajouter `quantite: 1`) | Le format réel de l'ancienne version n'est pas connu : d'autres différences pourraient provoquer d'autres bugs. |
| C. `(ligne.quantite \|\| 1)` dans `prixLigne` | Masque le problème : la quantité affichée reste « undefined » et les boutons + et − restent cassés. |
| D. Option A + ignorer toute sauvegarde illisible ou mal formée | Plus de code. Reste possible plus tard. |

**Incident pendant la correction** : le bloc avait été écrit `if (donnees.version !== VERSION) =>`, sans accolades ni contenu. `=>` sert à écrire une fonction fléchée, pas un `if` : l'erreur de syntaxe cassait la page. Le bloc a été complété par l'IA, à la demande de l'élève.

**Reste à traiter** :
- Dans le `return` final, `donnees.lignes || donnees` : la partie `|| donnees` ne sert plus, puisque seul un panier version 2 arrive jusque-là.
- Une sauvegarde abîmée (texte qui n'est pas du JSON valide) ferait encore échouer `JSON.parse` et bloquerait le démarrage de la page (option D).
- La cliente perd son ancien panier : la fleuriste peut l'en informer.
