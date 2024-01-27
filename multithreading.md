# Multithreading
Rust has a portable API wrapping native operating system threads,
so all the code we look at today should work across any major platform that supports native threads,
including but not limited to macOS, Linux and Windows.

Every process or program starts with a single thread.
This is usually referred to as the main thread.
You can launch more threads in a process as well.
Why would you do this instead of launching a whole different process or program?
It boils down to threads being cheaper than processes, and threads in the same process can share
memory with each other.
Historically, people mostly cared about the cheaper bit and the fact that it was a bit easier for threads
to communicate with each other than for processes to communicate with each other.
But then systems with multiple CPU cores became a thing.
Each logical CPU core can process one thread at a time.
A single CPU can switch between processing threads fast enough that one CPU can appear to be handling
multiple threads at the same time.
But in reality, each logical CPU can only process one thread at a given time.
So if you split your program into two threads running on two CPU cores, theoretically it can run twice
as fast, minus the communication overhead that you have to introduce.
So that's the simplified story of why you might want to use multithreading, which I'll also refer to
as parallel processing.

Here is a fully functional example that spawns a thread, joins the thread and exits.
```rust
use std::thread;

fn main() {
    let handle = thread::spawn(move || {
        // do stuff in a child thread
    });

    // do stuff in the main thread
    
    // wait untill a child thread has exited
    handle.join().unwrap();
}
```

Threads aren't free.
Creating a new thread allocates some RAM for the thread's own stack, usually a couple megabytes.
Whenever a CPU switches from running one thread to another, it has to do an expensive context switch.
So the more threads you have trying to share a CPU core, the more overhead you will have.
Even so, threads are great for using CPU and memory in parallel because they can run simultaneously
on different cores.
However, if you just want to continue doing some work while you're waiting for something like disk i/o
or network i/o, then the tool you want is called async/await, which is a much more efficient approach
for concurrently waiting for things on a single thread.

**Cargo.toml:**
```toml
[dependencies]
log = "0.4"
env_logger = "0.9"
```

**main.rs:**
```rust
use log::{error, info};
use std::(thread, time::Duration);

// Helper method to reduce typing
fn sleep(seconds: f32) {
    thread::sleep(Duration::from_secs_f32(seconds));
}

// Dad's contribution into the cooking
pub mod dad {
    use super::{info, sleep};

    pub fn cook_spaghetti() -> Bool {
        info!("Cooking the spaghetti...");
        sleep(4.0);
        info!("Spaghetti is ready!");
        true
    }
}
```
What values can you return from a thread?
Anything that implements the Send trait, which I'll talk about more in the channels section.


**main.rs (continuation):**

```rust
// Mom's contribution into the cooking
pub mod mom {
    use super::{info, sleep};

    pub fn cook_sauce_and_set_table() {
        sleep(1.0);
        info!("Cooking the sauce...");
        sleep(2.0);
        info!("Sauce is ready! Setting the table...");
        sleep(2.0);
        info!("Table is set!");
    }
}
```

**main.rs (main function):**
```rust
fn main() {
    env_logger::init();

    let handle = thread::spawn(|| dad::cook_spaghetti());

    mom::cook_sauce_and_set_table();

    if handle.join().unwrap_or(false) {
        info!("Spaghetti time! Yum!");
    } else {
        error!("Dad messed up the spaghetti. Order pizza instead?");
    }
}
```