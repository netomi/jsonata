---
id: version-2.0.0-regex
title: Using Regular Expressions
sidebar_label: Regular Expressions
original_id: regex
---

The regular expression is a syntax for matching and extracting parts of a string.  JSONata provides first class support for regular expressions surrounded by the familiar slash delimeters found in many scripting languages.  

`/regex/flags`

where:
- `regex` - the regular expression
- `flags` - optionally either or both of: 
    - `i` - ignore case
    - `m` - multiline match

 ## Functions which use regular expressions

A number of functions are available that take a regular expression as a parameter

- [$match()](string-functions#match)
- [$contains()](string-functions#contains)
- [$split()](string-functions#split)
- [$replace()](string-functions#replace)

__Examples__

 ## Regular expressions in query predicates

Regexes are often used in query predicates (filter expressions) when selecting objects that contain a matching string property.  For this, a short cut notation can be used as follows:

`path.to.object[stringProperty ~> /regex/]`

The `~>` is the [chain operator](control-operators#chain), and its use here implies that the result of `/regex/` is a function.  We'll see below that this is in fact the case.

__Examples__

``Account.Order.Product[`Product Name` ~> /hat/i ]``

will match all products that have 'hat' in their name.

 ## Generic matchers

The JSONata type system is based on the JSON type system, with the addition of the function type.  In order to accommodate the regex as a standalone expression, the syntax `/regex/` evaluates to a function.  Think of the `/regex/` syntax as a higher-order function that results in a 'matching function' when evaluated.  The `regex` between the slashes is the parameter to this HOF and the function that gets returned, when applied to _its_ string parameter, will return a structure that contains details of parts of the string that have been matched.  If nothing is matched, then it returns the empty sequence (i.e. JavaScript `undefined`).

__Example__

`$matcher := /[a-z]*an[a-z]*/i`

Evaluation of the regex returns a function, and this has been bound to a variable `$matcher`.  Later, the `$matcher` function is invoked on a string:

`$matcher('A man, a plan, a canal, Panama!')`

This returns the following JSONata object (JSON, but also with a function property):

```
{
  "match": "man",
  "start": 2,
  "end": 4,
  "groups": [],
  "next": "<native function>#0"
}
```

This contains information of the first matching substring within this famous palindrome, specifically:
- `match` - the substring within the original string that matches the regex
- `start` - the starting position (zero offset) of the matching substring within the original string
- `end` - the endinging position of the matching substring within the original string
- `groups` - if capturing groups are used in the regex, then this array contains a string for the text captured by each group
- `next()` - when invoked, will return details of the second occurrence of any matching substring (and so on).

In this example, invoking `next()` will return:

```
{
  "match": "canal",
  "start": 17,
  "end": 22,
  "groups": [],
  "next": "<native function>#0"
}
```

and so on, until it eventually returns the empty sequence.

 ## Writing a custom matcher

We've learned that the regex syntax is just a function generator, and the signature and return structure of the generated 'matcher' function is well defined.  The four regex-aware functions (`$match`, `$contain`, `$split`, `$replace`) simply invoke this function as part of their implementation.  Apart from that, they have no awareness that these matcher functions were generated by the regex syntax.

So it's possible to write any user-defined matcher function, provided it conforms to this contract.  This can be done as a JSONata lambda function or (more likely) as an extension function.  It can then be passed to these four 'regex-aware' functions and they will search using the custom matcher rather than a regex.