# Introduction

Since it is a common task to map [MARC] data to arbitrary formats, these mappings are normally based on a set of definitions of MARC fields and subfields called *MARC field specification* or short *MARC spec*.

There are already implementations of MARC specs in tools like [marcspec], [catmandu], [solrmarc] and [easyM2R]. Each of them using a different flavour of MARC spec. This document is an approach to normalize such field specifications.

The hereby described specification **MARCspec** can help to build reusable MARCspec parsers and validators and facilitates the exchange of mapping definitions.

## Status of this document

The current version of this proposal is a preliminary draft for open discussion. [Feedback](https://github.com/cklee/marc-spec/issues) is welcome!

## Terminology

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

See also [Definition of MARC related terms used in this spec].

## What is a MARCspec?

Machine-Readable Cataloguing (MARC) is a document based key-value exchange format for bibliographic data. The data of a MARC record consists of fields, subfields and its contents. There are two kinds of MARC fields: *fixed fields* and *data fields*. The *field content* in the the *fixed fields* can be accessed through its *character position or range*. Only *data fields* are divided into *subfields*. *Subfields* can also be contextualized through *indicators*. There is an *indicator 1* and an *indicator 2* for all *data fields*, both are optional. See [MARC 21 Principles] for a deeper explanation of the MARC 21 format.

**MARCspec** is very much like XPath is for XML. With MARCspec one can reference data on different levels of a MARC record defined through the *fields*, *character positions*, *subfields* and *indicators*.

The *data* of the MARC record being referenced may be represented through a *set of data*, having zero or more *data elements*. MARCspec does neither define the form of this referenced *set of data*, nor the encoding of the referenced *data content*.  

# Limitations of MARCspec as string

A MARCspec might not fulfill all requirements of definition for a reference to the desired set of data like XPath does for XML. This is because of the nearly unlimited number of options accessing data in a MARC record, especially when it comes to delimiters based on cataloging rules. Thus a MARCspec has to concentrate on the basic references and let all other data processing to subsequent data processing functions of tools having implemented MARCspec.

To enable support for other [ISO 2709] applications MARCspecs syntax does not distinguish between types of fields like in the [MARC] record structure. A valid MARCspec might violate the MARC record structure. It is led to the MARCspec aware tools wheater to check for MARC record structure violation or not. 

## Basic references

A MARCspec as string should cope the following basic references:

* Reference to *field data* of *fields* by *field tag*
* Reference to *field data* of one or more repetitions of a *field* by *field index* or field index range
* Reference to *field data* of *varibale fields* within the context of *indicators*
* Reference to substring of *field content* in *fields* by *character position or range*
* Reference to *data content* in *variable fields* by *subfield tags*
* Reference to *data content* of one or more repetitions of a *subfield* by *subfield index* or subfield index range
* Reference to substring of *data content* in *subfields* by *character position or range*

References a MARCspec as string does not cope

* Reference to *data elements* in related records
* Reference to *data content* wthin the context of other *data content*
* Reference to *data elements* (entries) in the MARC record directory

## Form of MARCspec as string

**This section is normative.**

The **Augmented BNF for Syntax Specifications: ABNF** [RFC 2234] is used to define the form of the MARCspec as string.

The whole ABNF for MARCspec shows as follows

```
alphaupper               = %x41-5A ; A-Z
alphalower               = %x61-7A ; a-z
indicator                = 1*1( alphalower / DIGIT )
indicator1               = indicator
indicator2               = indicator
indicators               = "_"( indicator1 / "_" ) [ indicator2 / "_" ]
lastCharacter            = "-" ; the last character of the referenced data content
characterPosition        = *DIGIT / lastCharacter 
characterRange           = *DIGIT "-" [ *DIGIT ]
characterPositionOrRange = characterPosition / characterRange
index                    = "[" characterPositionOrRange "]"
characterSpec            = "/" characterPositionOrRange
subfieldChar             = %x21-3F / %5B-7B / %7D-7E ; ! " # $ % & ' ( ) * + , - . / 0-9 : ; < = > ? [ \ ] ^ _ \` a-z { } ~
subfieldTagRange         = %x61-7A "-" %x61-7A / %x30-39 "-" %x30-39 ; a-z"-"a-z / 0-9"-"0-9 
subfieldTags             = 1*( "$" ( subfieldChar [ index ] / subfieldTagRange ) )
subfieldSpec             = subfieldTags [ indicators ] / indicators subfieldTags
fieldTag                 = 3( alphalower / DIGIT / "." ) / 3( alphaupper / DIGIT / "." )
fieldSpec                = fieldTag [ index ]
MARCspec                 = fieldSpec [ characterSpec / subfieldSpec ]
``` 

### General form

Every MARCspec MUST have a fieldSpec either optionally followed by a characterSpec or a subfieldSpec.

```
MARCspec = fieldSpec [ characterSpec / subfieldSpec ]
```

### Reference to field data

A **fieldSpec** is a reference to *field data* of a field. It consits of the three character *field tag* and optionally an *index*.

The **field tag** may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both) or the character ".". The character "." is interpreted as a wildcard. E.g. *3..* is then a reference to the *data elements* in all *fields* beginning with *3*. 

