# Strings

## Overview

Like many languages, V has built-in support for strings.
Like Go, V uses
[UTF-8](https://en.wikipedia.org/wiki/UTF-8)
to encode strings.
All strings in V are an immutable byte array.

Strings in V are specified using single or double quotes.
Both options are equivalent; however, it is recommended to use single quotes for
strings that do not contain single quotes.

That is, as long as the string does not contain other single quotes,
it is better to use single quotes:

```v play
println('Just string')
```

But if the string contains single quotes, then it is better to use double quotes to
avoid having to escape the single quotes:

```v play
println("String with 'single quotes'")
```

[vfmt](../../tools/builtin-tools.md#v-fmt) will automatically convert double quotes to single quotes
when possible.

Escaping in strings is supported as in C:

```v nofmt play
println('\r\n'.len) // 2
```

Since strings are stored in UTF-8, any characters can be used, including emoji:

```v play
println('🌎') // 🌎
```

Arbitrary bytes can be directly specified using `\x##` notation where `#` is a hex digit:

```v play
println('\xc0'[0]) // u8(0xc0)
```

Or using octal escape `\###` notation where `#` is an octal digit:

```v play
println('\141ardvark') == 'aardvark' // true
```

Unicode can be specified directly as `\u####` where # is a hex digit and will be converted
internally
to its UTF-8 representation:

```v play
println('\u2605') // ★
println('\xe2\x98\x85') // ★
```

Since all characters in V are stored in UTF-8, the length of the string may differ from the number
of visible characters
in it:

```v play
s := 'hello 🌎' // emoji takes 4 bytes
println(s.len) // 10
```

String values are immutable.
You cannot mutate elements:

```v failcompile play
mut s := 'hello 🌎'
s[0] = `H`
//   ^ error: cannot assign to s[i] since V strings are immutable
// (note, that variables may be mutable but string values are always
//  immutable, like in Go and Java)
```

Note that indexing a string will produce a `byte`, not a `rune` nor another `string`.
Indexes correspond to _bytes_ in the string, not Unicode code points.
If you want to convert the `byte` to a `string`, use the `.ascii_str()` method on the `byte`:

```v play
country := 'Netherlands'
println(country[0]) // 78
println(country[0].ascii_str()) // N
```

## String conversion

### to `int` or other numeric types

To get an int from a string, use the `.int()` or `.<type-name>()` method:

```v play
s := '42'
println(s.int()) // 42

println('100'.u8()) // 100
println('100'.i16()) // 100
```

All int literals are supported:

```v play
println('0xc3'.int()) // 195
println('0o10'.int()) // 8
println('0b1111_0000_1010'.int()) // 3850
println('-0b1111_0000_1010'.int()) // -3850
```

### to `f32` or `f64`

To get a float from a string, use the `.f32()` or `.f64()` method:

```v play
println('3.14'.f32()) // 3.14
println('3.14'.f64()) // 3.14
```

### to `bool`

To get a bool from a string, use the `.bool()` method:

```v play
println('true'.bool()) // true
println('false'.bool()) // false
```

### to `[]u8`

To get an array of bytes from a string, use the `.bytes()` method:

```v failcompile
arr := s.bytes()
println(arr.len) // 10
```

### from `[]u8`

To turn an array of bytes into a string, use the `.bytestr()` method:

```v failcompile
s2 := arr.bytestr()
```

### to `[]rune`

To get an array of runes from a string, use the `.runes()` method:

```v play
arr := 'hello 🌎'.runes()
println(arr.len) // 7
```

### from `[]rune`

To turn an array of runes into a string, use the `.string()` method:

```v play
arr := 'hello 🌎'.runes()
s2 := arr.string()
println(s2) // hello 🌎
```

For more advanced `string` processing and conversions, refer to the
[`vlib/strconv`](https://modules.vosca.dev/standard_library/strconv.html)
module.

## Raw strings

V also supports "raw" strings, which do not handle escaping and do not support string interpolation.

For raw strings, prepend string literal with `r`:

```v play
s := r'hello\nworld' // the `\n` will be preserved as two characters
println(s) // hello\nworld
```

## C strings

V also supports C strings, which are null-terminated arrays of bytes.

For raw strings, prepend string literal with `c`:

```v play
s := c'hello\nworld'
println(typeof(s)) // &u8
println(s) // &104
```

## String interpolation

Basic interpolation syntax is pretty simple – use `${` before a variable name and `}` after.
The variable will be converted to a string and embedded into the literal:

```v play
name := 'Bob'
println('Hello, ${name}!') // Hello, Bob!
```

It also works with fields:

```v failcompile
println('age = ${user.age}')
```

And with complex expressions:

```v failcompile
println('can register = ${user.age > 13}')
```

### Format specifiers

To control the output format, you can use format specifiers like those used in C in `printf()`.
Format specifiers are optional and specify the output format.
The compiler takes care of the storage size, so there is no `hd` or `llu`.

To use a format specifier, follow this pattern:

```text
${varname:[flags][width][.precision][type]}
```

#### Flags

Zero or more of the following:

- `-` – left-align output within the field. The default is to right-align.
- `0` – use `0` as the padding character instead of the default `space` character.

> **Note**
> V does not currently support the use of `'` or `#` as format flags, and V supports but
> doesn't need `+` to right-align since that's the default.

#### Width

Integer value describing the minimum width of total field to output.

#### Precision

Integer value preceded by a `.` will guarantee that many digits after the decimal
point, if the input variable is a float.

Ignored if variable is an integer.

#### Type

- `f` and `F` specify the input is a float and should be rendered as such.
- `e` and `E` specify the input is a float and should be rendered as an exponent (partially broken).
- `g` and `G` specify the input is a float--the renderer will use floating point notation for small
  values and exponent
  notation for large values.
- `d` specifies the input is an integer and should be rendered in base-10 digits.
- `x` and `X` require an integer and will render it as hexadecimal digits.
- `o` requires an integer and will render it as octal digits.
- `b` requires an integer and will render it as binary digits.
- `s` requires a string (almost never used).

> **Note**
> When a numeric type can render alphabetic characters, such as hex strings or special values
> like `infinity`, the lowercase version of the type forces lowercase alphabetic and the
> uppercase version forces uppercase alphabetic.

In most cases, it is best to leave the format type empty.
Floats will be rendered by default as `g`, integers will be rendered
by default as `d`, and `s` is almost always redundant.
There are only three cases where specifying a type is recommended:

- format strings are parsed at compile time, so specifying a type can help detect errors then
- format strings default to using lowercase letters for hex digits and the `e` in exponents.
  Use an uppercase type to force the use of uppercase hex digits and an uppercase `E` in exponents.
- format strings are the most convenient way to get hex, binary, or octal strings from an integer.

See
[Format Placeholder Specification](https://en.wikipedia.org/wiki/Printf_format_string#Format_placeholder_specification)
for more information.

```v nofmt play
x := 123.4567
println('[${x:.2}]') // round to two decimal places => [123.46]
println('[${x:10}]') // right-align with spaces on the left => [   123.457]
println('[${int(x):-10}]') // left-align with spaces on the right => [123       ]
println('[${int(x):010}]') // pad with zeros on the left => [0000000123]
println('[${int(x):b}]') // output as binary => [1111011]
println('[${int(x):o}]') // output as octal => [173]
println('[${int(x):X}]') // output as uppercase hex => [7B]

println('[${10.0000:.2}]') // remove insignificant 0s at the end => [10]
println('[${10.0000:.2f}]') // do show the 0s at the end, even though they
                            // do not change the number => [10.00]
```

## String operators

V defines the `+` and `+=` operators for strings:

`+` – used for string concatenation

```v play
name := 'Bob'
bobby := name + 'by'
println(bobby) // Bobby
```

`+=` – used to append to a string

```v play
mut s := 'Hello, '
s += 'World!'
println(s) // Hello, World!
```

All operators in V must have values of the same type on both sides.
You cannot concatenate an integer to a string:

```v failcompile play
age := 20
println('age = ' + age)
//      ^^^^^^^^^^^^^^ error: infix expr:
//                     cannot use `int` (right expression) as `string`
```

To concatenate a string with a number, you must first convert the number to a string.

```v play
age := 21
println('age = ' + age.str()) // age = 21
```

or use string interpolation (**preferred**):

```v play
age := 22
println('age = ${age}') // age = 22
```

See all methods of
[string](https://modules.vosca.dev/standard_library/builtin.html#string)
and related modules
[`strings`](https://modules.vosca.dev/standard_library/strings.html),
[`strconv`](https://modules.vosca.dev/standard_library/strconv.html).
