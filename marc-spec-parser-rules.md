# Rules for MARCspec parsers

The following rules can help building MARCspec parsers. Rules with the keywords ***MUST** and ***MUST NOT** must always be *true*, otherwise it is not a valid MARCspec and some exception should be thrown.

1. The string MUST be of 3 characters at minimum.
2. The string MUST NOT have whitespaces.
3. Assuming the first three characters of the string is the field tag
  1. The first three characters of the string MUST be valid for the regular expression /[.0-9a-z]{3,3}|[.0-9A-Z]{3,3}/.
  2. If 3.1. is valid, the first three characters of the string is the field tag.
4. Continue if string has more than 3 characters.
5. Check if the fourth character is '['. If it's not, continue with 6.
  1. The character ']' MUST be found in the string.
  2. Assuming the string between the character '[' and the first character ']' is the field index.
  3. The index MUST NOT be empty.
  4. Each character of the index MUST be valid of the regular expression /[0-9-]/.
  5. The character '-' MUST NOT be the first character of the index.
  6. The index MUST have only one '-' character.
  7. Check if character '-' is present. If not, continue with 5.vii.b.
    1. Assuming index range.
      1. Index range MUST be two characters minimum.
      2. Check if one or more digits follow the character '-'. If not, continue with 5.vii.a.c.
        1. The digits behind the character '-' is the index range end.
      3. The index range has no end. 
    2. Assuming specific index. The digit(s) is the index' number.
6. Now work with the substring after the field tag and the optionally index (now called data content ref or short dcr).
  1. Check if first character of dcr is '/'. If not continue with 7.
  2. Assuming character position or range after character '/'.
    1. Proceed with steps 5.iii. to 5.vii.b. by exchanging the keyword 'index' with 'character position'.
7. Check if character '\_' is present in dcr and that it's not preceded by the character '$'. If not (meaning there is no indicators spec), continue with 9.
8. Assuming indicators spec in dcr.
  1. Check if first character of dcr is '\_'. If not continue with 8.i.b.
    1. The substring after the first character '\_' and (if present) before the first character '$' is the assumed indicators spec. Continue with 8.ii.
    2. The substring after the last character '\_' till the end of dcr is the assumed indicators spec.  
  2. The indicators spec MUST be one character at minimum and two characters at maximum. Each character in the indicators spec MUST be valid for the regular expression /[a-z0-9\_]/.
    1. If the first character is not '\_', it is the indicator 1.
    2. If the second character is present and it is not '\_', it is the indicator 2.
    3. Continue with 10.
9. Only continue if the first character of dcr is '$'. 
10. The assumed subfield spec is either the substring after the indicator spec (if present) or the whole dcr.
  1. Subfields MUST NOT appear for leader or control fields.
  2. Each character of subfield spec MUST be valid for the regular expression /[!\"#$%&'()*+,-.\/0-9:;<=>?[\\]^\_`a-z{}~]/.
  3. Each subfield tag or subfield tag range MUST be prefixed with the character '$'.
  4. For each subfield tag or subfield tag range continue.
    1. Only a subfield tag not a subfield tag range can have an index. If an index is present continue with 5.ii. until 5.vii.b. by exchanging the keyword 'field' with 'subfield'. The character before the index is the subfield tag. End here. 
    2. If the second character is '-', assuming subfield range.
      1. Subfield range MUST be 3 characters.
      2. The first and the third character MUST both either be alphabetic lower case, alphabetic upper case or digits.
      3. All characters in that range are the subfield tags. End here. 
    3. If no subfield index or subfield range is given and there is only one character given, then this is the subfield tag. End here.
