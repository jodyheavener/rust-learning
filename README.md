# Rust course

Rust provides:

1. Concurrency
2. Safety
3. Speed

## Fundamentals

### Cargo

Cargo is

- A package manager. You can search for, install, and use packages with it.
- The build system.
- The tests runner
- Documentation generator.

Config files in Rust use the TOML (Tom's Obvious Minimal Language) format. Rust source files use the `.rs` extension.

The TOML config file is the authoritative source of information about your project.

Run `cargo new hello` to generate a simple example project. Use `cargo run` to compile and run it.

### Variables

Rust is a braced, typed language:

```rust
let bunnies: i32 = 4;
```

A `let` can destructure data on the right hand side and assign it to the variables on the left hand side, so you can create multiple variables at once:

```rust
// Tuple pattern
let (bunnies, carrots) = (8, 50);
```

All variables are immutable unless you make them mutable, which means that unless you specify `mut` a value cannot be changed once it's set:

```rust
let mut bunnies = 4;
bunnies = 10; // This works because of `mut`
```

Constants are super immutable. They use the `const` keyword, are conventionally written in ALL_CAPS, the type annotation is required, and the value must be a constant expression that can be determined at compile time (like a literal):

```rust
const WARP_FACTOR: f64 = 9.9;
```

Why use a constant? Two reasons:

1. You can place a constant outside of a function at module scope. A global, immutable value.
2. Because constants are inlined at compile time they are really fast.

### Scope

The scope of a variable begins where it's created and extends to the end of the block. Along the way it is accessible from nested blocks. There is no garbage collector; values are immediately dropped when they go out of scope:

```rust
fn main() {
    let x = 5;
    {
        let y = 99;
        println("{}, {}", x, y); // Good - x and y are in scope
    }
    println("{}, {}", x, y); // Error - y is not in scope
}
```

But scoping errors will be found at compile time.

Variables can also be shadowed; they are local to their scope:

```rust
fn main() {
    let x = 5;
    {
        let x = 99;
        println("{}", x); // Prints 99
        // As soon as this block ends the inner x value is
        // dropped and does not redefine x in the outer scope
    }
    println("{}", x); // Prints 5
}
```

Variables can be shadowed in the same scope; for example you could make a mutable value immutable:

```rust
fn main() {
    let mut x = 5;
    let x = x; // Redefine with different mutability
}
```

You can also shadow a variable to a different type:

```rust
fn main() {
    let meme = "Look at all those chickens";
    let meme = vine_clip(); // Redefine as different type
}
```

### Memory Safety

Variables need to be declared before being used. It won't even compile if any variable is used before being defined.

This will not work:

```rust
fn main() {
    let enigma: i32;
    if true {
        enigma = 42;
    }
    // The compiler can't guarantee the variable gets
    // assigned a value, so it will fail.
    println("enigma is {}", enigma); // Error
}
```

This _will_ work:

```rust
fn main() {
    let enigma: i32;
    if true {
        enigma = 42;
    } else {
        enigma = 7;
    }
    // The compiler knows that the variable has to
    // be one of the other value, and will compile.
    println("enigma is {}", enigma); // Error
}
```

### Functions

Functions are created with the `fn` keyword, names should be snake case, and definitions do _not_ need to appear before being called. Prefer 4-space indentation

```rust
fn main() {
    do_stuff();
}

fn do_struff() {
}
```

Arguments are written as name-colon-type, separated by comma. Specify the return type with `->` after the argument brackets:

```rust
fn do_stuff(qty: f64, oz: f64) -> f64 {
    return qty * oz;
}
```

There is also a short-hand way to return with a "tail expression". Remove the `return` and semi-colon off the last item in the block to implicitly return it:

```rust
fn do_stuff(qty: f64, oz: f64) -> f64 {
    qty * oz
}

// This:
{ return true; }
// Is the same as this:
{ true }
```

The short-hand is preferred in Rust.

A Rust function does not support variable numbers of arguments, or variable types for a single argument. But macros, like `println`, do. A macro looks like a function but is always called with an exclamation mark:

```rust
fn do_stuff(qty: f64, oz: f64) -> f64 {
    println!("{} {}-oz tomatoes", qty, oz);
    qty * oz
}
```

### Module system

It is powerful and flexible.

Here's how you can import one function into another function using its absolute path:

```rust
// lib.rs
pub fn greet() { // Note the pub keyword
    println!("Hi!");
}

// main.rs
fn main() {
    hello::greet();
 // ^---------------- Library name (defined in Cargo.toml)
 //      ^----------- Scope operator
 //        ^--------- Function name
}
```

However specifying an absolute path for every imported function can become cumbersome and unwieldy, especially if the project name is long.

Instead, you can _use_ the `use` statement. It brings an item from some path into scope. Using the above example, this is how `main.rs` could be set up instead:

```rust
use hello::greet;

fn main() {
    greet();
}
```

This is generally how you would import functions from the Rust standard library or any other libraries you use:

```rust
// Import HashMap from the standard library
use std::collections::HashMap;
```

Use curly braces to import multiple items:

```rust
use hello::{greet, wave, send};
```

Searching Google for `rust std [whatever]` is good for finding documentation on anything in the standard library.

[Crates.io](crates.io) can be used for third-party libraries. It is Rust's package registry. "crates" are Rust's "packages"; the term can be used interchangeably.

Once you have found a dependency you'd like to use you'll add it to your `Cargo.toml` file under `[dependencies]`:

```toml
[dependencies]
rand = "0.6.5"
```

With the above `rand` library added you can use it in a function:

```rust
// Either by absolute path
fn main() {
    let x = rand::thread_rng().gen_range(0, 100);
}

// Or by bringing a specific item into scope with `use`
use rand::thread_rng;

fn main() {
    let x = thread_rng().gen_range(0, 100);
}
```

## Primitive Types & Control Flow

### Scalar types

There are **four** scalar types:

#### Integer

Whole numbers. There are a lot of integer types. They start `u` or `i` and are followed by the number of bits the integer has, except for `size` which is the size of the platform's pointer type, can represent every memory address, and is usually what you would use to index into an array or vector.

Unsigned (begins with `u`):

- `u8`
- `u16`
- `u32`
- `u64`
- `u128`
- `usize`

Signed (begins with `i`):

- `i8`
- `i16`
- `i32`
- `i64`
- `i128`
- `isize`

If you don't annotate an integer literal it defaults to `i32` because it's generally the fastest bit integer.

The terms "u8" and "byte" are interchangeable in Rust.

Any numeric value can have any number of underscores in it for improved readability.

```rust
1000000 == 1_000_000 // Decimal
0xdeadbeef == 0xdead_beef // Hex
0o77543211 == 0o7754_3211 // Octal
0b1110011 == 0b111_0011 // Binary
```

#### Float

Numbers with a decimal point. There are only two floating point types:

- `f32`
- `f64`

`f64` is the default because it has more precision but can be slower on machines that don't 64 bit architecture.

Literals are simple, and don't require a special suffix, but you do need at **least one digit** before the decimal:

```rust
3.124 // Good
0.1 // Good
.1 // Bad
```

**Handy tip:**

Numerical values (integers and floats) can alternatively place the type as a suffix to the literal:

```rust
// Typical method
let x: u15 = 5;
let y: f32 = 3.14;

// Alternative method
let x = 5u16;
let y = 3.14f32;
```

#### Boolean

The type is annotated with `bool` and the values can be:

- `true`
- `false`

Booleans are not integers unless you cast them using `true as u8` or `false as u8`.

#### Character

The character type is misnamed. Although you specify it with `char` it actually represents a single unicode scalar value, which could be any of the following:

- a character from our alphabet, such as `b`
- a character from a different alphabet, such as `Ф`
- an ideograph, such as `丈`
- a diacritic, such as `◌҃`
- an emoji, such as `🐶`
- a non-printable unicode control character, that could represent anything from a sound to an action

A character is always 4 bytes, which makes an array of characters a UCS-4 / UTF-32 string.

Character literals are written using single quotes.

But characters are relatively useless. Strings are UTF-8 and characters are not. Strings do not use characters internally. So chances are if you want to deal with a single character it'll actually be a UTF-8 string, and not a character literal.

### Compound Types

Compound types gather multiple values of other types into one type.

#### Tuple

Tuples store multiple values of any type:

```rust
let info = (1, 3.3, 999);
let info: (u8, f64, i32) = (1, 3.3, 999);
```

The are two ways to access members of a tuple:

1. Dot syntax (field access expression):
   ```rust
   let info = (1, 3.3, 999);
   let jets = info.0;
   let fuel = info.1;
   let ammo = info.2;
   ```
2. Destructuring:
   ```rust
   let info = (1, 3.3, 999);
   let (jets, fuel, ammo) = info;
   ```

Currently tuples have a **maximum arity (item count) of 12**. You can still create one beyond that, but with limited functionality.

#### Array

Arrays store multiple values of the same type. You can create them two ways:

1. Square brackets:
   ```rust
   let buf = [1, 2, 3];
   ```
2. A value and how many you want:
   ```rust
   let buf = [0; 3];
   ```

The type annotation for an array always uses the semi-colon form:

```rust
let buf: [u8; 3] = [1, 2, 3];
```

You index values in an array (e.g. `buf[0]`). Note that arrays are **limited to a size of 32**, above which they lose of their functionality. For very large collections you would typically use vectors or slices of vectors instead of arrays.

### Strings

There are 6 types of strings in the Rust standard library, but we mostly care about 2 of them that overlap each other.

A literal string is always a borrowed string slice. A borrowed string slice is often referred to as a string. This can be confusing when you learn that the other string type is `String`. The biggest different between the two is that the data in a borrowed string slice cannot be modified, while the data in a String can be.

You will often create a string by calling the `to_string` method on a borrowed string slice, or by passing a borrowed string slice to `String::from`:

```rust
let msg = "party".to_string();
let msg = String::from("party");
```

A borrowed string slice is internally made up of a pointer to some bytes, and a length. A String is made up of a pointer to some bytes, a length, and a capacity that may be greater than what is currently being used. In this sense a borrowed string slice is a subset of a String in more ways than one, which is why they share a bunch of other characteristics.

For example, both string types are valid UTF-8.

Also, they cannot be indexed by character position because strings can contain any unicode value, so you may not necessarily be able to target a character at a specific index if it is made up of multiple bytes. You can work around this by explicitly calling the `bytes` method on a string to access the vector of UTF-8 bytes which you can index into. You can use the `char` method to return an iterator that you can then use to iterate through the unicode scalars. Finally, you could use a package like `unicode-segmentation` that provides handy functions that return iterators that handle graphemes. There are also a variety of [methods](https://doc.rust-lang.org/std/string/struct.String.html) that you can use to manipulate strings.

### Control Flow

#### "If" expressions

You don't need parenthesis around expressions:

```rust
if num == 5 {
    msg = "five";
}
```

The condition **must** evaluate to a boolean because Rust does not like type coercion.

Chaining is done with `else` and `if` with a space between:

```rust
if num == 5 {
    msg = "five";
} else if num == 4 {
    msg = "four";
} else {
    msg = "other";
}
```

Because it's an expression it can be assigned as a value:

```rust
msg = if num == 5 {
    "five" // No semi-colons to make them tail expressions
} else if num == 4 {
    "four" // No "return"
} else {
    "other" // All must return the same type
}; // Semi-colon is required when using as a value
```

Ternaries do not exist; use "if":

```rust
num = if a { b } else { c };
```

#### Unconditional loops

If the compiler knows a loop is unconditional there are some good optimizations it can do.

You can break out of a loop:

```rust
loop {
    break;
}
```

You can break out of a nested loop by naming your loops:

```rust
// Note the single apostrophe
'bob: loop {
    loop {
        loop {
            // Tell break which loop you'd like to break out of
            break 'bob;
        }
    }
}
```

The `continue` keyword is similar to `break` in that you can specify which loop to target.

`while` loops have all the same behaviors as unconditional loops except they also terminate the loop when the (boolean) condition evaluates to false.

You can think of a `while` loop as syntactic sugar on top of an unconditional loop:

```rust
while keep_doing() {
    // Do something
}

// Is the same as
loop {
    if !keep_doing() { break }
    // Do something
}
```

And you can simulate do-while by moving the condition to the bottom of the block:

```rust
loop {
    // Do something
    if !keep_doing() { break }
}
```

#### For-loops

Rust's for-loop iterates over any iterable value:

```rust
for num in [7, 8, 9].iter() {
    // Do something with num
}
```

You can destructure values local to the block:

```rust
let array = [(1,2), (3,4)];

for (x, y) in array.iter() {
    // Do something with x and y
}
```

Ranges are two number separated by two dots. The first number is inclusive and the second is exclusive (unless you prefix the second number with `=`):

```rust
// Will count from 0 to 49
for num 0..50 {
    // Do something with num
}

// Will count from 0 to 50
for num 0..=50 {
    // Do something with num
}
```

## Moving and Assigning Values

### Ownership

Ownership is what makes safety guarantees possible. It makes informative compiler messages possible.

There are three rules to ownership:

1. Each value has an owner. There is no value in memory that doesn't have a variable that owns it.
2. There is only one owner. No variables may share ownership. Other variables may borrow the value, but there is only one owner.
3. Values are dropped as soon as the owner goes out of scope.

Ownership in action:

```rust
// Create a string
let s1 = String::from("abc");
// Assign it as the value of s2
// The value of s1 is not copied; it is moved
let s2 = s1;

// Trying to use s1 after it has been moved would result in an error
// "value borrowed here after move"
println!("{}", s1);
```

In the above example, if `s1` was mutable we could assign it a new value. But since it is not it is garbage.

If we wanted to instead make a copy of the value we could use the `clone` method:

```rust
let s1 = String::from("abc");
let s2 = s1.clone();
println!("{} equals {}", s1, s2);
```

Movement occurs with functions as well, when you pass variables as arguments:

```rust
let s1 = String::from("abc");
// s1 is moved to the function
do_stuff(s1);
println!("{}", s1); // Error!

fn do_stuff(s: String) {
    // Do something
}
```

To solve this we could move it back when we're done:

```rust
let mut s1 = String::from("abc");
// s1 is given to the function, and a string is returned
s1 = do_stuff(s1);
println!("{}", s1); // Works

fn do_stuff(s: String) -> String {
    // Return s as a tail expression
    s
}
```

But this is kind of gross. Passing a value to the function usually means the function is going to consume it and there is no guarantee it will come back the same. For most cases you should use references.

### References & Borrowing

References allow you to pass a value to a function while the initial variable retains ownership. Both the argument type annotation and the function call argument must be prefixed with an `&`:

```rust
let s1 = String::from("abc");
// s1 is passed as a reference
do_stuff(&s1);
println!("{}", s1); // Works

// This function only borrows the value
fn do_stuff(s: &String) {
    // Do something
}
```

References must always be valid. Rust won't let you create a reference that outlives the data that it is referencing.

References default to immutable, but can be made mutable:

```rust
let s1 = String::from("abc");
// Note the added space before s1
do_stuff(&mut s1);

fn do_stuff(s: &mut String) {
    // Do something
}
```

If you want to read from or write to the value of the reference function argument you may need to de-reference it:

```rust
fn do_stuff(s: &mut String) {
    // De-referencing is happening automatically
    s.insert_str(0, "Hi, ");
    // Manually, it would look like
    (*s).insert_str(0, "Hi, ");

    // And in cases where you need to do it yourself
    *s = String::from("Replacement");
}
```

The compiler will help you here by providing detailed compilation error messages.

## Value Groups

### Structs

In other languages you have classes, in Rust you have structs. Structs can have data fields, methods, and associated functions. The syntax for the struct and its fields is the keyword `struct`, then the name of the struct in camel case, then curly braces, and then the fields and their type annotations in a list separated by commons. As a nice touch you can use a comma on the last item in a struct and the compiler won't yell at you:

```rust
struct RedFox {
    enemy: bool,
    life: u8,
}
```

Instatiating a struct, while verbose, is straightforward. You need to specify a value for every field:

```rust
let fox = RedFox {
    enemy: true,
    life: 70,
}
```

Typically you would create an associated function as a constructor to create a struct with default values, and then call that:

```rust
// Implementation block for a struct
impl RedFox {
    // Associated function ()
    fn new() -> Self {
        Self {
            enemy: true,
            life: 70,
        }
    }
}
```

In the above there is an implementation block that is named after the struct whose functions and methods you are going to implements.

Inside it there as an associated function, because it doesn't have a form of `self` as its first parameter. In other languages this would be considered a class method. Associated functions are often used as constructors, with `new` being the conventional name you would use when you want to create a new struct with default values.

Inside the implementation block you can use `Self` in place of the struct name. You could also use the struct name, but `Self` is better.

Now let's set up a new struct instance:

```rust
let fox = RedFox::new();
```

In the above, the scope operator is double colons (`::`), which is used to access parts of namespace-like things (think back to module `use` statements). Here we're using it access an associated function of the struct.

Once instantiated you can get and set fields and call methods with dot syntax.

Methods are also defined in the implementation block. Methods always take some form of `self` as heir first argument:

```rust
impl RedFox {
    // associated function
    fn function() ...

    // methods
    fn move(self) ...
    fn borrow(&self) ...
    fn mut_borrow(&mut self) ...
}
```

### Traits

Where most languages would call "class inheritance", Rust calls "struct inheritance". But this does not exist in Rust. Why? They chose a better way to solve the problem we wish other languages solved in class inheritance, using Traits.

Traits are similar to interfaces in other languages. Rust takes the composition over inheritance approach.

```rust
// Our struct from above
struct RedFox {
    enemy: bool,
    life: u8,
}

trait Noisy {
    fn get_noise(&self) -> &str;
}
```

Traits define required behavior; functions and methods that a struct must implement if it wants to have that trait. In the above, the `Noisy` trait specifies that the struct must have a method `get_noise` that returns a borrowed string slice if it wants to be Noisy. Here's how you would implement this:

```rust
// Implement trait for struct
impl Noisy for RedFox {
    // Implement the method that returns the borrowed string
    fn get_noise(&self) -> &str { "Woof" }
}
```

Why didn't we just implement the method directly on the struct's main implementation? Because once we have a trait involved we can start writing generic functions that accept any value that implements the trait:

```rust
// This function takes an item of type T, which can
// be anything that implements the Noisy trait
fn print_noise<T: Noisy>(item: T) {
    // Now the function can use any behavior on item that
    // the Noisy trait defines
    println!("{}", item.get_noise());
}
```

As long as one of either the trait or the struct is defined in your project you can implement any trait for any struct:

```rust
fn print_noise<T: Noisy>(item: T) {
    println!("{}", item.get_noise());
}

// Implement Noisy on built-in u8 unsigned integer
impl Noisy for u8 {
    fn get_noise(&self) -> &str { "BYTE" }
}

fn main() {
    // This is an integer with the type as a suffix to the literal
    // And because it now implements Noisy we can use it as value
    // for the print_noise generic function
    print_noise(5_u8);
}
```

There is a special trait called `Copy`. If your values implements this then it will be copied instead of moved in move situations; this is useful for small values that fit entirely on the stack. If the type uses the heap at all then it cannot implement this trait.

While structs do not implement inheritance, traits _do_ implement inheritance. A trait can inherit from another trait, so inheritance hierarchy is possible. Making your trait inherit from the parent trait really just means that anyone who implements the trait must also implement the parent traits as well.

Traits can also have default behaviors, so if you design structs and traits carefully enough you might not have to implement some of the methods at all. To implement default trait behavior, inside of a trait definition instead of ending your function or method definition with a semi-colon add a block with your default behavior. Then when you implement the trait for your struct just don't provide a new definition for the method whose default implementation you want to use:

```rust
trait Run {
    // Trait with default implementation block
    fn run(&self) {
        println!("I'm running!");
    }
}

// Empty Robot struct
struct Robot {}
// Implementing the Run trait for the Robot struct, but not
// re-implementing `run` to allow it to use the default
impl Run for Robot {}

fn main( {
    let robot = Robot {};
    robot.run();
})
```

Traits cannot currently define fields. This may change down the road. The workaround for now is to define setter and getter methods in your traits.

### Collections

All available in the standard library.

#### Vec<T>

A vector is a generic collection that holds a bunch of one type, and is useful where you would use lists or arrays in other languages. It's the most commonly used collection.

When you create a vector you specify the one type of object it will store. Once you set it up you can push to and pop it. Since vectors store objects of known size into memory, you can index into it as well.

```rust
let mut v: Vec<i32> = Vec::new();
v.push(2);
v.push(4);
v.push(6);
let x = v.pop(); // x is 6
println("{}", v[1]); // prints "4"
```

There is a macro that makes creating vectors from literal values much more ergonomic:

```rust
let mut v = vec![2, 4, 6];
```

Vectors have a lot of [methods](https://doc.rust-lang.org/std/vec/struct.Vec.html).

#### HashMap<K, V>

HashMaps are a generic collection where you specify the key and the value types. In some languages this would be called a dictionary. The main purpose is to be able to look up an item by key.

In a HashMap you specify the type of the key and the type of the value. Once you have a HashMap you can insert, remove, and many [other things](https://doc.rust-lang.org/std/collections/struct.HashMap.html).

```rust
let mut h: HashMap<u8, bool> = HashMap::new();
h.insert(5, true);
h.insert(6, false);
let have_five = h.remove(&5).unwrap();
```

#### Other important collections...

- VecDeque uses a ring buffer to implement a double-ended queue which can efficiently add items from the front or back, but is not great for other operations.
- LinkedList can quickly add or remove items at an arbitrary point in a list.
- HashSet is a hashing implementation of a Set that performs Set operations really efficiently.
- BinaryHeap is like a priority queue which always pops off the max value.
- BTreeMap and BTreeSet are alternate map and set implementations using a modified binary tree. You usually only choose these over the has variants if you need the map keys or set values to always be sorted.

### Enum

Enums in Rust are more like Algebraic Data Types than C-like enums. Specify an enum with the keyword `enum`, the name of it in camel-case, and the items comma-separated in a block:

```rust
enum Color {
    Red,
    Green,
    Blue,
}

let color = Color::Red;
```

The real power of enums in Rust comes from associating data and methods with the variants:

```rust
enum DispenserItem {
    // Named variant with no data
    Empty,
    // A single type of data
    Ammo(u8),
    // A tuple of data
    Things(String, i32),
    // An anonymous struct of data
    Place {x: i32, y: i32},
}
```

You can implement functions and methods for enums:

```rust
impl DispenserItem {
    fn display(&self) { }
}
```

You can also use generics. `Option` is a generic enum in the standard library that uou will use all the time. It represents when something either absent or present; if you're trying to reach for null or nil values you'll probably want to use Option in Rust.

```rust
enum Options<T> {
    Some<T>,
    None,
}
```

Because enums can represent all sorts of data you need to use patterns to examine them. If you want to check for a single variant you use the `if let` expression.

```rust
// if-let takes a pattern that will match one of the variants
// If the pattern does match then the condition is true and the
// variables inside the pattern are created for the scope of
// the if-let block.
if let Some(x) = my_variable {
    println!("value is {}", x);
}
```

This is great if you only care about one variant matching or not, but not as great if you care about all the variants at once. Int hat case you'll use the `match` expression:

```rust
match my_variable {
    Some(x) => {
        println!("value is {}", x);
    },
    None => {
        println!("no value");
    }
}
```

Match expressions require you to write a branch for every possible outcome (exhaustive), but `_` can be used as a default/anything else branch.

Note that while you will often see blocks used as a branch arm, any expression will do, including things like function calls and care values:

```rust
match my_variable {
    Some(x) => x.squared() + 1,
    None => 42,
}
```

All branch arms need to return nothing or they must return the same type.

Two important enums are [Option](https://doc.rust-lang.org/std/option/enum.Option.html) and [Result](https://doc.rust-lang.org/std/result/enum.Result.html).

## Final Bits...

### Closures

You'll encounter closures when you want to spawn a thread, or when you want to do some functional programming with iterators, and some other places in the standard library.

A closure is an anonymous function that can borrow or capture some data from the function it is nested in. The syntax is a parameter list between two pipes without type annotations followed by a block. This creates an anonymous function you can call later. The types of the arguments and the return value are inferred by how you use them and what you return.

```rust
let add = |x, y| { x + y };

add(1, 2); // => 3
```

You don't have to have parameters, you could leave the list empty (`|| => { ... }`). Closures will borrow a reference to values in the enclosing scope:

```rust
// Create s
let s = "hi".to_string();
// Create a closure that borrows a reference to s
let f = || {
    println!("{}", s);
}

f(); // prints "hi"
```

Closures are very handy for vectors. Just call `iter` and a bunch of methods that use closures will be available to you:

```rust
let mut v = vec![2, 4, 6];

v.iter()
    // multiply values by 3
    .map(|x| x * 3)
    // filter out any values less than 10
    .filter(|x| *x > 10)
    // sum the values together
    .fold(0, |acc, x| acc + x);
```

### Threads

Threads are fully cross-platform supported.

Basic example:

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(move || {
        // do stuff in a child thread
    });

    // do stuff simultaneously in the main thread

    // wait until thread has exited
    handle.join().unwrap();
}
```

Also check out [async-await](https://blog.rust-lang.org/2019/11/07/Async-await-stable.html) which is a much more efficient way to wait for things.
