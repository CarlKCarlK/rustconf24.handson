# *Skip:* WASM Browser #1

Related Links:

* [Prerequisite Setup](setup.md)

## Install

Install the WASM Unknown OS target, bindings to JavaScript, and command-line WASM tool.

```bash
rustup target add wasm32-unknown-unknown
cargo install wasm-pack
cargo install wasm-bindgen-cli
```

> Aside: For time and simplicity, we are skipping Chromedriver testing.

## Test Set Up

```bash
# cd to top of projects directory
cargo new hello_wasm_browser
cd hello_wasm_browser
```

Add dependencies to `Cargo.toml`:

```bash
cargo add wasm-bindgen@0.2.93
cargo add wasm-bindgen-test@0.3.43 --dev
```

Edit `src/main.rs`

```rust
fn main() {
    println!("Hello, world!");
}

#[cfg(test)]
mod tests {
    use wasm_bindgen_test::wasm_bindgen_test;
    wasm_bindgen_test::wasm_bindgen_test_configure!(run_in_browser);

    #[test]
    #[wasm_bindgen_test]
    fn test_wasm() {
        assert_eq!("1".parse(), Ok(1));
    }
}
```

Create `.cargo/config.toml`:

```toml
[target.wasm32-wasip1]
runner = "wasmtime run --dir ."

[target.wasm32-unknown-unknown]
runner = "wasm-bindgen-test-runner"
```

Run native. Test native and WASM "browser"

```bash
cargo run
cargo test
cargo test --target wasm32-unknown-unknown
# optional: cargo test --target wasm32-wasip1
```

Should see

```text
Hello, world!

test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out
```

If you like, change the test to `"2".parse()`, so it fails. Re-run `cargo test --target wasm32-unknown-unknown` and see:

```text
test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out
console.log div contained:
    panicked at src/main.rs:12:9:
    assertion `left == right` failed
      left: Ok(2)
     right: Ok(1)
 ```
