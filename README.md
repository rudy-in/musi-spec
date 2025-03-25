# Musi Language Specification (Under development)

## 1. Introduction
Musi is a general-purpose programming language designed w/ simplicity, memory-safety and performance in mind. Instead of a garbage collector, issues w/ the memory are prevented by the 'Automatic Reference Counter' (ARC), which tracks reference counts for every appropriate given object in memory. Meaning its like C/C++ but expressed as a scripting language like Bash or Lua but memory safe and designed to be simplified for noobs with performance almost equal to Rust. Since objects are strong references by default
so they get tracked
Musi runs code from top to bottom. Unlike many langs, there's no `main()`.
Instead, bottom-most body in file is main execution point this means that this code:
```musi
import "math"

func add(a : Int, b : Int) -> Int := a + b

# main code starts at the bottom
var name := input("enter your name: ")
writeln("hello, $name!")
```
is read by the compiler as:

```musi
writeln("hello, $name!")
var name := input("enter your name: ")
func add(a : Int, b : Int) -> Int := a + b
import "math"
```                

### 1.1 Hello world

```musi
writeln("hello world")
```
`writeln` prints any text or number in a new line, we can also use `write` at this point like

```musi
write("hello world")
```
the difference is if you execute it like this it will not print in a newline. Lets spam both of these and see the output to differentiate.
```musi
writeln("hello world")
writeln("hello world")
writeln("hello world")
writeln("hello world")
```
this outputs like this
```
hello world
hello world
hello world
hello world
```
when we do the same with `write` 
```musi
write("hello world")
write("hello world")
write("hello world")
write("hello world")
```
this outputs to 
```
hello worldhello worldhello worldhello world
```
now we know what to use for a neat hello world print and stamp that onto our resume (joking)

## 2. Basic Syntax and Structure

### 2.1 Comments

They start w/ hash symbol (`#`):

```musi
# this is comment
var count : Int := 5  # comment at end of line

# you can also write
# multi-line comments
# like this
# because i haven't
# figured out
# syntax for multi-line
# yet
# but when i do
# we can have
# the equivalent of comment blocks
# inside expressions and statements
```

### 2.2 Indentation and Structure

Musi uses spaces at line starts to show which parts of code belong to one other.
One must use equal spacing for each level.

```musi
proc example() :=
 |   if condition then
 |     | do_something()
 |   else
 |     | do_other_thing()
```

### 2.3 Operators

| Level | Operators                                          | Description                                 |
| ----- | -------------------------------------------------- | ------------------------------------------- |
| 1     | `()`, `.`                                          | Call, member access                         |
| 2     | `-`, `+`, `not`                                    | Unary denial, plus, logical/bitwise NOT     |
| 3     | `^`                                                | Power                                       |
| 4     | `*`, `/`, `mod`, `rem`                             | Multiplication, division, modulo, remainder |
| 5     | `+`, `-`                                           | Addition, subtraction                       |
| 6     | `shl`, `shr`, `rol`, `ror`                         | Shifts and rotations                        |
| 7     | `..`, `..<`                                        | Inclusive range, exclusive range            |
| 8     | `=`, `/=`, `>`, `>>`, `<`, `<<`, `>=`, `<=`, `<=>` | Comparisons                                 |
| 9     | `and`, `nand`                                      | Logical/bitwise AND, NAND                   |
| 10    | `or`, `nor`, `xor`, `xnor`                         | Logical/bitwise OR, NOR, XOR, XNOR          |
| 11    | `:=`                                               | Assignment                                  |

Keyword operators `and`, `or` and `not` can work on `true`/`false` values or on
bits of numbers:

```musi
# logical ops w/ bools
let can_access : Bool := admin and logged_in
let is_active : Bool := not suspended

# bitwise ops w/ ints
let bits : Int := 0b1100 and 0b1010
```

### 2.4 Naming Conventions

Here's how we (idiomatically) name things in Musi, at least for when we have
official fmt tooling:

```musi
let GLOBAL_CONST := _       # SCREAMING_SNAKE_CASE
let local_const := _        # snake_case
var local_var := _          # snake_case

proc my_proc() := _         # snake_case
func my_func() := _         # snake_case

type MyType as Int          # PascalCase

enum MyEnum                 # PascalCase
    case OptionA            # PascalCase

interface MyInterface       # PascalCase
struct MyStruct             # PascalCase
```

## 3. Variables, Constants, and Types

### 3.1 Variables and Constants

We support both mutable and immutable data:

