# RangeSetBlaze to WASI and WASM Browser

Related Links:

* [Prerequisite Setup](setup.md)
* [Final Result](https://github.com/CarlKCarlK/range-set-blaze/tree/rustconf24.wasm1)

## Start native RangeSetBlaze

Clone a branch. Create and switch to a new branch. Run tests.

```bash
# top of projects directory
git clone --branch rustconf24.0 --single-branch https://github.com/CarlKCarlK/range-set-blaze.git rustconf24.wasm1
cd rustconf24.wasm1
git checkout -b rustconf24.wasm1

cargo test
```

## To WASI

Let's just cheat and merge the WASI branch into our branch:

```bash
git fetch origin rustconf24.wasi:refs/remotes/origin/rustconf24.wasi
git merge origin/rustconf24.wasi

cargo test --target wasm32-wasip1
```  

Behind the scenes it is doing:

* In `.github/ci.yml`, adding rust WASI test (and 32-bit Ubuntu)
* In `Cargo.toml`'s `dev-dependencies`, make `criterion` turn off `rayon` feature when `wasm32`.
* In `lib.rs` and `tests/integration_tests.rs`, use `u64` rather than `usize`.
* In `useful.md` add `cargo test --target wasm32-wasip1`.

## To WASM Browser

Let's cheat again:

```bash
git fetch origin rustconf24.wasm1:refs/remotes/origin/rustconf24.wasm1
git merge origin/rustconf24.wasm1

cargo test --target wasm32-unknown-unknown
 # On Windows, ignore `os error 10004` error
```

Behind the scenes it is doing:

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
