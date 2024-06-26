# RangeSetBlaze to WASI and WASM Browser

## To WASI

* In `.github/ci.yml`, rust WASI test (and 32-bit Ubuntu)
* In `Cargo.toml`, make `dev-dependencies` and `criterion`, turn off `rayon` feature when `wasm32`.
* In `lib.rs` and `tests/integration_tests.rs`, use `u64` rather than `usize`.
* In `useful.md` add `cargo test --target wasm32-wasip1`.

## To WASM Browser

* In `.cargo/config.toml`, add

    ```toml
    [target.wasm32-unknown-unknown]
    runner = "wasm-bindgen-test-runner"
    ```

* In `.github/ci.yml`, add `test_wasm_unknown_unknown` tests.
* In `Cargo.toml`,
  * add workspace `"tests/wasm-demo"`,
  * add `dev-dependencies` `wasm-bindgen-test`
  * add WASM for Browser only `dev-dependencies`:

    ```toml
    [target.'cfg(all(target_arch = "wasm32", target_os = "unknown"))'.dev-dependencies]
    getrandom = { version = "0.2", features = ["js"] }
    web-time = "1.1.0"
    ```

* In every file with tests (`src/tests.rs`, `tests/api.rs`, `tests/integration_test.rs`):
  * at the top, add:

    ```rust
    use wasm_bindgen_test::wasm_bindgen_test;
    wasm_bindgen_test::wasm_bindgen_test_configure!(run_in_browser);
    ```

  * Before every test possible add `#[wasm_bindgen_test]` before `#[test]`.
* In `useful.md` add `cargo test --target wasm32-unknown-unknown`.
