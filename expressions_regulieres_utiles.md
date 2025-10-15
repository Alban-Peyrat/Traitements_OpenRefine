# Liste d'expression régulières utiles

Liste d'expressions régulières qui peuvent être surtout utilisées pour filtrer les données via des filtres textuels.

[Regex101](https://regex101.com) est un exemple de site permettant de tester des expressions régulières.

## Table des matières

* [PPN valides](#ppn-valides)
* [Date au format YYYY-MM-DD](#date-au-format-yyyy-mm-dd)

## PPN valides

Uniquement le PPN, sans espaces avant / après :

``` Regexp
^\d{8}[\d|X]$
```

PPN avec possible espaces avant / après :

``` Regexp
^\s*\d{8}[\d|X]\s*$
```

PPN préfixé de `(PPN)` ou `PPN` avec possible espaces avant / après :

``` Regexp
^\s*(?:PPN|\(PPN\))?\s*\d{8}[\d|X]\s*$
```

## Date au format YYYY-MM-DD

Stricte, mais ne vérifie pas si le jour existe vraiment (ex : 31 février) :

``` Regexp
^\d{4}\-(0[1-9]|1[012])\-(0[1-9]|[12][0-9]|3[01])$
```

Simple :

``` Regexp
^\d{4}\-\d{2}\-\d{2}$
```