
Avec MarcEdit, faire un export tabulé des champs voulus avec le délimiteur au sein d'un champ : `###|###`

Column → Edit cells → Split multi-valued cells... : by separator `###|###`

## Keep only unique values

* Sort column by text
* Reorder rows permanently
* Column → Edit cells → Blank down
* Column → Facet → Customized facets → Facet by blank
* Select `true`
* All → Edit rows → Remove matching rows

```JSON
// Repalce MY_COLUMN by the column name
[
  {
    "op": "core/multivalued-cell-split",
    "columnName": "MY_COLUMN",
    "keyColumnName": "MY_COLUMN",
    "mode": "separator",
    "separator": "###|###",
    "regex": false,
    "description": "Split multi-valued cells in MY_COLUMN"
  },
  {
    "op": "core/row-reorder",
    "mode": "row-based",
    "sorting": {
      "criteria": [
        {
          "valueType": "string",
          "column": "MY_COLUMN",
          "blankPosition": 2,
          "errorPosition": 1,
          "reverse": false,
          "caseSensitive": false
        }
      ]
    },
    "description": "Reorder rows"
  },
  {
    "op": "core/blank-down",
    "engineConfig": {
      "facets": [],
      "mode": "row-based"
    },
    "columnName": "MY_COLUMN",
    "description": "Blank down cells in column MY_COLUMN"
  },
  {
    "op": "core/row-removal",
    "engineConfig": {
      "facets": [
        {
          "type": "list",
          "name": "MY_COLUMN",
          "expression": "isBlank(value)",
          "columnName": "MY_COLUMN",
          "invert": false,
          "omitBlank": false,
          "omitError": false,
          "selection": [
            {
              "v": {
                "v": true,
                "l": "true"
              }
            }
          ],
          "selectBlank": false,
          "selectError": false
        }
      ],
      "mode": "row-based"
    },
    "description": "Remove rows"
  }
]
```