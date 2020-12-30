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
