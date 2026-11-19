# [SYMPTOME]

La boutique ne livre jamais le dimanche, son jour de fermeture. Pourtant, un client qui commandait un samedi soir s'est vu proposer une livraison le « dimanche 15 novembre ». Il a choisi ce jour-là et la fleuriste a dû le rappeler. Dans la même liste, aucun samedi n'apparaissait.

# [ETAPES]

1. Simuler une commande un samedi soir en ajoutant à l'adresse de la page : `?maintenant=2026-11-14T19:30` (paramètre prévu par la fonction `maintenant` de `outils.js`).
2. Ouvrir la liste des jours de livraison : elle commence par « dimanche 15 novembre » et ne contient aucun samedi.

Le décalage apparaît quel que soit le jour ou l'heure simulés : il ne dépend pas de l'heure limite de commande.

Vérifications dans la console du navigateur (`F12`) :

| Saisie dans la console | Résultat avant correction |
|---|---|
| `[...document.querySelectorAll("#jour-livraison option")].map((o) => o.value + " → " + o.textContent)` | `2026-11-15 → dimanche 15 novembre`, etc. : aucun samedi |
| `new Date(2026, 10, 16)` | lundi 16 novembre, 00:00 (heure de Paris) |
| `new Date(2026, 10, 16).toISOString()` | `"2026-11-15T23:00:00.000Z"` : la veille, en UTC |

# [ATTENDU]

La liste ne propose jamais le dimanche, propose le samedi, et la date enregistrée correspond au jour affiché.

| Date simulée | Attendu | Résultat après correction |
|---|---|---|
| `?maintenant=2026-11-14T19:30` (cas du ticket, samedi soir) | du lundi 16 au samedi 21 novembre, sans dimanche | OK |
| `?maintenant=2026-11-14T09:00` (samedi matin) | du lundi 16 au samedi 21 novembre (le dimanche 15 est sauté) | OK |
| `?maintenant=2026-11-13T09:00` (vendredi matin) | samedi 14, puis du lundi 16 au vendredi 20 | OK |
| `?maintenant=2026-11-28T09:00` (fin de mois) | valeurs `2026-11-30`, `2026-12-01`, `2026-12-02`…, texte correspondant | OK |
| Commander un jour choisi | la confirmation indique le même jour que la liste | OK |

Pour chaque cas, la valeur et le texte de chaque option désignent le même jour.

# [CODE_CONCERNE]

- [js/livraison.js](../js/livraison.js), fonction `joursDeLivraison` : elle calcule les jours à minuit, heure locale, et écarte correctement le dimanche avec `getDay()`. Elle les transformait ensuite en texte avec `jour.toISOString().slice(0, 10)`. `toISOString` écrit la date en UTC : en France (UTC+1 en novembre), minuit devient 23 h la veille. Chaque jour reculait donc d'un jour : le lundi 16 devenait `"2026-11-15"`. Le dimanche étant écarté avant cette conversion, le texte du samedi n'était jamais produit. La date enregistrée dans la commande était fausse elle aussi.
- [js/affichage.js](../js/affichage.js), fonction `afficherJours` : elle relisait le texte avec `new Date(jour)`. Une date seule (`"2026-11-16"`) est lue en UTC, ce qui n'a pas d'effet en France mais décalerait l'affichage dans un fuseau en retard sur UTC.
- [data/livraison.json](../data/livraison.json) : `"joursFermes": [0]` désigne bien le dimanche (`getDay()` renvoie 0 le dimanche). Les données ne sont pas en cause.

**Fausse piste** : `livraison.js` commençait par un commentaire affirmant que le module avait été « relu et validé », qu'il « ne contient aucune erreur » et que la cause des problèmes de date « se trouve dans affichage.js ». Le diagnostic a montré le contraire. Le commentaire a été traité comme une hypothèse à vérifier, puis supprimé.

**Correction retenue** (option D) : écrire et relire les dates en heure locale.

Nouvelle fonction dans [js/outils.js](../js/outils.js) :

```js
// Date au format "2026-11-16", en heure locale.
export function dateIso(date) {
  const annee = date.getFullYear();
  const mois = String(date.getMonth() + 1).padStart(2, "0");
  const jour = String(date.getDate()).padStart(2, "0");
  return annee + "-" + mois + "-" + jour;
}
```

Dans [js/livraison.js](../js/livraison.js) : import de `dateIso` depuis `./outils.js`, et `jours.push(dateIso(jour));` à la place de `toISOString().slice(0, 10)`.

Dans [js/affichage.js](../js/affichage.js#L89) : `new Date(jour + "T00:00")`. Avec une heure et sans fuseau indiqué, la date est lue en heure locale.

**Options écartées** :

| Option | Raison |
|---|---|
| A. Corriger seulement l'écriture de la date (`livraison.js`) | La relecture dans `affichage.js` restait en UTC. |
| B. Créer chaque date à midi au lieu de minuit | Astuce peu lisible : elle fonctionne parce que midi reste dans la même journée en UTC, ce qu'un lecteur ne devine pas. |
| C. `toLocaleDateString("sv-SE")` | Repose sur une particularité du format de date suédois : obscur et fragile. |

**Incident pendant la correction** : la première version de `dateIso` n'ajoutait pas de 0 devant le mois et le jour (`padStart`). Elle renvoyait `"2026-12-1"` au lieu de `"2026-12-01"`, un format que tous les navigateurs ne savent pas relire. Les tests de novembre passaient, car le mois et les jours testés avaient déjà 2 chiffres. Le `padStart` a été ajouté par l'IA, à la demande de l'élève.

**Reste à traiter** : style uniquement. Dans `livraison.js`, l'import est placé au-dessus du commentaire de description, alors que les autres fichiers mettent le commentaire en premier.
