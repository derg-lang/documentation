# Derg programming language design

## Table of contents

1. [Variables](#variables)
    1. [Immutable](#immutable)
    2. [Varying](#varying)
    3. [Mutable](#mutable)
2. [Functions](#functions)
    1. [Parameter list](#parameter-list)
    2. [Return value](#return-value)
    3. [Error value](#error-value)
    4. [Lambdas](#lambdas)
3. [Classes](#classes)
    1. [Types](#types)
    2. [Properties](#properties)
    3. [Constructors](#constructors)
    4. [Destructors](#destructors)
    5. [Methods](#methods)
4. [Identifiers](#identifiers)

## Variables

Any expression which should be referenced by name must be stored in a variable.
A variable allows re-using a result of evaluating an expression, and allows the
outcome to be accessed by reference. Variables thus binds a specific value to a
name.

Variables come in multiple forms, with different properties. A crucial concept
of variables is their mutability. A variable may be either physically immutable,
and/or logically immutable. For example, any variable which is stored in
read-only memory, or assigned only on instance creation, is a physically
immutable variable. Logically immutable variables are not permitted to change in
a localized part of the codebase, but in other locations might be updated.
Variables which are physically immutable are by extension also logically
immutable.

All variables must be assigned a value upon declaration, no exceptions.

### Immutable

Any variable which is immutable is considered a constant at the use-sites. These
variables may be declared at any time, and developers are guaranteed that the
values of them will never visibly change. The const-ness applies to every
property recursively as well, meaning no instance property may change either.

Note that pointers are not excluded from the const-ness - the pointer itself
must naturally remain unchanged, but the memory address the pointer points to
cannot be changed either.

Immutable variables may be declared using the `val` keyword:

```derg
val foo = 42
```

Any immutable variable cannot change either itself, nor any of the properties.
Member functions cannot be invoked, unless the `self` parameter is marked `val`.

// TODO: properties which are declared to bypass immutability are permitted to
change, even when `self` is immutable

The default choice for any new variable should always be `val` - the most
restrictive form should always be preferred. Only when the variable is proven to
require mutability, should the constraint be lifted. When making read-only
member functions, the `self` parameter should always be declared immutable, to
ensure the program remains const-correct.

### Varying

Whenever a variable is permitted to change its properties, it is varying. Note
that the *variable itself* is considered immutable and cannot change - it is the
properties within the instance which is permitted to mutate.

Varying variables may be declared using the `var` keyword:

```derg
var foo = 42
```

Any instance which is considered varying permits both immutable and varying
member functions to be invoked, so long as `self` is declared `var`. A common
use-case for the varying variable is to make a mutating member function, i.e. a
function to add a value to a list.

### Mutable

The least restricted form of variable is the mutable form. By declaring a
variable as mutable, there are no restrictions applicable to it. All properties
which are mutable, may be mutated, and mutating member functions may be invoked.
Additionally, the variable may be re-assigned a value.

Mutable variables may be declared using the `mut` keyword:

```derg
mut foo = 42

foo = 9001 // Re-assignement is permitted as well
```

Mutable variables should be kept to a minimum to ensure program state is less
tricky to keep in mind. An example of a mutable variable is a counter, or
accumulator.

## Functions

Smaller sub-routines of the program may be defined as a function. The function
represents a small unit of work which may be repeatedly executed, and given
different parameters to do different work or compute different results. The
functions thus form the basics of most programs, and is a crucial part in laying
out the overall structure of the program.

Functions may be declared using the `fun` keyword:

```derg
fun foo(val param: Int)
{
   // Arbitrary workload
}

foo(42)         // Invoke foo
foo(param = 42) // Invoke foo using named parameters
```

### Parameter list

The parameter list may contain arbitrary number of parameters, which may be
accessed inside the function to either calculate some specific value, or perform
different work. Parameters work similarly to variables, although they do not
have to be explicitly given a value.

A parameter list may be specified in the following manner:

```derg
fun foo(
   val param1: Int,
   var param2: Int = 2,
   ...
)
{
   // Arbitrary workload
}
```

Note that a trailing comma is permitted at the end of the parameter list.

### Return value

A function which only performs some arbitrary work is of limited use in normal
programs. By specifying the function return type, it is possible to compute a
value within the function and provide this value to the caller.

The return type may be specified in the following manner:

```derg
fun foo() -> Int
{
   return 42 // A return statement is mandatory
}
```

Any function which does *not* return a particular value, may still want to
return early if some precondition fails:

```derg
fun foo(val param: Bool)
{
   if !param
      return // Early return if some precondition fails
   // Do arbitrary work
}
```

### Error value

A reality in programming is that software *will* fail, and any programmer *must*
be prepared to deal with issues when the do arise. In order to avoid
contaminating the return value with noisy error handling approaches, a separate
error channel is utilized. This allows every function to return exactly *one* of
two values - either the return value, or the raised error.

The error type may be declared in a similar manner to the return type:

```derg
fun foo(): Bool -> Int32
{
   raise false
   return 0 // This line is never executed
}
```

Note that the error and value type do not need to be the same, and having an
error value does not mean the return value must *also* be specified - one can be
specified without the other. This is an especially powerful feature when
specific errors must be raised from a function, to indicate a specific error
case, even if the function does not necessarily return anything.

Any function which may raise an error forces callers to deal with the error when
invoked - either by re-raising the error, or by injecting some default value.
Errors may be re-raised by the raise operator `!`, and may be replaced with a
default value using the catch operator `?`, in the following manner:

```derg
fun foo(): auto -> Int
{
   return 1 / 0 ! // The divide-by-zero error is passed on to the caller
}

fun bar() -> Int
{
   return 1 / 0 ? -1 // The divide-by-zero error is evaluated as `-1` instead
}
```

### Lambdas

// TODO: lambdas are a special type of functor object which are callable.
Lambdas allow variables to be captured and used within a subroutine

## Classes

Encapsulation of data is performed via classes. Developers may group various
related data together into a data type, or define a collection of types as a
union type. Types may be extended by appending member functions, allowing
behavior to be associated with a specific type.

### Types

A data type may be declared using the `type` keyword:

```derg
type Foo
```

Types are not compatible with each other. If a function expects a specific type
as a parameter, only that type will be accepted. Sometimes it is desirable to
define a new type with the same operations as another type, but with different
semantics. An alias type inherits all properties and functionality of the base
type, and is for most cases not considered the same type.

As an example, the health of a monster in a video game is different from its
speed, although both may be represented with floating point values. These
numerical values should not be allowed to be mixed.

An alias type may be defined using the `type` keyword as well:

```derg
type Health = Int
```

Note that the source type (here `Int`) is implicitly promoted to the alias
type (here `Health`), but not the other way around. Converting back is only
possible when requested explicitly by the developer.

// TODO: revisit alias types - they may interfere too much with union types.
Also re-visit implicit conversions - implicit conversions are scary!

Several types may be referenced at the same time through a union type. The union
type conveys a concrete instance is *one* of the types specified in the union.
in order to access the information stored in the instance, the developer is
forced to determine which type the instance is first.

A union type may be defined using the `type` keyword as well:

```derg
type Value =
   Int,
   Bool,
   Float,
   String,
```

Trailing commas are permitted when declaring a union type.

#### References

// TODO: any lvalue may be referenced by another variable, although this is a
more complicated situation involving lifetimes

#### Pointers

// TODO: all variables are given a specific location in memory - a pointer
allows the developer to access variables by their memory address, which opens a
terrifying can of worms

### Properties

Different types may have different data stored within them. Every property of a
type is represented as a variable, although they are *not required* to be
initialized at the declaration time. Note that all properties are *still
required* to be initialized in the constructor, however.

Properties may be declared in the following manner:

```derg
type Foo
{
   val prop1: Int
   val prop2: Int = 42 // Default values *may* be overwritten by constructors
}
```

### Constructors

Constructors are specialized functions which instantiates an instance of a
specific type. The constructors are named functions and invoked in the same
manner as any other function, although they may fail to create the desired
instance.

All constructors are also granted implicit access to the `var self: Self`
parameter, where `Self` is the type of the instance being constructed. All
constructors are automatically given the return type `Self`.

Constructors may be declared using the `ctor` keyword:

```derg
type Foo
{
   val prop1: Int
   val prop2: Int = 42
}

// Default constructor
ctor Foo()
{
   self.prop1 = 0
}

// Named constructor
ctor Foo::some_factory_name(val param: Int)
{
   self.prop1 = param
}
```

The default constructor is only available if all properties are assigned a
default value, or it is explicitly defined.

Note that all properties *without* a default value *must* be assigned in every
constructor. All properties are initialized in the order they are assigned in
the constructors; default values not overridden by the constructor are
initialized first. No property (except non-overridden default-initialized
properties) may be accessed in constructors, and constructors are not permitted
to invoke other constructors for the same type.

To instantiate a new instance of a type, the constructor may be invoked like any
ordinary static function:

```derg
val foo = Foo()
val bar = Foo::some_factory_name(42)
```

### Destructors

All instances *must* be destroyed when they leave scope, which involves invoking
exactly one destructor. For almost all types, the default destructor will be
invoked automatically when the instance is removed from the current stack,
although in some cases developers want to define their own destructors.

All destructors are also granted implicit access to the `move var self: Self`
parameter, where `Self` is the type of the instance being constructed. This
means that every destructor takes ownership of the instance, preventing any
other usage of the instance later.

Non-default destructors are permitted any number of parameters, and may also
return any arbitrary value.

A destructor may be declared using the `dtor` keyword:

```derg
type Foo

// Default destructor
dtor Foo()
{
   // Arbitrary workload
}

// Custom destructor with a parameter
dtor Foo::some_cleanup_name(val param: Int)
{
   // Arbitrary workload
}
```

The destructor is only available if all properties are default-destructible AND
no custom destructor is declared, or it is explicitly defined.

Whenever an instance goes out of scope or is re-assigned a new value, the
default destructor is invoked if defined. If the default destructor is not
defined, then the instance is not permitted to go out of scope or be re-assigned
before any of the other custom destructors have been invoked instead.

Destructors may thus be invoked in the following manners:

```derg
fun workload1()
{
   val foo = Foo()
} // The default destructor of `foo` is invoked here

fun workload2()
{
   val foo = Foo()
   foo.some_cleanup_name(42) // Invoke a custom destructor
} // The default destructor of `foo` is *not* invoked here
```

### Methods

Member functions are behaving in the same manner as function, except they may
have access to a type's internal state. Any member function declared in the same
module as the type itself is permitted access to both the public and private
properties. Any member function declared outside the module, is only permitted
access to the public properties.

When declaring a member function, the parameter `self` must be provided. This
special parameter references the instance which the member function is invoked
upon - the type of `self` will *always* be the same type as the type the member
function is acting on, and will always be a reference. `self` may only ever be
a `val` or `var`, and can never be passed as `copy` or `take`.

Methods may be declared using the `fun` keyword, and are in the namespace of the
specific type:

```derg
type MyType

fun MyType::instance_method(val self)
{
   // Arbitrary work
}

val foo = MyType()
foo.instance_method()   // Invoke instance_method on foo
```

## Identifiers

// TODO: identifiers are the names of variables, functions, types, namespaces,
concepts, and every other resource which may be named.
