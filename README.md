# EX-Liste

La EX-Liste est un système de décompte automatisé de l'activité aux arènes éligibles aux raids EX pour un serveur Discord. Le décompte de points est réalisé d'après une théorie plus ou moins vérifiée mais communément admise qui prévoit qu'un certain total (on parle de 250) doit être atteint à une arène pour qu'un raid EX y soit programmé lors de la prochaine vague de distribution de passes de raid EX.

La EX-Liste repose sur la *database* offerte par le bot YAGPDB
([https://yagpdb.xyz](https://yagpdb.xyz) ; [https://docs.yagpdb.xyz/reference/templates#database](https://docs.yagpdb.xyz/reference/templates#database)). YAGPDB offre une database pouvant contenir jusqu'à `50 * n` entrées, où `n` est le nombre de membres du serveur. La EX-Liste a besoin d'une entrée par arène pour compter son *EX-Score*, et une entrée par joueur utilisant la EX-Liste pour compter sa *EX-Gloire*.

La EX-Liste est constituée de huit commandes personnalisées (*custom command*) enregistrées dans le panneau de configuration de YAGPDB.

> Les commandes évoquées ci-après doivent être précédées d'une mention du bot (`‌@YAGPDB.xyz `) ou du préfixe défini dans le panneau de configuration de YAGPDB (par exemple `%`).

## Installation

Après avoir importé chacune des huit commandes, il faut adapter les tags *hard-codés* dans trois d'entre elles :
- `pointer`
- `tags`
- `EX-corriger-score`

Les tags sont les abréviations du nom des arènes pour lesquelles on veut suivre le EX-Score. Ils sont utilisés comme *clé* (préfixée par `"gym"`) sous laquelle est enregistré le EX-Score dans la database. Les tags n'ont pas de longueur minimum ou maximum requise mais doivent-être en majuscules.

La commande `tags` est la plus simple à adapter : il s'agit d'un simple texte donnant la liste des abréviations et les arènes auxquelles elles correspondent.

Pour adapter les commandes `pointer` et `EX-corriger-score`, il faut localiser l'endroit où la commande contrôle que l'argument fourni correspond bien à un tag possible :

```
...
{{ if or (eq $gym "<tag>") }}
...
```

`$gym` est une variable interne à la commande. `<tag>` est une des valeurs admises. Il faut répéter la fonction `(eq $gym "<tag>")` à l'intérieur de `{{ if or ... }}` autant de fois que nécessaire. Par exemple :

```{{ if or (eq $gym "ABC") (eq $gym "DEF") (eq $gym "XYZ") }}```

*Répétez l'opération **pour les deux commandes** `pointer` et `EX-corriger-score` !*

## Utilisation

Pour compter son activité à une arène éligible aux raids EX et en augmenter le EX-Score, un joueur utilise la commande suivante :

```pointer <n> <tag>```

`pointer` est le nom de la commande. `<n>` est le niveau du raid auquel le joueur a participé. `<tag>` est l'abréviation du nom de l'arène de laquelle on souhaite augmenter le EX-Score.

Le niveau du raid doit être un entier compris entre 1 et 5. Si l'argument renseigné n'est pas en entier, YAGPDB enverra un message d'erreur : `'<n>' is not a whole number`. Si l'argument est en entier mais inférieur à 1 ou supérieur à 5, YAGPDB enverra un autre message d'erreur, en français.

Le `<tag>` de l'arène doit être parmi une liste prédéfinie. Son entrée (*input*) est insensible à la casse mais sera immédiatement convertie en majuscules. En cas d'erreur, YAGPDB indiquera ne pas avoir reconnu le tag et proposera d'en afficher la liste avec la commande `tags`.

## Réponses et redirection

Les messages d'erreur sont envoyés là où la commande a été utilisée. Les confirmations (quand tout se passe bien) sont envoyées dans un canal en particulier dont on peut fournir l'identifiant dans la commande `pointer` :

```
...
{{ sendMessage <identifiant> $response }}
...
```

*Pour ne pas rediriger les confirmations, remplacez `<identifiant>` par `nil`.*

Les confirmations rappellent le nombre de points ajoutés à l'arène, son nouvel EX-Score et la nouvelle EX-Gloire de l'utilisateur.

## EX-Score

Si les arguments fournis par l'utilisateur avec la commande `pointer` sont corrects, le EX-Score de l'arène est incrémenté d'un certain nombre de points en fonction du niveau du raid :
- niveaux 1 et 2 : 1 point
- niveaux 3 et 4 : 3 points
- niveau 5 : 5 points

## EX-Gloire

Si les arguments fournis par l'utilisateur avec la commande `pointer` sont corrects, sa EX-Gloire est incrémentée du résultat de deux lancers de dé. Si le résultat est 7 (1 chance sur 6), la commande `pointer` enverra un message de félicitations supplémentaires :

```
:sunglasses: <user> a utilisé à merveille la commande `@YAGPDB.xyz pointer <n> <tag>. *Very smart!*
```

L'objectif est d'afficher de temps en temps un exemple de la commande pour aider les autres utilisateurs.

## Les classements

YAGPBD affichera le classement des arènes par ordre décroissant de EX-Score en réponse à la commande `EX-Liste`.
De même pour le classement des utilisateurs par EX-Gloire avec la commande `EX-Gloire`.

*Ces commandes ne renvoient que les 100 premiers éléments, au maximum.*

## Corrections

Deux commandes permettent de corriger une entrée de la EX-Liste : `%EX-corriger-score <tag> <corr>` et `%EX-corriger-gloire <@user> <corr>`.

`<@user>` est la mention d'un utilisateur. `<corr>` est un entier non-nul à appliquer au EX-score d'une à la EX-Gloire d'un utilisateur (positif ou négatif).

*Veillez à en réserver l'usage aux administrateurs, par exemple.*

## Réinitialisations

Deux commandes permettent de réinitialiser les classements : `%EX-reset-scores` et `%EX-reset-gloires`. Elles opèrent par tranches de huit enregistrements. Huit enregistrements de EX-Score ou de EX-Gloire sont récupérés (selon la commande) avec `dbGetPattern` et supprimés un à un avec `dbDel`. Un nouvel usage de `dbGetPattern` permet de savoir combien d'enregistrements sont encore présents dans la database (dans la limite de 100). Répétez ces commandes jusqu'à ce qu'elles indiquent ne plus y avoir d'enregistrement dans la database.

*Veillez à en réserver l'usage aux administrateurs, par exemple.*

Opérer par tranches de huit enregistrements permet de rester dans la limite de 10 actions sur la database par *custom command* pour les serveurs non-premium. En revanche utiliser deux fois `dbGetPattern` nécessite un abonnement premium, sans quoi il faudra les adapter pour n'utiliser qu'une fois cette fonction.
