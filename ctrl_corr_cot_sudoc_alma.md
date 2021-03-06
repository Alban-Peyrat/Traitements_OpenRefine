# Contrôle des correspondances des cotes Sudoc-Alma (ex-CoCo-SAlma)

Le but de ce traitement est de contrôler la correspondance des cotes pour un PPN entre les données d'exemplaire du Sudoc et les holdings d'Alma.
Il signale également la présence de plusieurs cotes pour un même titre.

## Interpréter les résultats

### Colonne `Correspondance ?`

Deux résultats différents sont possibles, accompagnés ou non de précisions :
* `OUI` : toutes les cotes sont strictement égales entre les deux bases de données,
* `NON` : tous les cas non couverts par `OUI`.

En plus de ces deux résultats, des précisions peuvent être apportées :
* `Erreur requête Sudoc` : la requête du Sudoc a échoué, sans plus de détails pour le moment,
* `Erreur requête Alma` : la requête SRU d'Alma a échoué, sans plus de détails pour le moment,
* `Plusieurs cotes Sudoc` : plusieurs cotes ont été détectées dans le Sudoc,
* `Plusieurs cotes Alma` : plusieurs cotes ont été détectées dans Alma,
* `Exemplaire(s) sans cote Sudoc` : une ou plusieurs localisations dans le Sudoc n'ont pas de `$a`,
* `Exemplaire(s) sans cote Alma` : une ou plusieurs holdings dans Alma ont été détectées comme appartenant à la bibliothèque mais aucune cote n'y était renseignée,
* `Aucune localisation Sudoc` : aucune localisation dans le Sudoc pour le RCR,
* `Aucune holding Alma` : aucune holding n'a été trouvée pour la bibliothèque dans Alma,
* `Trop de résultats Alma` : plusieurs notices bibliographiques ont été renvoyées par la requête SRU dans Alma, la recherche de cote n'a pas eu lieu,
* `PPN inconnu dans Alma` : aucune notice bibliographique n'a été renvoyée par la requête SRU dans Alma,
* `Dysfonctionnement Alma` : n'apparaît supposément jamais, indique qu'un nombre négatif de notices bibliographiques ont été renvoyées par la requête SRU dans Alma.

### Colonnes des cotes

Dans les deux colonnes des cotes, plusieurs valeurs anormales peuvent apparaître :
* `[Erreur requête]` : indique que la requête a échoué, sans plus de détails pour le moment,
* `[Ex. sans cote]` : pour le Sudoc, signifie qu'une `930` avec le RCR a bien été trouvée mais qu'elle ne possédait pas de `$a`, pour Alma, cela signifie qu'une holding pour la bibliothèque a bien été trouvée, mais qu'elle ne possédait pas de `;` (séparateur présent avant la cote),
* `[Pas d'holding]` : spécifique à Alma, indique qu'aucune holding n'a été trouvée pour la bibliothèque,
* `[PPN inconnu]` : spécifique au SRU d'Alma, indique qu'aucune notice bibliographique n'a été renvoyée par la requête,
* `[Trop de notices renvoyées]` : spécifique au SRU d'Alma, indique que plusieurs notices bibliographiques ont été renvoyées par la requête, la recherche de cote n'a pas eu lieu,
* `[Dysfonctionnement Alma]` : n'apparaît supposément jamais, spécifique au SRU d'Alma, indique qu'un nombre négatif de notices bibliographiques ont été renvoyées par la requête,
* `[Pas de loc.]` :  aucune localisation dans le Sudoc pour le RCR,
* aucune valeur affichée : spécifique à Alma sans utiliser le SRU (supposément), signifie que le script n'a pas été en mesure d'identifier une `(` après le `;`, ce qui arrive notamment si le champ `Numérotation A` des exemplaires de la holding est renseigné.

Comme la méthode utilisée pour récupérer les cotes d'Alma n'est pas la plus précise possible, il peut arriver que celles-ci soient incorrectes, pour diverses raisons.

### Colonne `PPN`

Si le PPN n'a pas pu clairement être identifié, sa valeur deviendra `XXXXXXXXX`, et la requête ne sera pas envoyé à l'API de l'Abes.

## Procédure

* Créez la liste :
  * Exportez d'Alma des __Titres physiques__.
Créez ensuite un fichier contenant uniquement les colonnes `Numéro de notice` et `Disponibilité` (si vous souhaitez récupérer les cotes depuis l'export Alma).
  * Ou créez une liste de PPNs d'une manière quelconque et utilisez le SRU d'Alma pour récupérer les cotes plus tard.

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

### Récupérer les cotes à partir d'un export Alma

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

### Récupérer les cotes à partir du SRU d'Alma

* Créez une colonne `SRU Alma` basée sur la colonne `PPN` utilisant des URLs en utilisant l'expression `GREL` suivante (__pensez à remplacer `{SRU (UNM default)}` par la valeur associée dans le fichier `VALEURS_UB.txt`__) :

``` GREL
if(value != null, "{SRU (UNM default)}" + "alma.other_system_number=(PPN)" + value, null)
```

* Créez une colonne `Cotes Alma` basée sur la colonne `SRU Alma` en utilisant l'expression `Python / Jython` suivante (__pensez à changer la valeur de `bibID` (identifiant de la bibliothèque dans Alma) si nécessaire__) :

``` Python
bibID = "1302100000"
ns = {'sru': 'http://www.loc.gov/zing/srw/',
        'marc': 'http://www.loc.gov/MARC21/slim',
        'unm': 'info:srw/schema/8/unimarcxml-v0.1'}
if value is None:
    return "[Erreur requête]"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

nb_res = int(element.find("sru:numberOfRecords",ns).text)
if nb_res > 1:
  return "[Trop de notices renvoyées]"
elif nb_res == 0:
  return "[PPN inconnu]"
elif nb_res < 0:
  return "[Dysfonctionnement Alma]"

# Get la cote
rec = element.find(".//unm:record", ns)
AVAs = rec.findall(".//unm:datafield[@tag='AVA']", ns)
cotes = []
for AVA in AVAs:
  if bibID == AVA.find("unm:subfield[@code='b']", ns).text:
    cotes.append(AVA.find("unm:subfield[@code='d']", ns).text)
if cotes != []:
  cotes.sort()
else:
  cotes.append("[Pas d'holding]")
return ";".join(cotes)
```

## Suite de la procédure

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
if "[Erreur requête]".encode("utf-8") in sudoc.encode("utf-8"):
    output.append("Erreur requête Sudoc")
if "[Erreur requête]".encode("utf-8") in alma.encode("utf-8"):
    output.append("Erreur requête Alma")
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
if "[Trop de notices renvoyées]".encode("utf-8") in alma.encode("utf-8"):
    output.append("Trop de résultats Alma")
if "[PPN inconnu]" in alma:
    output.append("PPN inconnu dans Alma")
if "[Dysfonctionnement Alma]" in alma:
    output.append("Dysfonctionnement Alma")
return ";".join(output)
```

* Réorganisez les colonnes (`Réponse Sudoc MARCXML`, `Disponibilité`, `SRU Alma`  à supprimer, réorganiser l'ordre des colonnes, etc.).
* Exportez.
