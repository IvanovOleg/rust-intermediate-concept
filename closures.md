# Closures
A closure is an anonymous function that can borrow or capture some data from the scope it is nested in.
The syntax for a closure is a parameter list between two pipes, usually without type annotations, followed by an expression.

```rust
|params| expr
|params| { expr1; expr2 }
```

That expression will often be a block, which contains multiple of its own expressions. This creates a closure or anonymous function that you can assign to a variable pass around and call later. The compiler figures out the types of the arguments and the type of the return value by how you use the closure.

```rust
let add = |x, y| x + y;
add(1, 2) // returns 3
```
Those blocks are optional:

```rust
|| x + y;
// or
|| {};
```

A closure will automatically borrow a reference to values in the enclosing scope.

```rust
let h = "❤️";

let h = || {
    println!("{}", h); // we don't pass "h", but it gets borrowed automatically
}

f(); // prints ❤️
```
This borrowing-by-default behavior of the closure is great as long as the closure gets dropped before the value it is referencing gets dropped. This won't compile if the compiler can't guarantee that the value will live longer than the closure
that is referencing it. Lucky for us, closures also support move semantics by adding the keyword move before the closure.

```rust
let h = "❤️";

let f = move || {
    println!("{}", h); // we don't pass "h", but it gets borrowed automatically
}

f(); // prints ❤️
```

This will make the closure move any variables it uses into itself and take ownership of them. Now "h" is owned by the closure, and will only get dropped when the closure itself goes out of scope and gets dropped. So we could send this closure over to another thread or return it from a function without needing to worry about when "h" gets dropped.

Of course, this also means you no longer have access to "h" in the outer scope. If you still need that value outside the closure as well,
then another option is to make a clone of "h" and pass the cloned value to the move closure instead.

```rust
let h = "❤️";
let h2 = h.clone();

let f = move || {
    println!("{}", h2); // we use a cloned value
}

f(); // prints ❤️
```
Now the closure has its own clone of the heart, and we can still use "h" outside of the closure.

If you want to pass closures to or from functions, then you will need to specify the type of the closure in terms of one of these three traits from standard ops.
```rust
use std::ops;

fn
fnMut
fnOnce
```