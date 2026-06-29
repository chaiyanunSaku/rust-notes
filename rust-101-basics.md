# Rust 101 — Core Basics

## 1. Variables
- `let x = 5;` → "let x be 5"
- Variables are **immutable by default** (can't change).
- Use `mut` to allow changes:
```rust
  let mut x = 5;
  x = 6; // now OK
```
- Why? Most variables shouldn't change. Making "changeable" opt-in prevents bugs.

## 2. Types (what kind of value)
```rust
let age: i32 = 25;      // i32 = 32-bit integer (whole number)
let price: f64 = 9.99;  // f64 = 64-bit float (decimal)
let happy: bool = true; // bool = true/false
let letter: char = 'A'; // char = single character (single quotes)
```
- `: i32` is a **type annotation** (telling Rust the type).
- **Type inference**: Rust can usually guess, so you can skip it:
```rust
  let age = 25;     // inferred as integer
  let price = 9.99; // inferred as float
```
- Integer types: `i8 i16 i32 i64` (signed, can be negative),
  `u8 u16 u32 u64` (unsigned, 0 or positive — u = unsigned).
- Mostly you'll use `i32`, `u32`, `usize` early on.

## 3. Two String Types (famous gotcha)
```rust
let a: &str = "hello";              // string slice: fixed, borrowed view
let b: String = String::from("hi"); // String: owned, growable text
```
- `String` = text you **own** and can grow/edit (your editable document).
- `&str` = a **read-only view** into text someone else owns (a window).
- Anything in `"quotes"` is a `&str`.
- Make a `String` when you need to modify/build up text.

## 4. Functions
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}
```
- `fn` = define function
- `(a: i32, b: i32)` = parameters WITH types (always required)
- `-> i32` = return type
- **Last expression with NO semicolon = the return value.**
```rust
  fn add(a: i32, b: i32) -> i32 { a + b }        // idiomatic
  fn add(a: i32, b: i32) -> i32 { return a + b; } // same thing
```
- A semicolon throws the value away → would break the return.

## 5. struct — bundle related data (AND)
```rust
struct Person {
    name: String,
    age: u32,
}

let alice = Person {
    name: String::from("Alice"),
    age: 30,
};
println!("{} is {}", alice.name, alice.age); // access with dot
```
- A struct groups values that belong together.
- A Person has a name AND an age.

## 6. enum — one of several choices (OR)
```rust
enum Direction {
    North,
    South,
    East,
    West,
}
let heading = Direction::North;
```
- A value is EXACTLY ONE choice from a fixed list.
- **struct = AND, enum = OR** (key contrast).

Variants can carry data:
```rust
enum Shape {
    Circle(f64),          // carries radius
    Rectangle(f64, f64),  // carries width, height
}
let c = Shape::Circle(2.0);
let r = Shape::Rectangle(3.0, 4.0);
```

## 7. match — react to which variant
```rust
fn describe(d: Direction) -> &str {
    match d {
        Direction::North => "up",
        Direction::South => "down",
        Direction::East  => "right",
        Direction::West  => "left",
    }
}
```
- Like a supercharged if/else chain.
- Rust **forces you to handle every case** (won't compile otherwise).

## 8. Two built-in enums you'll use constantly

### Option<T> — "maybe a value, maybe not" (replaces null)
```rust
enum Option<T> {
    Some(T), // value present
    None,    // nothing
}

fn first_char(s: &str) -> Option<char> {
    s.chars().next() // Some('h') for "hi", None for ""
}

match first_char("hi") {
    Some(c) => println!("first: {}", c),
    None => println!("empty!"),
}
```
- Makes "might be missing" visible in the type → no surprise null crashes.

### Result<T, E> — "success or error" (replaces exceptions)
```rust
enum Result<T, E> {
    Ok(T),  // success
    Err(E), // failure
}
```
- Functions that might fail return Result; you match to handle both.

## 9. Generics — the <T>
- `T` = placeholder for "some type — you decide which."
```rust
fn biggest<T: Ord>(a: T, b: T) -> T {
    if a > b { a } else { b }
}
biggest(3, 7);     // T = i32
biggest('a', 'z'); // T = char
biggest(1.5, 2.5); // T = f64
```
- Write the function once, works for many types.
- `T: Ord` means "any type T that can be ordered (compared with >)".
- So `Option<i32>` = maybe-an-integer, `Option<String>` = maybe-a-string.

---

## One-line summary
Variables + `mut` · types + inference · `String` vs `&str` ·
functions (no-semicolon return) · **struct = AND, enum = OR** ·
`match` (handle every case) · `Option`/`Result` · generics `<T>`.


```rust 
let mut m = 'h'
```

hi