# Compile-time reflection

V provides compile time reflection to get information about data types and structures at compile
time.
With its help, you can, for example, make your own efficient serializer for any data format that
will be generated during compilation.

## Fields

Each
[structure](../structs/overview.md),
[union](../unions.md)
and
[interface](../interfaces.md)
has a `fields` field that contains information about the fields of the structure:

```v play
struct User {
	name string
	age  int
}

fn main() {
	$for field in User.fields {
		println(field.name)
	}
	// name
	// age
}
```

The `fields` field is of type `[]FieldData`.
You can see what fields this type has in
[Standard Library API documentation](https://modules.vosca.dev/standard_library/builtin.html#FieldData).
For example, `FieldData` has a `typ` field that contains the type of the field.

While processing fields, you can separate logic by field name:

```v play
struct User {
	name string
	age  int
}

fn main() {
	$for field in User.fields {
		$if field.name == 'name' {
			println('found name field')
		}
	}
}
```

or by field type:

```v play
struct User {
    name string
    age  int
}

fn main() {
    $for field in User.fields {
        $if field.typ is string {
            println('found string field')
        }
    }
}
```

### Field access

While processing the fields of a structure, you can access the fields of
an object of that structure using the syntax `object.$(field.name)`:

```v play
struct User {
    name string
    age  int
}

fn main() {
    user := User{
        name: 'John'
        age: 25
    }
    $for field in User.fields {
        print('${field.name}: ')
        println(user.$(field.name))
    }
}

// name: John
// age: 25
```

You can also overwrite object field values:

```v play
struct User {
    name string
    age  int
}

fn main() {
    user := User{
        name: 'John'
        age: 25
    }
    $for field in User.fields {
		$if field.typ is string {
        	user.$(field.name) = 'Mark'
		}
    }
    println(user)
}

// User{
//     name: 'Mark'
//     age: 25
// }
```

> **Note**
> Accessing object fields via `object.$(field.name)` only works
> inside the `$for field in T.fields` loop.

## Methods

Each type has a `methods` field, which contains information about the methods of the type.
This field is of type `[]FunctionData`.
You can see what fields this type has in
[Standard Library API documentation](https://modules.vosca.dev/standard_library/builtin.html#FunctionData).

```v play
struct App {}

fn (mut app App) method_one() {}

fn (mut app App) method_two() int {
	return 0
}

fn (mut app App) method_three(s string) string {
	return s
}

fn main() {
	$for method in App.methods {
		$if method.typ is fn (string) string {
			println('${method.name} is `fn(string) string`')
		} $else {
			println('${method.name} is NOT `fn(string) string`')
		}

		$if method.return_type !is int {
			println('${method.name} does NOT return `int`')
		} $else {
			println('${method.name} DOES return `int`')
		}

		$if method.args[0].typ !is string {
			println("${method.name}'s first arg is NOT `string`")
		} $else {
			println("${method.name}'s first arg is `string`")
		}

		$if method.typ is fn () {
			println('${method.name} is a void method')
		} $else {
			println('${method.name} is NOT a void method')
		}
		println('')
	}
}
```

### Method call

While processing the methods of a type, you can call the methods of
an object of that type using the syntax `object.$method(args)`:

```v play
struct App {}

fn (mut app App) method_one() {}

fn (mut app App) method_two() int {
    return 0
}

fn (mut app App) method_three(s string) string {
    return s
}

fn main() {
    app := App{}
    $for method in App.methods {
        $if method.typ is fn (string) string {
            println(app.$method('hello'))
        }

        $if method.typ is fn () int {
            println(app.$method())
        }
    }
}
```

## Attributes

Each type has an `attributes` field, which contains information about the attributes of the type.
This field is of type `[]StructAttribute`.
You can see what fields this type has in
[Standard Library API documentation](https://modules.vosca.dev/standard_library/builtin.html#StructAttribute).

```v play
[name: "user"]
union User {
	name string
	age  int
}

fn main() {
	$for attr in User.attributes {
		println("'${attr.name}' with value '${attr.arg}'")
	}
	// 'name' with value 'user'
}
```

> **Note**
> Currently, function attributes cannot be retrieved.

## Enums values

Each enum has a `values` field, which contains information about the values of the enum.
The `values` field is of type `[]EnumData`.
You can see what fields this type has in
[Standard Library API documentation](https://modules.vosca.dev/standard_library/builtin.html#EnumData).

```v play
enum Color {
    red
    green
    blue
}

fn main() {
    $for value in Color.values {
        println("${value.name} has value ${value.value}")
    }
    // red has value 0
    // green has value 1
    // blue has value 2
}
```

## Type checking

The types stored in the `typ` field in `FieldData` or `FunctionData` can be compared using the `is`
operator:

```v nofmt failcompile
$if method.typ is fn (string) string {
	println('${method.name} is of type `fn (string) string`')
}

$if field is string {
	println('${field.name} is of type string')
}
```

`!is` is also supported:

```v nofmt failcompile
$if field.typ !is string {
    println('${field.name} is NOT of type string')
}
```

As with normal `if` expressions, you can use the `in` statement:

```v nofmt failcompile
$if field.typ in [string, int] {
	println('${field.name} is of type int or float')
}
```

For the convenience of checks, V defines some type constants:

- `$int` - any integer type
- `$float` - any floating point type
- `$array` - any array
- `$map` - any map
- `$struct` - any structure
- `$interface` - any interface
- `$enum` – any enum
- `$alias` - any alias type
- `$sumtype` – any summary type
- `$function` - any function

Example:

```v nofmt failcompile
$if field.typ is $int {
	println('${field.name} is of type int')
}

$if field.typ is $enum {
	println('${field.name} is Enum')
}
```

> **Note**
> To check the type of field, you need to use `$if` rather than `if`.
