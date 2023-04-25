# Analyser les procblèmes de passerelle Sudoc-Koha

## Récupérer les notices du Sudoc

* Créer la colonne _PPN_ contenant la liste des PPN à analyser
* Créez une colonne _Sudoc MARCXML_ basée sur la colonne _PPN_ utilisant des URLs en utilisant l'expression `GREL` suivante :

``` GREL
if(value != "XXXXXXXXX", "https://www.sudoc.fr/"+value+".xml", null)
```

# Analyse des problèmes de type de documents

* Créez une colonne _Leader pos. 6_ basée sur la colonne _Sudoc MARCXML_ en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

leader = element.find("./leader")
return leader.text[6:7]
```

* Créez une colonne _Leader pos. 7_ basée sur la colonne _Sudoc MARCXML_ en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

leader = element.find("./leader")
return leader.text[7:8]
```

* Ajouter une _Text facet_ pour les colonnes _Leader pos. 6_ et _Leader pos. 7_
* Ajouter un _Text filter_ pour la colonne _PPN_, en paramétrant sur _Expression régulière_ en mode _Invert_ (pour pouvoir exclure les PPN traités en les séparant par des `|`)

## Cas des leader pos. 6-7 = `as`

### 110$a pos. 1

* Créez une colonne _110$a pos. 1_ basée sur la colonne _Sudoc MARCXML_ en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

# If no 110, returns empty string
if element.find("datafield[@tag='110']") == None:
    return ""

return element.find("datafield[@tag='110']/subfield[@code='a']").text[:1]
```

* Ajouter une _Text facet_ pour les colonnes _110$a pos. 1_

### Nombre de 011

* Créez une colonne _Nb 011_ basée sur la colonne _Sudoc MARCXML_ en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

return len(element.findall("datafield[@tag='011']"))
```

* Ajouter une _Text facet_ pour la colonne _Nb 011_

### Nombre de 207

* Créez une colonne _Nb 207_ basée sur la colonne _Sudoc MARCXML_ en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

return len(element.findall("datafield[@tag='207']"))
```

* Ajouter une _Text facet_ pour la colonne _Nb 207_