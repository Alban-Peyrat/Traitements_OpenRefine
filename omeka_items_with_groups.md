# Identifier les contenus avec des droits d'accès (#AR60)

Permet d'identifier à partir d'un export complet d'Omeka les documents ayant des droits d'accès erronés.

## Contrôles

Les informations contrôlées sont les suivantes

* les contenus ayant des groupes associés mais aucun groupe associé aux médias liés
* les contenus ayant des groupes associés et des groupes associés aux médias liés


# Open Refine

* Ouvrez le fichier (ou importez les données dans un nouveau projet).
* Créez une colonne `Group in both` basée sur la colonne `Numéro de notice` en utilisant l'expression `Python / Jython` suivante :

``` Python
import json

item_groups = value
medias = cells["media:full"].value[:-1]
medias = medias.split(";")
medias = "[" + ",".join(medias) + "]"
medias = json.loads(medias)

medias_have_groups = False
for media in medias:
    if len(media["o-module-group:group"]) > 0:
        medias_have_groups = True

if item_groups != None:
    return medias_have_groups
    # faut que je sépare les différents fichiers basés sur `;` : vérifiez que ya aps de pb posés


import re
reg = "(?![X])\D+"
ind = value.find("(PPN)")+5
if ind != 4:
    return value[ind:ind+9]
elif value.find("PPN ") != -1:
    return value[value.find("PPN ")+4:value.find("PPN ")+13]
elif value.find("PPN") != -1:
    return value[value.find("PPN")+3:value.find("PPN ")+12]
elif len(value.strip()) == 9 and len(re.sub(reg, "", str(value))) == 9:
    return value.strip()
elif len(value.strip()) < 9 and len(re.sub(reg, "", str(value))) < 9 and len(re.sub(reg, "", str(value))) == len(value.strip()):
    return "0" * (9-len(value.strip())) + value.strip()
else:
    return "XXXXXXXXX"
```