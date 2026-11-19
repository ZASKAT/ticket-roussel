# Historique des prompts

**Prompt donné a ChatGPT gratuit pour qu'il l'optimise pour Claude Opus 5.5 Medium**

Synthetise en prompt .md sans inventer

Ceci est un exercise pour moi, je dois apprendre a utilisé une IA pour explorer un code inconnu.

Dans ce dossier, il y a des pdf qui expliquent les étapes et les attendus en forme de cahier des charges, **NE JAMAIS LES LIRES !**

Je vais te donner étape par étape les consignes que je vais renseigner dans diagnostic.md moi même.

Il y a des erreurs dans ce code:

- **NE JAMAIS ME DONNER LES REPONSES DIRECTEMENT**

- **FAIRE DES HYPOTHESES SUR LES ERREURS**

# Prompt – Exploration d'un code inconnu avec une IA (Optimisé donné par ChatGPT-luna)
## Contexte et objectif

Cet exercice a pour objectif de m'apprendre à utiliser une IA pour explorer, comprendre et diagnostiquer un code informatique que je ne connais pas.

## Règles impératives
### 1. Documentation du projet
- Des fichiers PDF sont présents dans le dossier du projet.
- **NE JAMAIS LIRE, OUVRIR OU ANALYSER CES PDF.**
- Ils contiennent des explications sur les étapes à suivre et les attendus sous forme de cahier des charges.
### 2. Consignes de travail
- Je fournirai les consignes progressivement, étape par étape.
- Je renseignerai moi-même ces consignes dans le fichier diagnostic.md.
- Ne pas anticiper les étapes suivantes.
### 3. Approche pédagogique
- **NE JAMAIS ME DONNER DIRECTEMENT LES RÉPONSES.**
- Ne pas corriger directement les erreurs à ma place.
- M'accompagner dans l'exploration et la compréhension du code.
- Formuler des hypothèses sur les erreurs présentes dans le code.
- Me laisser analyser les hypothèses et progresser par moi-même.

## Objectif final

M'apprendre à utiliser une IA comme outil d'exploration et de diagnostic d'un code inconnu, sans qu'elle réalise le travail de réflexion et de résolution à ma place.

# Réponse:
J'ai lu ton fichier [diagnostic.md](boutique-roussel/prompts/diagnostic.md) et je vais suivre ces règles pendant tout l'exercice :

- **Les PDF** : je ne les ouvrirai pas, je ne les lirai pas et je ne les analyserai pas.
- **Étape par étape** : j'attends la consigne que tu écriras dans diagnostic.md, et je ne prends pas d'avance sur les suivantes.
- **Pas de réponses toutes faites** : je ne corrige pas le code à ta place. Je te propose des hypothèses, des questions et des pistes pour chercher, et c'est toi qui fais l'analyse et qui conclus.

J'ai aussi enregistré ces règles dans ma mémoire pour ce projet, pour qu'elles restent valables dans les prochaines sessions.

Pour l'instant, le fichier ne contient que les règles et aucune consigne d'étape, donc je n'ai pas encore regardé le code. Quand tu as écrit la première consigne dans diagnostic.md, préviens-moi et on commence.

Un conseil pour bien démarrer : avant chaque question que tu me poses, note ce que tu penses déjà. Ensuite, compare avec mes hypothèses. C'est comme ça que tu progresseras le plus.

# Ticket 1:
Bonjour, une cliente m’a appelée hier. Elle a voulu utiliser le code NOEL10 de ma lettre d’information 
depuis son téléphone, et le site lui a répondu que ce code n’existe pas. Sur mon ordinateur, le code 
marche très bien. Elle a fini par commander sans la réduction et elle n’était pas contente. Pouvez-vous 
regarder ?
Il semblerai que le problème vienne soit de prix, soit de promo, soit de promotion

Pourtant je n'arrive pas a répliquer le bug, je pense que c'est lié au téléphone

# Réponse: 
Bonne intuition sur le téléphone, mais il faut aller plus loin. Le téléphone en lui-même n'est pas le bug : la vraie question est **ce que la cliente a réellement tapé, et qui est différent de ce que tu tapes sur ton ordinateur**.

