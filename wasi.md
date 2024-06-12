# WASM WASI

## Install

```bash
rustup target add wasm32-wasip1
cargo install wasmtime-cli
```

## Test Set Up

```bash
cargo new hello_wasm
cd hello_wasm
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
