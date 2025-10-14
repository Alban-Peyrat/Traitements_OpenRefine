# Manipuler des exports CSV provenant d'Omeka-S

Notes : 

* Tous les traitements décrits ici sont basés sur les exports CSV d'Omeka-S effectuées avec le module _[Export](https://github.com/biblibre/omeka-s-module-Export)_ version 1.7.0 de BibLibre, fourchée du [module de Laura Eiford](https://github.com/lauraleif/Export).
* Les exports JSON d'opérations & noms de colonnes sont basés sur l'instance Omeka-S d'ArchiRès, donc des divergences peuvent exister avec d'autres instances.

## Tables des matières

1. [Identifier les contenus qui cassent l'export CSV](#1-identifier-les-contenus-qui-cassent-lexport-csv)
1. [Identifier les droits d'accès des contenus & médias](#2-identifier-les-droits-daccès-des-contenus--médias)

## 1. Identifier les contenus qui cassent l'export CSV

JSON : [`json/items_breaking_CSV_export.json`](./json/items_breaking_CSV_export.json) (assume que _Colonne 212_ est celle contenant l'intégralité du contenu au format JSON)

Permet d'identifier les documents qui "cassent" la lecture des exports CSV sous certains logiciels, en attributant à chaque ligne la valeur `ERREUR` ou `OK`.

Créer une nouvelle colonne ou une facette personnalisée basée sur la colonne `Column 213` (contenu au format en JSON, première colonne sans nom) en utilisant l'expression _Python / Jython_ suivante :

``` Python
import json

try:
    val = json.loads(value)
except ValueError:
    return "ERREUR"
else:
    return "OK"
```

Il est également possible d'utiliser une expression _GREL_, mais elle est moins "stricte" sur la détection d'erreur :

``` GREL
if(isError(value.parseJson()), "ERREUR", "OK")
```

## 2. Identifier les droits d'accès des contenus & médias

JSON : [`json/extract_visibility.json`](./json/extract_visibility.json) (assume que _Colonne 212_ est celle contenant l'intégralité du contenu au format JSON)

Permet d'identifier les droits d'accès des contenus & médias, en soulignant les incohérences.
Basé sur une instance utilisant le module [Group (module for Omeka S)](https://github.com/Daniel-KM/Omeka-S-module-Group) de Daniel-KM.

### Interprétation des résultats

Colonne *summary* :


* `[WARNING]_HYBRID_MEDIAS_ILLEGAL_ITEM_VISIBILITY` : _valeur anormale_
* `[WARNING]_ILLEGAL_ITEM_VISIBILITY` : _valeur anormale_
* `[WARNING]_PRIVATE_ITEM_WITH_HYBRID_MEDIAS` : **le contenu est privé mais certains médias sont publics, d'autres sont privés**
* `[WARNING]_PRIVATE_ITEM_WITH_PUBLIC_MEDIAS` : **le contenu est privé mais tous ses médias sont publics**
* `[WARNING]_PRIVATE_MEDIAS_ILLEGAL_VISIBILITY` : _valeur anormale_
* `[WARNING]_PUBLIC_MEDIAS_ILLEGAL_ITEM_VISIBILITY` : _valeur anormale_
* `AUTHENTIFICATION` : tous les médias sont privés et des groupes leur sont attribués
* `AUTHENTIFICATION_-_MEDIALESS` : le contenu est privé avec des groupes attribués mais aucun média n'est rattaché à celui-ci
* `HYBRID` : le contenu est public et certains médias sont publics, d'autres sont privés
* `HYBRID_AUTH_PRIVATE` : tous les médias sont privés mais certains ont des groupes attribués, d'autres n'en n'ont pas
* `PRIVATE` : tous les médias sont privés mais n'ont aucun groupe attribué
* `PRIVATE_-_MEDIALESS` : le contenu est privé sans groupe attribué mais aucun média n'est rattaché à celui-ci
* `PUBLIC` : le contenu est public et tous ses médias le sont aussi
* `PUBLIC_-_MEDIALESS` : le contenu est public mais aucun média n'est rattaché à celui-ci

Colonne *item_public* :

* `ILLEGAL_VALUE` : _valeur anormale_
* `PRIVATE_ITEM` : le contenu est privé
* `PUBLIC_ITEM` : le contenu est public

Colonne *item_groups* :

* `ITEM_GROUPLESS` : aucun groupe n'est attribué au contenu
* `ITEM_HAVE_GROUPS` : des groupes sont attribués au contenu

Colonne *medias_public* :

* `HYBRID_MEDIAS_VISIBILITY` : certains médias sont publics, d'autres sont privés
* `NO_MEDIA` : aucun média n'est rattaché au contenu
* `PRIVATE_MEDIAS` : tous les médias sont privés
* `PUBLIC_MEDIAS` : tous les médias sont publics

Colonne *medias_groups* :

* `MEDIAS_GROUPLESS` : tous les médias n'ont aucun groupe attribué
* `MEDIAS_HAVE_GROUPS` : des groupes sont attribués à tous les médias
* `MEDIA_HYBRID_GROUP` : certains médias ont des groupes attribués, d'autres n'en n'ont pas
* `NO_MEDIA` : aucun média n'est rattaché au contenu

### Opérations à effectuer

* Créez une colonne `item_public` basée sur `o:is_public` avec l'expression *Python / Jython* suivante :

``` Python
if value == "1":
    return "PUBLIC_ITEM"
if value == None:
    return "PRIVATE_ITEM"
return "ILLEGAL_VALUE"
```

* Créez une colonne `item_groups` basée sur `o:id` avec l'expression *Python / Jython* suivante :

```Python
try:
    cells["o-module-group:group"]
except KeyError:
    return "ITEM_GROUPLESS"
else:
    return "ITEM_HAVE_GROUPS"
```

* Créez une colonne `medias_public` basée sur la colonne `o:id` en utilisant l'expression _Python / Jython_ suivante :

``` Python
import json

try:
    cells["media:full"]
except KeyError:
    return "NO_MEDIA"

medias = cells["media:full"].value[:-1]
medias = medias.split(";")
medias = "[" + ",".join(medias) + "]"
medias = json.loads(medias)

medias_public = False
medias_private = False
for media in medias:
    if media["o:is_public"]:
        medias_public = True
    else:
        medias_private = True

if medias_public and not medias_private:
    return "PUBLIC_MEDIAS"
elif not medias_public and medias_private:
    return "PRIVATE_MEDIAS"
else:
    return "HYBRID_MEDIAS_VISIBILITY"
```

* Créez une colonne `medias_groups` basée sur la colonne `o:id` en utilisant l'expression _Python / Jython_ suivante :

``` Python
import json

try:
    cells["media:full"]
except KeyError:
    return "NO_MEDIA"

medias = cells["media:full"].value[:-1]
medias = medias.split(";")
medias = "[" + ",".join(medias) + "]"
medias = json.loads(medias)

medias_have_groups = False
medias_dont_have_groups = False
for media in medias:
    if len(media["o-module-group:group"]) > 0:
        medias_have_groups = True
    else:
        medias_dont_have_groups = True

if medias_have_groups and not medias_dont_have_groups:
    return "MEDIAS_HAVE_GROUPS"
elif not medias_have_groups and medias_dont_have_groups:
    return "MEDIAS_GROUPLESS"
else:
    return "MEDIA_HYBRID_GROUP"
```

* Créez une colonne `summary` basée sur la colone `o:id` en utilisant l'expression *Python / Jython* suivante :

``` Python
if cells["medias_public"].value == "NO_MEDIA":
    if cells["item_public"].value == "PUBLIC_ITEM":
        return "PUBLIC_-_MEDIALESS"
    if cells["item_public"].value == "PRIVATE_ITEM" and cells["item_groups"].value == "ITEM_HAVE_GROUPS":
        return "AUTHENTIFICATION_-_MEDIALESS"
    if cells["item_public"].value == "PRIVATE_ITEM" and cells["item_groups"].value == "ITEM_GROUPLESS":
        return "PRIVATE_-_MEDIALESS"
    return "[WARNING]_ILLEGAL_ITEM_VISIBILITY"
if cells["medias_public"].value == "PUBLIC_MEDIAS":
    if cells["item_public"].value == "PUBLIC_ITEM":
        return "PUBLIC"
    if cells["item_public"].value == "PRIVATE_ITEM":
        return "[WARNING]_PRIVATE_ITEM_WITH_PUBLIC_MEDIAS"
    return "[WARNING]_PUBLIC_MEDIAS_ILLEGAL_ITEM_VISIBILITY"
if cells["medias_public"].value == "HYBRID_MEDIAS_VISIBILITY":
    if cells["item_public"].value == "PUBLIC_ITEM":
        return "HYBRID"
    if cells["item_public"].value == "PRIVATE_ITEM":
        return "[WARNING]_PRIVATE_ITEM_WITH_HYBRID_MEDIAS"
    return "[WARNING]_HYBRID_MEDIAS_ILLEGAL_ITEM_VISIBILITY"
if cells["medias_public"].value == "PRIVATE_MEDIAS":
    if cells["medias_groups"].value == "MEDIAS_HAVE_GROUPS":
        return "AUTHENTIFICATION"
    if cells["medias_groups"].value == "MEDIAS_GROUPLESS":
        return "PRIVATE"
    if cells["medias_groups"].value == "MEDIA_HYBRID_GROUP":
        return "HYBRID_AUTH_PRIVATE"
    return "[WARNING]_PRIVATE_MEDIAS_ILLEGAL_VISIBILITY"
```

* Réordoner les colonnes pour avoir :
    1. `o:id`
    1. `koha:biblionumber`
    1. `summary`
    1. `item_public`
    1. `item_groups`
    1. `medias_public`
    1. `medias_groups`