The special *field tag* "LDR" is the field tag for the *leader*. 

```
alphaupper   = %x41-5A ; A-Z
alphalower   = %x61-7A; a-z
fieldTag     = 3(alphalower / DIGIT / ".") / 3(alphaupper / DIGIT / ".") / "LDR"
fieldSpec    = fieldTag [ index ]
``` 

### Reference to substring

A **characterSpec** is a reference to a character or a range of characters within a field. It consits of the *character position or range* prefixed with the charcter "/".

```
characterSpec = "/" characterPositionOrRange
```

A *character position or range* is either a *character postion* or a *character range*. The *character postion* is either one or more digits or the character "-" as a symbol for the last character of the referenced *data content*. A *character range* consists of one or more digits representing the character starting position followed by the character "-" and optionally followed by one or more digits representing the character ending position. If the character ending position is omitted, this means the character ending position is of the last character of the referenced *data content*.

```
lastCharacter            = "-" ; the last character of the referenced data content
characterPosition        = *DIGIT / lastCharacter 
characterRange           = *DIGIT "-" [ *DIGIT ]
characterPositionOrRange = characterPosition / characterRange
```

### Reference to data content

The **subfieldSpec** is a reference to the value of a *subfield* of a field. It consits of the *subfield tags* either preeceded or optionally followed by indicators.

```
subfieldSpec = subfieldTags [ indicators ] / indicators subfieldTags
```

*Subfield tags* consist of one or more lowercase alphabetic, digits or special characters each of them prefixed with the character "$". A single *subfield tag* may be followed by an *index* (which in case of repeatable subfields is a reference to a specific repetition) and a *characterSpec*. Instead of a list of subfields it is also possible to define a range of subfields. A range of subfields is restricted to either alphabetic or numeric *subfield tags*. Neither *index* nor *characterSpec* are not allowed for subfield ranges.

In a list of *subfield tags*, these may occur in non ordered sequence.

```
subfieldChar     = %x21-3F / %5B-7B / %7D-7E ; ! " # $ % & ' ( ) * + , - . / 0-9 : ; < = > ? [ \ ] ^ _ \` a-z { } ~
subfieldTagRange = %x61-7A "-" %x61-7A / %x30-39 "-" %x30-39 ; a-z"-"a-z / 0-9"-"0-9 
subfieldTags     = 1*( "$" ( subfieldChar [ index ] [ characterSpec ] / subfieldTagRange ) )
```

### Reference to contextualized data

*Indicators* consist of *indicator 1* and *indicator 2*. Both are optional. If *indicator 1* is not specified is must be replaced by the character "\_". If *indicator 2* is not specified it might be replaced by the character "\_" or left blank.

```
indicators = "_"(indicator1 / "_") [ indicator2 / "_" ]
```

