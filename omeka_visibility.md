# Identifier les contenus avec des droits d'accès (#AR60)

Permet d'identifier à partir d'un export complet d'Omeka les documents ayant des droits d'accès erronés.

# Open Refine

_Opérations disponibles dans [`json/omeka_visibility.json`](./json/omeka_visibility.json)_


* Ouvrez le fichier (ou importez les données dans un nouveau projet).
* Créez une colonne `public_item` basée sur `o:is_public` avec l'expression *Python / Jython* suivante :

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

* Créez une colonne `medias_public` basée sur la colonne `o:id` en utilisant l'expression `Python / Jython` suivante :

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

* Créez une colonne `medias_group` basée sur la colonne `o:id` en utilisant l'expression `Python / Jython` suivante :

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
    if cells["public_item"].value == "PUBLIC_ITEM":
        return "PUBLIC_MEDIALESS"
    if cells["public_item"].value == "PRIVATE_ITEM" and cells["item_groups"].value == "ITEM_HAVE_GROUPS":
        return "AUTHENTIFICATION_MEDIALESS"
    if cells["public_item"].value == "PRIVATE_ITEM" and cells["item_groups"].value == "ITEM_GROUPLESS":
        return "PRIVATE_MEDIALESS"
    return "ILLEGAL_ITEM_VISIBILITY"
if cells["medias_public"].value == "PUBLIC_MEDIAS":
    if cells["public_item"].value == "PUBLIC_ITEM":
        return "PUBLIC"
    if cells["public_item"].value == "PRIVATE_ITEM":
        return "PUBLIC_MEDIAS_ITEM_PRIVATE_MISCONFIGURED"
    return "PUBLIC_MEDIAS_ILLEGAL_ITEM_VISIBILITY"
if cells["medias_public"].value == "HYBRID_MEDIAS_VISIBILITY":
    if cells["public_item"].value == "PUBLIC_ITEM":
        return "HYBRID"
    if cells["public_item"].value == "PRIVATE_ITEM":
        return "HYBRID_MEDIAS_ITEM_PRIVATE_MISCONFIGURED"
    return "HYBRID_MEDIAS_ILLEGAL_ITEM_VISIBILITY"
if cells["medias_public"].value == "PRIVATE_MEDIAS":
    if cells["medias_group"].value == "MEDIAS_HAVE_GROUPS":
        return "AUTHENTIFICATION"
    if cells["medias_group"].value == "MEDIAS_GROUPLESS":
        return "PRIVATE"
    if cells["medias_group"].value == "MEDIA_HYBRID_GROUP":
        return "HYBRID_AUTH_PRIVATE"
    return "PRIVATE_MEDIAS_ILLEGAL_VISIBILITY"
```

* Réordoner les colonnes pour avoir :
    1. `o:id`
    1. `koha:biblionumber`
    1. `summary`
    1. `public_item`
    1. `item_groups`
    1. `medias_public`
    1. `medias_group`