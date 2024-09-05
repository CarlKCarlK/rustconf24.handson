# RangeSetBlaze to `no_std`

Related Links:

* [Prerequisite Setup](setup.md)
* [Final Result](https://github.com/CarlKCarlK/range-set-blaze/tree/rustconf24.nostd)

## Start with WASM-for-Browser Code Branch

Clone a branch. Create and switch to a new branch. Run WASM for browser tests.

> Note: If you cut-and-paste, remember to add an *`ENTER`* at the end of each paste.

```bash
# top of projects directory
# create project via git
git clone --branch rustconf24.wasm1 --single-branch https://github.com/CarlKCarlK/range-set-blaze.git rustconf24.nostd
cd rustconf24.nostd
git checkout -b rustconf24.nostd
cargo check --target wasm32-unknown-unknown

# Optional: test wasm for browser if Chromedriver is installed
# cargo test --target wasm32-unknown-unknown
 # On Windows, ignore `os error 10004` error.
```

## Find `no_std`-Compatible Dependencies

To see if our project is `no_std`-compatible, we compile it to a `no_std` target. The `thumbv7m-none-eabi` target is a popular choice. It is an embedded processor (so, no operating system) that we can later emulate.

```bash
rustup target add thumbv7m-none-eabi
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

Cargo features with names like `std` and `use_std` suggest a need for the standard library. We research each such dependency by, for example, finding it on GitHub and reading its README and `Cargo.toml` files.

We then edit our `Cargo.toml`, creating our own `alloc` feature that avoids the `std` features of our dependencies. In addition, we add the `use_alloc` feature to `itertools`. Also, we increase the version of `gen_ops` to `0.4.0`.

```toml
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

Run "cargo tree" on our new feature:

```bash
cargo tree --features alloc --no-default-features --edges no-dev --format "{p} {f}" 
```

Run the "check" with our new feature:

```bash
cargo check --features alloc --no-default-features --target thumbv7m-none-eabi
```

It gets through our dependencies. But we now see hundreds of errors in our code.

## Making the Main (Non-Test) Code `no_std` (and `alloc`)

To start to fix the errors,

The top of `src/lib.rs` previously pulled in a `README.md` file. We'll keep that and add a `no_std`, `alloc`, and (conditional)`std` declaration:

```rust
#![doc = include_str!("../README.md")]
#![warn(missing_docs)]
#![no_std]
extern crate alloc;
#[cfg(feature = "std")]
extern crate std;
```

This says we won't necessarily use the standard library, but we will still use allocate memory. Also, when compiling with the `std` cargo feature (our default), we will use the standard library.

```bash
cargo check --features alloc --no-default-features --target thumbv7m-none-eabi
```

This reduces the errors to less then 40. We get one error for every place we use ``std::``, for example in `lib.rs`:

```rust
use std::cmp::max;
use std::cmp::Ordering;
use std::collections::BTreeMap;
```

These lines need to be changed to use either `core::` or (if memory related) `alloc::`.
For now, let's cheat and make this change by pulling in branch.

```bash
git fetch origin rustconf24.nostd1:refs/remotes/origin/rustconf24.nostd1
git reset --hard origin/rustconf24.nostd1
```

Try "check" again and it works!

```bash
cargo check --features alloc --no-default-features --target thumbv7m-none-eabi
```

This branch also updated the top of `src/tests.rs` with these lines:

> Note: You don't need to make this change because the branch did it for you.

```rust
#![cfg(test)]
extern crate std;
use std::prelude::v1::*;
use std::{format, print, println, vec};
```

Which allows native testing to work. (Cancel the test with `Ctrl-C` if you want to save time.)

```bash
cargo test
```

## Create an Example Embedded Project

Be sure `QEMU` is installed and on your path. See [setup](setup.md#qemu-emulator-for-embedded).
You should be able to run QEMU. Test with

```bashqemu-system-arm --version
```

* Create a `tests/embedded/Cargo.toml` that depends on your local project with "no default features" and "alloc":

```toml
[package]
edition = "2021"
name = "embedded"
version = "0.1.0"

[dependencies]
alloc-cortex-m = "0.4.4"
cortex-m = "0.7.7"
cortex-m-rt = "0.7.3"
cortex-m-semihosting = "0.5.0"
panic-halt = "0.2.0"
# reference to local project
range-set-blaze = { path = "../..", features = ["alloc"], default-features = false }
```

* Create a file `tests/embedded/src/main.rs`:

```rust
// based on https://github.com/rust-embedded/cortex-m-quickstart/blob/master/examples/allocator.rs
// and https://github.com/rust-lang/rust/issues/51540
#![feature(alloc_error_handler)]
#![no_main]
#![no_std]

extern crate alloc;
use alloc::string::ToString;
use alloc_cortex_m::CortexMHeap;
use core::{alloc::Layout, iter::FromIterator};
use cortex_m::asm;
use cortex_m_rt::entry;
use cortex_m_semihosting::{debug, hprintln};
use panic_halt as _;
use range_set_blaze::RangeSetBlaze;

#[global_allocator]
static ALLOCATOR: CortexMHeap = CortexMHeap::empty();
const HEAP_SIZE: usize = 1024; // in bytes

#[entry]
fn main() -> ! {
    unsafe { ALLOCATOR.init(cortex_m_rt::heap_start() as usize, HEAP_SIZE) }

    // test goes here
    let range_set_blaze = RangeSetBlaze::from_iter([100, 103, 101, 102, -3, -4]);
    hprintln!("{:?}", range_set_blaze.to_string());

    // exit QEMU/ NOTE do not run this on hardware; it can corrupt OpenOCD state
    if range_set_blaze.to_string() != "-4..=-3, 100..=103" {
        debug::exit(debug::EXIT_FAILURE);
    }

    debug::exit(debug::EXIT_SUCCESS);
    loop {}
}

#[alloc_error_handler]
fn alloc_error(_layout: Layout) -> ! {
    asm::bkpt();
    loop {}
}
```

* Copy `build.rs` and `memory.x` from [cortex-m-quickstart’s GitHub](https://github.com/rust-embedded/cortex-m-quickstart/tree/master) to `tests/embedded/`. For example,

Linux:

```bash
cd tests/embedded
wget https://raw.githubusercontent.com/rust-embedded/cortex-m-quickstart/master/build.rs
wget https://raw.githubusercontent.com/rust-embedded/cortex-m-quickstart/master/memory.x
```

Windows:

```cmd
cd tests/embedded
powershell -Command "Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/rust-embedded/cortex-m-quickstart/master/build.rs' -OutFile 'build.rs'"
powershell -Command "Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/rust-embedded/cortex-m-quickstart/master/memory.x' -OutFile 'memory.x'"
```

* Create a `tests/embedded/.cargo/config.toml` containing:

```toml
[target.thumbv7m-none-eabi]
runner = "qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel"

[build]
target = "thumbv7m-none-eabi"
```

* Update the project’s main `Cargo.toml` by adding `tests/embedded` to the workspace:

```toml
[workspace]
members = [".", "tests_common", "tests/wasm-demo", "tests/embedded"]
```

* Run the test with

```bash
cd tests/embedded
# to support #![feature(alloc_error_handler)]
rustup override set nightly 
rustup target add thumbv7m-none-eabi
cargo run
```

It should output the log message: `"-4..=-3, 100..=103"`.

## LATER: CI and Keywords

Add this to main `.github/workflows/ci.yml`:

```yml
test_thumbv7m-none-eabi:
    name: Setup and Check Embedded
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Set up Rust
        uses: dtolnay/rust-toolchain@master
        with:
          toolchain: stable
          target: thumbv7m-none-eabi
      - name: Install check stable and nightly
        run: |
          cargo check --target thumbv7m-none-eabi --features alloc --no-default-features
          rustup override set nightly
          rustup target add thumbv7m-none-eabi
          cargo check --target thumbv7m-none-eabi --features alloc --no-default-features
          sudo apt-get update && sudo apt-get install qemu qemu-system-arm
      - name: Test Embedded (in nightly)
        timeout-minutes: 1
        run: |
          cd tests/embedded
          cargo run
```

In the main `Cargo.toml`, add keywords and categories for WASM and "no standard".

```toml
[package]
#...
keywords = ["set", "range", "data-structures", "no_std", "wasm"]
categories = ["data-structures", "no-std", "wasm"]
```

The spelling (`no_std` and `no-std`) is important. The maximum number of keywords is five; likewise, categories.
