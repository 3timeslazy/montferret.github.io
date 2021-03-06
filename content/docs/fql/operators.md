---
title: "Operators"
weight: 3
draft: false
---

# Operators
FQL supports a number of operators that can be used in expressions. There are comparison, logical, arithmetic, and the ternary operator.

## Comparison operators
Comparison (or relational) operators compare two operands. They can be used with any input data types, and will return a boolean result value.

The following comparison operators are supported:

- ``==`` equality
- ``!=`` inequality
- ``<`` less than
- ``<=`` less or equal
- ``>`` greater than
- ``>=`` greater or equal
- ``IN`` test if a value is contained in an array
- ``NOT IN`` test if a value is not contained in an array
- ``LIKE`` tests if a string value matches a pattern
- ``=~`` tests if a string value matches a regular expression
- ``!~`` tests if a string value does not match a regular expression

Each of the comparison operators returns a boolean value if the comparison can be evaluated and returns true if the comparison evaluates to true, and false otherwise.

The comparison operators accept any data types for the first and second operands. However, ``IN`` and ``NOT IN`` will only return a meaningful result if their right-hand operand is an array, and ``LIKE`` will only execute if both operands are string values. The comparison operators will not perform any implicit type casts if the compared operands have different or non-sensible types.

Some examples for comparison operations in FQL:

{{< code fql >}}
0 == NONE                 // false
1 > 0                     // true
true != NONE              // true
45 <= "yikes!"            // true
65 != "65"                // true
65 == 65                  // true
1.23 > 1.32               // false
1.5 IN [ 2, 3, 1.5 ]      // true
"foo" IN NONE             // false
42 NOT IN [ 17, 40, 50 ]  // true
"abc" == "abc"            // true
"abc" == "ABC"            // false
"foo" LIKE "f%"           // true
"foo" =~ "^f[o].$"        // true
"foo" !~ "[a-z]+bar$"     // true
{{</ code >}}

The ``LIKE`` operator checks whether its left operand matches the pattern specified in its right operand. The pattern can consist of regular characters and wildcards. The supported wildcards are _ to match a single arbitrary character, and % to match any number of arbitrary characters. Literal % and _ need to be escaped with a backslash. Backslashes need to be escaped themselves, which effectively means that two reverse solidus characters need to preceed a literal percent sign or underscore. In arangosh, additional escaping is required, making it four backslashes in total preceeding the to-be-escaped character.

{{< code fql >}}
"abc" LIKE "a%"              // true
"abc" LIKE "_bc"             // true
"a_b_foo" LIKE "a\\_b\\_foo" // true
{{</ code >}}

## Array comparison operators
The comparison operators also exist as array variant. In the array variant, the operator is prefixed with one of the keywords ``ALL``, ``ANY`` or ``NONE``. Using one of these keywords changes the operator behavior to execute the comparison operation for all, any, or none of its left hand argument values. It is therefore expected that the left hand argument of an array operator is an array.

{{< code fql >}}
[ 1, 2, 3 ] ALL IN [ 2, 3, 4 ]   // false
[ 1, 2, 3 ] ALL IN [ 1, 2, 3 ]   // true
[ 1, 2, 3 ] NONE IN [ 3 ]        // false
[ 1, 2, 3 ] NONE IN [ 23, 42 ]   // true
[ 1, 2, 3 ] ANY IN [ 4, 5, 6 ]   // false
[ 1, 2, 3 ] ANY IN [ 1, 42 ]     // true
[ 1, 2, 3 ] ANY == 2             // true
[ 1, 2, 3 ] ANY == 4             // false
[ 1, 2, 3 ] ANY > 0              // true
[ 1, 2, 3 ] ANY <= 1             // true
[ 1, 2, 3 ] NONE < 99            // false
[ 1, 2, 3 ] NONE > 10            // true
[ 1, 2, 3 ] ALL > 2              // false
[ 1, 2, 3 ] ALL > 0              // true
[ 1, 2, 3 ] ALL >= 3             // false
["foo", "bar"] ALL != "moo"      // true
["foo", "bar"] NONE == "bar"     // false
["foo", "bar"] ANY == "foo"      // true
{{</ code >}}

## Logical operators
The following logical operators are supported in FQL:

- ``&&`` logical and operator
- ``||`` logical or operator
- ``!`` logical not/negation operator

FQL also supports the following alternative forms for the logical operators:

- ``AND`` logical and operator
- ``OR`` logical or operator
- ``NOT`` logical not/negation operator

The alternative forms are aliases and functionally equivalent to the regular operators.

The two-operand logical operators in FQL will be executed with short-circuit evaluation (except if one of the operands is or includes a subquery. In this case the subquery will be pulled out an evaluated before the logical operator).

The result of the logical operators in FQL is defined as follows:

- ``lhs && rhs`` will return ``lhs`` if it is ``false`` or would be ``false`` when converted into a boolean. If ``lhs`` is ``true`` or would be ``true`` when converted to a boolean, ``rhs`` will be returned.
- ``lhs || rhs`` will return ``lhs`` if it is ``true`` or would be ``true`` when converted into a boolean. If ``lhs`` is ``false`` or would be ``false`` when converted to a boolean, ``rhs`` will be returned.
- ``! value`` will return the negated value of value converted into a boolean

{{< code fql >}}
u.age > 15 && u.address.city != ""
true || false
NOT u.isInvalid
1 || ! 0
{{</ code >}}

Passing non-boolean values to a logical operator is allowed. Any non-boolean operands will be casted to boolean implicitly by the operator, without making the query abort.

The conversion to a boolean value works as follows:

- ``NONE`` will be converted to ``false``
- boolean values remain unchanged
- all numbers unequal to zero are ``true``, zero is ``false``
- an empty string is ``false``, all other strings are ``true``
- arrays, objects, binary and custom types are ``true``, regardless of their contents

The result of *logical and* and *logical or* operations can now have any data type and is not necessarily a boolean value.

For example, the following logical operations will return boolean values:

{{< code fql >}}
25 > 1 && 42 != 7                          // true
22 IN [ 23, 42 ] || 23 NOT IN [ 22, 7 ]    // true
25 != 25                                   // false
{{</ code >}}

whereas the following logical operations will not return boolean values:

{{< code fql >}}
1 || 7                                     // 1
NONE || "foo"                              // "foo"
NONE && true                               // NONE
true && 23                                 // 23
{{</ code >}}

## Arithmetic operators
Arithmetic operators perform an arithmetic operation on two numeric operands. The result of an arithmetic operation is again a numeric value.

FQL supports the following arithmetic operators:

- ``+`` addition
- ``-`` subtraction
- ``*`` multiplication
- ``/`` division
- ``%`` modulus

Unary plus and unary minus are supported as well:

{{< code fql >}}
LET x = -5
LET y = 1
RETURN [-x, +y]
// [5, 1]
{{</ code >}}

For exponentiation, there is a numeric function POW(). The syntax base ** exp is not supported.

Some example arithmetic operations:

{{< code fql >}}
1 + 1
33 - 99
12.4 * 4.5
13.0 / 0.1
23 % 7
-15
+9.99
{{</ code >}}

The arithmetic operators accept operands of any type. Passing non-numeric values to an arithmetic operator will cast the operands to numbers:

- ``NONE`` will be converted to ``0``
- ``false`` will be converted to ``0``, ``true`` will be converted to ``1``
- a valid numeric value remains unchanged, but ``NaN`` and Infinity will be converted to ``0``
- string values are converted to a number if they contain a valid string representation of a number. Any whitespace at the start or the end of the string is ignored. Strings with any other contents are converted to the number ``0``
- an empty array is converted to ``0``, an array with one member is converted to the numeric representation of its sole member. Arrays with more members are converted to the number ``0``.
- objects, binary and custom types are converted to the number ``0``.

Here are a few examples:

{{< code fql >}}
1 + "a"                 // 1
1 + "99"                // 100
1 + NONE                // 1
NONE + 1                // 1
3 + [ ]                 // 3
24 + [ 2 ]              // 26
24 + [ 2, 4 ]           // 0
25 - NONE               // 25
17 - true               // 16
23 * { }                // 0
5 * [ 7 ]               // 35
24 / "12"               // 2
1 / 0                   // 0
{{</ code >}}

## Ternary operator
FQL also supports a ternary operator that can be used for conditional evaluation. The ternary operator expects a boolean condition as its first operand, and it returns the result of the second operand if the condition evaluates to true, and the third operand otherwise.

{{< code fql >}}
u.age > 15 || u.active == true ? u.userId : null
{{</ code >}}

There is also a shortcut variant of the ternary operator with just two operands. This variant can be used when the expression for the boolean condition and the return value should be the same:

{{< code fql >}}
u.value ? : 'value is null, 0 or not present'
{{</ code >}}

## Range operator
FQL supports expressing simple numeric ranges with the ``..`` operator. This operator can be used to easily iterate over a sequence of numeric values.

The ``..`` operator will produce an array of the integer values in the defined range, with both bounding values included.

{{< code fql >}}
2010..2013
{{</ code >}}

will produce the following result:

{{< code fql >}}
[ 2010, 2011, 2012, 2013 ]
{{</ code >}}

## Operator precedence
The operator precedence in FQL is similar as in other familiar languages (lowest precedence first):
- ``? :`` ternary operator
- ``||`` logical or
- ``&&`` logical and
- ``==``, ``!=`` equality and inequality
- ``IN`` in operator
- ``<``, ``<=``, ``>=``, ``>`` less than, less equal, greater equal, greater than
- ``+``, ``-`` addition, subtraction
- ``*``, ``/``, ``%`` multiplication, division, modulus
- ``!``, ``+``, ``-`` logical negation, unary plus, unary minus
- ``()`` function call
- ``.`` member access
- ``[]`` indexed value access

The parentheses ``(`` and ``)`` can be used to enforce a different operator evaluation order.


