# Introduction

Since it became a common task to map [MARC] data to arbitrary formats, these mappings are usually based on a set of rules, defining what data of a MARC record should be accessed. These rules are commonly called *MARC field specification*.

There are already several tools that each use their own and therefore different *MARC field specification*. The hereby described specification **MARCspec** is an approach to normalizing such field specifications in terms of unification and interchangeability.

# Status of this document

The current version of this proposal is a preliminary draft for open discussion. [Feedback](https://github.com/cklee/marc-spec/issues) is welcome!

# Terminology

The keywords 'MUST', 'MUST NOT', 'REQUIRED', 'SHALL', 'SHALL NOT', 'SHOULD', 'SHOULD NOT', 'RECOMMENDED', 'MAY', and 'OPTIONAL' in this document are to be interpreted as described in [RFC 2119].

See also [Definition of MARC related terms used in this spec].

# What is a MARCspec?

Machine-Readable Cataloguing (MARC) is a document based exchange format for bibliographic and other library related data. A MARC record consists of three main sections: the **leader**, the **directory**, and the **variable fields** with the **data content**.

There are two kinds of (variable) fields: (variable) **control fields** and (variable) **data fields**. The term **fixed field** stands for fields whose length does not vary like the leader and some *control fields*. The **field content** in the *fixed fields* can be accessed through its **character position or range**. Only *data fields* are divided into *subfields*. *Subfields* can also be contextualized through indicators. There is an *indicator 1* and an *indicator 2* for all data fields, both are optional.

A **MARCspec** is a reference to field data of a MARC record and is very much like XPath for XML. With MARCspec one can reference data on different levels of a MARC record defined through the *fields*, *character positions*, *subfields* and *indicators*.

The *data* of the MARC record being referenced may be represented through a *set of data*, having zero or more *data elements*. MARCspec does neither define the form of this referenced *set of data*, nor the encoding of the referenced *data content*.  

# Limitations of MARCspec as string

A MARCspec might not fulfil all requirements of definition for a reference to the desired set of data like XPath does for XML. This is because of the nearly unlimited number of options accessing data in a MARC record, especially when it comes to delimiters based on cataloging rules. Thus a MARCspec has to concentrate on the basic references and let all other data processing to subsequent data processing functions of tools having implemented MARCspec.

To enable support for other [ISO 2709] applications MARCspecs syntax does not distinguish between types of fields like in the [MARC] record structure. A valid MARCspec might violate the MARC record structure. It is led to the MARCspec aware tools weather to check for MARC record structure violation or not. 

# Basic references

A MARCspec allows the following basic references:

- Reference to *record data* (except data from the record directory)
- Reference to *field data*
- Reference to *data content* of subfields
- Reference to substrings of *data content* of fixed fields and subfields
- Reference to *values* of indicators

References a MARCspec does not allow are:

- Reference to single *designators*, *field tags* or *subfield codes*
- Reference to a position index of a specific field, subfield or character
- Reference to data in related records
- Reference to *data elements* (entries) in the MARC record directory

# Form of MARCspec

**This section is normative.**

The primary form of a MARCspec is a string.

The **Augmented BNF for Syntax Specifications: ABNF** [RFC 5234] is used to define the form of the MARCspec as string.

The whole ABNF for MARCspec shows as follows

    alphaupper        = %x41-5A
                        ; A-Z
    alphalower        = %x61-7A
                        ; a-z
    DIGIT             =  %x30-39
                        ; 0-9
    VCHAR             =  %x21-7E
                        ; visible (printing) characters
    positiveDigit     = %x31-39
                        ;  "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9"
    positiveInteger   = "0" / positiveDigit [1*DIGIT]
    fieldTag          = 3(alphalower / DIGIT / ".") / 3(alphaupper / DIGIT / ".")
    position          = positiveInteger / "#"
    range             = position "-" position
    positionOrRange   = range / position
    characterSpec     = "/" positionOrRange
    index             = "[" positionOrRange "]"
    fieldSpec         = fieldTag [index] [characterSpec]
    abrFieldSpec      = index [characterSpec] / characterSpec
    subfieldChar      = %x21-3F / %x5B-7B / %x7D-7E
                        ; ! " # $ % & ' ( ) * + , - . / 0-9 : ; < = > ? [ \ ] ^ _ \` a-z { } ~
    subfieldCode      = "$" subfieldChar
    subfieldCodeRange = "$" ( (alphalower "-" alphalower) / (DIGIT "-" DIGIT) )
                        ; [a-z]-[a-z] / [0-9]-[0-9]
    abrSubfieldSpec   = (subfieldCode / subfieldCodeRange) [index] [characterSpec]
    subfieldSpec      = fieldTag [index] abrSubfieldSpec
    abrIndicatorSpec  = [index] "^" ("1" / "2")
    indicatorSpec     = fieldTag abrIndicatorSpec
    comparisonString  = "\" *VCHAR
    operator          = "=" / "!=" / "~" / "!~" / "!" / "?"
                        ; equal / unequal / includes / not includes / not exists / exists
    abbreviation      = abrFieldSpec / abrSubfieldSpec / abrIndicatorSpec
    subTerm           = fieldSpec / subfieldSpec / indicatorSpec / comparisonString / abbreviation
    subTermSet        = [ [subTerm] operator ] subTerm
    subSpec           = "{" subTermSet *( "|" subTermSet ) "}"
    MARCspec          = fieldSpec *subSpec / (subfieldSpec *subSpec *(abrSubfieldSpec *subSpec)) / indicatorSpec *subSpec

# MARCspec explained

## General form

Every **MARCspec** is either a spec for *field data*, *subfield data* or *indicator values*. All specs can be contextualized through *subSpecs* (see section [SubSpecs]).

    MARCspec = fieldSpec *subSpec / (subfieldSpec *subSpec *(abrSubfieldSpec *subSpec)) / indicatorSpec *subSpec

## Reference to field data

A **fieldSpec** is a reference to *field data* of a field. It consists of the three character *field tag*, followed optionally

- by an *index* (see [Reference to occurrence]) and
- by an *characterSpec* (for *fixed fields*) (see section [Reference to substring]).

The **field tag** may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both) or the character `.`. The character `.` is interpreted as a wildcard. E.g. '3..' is then a reference to the *data elements* in all *fields* beginning with '3'. 

