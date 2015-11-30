# Introduction

Since it became a common task to map [MARC] data to arbitrary formats, these mappings are normally based on a set of definitions of MARC fields and subfields called *MARC field specification* or short *MARC spec*.

There are already implementations of *MARC specs* in tools like [marcspec], [catmandu], [solrmarc], [easyM2R] etc.. Each of them using a different flavour of *MARC spec*. This document is an approach to normalize such field specifications.

The hereby described specification __MARCspec__ can help to build reusable MARCspec parsers and validators and facilitates the exchange of mapping definitions.

## Status of this document

The current version of this proposal is a preliminary draft for open discussion. [Feedback](https://github.com/cklee/marc-spec/issues) is welcome!

## Terminology

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

See also [Definition of MARC related terms used in this spec].

## What is a MARCspec?

Machine-Readable Cataloguing (MARC) is a document based key-value exchange format for bibliographic and other library related data. A MARC record consists of three main sections: the __leader__, the __directory__, and the __variable fields__ with the __data content__. There are two kinds of (variable) fields: (variable) __control fields__ and (variable) __data fields__. The term __fixed field__ stands for fields whose length does not vary like the leader and some of the *control fields*. The __field content__ in the the *fixed fields* can be accessed through its __character position or range__. Only *data fields* are divided into *subfields*. *Subfields* can also be contextualized through indicators. There is an indicator 1 and an indicator 2 for all data fields, both are optional.

A __MARCspec__ is a reference to field data of a MARC record and is very much like XPath for XML. With MARCspec one can reference data on different levels of a MARC record defined through the *fields*, *character positions*, *subfields* and *indicators*.

The *data* of the MARC record being referenced may be represented through a *set of data*, having zero or more *data elements*. MARCspec does neither define the form of this referenced *set of data*, nor the encoding of the referenced *data content*.  

# Limitations of MARCspec as string

A MARCspec might not fulfil all requirements of definition for a reference to the desired set of data like XPath does for XML. This is because of the nearly unlimited number of options accessing data in a MARC record, especially when it comes to delimiters based on cataloging rules. Thus a MARCspec has to concentrate on the basic references and let all other data processing to subsequent data processing functions of tools having implemented MARCspec.

To enable support for other [ISO 2709] applications MARCspecs syntax does not distinguish between types of fields like in the [MARC] record structure. A valid MARCspec might violate the MARC record structure. It is led to the MARCspec aware tools weather to check for MARC record structure violation or not. 

## Basic references

A MARCspec allows the following basic references:

- Reference to *record data* (except data from the record directory)
- Reference to *field data*
- Reference to *data content* of subfields
- Reference to substrings of *data content* of fixed fields and subfields

References a MARCspec does allow are:

- Reference to single *designators*, field tags or subfield codes
- Reference to a position index of a specific field, subfield or character
- Reference to data in related records
- Reference to *data elements* (entries) in the MARC record directory

## Form of MARCspec as string

__This section is normative.__

The __Augmented BNF for Syntax Specifications: ABNF__ [RFC 5234] is used to define the form of the MARCspec as string.

The whole ABNF for MARCspec shows as follows

    alphaupper        = %x41-5A
                        ; A-Z
    alphalower        = %x61-7A
                       ; a-z
    positiveDigit     = %x31-39
                        ;  "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9"
    positiveInteger   = "0" / positiveDigit [1*DIGIT]
    indicator         = alphalower / DIGIT
    indicator1        = indicator
    indicator2        = indicator
    indicators        = "_" (indicator1 / "_") [indicator2 / "_"]
    fieldTag          = 3(alphalower / DIGIT / ".") / 3(alphaupper / DIGIT / ".")
    position          = positiveInteger / "#"
    range             = position "-" position
    positionOrRange   = range / position
    characterSpec     = "/" positionOrRange
    subfieldChar      = %x21-3F / %x5B-7B / %x7D-7E
                        ; ! " # $ % & ' ( ) * + , - . / 0-9 : ; < = > ? [ \ ] ^ _ \` a-z { } ~
    subfieldCode      = "$" subfieldChar
    subfieldCodeRange = "$" ( (alphalower "-" alphalower) / (DIGIT "-" DIGIT) )
                        ; [a-z]-[a-z] / [0-9]-[0-9]
    index             = "[" positionOrRange "]"
    fixedField        = fieldTag [index] [characterSpec]
    variableField     = fieldTag [index] [indicators]
    fieldSpec         = fixedField / variableField
    subfieldSpec      = (subfieldCode / subfieldCodeRange) [index] [characterSpec]
    comparisonString  = "\" *VCHAR
    operator          = "=" / "!=" / "~" / "!~" / "!" / "?"
                        ; equal / unequal / includes / not includes / not exists / exists
    abrFieldSpec      = index [ (characterSpec / indicators) ] / characterSpec / indicators
    abrSubfieldSpec   = index [characterSpec] / characterSpec
    abbreviation      = abrFieldSpec / abrSubfieldSpec
    subTerm           = fieldSpec / subfieldSpec / comparisonString / abbreviation
    subTermSet        = [ [subTerm] operator ] subTerm
    subSpec           = "{" subTermSet *( "|" subTermSet ) "}"
    MARCspec          = (variableField *subSpec *(subfieldSpec *subSpec)) / fixedField *subSpec

### General form

Every __MARCspec__ consists of a *fixed field* spec or *variable field* spec. *Variable fields* followed optionally by one or more *subfieldSpecs*. Both *fieldSpec* and *subfieldSpec* can be contextualized through *subSpecs* (see section [SubSpecs]).

    fieldSpec = fixedField / variableField
    MARCspec  = (variableField *subSpec *(subfieldSpec *subSpec)) / fixedField *subSpec

### Reference to field data

A __fieldSpec__ is a reference to *field data* of a field. It consists of the three character *field tag*, followed optionally

- by an *index* (see section [Reference to repetitions]) and
- by an *characterSpec* (for *fixed fields*) (see section [Reference to substring]) or
- by *indicators* (for *variable fields*) (see section [Reference to data content]).

The __field tag__ may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both) or the character ```.```. The character ```.``` is interpreted as a wildcard. E.g. "3.." is then a reference to the *data elements* in all *fields* beginning with "3". 

