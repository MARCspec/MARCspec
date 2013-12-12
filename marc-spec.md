# MARC spec as string

Along with the shift to [Linked Data] it becomes a common task to map [MARC 21] data to [RDF]. Mappings for MARC 21 are normally based on a set of definitions of MARC fields and subfields called *MARC spec*.

Linked Data is designed to encode the information for exchange in a [Uniform Resource Identifier (URI)]. Encoding MARC specs within URIs enables the exchange of MARC mappings on a global level. But the transport of MARC specs via URIs makes it necessary to encode the spec as a string. This document is a proposal for encoding of MARC spec as string.

## Status of this document

The current version of this proposal is a preliminary draft for open discussion. [Feedback](https://github.com/cklee/marc-spec/issues) is welcome!

**Version**

{VERSION}

**Revision history**

{GIT_CHANGES}



## Terminology

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].


## What is a MARC spec?

Machine-Readable Cataloging (MARC) is a document based key-value exchange format for bibliographic data. The data of a MARC record consits of fields, subfields and its contents. There are three kinds of MARC fields: the Leader, the control field and data field. The data in the Leader and in the control fields can be accessed through its character position or character position range. Only data fields are divided into subfields. Subfields can also be contextualized through indicators. There is an indicator 1 and an indicator 2 for all datafields, both are optional. See [MARC 21 Principles] for a deeper explanation of the MARC 21 format.

A **MARC spec** is a reference to the content data in a MARC record defined through the fields, character positions, subfields and indicators. The content of the MARC record beeing referenced may be represented through a set of data, having zero to n data elements.

## Expressing MARC specs as string

A MARC spec as string might not fullfil all requirements of definition for a reference to the desired set of data. This is because of the nearly unlimited number of options accessing data in a MARC record, like through substrings, regular expressions, rules for repeatable fields and subfields. Thus a MARC spec has to concentrate on the basic references and let all other data processing to subsequent data processing functions. Other MARC spec definitions like used in [marcspec] and [solrmarc] might be more expressive on the level of data processing.

### Basic references

A MARC spec as string should cope the following basic references:

* Reference to data in the leader and control fields plus character position and range
* Reference to data in data fields or subfields
* Reference to data in subfields of data fields within the context of indicators

References a MARC spec as string does not cope

* Reference to data through a field list
* Reference to data in subfields internally structured through delimiters or key characters 

### Form of MARC spec as string

The **Augmented BNF for Syntax Specifications: ABNF** [RFC 2234] is used to define the form of the MARC spec as string.

A MARC spec as string consits of a field tag optionally followed by a character position or range prefixed with the character "~" or followed by zero to n subfield tags optionally followed by the indicators prefix with the character "\_".

```
marcspec = (fieldTag ["~" characterPositionOrRange] / fieldTag subfieldTags ["_" indicators])
```

A field tag consits of three characters. Within MARC 21 there are only digits allowed (except for the Leader wich is "LDR"), but using the character "X" instead of a digit this must be interpreted as a wildcard. E.g. 2XX is then a reference to the data in all fields beginning with 2.

```
fieldTag = [3*3(DIGIT / "X") / "LDR"]
```

A character position or range consits of one to n digits represententing the character position optionally followed by one to n digits prefixed with the character "-" representing the character range.

```
characterPositionOrRange = *DIGIT ["-" *DIGIT]
```

Subfield tags consits of one to n characters.

```
subfieldTags = *CHAR
```

Indicators consits of indicator 1 and indicator 2. Both are optional. If indicator 1 is not specified is must be replaced by the character "\_". If indicator 2 is not specified it might be replaced by the character "\_" or left blank.

```
indicators = (indicator1 / "_") ([indicator2] / "_")
```

Both indicators 1 and 2 consits of one character or one digit if present.

```
indicator1 = indicator
indicator2 = indicator
indicator = 1*1(CHAR / DIGIT)
```

## Examples

### Reference to data in the leader and control fields plus character position and range

Reference to all data in the Leader.

```
LDR
```

Reference to data in the Leader from character position 0 to character position 4 (5 characters).

```
LDR~0-4
```

Reference to data in the Leader at character position 6 (1 character).

```
LDR~6
```

Reference to data in the control field 007 (see [MARC spec as string interpretation](#interpretation) for further explanation) at character position 0 (1 character).

```
007~0
```

### Reference to data in data fields or subfields

Reference to all content data in the field "245".

```
245
```

Reference to data in the subfield "a" of field "245".

```
245a
```

Reference to data in the subfields "a", "b" and "c" of field "245".

```
245abc
```

Reference to data in the subfield "a" of all fields beginning with "24".

```
24Xa
```

Reference to data in the subfield "a" of all fields beginning with "2" and ending with "5".

```
2X5a
```

Reference to data in the subfield "a" of all fields beginning with "2".

```
2XXa
```

### Reference to data in subfields of data fields within the context of indicators

Reference to data in the subfield "a" within the context of indicator 1 with the value 1.

```
245a_1

or

245a_1_
```

Reference to data in the subfield "a" within the context of indicator 1 with the value 1 and indicator 2 with the value 0.

```
245a_10
```

Reference to data in the subfield "a" within the context of indicator 2 with the value 0.

```
245a__0
```

## MARC spec as string interpretation

[MARC spec as string interpretation]: #interpretation

Because of the limited expressivity of the MARC spec as string there must be some kind of implicit interpretation. Therefor this section is normative.

A Marc spec with only the field tag is a reference to all content data in the field. That does NOT include the subfield tags.

For repeatable fields the field tag MUST be interpreted as a reference to the data in all repetitions.

For the character position or range the digit "0" is always a reference to the first character in the Leader or control field.

Subfield tags may occour in in non ordered sequence. 


## Normative references

* [RFC 2119]
* [RFC 2234]

## Informative references

* [MARC 21]
* [RDF]
* [Linked Data]
* [Uniform Resource Identifier (URI)]
* [MARC 21 Principles]
* [marcspec]
* [solrmarc]



[MARC 21]: http://www.loc.gov/marc/
[RDF]: http://www.w3.org/TR/rdf-primer/
[Linked Data]: http://www.w3.org/DesignIssues/LinkedData.html
[Uniform Resource Identifier (URI)]: http://www.ietf.org/rfc/rfc3986.txt
[MARC 21 Principles]: http://www.loc.gov/marc/96principl.html
[marcspec]: https://github.com/billdueber/marcspec
[solrmarc]: https://code.google.com/p/solrmarc/
[RFC 2119]: http://www.ietf.org/rfc/rfc2119.txt
[RFC 2234]: https://www.ietf.org/rfc/rfc2234.txt