```musi
# mutable variables
var counter : Int := 42
var name : Str := "Alice"

# immutable variable
let max_users : Int := 1000

# global constant (also immutable)
let PI : Real := 3.14159
```

### 3.2 Basic Types

We have these basic types:

```musi
# numbers
var a : Real := 3.14    # decimal number, like '3.14', '0.5', '-1.0'
var b : Int := 42       # integer, like '42', '-5', '0'
var c : Nat := 0N       # 0-incl. natural number

# text
var d : Str := "hello"  # text in quotation marks (multi-line supported)

# logical
var e : Bool := true    # can only be 'true' or 'false'
```

### 3.3 Type Aliasing

Use `type` keyword to make new names for types:

```musi
type UserId as Int
type UserMap as Map<UserId, Str>
```

### 3.4 Type Checking and Casting

We provide ops for type checking and casting:

```musi
var num : Real := 1.1
var int_value : Int := Int(num)  # -> 1

# type checking w/ 'as?'
var any_value : Any := get_value()
var maybe_str : Str? := any_value as? Str  # -> '.None' if not 'Str'

# force casting w/ 'as!'
proc process_str(value : Any) :=
    var str : Str := value as! Str  # throws error if not 'Str'
    print(str.length())
```

### 3.5 Option and Result Types

We provide `Maybe<T>` and `Either<L, R>` types for handling optional values and
ops that might fail:

```musi
# Maybe - optional values
func find_user(id : Str) -> Maybe<User> :=
    when database.get(id)
    = .Some(user) then .Some(user)
    else .None

# using 'Maybe' values
proc greet_user(id : Str) :=
    when find_user(id)
    = .Some(user) then writeln("hello, ${user.name}")
    = .None then writeln("user not found")

# Either - operation that might fail
func divide(a : Int, b : Int) -> Either<Str, Int> :=
    when b = 0 then .Left("division by zero")
    else .Right(a / b)

# using 'Either' values
proc show_result(a : Int, b : Int) :=
    when divide(a, b)
    = .Right(result) then writeln("result: $result")
    = .Left(error) then writeln("error: $error")
```

There's no `nil` (or `null`) type because I feel like it's recipe for disaster.
Instead, use `Maybe<T>` w/ `Some(value)` and `None` for optional values.

## 4. Control Flow

### 4.1 Conditionals

These let you make choices based on `true`/`false` conditions:

```musi
if temp > 30 then
    writeln("hot")
else if temp > 20 then
    writeln("comfortable")
else
    writeln("cold")

# using 'unless' (syntax sugar for 'if not')
# can also use: "unless warm then wear_coat()"
wear_coat() unless warm
```

### 4.2 Pattern Matching w/ When

This lets you check data and run different stuffs based on what you find:

```musi
# basic values
func describe(x : Int) -> Str :=
    when x
    = 0 then "zero"
    = 1 then "one"
    = n where n < 0 then "negative"
    else "positive"

# list structure
func first(list : [Int]) -> Maybe<Int> :=
    when list
    = [] then .None
    # [head | tail]
    = [x | _] then .Some(x)
```

It also provides alternative to nested `if` expressions:

```musi
let grade := when score
    in 90..100 then "A"
    in 80..89 then "B"
    in 70..79 then "C"
    else "F"
```

### 4.3 Loops

#### 4.3.1 For Loops

Ones w/ numeric ranges:

```musi
# incl. range (incl. 10)
for i in 1..10 loop
    writeln("item $i")

# excl. range (excl. 10)
for i in 1..<10 loop
    writeln("item $i")

# w/ step size
for i in 0..10 step 2 loop
    writeln(i)
```

One that goes through collections:

```musi
for fruit in ["apple", "banana", "orange"] loop
    writeln("I like $fruit")
```

`yield` keyword turns loops into value assigners:

```musi
# [1, 4, 9, 16, 25]
let squares := for i in 1..5 yield i * i

# w/ conditions
# [4, 16, 36, 64, 100]
let even_squares := for i in 1..10 where i mod 2 = 0 yield i * i
```

#### 4.3.2 While and Until Loops

These run as long as something is true:

```musi
var count : Int := 3
while count > 0 loop
    writeln("countdown: $count")
    count := count - 1
```

Can also run code at least once before checking to stop:

```musi
var password : Str := ""
until password.length() >= 8 loop
    password := input("enter password: ")
```

Use `while true`, `for i in 1..` or `loop` to create infinite loop:

