# Manipuler des données du Sudoc

## Table des matières

* [Normaliser un PPN](#normaliser-un-ppn)
* [Récupérer la notice XML via le webservice de l'Abes](#récupérer-la-notice-xml-via-le-webservice-de-labes)
* [XML : extraire le label (leader)](#xml--extraire-le-label-leader)
* [XML : extraire un controlfield](#xml--extraire-un-controlfield)
* [XML : extraire un sous-champ (filtre possible par ILN, RCR ou EPN)](#xml--extraire-un-sous-champ-filtre-possible-par-iln-rcr-ou-epn)
* [XML : extraire un champ complet au format WinIBW (filtre possible par ILN, RCR ou EPN)](#xml--extraire-un-champ-complet-au-format-winibw-filtre-possible-par-iln-rcr-ou-epn)
* [XML : compter le nombre de sous-champ (filtre possible par ILN, RCR ou EPN)](#xml--compter-le-nombre-de-sous-champs-filtre-possible-par-iln-rcr-ou-epn)

## Normaliser un PPN

Supprime `PPN` & `(PPN)`, les espaces avant / après et rajoute les `0` devant s'ils sont manquants.
Renvoie `null` si vide.

Utiliser l'expression _GREL_ suivante :

``` GREL
with(value.replace(/(?:PPN|\(PPN\))/, "").strip(), v, if(isBlank(v), null, if(v.length < 9, v, "000000000".substring(v.length()) + v)))
```

## Récupérer la notice XML via le webservice de l'Abes

En se basant sur le colonne contenant le PPN, ajouter une colonne via URL avec une des expressions _GREL_ suivantes.
Remplacer `sudoc` par `idref` pour interroger IdRef plutôt que le Sudoc.

Si le PPN n'est pas vide :

``` GREL
if(isBlank(value), null, "https://www.sudoc.fr/"+value+".xml")
```

Si le PPN est valide :

``` GREL
if(isNull(value.match(/^\d{8}[\d|X]$/)), null, "https://www.sudoc.fr/"+value+".xml")
```

Normaliser le PPN et interroger seulement s'il est valide :

``` GREL
with(with(value.replace(/(?:PPN|\(PPN\))/, "").strip(), v, if(isBlank(v), null, if(v.length < 9, v, "000000000".substring(v.length()) + v))), ppn, if(isNull(ppn.match(/^\d{8}[\d|X]$/)), null, "https://www.sudoc.fr/"+ppn+".xml"))
```

## XML : extraire le label (leader)

Se base sur le contenu renvoyé par un fecth d'URL des notices XML.
Utiliser l'expression _Python / Jython_ ci-dessous.

Note : toute valeur retournée entre `[]` est une erreur dans l'extraction des données.

``` Python
if value is None:
    return "[Erreur requête]"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
if element.find("controlfield[@tag='001']") == None:
    return "[Problème requête, pas de 001]" #HTTP 404 is returned supposedly, so this doesn't happen

try:
    return element.find("leader").text
except:
    return "[Leader introuvable]"
```

## XML : extraire un controlfield

_Rappel : les controlfields sont les champs `001` à `009`_

Se base sur le contenu renvoyé par un fecth d'URL des notices XML.
Utiliser l'expression _Python / Jython_ ci-dessous en modifiant au début :

* `999` dans `tag = "999"` par le numéro du champ voulu

Notes :

* Toute valeur retournée entre `[]` est une erreur dans l'extraction des données
* Si le controlfield n'existe pas sur la notice, retourne `null`

``` Python
tag = "999"

if value is None:
    return "[Erreur requête]"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
if element.find("controlfield[@tag='001']") == None:
    return "[Problème requête, pas de 001]" #HTTP 404 is returned supposedly, so this doesn't happen

try:
    return element.find("controlfield[@tag='" + tag + "']").text
except:
    return None
```

## XML : extraire un sous-champ (filtre possible par ILN, RCR ou EPN)

Se base sur le contenu renvoyé par un fecth d'URL des notices XML.
Utiliser l'expression _Python / Jython_ ci-dessous en modifiant au début :

* `999` dans `tag = "999"` par le numéro du champ voulu
* `a` dans `code = "a"` par le code du sous-champ voulu
* `-1` dans `ILN = "-1"` par l'ILN voulu (laisser à `-1` pour ne pas filtrer par ILN)
* `-2` dans `RCR = "-2"` par le RCR voulu (laisser à `-2` pour ne pas filtrer par RCR)
* `-3` dans `EPN = "-3"` par l'EPN voulu (laisser à `-3` pour ne pas filtrer par EPN)
* `|` dans `sep = "|"` pour sélectionner le séparateur entre chaque champs si plsuieurs sont trouvées
* `;` dans `same_field_sep = ";"` pour sélectionner le séparateur entre chaque sous-champ au sein du même champ si plsuieurs sont trouvées

Notes :

* Toute valeur retournée entre `[]` est une erreur dans l'extraction des données
* Le filtre par ILN vérifie si le champ contient un `$1` strictement égal à l'ILN
* Le filtre par RCR vérifie si le champ contient un `$5` commençant par le RCR (vérifie les 9 premires caractères)
* Le filtre par EPN vérifie si le champ contient un `$5` strictement égal à l'EPN
* Si un des filtres est défini, en cas d'erreur lors de l'analyse des filtres, le champ est exclu par défaut
* Retourne `null` si aucun champ ne contient le sous-champ

``` Python
tag = "999"
code = "a"
ILN = "-1"
RCR = "-2"
EPN = "-3"
sep = "|"
same_field_sep = ";"

if value is None:
    return "[Erreur requête]"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
if element.find("controlfield[@tag='001']") == None:
    return "[Problème requête, pas de 001]" #HTTP 404 is returned supposedly, so this doesn't happen

if ILN == "-1":
    ILN = None
if RCR == "-2":
    RCR = None
if EPN == "-3":
    EPN = None

def filter_out(field, ILN, RCR, EPN):
    # If no filter is defined, return False by default
    # If at least one is defined, return True by default
    if ILN == None and RCR == None and EPN == None:
        return False
    try:
        if ILN != None:
            # ILN is defined but no $1 : exclude the field
            if type(field.find("subfield[@code='1']")) == type(None):
                return True
            # ILN does not match $1
            if field.find("subfield[@code='1']").text != ILN:
                return True

        if RCR != None:
            # RCR is defined but no $5 : exclude the field
            if type(field.find("subfield[@code='5']")) == type(None):
                return True
            # RCR does not match $5
            if field.find("subfield[@code='5']").text[:9] != RCR:
                return True

        if EPN != None:
            # EPN is defined but no $5 : exclude the field
            if type(field.find("subfield[@code='5']")) == type(None):
                return True
            # EPN does not match $5
            if field.find("subfield[@code='5']").text != EPN:
                return True
        # Avery active filter matched nicely
        return False
    except:
        # Default to true
        return True
    # Default to true
    return True

fields = []
for field in element.findall(".//datafield[@tag='" + tag + "']"):
    try:
        # Checks if this field should be filterred out
        if filter_out(field, ILN, RCR, EPN):
            continue
        # Checks if there's a $a
        subfs = []
        for subf in field.findall(".//subfield[@code='" + code + "']"):
            if subf.text != None:
                subfs.append(subf.text)
        if len(subfs) > 0:
            fields.append(same_field_sep.join(subfs))
    except:
        continue
if len(fields) == 0:
    return None
return sep.join(fields)
```

## XML : extraire un champ complet au format WinIBW (filtre possible par ILN, RCR ou EPN)

Se base sur le contenu renvoyé par un fecth d'URL des notices XML.
Compatible avec les datafields & les controlfields.
Utiliser l'expression _Python / Jython_ ci-dessous en modifiant au début :

* `999` dans `tag = "999"` par le numéro du champ voulu
* `-1` dans `ILN = "-1"` par l'ILN voulu (laisser à `-1` pour ne pas filtrer par ILN)
* `-2` dans `RCR = "-2"` par le RCR voulu (laisser à `-2` pour ne pas filtrer par RCR)
* `-3` dans `EPN = "-3"` par l'EPN voulu (laisser à `-3` pour ne pas filtrer par EPN)
* `\n` dans `sep = "\n"` pour sélectionner le séparateur entre chaque champs si plsuieurs sont trouvées

Notes :

* Toute valeur retournée entre `[]` est une erreur dans l'extraction des données
* Le filtre par ILN vérifie si le champ contient un `$1` strictement égal à l'ILN
* Le filtre par RCR vérifie si le champ contient un `$5` commençant par le RCR (vérifie les 9 premires caractères)
* Le filtre par EPN vérifie si le champ contient un `$5` strictement égal à l'EPN
* Si un des filtres est défini, en cas d'erreur lors de l'analyse des filtres, le champ est exclu par défaut
* Retourne `null` si aucun champ ne correspond aux filtres

``` Python
tag = "999"
ILN = "-1"
RCR = "-2"
EPN = "-3"
sep = "\n"

if value is None:
    return "[Erreur requête]"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
if element.find("controlfield[@tag='001']") == None:
    return "[Problème requête, pas de 001]" #HTTP 404 is returned supposedly, so this doesn't happen

if int(tag) < 10:
    try:
        return tag + " " + element.find("controlfield[@tag='" + tag + "']").text
    except:
        return None

if ILN == "-1":
    ILN = None
if RCR == "-2":
    RCR = None
if EPN == "-3":
    EPN = None

def filter_out(field, ILN, RCR, EPN):
    # If no filter is defined, return False by default
    # If at least one is defined, return True by default
    if ILN == None and RCR == None and EPN == None:
        return False
    try:
        if ILN != None:
            # ILN is defined but no $1 : exclude the field
            if type(field.find("subfield[@code='1']")) == type(None):
                return True
            # ILN does not match $1
            if field.find("subfield[@code='1']").text != ILN:
                return True

        if RCR != None:
            # RCR is defined but no $5 : exclude the field
            if type(field.find("subfield[@code='5']")) == type(None):
                return True
            # RCR does not match $5
            if field.find("subfield[@code='5']").text[:9] != RCR:
                return True

        if EPN != None:
            # EPN is defined but no $5 : exclude the field
            if type(field.find("subfield[@code='5']")) == type(None):
                return True
            # EPN does not match $5
            if field.find("subfield[@code='5']").text != EPN:
                return True
        # Avery active filter matched nicely
        return False
    except:
        # Default to true
        return True
    # Default to true
    return True

fields = []
for field in element.findall(".//datafield[@tag='" + tag + "']"):
    try:
        # Checks if this field should be filterred out
        if filter_out(field, ILN, RCR, EPN):
            continue
        output = tag + " " + field.get("ind1") + field.get("ind2")
        for subf in field.findall(".//subfield"):
            output += "$" + subf.get("code")
            if subf.text != None:
                output += subf.text
        fields.append(output)
    except:
        continue
if len(fields) == 0:
    return None
return sep.join(fields)
```

## XML : compter le nombre de sous-champs (filtre possible par ILN, RCR ou EPN)

Se base sur le contenu renvoyé par un fecth d'URL des notices XML.
Pour les controlfield, compte uniquement s'il est présent ou non.
Utiliser l'expression _Python / Jython_ ci-dessous en modifiant au début :

* `999` dans `tag = "999"` par le numéro du champ voulu
* `a` dans `code = "a"` par le code du sous-champ voulu
* `-1` dans `ILN = "-1"` par l'ILN voulu (laisser à `-1` pour ne pas filtrer par ILN)
* `-2` dans `RCR = "-2"` par le RCR voulu (laisser à `-2` pour ne pas filtrer par RCR)
* `-3` dans `EPN = "-3"` par l'EPN voulu (laisser à `-3` pour ne pas filtrer par EPN)
* `0` dans `sep = "0"` pour sélectionner le séparateur entre chaque champs si plusieurs sont trouvés
  * Si laissé sur `0`, la somme totale est calculée, sinon, chaque champ affiche sa valeur
  * Exemple : `sep = "0"` avec une `955` qui comporte 1 `$i` & une autre `955` comportant 2 `$i` retournera `3`
  * Exemple 2 : `sep = ";"` avec une `955` qui comporte 1 `$i` & une autre `955` comportant 2 `$i` retournera `1;2`

Notes :

* Toute valeur retournée entre `[]` est une erreur dans l'extraction des données
* Le filtre par ILN vérifie si le champ contient un `$1` strictement égal à l'ILN
* Le filtre par RCR vérifie si le champ contient un `$5` commençant par le RCR (vérifie les 9 premires caractères)
* Le filtre par EPN vérifie si le champ contient un `$5` strictement égal à l'EPN
* Si un des filtres est défini, en cas d'erreur lors de l'analyse des filtres, le champ est exclu par défaut

``` Python
tag = "999"
code = "a"
ILN = "-1"
RCR = "-2"
EPN = "-3"
sep = "0"

if value is None:
    return "[Erreur requête]"

from xml.etree import ElementTree as ET
element = ET.fromstring(value.encode("utf-8"))
if element.find("controlfield[@tag='001']") == None:
    return "[Problème requête, pas de 001]" #HTTP 404 is returned supposedly, so this doesn't happen

if int(tag) < 10:
    return len(element.findall(".//controlfield[@tag='" + tag + "']"))

if ILN == "-1":
    ILN = None
if RCR == "-2":
    RCR = None
if EPN == "-3":
    EPN = None

def filter_out(field, ILN, RCR, EPN):
    # If no filter is defined, return False by default
    # If at least one is defined, return True by default
    if ILN == None and RCR == None and EPN == None:
        return False
    try:
        if ILN != None:
            # ILN is defined but no $1 : exclude the field
            if type(field.find("subfield[@code='1']")) == type(None):
                return True
            # ILN does not match $1
            if field.find("subfield[@code='1']").text != ILN:
                return True

        if RCR != None:
            # RCR is defined but no $5 : exclude the field
            if type(field.find("subfield[@code='5']")) == type(None):
                return True
            # RCR does not match $5
            if field.find("subfield[@code='5']").text[:9] != RCR:
                return True

        if EPN != None:
            # EPN is defined but no $5 : exclude the field
            if type(field.find("subfield[@code='5']")) == type(None):
                return True
            # EPN does not match $5
            if field.find("subfield[@code='5']").text != EPN:
                return True
        # Avery active filter matched nicely
        return False
    except:
        # Default to true
        return True
    # Default to true
    return True

fields = []
for field in element.findall(".//datafield[@tag='" + tag + "']"):
    try:
        # Checks if this field should be filterred out
        if filter_out(field, ILN, RCR, EPN):
            continue
        # Count nb of subfields with this code
        fields.append(len(field.findall(".//subfield[@code='" + code + "']")))
    except:
        continue
if len(fields) == 0:
    return 0
if sep != "0":
    return sep.join([str(field) for field in fields])
return sum(fields)

```
