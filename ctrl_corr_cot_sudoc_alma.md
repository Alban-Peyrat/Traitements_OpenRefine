# Contrôle des correspondances des cotes Sudoc-Alma (ex-CoCo-SAlma)

Le but de ce traitement est de contrôler la correspondance des cotes pour un PPN entre les données d'exemplaire du Sudoc et les holdings d'Alma.
Il signale également la présence de plusieurs cotes pour un même titre.

## Interpréter les résultats

### Colonne `Correspondance ?`

Deux résultats différents sont possibles, accompagnés ou non de précisions :
* `OUI` : toutes les cotes sont strictement égales entre les deux bases de données,
* `NON` : tous les cas non couverts par `OUI`.

En plus de ces deux résultats, des précisions peuvent être apportées :
* `Plusieurs cotes Sudoc` : plusieurs cotes ont été détectées dans le Sudoc,
* `Plusieurs cotes Alma` : plusieurs cotes ont été détectées dans Alma,
* `Exemplaire(s) sans cote Sudoc` : une ou plusieurs localisations dans le Sudoc n'ont pas de `$a`,
* `Exemplaire(s) sans cote Alma` : une ou plusieurs holdings dans Alma ont été détectées comme appartenant à la bibliothèque mais aucune cote n'y était renseignée,
* `Aucune localisation Sudoc` : _au 09/06/2022, le script doit être revu pour gérer les cas où aucune cote n'est détectée pour le RCR sous la notice et le cas où la connexion à l'API a échoué_,
* `Aucune holding Alma` : aucune holding n'a été trouvée pour la bibliothèque dans Alma.

### Colonnes des cotes

Dans les deux colonnes des cotes, plusieurs valeurs anormales peuvent apparaître :
* `[Ex. sans cote]` : pour le Sudoc, signifie qu'une `930` avec le RCR a bien été trouvée mais qu'elle ne possédait pas de `$a`, pour Alma, cela signifie qu'une holding pour la bibliothèque a bien été trouvée, mais qu'elle ne possédait pas de `;` (séparateur présent avant la cote),
* `[Pas d'holding]` : spécifique à Alma, indique qu'aucune holding n'a été trouvée pour la bibliothèque,
* `[Pas de loc.]` :  _au 09/06/2022, le script doit être revu pour gérer les cas où aucune cote n'est détectée pour le RCR sous la notice et le cas où la connexion à l'API a échoué_,
* aucune valeur affichée : spécifique à Alma (supposément), signifie que le script n'a pas été en mesure d'identifier une `(` après le `;`, ce qui arrive notamment si le champ `Numérotation A` des exemplaires de la holding est renseigné.

Comme la méthode utilisée pour récupérer les cotes d'Alma n'est pas la plus précise possible, il peut arriver que celles-ci soient incorrectes, pour diverses raisons.

### Colonne `PPN`

Si le PPN n'a pas pu clairement être identifié, sa valeur deviendra `XXXXXXXXX`, et la requête ne sera pas envoyé à l'API de l'Abes.

## Procédure

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

* Créez une colonne `Cotes Alma` basée sur la colonne `Disponibilité` en utilisant l'expression `Python / Jython` suivante (__pensez à changer la valeur de `lib` si nécessaire (nom dans Alma)__) :

``` Python
lib = "BU SVS - Josy Reiffers"
cote = []
for hold in value.split("\n"):
    if lib in hold:
        if ";" in hold:
            ind = hold.find(";")
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

* Créez une colonne `Cotes Sudoc` basée sur la colonne `Réponse Sudoc MARCXML` en utilisant l'expression `Python / Jython` suivante (__pensez à changer la valeur de `rcr` si nécessaire__) :

``` Python
rcr = "330632101"

if value is None:
    return "[Erreur requête]"

cote = []
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
if element.find("controlfield[@tag='001']") == None:
    return "[PPN incorrect]" #HTTP 404 is returned supposedly, so this doesn't happen
for loc in element.findall(".//datafield[@tag='930']"):
    if loc.find("subfield[@code='b']").text == rcr:
        # Checks if there's a $a
        if loc.find("subfield[@code='a']") != None:
            cote.append(loc.find("subfield[@code='a']").text)
        else:
            cote.append("[Ex. sans cote]")
if len(cote) > 0:
    cote.sort()
else:
    cote.append("[Pas de loc.]")
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