The special *field tag* `LDR` is the *field tag* for the *leader*. 

    alphaupper = %x41-5A ; A-Z
    alphalower = %x61-7A; a-z
    DIGIT      =  %x30-39; 0-9
    fieldTag   = 3(alphalower / DIGIT / ".") / 3(alphaupper / DIGIT / ".")
    fieldSpec  = fieldTag [index] [characterSpec]

<div class="note">
A **fieldSpec** without an explicitly given *index* is always a reference to all occurrences of the field(s) (see [MARCspec interpretation] for implicit rules). One MUST also conclute that a **fieldSpec** without an explicitly given *index* is an abbreviation of a reference with the starting index `0` and the ending index `#` (see [Reference to occurrence] examples).
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
Reference to *data elements* of all repetitions of the '100' field.

    100

</div>

## Reference to substring

[Reference to substring]: #reference-to-substring

A **characterSpec** is a reference to a character or a range of characters within a *field* or *subfield*. It consists of a *position or range* prefixed with the character `/`.

    characterSpec = "/" positionOrRange

A **positionOrRange** is either a *postion* or a *range*.

The **postion** is either a *positive integer* or the character `#` as a symbol for the last character of the referenced *data content*. 

The **range** consists of two *positions* concatenated with the character `-`. 

    positiveDigit   = %x31-39
                        ;  "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9"
    positiveInteger = "0" / positiveDigit [1*DIGIT]
    position        = positiveInteger / "#"
    range           = position "-" position
    positionOrRange = range / position

<div class="note">
Interpretation of a *range* differs through the position of the special *position* character `#` as a symbol for the last character of the referenced *data content* (see [MARCspec interpretation] for implicit rules).
</div>

