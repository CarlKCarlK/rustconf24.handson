# WASM Browser

## Install

Install the WASM Unknown OS target, bindings to JavaScript, and command-line WASM tool.

```bash
rustup target add wasm32-unknown-unknown
cargo install wasm-bindgen-cli
cargo install wasm-pack
```

## Test Set Up

```bash
cargo new hello_wasm_browser
cd hello_wasm_browser
```

Edit `Cargo.toml`

```toml
[dependencies]
wasm-bindgen = "0.2"

[dev-dependencies]
wasm-bindgen-test = "0.3.42"
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

    #[wasm_bindgen_test]
    fn test_wasm() {
        assert_eq!("1".parse(), Ok(1));
    }
}
```

Run native and test WASM "browser"

```bash
cargo run
wasm-pack test --chrome --headless
```

Should see

```text
Hello, world!

Starting new webdriver session...
DevTools listening on ws://127.0.0.1:60482/devtools/browser/8d5dccb0-8658-4e26-afed-546eccd8d1a4
running 1 test

test hello_wasm_browser::tests::test_wasm ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 filtered out
```

If you like, change the test to `"2".parse()`, so it fails. Re-run `wasm-pack test --chrome --headless` and see:

```text
test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 filtered out
console.log div contained:
    panicked at src/main.rs:12:9:
    assertion `left == right` failed
      left: Ok(2)
     right: Ok(1)
 ```
 