If present both *indicator 1* and *indicator 2* consists of one lowercase alphabetic or numeric character.

```
indicator  = 1*1(alphalower / DIGIT)
indicator1 = indicator
indicator2 = indicator
```

### Reference to repetitions

An index has the same form as the *character position or range* but is enclosed with the characters "\[" and "]".

```
index = "[" characterPositionOrRange "]"
```

## MARCspec interpretation

[MARCspec interpretation]: #interpretation

Because of the limited expressivity of the MARCspec there must be some kind of implicit interpretation.

* A MARCspec without *subfield tags* or *character position or range* is a reference to all *data elements* of the field.

* For repeatable *fields* the *field tag* without an *index* MUST be interpreted as a reference to the data in all repetitions.

* For repeatable *subfields* the *subfield tag* without an *index* MUST be interpreted as a reference to the data content in all repetitions.

* For the character position or range the digit "0" is always a reference to the first character in the *leader* or *control field*.

* Omitted *indicators* in a MARCspec are interpreted as wildcards for variable field indicators in the MARC record.

# Definition of MARC related terms used in this spec

[Definition of MARC related terms used in this spec]: #definition-of-marc-related-terms-used-in-this-spec

MARCspec does not redefine terms already used by the [Network Development and MARC Standards Office]. MARCspec lists the definition of core terms taken from the [RECORD STRUCTURE] document and adds some for clarification purposes.

| term                          | definition   |
|-------------------------------|--------------|
|**character position or range**|The position or range of positions of characters in *control fields* and *leader*. The first charater has always the position *0*.|
|**content designation**        |The codes and conventions established explicitly by MARC 21 to identify and further characterize the *data elements* within a record and to support the manipulation of that data.|
|**content of field**           |see *field content*|
|**control field**              |A *variable field* containing information useful or required for the processing of the record. Control fields are assigned *tags* beginning with two zeroes. Control fields with fixed length data elements are restricted to ASCII graphics.|
|**data content**               |see *data element value*|
|**data element**               |A defined unit of information|
|**data element identifier**    |A one-character code used to identify individual *data elements* within a *variable field*. The *data element* may be any ASCII lowercase alphabetic, numeric, or graphic symbol except blank.|
|**data element value**         |The value of a *data element* in a *data field*.|
|**data field**                 |A *variable field* containing bibliographic or other data. Data fields are assigned *tags* beginning with characters other than two zeroes. Data fields contain data in any MARC 21 character set unless a field-specific restriction applies.|
|**data in field**              |see *field content*|
|**delimiter**                  |ASCII control character 1F(hex) (represented graphically in MARC 21 documentation as ASCII control character 1F (hex) or $), which is combined with a *data element identifier* to make up the *subfield code* which precedes each individual *data element* within a variable field. The ASCII name for the delimiter is unit separator (US).|
|**field**                      |A defined character string that may contain one or more *data elements*.|
|**field content**              |All *data elements* in a field without *indicators*. See also *field data*.|
|**field data**                 |All *data elements* in a field including *indicators*.|
|**field index**                |see *index*|
|**field tag**                  |see *tag*|
|**fixed field**                |A *field* whose length does not vary. The term is occasionally used to refer to *variable control fields*, especially those that contain coded data such as fields 007 or 008.|
|**index**                      |The count of repeatable *fields* and *subfields*. The first *field* in a list of repeatable *fields* or the first *subfield* in a list of repeatable *subfields* has always the index *0*.|
|**indicator**                  |A *data element* associated with a *data field* that supplies additional information about the field. An indicator may be any ASCII lowercase alphabetic, numeric, or blank. Indicators are not used in *control fields*.|
|**leader**                     |A *fixed field* that occurs at the beginning of each record and provides information for the processing of the record.|
|**set of data**                |A set of *data elements* referenced by a MARCspec.|
|**subfield**                   |A *data element* including its *subfield code*. It's identified by a *data element identifier* within a *variable field*.|
|**subfield code**              |The two-character combination of a *delimiter* followed by a *data element identifier*. Subfield codes are not used in *control fields*.|
|**subfield index**             |see *index*|
|**subfield tag**               |see *data element identifier*|
|**tag**                        |A three character string used to identify or label an associated *field*. The tag may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both).|
|**variable control field**     |see *control field*|
|**variable data field**        |see *data field*|
|**variable field**             |A *field* whose length is determined for each occurrence by the length of data comprising that occurrence. There are two types of variable fields *control fields* and *data fields*.|

