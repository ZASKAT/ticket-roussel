# [SYMPTOME]

Une cliente essaie d'utiliser le code promo `NOEL10` (lettre d'information) depuis son téléphone. Le site répond « Ce code promo n'existe pas. ». Sur ordinateur, le même code fonctionne. La cliente a commandé sans la réduction.

# [ETAPES]

Le bug se reproduit sans téléphone, en tapant ce qu'un clavier mobile produit souvent (majuscule automatique, espace ajouté par l'autocorrecteur) :

1. Ajouter un article au panier.
2. Dans le champ « Code promo », saisir une variante de `NOEL10` :
   - `Noel10` → refusé
   - `noel10` → refusé
   - `NOEL10 ` (espace à la fin) → refusé
3. Valider : le message « Ce code promo n'existe pas. » s'affiche.

Remarque : la vue mobile des outils de développement ne suffit pas à reproduire le bug, car elle ne simule pas le clavier du téléphone.

# [ATTENDU]

Le code doit être accepté quelle que soit la casse et malgré les espaces au début ou à la fin.

| Saisie | Attendu | Résultat après correction |
|---|---|---|
| `NOEL10` | appliqué (non-régression) | OK |
| `noel10`, `Noel10` | appliqué | OK |
| `NOEL10 `, ` NOEL10` | appliqué | OK |
| `N0EL10` (zéro) | refusé : autre code | OK |
| `BIENVENUE5`, panier < 30 € | message du minimum d'achat | OK |
| Rechargement de la page après `noel10` | remise conservée | OK |

# [CODE_CONCERNE]

- [js/promo.js](../js/promo.js), fonction `appliquerPromo` : la saisie était comparée au code **exactement**, caractère par caractère.
- [js/app.js](../js/app.js) : importe `appliquerPromo` depuis `promo.js` (ligne 6) et l'appelle à la validation du formulaire et au recalcul du panier avec le code sauvegardé.
- [js/promotions.js](../js/promotions.js) : une « version 2 » qui nettoie déjà la saisie, mais aucun fichier ne l'importe (code mort). Son interface est différente : elle prend une liste en paramètre et renvoie seulement un nombre, sans message.

**Correction retenue** : nettoyer la saisie dans `promo.js` avant la comparaison.

```js
const regle = promos.find((promo) => promo.code === saisie.trim().toUpperCase());
```

C'est une seule ligne dans un seul fichier, et les messages existants (code inexistant, minimum d'achat) sont conservés. Brancher `promotions.js` aurait demandé d'adapter `app.js` et aurait fait perdre le message du minimum d'achat.

**Incident pendant la correction** : une faute de frappe (`toUppercase` au lieu de `toUpperCase`) a cassé l'affichage, car le code sauvegardé est réappliqué à chaque rechargement. L'erreur était visible dans la console du navigateur (F12).

**Reste à traiter** :
- `promotions.js` est du code mort en doublon : le supprimer ou le signaler.
- `etat.codePromo` sauvegarde la saisie brute (ex. `noel10 `). Cela fonctionne puisqu'elle est nettoyée à chaque appel, mais on pourrait sauvegarder le code nettoyé.
