# Traitements_OpenRefine

Procédures pour différents traitements dans Open Refine

## Liste des traitements
* [Nombre de bibliothèques localisées dans le Sudoc (ex-Projet P.A.)](./nb_bib_loc_sudoc.md)
* [Contrôle des correspondances des cotes Sudoc-Alma (ex-CoCo-SAlma)](./ctrl_corr_cot_sudoc_alma.md)

## Communs des traitements

Liste d'informations communes à plusieurs / tous les traitements

### Isolement et formatage du PPN pour le Sudoc

Le code ci-dessous permet d'isoler et formater les PPN selon cette logique :
* Recherche `(PPN)` dans la cellule, s'il le trouve, prend les 9 prochains caractères,
* Sinon, recherche `PPN ` _(suivi d'un espace)_ dans la cellule, s'il le trouve, prend les 9 prochains caractères,
* Sinon, recherche `PPN` _(sans espace après)_ dans la cellule, s'il le trouve, prend les 9 prochains caractères,
* Sinon, vérifie s'il y a 9 caractères dans la cellule après avoir retirer les espaces de début et de fin __et__ s'il y a 9 caractères dans la cellule après avoir supprimer tous carctères qui n'est ni un chiffre, ni `X` : si les deux conditions sont remplis, prend la valeur de la cellule en retirant les espaces de début et de fin,
* Sinon, vérifie s'il y a moins de 9 caractères dans la cellule après avoir retirer les espaces de début et de fin __et__ s'il y a moins 9 caractères dans la cellule après avoir supprimer tous carctères qui n'est ni un chiffre, ni `X` __et__ si les longueurs des deux vérifications précédentes sont égales : si les trois conditions sont remplis, prend la valeur de la cellule en retirant les espaces de début et de fin et rajoute devant autant de 0 que nécessaire pour atteindre les 9 caractères.

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

### Isolement de la cote depuis un export `Titres physiques` d'Alma

Le code ci-dessous permet d'isoler les cotes associées à une bibliothèque pour un titre donné dans un export `Titres physiques` d'Alma.
La variable `lib` doit contenir le nom affiché dans les exports Alma pour la bibliothèque voulue.
Si plusieurs cotes sont détectées, elles seront par défaut séparées par des `;`.

La détection fonctionne selon la logique suivante :
* Divise la cellule (de la colonne `Disponibilité`) en utilisant comme séparateur le retour à la ligne,
* Pour chaque holding résultant de la division, vérifie si le nom de la bibliothèque s'y trouve : s'il n'y est pas, passe à la holding suivante,
* S'il s'y trouve, vérifie s'il y a un `;` (séparateur se trouvant avant la cote),
* Si un `;` est trouvé, la valeur de la cote pour cette holding est tout ce qui se trouve entre le premier `;` et la première `(` situé après le premier `;`, à laquelle sont retirés les espaces de début et de fin,
* Si aucun `;` n'est trouvé, renvoie `[Ex. sans cote]`,
* Une fois toutes les holdings traitées, vérifie le nombre de holdings associées à la bibliothèque qui ont été identifiées :
  * S'il n'y a en aucune, renvoie comme valeur finale `[Pas d'holding]`,
  * S'il y a en a au moins une, renvoie toutes les cotes triées par ordre croissant.

Quelques points de vigilance :
* Si la localisation contient un `;`, la cote renvoyée peut être erronée,
* Si la cote contient une `(`, la cote renvoyée peut être erronée,
* Si le champ `Numérotation A` d'au moins 1 exemplaire de la holding est complété, la détection échouera puisque aucune `(` ne se trouvera après le `;`,
* Il y a probablement d'autres cas susceptibles de provoquer une mauvaise détection de la cote, mais la majorité des cas sont pris en compte par ce script.


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

### Rappels pour les requêtes

*Throttle speed* définit l'intervalle entre les requêtes.
__[Il n'est pas recommandé de le définir à moins de 1000 ms](https://docs.openrefine.org/manual/columnediting#add-column-by-fetching-urls).__

### Rappels pour retraiter sur Excel ensuite

Pour remplacer par un retour à la ligne dans Excel, entrez le caractère correspondant à `Alt+010`.
