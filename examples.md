# Examples

## Reference to field data examples

Reference to *field data* of the *leader*.

    LDR

Reference to all *field data* of fields having a field tag starting with *00*.

    00.

Reference to all *field data* of fields having a field tag starting with *7*.

    7..

Reference to *data elements* of all repetitions of the "100" field.

    100

## Reference to field data with repetitions examples

Reference to the first "300" field.

    300[0]

Reference to the second of the "300" field.

    300[1]

Reference to the first, second and third of the "300" field.

    300[0-2]

Reference to all but the first of the "300" field.

    300[1-#]

Reference to the last of the "300" field.

    300[#]

Reference to the last two of the "300" field.

    300[#-1]

## Reference to substring examples

Reference to substring of *field data* in the *leader* from character position 0 to character position 4 (5 characters).

    LDR/0-4

Reference to data in the *leader* at character position 6 (1 character).

    LDR/6

Reference to data in the control field 007 at character position 0 (1 character).

    007/0

Reference to all data but the first character in the control field "007".

    007/1-#

Reference to the last character in the control field "007".

    007/#

Reference to the last two characters of the value of the subfield "a" of field "245".

    245$a/#-1

## Reference to data content examples

Reference to value of the subfield "a" of field "245".

    245$a

Reference to the value of the subfields "a", "b" and "c" of field "245".

    245$a$b$c

Same as above, but with the use of a *subfield code range*.

    245$a-c

Reference to the value of the subfields "_" and "$" of field "300".

    300$_$$

## Reference to data content with repetitions examples

Reference to value of the subfield "a" of the first "300" field.

    300[0]$a

Reference to the value of the first subfield "a" of the field "300"

    300$a[0]

Reference to the value of the last subfield "a" of the field "300"

    300$a[#]

Reference to the value of the last two repetitions of subfield "a" of the field "300"

    300$a[#-1]


## Reference to contextualized data with indicators examples

Reference to *data content* in the subfield "a" within the context of *indicator 1* with the value "1".

    245_1$a

or

    245_1_$a

Reference to the value of the subfield "a" within the context of *indicator 1* with the value "1" and *indicator 2* with the value "0".

    245_10$a

Reference to the value of the subfield "a" within the context of *indicator 2* with the value "0".

    245__0$a

Reference to value of the subfield "a" of the first three repetitions of field "307"  within the context of *indicator 1* with the value "8". This will NOT reference the first three 307 fields that are in the context of indicator 1.

    307[0-3]_8$a

## Reference to contextualized data with subSpecs examples

###Checking dependencies via string comparison

If Leader/06 = t: Books

Reference to character with position "18" of field "008", if character with position "06" in Leader equals "t".

    008/18{LDR/6=\t}

---

If Field 007/00 = a and t

Reference to subfield "b" of field 245, if character with position "0" of field 007 equals "a" OR "t".

    245$b{007/0=\a|007/0=\t}

---

If Leader/06 = a and Leader/07 = a, c, d, or m: Books

Reference to character with position "18" of field "008", if character with position "06" in Leader equals "a" AND character with position "07" in Leader equals "a", "c", "d" OR "m".

    008/18{LDR/6=\a}{LDR/7=\a|LDR/7=\c|LDR/7=\d|LDR/7=\m}

---

Example data:

100  1#$6880-01$aZilbershtain, Yitshak ben David Yosef.<br>
880  1#$6100-01/(2/r$a, יצחק יוסף בן דוד.

Reference data content of subfield "a" of field "880", if data content of subfield "6" of field "100" includes the string "-01" (characters with index range 3-5 of field "800") and the string "880".

    880$a{100_1$6~$6/3-5}{100_1$6~\880}

---

###Checking existence of fields

Reference data content of subfield "c" of field "020", if subfield "a" of field "020" exists.

    020$c{$a}

---

Reference data content of subfield "z" of field "020", if subfield "a" of field "020" does not exist.

    020$z{!$a}

---

###Abbreviation of fieldSpec or subfieldSpec

As of [MARCspec interpretation] a MARCspec without an explicitly given index is always an abbreviations of *n* references this example shows how these specs are interpreted.

Example Data:

    020 ##$a0394170660$qRandom House$c$4.95
    020 ##$a0491001304

Reference to data content of subfield "q" of field "020" if subfield "c" exists.

    020$q{$c}

same as

    020[0-#]$q[0-#]{$c[0-#]}

same as

    020[0]$q[0]{?020[0]$c[0]} OR // true
    020[1]$q[0]{?020[1]$c[0]} // false

---

Example Data:

    020 ##$a0394170660$qRandom House$qpaperback$c$4.95
    020 ##$a0394502884$qRandom House$qhardcover$c$12.50 

Reference to data content of subfield "c" if data content of one repetition of subfield "q" equals the comparison string "paperback".

    020$c{$q=\paperback}

same as

    020[0-#]$c[0-#]{$q[0-#]=\paperback}

same as 

    020[0]$c[0]{020[0]$q[0]=\paperback} OR // false
    020[0]$c[0]{020[0]$q[1]=\paperback} OR // true
    020[1]$c[0]{020[1]$q[0]=\paperback} OR // false
    020[1]$c[0]{020[1]$q[1]=\paperback}    // false

---

Reference to data of the first repetition of field "800",
 if data content of subfield "a" within the context of indicator 2 is "1"
 of the preceding fieldSpec includes the comparisonString "Poe".
 
    800[0]{800[0]__1$a~\Poe}

An abbreviated subTerm like ```__1$a``` in

    800[0]{__1$a~\Poe}

is __invalid__! An abbreviated subterm MUST only be one of fieldspec or subfieldspec.

---

Reference of data content of subfield "a" of field "245",
 if last character of the preceding spec equals the comparisonString "/".
 
    245$a{/#=\/}

same as

    245$a{245$a/#=\/}
