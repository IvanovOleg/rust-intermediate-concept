# Channels
Now let's talk about how to communicate between threads with channels.
The standard library has a channel implementation in the std::sync::mpsc module that was initially
created by the Rust core developers for use in the part of Firefox that's called Servo.
I don't recommend using it.

But the Rust core team won't break standard library compatibility, so this has been stuck there ever since.
But they kept moving forward and reimplemented channels from scratch with a much better design in the
crossbeam library, so use crossbeam channels instead.
They're faster, more efficient and have way more features.

What is a channel anyway?
A channel is a one way queue that threads can use to send one type of value to another thread.
To be able to send a value across a channel.
It needs to satisfy the Send trait, which I mentioned in the multithreading section.
Send is a marker trait, much like Copy or Eq.
Only instead of implementing it yourself, the compiler will automatically implement it for anything that
will be safe to send between threads.
All primitives are Send. In fact, most standard library types are Send.

## Bounded Channel
A bounded channel has a fixed capacity.
Once the channel is full, the sending thread will block if it tries to send another value through the channel.
That sending thread will resume once a receiver pulls something off of the channel.
This is a great way to apply backpressure and make sure that one of your threads isn't generating more
work for others than they can handle.

```rust
channel::bounded(8);
```
Thread B -> 0 1 2 3 4 5 6 7 -> Thread Y

## Unbounded Channel
The other flavor of channel is the unbounded channel, which will allow the size of the channel to grow
indefinitely--well, until you run out of memory and crash anyway.
This type of channel is great if you have bursty loads, but you're confident they'll never be so high
that you'll run out of memory.

```rust
channel::unbounded();
```
Thread B -> ... -> Thread Y


Both types of channels can have multiple receivers.
Only one receiver will get the value that was sent.
Which receiver will get
the value is undefined.
Channels can also have multiple senders.
Whichever sender tries to send first gets its item into the channel first.
And finally, you can have multiple senders and multiple receivers at the same time, but the flow only
goes in one direction. If you want bidirectional communication, then you can use multiple channels, but you'll have to be
very careful with your design if you do this.
If your channel flow is cyclical, you have the potential to deadlock.
For example, if each of the threads in this diagram sent values fast enough to fill up the bounded
channels and then blocked waiting for the channel to regain capacity, then neither thread could get
unblocked to receive something off of the receiving end of the other channel.
So it's a good idea to keep your channels flowing in a directed acyclic graph so you don't have to
worry about this sort of situation at all.

**Cargo.toml:**
```toml
[dependencies]
crossbeam = "0.8";
```

**main.rs:**

```rust
use crossbeam::channel::{self, Receiver, Sender};
use std::{thread, time::Duration};

#[derive(Debug)]
enum Lunch {
    Soup,
    Salad,
    Sandwich,
    HotDog,
}
```

Receiver and sender are traits that receiving and sending ends of the channel implement.

```rust
fn cafeteria_worker(name: &str, orders: Receiver<&str>, lunches: Sender<Lunch>) {
    for order in orders {
        println!("{} receives order fo {}", name, order);
        let lunch = match &order {
            x if x.contains("soup") => Lunch::Soup,
            x if x.contains("salad") => Lunch::Salad,
            x if x.contains("sandwich") => Lunch::Sandwich,
            _ => Lunch::HotDog,
        };

        for _ in 0..order.len() {
            thread::sleep(Duration::from_secs_f32(0.1))
        }

        println!("{} sends a {:?}", name, lunch);
        if lunches.send(lunch).is_err {
            break;
        }
    }
}
```
First, instead of matching on order itself, I matched an immutable reference to order.
Why would I do that?
Well because this particular match expression wants to move the value of order into the expression.
But I want to use order again after the match expression, so I create an immutable reference to order
and match on that instead.
```rust
fn main() {
    let (orders_tx, orders_rx) = channel::unbounded();
    let orders_rx2 = orders_rx.clone();

    let (lunches_tx, lunches_rx) = channel::unbounded();
    let lunches_tx2 = lunches_tx.clone();

    let alice_handle = thread::spawn(|| cafeteria_worker("alice", orders_rx2, lunches_tx2));
    let zack_handle = thread::spawn(|| cafeteria_worker("zack", orders_rx, lunches_tx));

    for order in vec!["polish dog", "ceasar salad", "onion soup", "reuben sandwich"] {
        println!("ORDER: {}", order);
        let _ = orders_tx.send(order);
    }

    drop(orders); // drops the value

    for lunch in lunches_rx {
        println!("Order UP! -> {:?}", lunch);
    }

    let _ = alice_handle.join();
    let _ = zack_handle.join();
}
```