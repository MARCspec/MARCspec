# MARCspec as string

Along with the shift to [Linked Data] it becomes a common task to map [MARC 21] data to [RDF]. Mappings for MARC 21 are normally based on a set of definitions of MARC fields and subfields called *MARC spec*.

Linked Data is designed to encode the information for exchange in a [Uniform Resource Identifier (URI)]. Encoding MARCspecs within URIs enables the exchange of MARC mappings on a global level. But the transport of MARCspecs via URIs makes it necessary to encode the spec as a string. This document is a proposal for encoding of MARCspec as string.

There are already implementations of MARC field specifications in tools like [marcspec], [catmandu], [solrmarc] and [easyM2R]. MARCspec can help to build reusable MARCspec parsers and validators.

## Status of this document

The current version of this proposal is a preliminary draft for open discussion. [Feedback](https://github.com/cklee/marc-spec/issues) is welcome!

## Terminology

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119].

See also [Definition of MARC related terms used in this spec].

## What is a MARCspec?

Machine-Readable Cataloguing (MARC) is a document based key-value exchange format for bibliographic data. The data of a MARC record consists of fields, subfields and its contents. There are three kinds of MARC fields: the *leader*, the control field and data field. The data content in the *leader* and in the control fields can be accessed through its character position or character position range. Only data fields are divided into subfields. Subfields can also be contextualized through indicators. There is an indicator 1 and an indicator 2 for all data fields, both are optional. See [MARC 21 Principles] for a deeper explanation of the MARC 21 format.

A **MARCspec** is a reference to the content data in a MARC record defined through the *fields*, *character positions*, *subfields* and *indicators*. The *data elements* of the MARC record being referenced may be represented through a *set of data*, having zero to n *data elements*. Neither MARCspec defines the form of this *set of data*, nor the encoding of the referenced *data content*.  

## Expressing MARCspecs as string

A MARCspec as string might not fulfil all requirements of definition for a reference to the desired set of data like XPath does for XML. This is because of the nearly unlimited number of options accessing data in a MARC record, like through substrings, regular expressions, rules for repeatable fields and subfields. Thus a MARCspec has to concentrate on the basic references and let all other data processing to subsequent data processing functions of tools implemented MARCspec.

### Basic references

A MARCspec as string should cope the following basic references:

* Reference to *data elements* in *fixed fields* by character position and range
* Reference to *data elements* in *varibale fields* without or within the context of *indicators*
* Reference to *data content* in *variable fields* by *subfield tags*
* Reference to *data elements* by an *field index*
* Reference to *data content* by an *subfield index*

References a MARCspec as string does not cope

* Reference to substrings of *data content*
* Reference to *data content* wthin the context of other *data content*
* Reference to *data elements* (entries) in the MARC record directory

### Form of MARCspec as string

**This section is normative.**

The **Augmented BNF for Syntax Specifications: ABNF** [RFC 2234] is used to define the form of the MARCspec as string.

A MARCspec as string consists of a *field tag* optionally followed by a character position or range prefixed with the character "~" or followed by zero to n *subfield tags* optionally followed by the *indicators* prefixed with the character "\_".

```
marcspec = fieldTag (["~" characterPositionOrRange] / [subfieldTags] ["_" indicators])
```

A *field tag* consists of three characters. Within MARC 21 there are only digits allowed (except for the *leader* which is "LDR"), but using the character "X" instead of a digit this must be interpreted as a wildcard. E.g. 2XX is then a reference to the *data elements* in all *fields* beginning with *2*.

```
fieldTag = 3*3(DIGIT / "X") / "LDR"
```

A *character position or range* consists of one to n digits representing the character position optionally followed by one to n digits prefixed with the character "-" representing the character range.

```
characterPositionOrRange = *DIGIT ["-" *DIGIT]
```

*Subfield tags* consist of one to n lowercase alphabetic, digits or special characters. In a MARCspec *subfield tags* may occur in non ordered sequence.

```
alphalower = %x61-7A   ; a-z
specialCharacter =  %x21-2F / %x3A-3F  ; ! " # $ % & ' ( ) * + ' - . / : ; < = > ?
subfieldTags = 1*(alphalower / DIGIT / specialCharacter)
```

*Indicators* consist of *indicator 1* and *indicator 2*. Both are optional. If *indicator 1* is not specified is must be replaced by the character "\_". If *indicator 2* is not specified it might be replaced by the character "\_" or left blank.

```
indicators = (indicator1 / "_") [ indicator2 / "_" ]
```

If present both *indicator 1* and *indicator 2* consists of one lowercase alphabetic or numeric character.

```
indicator  = 1*1(alphalower / DIGIT)
indicator1 = indicator
indicator2 = indicator
```

## MARCspec as string interpretation

[MARCspec as string interpretation]: #interpretation

**This section is normative.**

Because of the limited expressivity of the MARCspec as string there must be some kind of implicit interpretation.

* A MARCspec without *subfield tags* or *character position or range* is a reference to the *field content*.

* For repeatable *fields* the *field tag* without an *index* MUST be interpreted as a reference to the data in all repetitions.

