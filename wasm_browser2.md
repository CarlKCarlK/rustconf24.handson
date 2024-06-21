# WASM Browser #2

## Native Project: Install, Test, and Run

```bash
git clone --branch native_version --single-branch https://github.com/CarlKCarlK/rustconf24-good-turing.git good-turing
cd good-turing
cargo test
cargo run pg100.txt
cargo run --target wasm32-wasip1 pg100.txt # may need .cargo/config.toml
```

Outputs

```text
Prediction (words that appear exactly once on even lines): 10223
Actual distinct words that appear only on odd lines: 7967
```

Web Page:

Also, web serve the file `index.html`. Choose file `pg100.txt`. See output: `Lines in file: 196391`

## `src/main.rs` → `src/lib.rs`

Rename `src/main.rs` to `src/lib.rs`.

## `Cargo.toml`

Merge this:

```toml
[lib]
crate-type = ["cdylib", "rlib"]

[dependencies]
wasm-bindgen = "0.2"

[dev-dependencies]
wasm-bindgen-test = "0.3.42"
```

`cargo test` should still work.

## `lib.rs`

At the top, add:

```rust
use wasm_bindgen::prelude::*;
```

Add `pub` before `fn good_turning...`.

Add `#[wasm_bindgen]` before `pub fn good_turning...`.

In `mod tests`, add:

```rust
    use wasm_bindgen_test::wasm_bindgen_test;
    wasm_bindgen_test::wasm_bindgen_test_configure!(run_in_browser);
```

Add `#[wasm_bindgen_test]` before `fn test_process_file()`. (Leave `#[test]`, too.)

Run

```bash
cargo check --target wasm32-unknown-unknown
```

It will fail because WASM for the browser doesn't like tuples or `io::Error`
in `Result<(usize, usize), io::Error>`.

Remove `#[wasm_bindgen] pub ` from `fn good_turning. Create a new wrapper function:

```rust
#[wasm_bindgen]
pub fn good_turing_js(file_name: &str) -> Result<Vec<u64>, String> {
    match good_turning(file_name) {
        Ok((prediction, actual)) => Ok(vec![prediction as u64, actual as u64]),
        Err(e) => Err(format!("Error processing data: {}", e)),
    }
}
```

In the tests, remove `#[wasm_bindgen_test]` from `fn test_process_file`. Create a new test:

```rust
    #[test]
    #[wasm_bindgen_test]
    fn test_process_file_js() {
        let vec = good_turing_js("./pg100.txt").unwrap();
        assert_eq!(vec[0], 10223);
        assert_eq!(vec[1], 7967);
    }
```

These should now work and run both tests natively:

```bash
cargo check --target wasm32-unknown-unknown
cargo test
```

However, when we run the test in the browser:

```bash
wasm-pack test --chrome --headless
```

We get a run-time error: `Error processing data: operation not supported on this platform` because
WASM in the browser doesn't support reading from files.

We fix this by changing `fn good_turing` to work on generic `BufReader`'s.

Was:

```rust
fn good_turning(file_name: &str) -> Result<(usize, usize), io::Error> {
    let reader = BufReader::new(File::open(file_name)?);
```

Now:

```rust
use std::io::Read;

fn good_turning<R: BufReader>(reader: R) -> Result<(usize, usize), io::Error> {
```

The wrapper function `good_turing_js` now works on slices of `u8`:

```rust
#[wasm_bindgen]
pub fn good_turning_js(data: &[u8]) -> Result<Vec<u64>, String> {
    let reader = BufReader::new(data);
    match good_turning(reader) {
        Ok((prediction, actual)) => Ok(vec![prediction as u64, actual as u64]),
        Err(e) => Err(format!("Error processing data: {}", e)),
    }
}
```

We update the tests to use `BufReader` and slices of `u8`:

```rust
    use std::fs::File;

    #[test]
    fn test_process_file() {
        let reader = BufReader::new(File::open("pg100.txt").unwrap());
        let (prediction, actual) = good_turning(reader).unwrap();
        assert_eq!(prediction, 10223);
        assert_eq!(actual, 7967);
    }

    #[test]
    #[wasm_bindgen_test]
    fn test_good_turning_js() {
        let data = include_bytes!("../pg100.txt");
        let result = good_turning_js(data).unwrap();
        assert_eq!(result, vec![10223, 7967]);
    }
```

Comment out `fn main`. Remove any unneeded imports.

Now, we can run two native tests and one WASM in the browser test with:

```bash
cargo test
wasm-pack test --chrome --headless
```

## `index.html`

Compile to WASM

```bash
wasm-pack build --target web
```


Update `index.html` to

* Call `good_turing_rs`
* Read bytes instead of text
* Handle errors

The result is:

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Good-Turing Estimation</title>
</head>

<body>
    <h1>Good-Turing Estimation</h1>
    <input type="file" id="fileInput" />
    <p id="lineCount"></p>

    <script type="module">
        import init, { good_turning_js } from './pkg/good_turing.js';

        document.getElementById('fileInput').addEventListener('change', function (event) {
            const file = event.target.files[0];
            if (file) { // If a file was selected
                const reader = new FileReader();
                // What to do when the file is full read
                reader.onload = async function (e) {
                    await init(); // Ensure 'good_turning_js' is ready
                    // View the memory buffer as a Uint8Array
                    const u8array = new Uint8Array(e.target.result);
                    try {
                        const result = await good_turning_js(u8array);
                        document.getElementById('lineCount').innerHTML =
                            `Prediction (words that appear exactly once on even lines): ${result[0].toLocaleString()}<br>` +
                            `Actual distinct words that appear only on odd lines: ${result[1].toLocaleString()}`;
                    } catch (err) {
                        document.getElementById('lineCount').innerHTML = `Error: ${err}`;
                    }
                };
                // Now start to read the file as memory buffer
                reader.readAsArrayBuffer(file);
            } else {
                document.getElementById('lineCount').innerHTML = '';
            }
        });
    </script>
</body>

</html>
```

Serve the web page, open file `pg100.txt` and see

```text
Prediction (words that appear exactly once on even lines): 10,223
Actual distinct words that appear only on odd lines: 7,967
```