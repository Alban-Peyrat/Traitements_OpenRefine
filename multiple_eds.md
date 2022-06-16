# Détection de multiples éditions d'un même titre (ex-CA3)

Le but de ce traitement est de déterminer pour une liste de PPN donnée si d'autres éditions du titre sont présentes dans la bibliothèque.
Il serait plus exact de l'appeler _Détection de multiples expressions d'une même oeuvre_ car c'est ce qui se passe réellement.

## Interpréter les résultats

Ci-dessous, la signification de chaque colonne :
* `Numéro de notice` : la liste originelle servant à générer la colonne des PPNs normalisés,
* `PPN` : le PPN normalisé à partir de la colonne précédente.
Prend la valeur `XXXXXXXXX` si la détection du PPN a échouée [(cliquez pour plus de détails sur cette procédure)](../../#isolement-et-formatage-du-ppn-pour-le-sudoc),
* `579 ?` : la valeur du PPN présent en `579` (= PPN de la notice de l'oeuvre) dans la notice SUDOC du titre.
Prend la valeur `XXXXXXXXX` s'il n'y a pas de `579`.
* `PPNs liés` : l'ensemble des PPNs de notices bibliographiques liées à la notice d'oeuvre (inclus le PPN servant à obtenir celui de la notice d'oeuvre), séparés par `,`,
* `Résultats` : la liste des PPNs autres que celui de la colonne `PPN` qui possèdent au moins une holding (`AVA`) dans Alma pour la bibliothèque renseignée.
Chaque cote est suivie du nombre d'exemplaires disponibles et du nombres d'exemplaires total.
Les PPNs sont séparés par des `;`, les différentes holdings par des ` -- `.

## Procédure

### Open Refine

* Créez un nouveau projet qui ne contient qu'une seule colonne `Numéro de notice` contenant liste de PPN originelle.
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

* Créez une colonne `Sudoc MARCXML` basée sur la colonne `PPN` utilisant des URLs en utilisant l'expression `GREL` suivante :

``` GREL
if(value != "XXXXXXXXX", "https://www.sudoc.fr/"+value+".xml", null)
```

* Créez une colonne `579 ?` basée sur la colonne `Sudoc MARCXML` en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

u579 = element.find("./datafield[@tag='579']")
if u579 == None:
  return "XXXXXXXXX"
else:
  return u579.find("subfield[@code='3']").text
```

* Créez une colonne `IdRef Biblio` basée sur la colonne `579 ?` utilisant des URLs en utilisant l'expression `GREL` suivante :

``` GREL
if(value != "XXXXXXXXX", "https://www.idref.fr/services/biblio/"+value, null)
```

* Créez une colonne `PPNs liés` basée sur la colonne `IdRef Biblio` en utilisant l'expression `Python / Jython` suivante :

``` Python
from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

docs = element.findall(".//doc")
ppns = []
for doc in docs:
  ppns.append(doc.find("ppn").text)
return ",".join(ppns)
```

* Créez une colonne `SRU Alma` basée sur la colonne `PPNs liés` utilisant des URLs en utilisant l'expression `GREL` suivante (__pensez à remplacer `{SRU (UNM default)}` par la valeur associée dans le fichier `VALEURS_UB.txt`__) :

``` GREL
if(value != "", "{SRU (UNM default)}" + "alma.other_system_number=" + value.split(",").join("%20or%20alma.other_system_number="), null)
```

* Créez une colonne `Résultats` basée sur la colonne `SRU Alma` en utilisant l'expression `Python / Jython` suivante (__pensez à changer la valeur de `bibID` (identifiant de la bibliothèque dans Alma) si nécessaire__) :

``` Python
bibID = "1302100000"
ns = {'sru': 'http://www.loc.gov/zing/srw/',
        'marc': 'http://www.loc.gov/MARC21/slim',
        'unm': 'info:srw/schema/8/unimarcxml-v0.1'}

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))

records = element.findall(".//unm:record", ns)
result = []
for rec in records:
  u035s = rec.findall("unm:datafield[@tag='035']", ns)
  ppn = None
  for u035 in u035s:
    if "(PPN)" in u035.find("unm:subfield[@code='a']", ns).text:
      ppn = u035.find("unm:subfield[@code='a']", ns).text.replace("(PPN)", "")
  if ppn == None:
    ppn = "[MMS ID] : " + rec.find("unm:controlfield[@tag='001']", ns).text
  if ppn == cells["PPN"].value:
    continue

  AVAs = rec.findall(".//unm:datafield[@tag='AVA']", ns)
  cotes = []
  for AVA in AVAs:
    if bibID == AVA.find("unm:subfield[@code='b']", ns).text:
      cote = AVA.find("unm:subfield[@code='d']", ns).text
      nbTotal = AVA.find("unm:subfield[@code='f']", ns).text
      nbUnav = AVA.find("unm:subfield[@code='g']", ns).text
      cotes.append("{} ({} / {})".format(cote, int(nbTotal)-int(nbUnav), nbTotal))
  if cotes != []:
    result.append(str(ppn) + " : " + " -- ".join(cotes))
return ";".join(result)
```

* Réorganisez les colonnes (`Sudoc MARCXML`, `IdRef Biblio`, et `SRU Alma` à supprimer, réorganiser l'ordre des colonnes, etc.).
* Exportez.
