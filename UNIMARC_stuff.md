# 

## Diviser le leader par partie

_Edit column_ → _Split into several column..._ → _by field lengths_ : `5,1,4,1,1,5,3,4` + **do not guess cell type**

1. Record length (pos. 0-4)
1. Record status (pos. 5)
1. Implementation Codes (pos. 6-9)
1. Indicator length (pos. 10)
1. Subfield identifier length (pos. 11)
1. Base address of data (pos. 12-16)
1. Additional record definition (pos. 17-19)
1. Directory map (pos. 20-23)

## Diviser le 100$a par partie

_Edit column_ → _Split into several column..._ → _by field lengths_ : `8,1,4,4,3,1,1,3,1,4,4,2` + **do not guess cell type**

1. Date entered on file (pos. 0-7)
1. Type of date (pos. 8)
1. Date 1 (pos. 9-12)
1. Date 2 (pos. 13-16)
1. Target audience code (pos. 17-19)
1. Government publication code (pos. 20)
1. Modified record code (pos. 21)
1. Language of cataloguing (pos. 22-24)
1. Transliteration code (pos. 25)
1. Character set (pos. 26-29)
1. Additional character set (pos. 30-33)
1. Script of title (pos. 34-35)

Version avec les dates & character set fusionnées :  `8,1,8,3,1,1,3,1,8,2`

1. Date entered on file (pos. 0-7)
1. Type of date (pos. 8)
1. Dates (pos. 9-16)
1. Target audience code (pos. 17-19)
1. Government publication code (pos. 20)
1. Modified record code (pos. 21)
1. Language of cataloguing (pos. 22-24)
1. Transliteration code (pos. 25)
1. Character sets (pos. 26-33)
1. Script of title (pos. 34-35)

