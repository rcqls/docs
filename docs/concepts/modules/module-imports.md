# Module imports

Modules can be imported using the `import` keyword:

```v
import os

fn main() {
	// read text from stdin
	name := os.input('Enter your name: ')
	println('Hello, ${name}!')
}
```

This program can use any public definitions from the `os` module, such
as the `input` function.
See the
[standard library](https://modules.vosca.dev/standard_library/index.html)
documentation for a list of common modules and their public symbols.

By default, you have to specify the module prefix every time you call an external function.
This may seem verbose at first, but it makes code much more readable and easier to
understand – it is always clear which function from which module is being called.
This is especially useful in large code bases.

Cyclic module imports are not allowed, like in Go.

## Selective imports

You can also import specific functions and types from modules directly:

```v
import os { input }

fn main() {
	// read text from stdin
	name := input('Enter your name: ')
	println('Hello, ${name}!')
}
```

> **Note**
> This will import the module as well. Also, this is not allowed for
> constants – they must always be prefixed.

You can import several specific symbols at once:

```v
import os { input, user_os }

name := input('Enter your name: ')
println('Name: ${name}')
current_os := user_os()
println('Your OS is ${current_os}.')
```

## Module import aliasing

Any imported module name can be aliased using the `as` keyword:

```v failcompile
import crypto.sha256
import mymod.sha256 as mysha256

fn main() {
	v_hash := sha256.sum('hi'.bytes()).hex()
	my_hash := mysha256.sum('hi'.bytes()).hex()
	println(my_hash == v_hash) // true
}
```

You cannot define methods for imported types.
However, you _can_ redeclare a type using
[type aliases](../type-aliases.md)
and add methods for it:

```v play
import time
import math

type MyTime = time.Time

fn (mut t MyTime) century() int {
	return int(1.0 + math.trunc(f64(t.year) * 0.009999794661191))
}

fn main() {
	mut my_time := MyTime{
		year: 2020
		month: 12
		day: 25
	}
	println(time.new_time(my_time).utc_string())
	println('Century: ${my_time.century()}')
}
```
