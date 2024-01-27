# Publishing
Everything you publish to [crates.io](https://crates.io) will be there forever. There is now way to delete stuff. Be carefull not to publish any sensitive information. Do not publish useless packages. Package name should be unique (may be already changed). 

## Authentication
You need to log in to the crates.io using your github account. Then you can generate an API token in order to access [crates.io](https://crates.io) using:
```shell
cargo login
```
## Package Preparation
Only name an version ared required fields in the Cargo.toml file to publish a package, but it is strongly recommended to add additional information like:

```toml
[package]
name = "rusty_engine"
version = "2.0.0"
description = "Learn rust with simple, cross-platform, 2D game engine."
edition = "2021"
homepage = "https://github.com/cleancut/rusty_engine"
repository = "https://github.com/cleancut/rusty_engine"
readme = README.md
keywords = [ "game", "engine", "graphics", "audio", "rusty" ]
categories = ["game-engines"]
license = MIT or Apache-2.0

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]

```
Categories slug list is available at crates.io. Once you finish your **Cargo.toml**, you can publish it using:
```shell
cargo publish
```
Documentation of the published package appears at [docs.rs](https://docs.rs) automatically.