```musi
while true loop
    process_events()
    if should_stop() then break

for i in 1.. loop
    process_events()
    if should_stop() then break

loop
    process_events()
    if should_stop() then break
```

#### 4.3.3 Loop Control

`continue` and `break` keywords control loops:

```musi
for item in items loop
    if item.is_invalid() then continue     # skip this and move to next iteration
    if item.is_last() then break        # exit loop entirely
    process(item)
```

## 5. Procedures and Functions

We split between procedures (which only do something w/o giving back) and
functions (which give back a value):

### 5.1 Procedures

Procedures do stuff and not return values:

```musi
proc greet(name : Str) :=
    writeln("hello, $name!")

proc process_data(items : [Int]) :=
    var total : Int := 0
    for item in items loop
        total := total + item
    writeln("total: $total")
```

### 5.2 Functions

Functions determine and return values w/o changing state:

```musi
func add(a : Int, b : Int) -> Int :=
    a + b

func calculate_area(width : Real, height : Real) -> Real :=
    width * height

# multi-line function w/ explicit return
func fibonacci(n : Int) -> Int :=
    if n <= 1 then return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

### 5.3 Lambda Expressions

These create unnamed functions:

```musi
# simple lambda
let double := (x : Int) -> x * 2

# used w/ map
let numbers := [1, 2, 3, 4, 5]
let doubled_list := numbers.map(x -> x * 2)
```

### 5.4 Higher-Order Functions

These take functions as args or return them:

```musi
func apply_twice(f : Int -> Int, x : Int) -> Int :=
    f(f(x))

let result := apply_twice(x -> x * 2, 3)
```

### 5.5 Closures

These capture vars around them:

```musi
func make_counter() : () -> Int :=
    var count : Int := 0
    # implicit return
    () ->
        count := count + 1
        count

let counter := make_counter()
let a := counter()  # 1
let b := counter()  # 2
```

## 6. Structures and Enums

### 6.1 Structures

These create new mixed data types w/ added behaviour:

```musi
struct Point
    # all fields are public to us
    var x : Int := 0
    var y : Int := 0

    # constructor (initialiser)
    init(self, x : Int, y : Int) :=
        self.x := x
        self.y := y

    # static method (doesn't need instance)
    shared func origin() -> Point :=
        Point(0, 0)

    # instance method that works on struct's data
    func distance_to(self, other : Point) -> Int :=
        (self.x - other.x).abs() + (self.y - other.y).abs()

    # pure transform that returns new instance
    func shifted(self, dx : Int, dy : Int) -> Point :=
        Point(self.x + dx, self.y + dy)

    # method that changes struct
    proc move(self, dx : Int, dy : Int) :=
        self.x := self.x + dx
        self.y := self.y + dy

    func to_string(self) -> Str :=
        "(${self.x}, ${self.y})"
```

Using struct methods:

```musi
proc work_with_points() :=
    var p1 := Point(10, 20)
    var p2 := Point(15, 25)

    var dist := p1.distance_to(p2)
    var shifted_point := p1.shifted(5, 10)

    p1.move(3, -2)

    var origin := Point.origin()

    writeln(p1.to_string())
```

### 6.2 Abstract Structs

These define behaviour that must be filled in by main structs:

```musi
abstract struct Shape
    # placeholder w/o impl
    abstract func area(self) -> Real

    # w/ impl
    func perimeter(self) -> Real := 0.0
```

Main structs must implement all placeholders:

```musi
struct Circle : Shape
    var radius : Real := 0.0

    init(self, radius : Real) :=
        self.radius := radius

    # impl placeholder
    override func area(self) -> Real :=
        PI * self.radius * self.radius

    # impl default
    override func perimeter(self) -> Real :=
        2.0 * PI * self.radius
```

### 6.3 Enumerations

These define custom type that consists of fixed set of named choices. Each
choice can optionally contain added data:

```musi
enum Status
    case Success
    case Error(message : Str)
    case Pending

    func is_success(self) -> Bool :=
        when self = .Success then true
        else false

    func describe(self) -> Str :=
        when self
        = .Success then "operation completed"
        = .Error(msg) then "error: $msg"
        = .Pending then "operation in progress"
```

They can also contain data in each choice:

```musi
enum Shape
    case Circle(radius: Real)
        func area(self) -> Real :=
            PI * self.radius * self.radius

        func perimeter(self) -> Real :=
            2.0 * PI * self.radius

    case Rectangle(width: Real, height: Real)
        func area(self) -> Real :=
            self.width * self.height

        func perimeter(self) -> Real :=
            2.0 * (self.width + self.height)

        func is_square(self) -> Bool :=
            self.width = self.height

    case Triangle(a: Real, b: Real, c: Real)
        func perimeter(self) -> Real :=
            self.a + self.b + self.c

        func is_equilateral(self) -> Bool :=
            self.a = self.b and self.b = self.c
