# Documentation
Documentation is good when it is readable.
```rust
cargo doc
```
To see all possible options:
```rust
cargo doc -h
```
To generate a documentation without external dependencies and open a site use:
```rust
cargo doc --no-deps --open
```
Otherwise you will have to open *target/doc/packagename/index.html*

## Outer Documentation
```rust
/// Number of pieces in the puzzle
pub const PUZZLE_PIECES: u32 = 42;
```
Since constant is public, it's documentation will be included in the public website. By default a documentation for the private stuf is not generated, but it can be overridded by passing an argument to the cargo doc command. Only first paragraph will be displayed on the index page. Keep the most important information there.

```rust
/// Number of pieces in the puzzle
/// 
/// This is a separate paragraph
pub const PUZZLE_PIECES: u32 = 42;
```

Rust documentation supports markdown, same as github.com:
```rust
/// Number of pieces in the puzzle
/// 
/// # History
/// 
/// This is a separate paragraph
///  - Clickable Link: [`PUZZLE_PIECES`]
/// We tried `7`, but this is better. 
pub const PUZZLE_PIECES: u32 = 42;
```

Rust documentation should always start from the description and then you may put extra sections.

Here is a block documentation example with an intradoc link:
```rust
/** Number of pieces in the puzzle

# History

This is a separate paragraph
- Clickable Link: [`PUZZLE_PIECES`]
We tried `7`, but this is better.
**/
pub const PUZZLE_PIECES: u32 = 42;
```

Another way of defying an intradoc link:
```rust
/** Number of pieces in the puzzle

# History

This is a separate paragraph
- [Clickable Link](PUZZLE_PIECES)
We tried `7`, but this is better.
**/
pub const PUZZLE_PIECES: u32 = 42;
```

For things out of the scope, we need to use an absolute path to it:
```rust
/// [Spawn a thread](std::thread::spawn)
```

## Inner Documentation
Inner documentation is useful for something inside the module or a library and has the same rules, but different blocks:
```rust
//! Inline

/*!
block
!*/
```
Example:

```rust
//! Hi! I'm your friendly Rust Puzzle
//! Library documentation. Please
//! come in, sit down, and have a cup
//! of tea!
```

Documentation of the struct will go to the main section, documentation for fields will go a separate sections below.
```rust
/// This is a puzzle!
pub struct Puzzle {
    /// Number of pieces
    pub num_pieces: u32,
    /// Descriptive name
    pub name: String,
}
```

We usually don't document the implementation block itself, but we document associated functions:
```rust
impl Puzzle {
    /// Make a new puzzle!
    pub fn new() -> Self {
        Self {
            num_pieces: NUM_PIECES,
            name: "Forest Lake".into(),
        }
    }
}
```