The special *field tag* ```LDR``` is the *field tag* for the *leader*. 

    alphaupper    = %x41-5A ; A-Z
    alphalower    = %x61-7A; a-z
    fieldTag      = 3(alphalower / DIGIT / ".") / 3(alphaupper / DIGIT / ".")
    fixedField    = fieldTag [index] characterSpec
    variableField = fieldTag [index] indicators

<div class="note">
A *fieldSpec * without an explicitly given *index* is always a reference to all repetitions of the referenced field(s) (see [MARCspec interpretation] for implicit rules). One MUST also conclute that a *fieldSpec * without an explicitly given *index* is an abbreviation of a reference with the starting index ```0``` and the ending index ```#``` (see [Reference to repetitions] examples).
</div>
<div class="example">
Reference to *field data* of the *leader*.

    LDR

</div>
<div class="example">
Reference to all *field data* of fields having a field tag starting with *00*.

    00.

</div>
<div class="example">
Reference to all *field data* of fields having a field tag starting with *7*.

    7..

</div>
<div class="example">
Reference to *data elements* of all repetitions of the "100" field.

    100

</div>

### Reference to substring

A __characterSpec__ is a reference to a character or a range of characters within a *field* or *subfield*. It consists of a *position or range* prefixed with the character ```/```.

    characterSpec = "/" positionOrRange

A __position or range__ is either a *postion* or a *range*.

The __postion__ is either a *positive integer* or the character ```#``` as a symbol for the last character of the referenced *data content*. 

The __range__ consists of two *positions* concatenated with the character ```-```. 

    positiveDigit   = %x31-39
                        ;  "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9"
    positiveInteger = "0" / positiveDigit [1*DIGIT]
    position        = positiveInteger / "#"
    range           = position "-" position
    positionOrRange = range / position

<div class="note">
Interpretation of a *range* differs through the position of the special *position* character ```#``` as a symbol for the last character of the referenced *data content* (see [MARCspec interpretation] for implicit rules).
</div>

