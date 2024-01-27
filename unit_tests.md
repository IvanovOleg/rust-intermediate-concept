# Unit Tests
Testing of small units of code.

Let's say this function is in your lib.rs file and you want to make sure it works correctly by writing
some unit tests. Where do your unit tests go? The idiomatic place for your tests to go is in their
own submodule at the bottom of the same file as the code you are testing.
The convention is to put this submodule inline by putting curly braces right after the module and
defining it immediately, rather than making a separate file for your submodule.
The submodule is typically called test.

**lib.rs:**
```rust
pub fn snuggle(bunnies: u128) -> u128 {
    bunnies * 8
}

#[cfg(test)]
mod test {
    use super::*; // brings everything into the scope

    #[test] // This attribute tells cargo that the function it is applied to should be run by the test runner.
    fn snuggling_bunnies_multiply() {
        assert_eq!(snuggle(2), 16);
    }
}
```
We only want to compile and run the tests locally or in CI.
So we'll use the config attribute to tell Rust
we only want to compile this module when we're running tests.
cfg is pronounced config. The config attribute controls conditional compilation of the item it applies to.

The function can be private. It can be named whatever you want, and the function should have no parameters and return nothing or
a Result.

### Assert Macros
```rust
assert_eq!(5, 6); // assert_eq takes two arguments of the same type that implement the PartialEq trait. Fails if they are not equal.
assert_ne!(5, 5); // assert_ne takes two arguments of the same type that implement the PartialEq trait. Fails if they are equal.
assert!(5 >= 5); // covers all other scenarios, expect a boolean value
panic!("It is time to crash!"); // panic! fails the test

#[should_panic]
panic!("It is time to crash!"); // panic! it doesn't fail the test
```

In order to run test, you execute:
```shell
cargo test
```

### Two Definitions of Crate

1. crate = package
2. crate = binary or library

Crate as a package can include 1 or 0 libraries and any number of binaries. Every binary is a crate itself.


### Doc-Tests
Cargo will search out tests in your library documentation and run them, but only
for doc-tests in your library. Ignores binaries.

Example of the testing code snippet in the library documentation:
```rust
/// Example
/// 
/// ```
///  # use hello::snuggle;
/// let bunnies = snuggle(5);
/// asser_eq!(bunnies, 40);
/// ```
pub fn snuggle(bunnies: u128) -> u128{
    bunnies * 8
}
```
Doc-tests are typically in the form of documented examples, so the convention is to put them in an
"Example" section and then put the actual code down in a section fenced by triple backticks.
Inside the example, we write code as if it were inside of a test function in a binary outside of the library.
Hash or pound sign surrounded by spaces before the use statement?
This is how you hide code that's necessary for the test to run, but you don't want it displayed in
your documentation. So this documentation will show up like this.

### Unit Tests Result
Unit tests can also optionally return a Result.
```rust
#[test]
fn bunny_result() -> Result<(), ParseIntError> {
    let num_bunnies: u64 = "four".parse()?;
    assert_eq!(num_bunnies, 4);
    Ok(())
}
```
Why would you want to return a Result from a test? So you can use the question mark operator in the test,
of course! At the end of this test, you'll need to remember to return your Ok value.
The built-in parsing can't actually pass the word four into a number.

Doc-test will not run if unit tests fail (fail fast).

```rust
cargo test test::bunny_result // runs only a specific test
```