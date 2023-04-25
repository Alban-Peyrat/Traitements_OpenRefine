# 



# Open Refine

* Ouvrez le fichier (ou importez les données dans un nouveau projet).
* Renommez les colonnes
* Placez en première position l'information groupant les autres
* Placez en seconde position le nombre de différentes itérations pour cette information
* Créez une colonne `lists` basée sur la première colonne en utilisant l'expression `Python / Jython` suivante _(changez si nécessaire la valeur des variables )_:

``` Python
# -*- coding: utf-8 -*- 
# Configuration des variables
separator = "|"
nb_occ = "nb"

import json

# Extraction des colonnes et division de celles qui doivent l'être
cols = {}
for col in row.columnNames:
    if separator in str(cells[col].value):
        cols[col] = cells[col].value.split(separator)
    else:
        cols[col] = cells[col].value

# Crée la liste
output = []
for ii in range(int(cols[nb_occ])):
    borrower = {}
    for col in cols:
        if type(cols[col]) is list:
            borrower[col] = cols[col][ii]
    output.append(borrower)

return json.dumps(output)
```

* Créez une colonne `tests` basée sur la colonne `lists` en utilisant l'expression `Python / Jython` suivante :

``` Python
# -*- coding: utf-8 -*- 
# Configuration des variables
expirationdate = "2022-12-12"
restrictiondate = "2023-01-01"
restrictioncontent = "compte"

import json

val = json.loads(value)

output = []
for b in val:
    borrower = {}
    borrower["bor_nb"] = str(b["borrowernumbers"])
    borrower["card_nb"] = str(b["cardnumbers"])
    
    # Compte actif
    borrower["activeAccount"] = str(b["dateexpiry"] > expirationdate)

    # Suspension active
    borrower["activeSuspension"] =  str(b["restriction"] > restrictiondate)

    # Valeur de la suspension
    borrower["isMultipleAccount"] = str(restrictioncontent in str(b["noterestriction"]))

    output.append(borrower)
return json.dumps(output)
```

* Créez une colonne `Can delete` basée sur la colonne `tests` en utilisant l'expression `Python / Jython` suivante :

``` Python
# -*- coding: utf-8 -*- 
import json

val = json.loads(value)

output = []
for b in val:
    if (b["activeAccount"] == "True") + (b["activeSuspension"] == "True") + (b["isMultipleAccount"] == "True") == 0:
        output.append(b["card_nb"])

return "|".join(output)
```

* Créez une colonne `is double compte` basée sur la colonne `tests` en utilisant l'expression `Python / Jython` suivante :

``` Python
# -*- coding: utf-8 -*- 
import json

val = json.loads(value)

output = []
for b in val:
    if b["isMultipleAccount"] == "True":
        output.append(b["card_nb"])

return "|".join(output)
```