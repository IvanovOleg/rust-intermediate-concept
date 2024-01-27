# Creating Errors
Things to remember when creating errors for your public library:
### 1. Errors should be **enum**

```rust
pub enum PuzzleError {
    WontFit(u16),
    MissingPiece,
}
```
### 2. Group your errors as varians of a few enums as possible
### 3. You should only return **your** errors from your public library
```rust
// Do not return the other library errors
// pub enum FractalError {
//     SadSnowflake,
// }

// Convert them in your library errors
pub enum PuzzleError {
    WontFit(u16),
    MissingPiece,
    PrettyImageless,
}

// The only exception is a standard library errors that sometimes makes sence to pass f.e
// IO errors may be better to pass
```
### 4. Your enum should be non-exhaustive
```rust
#[non_exhaustive] // Force user to define a wildcard in the match expressions
pub enum PuzzleError {
    WontFit(u16),
    MissingPiece,
}
```
The non-exhaustive attribute makes it so you can't do a match expression without a wildcard, so this
match expression wouldn't compile. This is a good thing because if we allowed matching without a wildcard,
then any time we added a new error variant, all the code that looked like this would break for our users.
So a user of our library is forced to always include a default arm for their match expressions just
in case there are new variants in the future.

This will fail:
```rust
match error {
    PuzzleError::WontFit(_) => {}
    PuzzleError::MissingPiece => {}
    // fails since there is no default variant
}
```
This will work:

```rust
match error {
    PuzzleError::WontFit(_) => {}
    PuzzleError::MissingPiece => {}
    _ => {}
    // works since there is a default variant
}
```

### 5. Implement Debug + Display + Error

You have to implement Debug and Display to be able to implement Error.
```rust
pub trait Error: Debug + Display
```

Debug:

```rust
#[derive(Debug)] // #5 Debug + Display + Error
#[non_exhaustive] // #4 non-exhaustive
pub enum PuzzleError { // #1 enum
    WontFit(u16), // #2 group
    MissingPiece, // #3 only our errors
}
```

Display:

```rust
use std::fmt::{Display, Formatter};

impl Display for PuzzleError {
    fn fmt(&self, f: &mut Formatter) -> std::fmt::Result {
        use PuzzleError::*;
        match self {
            MissingPiece => write!(f, "Missing a piece"),
            WontFit(n) => write!(f, "Piece {} doesn't fit!", n),
        }
    }
}
```

Error:
```rust
use std::error::Error;

impl Error for PuzzleError {}
```

### 5b. Use thiserror
Cargo.toml:
```toml
[dependencies]
thiserror = "1.0"
```

main.rs:

```rust
use thiserror::Error;

#[derive(Debug, Error)] 
#[non_exhaustive]
pub enum PuzzleError {
    // #[error("Piece {piece_index} doesn't fit")]
    // WontFit(piece_index: u16),
    #[error("Piece {0} doesn't fit")]
    WontFit(u16),
    #[error("Missing a piece")]
    MissingPiece,
}
```