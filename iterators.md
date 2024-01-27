# Iterators

```rust
let v = vec![6, 7, 8, 9];

for num in v {
    println!("{}", num);
}

```

The Vector v here isn't actually used directly by the for loop. The for loop operates on any iterator, and if it's not an iterator, it calls the **into_iter()** method, which returns an actual iterator. **into_iter()** is a method of the IntoIterator trait. Anything that implements IntoIterator will be converted into an iterator by a for loop automatically.

The **into_iter()** method returns an iterator which takes ownership of the collection it is implemented on, consuming the collection.
This is why, in the Ultimate Rust Crash Course exercises, I was careful to only use for loops on vectors
that were no longer needed, because after a vector is used in a for loop, the vector is gone.

Let's go back and rewrite that for loop in functional programming style using **into_iter()** directly.
```rust
let v = vec![6, 7, 8];
v.into_iter().for_each(|num| println!("{}", num)); // works faster then a for loop and we can use Iterator Adapters
```

This time we explicitly call into_iter() to get the iterator, and then we call the iterator's for_each method. This method takes a closure that accepts a value from the vector as an argument. Our closure does the same println call that our for loop did.

### Iterator Adapters
An iterator adapter is a tool in the functional programming paradigm which takes an iterator and outputs a different iterator
that will take some action on values as they pass through it. Many of the iterator trait's methods are iterator adapters.

Let's modify our code to use a couple of iterator adapters:
```rust
let v = vec![6, 7, 8];
let total: i32 = v
    .into_iter() // 6, 7, 8
    .map(|x: i32| x * 3) // 18, 21, 24
    .filter(|y: &i32| *y % 2 == 0)  // let's use the filter iterator adapter to remove some values that don't meet a condition.  // 18, 24
    // this code without consumer will discard values
```

First, let's use map. Map is used to transform each item in some way. You give map a closure that takes ownership of the value, and whatever the closure returns becomes the new value. Map can return a different type than it received, but in this case, I chose to stick with integers for simplicity. Filter takes a closure that receives an immutable reference to the value, because we want to leave the value alone if it meets our condition. The closure should return true for values to keep and false for values to drop.

Look at the adapter documentation to find out more about it.

So far, nothing we've done will actually touch any numbers. Iterator adapters produce new iterators. Iterators are lazy and they don't do anything unless they're consumed. That's why you always need to end one of these chains of iterator adapters with an iterator consumer.

## Iterator Consumers
An iterator consumer is something that actually consumes the final iterator in some way, causing the chain of iterator adapters to do their processing. We've already seen one iterator consumer. **for_each()** is an iterator consumer that consumes each value and passes it to a closure whose return value is discarded.

```rust
let v = vec![6, 7, 8];
v.into_iter()
    .map(|x: i32| x * 3)
    .filter(|y: &i32| *y % 2 == 0)
    .for_each(|z| println!("{}", z)); // adding an iterator consumer
```

Calling for_each() makes the iterators actually do their thing. So this will print 18 and 24.

### Sum
There are a bunch of useful iterator consumers that you can use. One example is the **sum** method. Sum adds up all of the values and returns the sum.
```rust
let v = vec![6, 7, 8];
let total i32 = v // type annotation here is required since iterator consumer is not able to detect an output type
    .into_iter() // 6, 7, 8
    .map(|x: i32| x * 3) // 18, 21, 24
    .filter(|y| *y % 2 == 0)  // 18, 24
    .sum(); // 42
```

Unfortunately, we don't always have a variable we're assigning to. In that case, what do we annotate?
Let's remove the variable assignment and the semicolon at the end and pretend we're returning the value
of sum from a block. We can use a **turbofish** parameter.

### Turbofish

The turbofish is, perhaps, one of Rust's oddest quirks.
It's called the turbofish because the right side sort of looks like a fish that is swimming rapidly
to the right in turbo mode, leaving bubbles behind it, hence turbofish.
It's used to specify the type of the generic parameter or parameters for a generic function or method.
The weirdest part is that it goes between the method or function name and the argument list.
Behold, the turbo fish in all its glory.
Someday, this will hopefully be replaced with a simpler syntax.

```rust
let v = vec![6, 7, 8];
v.into_iter() // 6, 7, 8
    .map(|x: i32| x * 3) // 18, 21, 24
    .filter(|y| *y % 2 == 0)  // 18, 24
    .sum::<i32>(); // 42
```

### Collect
Collect will gather all the items and put them into a new collection. Unfortunately, collect doesn't know which collection type you would like.
Once again, we can annotate our new variable to help it out.

```rust
let v = vec![6, 7, 8];
let v2: Vec<i32> = v
    .into_iter() // 6, 7, 8
    .map(|x: i32| x * 3) // 18, 21, 24
    .filter(|y| *y % 2 == 0)  // 18, 24
    .collect();
```

It already knows what the type of the item is. So we can simplify this by replacing i32 with an underscore.
This is super helpful if you're dealing with a big, complicated item type.

```rust
let v = vec![6, 7, 8];
let v2: Vec<_> = v
    .into_iter() // 6, 7, 8
    .map(|x: i32| x * 3) // 18, 21, 24
    .filter(|y| *y % 2 == 0)  // 18, 24
    .collect();
```

Or use a turbofish:

```rust
let v = vec![6, 7, 8];
let v2: = v
    .into_iter() // 6, 7, 8
    .map(|x: i32| x * 3) // 18, 21, 24
    .filter(|y| *y % 2 == 0)  // 18, 24
    .collect::<Vec<_>>();
```

But what if you don't want to destroy the collection you're iterating over? Just like you can take
immutable or mutable references to a variable,
there are iterators that take immutable or mutable references to the items they iterate over.
```rust
v.into_iter(); // consumes v, returns owned items
v.iter(); // returns immutable references
v.iter_mut(); // returns mutable references
```

Syntactic shugar forms of above:
```rust
for _ in v
for _ in &v
for _ in &mut v
```

### Drain
One final corner case that we need to go over is how to empty out a collection without consuming the
collection itself. That's what the **drain** method is for. The drain method takes different arguments depending on which collection you're looking at.
But in all cases, it returns an iterator that takes ownership of some or all items in the collection,
removing the items from the collection, but leaving the collection itself intact so you can continue to use it,
the vector's drain method takes a range to indicate which items in the vector to drain. A bare range operator
(two dots) indicates that you want to empty out the entire vector.
```rust
v.drain();
h.drain(..);
```

The hash map's drain method doesn't take any arguments and returns all of the key value pairs in the hash map.
The rest of the standard library collections do something similar.