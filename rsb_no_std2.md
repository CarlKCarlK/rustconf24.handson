# RangeSetBlaze Testing

Related Links:

* [Prerequisite Setup](setup.md)
* [RangeSetBlaze `no_std` #1](rsb_no_std1.md)
* [Final Result](https://github.com/CarlKCarlK/range-set-blaze/tree/rustconf24.nostd)

## Set up

Be sure `QEMU` is installed and on your path. See [setup](setup.md#qemu-emulator-for-embedded).
You should be able to run QEMU. Test with

```bash
qemu-system-arm --version
```

Either continue work in folder `~/rustconf24.nostd` or clone the `rustconf24.nostd0` branch:

```bash
# create project via git
cd ~
git clone --branch rustconf24.nostd0 --single-branch https://github.com/CarlKCarlK/range-set-blaze.git rustconf24.nostd
cd rustconf24.nostd
git switch -c rustconf24.nostd
```

Test (std) and check (no_std) the project with:

```bash
# cd ~/rustconf24.nostd
cargo test
cargo check --target thumbv7m-none-eabi --features alloc --no-default-features
```

## Create an Example Embedded Project

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

* Copy `build.rs` and `memory.x` from [cortex-m-quickstart’s GitHub](https://github.com/rust-embedded/cortex-m-quickstart/tree/master) to `tests/embedded/`. For example, on Linux:

```bash
cd tests/embedded
wget https://raw.githubusercontent.com/rust-embedded/cortex-m-quickstart/master/build.rs
wget https://raw.githubusercontent.com/rust-embedded/cortex-m-quickstart/master/memory
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
rustup override set nightly # to support #![feature(alloc_error_handler)]
rustup target add thumbv7m-none-eabi
cargo run
```

(Also, put these lines in `tests/embedded/useful.md`.)

It should output the log message: `"-4..=-3, 100..=103"`.

## CI and Keywords

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

## Bonus checking CI (Ubuntu)

To change the origin repository on GitHub:

```bash
sudo apt-get install gh
gh auth login
git remote remove origin
gh repo create delete_me_rsb --public --source=. --remote=origin
```