```

## 7. Generics

These let you write stuffs that works w/ many types:

```musi
func identity<T>(x : T) -> T := x

# w/ multiple type params
func pair<A, B>(a : A, b : B) -> (A, B) := (a, b)

# value-level generics w/ params
func matrix<T, N : Int, M : Int>(default : T) -> [[T]] :=
    var result : [[T]] := []
    for i in 0..<N loop
        var row : [T] := []
        for j in 0..<M loop
            row.append(default)
        result.append(row)
    return result

# can be used in comptime calcs
func fixed_array<T, Size : Int>(value : T) -> [T] :=
    var arr : [T] := []
    for i in 0..<Size loop
        arr.append(value)
    return arr

# type constraints w/ 'where' clause
func sum<T>(items : [T]) -> T where T : Numeric :=
    var total : T := T.zero()
    for item in items loop
        total := total + item
    return total

# struct w/ generic params
struct Pair<A, B>
    var first : A
    var second : B

    # methods can use generic types
    func swap(self) -> Pair<B, A> :=
        Pair(self.second, self.first)
```

## 8. Interfaces

These let you define shared behaviour. Interfaces define a set of methods that
types can implement.

### 8.1 Defining Interfaces

These define contracts that types can fulfill:

```musi
interface Drawable
    # needies that implrs must provide
    func draw(self) -> Str

    # optionals w/ default impls
    func description(self) -> Str :=
        "a drawable object"
```

### 8.2 Implementing Interfaces

You implement interfaces in structures using inheritance syntax:

```musi
struct Circle with Drawable
    var x : Int := 0
    var y : Int := 0
    var radius : Int := 0

    init(self, x : Int, y : Int, radius : Int) :=
        self.x := x
        self.y := y
        self.radius := radius

    # impl needed interface method
    func draw(self) -> Str :=
        "circle at (${self.x}, ${self.y}) w/ radius ${self.radius}"

    # OPTIONAL: override default impl
    override func description(self) -> Str :=
        "Circle w/ radius ${self.radius}"
```

Enums can also implement interfaces:

```musi
enum Shape with Drawable
    case Rectangle(width : Int, height : Int)
    case Circle(radius : Int)

    # impl needed interface method
    func draw(self) -> Str :=
        when self
        = .Rectangle(w, h) then "Rectangle $w x $h"
        = .Circle(r) then "Circle w/ radius $r"
```

### 8.3 Using Interfaces for Polymorphism

Interfaces enable polymorphism:

```musi
# accept any type that impls 'Drawable'
proc render(item : Drawable) :=
    writeln(item.draw())

# or using a generic w/ interface bound
func create_description<T : Drawable>(item : T) -> Str :=
    "description: ${item.description()}"

let circle := Circle(10, 20, 5)
let shape := Shape.Rectangle(30, 40)

# both can be passed to funcs expecting 'Drawable'
render(circle)
render(shape)
```

### 8.4 Interface Composition

Types can implement multiple interfaces:

```musi
interface Sized
    func size(self) -> Int

interface Drawable
    func draw(self) -> Str

# combine interfaces in impl
struct Rectangle with Drawable, Sized
    var width : Int := 0
    var height : Int := 0

    # impl 'Drawable'
    func draw(self) -> Str :=
        "Rectangle ${self.width}x${self.height}"

    # impl 'Sized'
    func size(self) -> Int :=
        self.width * self.height
```

## 9. Memory Management

### 9.1 Automatic Reference Counting

We use Automatic Reference Counting (ARC) for memory management. Structs are
value types that are copied when assigned, while reference types are
automatically freed when their count reaches zero. Unlike langs that need manual
cleanup, we handle this automatically.

```musi
proc arc_example() :=
    # value types are copied
    var point1 := Point(10, 20)
    var point2 := point1  # creates full copy
    point2.x := 30        # doesn't affect 'point1'

    var resource := Resource()  # ref count = 1
    do
        var ref2 := resource    # ref count = 2
        # 'ref2' goes out of scope, count = 1
    # resource automatically freed when scope ends and count = 0
```

### 9.2 Weak References

To prevent memory leaks from circular references, we declare something as
`weak`, implying weakly referenced.

```musi
struct Parent
    var name : Str := ""
    var children : [Child] := []  # strong ref to children

