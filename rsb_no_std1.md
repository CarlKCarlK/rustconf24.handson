# RangeSetBlaze to `no_std`

Related Links:

* [Prerequisite Setup](setup.md)
* [Final Result](https://github.com/CarlKCarlK/range-set-blaze/tree/rustconf24.nostd)


## Start with WASM for Browser version

Clone a branch. Create and switch to a new branch. Run WASM for browser tests.

```bash
cd ~
git clone --branch rustconf24.wasm1 --single-branch https://github.com/CarlKCarlK/range-set-blaze.git rustconf24.nostd
cd rustconf24.nostd
git switch -c rustconf24.nostd

 wasm-pack build tests/wasm-demo --target web
 cargo test --target wasm32-unknown-unknown
```

The WASM for browser tests should succeed.

Also, can run `tests/wasm-demo/index.html` in a browser.

## Find `no_std`-Compatible Dependencies

To see if our project is `no_std`-compatible, we compile it on a `no_std` target. The `thumbv7m-none-eabi` target is a popular choice. It is an embedded processor (so, no operating system) that we can later emulate.

```bash
rustup target add thumbv7m-none-eabi # one time
cargo check --target thumbv7m-none-eabi
```

We see errors related to our dependencies.

To fix these, first, see what dependencies you are using and which of
their cargo features you are using.

```bash
cargo tree --edges no-dev --format "{p} {f}"
```

Which outputs:

```text
range-set-blaze v0.1.6 (C:\deldir\branches\rustconf24.nostd) 
├── gen_ops v0.3.0
├── itertools v0.13.0 default,use_alloc,use_std
│   └── either v1.12.0 use_std
├── num-integer v0.1.46 default,std
│   └── num-traits v0.2.19 default,i128,std
│       [build-dependencies]
│       └── autocfg v1.3.0
└── num-traits v0.2.19 default,i128,std (*)
```

Cargo features with names like `std` and `use_std` cargo features suggest a need for the standard library. We research each such dependency by, for example, finding it on GitHub and reading its README and `Cargo.toml` files.

We then edit our `Cargo.toml` removing the suspect Cargo features and adding the `use_alloc` feature to `itertools`:

```toml
[dependencies]
itertools = { version = "0.13.0", features=["use_alloc"], default-features = false }
num-integer = { version = "0.1.46", default-features = false }
num-traits = { version = "0.2.19", default-features = false }
gen_ops = "0.3.0"
```

Now, `cargo tree --edges no-dev --format "{p} {f}"` shows no suspicious cargo features. Moreover, `cargo check --target thumbv7m-none-eabi` gets through our dependencies. Now, all the  and shows errors in our code.

## Making the Non-Test Code `no_std` (and `alloc`)

At top of the `lib.rs`, add

```rust
#![no_std]

extern crate alloc;
```

This says we won't use the standard library, but we will still use allocated memory.

Now `cargo check --target thumbv7m-none-eabi` causes dozens of errors, one for every place we use ``std::``, for example:

```rust
use std::cmp::max;
use std::cmp::Ordering;
use std::collections::BTreeMap;
```

Trying changing each of these to use either `core::` or (if memory related) `alloc::`. For example:

```rust
use core::cmp::max;
use core::cmp::Ordering;
use alloc::collections::BTreeMap;
```

If you see errors on `String`, `Vec`, `Box`, `format!` etc. add

```rust
use alloc::string::String;
use alloc::vec::Vec;
use alloc::boxed::Box;
use core::fmt;
```

We still have error, related to one dependency named `gen_ops`. After some research,
the answer is to change its version from `0.3.0` to `0.4.0`.

Our main code now works, but we have 89 new errors in `src\test.rs`. We'll address that next.

## Test Code Must Use the Standard Library

In Rust, test code must use the standard library. To allow this, first update `Cargo.toml` by defining and using cargo features `std` and `alloc`:

```toml
[lib]

[features]
default = ["std"]
std = ["itertools/use_std", "num-traits/std", "num-integer/std"]
alloc = ["itertools/use_alloc", "num-traits", "num-integer"]

[dependencies]
itertools = { version = "0.13.0", optional = true, default-features = false }
num-integer = { version = "0.1.46", optional = true, default-features = false }
num-traits = { version = "0.2.19", optional = true, default-features = false }
gen_ops = "0.4.0"
```

Change the top of `src/lib.rs` to:

```rust
#![no_std]

extern crate alloc;

#[cfg(feature = "std")]
extern crate std;
```

Make the top of `src/tests.rs`:

```rust
#![cfg(test)]
use std::format;
use std::prelude::v1::*;
use std::vec;
use std::{print, println};
```

Now check that `no_std` compiles and that the `std` still run tests:

```bash
cargo check --target thumbv7m-none-eabi --features alloc --no-default-features
cargo test
```

Edit `useful.md` to include both lines.