<div class="example">
Reference to substring of *field data* in the *leader* from character position '0' to character position '4' (5 characters).

    LDR/0-4

</div>

<div class="example">
Reference to data in the *leader* at character position '6' (1 character).

    LDR/6

</div>

<div class="example">
Reference to data in the control field '007' at character position '0' (1 character).

    007/0

</div>

<div class="example">
Reference to all data but the first character in the control field '007'.

    007/1-#

</div>
<div class="example">
Reference to the last character in the control field '007'.

    007/#

</div>

<div class="example">
Reference to the last two characters of the value of the subfield 'a' of field '245'.

    245$a/#-1
</div>

## Reference to data content

[Reference to data content]: #reference-to-data-content

The **subfieldSpec** is a reference to the *data content* (value(s)) of (a) *subfield(s)* of a *variable field*. It consists of 

- a *field tag*, optionally followed by an *index*,
- a *subfieldCode* or a *subfieldCodeRange*,  optionally followed by an index and/or *characterSpec*.

<b></b>

    abrSubfieldSpec = (subfieldCode / subfieldCodeRange) [index] [characterSpec]
    subfieldSpec    = fieldTag [index] abrSubfieldSpec

A **subfieldCode** is a *subfieldChar* prefixed by the character `$`.

A **subfieldCodeRange** is prefixed by the character `$` and restricted to either two alphabetic or two numeric characters both concatenated with the character `-`.

A **subfieldChar** is a lowercase alphabetic, a numeric character or a special character.

    subfieldChar      = %x21-3F / %x5B-7B / %x7D-7E
    subfieldCode      = "$" subfieldChar
    subfieldCodeRange = "$" ( (%x61-7A "-" %x61-7A) / (%x30-39 "-" %x30-39) )

<div class="example">
Reference to value of the subfield 'a' of field '245'.

    245$a

</div>

<div class="example">
Reference to the value of the subfields 'a', 'b' and 'c' of field '245'.

    245$a$b$c

</div>

<div class="example">
Same as above, but with the use of a *subfield code range*.

    245$a-c

</div>

<div class="example">
Reference to values of subfields '_' and '$'.

    ...$_$$
</div>

## Reference to occurrence

[Reference to occurrence]: #reference-to-occurrence

For repeatable *fields* and *subfields* each occurrence can be referenced by its **index**. An index is a *position or range* enclosed with the characters `[` and `]`. The first repetition of a *field* or a *subfield* is always referenced with the index `[0]`. The last repetition of a *field* or a *subfield* is referenced with the index `[#]`.

    index = "[" positionOrRange "]"

<div class="example">
Reference to the first '300' field.

    300[0]

</div>

<div class="example">
Reference to the second of the '300' field.

    300[1]

</div>

<div class="example">
Reference to the first, second and third of the '300' field.

    300[0-2]

</div>

