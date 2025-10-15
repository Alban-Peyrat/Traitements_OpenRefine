# Liste de formules utiles

Liste de formules qui peiuvent être utlisées soit pour ajouter de nouvelles colonnes, soit tranformer une colonne déjà existantes, soit créer une facette personnalisée.
Sauf mention explicite du contraire, elles fonctionnent dans les trois contexte (mais ne sont pas forcément pertinentes dans les trois).

Pour une liste de manipulations ne requierant pas nécessairement de formules mais d'intéragir avec l'interface, voir [le document dédié](./manipulations_utiles.md).

## Table des matières

* [Récupérer les données d'un autre projet](#récupérer-les-données-dun-autre-projet)
* [Décoder les URLs encodées](#décoder-les-urls-encodées)
* [Encoder les URLs](#encoder-les-urls)

## Récupérer les données d'un autre projet

Dans le projet qui **recevra les données**, créer une colonne basée sur la colonne qui servira à faire le lien avec l'expression *GREL* suivante :

``` GREL
cell.cross("PROJET", "ID").cells["column"].value[0]
```

Où :

* `PROJET` doit prendre le nom du projet duquel l'on souhaite récupérer les données
* `ID` correspond au nom de la colonne dans le `PROJET` qui sert à faire le lien avec le projet actuel
* `column` correspond au nom de la colonne que l'on souhaite importer
* Note : `[0]` sert à récupérer la valeur du premier match :
  * Pour récupérer le nombre de matchs, remplacer `[0]` par `.length()` : `cell.cross("PROJET", "ID").cells["column"].value.length()`
  * Pour récupérer tous les matchs, supprimer `[0]` : `cell.cross("PROJET", "ID").cells["column"].value`
  * Pour récupérer tous les matchs en choisissant un séparateur, remplacer `[0]` par `.join("SEPARATEUR")` (indiquer le séparateur à la place de `SEPARATEUR`) : `cell.cross("PROJET", "ID").cells["column"].value.join("|")`

### Décoder les URLs encodées

Utiliser l'expression _Python / Jython_ suivante :

``` Python
import urllib
return urllib.unquote(value).decode("utf-8")
```

Exemple : `MEM%202000%20MARQU%C3%88S` devient `MEM 2000 MARQUÈS`

### Encoder les URLs

Utiliser l'expression _Python / Jython_ suivante :

``` Python
import urllib
return urllib.quote(value.encode("utf-8"))
```

Exemple : `MEM 2000 MARQUÈS` devient `MEM%202000%20MARQU%C3%88S`