<div class="example">
Reference to substring of *field data* in the *leader* from character position 0 to character position 4 (5 characters).

    LDR/0-4

</div>

<div class="example">
Reference to data in the *leader* at character position 6 (1 character).

    LDR/6

</div>

<div class="example">
Reference to data in the control field 007 at character position 0 (1 character).

    007/0

</div>

<div class="example">
Reference to all data but the first character in the control field "007".

    007/1-#

</div>
<div class="example">
Reference to the last character in the control field "007".

    007/#

</div>

<div class="example">
Reference to the last two characters of the value of the subfield "a" of field "245".

    245$a/#-1
</div>

### Reference to data content

The __subfieldSpec__ is a reference to the *data content* (value) of a *subfield*. It either consists of a *subfieldCode* or a *subfieldCodeRange* followed optionally by an *index* and a *characterSpec*.

    subfieldSpec = (subfieldCode / subfieldCodeRange) [index] [characterSpec]

A __subfieldCode__ is a *subfieldChar* prefixed by the character ```$```.

A __subfieldCodeRange__ is prefixed by the character ```$``` and restricted to either two alphabetic or two numeric characters both concatenated with the character ```-```.

A __subfieldChar__ is a lowercase alphabetic, a numeric character or a special character.

    subfieldChar      = %x21-3F / %x5B-7B / %x7D-7E
    subfieldCode      = "$" subfieldChar
    subfieldCodeRange = "$" ( (%x61-7A "-" %x61-7A) / (%x30-39 "-" %x30-39) )

<div class="example">
Reference to value of the subfield "a" of field "245".

    245$a

</div>

<div class="example">
Reference to the value of the subfields "a", "b" and "c" of field "245".

    245$a$b$c

</div>

<div class="example">
Same as above, but with the use of a *subfield code range*.

    245$a-c

</div>

<div class="example">
Reference to the value of the subfields "_" and "$" of field "300".

    300$_$$
</div>

### Reference to repetitions

For repeatable *fields* and *subfields* each repetition can be referenced by its __index__. An index is a *position or range* enclosed with the characters ```[``` and ```]```. The first repetition of a *field* or  a *subfield* is always referenced with the index ```[0]```. The last repetition of a *field* or  a *subfield* is referenced with the index ```[#]```.

    index = "[" positionOrRange "]"

<div class="example">
Reference to the first "300" field.

    300[0]

</div>

<div class="example">
Reference to the second of the "300" field.

    300[1]

</div>

<div class="example">
Reference to the first, second and third of the "300" field.

    300[0-2]

</div>

