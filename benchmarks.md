# Benchmarks
## Criterion
Rust has a built-in a way to benchmark your code,
but it's not really finished.
It's only available in the nightly compiler, and folks don't use it a whole lot yet.
So I'm going to introduce you to Criterion instead.

**Cargo.toml:**
```toml
[dev-dependencies]
criterion = { version = "0.3", features = ["html_reports"]}

[[bench]]
name = "snuggle_speed"
harness = false
```
Development dependencies will be compiled for development subcommands like "cargo test" and "cargo bench",
so that you can use extra libraries for tests or benchmarks that don't get compiled into your release product.
This is *a* bench section. Why does it have two square brackets?
Because that's the toml syntax for sections you can have more than one of.

You can make multiple bench sections, one for each file you want to put benchmarks into.
"name" will be the name of the rust source file that will hold your set of benchmarks. "harness"
needs to be false to disable the built-in benchmarking functionality so that Criterion can do the benchmarking instead.

Next, very similar to how integration tests work, we need a benches directory.
This is really similar to how it worked with integration tests.
All benchmark tests are in the benches directory, which is at the root of the project.
I'll go ahead and create snuggle_speed.rs in the benches directory.
Remember that this filename needs to match the name we put for the benchmark and Cargo.toml.

```shell
mkdir benches
touch benches/snuggle_speed.rs
```
Remember that this filename needs to match the name we put for the benchmark and Cargo.toml.

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};
use hello::snuggle;

pub fn snuggle_benchmark(c: &mut Criterion) {
    c.bench_function("snuggle 2", |b| b.iter(|| snuggle(black_box(2))));
}

criterion_group!(benches, snuggle_benchmark);
criterion_main!(benches);
```
At the top we import everything we need to use.
We can name our benchmark functions whatever we like, but it needs to accept a mutable reference
to a Criterion struct.
We call the .bench_function() method on the Criterion struct and pass it a name which will show up in the
output and be used as a key for the stored statistics.
Then we pass in a closure that takes a bencher, and inside the closure we call the bencher's .iter() method
passing that another closure which takes no arguments but calls our snuggle function.
And we make sure to wrap any literals in a black_box to prevent the compiler from doing clever optimizations
to fold away the call to our function.
And then we do some criterion macro stuff to generate code to make everything work.

```rust
cargo bench
```

Besides needing to compile all the dependencies for Criterion, each benchmark has a warm-up period
of three seconds and then a measurement period of five seconds.

By the way, don't benchmark on public CI servers.
They're super noisy, so your statistics will be all over the place.
If you want to do benchmarks on CI, you should hook up a custom runner on dedicated hardware that does
nothing but run one benchmark job at a time. My recommendation: always start with idiomatic code, then identify something that
is slow, write your benchmark, and then try to optimize it and measure what happens.

```shell
cat target/criterion/report/index.html
```