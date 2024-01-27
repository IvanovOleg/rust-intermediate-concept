# Logging
Logging is not handled by the standard library.
However, there is an official library called **log**, which forms the foundation for all rust logging.
All libraries should use **log** for simple logging.

Cargo.toml:
```toml
[dependencies]
thiserror = "1.0"
log = "0.4"
```
There are five levels of logging, each with its own corresponding macro. In order of descending priority,
error, warn, info, debug and trace.

```rust
use log::{error, warn, info, debug, trace};

error!("Serious stuff");
warn!("pay attention!");
info!("useful information");
debug!("extra information");
trace!("All the things");
```

By the way, the exclamation mark is not part of the name of a macro.
The exclamation mark is part of the syntax for calling a macro.
So when we import these macros, no exclamation mark, but when we call these macros, we end the macro
name with an exclamation mark. 

At runtime, the log level will be set to one of these five levels.
All messages of equal or higher priority to the level will be **sent**. All lower priority
messages will be **skipped**.

```rust
warn!("pay attention!");
warn!(target: "puzzle", "pay attention!");
```
The macro itself encodes the log level, warn in this case, and requires a message argument, but can also
optionally take a first "target" argument. Macros can define their own custom syntax for what they want to handle.
The log macros require that you put target colon value
if you choose to specify this argument. You couldn't do this in a function call, but macros receive
a token stream to parse, so you can make macros that parse anything. If you don't specify the target,
then it defaults to the name of the module you're in, which is nice.

```rust
warn!("pay attention, minion {}!", 352);
```

These macros work just like println, so the error message is treated like a template string if you
follow it with one or more arguments.

**library.rs:**
```rust
use log::{error, info};

impl Puzzle {
    /// Make a new puzzle!
    pub fn new() -> Self {
        let puzzle = Default::default();
        info!("Created a puzzle with new(): {:?}", puzzle);
        puzzle
    }

    /// Load a puzzle from a file
    pub fn from_file(_fh: File) -> Result<Self, PuzzleError> {
        error!("This file is missing a piece");
        Err(PuzzleError:MissingPiece)
    }
}
```

**main.rs:**

```rust
use anyhow::{Context, Result};
use log::info;
use puzzles::Puzzle;
use std::fs:file;

fn get_puzzle(filename: &str) -> Result<Puzzle> {
    let fh = File::open(filename)
        .with_context(|| format!("couldn't open the puzzle file {}", filename))?;
    let puzzle = Puzzle::from_file(fh).context("couldn't convert data into a puzzle") {
        Ok(p) => p,
        // Err(e) => Err(e),
        Err(_) => Puzzle::new(),
    };
    println!("Playing puzzle: {}", puzzle.name);
    Ok(())
}

fn main() -> Result<()> {
    let puzzle = get_puzzle("puzzle.dat").context("Couldn't get the first puzzle")?;
    info!("Playing puzzle: {}", puzzle.name);
    Ok(())
}
```
It doesn't output anything. Why?

## Interface

The log library defines a common interface via traits that all basic loggers should conform to.
So that simple logging from different libraries and applications will all be compatible.
Think of this as the plumbing that makes it so logging is able to interconnect between various libraries
and the applications that use them.
But just like plumbing, having the pipes in the wall is not sufficient.
You also need to provide somewhere for the output to go.
That's what we're missing.
Libraries use just the log module.
Just the plumbing. Applications can use the log module as well, but they also need to choose an additional
logging library that routes the output somewhere.
This is the most important concept I can teach you about logging in Rust.
Lots of people get confused about this.
There are lots of choices for those logging output libraries, depending on what you're trying to do.
Do you just want logs to go to stdout? To syslog? To a file? To Splunk? To a cloud vendor's
service? To some custom application?

For our example, We will choose one of the simplest options: env_logger. env_logger is a super simple logger.

Cargo.toml:
```toml
[dependencies]
thiserror = "1.0"
log = "0.4"
env_logger = "0.9"
```

```rust
use anyhow::{Context, Result};
use log::info;
use puzzles::Puzzle;
use std::fs:file;

fn get_puzzle(filename: &str) -> Result<Puzzle> {
    let fh = File::open(filename)
        .with_context(|| format!("couldn't open the puzzle file {}", filename))?;
    let puzzle = Puzzle::from_file(fh).context("couldn't convert data into a puzzle") {
        Ok(p) => p,
        // Err(e) => Err(e),
        Err(_) => Puzzle::new(),
    };
    println!("Playing puzzle: {}", puzzle.name);
    Ok(())
}

fn main() -> Result<()> {
    env_logger::init();
    let puzzle = get_puzzle("puzzle.dat").context("Couldn't get the first puzzle")?;
    info!("Playing puzzle: {}", puzzle.name);
    Ok(())
}
```
env_logger uses the ***RUST_LOG*** env variable to get a log level.

### Tracing
Framework for tracing.