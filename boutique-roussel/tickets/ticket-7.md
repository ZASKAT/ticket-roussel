# [SYMPTOME]

Une cliente voulait écrire un long message sur la carte, pour un mariage. Le site l'a bloquée au milieu de sa phrase. La fleuriste demande d'enlever cette limite pour que ses clientes puissent écrire ce qu'elles veulent.

Ce ticket est une **demande de changement**, pas un calcul faux. Le diagnostic a montré que la limite est volontaire. Le vrai défaut est que la cliente n'est pas prévenue.

# [ETAPES]

1. Choisir un bouquet et cocher « Carte avec votre message ».
2. Coller ou taper un long texte dans le champ « Votre message ».
3. La saisie s'arrête à 150 caractères, sans message ni compteur. Taper au clavier ne produit plus rien.

Pour compter les caractères, dans la console du navigateur (`F12`) :

```js
document.querySelector("#message-carte").value.length
```

| Essai | Résultat avant correction |
|---|---|
| Taper au-delà de 150 caractères | la saisie bloque, sans avertissement |
| Coller un texte de plus de 150 caractères | le texte est coupé à 150, sans avertissement |
| Indication de la place restante | aucune |

# [ATTENDU]

La limite de 150 caractères est conservée, car c'est la place disponible sur la carte. La cliente voit en permanence combien de caractères elle a utilisés.

| Action | Attendu | Résultat après correction |
|---|---|---|
| Cocher la carte | le compteur affiche « 0 / 150 caractères » | OK |
| Taper du texte | le compteur augmente à chaque caractère | OK |
| Effacer du texte | le compteur diminue | OK |
| Coller un long texte | le compteur s'arrête à « 150 / 150 caractères » | OK |
| Décocher la carte | le compteur disparaît avec le champ | OK |
| Ouvrir un autre bouquet | le compteur revient à « 0 / 150 caractères » | OK |
| Ajouter au panier avec un message | le panier fonctionne comme avant (non-régression) | OK |

# [CODE_CONCERNE]

- [data/catalogue.json](../data/catalogue.json), option `carte` : `"longueurMax": 150`. La limite est rangée dans les données, à côté du prix et du libellé de la carte. Elle correspond à la taille physique de la carte. Elle n'est pas en cause et n'a pas été modifiée.
- [js/composition.js](../js/composition.js) applique cette limite à deux endroits :
  - ligne 36 : `maxLength = reglesOptions.carte.longueurMax` règle l'attribut HTML `maxlength` du champ. Le navigateur refuse tout caractère au-delà de 150, au clavier comme au collage, sans rien afficher. C'est ce qui « bloque sans prévenir » ;
  - ligne 79 : `message.slice(0, reglesOptions.carte.longueurMax)` ne garde que les 150 premiers caractères du message enregistré. C'est une sécurité au cas où le champ contiendrait plus, par exemple après une modification de la page avec les outils du navigateur.

**Correction retenue** (option C) : afficher un compteur sous le champ.

Dans [index.html](../index.html#L65), dans `#zone-message`, sous le `<textarea>` :

```html
<p id="compteur-carte" class="mt-1 text-sm"></p>
```

Dans [js/composition.js](../js/composition.js), fonction `lireFormulaire`, appelée à chaque frappe (lignes 77-78) :

```js
document.querySelector("#compteur-carte").textContent =
  message.length + " / " + reglesOptions.carte.longueurMax + " caractères";
```

Et dans `remplirFormulaire`, appelée à l'ouverture d'un bouquet, pour remettre le compteur à zéro (lignes 65-66) :

```js
document.querySelector("#compteur-carte").textContent =
  options.message.length + " / " + reglesOptions.carte.longueurMax + " caractères";
```

Le compteur lit la limite dans `catalogue.json` : si la valeur change, le compteur suit sans autre modification.

**Options écartées** :

| Option | Raison |
|---|---|
| A. Supprimer la limite, comme demandé | Les messages trop longs ne tiendraient pas sur la carte : le problème passerait de la cliente à la fleuriste. |
| B. Augmenter la valeur dans `catalogue.json` | La capacité réelle de la carte est inconnue, et la cliente resterait bloquée sans être prévenue, simplement plus loin. |
| D. B et C ensemble | Demande de connaître la capacité réelle de la carte. Reste possible plus tard. |

**Incident pendant la correction** : les deux lignes du compteur étaient incomplètes dans `composition.js` (`document.querySelector("#compteur-carte")textContent ;`) : il manquait le point avant `textContent`, le `=` et le texte à afficher. La page était cassée. Les lignes ont été complétées par l'IA, à la demande de l'élève.

**Reste à traiter** :
- Répondre à la fleuriste : la limite est conservée parce que le message doit tenir sur la carte, et la cliente est désormais prévenue par le compteur.
- Lui demander combien de caractères tiennent réellement sur la carte. Si c'est plus de 150, il suffit de changer `longueurMax` dans `catalogue.json`.
