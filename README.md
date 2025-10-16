# Traitements_OpenRefine

[![Active Development](https://img.shields.io/badge/Maintenance%20Level-Actively%20Developed-brightgreen.svg)](https://gist.github.com/cheerfulstoic/d107229326a01ff0f333a1d3476e068d)

## Structure du dépôt

* Traitements spécifiques :
  * [Aux exports tabulés provenant de MarcEdit](./MarcEdit_exports/README.md) :
    * [Diviser le leader par partie](./MarcEdit_exports/README.md#diviser-le-leader-par-partie)
    * [Diviser le 100$a par partie](./MarcEdit_exports/README.md#diviser-le-100a-par-partie)
  * [Aux exports CSV d'Omeka-S](./Omeka/README.md) :
    * [1. Identifier les contenus qui cassent l'export CSV](./Omeka/README.md#1-identifier-les-contenus-qui-cassent-lexport-csv)
    * [2. Identifier les droits d'accès des contenus & médias](./Omeka/README.md#2-identifier-les-droits-daccès-des-contenus--médias)
  * [Données du Sudoc](./Sudoc/README.md) :
    * [Normaliser un PPN](./Sudoc/README.md#normaliser-un-ppn)
    * [Récupérer la notice XML via le webservice de l'Abes](./Sudoc/README.md#récupérer-la-notice-xml-via-le-webservice-de-labes)
    * [XML : extraire le label (leader)](./Sudoc/README.md#xml--extraire-le-label-leader)
    * [XML : extraire un controlfield](./Sudoc/README.md#xml--extraire-un-controlfield)
    * [XML : extraire un sous-champ (filtre possible par ILN, RCR ou EPN)](./Sudoc/README.md#xml--extraire-un-sous-champ-filtre-possible-par-iln-rcr-ou-epn)
    * [XML : extraire un champ complet au format WinIBW (filtre possible par ILN, RCR ou EPN)](./Sudoc/README.md#xml--extraire-un-champ-complet-au-format-winibw-filtre-possible-par-iln-rcr-ou-epn)
    * [XML : compter le nombre de sous-champ (filtre possible par ILN, RCR ou EPN)](./Sudoc/README.md#xml--compter-le-nombre-de-sous-champs-filtre-possible-par-iln-rcr-ou-epn)
* [Manipulations utiles](./manipulations_utiles.md) :
  * [Dupliquer des lignes pour séparer de multiples valeurs au sein d'une même cellule](./manipulations_utiles.md#dupliquer-des-lignes-pour-séparer-de-multiples-valeurs-au-sein-dune-même-cellule)
  * [Fusionner des lignes possédant la même clef](./manipulations_utiles.md#fusionner-des-lignes-possédant-la-même-clef)
* [Formules utiles](./formules_utiles.md) :
  * [Récupérer les données d'un autre projet](./formules_utiles.md#récupérer-les-données-dun-autre-projet)
  * [Décoder les URLs encodées](./formules_utiles.md#décoder-les-urls-encodées)
  * [Encoder les URLs](./formules_utiles.md#encoder-les-urls)
* [Expressions régulières utiles](./expressions_regulieres_utiles.md) :
  * [PPN valides](./expressions_regulieres_utiles.md#ppn-valides)
  * [Date au format YYYY-MM-DD](./expressions_regulieres_utiles.md#date-au-format-yyyy-mm-dd)
  