# MPilot Language Reference 1.0.0

## Introduction

This reference describes MPilot command file language. It uses a combination of modified BNF notation, plain-text 
descriptions and example code snippets. This reference is intended as a guide for creating MPilot implementations.
Whitespace, including newline and tab characters have no significance in the language. So the following examples are
equivalent:

```
data = EEMSRead(
    InFileName = dataset.nc,
    InFieldName = tmin
)

data=EEMSREAD(InFileName=dataset.nc, InFieldName=tmin)
```

## Language Versions

Language updates are grouped into release and assigned a version number in the form of `<major>.<minor>`.

**Minor** version updates indicate language additions and bug fixes that remain _compatible_ with previous language 
versions. For example: a command file written for MPilot 1.0 should work without modification of transcription using 
MPilot 1.1.

**Major** version updates indicate large, fundamental, and potentially backward-incompatible changes to the language.

Note: "revision" or "patch" versions are reserved for MPilot implementations to use for bug fix or minor updates not 
related to the language.

## Program & Commands

```
program     ::=  commands
commands    ::=  commands command
                 | command
command     ::=  identifier "=" identifier arguments
```

An MPilot program (command file) consists of one or more commands. The following is an example of a command.

```
pecies_endemic_and_listed_fz = CvtToFuzzy(
    InFieldName = species_endemic_listed,
    FalseThreshold = 0,
    TrueThreshold = 6,
    Metadata = [ColorMap:GnBu,DisplayName:High&nbsp;Species&nbsp;Endemic&nbsp;and&nbsp;Listed]
)
```

## Identifier

```
identifier      ::=  id_start
                     | id_start id_continue
id_start        ::=  letter | "_"
id_continue     ::=  letter id_continue
                     | digit id_continue
                     | letter
                     | digit
                     | "_"
letter          ::=  upper_letters | lower_letters
upper_letters   ::=  "A" | ... | "Z"
lower_letters   ::=  "a" | ... | "z"
digit           ::=  "0" | ... | "9"
```

Identifiers are constructed from all upper- and lower-case letters, underscore `_`, and digits. An identifier may not 
begin with a digit. They are used as names for variables and commands.

## Arguments

```
arguments       ::=  "(" argument_list ")"
                     | "(" ")"
argument_list   ::=  argument "," argument_list
                     | argument ","
                     | argument
argument        ::=  identifier "=" expression
```

MPilot commands are called with zero or more arguments, delimited by commas. A command may have zero arguments, in
which case it's called with empty parens `()`.

Each argument includes an identifier (the argument/parameter name) and an expression. For example:

```
FalseThreshold = 0
Weights = [1, 2, 3, 4]
```

## Expressions & Literals

```
expression      ::=  identifier
                     | string
                     | plain_string
                     | identifier expression
                     | number
                     | list
                     | tuple
                     | boolean
boolean         ::=  "True" | "False"
number          ::=  integer 
                     | float
                     | "+" number
                     | "-" number
integer         ::=  digit integer
                     | digit
float           ::=  float_base
                     | float_base float_exp integer
                     | integer float_exp integer
float_base      ::=  | integer "."
                     | integer "." integer
                     | "." integer
float_exp       ::=  "e"
                     | "E"
                     | "e+"
                     | "E+"
                     | "e-"
                     | "E-"
plain_string    ::=  plain_char plain_string
                     | plain_char
plain_char      ::=  <any character, excluding syntax characters: , = - + ( ) [ ] " ' \r \n>
string          ::=  '"' string_chars '"'
                     | "'" string_chars "'"
string_chars    ::=  string_char string_chars
                     | string_char
string_char     ::=  <any character, excluding "\" and the quote character used to delineate the string>
                     | "\" <any character>
```

Expressions consist of variables names, and literal values such as booleans, numbers, identifiers, strings, and lists.
Plain strings are string literals not surrounded by quotes or any other delineating character. Because they lack
delineation, they cannot contain certain syntax-specific characters, which would obfuscate parsing. For example, if a 
plain string were to contain a comma, the parser would not be able to determine whether that comma is part of the 
string, or whether it signifies the delineation between command arguments.

Quoted strings a delineated by either single or double quotes, and contain any character except the backslash `\` or
quote used to delineate the string OR an escape sequence consisting of a backslach `\` followed by any character.

## Lists

```
list        :=  "[" elements "]"
                | "[" "]"
elements    :=  element "," elements
                | element
                | element ","
element     :=  expression
```

Lists contain 0 or more expressions. Trailing commas are allowed, so the following examples are equivalent:

```
[1, 2]

[1, 2,]
```

## Tuples

```
tuple       := "[" tuple_pairs "]"
tuple_pairs := tuple_pair "," tuple_pairs
               | tuple_pair
               | tuple_pair ","
tuple_pair  := tuple_part ":" tuple_part
tuple_part  := string 
            | plain_string 
            | identifier
            | identifier tuple_part

```

Tuples consist of 0 or more key/value pairs, enclosed in `[` brackets `]` and with pairs separated by commas. Both the 
key and value parts are strings or plain strings. Note that there is no rule for an empty tuple. This is because an 
empty tuple is indistinguishable from an empty list. Therefore `[]` parses as an empty list. Because of this, parameter
validation should consider an empty list as a valid value for a tuple parameter.

```
[DisplayName: Agricultural density, Description: This is the ag density layer, ColorMap: binary]

["DisplayName": "Agricultural density", Description: "This is the ag density layer", ColorMap: "binary"]
``` 