# Examples

## Reference to field data

Reference to *field data* of the *leader*.

```
LDR
```

Reference to all *field data* of fields having a field tag starting with *00*.

```
00.
```

Reference to all *field data* of fields having a field tag starting with *7*.

```
7..
```

Reference to *data elements* of all repetions of the "100" field.

```
100
```

Reference to the first "300" field.

```
300[0]
```

Reference to the second of the "300" field.

```
300[1]
```

Reference to the first, second and third of the "300" field.

```
300[0-2]
```

Reference to all but the first of the "300" field.

```
300[1-]
```

Reference to the last of the "300" field.

```
300[-]
```

## Reference to substring

Reference to substring of *field data* in the *leader* from character position 0 to character position 4 (5 characters).

```
LDR/0-4
```

Reference to data in the *leader* at character position 6 (1 character).

```
LDR/6
```

Reference to data in the control field 007 at character position 0 (1 character).

```
007/0
```

Reference to all data but the first character in the control field "007".

```
007/1-
```

Reference to the last character in the control field "007".

```
007/-
```

Reference to the last character of the value of the subfield "a" of field "245".

```
245$a/-
```

## Reference to data content

Reference to value of the subfield "a" of field "245".

```
245$a
```

Reference to value of the subfield "a" of the first "300" field.

```
300[0]$a
```

Reference to the value of the subfields "a", "b" and "c" of field "245".

```
245$a$b$c
```

Same as above, but with the use of a *subfield tag range*.

```
245$a-c
```

Reference to the value of the first subfield "a" of the field "300"

```
300$a[0]
```

Reference to the value of the last subfield "a" of the field "300"

```
300$a[-]
```

Reference to the value of the subfields "_" and "$" of field "300".

```
300$_$$
```

Reference to *data content* in the subfield "a" within the context of *indicator 1* with the value "1".

```
245$a_1

or

245$a_1_

or

245_1$a

or

245_1_$a
```

Reference to the value of the subfield "a" within the context of *indicator 1* with the value "1" and *indicator 2* with the value "0".

```
245$a_10
```

Reference to the value of the subfield "a" within the context of *indicator 2* with the value "0".

```
245$a__0
```

# References

## Normative references

* [RFC 2119]
* [RFC 2234]
* [ISO 2709]

## Informative references

* [MARC 21]
* [RDF]
* [Linked Data]
* [Uniform Resource Identifier (URI)]
* [MARC 21 Principles]
* [marcspec]
* [solrmarc]
* [catmandu]

## Revision history

{GIT_CHANGES}


[MARC]: http://www.loc.gov/marc/
[RDF]: http://www.w3.org/TR/rdf-primer/
[Linked Data]: http://www.w3.org/DesignIssues/LinkedData.html
[Uniform Resource Identifier (URI)]: http://www.ietf.org/rfc/rfc3986.txt
[MARC 21 Principles]: http://www.loc.gov/marc/96principl.html
[marcspec]: https://github.com/billdueber/marcspec
[solrmarc]: https://code.google.com/p/solrmarc/
[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[RFC 2234]: https://www.ietf.org/rfc/rfc2234.txt
[catmandu]: http://librecat.org/
[ISO 2709]: http://en.wikipedia.org/wiki/ISO_2709
[easyM2R]: https://github.com/cKlee/easyM2R
[Network Development and MARC Standards Office]: http://www.loc.gov/marc/ndmso.html
[RECORD STRUCTURE]: http://www.loc.gov/marc/specifications/specrecstruc.html