## Indices dans le ticket
- La cliente a reçu le message **« ce code n'existe pas »**. Cherche ce texte exact dans le projet. Il te dira quelle fonction a répondu.
- Sur ton ordinateur, ça marche. Le code `NOEL10` est donc bien enregistré. C'est plutôt la **comparaison** entre la saisie et la liste qui pose question.

## Hypothèses à tester
1. **Le clavier du téléphone modifie la saisie sans qu'on le voie.** Pense à ce qu'un clavier mobile fait automatiquement au début d'un champ texte, ou quand on choisit une suggestion du correcteur. Essaie de reproduire ces modifications à la main sur ton ordinateur.
2. **Deux fichiers semblent faire le même travail.** Ouvre [promo.js](boutique-roussel/js/promo.js) et [promotions.js](boutique-roussel/js/promotions.js) et lis le commentaire en haut de chacun. Puis réponds à ces questions :
   - Lequel dit vouloir résoudre précisément ce problème ?
   - Lequel est **réellement utilisé** par l'application ? Regarde les `import` dans [app.js](boutique-roussel/js/app.js).
3. **L'outil de développement du navigateur** : il peut simuler un téléphone (vue mobile). Est-ce que ça suffit pour reproduire le bug, d'après toi ? Pourquoi ?

## À faire maintenant
- Essaie de reproduire le bug **sans téléphone**, en tapant des variantes de `NOEL10` sur ton ordinateur.
- Note dans diagnostic.md les variantes qui échouent et celles qui marchent.

Ensuite, explique-moi **pourquoi**, selon toi, elles échouent. On regardera ensemble si ton raisonnement tient avant de parler de correction.

# Ma réponse

1. la cliente a peut être mis un espace après NOEL10 ou son autocorrecteur lui a écrit Noel10.

2. promotions.js dit "Les codes sont comparés sans tenir compte des majuscules ni des espaces" alors que c'est faux, dans app.js il n'y a que promo.js qui est import.

3. L'outil de développement du navigateur ne suffit pas a vérifier le bug car le problème viens de la saisie d'un clavier d'un téléphone.

## Tests: 

- NOEL10 avec un espace après ne marche pas 
- N0EL10 ne marche pas 
- noel10 ne marche pas 
- Noel10 ne marche pas 

Je pense que promotions.js soit est bug soit n'est relié a rien (le fichier n'ai jamais appelé/obelete).

# Réponse:

Très bon travail : tu as reproduit le bug sans téléphone, et c'était l'étape la plus importante. Voici mes retours, point par point.

## Ce qui tient
- **Hypothèse 1** : tes tests la confirment. La saisie est comparée **telle quelle**, caractère par caractère. Un espace de trop ou une minuscule suffit à faire échouer le code.
- **Hypothèse 3** : c'est juste. La vue mobile du navigateur change la taille de l'écran, mais pas le clavier.

## Ce qu'il faut revoir
- **Le point 2 est à nuancer.** Tu dis que le commentaire de promotions.js est « faux ». Relis la ligne `const saisie = ...` dans [promotions.js](boutique-roussel/js/promotions.js) :
  - Que font `trim()` et `toUpperCase()` ?
  - Le code fait-il vraiment l'inverse de ce que dit le commentaire ?
  - Ou bien est-ce que le problème est ailleurs ?
- **Le test `N0EL10` (avec un zéro)** n'a pas le même statut que les trois autres. Ce code est réellement différent de `NOEL10`. Est-ce qu'on *doit* l'accepter ? Range-le à part dans tes notes.
- **« Jamais appelé »** : pour l'instant tu as seulement regardé app.js. Comment vérifier ça sur **tout** le projet ? C'est une bonne occasion d'utiliser la recherche globale de VS Code (`Ctrl+Shift+F`).

## Pour l'étape suivante
Avant de penser à corriger, compare les deux fonctions `appliquerPromo` de promo.js et de promotions.js :
1. Elles n'ont pas le même nombre de paramètres. Lesquels ?
2. Elles ne renvoient pas la même chose. Regarde comment app.js utilise le résultat (`resultat.valide`, `resultat.message`...).
3. Remplacer simplement l'`import` dans app.js suffirait-il ? Qu'est-ce qui casserait ?

