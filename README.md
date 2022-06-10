# Traitements_OpenRefine

Procédures pour différents traitements dans Open Refine

## Liste des traitements
* [Nombre de bibliothèques localisées dans le Sudoc (ex-Projet P.A.)](./nb_bib_loc_sudoc.md)
* [Contrôle des correspondances des cotes Sudoc-Alma (ex-CoCo-SAlma)](./ctrl_corr_cot_sudoc_alma.md)

## Communs des traitements

Liste d'informations communes à plusieurs / tous les traitements

### Isolement et formatage du PPN pour le Sudoc

Le code ci-dessous permet d'isoler et formater les PPN selon cette logique :
* Recherche `(PPN)` dans la cellule, s'il le trouve, prend les 9 prochains caractères
* Sinon, recherche `PPN ` _(suivi d'un espace)_ dans la cellule, s'il le trouve, prend les 9 prochains caractères
* Sinon, recherche `PPN` _(sans espace après)_ dans la cellule, s'il le trouve, prend les 9 prochains caractères
* Sinon, vérifie s'il y a 9 caractères dans la cellule après avoir retirer les espaces de début et de fin __et__ s'il y a 9 caractères dans la cellule après avoir supprimer tous carctères qui n'est ni un chiffre, ni `X` : si les deux conditions sont remplis, prend la valeur de la cellule en retirant les espaces de début et de fin
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

### Rappels pour les requêtes

*Throttle speed* définit l'intervalle entre les requêtes.
__[Il n'est pas recommandé de le définir à moins de 1000 ms](https://docs.openrefine.org/manual/columnediting#add-column-by-fetching-urls).__

### Rappels pour retraiter sur Excel ensuite

Pour remplacer par un retour à la ligne dans Excel, entrez le caractère correspondant à `Alt+010`.