<div class="example">
Reference to all but the first of the "300" field.

    300[1-#]

</div>

<div class="example">
Reference to the last of the "300" field.

    300[#]

</div>

<div class="example">
Reference to the last two of the "300" field.

    300[#-1]

</div>

<div class="example">
Reference to value of the subfield "a" of the first "300" field.

    300[0]$a

</div>

<div class="example">
Reference to the value of the first subfield "a" of the field "300"

    300$a[0]

</div>

<div class="example">
Reference to the value of the last subfield "a" of the field "300"

    300$a[#]

</div>

<div class="example">
Reference to the value of the last two repetitions of subfield "a" of the field "300"

    300$a[#-1]

</div>

### Reference to contextualized data

#### Indicators

__Indicators__ are prefixed by the character ```_```. There are two indicators: __indicator 1__ and __indicator 2__. Both are optional and either represented through a lowercase alphabetic or a numeric character. If *indicator 1* is not specified, it MUST be replaced by the character ```_```. If *indicator 2* is not specified it might be replaced by the character ```_``` or left blank.

    indicator  = alphalower / DIGIT
    indicator1 = indicator
    indicator2 = indicator
    indicators = "_" (indicator1 / "_") [indicator2 / "_"]


<div class="example">
Reference to *data content* in the subfield "a" within the context of *indicator 1* with the value "1".

    245_1$a

or

    245_1_$a

</div>

<div class="example">
Reference to the value of the subfield "a" within the context of *indicator 1* with the value "1" and *indicator 2* with the value "0".

    245_10$a

</div>

<div class="example">
Reference to the value of the subfield "a" within the context of *indicator 2* with the value "0".

    245__0$a

</div>

<div class="example">
Reference to value of the subfield "a" of the first three repetitions of field "307"  within the context of *indicator 1* with the value "8". This will NOT reference the first three 307 fields that are in the context of indicator 1.

    307[0-3]_8$a

</div>

#### SubSpecs

With a __subSpec__ the preceding *fieldSpec* or *subfieldSpec* gets contextualized. Every *subSpec* MUST be validated either __true__ or __false__. Is a *subSpec* *true*, the preceding spec is used to reference data. Is a subSpec *false*, the preceding spec doesn't get used to reference data.

A *subSpec* is enclosed with the characters ```{``` and ```}```. A *subSpec* consists of one or more sets of *subTerms* (the __left hand subTerm__ and the __right hand subTerm__) and an *operator*. This combination of *subTerms* and an *operator* can be *chained* through the character ```|``` (__OR__) within a *subSpec*. Multiple *subSpecs* can also be *repeated* one after another (__AND__).

    subTerm    = fieldSpec / subfieldSpec / comparisonString
    subTermSet = [ [subTerm] operator ] subTerm
    subSpec    = "{" subTermSet *( "|" subTermSet ) "}"

The __operator__ is one of

- ```=``` (as a symbol for "equal"), 
- ```!=``` (as a symbol for "unequal"),
- ```~``` (as a symbol for "includes"), 
- ```!~``` (as a symbol for "not includes")
- ```!``` (as a symbol for "not exists") or
- ```?``` (as a symbol for "exists").

    operator = "=" / "!=" / "~" / "!~" / "!" / "?"
 
A __subTerm__ is one of

- *fieldSpec*
- *subfieldSpec*
- *comparisonString*.

It is possible to __abbreviate__ a contextualized *fieldSpec* by only using

- *index* or
- *index* and *characterSpec*  or
- *index* and *indicators* or
- *characterSpec* or
- *indicators*

    abrFieldSpec = index [ (characterSpec / indicators) ] / characterSpec / indicators

as a *subTerm* (see [SubSpec abbreviation] and [Abbreviation of fieldSpec or subfieldSpec] for examples).

It is possible to __abbreviate__ a contextualized *subfieldSpec* by only using

- *index* or
- *index* and *characterSpec*  or
- *characterSpec*

    abrSubfieldSpec  = index [characterSpec] / characterSpec

as a *subTerm* (see [SubSpec abbreviation] and [Abbreviation of fieldSpec or subfieldSpec] for examples).

By omitting the *left hand subTerm*, this implicitly makes the preceding spec outside the subfieldSpec the *left hand subTerm* (see [MARCspec interpretation] for implicit rules). For *subSpecs* with omitted *left hand subTerm* the *operator* can also be omitted. Omitting the *operator* this implies the use of the *operator* ```?``` (exists).

A __comparisonString__ can be every combination of ASCII characters prefixed by the ```\``` character. For unambiguousness in a *comparisonString* the following characters MUST be escaped by the character ```\```:

- ```$```
- ```{```
- ```}```
- ```!```
- ```=```
- ```~```
- ```?```
- ```|```

In a *comparisonString* a whitespace MUST be encoded as the character combination ```\s```.

    comparisonString = "\" *VCHAR

<div class="example">
Checking dependencies via string comparison

If Leader/06 = t: Books

Reference to character with position "18" of field "008", if character with position "06" in Leader equals "t".

    008/18{LDR/6=\t}

</div>
<div class="example">
Checking dependencies via string comparison alternatives

If Field 007/00 = a and t

Reference to subfield "b" of field "245", if character with position "0" of field 007 equals "a" OR "t".

    245$b{007/0=\a|007/0=\t}

</div>
<div class="example">
Checking dependencies via string comparison chains

If Leader/06 = a and Leader/07 = a, c, d, or m: Books

Reference to character with position "18" of field "008", if character with position "06" in Leader equals "a" AND character with position "07" in Leader equals "a", "c", "d" OR "m".

    008/18{LDR/6=\a}{LDR/7=\a|LDR/7=\c|LDR/7=\d|LDR/7=\m}

</div>
<div class="example">
Checking dependencies via string comparison and content comparison
Example data:

100  1#$6880-01$aZilbershtain, Yitshak ben David Yosef.<br>
880  1#$6100-01/(2/r$a, יצחק יוסף בן דוד.

Reference data content of subfield "a" of field "880", if data content of subfield "6" of field "100" includes the string "-01" (characters with index range 3-5 of field "800") and the string "880".

    880$a{100_1$6~$6/3-5}{100_1$6~\880}

</div>
<div class="example">
Checking existence of fields

Reference data content of subfield "c" of field "020", if subfield "a" of field "020" exists.

    020$c{$a}

</div>
<div class="example">
Checking (non) existence of fields

Reference data content of subfield "z" of field "020", if subfield "a" of field "020" does not exist.

    020$z{!$a}

</div>

<div class="example">
Abbreviation of fieldSpec or subfieldSpec

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

</div>

<div class="example">
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

</div>

<div class="example">

Reference to data of the first repetition of field "800",
 if data content of subfield "a" within the context of *indicator 2* is "1"
 of the preceding fieldSpec includes the comparisonString "Poe".
 
    800[0]{800[0]__1$a~\Poe}

An abbreviated subTerm like ```__1$a``` in

    800[0]{__1$a~\Poe}

is __invalid__! An abbreviated subterm MUST only be one of fieldspec or subfieldspec.

</div>

<div class="example">

Reference of data content of subfield "a" of field "245",
 if last character of the preceding spec equals the comparisonString "/".
 
    245$a{/#=\/}

same as

    245$a{245$a/#=\/}

</div>

## MARCspec interpretation

[MARCspec interpretation]: #marcspec-interpretation

Because of the limited expressivity of the MARCspec there must be some kind of implicit interpretation.

1. A MARCspec without *subfield codes* or *position or range* is a reference to all *data elements* of the field.
2. A *fieldSpec * or a *subfieldSpec* without an explicitly given *index* is always an abbreviation of a reference with the starting index ```0``` and the ending index ```#``` (see [Abbreviation of fieldSpec or subfieldSpec]).
3. Omitted *indicators* in a MARCspec are interpreted as wildcards for variable field indicators in the MARC record.

### Interpretation order

1. For repeatable *fields* referenced by *index* and *indicators* the *fields* MUST first be referenced by *index*. *Indicators* work like a filter on the referenced *fields* as a second order. 

### Character position or range and field indizes interpretation

1. The *postion* character ```#``` is always a reference to the last character in the *data content*.
2. For character range, if the *positive integer* used for the character starting position is greater than the *positive integer* used for the character ending position, the current spec MUST NOT reference any data.
3. For character range, if the character ```#``` is used for the character starting position, the character indices MUST be interpreted backwards (like character ending position ```0``` for the last character, ```1``` for the last but one character, ```2``` for the last but two characters etc.).
4. These above rules also apply for *indices* (index).

### SubSpec interpretation

1. For __chained subTermSets__, if one *subTermSet* gets validated as true, the preceding spec gets referenced (OR) as long as all other *repeated SubSpecs* are validated as true.
2. For __repeated subSpecs__, if one *subSpec* gets validated as false, the preceding spec doesn't get referenced (AND).
3. For abbreviated *fieldSpec* or *subfieldSpec* as a *subTerm*, the last explicitly given *fieldTag* is the current *fieldTag*.
4. As a shortcut, the left hand *subTerm* might be omitted. This implicitly makes the last explicitly given *fieldTag* plus the last explicitly given *characterSpec* or *subfieldCodeSpec* the current (left hand) *subTerm*.
5. If the left hand *subTerm* is omitted, as a shortcut for the operator ```?```, the operator can also be omitted. 

### SubSpec abbreviation

The following tableshows how SubSpec abbreviation MUST be interpreted.

| corresponding spec type | corresponding spec end with | abbreviated spec begins with | interpretation      | example    |
|:-----------------------:|:---------------------------:|:----------------------------:|:-------------------:|:----------:|
|fieldSpec|index|index|valid FieldSpec with index|```...[2]{[1]} => ...[2]{...[1]}```|
|fieldSpec|index|characterSpec|valid fieldSpec with index and characterSpec|```...[1]{/0-3} => ...[1]{...[1]/0-3}```|
|fieldSpec|index|indicators|valid fieldSpec with index and indicators|```...[1]{_01} => ...[1]{...[1]_01}```|
|fieldSpec|characterSpec|index|valid fieldSpec with index|```.../0-7{[0]} => .../0-7{005[0]}```|
|fieldSpec|characterSpec|characterSpec|valid fieldSpec with characterSpec|```.../0-7{/0=\2} => .../0-7{.../0}```|
|fieldSpec|characterSpec|indicators|__invalid__ fieldspec since characterSpec denotes a fixedField, which can't be used with indicators|```.../0-7{_1} => .../0-7{.../0-7_1}```|
|fieldSpec|indicators|index|valid fieldSpec with index|```...[1]_1{[0]} => ...[1]_1{...[0]}```<br/>```..._1{[1]} => ..._1{...[1]}```<br/>```...[1]_1{[0]_0} => ...[1]_1{...[1]_1}```|
|fieldSpec|indicators|characterSpec|__invalid__ fieldspec since indicators denotes a variableField, which can't be used with characterSpec|```245_00{/0-2} => 245_00{245_00/0-2}```|
|fieldSpec|indicators|indicators|valid fieldSpec with indicators|```..._1{_01} => ..._1{..._01}```|
|subfieldSpec|index|index|valid subfieldSpec with index|```...$a[0]{[1]} => ...$a[0]{...$a[1]}```|
|subfieldSpec|index|characterSpec|valid subfieldSpec with index and characterSpec|```...$a[0]{/0} => ...$a[0]{...$a[0]/0}```|
|subfieldSpec|characterSpec|index|valid subfieldSpec with index|```...$a/1{[1]} => ...$a/1{...$a[1]}```<br/>```...$a/1{[1]/1} => ...$a/1{...$a[1]/1}```|
|subfieldSpec|characterSpec|characterSpec|valid subfieldSpec with characterSpec|```...$a/1{/0} => ...$a/1{...$a/0}```|


### SubSpec validation

*SubSpecs* get validated by the following rules:

A *subSpec* is __true__, if

- with the operator ```=``` one of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator ```!=``` none of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator ```~``` one of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator ```!~``` none of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator ```?``` by the *right hand subTerm* referenced data exists.
- with the operator ```!``` by the *right hand subTerm* no referenced data exists.
- one of the *chained subTermSets* is validated as true (OR) and all other *repeated subSpecs* are validated as true.
- all of the *repeated subSpecs* are validated as true (AND).

A *subSpec* is __false__, if

- the left hand *subTerm* does not reference any data (null).
- with the operator ```=``` none of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator ```!=``` one of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator ```~``` none of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator ```!~``` one of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator ```?``` by the *right hand subTerm* no referenced data exists (null).
- with the operator ```!``` by the *right hand subTerm* referenced data exists.
- all of the *chained subTermSets* are validated as false (OR).
- one of the *repeated subSpecs* is validated as false (AND).

### SubTerm validation table

| operator  | right is null | left equals right | right is subpart of left | left is subpart of right | other |
|:---------:|:-------------:|:-----------------:|:------------------------:|:------------------------:|:-----:|
|__=__      |false          |true               |false                     |false                     |false  |
|__!=__     |true           |false              |true                      |true                      |true   |
|__~__      |false          |true               |true                      |false                     |false  |
|__!~__     |true           |false              |false                     |true                      |true   |
|__?__      |false          |true               |true                      |true                      |true   |
|__!__      |true           |false              |false                     |false                     |false  |

# Definition of MARC related terms used in this spec

[Definition of MARC related terms used in this spec]: #definition-of-marc-related-terms-used-in-this-spec

MARCspec does not redefine terms already used by the [Network Development and MARC Standards Office]. MARCspec lists the definition of core terms taken from the [RECORD STRUCTURE] document and adds some for clarification purposes.

| term                          | definition   |
|-------------------------------|--------------|
|__content designation__        |The codes and conventions established explicitly by MARC 21 to identify and further characterize the *data elements* within a record and to support the manipulation of that data.|
|__content of field__           |see *field content*|
|__control field__              |A *variable field* containing information useful or required for the processing of the record. Control fields are assigned *tags* beginning with two zeroes. Control fields with fixed length data elements are restricted to ASCII graphics.|
|__data content__               |see *data element value*|
|__data element__               |A defined unit of information|
|__data element identifier__    |A one-character code used to identify individual *data elements* within a *variable field*. The *data element* may be any ASCII lowercase alphabetic, numeric, or graphic symbol except blank.|
|__data element value__         |The value of a *data element* in a *data field*.|
|__data field__                 |A *variable field* containing bibliographic or other data. Data fields are assigned *tags* beginning with characters other than two zeroes. Data fields contain data in any MARC 21 character set unless a field-specific restriction applies.|
|__data in field__              |see *field content*|
|__delimiter__                  |ASCII control character 1F(hex) (represented graphically in MARC 21 documentation as ASCII control character 1F (hex) or $), which is combined with a *data element identifier* to make up the *subfield code* which precedes each individual *data element* within a variable field. The ASCII name for the delimiter is unit separator (US).|
|__field__                      |A defined character string that may contain one or more *data elements*.|
|__field content__              |All *data elements* in a field without *indicators*. See also *field data*.|
|__field data__                 |All *data elements* in a field including *indicators*.|
|__field index__                |see *index*|
|__field tag__                  |see *tag*|
|__fixed field__                |A *field* whose length does not vary. The term is occasionally used to refer to *variable control fields*, especially those that contain coded data such as fields 007 or 008.|
|__index__                      |The count of repeatable *fields* and *subfields*. The first *field* in a list of repeatable *fields* or the first *subfield* in a list of repeatable *subfields* has always the index *0*.|
|__indicator__                  |A *data element* associated with a *data field* that supplies additional information about the field. An indicator may be any ASCII lowercase alphabetic, numeric, or blank. Indicators are not used in *control fields*.|
|__leader__                     |A *fixed field* that occurs at the beginning of each record and provides information for the processing of the record.|
|__set of data__                |A set of *data elements* referenced by a MARCspec.|
|__subfield__                   |A *data element* including its *subfield code*. It's identified by a *data element identifier* within a *variable field*.|
|__subfield code__              |The two-character combination of a *delimiter* followed by a *data element identifier*. Subfield codes are not used in *control fields*.|
|__subfield index__             |see *index*|
|__subfield code__               |see *data element identifier*|
|__tag__                        |A three character string used to identify or label an associated *field*. The tag may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both).|
|__variable control field__     |see *control field*|
|__variable data field__        |see *data field*|
|__variable field__             |A *field* whose length is determined for each occurrence by the length of data comprising that occurrence. There are two types of variable fields *control fields* and *data fields*.|

# References

## Normative references

- [RFC 2119]
- [RFC 5234]
- [ISO 2709]

## Informative references

- [MARC]
- [MARC 21 Principles]
- [RECORD STRUCTURE]
- [marcspec]
- [solrmarc]
- [catmandu]
- [easyM2R]
- [Network Development and MARC Standards Office]


[MARC]: http://www.loc.gov/marc/
[MARC 21 Principles]: http://www.loc.gov/marc/96principl.html
[marcspec]: https://github.com/billdueber/marcspec
[solrmarc]: https://code.google.com/p/solrmarc/
[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[RFC 5234]: http://tools.ietf.org/html/rfc5234
[catmandu]: http://librecat.org/
[ISO 2709]: http://en.wikipedia.org/wiki/ISO_2709
[easyM2R]: https://github.com/cKlee/easyM2R
[Network Development and MARC Standards Office]: http://www.loc.gov/marc/ndmso.html
[RECORD STRUCTURE]: http://www.loc.gov/marc/specifications/specrecstruc.html