Choisis ensuite l'endroit où tu préfères agir et explique pourquoi :
- (a) rendre promo.js tolérant ;
- (b) brancher promotions.js ;
- (c) nettoyer la saisie dans app.js.

Pense aussi à ce qui est enregistré dans `etat.codePromo`.

# Ma réponse

1. appliquerPromo dans promo.js a saisie et montant alors que promotions.js a code, montant et liste.

2. Je crois que resultat.valide sert a vérifier si le code est approuvé par une fonction entérieure et que resultat.message sert a adapter le message selon si le code est bon (Ce code promo n'existe pas./Code NOEL10 appliqué.)

3. Remplacer promo.js pour promotions.js casserait l'affichage du message d'erreur et de validation et l'affichage de la réduction

je pense que (b) brancher promotions.js est pertinent, je pense qu'il faut changer le nom de la fonction appliquerPromo dans promotions.js par autre chose 

# Réponse:

Compris. Je t'expliquerai le code directement et je garde les questions pour la démarche de diagnostic. J'ai noté cette préférence dans ma mémoire du projet.

## Ce que fait le code
- **promo.js** est le fichier utilisé par app.js. Il compare la saisie au code **exactement**, caractère par caractère. `Noel10` ou `NOEL10 ` (avec un espace) ne correspondent donc à rien.
- **promotions.js** est une version 2 qui nettoie la saisie avant de comparer : elle retire les espaces autour et met tout en majuscules. Mais **aucun fichier ne l'importe**, donc elle ne sert à rien aujourd'hui.
- Les deux versions ne sont pas interchangeables :
  - promo.js renvoie un objet `{ valide, remise, message }`, et app.js s'en sert pour afficher les messages.
  - promotions.js renvoie seulement un nombre (la remise, ou `0`). Elle ne distingue plus « code inexistant » de « minimum d'achat non atteint ».
  - promotions.js attend aussi la liste des promos en paramètre, alors que promo.js la garde en mémoire.
- app.js enregistre la saisie **brute** de la cliente dans `etat.codePromo`, puis la sauvegarde dans le stockage du navigateur.

## Ta décision de diagnostic
Tu as deux grandes options :

| | (a) Ajouter le nettoyage dans promo.js | (b) Brancher promotions.js |
|---|---|---|
| Fichiers touchés | 1, environ une ligne | app.js en plusieurs endroits |
| Messages d'erreur | conservés | à reconstruire, sinon on perd le message du minimum d'achat |
| Risque de casser autre chose | faible | plus élevé |
| Code mort | promotions.js reste inutilisé (à signaler ou supprimer) | promo.js devient inutilisé |

À toi de trancher et de noter ta justification dans diagnostic.md.
Bon choix pour un ticket de maintenance. Je te laisse faire la modification. Voici les points d'attention et de quoi vérifier ton travail.

## Pendant la modification
- Fais le nettoyage **avant** la comparaison avec `promo.code`. Tu peux t'inspirer de la ligne de nettoyage qui existe déjà dans promotions.js.
- appliquerPromo est appelée à **deux endroits** dans app.js :
  - à la soumission du formulaire ;
  - au recalcul du panier, avec le code sauvegardé.
  
  Si tu corriges dans promo.js, les deux cas en profitent automatiquement.

## Les tests à refaire après
| Saisie | Attendu |
|---|---|
| `NOEL10` | appliqué (non-régression) |
| `noel10`, `Noel10` | appliqué |
| `NOEL10 ` et ` NOEL10` | appliqué |
| `N0EL10` (avec un zéro) | refusé : c'est un autre code |
| `BIENVENUE5` avec un panier de moins de 30 € | message du minimum toujours affiché |

Recharge aussi la page après avoir appliqué `noel10` : la remise doit être toujours là.

Regarde enfin le message de confirmation. Doit-il afficher ce que la cliente a tapé (`noel10`), ou le vrai code (`NOEL10`) ?

## Pour clôturer le ticket
Note dans diagnostic.md :
- la cause ;
- la correction choisie et pourquoi ;
- le résultat des tests ;
- ce que tu fais de promotions.js. Il devient du code mort et doublon : le supprimer, ou le signaler sans y toucher ?

Préviens-moi quand c'est fait. Je pourrai relire ta modification si tu veux.
## Pour la partie « utiliser une IA »
Ce qui a fait avancer ce ticket, c'est la méthode :
1. reproduire le bug sans téléphone ;
2. chercher le message d'erreur exact dans le code ;
3. remonter les `import` pour savoir quel code tourne vraiment ;
4. comparer les options de correction selon ce qu'elles risquent de casser.

Tu peux réutiliser cette méthode comme modèle de prompt pour les prochains tickets. Par exemple : « Voici le symptôme et le message affiché. Quel fichier produit ce message ? Quel code est réellement exécuté ? Quelles corrections sont possibles et que risque chacune ? »

# Ma réponse:

je choisis (a), je modifie promo.js

# Réponse:

Bon choix pour un ticket de maintenance. Je te laisse faire la modification. Voici les points d'attention et de quoi vérifier ton travail.

## Pendant la modification
- Fais le nettoyage **avant** la comparaison avec `promo.code`. Tu peux t'inspirer de la ligne de nettoyage qui existe déjà dans promotions.js.
- appliquerPromo est appelée à **deux endroits** dans app.js :
  - à la soumission du formulaire ;
  - au recalcul du panier, avec le code sauvegardé.
  
  Si tu corriges dans promo.js, les deux cas en profitent automatiquement.

## Les tests à refaire après
| Saisie | Attendu |
|---|---|
| `NOEL10` | appliqué (non-régression) |
| `noel10`, `Noel10` | appliqué |
| `NOEL10 ` et ` NOEL10` | appliqué |
| `N0EL10` (avec un zéro) | refusé : c'est un autre code |
| `BIENVENUE5` avec un panier de moins de 30 € | message du minimum toujours affiché |

Recharge aussi la page après avoir appliqué `noel10` : la remise doit être toujours là.

Regarde enfin le message de confirmation. Doit-il afficher ce que la cliente a tapé (`noel10`), ou le vrai code (`NOEL10`) ?

## Pour clôturer le ticket
Note dans diagnostic.md :
- la cause ;
- la correction choisie et pourquoi ;
- le résultat des tests ;
- ce que tu fais de promotions.js. Il devient du code mort et doublon : le supprimer, ou le signaler sans y toucher ?

Préviens-moi quand c'est fait. Je pourrai relire ta modification si tu veux.

# Ma Réponse:

je ne sais pas faire

# Réponse:

Pas de souci. Le diagnostic, c'est toi qui l'as fait. Il ne reste qu'à l'écrire en JavaScript, alors je te guide pas à pas et tu tapes toi-même.

## Où modifier
Ouvre [promo.js:14-15](boutique-roussel/js/promo.js#L14-L15). Tu y trouves :

```js
export function appliquerPromo(saisie, montant) {
  const regle = promos.find((promo) => promo.code === saisie);
```

La ligne 15 cherche dans la liste le code **exactement égal** à `saisie`. C'est là qu'il faut nettoyer la saisie avant de comparer.

## Quoi écrire
Remplace `saisie` par une version nettoyée à cet endroit. En JavaScript, on enchaîne deux opérations sur un texte :
- `.trim()` retire les espaces au début et à la fin ;
- `.toUpperCase()` met tout en majuscules.

Ça donne `saisie.trim().toUpperCase()`. C'est exactement ce que fait déjà promotions.js à sa ligne 7.

Mets ce morceau à la place du `saisie` qui suit `===` sur la ligne 15, puis enregistre le fichier.

## Ensuite
1. Recharge la page dans le navigateur avec `Ctrl+F5` pour être sûr d'avoir le nouveau fichier.
2. Refais les tests du tableau de mon message précédent.

Si un test échoue, ou si le site ne s'affiche plus, colle-moi la ligne que tu as écrite et on regarde ensemble.

# Ma réponse:

les affichages ne marchent plus

# Réponse

J'ai regardé ta modification. Tu étais au bon endroit, c'est une **faute de frappe**, pas une erreur de raisonnement.

## Comment la trouver toi-même
1. Ouvre la page, puis la console du navigateur avec `F12` et l'onglet **Console**.
2. Recharge la page. Tu devrais voir une erreur en rouge qui mentionne `toUppercase` et indique le fichier et la ligne.

C'est le réflexe à garder : **quand l'affichage casse après une modification, la console dit presque toujours pourquoi.**

## L'indice
Compare lettre par lettre ce que tu as écrit avec la ligne 7 de [promotions.js](boutique-roussel/js/promotions.js#L7). En JavaScript, les majuscules et minuscules comptent dans les noms de fonctions. Si une lettre ne correspond pas, la fonction n'existe pas.

## Pourquoi tout l'affichage casse
app.js appelle appliquerPromo à chaque affichage du panier, même sans code saisi. Une erreur dans cette fonction bloque donc tout le reste. Pense à le noter dans diagnostic.md : une petite modification peut avoir un gros impact.

Corrige la lettre, recharge avec `Ctrl+F5`, puis refais les tests.

# Ma réponse:

ça marche, tous les test passent

# Réponse

Bravo, le ticket 1 est résolu. J'ai vérifié ta ligne dans promo.js et elle est correcte.

## Une correction de ma part
Je t'ai dit plus tôt qu'app.js appelle appliquerPromo à chaque affichage, même sans code saisi. **C'était faux.** Dans [app.js:99](boutique-roussel/js/app.js#L99), la fonction n'est appelée que si un code est enregistré.

Pourquoi l'affichage cassait quand même, alors ? Un code de tes tests précédents était sauvegardé dans le navigateur. À chaque rechargement de la page, l'appli réessayait de l'appliquer et tombait sur la faute de frappe.

C'est une bonne leçon sur l'usage d'une IA : **vérifie ses affirmations dans le code**. Elle peut se tromper avec beaucoup d'assurance.

## Pour clôturer dans diagnostic.md
- **Symptôme** : le code NOEL10 était refusé sur téléphone.
- **Cause** : promo.js comparait la saisie exactement. Le clavier mobile ajoutait une majuscule automatique ou un espace.
- **Correction** : option (a). La saisie est nettoyée dans promo.js, ce qui couvre une seule ligne et garde les messages.
- **Tests** : ta liste, avec le résultat de chaque cas.
- **Incident pendant la correction** : la faute de frappe sur `toUpperCase` a cassé l'affichage, et la console l'a montrée.
- **Reste à traiter** : promotions.js est du code mort. Supprime-le ou signale-le, selon ce que tu décides.

Il reste aussi une question facultative. Le message affiche `Code NOEL10 appliqué` avec le vrai code, mais `etat.codePromo` sauvegarde toujours la saisie brute, par exemple `noel10 `. Ça fonctionne maintenant, parce que la saisie est nettoyée à chaque appel. Tu peux le noter comme remarque, sans forcément le corriger.

Quand tu es prêt, envoie-moi le ticket 2.

# Ma réponse:

synthetise le ticket 1 dans ticket-N.md en changeant le nom en ticket-1.md et en reprennant la structure a l'intérieur

# Réponse:

Écrit pour : la personne qui va relire le ticket (formateur ou équipe), donc factuel et sans le dialogue pédagogique.

J'ai renommé ticket-N.md en [ticket-1.md](boutique-roussel/tickets/ticket-1.md) avec `git mv` et j'ai gardé ses 4 sections :

- **SYMPTOME** : ce que la cliente a vécu.
- **ETAPES** : comment reproduire le bug sans téléphone, avec les variantes qui échouent.
- **ATTENDU** : le comportement voulu, et le tableau des tests avec leur résultat après correction.
- **CODE_CONCERNE** : les trois fichiers impliqués, la correction retenue et sa justification, la faute de frappe qui a cassé l'affichage pendant la correction, et ce qui reste à traiter (promotions.js et la saisie brute sauvegardée).

Le modèle n'avait pas de section pour la correction, alors je l'ai mise dans CODE_CONCERNE plutôt que d'ajouter une section.

Comme le fichier a été renommé, il n'y a plus de modèle vide pour le ticket 2. Je peux le recréer quand tu en auras besoin. Je n'ai rien commité.