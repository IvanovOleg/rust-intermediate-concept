# Common Traits
What can implement a trait?
* struct
* enum
* closure
* function

## Derivable Traits

### Debug
```rust
#[derive(Debug)]
```
The trait can be derived if there is a derive attribute defined for it. The documentation for the trait can usually tell you if it can me derived.
One of the most common traits to derive is **Debug**. As long as everything in your struct or enum is **Debug**, you can derive **Debug** for your struct or enum. All primitives and most common types in the library are already Debug.
Debug trait is used for the debug formatting and pretty debug formatting.
```rust
println!("{:?}", puzzle); // Debug
println!("{:#?}", puzzle); // Pretty debug
```

```rust
#[derive(Debug)]
pub struct Puzzle {
    pub num_pieces: u32,
    pub name: String,
}

println!("{:?}", puzzle); // debug example: Puzzle { num_pieces: 500, name: "Draconic Equestarian" }
println!("{:#?}", puzzle); // pretty debug example
// Puzzle {
//     num_pieces: 500,
//     name: "Draconic Equestarian"
// }
```

### Clone
Implementing the **clone** trait allows your value to be cloned using the **.clone()** method. As long as everything in your struct or enum is **Clone**, you can derive **Clone** for your struct or enum.

```rust
#[derive(Debug, Clone)]
pub struct Puzzle {
    pub num_pieces: u32,
    pub name: String,
}

let puzzle2 = puzzle.clone(); // clones the value of the puzzle into the puzzle2
```

### Copy
If your type implements copy then it will be copied instead of moved in the move situation. This makes sence for a small values that sit entirely in the stack. That's why all primitive types are Copy. If a type uses a heap at all, it can not implement Copy.
We can't implement copy here since String is not a Copy.
```rust
#[derive(Debug, Clone)]
pub struct Puzzle {
    pub num_pieces: u32,
    pub name: String,
}

#[derive(Clone, Copy)] // copy is a subtrait of clone, that's why we need to derive clone as well
pub enum PuzzleType {
    Jigsaw,
}
```
Tiny structs or enums with a trivial amount of data a perfect candidates to derive copy since it is faster than deal with references and move semantics.

## Traits You Implement
### Three steps to manually implement a trait:
1. Bring a trait to the scope
2. Boilerplate (use ide to generate or find an implementation example in docs of the trait)
3. Implementation

### Default

```rust
#[derive(Default)] // gives you zero values or empty collections and is not useful
```
step 2
```rust 
impl Default for Puzzle {
    fn default() -> Self {
        todo!() // get's translated to panic!("not yet implemented!");
    }
}
```

step 3

```rust
impl Default for Puzzle {
    fn default() -> Self {
        Puzzle {
            num_pieces: PUZZLE_PIECES,
            name: "Forest Lake".to_string(),
        }
    }
}
```

Default is very useful when you need to customize a part of the struct and leave the rest with default values:

```rust
let puzzle = Puzzle {
    num_pieces: 3000, // customized field
    ..Default::default() // struct update syntax using a range operator ".." brings the rest of fields as default
}
```

### PartialEq / Eq
PartialEq is a trait that does actual calculations to test for equality. Eq is a marker trait that you can implement
if equality logic is reflexive, transitive and symmetric.
If every possible value is not equal to itself, then the type cannot have the Eq marker trait.
For example floats have:

```rust
NaN != NaN // not a number is not equal to itself
```

An example of the boilerplate for a PartialEq:
```rust
impl PartialEq for Puzzle {
    fn eq(&self, other: &Self) -> bool {
        todo!()
    }
}
```

Implementation example:
```rust
impl PartialEq for Puzzle {
    fn eq(&self, other: &Self) -> bool {
        (self.num_pieces == other.num_pieces) && (self.name == other.name)
    }
}

impl Eq for Puzzle {}
```
Implementing Eq allows us to use Puzzle as a key in the HashMap and a couple of other edge cases like that.
Mostly you will be checking for equality using the **PartialEq** trait.


If we drom the Eq implementation, then we can update our PartialEq and basically define what equals means like this :

```rust
impl PartialEq for Puzzle {
    fn eq(&self, other: &Self) -> bool {
        (self.num_pieces == other.num_pieces) && (self.name.to_lowercase() == other.name.to_lowercase()) // this ignores a case in letters
    }
}
```

### From / Into
If you implement **From** then **Into** gets automatically implemented for you. But it is more idiomatic to use **Into**. So you need to implement **From** to use **Into**.

```rust
From<T> for U
Into<U> for T
```
Both describe the same transformation but from different perspectives.
```rust
From<Puzzle> for String
Into<String> for Puzzle
```

Boilerplate:
```rust
impl From<Puzzle> for String {
    fn from(puzzle: Puzzle) -> Self {
        todo!()
    }
}
```

Implementation:
```rust
impl From<Puzzle> for String {
    fn from(puzzle: Puzzle) -> Self {
        puzzle.name // returs just name as a String, other iformation gets dropped
    }
}

let puzzle = Puzzle::default();
let s = String::from(puzzle);
// or
let t: String = puzzle.into();
```

It is more idiomatic to use **.into()** with generic functions:
Function takes an argument "s" of type "T" which is defined as anything that implements **Into\<String>** trait, then we print that argument by calling
it's **.into()** method.
```rust
pub fn show<T: Into<String>>(s:T) {
    println!("{}", s.into());
}

let puzzle = Puzzle::default();
show(puzzle); // puzzle has been consumed 8(
```

To avoid consumption we can use an immutable reference instead:
```rust
impl From<&Puzzle> for String {
    fn from(puzzle: &Puzzle) -> Self {
        puzzle.name.clone(); // we clone just a name that is enough for our method
    }
}

pub fn show<T: Into<String>>(s:T) {
    println!("{}", s.into());
}

let puzzle = Puzzle::default();
show(&puzzle); // puzzle has NOT been consumed 8)
```