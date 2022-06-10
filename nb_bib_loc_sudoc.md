# Nombre de bibliothèques localisées dans le Sudoc (ex-Projet P.A.)

Le but de ce traitement est de pouvoir déterminer pour une liste de PPN donnée le nombre de bibliothèques également localisées dans le Sudoc.

## Procédure

### Création de la liste

Exportez d'Alma des __Titres physiques__ ou créer une liste de PPN.
Créez ensuite un fichier contenant uniquement la colonne `Numéro de notice` (et la colonne `Disponibilité` si vous souhaitez exporter les cotes).

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

* _Optionnel :_ Créez une colonne `Cotes` basée sur la colonne `Disponibilité` en utilisant l'expression `Python / Jython` suivante (__pensez à changer la valeur de `lib` si nécessaire (nom dans Alma)__) :

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

* Créez une colonne `Réponse Multiwhere` basée sur la colonne `PPN` utilisant des URLs en utilisant l'expression `GREL` suivante :

``` GREL
if(value != "XXXXXXXXX", "https://www.sudoc.fr/services/multiwhere/"+value+".xml", null)
```

* Créez une colonne `Nb de bib. localisée` basée sur la colonne `Réponse Multiwhere` en utilisant l'expression `Python / Jython` suivante (__pensez à changer la valeur de `rcr` et de `seuil` si nécessaire__) :

``` Python
rcr = "330632101"
seuil = 5

if value is None:
    return "NON;0;VÉRIFIER"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
for query in element.findall(".//query"):
    # On regarde si une localisation existe pour le PPN
    # Original by Alexandre Faure (louxfaure) in Sudoc/services/alma_to_sudoc.py
    for library in query.findall(".//library"):
        if rcr == library.find("rcr").text:
            is_located = "OUI"
            break
        is_located = "NON"
    if is_located :
        nbBib = len(query.findall(".//library"))-1
    else :
        nbBib = len(query.findall(".//library"))
    if nbBib > seuil:
        return str(is_located) + ";" + str(nbBib) + ";" + "DÉSHERBER"
    else:
        return str(is_located) + ";" + str(nbBib) + ";" + "CONSERVER"
```

* Divisez la colonne `Nb de bib. localisée` en utilisant `;` comme séparateur, ce qui donnera les colonnes :
  * `Notre bib. est localisée`
  * `Nb de bib. localisées`
  * `À conserver`
* Réorganisez les colonnes (`Réponse Multiwhere` et `Disponibilité` à supprimer, réorganiser l'ordre des colonnes, etc.).
* Exportez.
