# Contrôle des correspondances des cotes Sudoc-Alma (ex-CoCo-SAlma)

Le but de cet outil est de contrôler la correspondance des cotes pour un PPN entre les données d'exemplaire du Sudoc et les holdings d'Alma.
Il signale également la présence de plusieurs cotes pour un même titre.

### Création de la liste

Exportez d'Alma des __Titres physiques__.
Créez ensuite un fichier contenant uniquement les colonnes `Numéro de notice` et `Disponibilité`.

__Notes sur les cotes :__
* Si la localisation contient dans son nom `;` ou que la cote contient `(`, la détection de la cote peut mal fonctionner,
* Si plusieurs cotes sont détectées, elles seront séparées par des points-virgules.

### Open Refine

* Ouvrez le fichier (ou importez les données dans un nouveau projet).
* Créez une colonne `PPN` basée sur la colonne `Numéro de notice` en utilisant l'expression `Python / Jython` suivante :

``` Python
ind = value.find("(PPN)")+5
if ind == 4:
     return "XXXXXXXXX"
else:
     return value[ind:ind+9]
```

* Créez une colonne `Cotes Alma` basée sur la colonne `Disponibilité` en utilisant l'expression `Python / Jython` suivante :

``` Python
lib = "BU SVS - Josy Reiffers"
cote = []
for hold in value.split("\n"):
    if lib in hold:
        if ";" in hold:
            ind = hold.find(";")
            # Attention, ne renvoie pas les cotes avec des parenthèses
            cote.append(hold[ind+1:hold[ind:].find("(")+ind].strip())
        else:
            cote.append("[Ex. sans cote]")
if len(cote) > 0:
    cote.sort()
else:
    cote.append("[Pas d'holding]")
return ";".join(cote)
```
* Créez une colonne `Réponse Sudoc MARCXML` basée sur la colonne `PPN` utilisant des URLs en utilisant l'expression `GREL` suivante :

``` GREL
if(value != "XXXXXXXXX", "https://www.sudoc.fr/"+value+".xml", null)
```

* Créez une colonne `Cotes Sudoc` basée sur la colonne `Réponse Sudoc MARCXML` en utilisant l'expression `Python / Jython` suivante :

``` Python
rcr = "330632101"

if value is None:
    return "[Pas de loc.]"

cote = []
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
for loc in element.findall(".//datafield[@tag='930']"):
    if loc.find("subfield[@code='b']").text == rcr:
        # Checks if there's a $a
        if loc.find("subfield[@code='a']") != None:
            cote.append(loc.find("subfield[@code='a']").text)
        else:
            cote.append("[Ex. sans cote]")
cote.sort()
return ";".join(cote)
```

* Créez une colonne `Correspondance ?` basée sur la colonne `Cotes Sudoc` en utilisant l'expression `Python / Jython` suivante (__attention, si vous avez appelé la colonne des cotes Alma d'une autre manière que `Cotes Alma`, changez la valeur de `nom_col_cotes_alma`__) :

``` Python
nom_col_cotes_alma = "Cotes Alma"

sudoc = value
alma = cells[nom_col_cotes_alma].value

output = []
if sudoc == alma:
    output.append("OUI")
else:
    output.append("NON")

# Analyse plus profonde des problèmes
if ";" in sudoc:
    output.append("Plusieurs cotes Sudoc")
if ";" in alma:
    output.append("Plusieurs cotes Alma")
if "[Ex. sans cote]" in sudoc:
    output.append("Exemplaire(s) sans cote Sudoc")
if "[Ex. sans cote]" in alma:
    output.append("Exemplaire(s) sans cote Alma")
if "[Pas de loc.]" in sudoc:
    output.append("Aucune localisation Sudoc")
if "[Pas d'holding]" in alma:
    output.append("Aucune holding Alma")

return ";".join(output)
```

* Réorganisez les colonnes (`Réponse Sudoc MARCXML` et `Disponibilité` à supprimer, réorganiser l'ordre des colonnes, etc.).
* Exportez.
