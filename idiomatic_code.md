# Idiomatic code
## adjective
1. expressions that are natural for a native speaker
2. appropriate to a style of the particular group

## rustfmt
A tool that reformats our code
```rust
cargo fmt
```

Poor code:
```rust
struct Thing {field: i32}
enum
Choice {
A, B, C
}
fn main()
{
println!("Hello World!");
}
```

After cargo fmt:
```rust
struct Thing {
    field: i32,
}
enum Choice {
    A, B, C,
}
fn main() {
    println!("Hello World!");
}

```
You can customize how rustfmt works by putting the **.rustfmt.toml** file at the root of your project.

## clippy
Compiles your project and checks for problems
```rust
cargo clippy
```
Categories of problems:
1. Style (syntactic check and idiomatic check)
2. Correctness (warns if code compiles, but is written wrong or useless)
3. Complexity (gives warnings to simplify your complex code)
4. Performance


### How to ignore warnings?
You need to find out what warning you get, then you need to add this attribute right outisde a function for example
```rust
#[allow(clippy::too_many_arguments)]
fn main() {
    ...
}
```