struct Child
    var name : Str := ""
    weak var parent : Parent?     # weak ref to parent

    # weak refs don't incr. ref count
    # and automatically become '.None' when ref'd object is freed
```

### 9.3 Deterministic Cleanup w/ `defer`

`defer` keyword ensures cleanup operations are performed when current scope
exits, regardless of how it exits:

```musi
proc process_file(path : String) :=
    var file := open_file(path)
    defer close_file(file)  # called when proc exits

    # starting file processing...
    if error_condition then
        return  # file still gets closed

    # processing...
    # file closed automatically at end of proc
```

## 10. Modules and Organisation

### 10.1 File Structure as Modules

Each file is module. Module name comes from file name, and folder name gives
namespace.

File named `math.ms` gives module `math`, and file named `collections/list.ms`
gives module `collections.list`.

### 10.2 Importing Modules

Use `import` keyword to get code from other modules:

```musi
import math
import collections.list

# using 'as' to rename modules
import math as maths
import collections.linked_list as list

# multiple modules at once
import (
    math as maths,
    collections/linked_list as list
)
```

### 10.3 Exporting Declarations

By default, all declarations in file are private, which means they can only be
used inside that file. Use `export` to make them available to other modules:

```musi
# not visible outside this file
func helper(values : [Int]) -> Int :=
    values.sum()

# visible to other modules that import this one
export func calculate_total(values : [Int]) -> Int :=
    helper(values) * 2

export let MAX_USERS : Int := 1000

export struct User
    # all fields are public
    var name : Str := ""
    var email : Str := ""
    var id : Int := 0

    #  only usable here
    func private_helper() -> Bool :=
        self.email.contains("@")

    # usable outside this
    export func is_valid(self) -> Bool :=
        private_helper() and self.name.length() > 0
```

This way, we keep things simple:

- Use `export` to control what other modules can see
- Know that all `struct` members are publically accessable
- No need for complex visibility rules

## 11 Function Composition

Combine one another to create new functions:

```musi
func compose<A, B, C>(f : B -> C, g : A -> B) -> (A -> C) :=
    x -> f(g(x))

let double := x -> x * 2
let increment := x -> x + 1
let double_then_increment := compose(increment, double)
```

### 11.1 Currying and Partial Application

```musi
func add(a : Int) -> (Int -> Int) := b -> a + b

let add5 := add(5)
let result := add5(3)  # 8
```

## 12. Error Handling

### 12.1 Using Result Types

Use `Either<L, R>` for error handling:

```musi
enum Either<L, R>
    case Left(value : L)   # usually used for errors
    case Right(value : R)  # usually used for success

    func map<U>(self, f : R -> U) -> Either<L, U> :=
        when self
        = .Left(v) then .Left(v)
        = .Right(v) then .Right(f(v))

    func flat_map<U>(self, f : R -> Either<L, U>) -> Either<L, U> :=
        when self
        = .Left(v) then .Left(v)
        = .Right(v) then f(v)

# using 'Either' for error handling
func divide(a : Int, b : Int) -> Either<Str, Int> :=
    when b = 0 then .Left("division by zero")
    else .Right(a / b)

proc use_division() :=
    let result := divide(10, 2)
    when result
    = .Right(value) then writeln("result: $value")
    = .Left(msg) then writeln("error: $msg")
```

### 12.2 Using Maybe Types

`Maybe<T>` handles optional values:

```musi
enum Maybe<T>
    case Some(value : T)
    case None

    func map<U>(self, f : T -> U) -> Maybe<U> :=
        when self
        = .Some(v) then .Some(f(v))
        = .None then .None

    func flat_map<U>(self, f : T -> Maybe<U>) -> Maybe<U> :=
        when self
        = .Some(v) then f(v)
        = .None then .None

    func or_else(self, default : T) -> T :=
        when self
        = .Some(v) then v
        = .None then default

# using 'Maybe' to handle possible missing values
func find_user(id : Int) -> Maybe<User> :=
    # impl here

# pat-matting on 'Maybe'
proc greet(id : Int) :=
    when find_user(id)
    = .Some(user) then writeln("hello, ${user.name}")
    = .None then writeln("user not found")

# using transforms
func get_email_domain(id : Int) -> Maybe<Str> :=
    find_user(id)
    |> flat_map(user -> user.email)         # 'Maybe<Str>'
    |> map(email -> email.split("@")[1])    # 'Maybe<Str>'
```
