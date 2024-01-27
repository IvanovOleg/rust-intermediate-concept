# Handling Errors
Handling errors in an application. Previously we mostly talked about errors in scope of the
**Result** enum:
```rust
#[must_use]
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### Options we have:
1. **panic** (if error is non-recoverable)
```rust
// manual panic
!panic("Your computer is on fire");

// Same thing if a result is Result::Err
result.expect("Your computer is onm fire")

// Same thing, but without a message
result.unwrap();
```

You should never use panic in your library, unless there is no way to avoid it. Do not try to catch panics.

2. **Handle or Return** (If error is recoverable)

The "if let" expression is super useful if you care more about the error than the success value.
Remember that "if let" takes a pattern on the left side. If the pattern matches the value on the right
side, then the pattern's variables are bound for the scope of the "if let" block.
```rust
if let Err(e) = my_result {
    println!("Warning: {}", e);
}
```
So if we received an error here, then we're just printing out a warning and then continuing on with our program.
Often, though, you're going to want to supply some sort of default value to use if there was an error.
So in that case, you could use a match expression to handle both the success and error cases and make
sure they both return the same type.

```rust
lat score = get_saved_score() {
    Ok(x) => x,
    Err(_) => 0,
}
```
In this example, if we're unable to retrieve the saved score from disk, we don't even care about the error.
We just want the default score of zero.A lot of these situations can be handled by Result's helper methods, so I encourage you to read through
the documentation for Result. For example, this match expression I'm showing you here can be simplified to a call to Result's, .unwrap_or()
method, which does exactly the same thing as the match expression above.

```rust
let score = get_saved_score().unwrap_or(0);
```

3. **Return the Error**

Now what about when we don't know what to do with an error?
In that case, we should return the error to the caller to handle it.
It looks like this.

```rust
fn poem() -> Result<String, io::Error> {
    let file == match File::open("pretty_words.txt") {
        Ok(f) => f,
        Err(e) => return Err(e),
    };
    // do stuff with a file
}
```
Here, the poem function returns a result whose Ok value is a string or whose Err value is an io::Error.
Then any time we do something that might return an io::Error, we use a match expression that always
looks sort of the same.

If it's an Ok, we unwrap the value and if it's an Err, we don't know what to do with it.
So we rewrap the error in an Err variant and return it from the function.
This is super, super common. So common that Rust has a special syntax for it.
The question mark operator, which is also referred to as the try operator because it began life as
the try macro back in Rust 2015.

```rust
fn poem() -> Result<String, io::Error> {
    let file == match File::open("pretty_words.txt")?;
    // do stuff with a file
}
```

This does the exact same thing as that match expression.
If the result is an Ok, then a unwraps the inner value.
If the result is an Err variant, then it returns the error from the function.
This makes it super convenient to chain a bunch of method calls that could all return an error compatible
with the function signature.

For instance, this is a made up example of a chain of calls that each have the same sort of error value
in their Result.

```rust
optimus.stand()?.transform()?.rollout()?.chase()?
```

In the happy path each of these method calls succeeds and evaluates to a new value, which is unwrapped and gets its own
method called. The unhappy path is returning early from whatever function the statement is located in.
This looks good.

```rust
pub fn autobots_rollout() -> Result<Vehicle, TransformerError> {
    let optimus = Transformer::new();
    optimus.stand()?.transform()?.rollout()?.chase()?
}
```

We know that either one of these question marks is going to return a transformer error or the whole
chain is going to succeed and return a vehicle.

But what if you're dealing with multiple error types in your application? In a library
my advice is to handle any error that isn't your own type and convert it to a variant of your library's
error type, and you're going to need to do that with your own match expression. In your library,
you'll be manually handling other libraries' errors until they're converted to your error type and then
using the question mark operator for your own errors that are flowing up and down your libraries call stack.


Do you only get to choose one error type to use the question mark on and have to use match expressions
on all the rest of the error types?
No, we can leverage the power of traits.

```toml
[dependencies]
anyhow = "1.0"
puzzles = { path = "../puzzles" }
```

```rust
use anyhow::Result;
use puzzles::Puzzle;
use std::fs::File;

fn get_puzzle(filename: &str) -> Result<Puzzle>{
    let fh = File::open(filename)?;
}
```

This is where the magic begins.
io::Error implements the Error trait, so the anyhow result will accept it, which is convenient for us,
but the convenience doesn't stop there!

```rust
use anyhow::{Context, Result};
use puzzles::Puzzle;
use std::fs::File;

fn get_puzzle(filename: &str) -> Result<Puzzle>{
    let fh = File::open(filename).context("couldn't open the puzzle file")?;
}
```
This is sort of like how we can use the .expect() method on normal Result to crash the program with a
bit of context, only without the crashing and with more sophisticated handling.

```rust
use anyhow::{Context, Result};
use puzzles::Puzzle;
use std::fs::File;

fn get_puzzle(filename: &str) -> Result<Puzzle>{
    let fh = File::open(filename)
        .with_context(|| format!("couldn't open the puzzle file {}"))?;
    let puzzle = Puzzle::from_file(fh).context("couldn't convert data into the puzzle")?;
    Ok(puzzle)
}

fn main() -> Result<()>{
    let puzzle = get_puzzle("puzzle.dat").context("couldn't get the first puzzle")?;
    println!("Playing puzzle: {}", puzzle.name);
    Ok((())
}

```

Let's say we want more than just this hardcoded context. Can do! Using the **with_context()** method
we can pass in a closure that does something fancier!
The closure will only be called if an error occurs, so we avoid the overhead of allocating and formatting
the string in the success case.

Next, let's use that file handle to read in the data and convert it into a Puzzle.
I didn't show you the **from_file()** method before, but it takes a file handle and returns a normal Result
with either a Puzzle or a PuzzleError that we just spent all that time defining in the last section.

We've got two different errors in play here: an io::Error and a PuzzleError.
In both cases, we don't know what to do with them in this function, so we're passing them up to the
caller to decide what to do. Finally, if everything was successful, we just return Ok with our Puzzle.

Things can go wrong, but if they do, we just pass the error up the stack for someone else to worry about.
In this case, it's going to be our main function.
The main function can also optionally return a Result because the result implements the Termination trait.