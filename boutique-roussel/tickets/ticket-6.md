# [SYMPTOME]

Le site annonce la livraison offerte dès 60 € d'achat. Une cliente de Tours a commandé un Blanc pur en taille moyenne, avec le vase. Son sous-total affichait 60,00 €, mais 5,90 € de livraison lui ont été facturés. La fleuriste l'a remboursée.

# [ETAPES]

1. Ajouter au panier un bouquet « Blanc pur » (38,90 €), en taille moyenne (+8,20 €), avec le vase (+12,90 €).
2. Saisir un code postal de Tours, par exemple 37000.
3. Le sous-total affiche 60,00 €, mais la livraison est facturée 5,90 € au lieu d'être offerte.

Vérification dans la console du navigateur (`F12`) :

| Saisie dans la console | Résultat avant correction |
|---|---|
| `38.9 + 8.2 + 12.9` | `59.99999999999999` |
| `38.9 + 8.2 + 12.9 >= 60` | `false` |

# [ATTENDU]

Un sous-total de 60,00 € donne droit à la livraison offerte.

| Cas | Attendu | Résultat après correction |
|---|---|---|
| Cas du ticket : Blanc pur, taille moyenne, avec vase, livré à Tours | sous-total 60,00 €, livraison 0,00 € | OK |
| Panier juste sous 60 € | livraison payante (non-régression) | OK |
| Panier au-dessus de 60 € | livraison 0,00 € (non-régression) | OK |
| Code promo NOEL10 | remise correcte (non-régression) | OK |
| Panier vide | aucune erreur dans la console | OK |
| Panier de 2 bouquets | sous-total correct | OK |

# [CODE_CONCERNE]

- [js/livraison.js](../js/livraison.js), fonction `fraisLivraison` : elle offre la livraison si `montant >= regles.seuilGratuite`. Elle n'est pas en cause : la comparaison est bien `>=`, et le seuil vaut bien 60 ([data/livraison.json](../data/livraison.json)).
- [js/app.js](../js/app.js), fonction `rafraichir` : elle passe à `fraisLivraison` le même `montant` que celui qui est affiché. Elle n'est pas en cause.
- [js/outils.js](../js/outils.js), fonction `formaterPrix` : elle arrondit le montant à 2 décimales **à l'affichage seulement**. C'est pour cela que l'écart était invisible.
- [js/prix.js](../js/prix.js), fonction `sousTotal` : elle additionnait les prix sans arrondir. En JavaScript, les nombres décimaux sont stockés en binaire et 38,9, 8,2 ou 12,9 ne peuvent pas y être représentés exactement. La somme valait donc `59.99999999999999`, et non 60, ce qui rendait la comparaison `>= 60` fausse.

**Correction retenue** (option C) : arrondir le sous-total au centime, à la source, avec la fonction `arrondir` qui existait déjà dans `outils.js` mais n'était utilisée nulle part ([prix.js:41](../js/prix.js#L41)).

```js
import { arrondir } from "./outils.js";

export function sousTotal(lignes) {
  return arrondir(lignes.reduce((total, ligne) => total + prixLigne(ligne), 0));
}
```

La livraison, la remise et le total reçoivent ainsi tous un sous-total exact au centime.

**Options écartées** :

| Option | Raison |
|---|---|
| A. Arrondir seulement dans l'appel à `fraisLivraison` (`app.js`) | Ne corrige que cet appel : les autres calculs gardent le montant imprécis. |
| B. Arrondir dans la comparaison de `fraisLivraison` (`livraison.js`) | Ne protège que la livraison. |
| D. Calculer tous les prix en centimes (nombres entiers) | Plus solide, mais modification trop large : données, calculs et affichage. |
| E. Comparer avec une tolérance (`montant >= seuil - 0.005`) | Bricolage, avec un nombre « magique » peu lisible. |

**Incidents pendant la correction** :
1. L'import était écrit `"./outils"`, sans l'extension `.js`. Le navigateur ne devine pas l'extension : il n'a pas trouvé le fichier et la page ne s'affichait plus. L'erreur était visible dans la console.
2. `arrondir` avait été placé à l'intérieur du `reduce`, et le `0` (la valeur de départ du total) était devenu un argument d'`arrondir`. Sans valeur de départ, `reduce` prend la première ligne du panier (un objet) comme total, et provoque une erreur sur un panier vide. Une première vérification avait conclu à tort que les tests passaient : la correction n'était pas enregistrée ou le navigateur utilisait encore l'ancienne version du fichier. La ligne a ensuite été corrigée par l'IA, à la demande de l'élève.

**Reste à traiter** : seul le sous-total est arrondi. La remise en pourcentage ([promo.js](../js/promo.js)) et le total (`totalCommande` dans `prix.js`) sont encore calculés avec des nombres non arrondis. L'affichage masque ces écarts, mais ils poseraient problème si ces valeurs étaient comparées à un seuil.
