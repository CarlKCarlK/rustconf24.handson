# WASM WASI

Related Links:

* [Prerequisite Setup](setup.md)
* [Final Result](https://github.com/CarlKCarlK/range-set-blaze/tree/rustconf24.wasi)

## Install

```bash
rustup target add wasm32-wasip1
cargo install wasmtime-cli
```

## Test Set Up

```bash
# Start in projects folder
cargo new hello_wasi
cd hello_wasi
```

Edit `src/main.rs`

```rust
fn main() {
    #[cfg(not(target_arch = "wasm32"))]
    println!("Hello, world!");
    #[cfg(target_arch = "wasm32")]
    println!("Hello, WebAssembly!");
}
```

Create a `.cargo/config.toml`

```toml
[target.wasm32-wasip1]
runner = "wasmtime run --dir ."
```

Run native and WASM.

```bash
cargo run
cargo run --target wasm32-wasip1
```

Should see

```text
Hello, world!
Hello, WebAssembly!
```

## Port RangeSetBlaze

Clone a branch. Create and switch to a new branch. Run native tests.

```bash
# Start in projects folder
git clone --branch rustconf24.0 --single-branch https://github.com/CarlKCarlK/range-set-blaze.git rustconf24.wasi
cd rustconf24.wasi
git checkout -b rustconf24.wasi
cargo test
```

The native tests succeed.

Create a `.cargo/config.toml`

```toml
[target.wasm32-wasip1]
runner = "wasmtime run --dir ."
```

Run the tests on WASM WASI:

```bash
cargo test --target wasm32-wasip1
```

Fails with:

```text
 error: Rayon cannot be used when targeting wasi32. Try disabling default features.
  --> C:\Users\carlk\.cargo\registry\src\index.crates.io-6f17d22bba15001f\criterion-0.5.1\src\lib.rs:31:1
   |
31 | compile_error!("Rayon cannot be used when targeting wasi32. Try disabling default features.");
 ```

After looking at `Criterion`'s [`Cargo.toml`](https://github.com/bheisler/criterion.rs/blob/master/Cargo.toml), we change our `Cargo.toml`'s `[dev-dependencies]` to turn off Criterion's `rayon` feature:

```toml
# was criterion = { version = "0.5.1", features = ["html_reports"] }
criterion = { version = "0.5.1", features = ["html_reports", "plotters", "cargo_bench_support"], default-features = false }

#...
[target.'cfg(not(target_arch = "wasm32"))'.dev-dependencies]
criterion = { version = "0.5.1", features = ["rayon"] }
```

Test again (`cargo test --target wasm32-wasip1`) and get

```rust
#[test]
fn test_demo_i32_len() {
    assert_eq!(demo_i32_len(i32::MIN..=i32::MAX), u32::MAX as usize + 1);
                                                  ^^^^^^^^^^^^^^^^^^^^^ attempt to compute `usize::MAX + 1_usize`, which would overflow    
}
```

Change to

```rust
#[test]
// tests/integration_test.rs
fn test_demo_i32_len() {
    assert_eq!(demo_i32_len(i32::MIN..=i32::MAX), u32::MAX as u64 + 1);
}
// src/lib.rs -- change isize/usize to i64/u64
pub fn demo_i32_len(range: RangeInclusive<i32>) -> u64 {
    let (start, end) = range.into_inner();
    if start > end {
        return 0;
    }
    (end as i64 - start as i64) as u64 + 1
}
```

Test again and it works. However,

**If a feature isn’t tested, it doesn’t exist.**

Add this to `.github/workflows/ci.yml`

```yml
  test_wasip1:
      name: Test WASI P1
      runs-on: ubuntu-latest
      steps:
        - name: Checkout
          uses: actions/checkout@v4
        - name: Set up Rust
          uses: dtolnay/rust-toolchain@master
          with:
            toolchain: stable
            targets: wasm32-wasip1
        - name: Install Wasmtime
          run: |
            curl https://wasmtime.dev/install.sh -sSf | bash
            echo "${HOME}/.wasmtime/bin" >> $GITHUB_PATH
        - name: Run WASI tests
          run: cargo test --verbose --target wasm32-wasip1
```

*OPTIONAL*:
Check into Github and see the new wasi test pass.
