# Integration Tests
The first thing we need is a tests directory. All integration tests go in this directory, which is at the root of the project.
So if we take a look, we have the src directory with our library that we just made, and now we
have a separate tests directory at the same level as src.
```shell
mkdir tests
touch tests/anything.rs
```

**tests/anything.rs:**
```rust
use hello::snuggle;

#[test]
fn it_works_from_outside() {
    assert!(snuggle(4) == 32));
}
```

Convention is to put as much as possible into the library and as less as possible into the binary.
Then the only thing you're binary does is maybe handle command line arguments and then call into your library.
The idea is: your binary will be so small that you won't need to test it.
But if you really want to test it anyway, the typical way to do it is by using **std::process::Command**
to execute your compiled binary in a subprocess and then figure out how to check if it worked.

```rust
use std::process::Command;

let output = if cfg!(target_os = "windows") {
    Command::new(cmd)
        .args(&["/C", "echo hello"])
        .output()
        .expect("failed to execute process")
} else {
    Command::new(sh) {
        .arg("-c")
        .arg("echo hello")
        .output()
        .expect("failed to execute process")
    }
};

let hello = output.stdout();
```