* For repeatable *subfields* the *subfield tag* without an *index* MUST be interpreted as a reference to the data content in all repetitions.

* For the character position or range the digit "0" is always a reference to the first character in the *leader* or *control field*.

* Omitted *indicators* in a MARCspec are interpreted as wildcards for variable field indicators in the MARC record.

## Definition of MARC related terms used in this spec

[Definition of MARC related terms used in this spec]: #terms

MARCspec does not redefine terms already used by the [Network Development and MARC Standards Office]. MARCspec lists the definition of core terms taken from the [RECORD STRUCTURE] document and adds some for clarification purposes.

**character position or range**

The position or range of positions of characters in *control fields* and *leader*. The first charater has always the position *0*.

**content designation**

The codes and conventions established explicitly by MARC 21 to identify and further characterize the *data elements* within a record and to support the manipulation of that data.

**content of field**

see *field content*

**control field**

A *variable field* containing information useful or required for the processing of the record. Control fields are assigned *tags* beginning with two zeroes. Control fields with fixed length data elements are restricted to ASCII graphics.

**data content**

A *data element* in a *data field* excluding *subfield codes* and *indicators*.

**data element**

A defined unit of information.

**data element identifier**

A one-character code used to identify individual *data elements* within a *variable field*. The *data element* may be any ASCII lowercase alphabetic, numeric, or graphic symbol except blank.

**data field**

A *variable field* containing bibliographic or other data. Data fields are assigned *tags* beginning with characters other than two zeroes. Data fields contain data in any MARC 21 character set unless a field-specific restriction applies.

**data in field**

see *field content*

**delimiter**

ASCII control character 1F(hex) (represented graphically in MARC 21 documentation as ASCII control character 1F (hex) or $), which is combined with a *data element identifier* to make up the *subfield code* which precedes each individual *data element* within a variable field. The ASCII name for the delimiter is unit separator (US).

**field**

A defined character string that may contain one or more *data elements*.

**field content**

All *data elements* in a field without *indicators*. See also *field data*.

**field data**

All *data elements* in a field including *indicators*.

**field index**

see *index*

**field tag**

see *tag*

**fixed field**

A *field* whose length does not vary. The term is occasionally used to refer to *variable control fields*, especially those that contain coded data such as fields 007 or 008.

**index**

The count of repeatable *fields* and *subfields*. The first *field* in a list of repeatable *fields* or the first *subfield* in a list of repeatable *subfields* has always the index *0*.

**indicator**

A *data element* associated with a *data field* that supplies additional information about the field. An indicator may be any ASCII lowercase alphabetic, numeric, or blank. Indicators are not used in *control fields*.

**leader**

A *fixed field* that occurs at the beginning of each record and provides information for the processing of the record.

**set of data**

A set of *data elements* referenced by a MARCspec. 

**subfield**

A *data element* including its *subfield code*. It's identified by a *data element identifier* within a *variable field*.

**subfield code**

The two-character combination of a *delimiter* followed by a *data element identifier*. Subfield codes are not used in *control fields*.

**subfield index**

see *index*

**subfield tag**

see *data element identifier*

**tag**

A three character string used to identify or label an associated *variable field*. The tag may consist of ASCII numeric characters (decimal integers 0-9) and/or ASCII alphabetic characters (uppercase or lowercase, but not both).

**variable control field**

see *control field*

**variable data field**

see *data field*

**variable field**

A *field* whose length is determined for each occurrence by the length of data comprising that occurrence. There are two types of variable fields *control fields* and *data fields*.

## Examples

### Reference to data in the leader and control fields plus character position and range

Reference to all data in the *leader*.

```
LDR
```

Reference to data in the *leader* from character position 0 to character position 4 (5 characters).

```
LDR~0-4
```

Reference to data in the *leader* at character position 6 (1 character).

```
LDR~6
```

Reference to data in the control field 007 (see [MARCspec as string interpretation](#interpretation) for further explanation) at character position 0 (1 character).

```
007~0
```

Reference to all control fields.

```
00X
```

### Reference to data in data fields or subfields

Reference to the *field content* in the field "245".

```
245
```

Reference to *data content* in the subfield "a" of field "245".

```
245a
```

Reference to *data content* in the subfields "a", "b" and "c" of field "245".

```
245abc
```

Reference to *data content* in the subfield "a" of all fields beginning with "24".

```
24Xa
```

Reference to *data content* in the subfield "a" of all fields beginning with "2" and ending with "5".

```
2X5a
```

Reference to *data content* in the subfield "a" of all fields beginning with "2".

```
2XXa
```

### Reference to data in subfields of data fields within the context of indicators

Reference to *data content* in the subfield "a" within the context of *indicator 1* with the value *1*.

```
245a_1

or

245a_1_
```

Reference to *data content* in the subfield "a" within the context of *indicator 1* with the value *1* and *indicator 2* with the value *0*.

```
245a_10
```

Reference to *data content* in the subfield "a" within the context of *indicator 2* with the value *0*.

```
245a__0
```

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


[MARC 21]: http://www.loc.gov/marc/
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