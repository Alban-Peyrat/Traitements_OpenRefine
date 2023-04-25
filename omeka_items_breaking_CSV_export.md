# Identifier les contenus qui cassent l'export CSV (#AR176)

Permet d'identifier à partir d'un export complet d'Omeka les documents qui "cassent" l'export CSV.

# Open Refine

* Ouvrez le fichier (ou importez les données dans un nouveau projet).
* Créez une colonne `Valid_JSON` basée sur la colonne `Column 213` (contenu en JSON) en utilisant l'expression `Python / Jython` suivante :

``` Python
import json

try:
    val = json.loads(value)
except ValueError:
    return "ERR"
else:
    return "OK"
```

* Appliquer une Text_facet à `Valid_JSON` pour filtrer uniquement les `ERR`