<div class="example">
Reference to all but the first of the '300' field.

    300[1-#]

</div>

<div class="example">
Reference to the last of the '300' field.

    300[#]

</div>

<div class="example">
Reference to the last two of the '300' field.

    300[#-1]

</div>

<div class="example">
Reference to value of the subfield 'a' of the first '300' field.

    300[0]$a

</div>

<div class="example">
Reference to the value of the first subfield 'a' of the field '300'

    300$a[0]

</div>

<div class="example">
Reference to the value of the last subfield 'a' of the field '300'

    300$a[#]

</div>

<div class="example">
Reference to the value of the last two repetitions of subfield 'a' of the field '300'

    300$a[#-1]

</div>

## Reference to indicator values

An **indicatorSpec** is a reference to the value of either **indicator 1** or **indicator 2** of a variable field. It consists of a **field tag**, followed by 

- optionally an **index**
- a caret symbol ('^') 
- and either the number 1 for **indicator 1** or number 2 for **indicator 2**.

<b></b>

    abrIndicatorSpec = [index] "^" ("1" / "2")
    indicatorSpec    = fieldTag abrIndicatorSpec

<div class="example">
Reference to value(s) of *indicator 1* of all occurrences of field '880'.

    880^1

</div>
<div class="example">
Reference to value of *indicator 2* of first repetition of field '880'.

    880[1]^1

</div>

## SubSpecs

### General

With a **subSpec** the preceding *fieldSpec*, *subfieldSpec* or *indicatorSpec* gets contextualized. Every *subSpec* MUST be validated either **true** or **false**. Is a *subSpec* **true**, the corresponding spec (the last spec outside of the subSpec) is used to reference data. Is a subSpec **false**, the corresponding spec does not reference data.

A *subSpec* is enclosed with the characters `{` and `}`. A *subSpec* consists of one or more sets of **subTerms** (the **left hand subTerm** and the **right hand subTerm**) and an **operator**. This combination of *subTerms* and *operator* can be chained through the character `|` (**OR**) within a *subSpec*. Multiple *subSpecs* can also be *repeated* one after another (**AND**).

    subTerm    = fieldSpec / subfieldSpec / indicatorSpec / comparisonString / abbreviation
    subTermSet = [ [subTerm] operator ] subTerm
    subSpec    = "{" subTermSet *( "|" subTermSet ) "}"

The **operator** is one of

- `=` (as a symbol for 'equal'), 
- `!=` (as a symbol for 'unequal'),
- `~` (as a symbol for 'includes'), 
- `!~` (as a symbol for 'not includes')
- `!` (as a symbol for 'not exists') or
- `?` (as a symbol for 'exists').

<b></b>

    operator = "=" / "!=" / "~" / "!~" / "!" / "?"
 
A **subTerm** is one of

- *fieldSpec*,
- *subfieldSpec*,
- *indicatorSpec*,
- *comparisonString* or
- *abbreviation*.

By omitting the *left hand subTerm*, this implicitly makes the corresponding spec outside the *subSpec* the *left hand subTerm* (see [MARCspec interpretation] for implicit rules). For *subSpecs* with omitted *left hand subTerm* the *operator* can also be omitted. Omitting the *operator* implies the usage of the *operator* `?` (exists).

<div class="example">
Checking existence of fields

Reference data content of subfield 'c' of field '020', if subfield 'a' of field '020' exists.

    020$s{?020$a}

same as

    020$c{020$c?020$a}

</div>

<div class="example">
Checking (non) existence of fields

Reference data content of subfield 'z' of field '020', if subfield 'a' of field '020' does not exist.

    020$z{!020$a}

same as

    020$z{020$z!020$a}

</div>

A **comparisonString** can be every combination of ASCII characters prefixed by the `\` character. For unambiguousness in a *comparisonString* the following characters MUST be escaped by the character `\`:

- `$`
- `{`
- `}`
- `!`
- `=`
- `~`
- `?`
- `|`

In a *comparisonString* a whitespace MUST be encoded as the character combination `\s`.

    comparisonString = "\" *VCHAR

<div class="example">
Checking dependencies via string comparison

If Leader/06 = t: Books

Reference to character with position '18' of field '008', if character with position '06' in Leader equals 't'.

    008/18{LDR/6=\t}

</div>
<div class="example">
Checking dependencies via string comparison alternatives

If Field 007/00 = a and t

Reference to subfield 'b' of field '245', if character with position '0' of field 007 equals 'a' OR 't'.

    245$b{007/0=\a|007/0=\t}

</div>
<div class="example">
Checking dependencies via string comparison chains

If Leader/06 = a and Leader/07 = a, c, d, or m: Books

Reference to character with position '18' of field '008', if character with position '06' in Leader equals 'a' AND character with position '07' in Leader equals 'a', 'c', 'd' OR 'm'.

    008/18{LDR/6=\a}{LDR/7=\a|LDR/7=\c|LDR/7=\d|LDR/7=\m}

</div>
<div class="example">
Checking dependencies via string comparison and content comparison

Example data:

100  1#$6880-01$aZilbershtain, Yitshak ben David Yosef.<br>
880  1#$6100-01/(2/r$a, יצחק יוסף בן דוד.

Reference data content of subfield 'a' of field '880', if data content of subfield '6' of field '100' includes the string '-01' (characters with index range 3-5 of field '800') and the string '880'.

    880$a{100$6~$6/3-5}{100$6~\880}

</div>

### Abbreviations

[Abbreviations]: #abbreviations

When used as a *subTerm*, *fieldSpec*, *subfieldSpec* and *indcatorSpec* can be **abbreviated**.

    abbreviation = abrFieldSpec / abrSubfieldSpec / abrIndicatorSpec

See also [SubSpec abbreviation rules].

An abbreviated **fieldSpec** is one of

- *index* or
- *index* and *characterSpec* or
- *characterSpec*.

<b></b>

    abrFieldSpec = index [characterSpec] / characterSpec

<div class="example">
Reference to third character of second field '007', if first character of of second field '007' equals 'v'. 

    007[1]/3{/0=\v}

same as

    007[1]/3{007[1]/0=\v}

</div>

An abbreviated **subfieldSpec** is one of

- *subfieldCode* or *subfieldCodeRange*, *index* and *characterSpec* or
- *subfieldCode* or *subfieldCodeRange* and *index* or
- *subfieldCode* or *subfieldCodeRange* and *characterSpec* or 
- *subfieldCode* or *subfieldCodeRange*.

<b></b>

    abrSubfieldSpec  = (subfieldCode / subfieldCodeRange) [index] [characterSpec]

<div class="example">
Reference to data content of subfield 'c' of field 020, if subfield 'a' of field '020' exists.

    020$c{$a}

same as

    020$c{020$a}

</div>
<div class="example">

Reference of data content of subfield 'a' of field '245',
 if last character of the preceding spec equals the comparisonString '/'.
 
    245$a{/#=\/}

same as

    245$a{245$a/#=\/}

</div>

An abbreviated **indicatorSpec** is one of

- *index* and *indicator 1*
- *index* and *indicator 2*
- *indicator 1* or *indicator 2*.

<b></b>

    abrIndicatorSpec  = [index] "^" ("1" / "2")

<div class="example">

Reference to data of the first field '800', having '1' as value for *indicator 2* and data content of subfield 'a' includes the comparisonString 'Poe'.

    800[0]{$a~\Poe}{^2=1}

same as

    800[0]{800[0]$a~\Poe}{800[0]^2=1}

</div>

#### Implicit abbreviations

Using an abbreviated *subfieldSpec* without *subfieldCode* or *subfieldCodeRange* makes the *subfieldCode* or the *subfieldCodeRange* of the corresponding *subfieldSpec* implicit.

<div class="example">

    245$a{/0-2=\The}

same as

    245$a{245$a/0-2=\The}

</div>

According to [MARCspec interpretation] a MARCspec without an explicitly given index is always an abbreviation of *n* references. The fololwing examples show how these specs are interpreted.

<div class="example">
Example Data:

    020 ##$a0394170660$qRandom House$c$4.95
    020 ##$a0491001304

Reference to data content of subfield 'q' of field '020' if subfield 'c' exists.

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

Reference to data content of subfield 'c' if data content of one repetition of subfield 'q' equals the comparison string 'paperback'.

    020$c{$q=\paperback}

same as

    020[0-#]$c[0-#]{$q[0-#]=\paperback}

same as 

    020[0]$c[0]{020[0]$q[0]=\paperback} OR // false
    020[0]$c[0]{020[0]$q[1]=\paperback} OR // true
    020[1]$c[0]{020[1]$q[0]=\paperback} OR // false
    020[1]$c[0]{020[1]$q[1]=\paperback}    // false

</div>

# MARCspec interpretation

[MARCspec interpretation]: #marcspec-interpretation

**This section is normative.**

Because of the limited expressivity of the MARCspec, there must be some kind of implicit interpretation.

1. A MARCspec without *subfield codes* or *position or range* is a reference to all *data elements* of the field.
2. A *fieldSpec*, a *subfieldSpec* or an *indicatorSpec* without an explicitly given *index* is always an abbreviation of a reference with the starting index `0` and the ending index `#`.

## Character position or range and field indizes interpretation

1. The *postion* character `#` is always a reference to the last character in the *data content*.
2. For character range, if the *positive integer* used for the character starting position is greater than the *positive integer* used for the character ending position, the current spec MUST NOT reference any data.
3. For character range, if the character `#` is used for the character starting position, the character indices MUST be interpreted backwards (like character ending position `0` for the last character, `1` for the last but one character, `2` for the last but two characters etc.).
4. These above rules also apply for *index*.

## SubSpec interpretation

1. For **chained subTermSets**, if one *subTermSet* gets validated as true, the preceding spec gets referenced (OR) as long as all other *repeated SubSpecs* are validated as true.
2. For **repeated subSpecs**, if one *subSpec* gets validated as false, the preceding spec doesn't get referenced (AND).
3. For abbreviated *fieldSpec*, *subfieldSpec* or *indicatorSpec* as a *subTerm*, the last explicitly given *fieldTag* outside of the subSpec is the current *fieldTag*.
4. As a shortcut, the *left hand subTerm* might be omitted. This implicitly makes the last explicitly given *fieldTag* outside of the subSpec plus the last explicitly given *characterSpec* or *subfieldCodeSpec* the *left hand subTerm*.
5. If the *left hand subTerm* is omitted, as a shortcut for the operator `?`, the operator can also be omitted. 

## SubSpec abbreviation rules

[SubSpec abbreviation rules]: #subspec-abbreviation-rules

The following table shows how SubSpec abbreviation MUST be interpreted.

| corresponding spec type | corresponding spec end with | abbreviated spec begins with |                                             interpretation                          |                                                             example                                                             |
| :---------------------: | :-------------------------: | :--------------------------: | :---------------------------------------------------------------------------------: | :-----------------------------------------------------------------------------------------------------------------------------: |
|        fieldSpec        |            index            |            index             |                                       valid fieldSpec with index                                        |                                                `...[2]{[1]}` => `...[2]{...[1]}`                                                |
|        fieldSpec        |            index            |        characterSpec         |                              valid fieldSpec with index and characterSpec                               |                                             `...[1]{/0-3}` => `...[1]{...[1]/0-3}`                                              |
|        fieldSpec        |            index            |        indicatorSpec         |                                     valid indicatorSpec with index                                      |                                               `...[1]{^1}` => `...[1]{...[1]^1}`                                                |
|        fieldSpec        |        characterSpec        |            index             |                                       valid fieldSpec with index                                        |                                               `.../0-7{[0]}` => `.../0-7{005[0]}`                                               |
|        fieldSpec        |        characterSpec        |        characterSpec         |                                   valid fieldSpec with characterSpec                                    |                                              `.../0-7{/0=\2}` => `.../0-7{.../0}`                                               |
|        fieldSpec        |        characterSpec        |        indicatorSpec         | **invalid** indicatorSpec since characterSpec denotes a fixedField,<br/> which can't be used with indicators |                                                    `.../0-7{^1}` => invalid                                                     |
|      subfieldSpec       |            index            |            index             |                                      valid subfieldSpec with index                                      |                                             `...$a[0]{[1]}` => `...$a[0]{...$a[1]}`                                             |
|      subfieldSpec       |            index            |        characterSpec         |                             valid subfieldSpec with index and characterSpec                             |                                            `...$a[0]{/0}` => `...$a[0]{...$a[0]/0}`                                             |
|      subfieldSpec       |        characterSpec        |            index             |                                      valid subfieldSpec with index                                      |                    `...$a/1{[1]}` => `...$a/1{...$a[1]}`<br/><br/>`...$a/1{[1]/1}` => `...$a/1{...$a[1]/1}`                     |
|      subfieldSpec       |        characterSpec        |        characterSpec         |                                  valid subfieldSpec with characterSpec                                  |                                               `...$a/1{/0}` => `...$a/1{...$a/0}`                                               |
|      indicatorSpec      |              1              |              1               |                                   valid indicatorSpec for indicator 1                                   |                                              `...^1{^1!=\_}` => `...^1{...^1!=\_}`                                              |
|      indicatorSpec      |              1              |              2               |                                   valid indicatorSpec for indicator 1                                   |                                              `...^1{^2!=\_}` => `...^1{...^2!=\_}`                                              |
|      indicatorSpec      |              2              |              1               |                                   valid indicatorSpec for indicator 2                                   |                                              `...^2{^1!=\_}` => `...^2{...^1!=\_}`                                              |
|      indicatorSpec      |              2              |              2               |                                   valid indicatorSpec for indicator 2                                   |                                              `...^2{^2!=\_}` => `...^2{...^2!=\_}`                                              |
|      indicatorSpec      |           1 or 2            |            index             |                        valid fieldSpec, subfieldSpec or indicatorSpec with index                        | `...^2{[1]}` => `...^2{...[1]}`<br/><br/>`...^2{[1]$a}` => `...^2{...[1]$a}`<br/><br/>`...^2{[1]^1=\_}` => `...^2{...[1]^1=\_}` |
|      indicatorSpec      |           1 or 2            |        characterSpec         |             **invalid** indicatorSpec; characterSpec is not applicable to indicator values              |                                                    `...^2{/0=\1}` => invalid                                                    |

## SubSpec validation

*SubSpecs* get validated by the following rules:

A *subSpec* is **true**, if

- with the operator `=` **one** of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator `!=` **none** of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator `~` **one** of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator `!~` **none** of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator `?` by the *right hand subTerm* referenced data **exists**.
- with the operator `!` by the *right hand subTerm* referenced data **not exists**.
- **one** of the *chained subTermSets* is validated as true (OR) and all other *repeated subSpecs* are validated as true.
- **all** of the *repeated subSpecs* are validated as true (AND).

A *subSpec* is **false**, if

- the left hand *subTerm* **does not** reference any data.
- with the operator `=` **none** of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator `!=` **one** of the referenced values of the *left hand subTerm* is equal to one of the referenced values of the *right hand subTerm*.
- with the operator `~` **none** of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator ``!~` **one** of the referenced values of the *left hand subTerm* includes one of the referenced values of the *right hand subTerm*.
- with the operator `?` by the *right hand subTerm* referenced data **not exists**.
- with the operator `!` by the *right hand subTerm* referenced data **exists**.
- **all** of the *chained subTermSets* are validated as false (OR).
- **one** of the *repeated subSpecs* is validated as false (AND).

## SubTerm validation table

| operator | right is null | left equals right | right is subpart of left | left is subpart of right | other |
| :------: | :-----------: | :---------------: | :----------------------: | :----------------------: | :---: |
|  **=**   |     false     |       true        |          false           |          false           | false |
|  **!=**  |     true      |       false       |           true           |           true           | true  |
|  **~**   |     false     |       true        |           true           |          false           | false |
|  **!~**  |     true      |       false       |          false           |           true           | true  |
|  **?**   |     false     |       true        |           true           |           true           | true  |
|  **!**   |     true      |       false       |          false           |          false           | false |

# Definition of MARC related terms used in this spec

[Definition of MARC related terms used in this spec]: #definition-of-marc-related-terms-used-in-this-spec

MARCspec does not redefine terms already used by the [Network Development and MARC Standards Office]. MARCspec lists the definition of core terms taken from the [RECORD STRUCTURE] document and adds some for clarification purposes.

|            term             |                                                                                                                                                                   definition                                                                                                                                                                   |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **content designation**     | The codes and conventions established explicitly by MARC 21 to identify and further characterize the *data elements* within a record and to support the manipulation of that data.                                                                                                                                                             |
| **content of field**        | see *field content*                                                                                                                                                                                                                                                                                                                            |
| **control field**           | A *variable field* containing information useful or required for the processing of the record. Control fields are assigned *tags* beginning with two zeroes. Control fields with fixed length data elements are restricted to ASCII graphics.                                                                                                  |
| **data content**            | see *data element value*                                                                                                                                                                                                                                                                                                                       |
| **data element**            | A defined unit of information                                                                                                                                                                                                                                                                                                                  |
| **data element identifier** | A one-character code used to identify individual *data elements* within a *variable field*. The *data element* may be any ASCII lowercase alphabetic, numeric, or graphic symbol except blank.                                                                                                                                                 |
| **data element value**      | The value of a *data element* in a *data field*.                                                                                                                                                                                                                                                                                               |
| **data field**              | A *variable field* containing bibliographic or other data. Data fields are assigned *tags* beginning with characters other than two zeroes. Data fields contain data in any MARC 21 character set unless a field-specific restriction applies.                                                                                                 |
| **data in field**           | see *field content*                                                                                                                                                                                                                                                                                                                            |
| **delimiter**               | ASCII control character 1F(hex) (represented graphically in MARC 21 documentation as ASCII control character 1F (hex) or $), which is combined with a *data element identifier* to make up the *subfield code* which precedes each individual *data element* within a variable field. The ASCII name for the delimiter is unit separator (US). |
| **field**                   | A defined character string that may contain one or more *data elements*.                                                                                                                                                                                                                                                                       |
| **field content**           | All *data elements* in a field without *indicators*. See also *field data*.                                                                                                                                                                                                                                                                    |
| **field data**              | All *data elements* in a field including *indicators*.                                                                                                                                                                                                                                                                                         |
| **field index**             | see *index*                                                                                                                                                                                                                                                                                                                                    |
| **field tag**               | see *tag*                                                                                                                                                                                                                                                                                                                                      |
| **fixed field**             | A *field* whose length does not vary. The term is occasionally used to refer to *variable control fields*, especially those that contain coded data such as fields 007 or 008.                                                                                                                                                                 |
| **index**                   | The count of repeatable *fields* and *subfields*. The first *field* in a list of repeatable *fields* or the first *subfield* in a list of repeatable *subfields* has always the index *0*.                                                                                                                                                     |
| **indicator**               | A *data element* associated with a *data field* that supplies additional information about the field. An indicator may be any ASCII lowercase alphabetic, numeric, or blank. Indicators are not used in *control fields*.                                                                                                                      |
| **leader**                  | A *fixed field* that occurs at the beginning of each record and provides information for the processing of the record.                                                                                                                                                                                                                         |
| **set of data**             | A set of *data elements* referenced by a MARCspec.                                                                                                                                                                                                                                                                                             |
| **subfield**                | A *data element* including its *subfield code*. It's identified by a *data element identifier* within a *variable field*.                                                                                                                                                                                                                      |
| **subfield code**           | The two-character combination of a *delimiter* followed by a *data element identifier*. Subfield codes are not used in *control fields*.                                                                                                                                                                                                       |
| **subfield index**          | see *index*                                                                                                                                                                                                                                                                                                                                    |
| **subfield designator**     | see *data element identifier*                                                                                                                                                                                                                                                                                                                  |
| **tag**                     | A three character string used to identify or label an associated *field*. The tag may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both).                                                                                                                    |
| **variable control field**  | see *control field*                                                                                                                                                                                                                                                                                                                            |
| **variable data field**     | see *data field*                                                                                                                                                                                                                                                                                                                               |
| **variable field**          | A *field* whose length is determined for each occurrence by the length of data comprising that occurrence. There are two types of variable fields *control fields* and *data fields*.                                                                                                                                                          |

# References

## Normative references

- [RFC 2119]
- [RFC 5234]
- [ISO 2709]

## Informative references

- [MARC]
- [MARC 21 Principles]
- [RECORD STRUCTURE]
- [Network Development and MARC Standards Office]


[MARC]: http://www.loc.gov/marc/
[MARC 21 Principles]: http://www.loc.gov/marc/96principl.html
[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[RFC 5234]: http://tools.ietf.org/html/rfc5234
[ISO 2709]: http://en.wikipedia.org/wiki/ISO_2709
[Network Development and MARC Standards Office]: http://www.loc.gov/marc/ndmso.html
[RECORD STRUCTURE]: http://www.loc.gov/marc/specifications/specrecstruc.html