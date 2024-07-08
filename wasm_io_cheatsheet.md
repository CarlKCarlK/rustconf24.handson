# WASM Input/Output Cheatsheet

| Allowed | Type             | Where        | Notes                                                                 |
|---------|------------------|--------------|-----------------------------------------------------------------------|
| Yep!    | `u8`               | input, output | Also, `i8`, `u16`, `i16`, `u32`, `i32`                             |
| Nope    | `&u8`              | input        | Can't pass copy types by reference                                  |
| Nope    | `&u8`              | output       | Can't return reference types because of lifetime issues             |
| Avoid   | `usize`, `isize`   | input, output | Works but 32-bit, not 64-bit, so confusing                         |
| Yep!    | `u64`, `i64`       | input, output | On JS side, must use `BigInt(n)`                                   |
| Nope    | `u128`, `i128`     | input, output |                                                                    |
| Yep!    | `f32`, `f64`       | input, output |                                                                    |
| Yep!    | `bool`             | input, output |                                                                    |
| Avoid   | `char`             | input, output | Converted from/to JS string and not length checked                 |
| Yep!    | `String`           | input, output | Copies memory                                               |
| Yep!    | `&str`             | input        | Copies memory                                               |
| Nope    | `&str`, `&[u8]`    | output       | As before, can't return reference types                               |


| Allowed | Type             | Where        | Notes                                                                 |
|---------|------------------|--------------|-----------------------------------------------------------------------|
| Yep!    | `Vec<u8>`          | input, output | Copies memory                                               |
| Yep!    | `&[u8]`            | input        | Copies memory                                               |
| Yep!    | `&mut [u8]`        | input        | Copies memory twice [(link)](https://stackoverflow.com/a/78634853/5976009)                                        |
| Nope    | `&mut u8`          | input        | Can't mutate a copy value                                             |
| Nope    | `&mut String`      | input        | Can't mutate a String                                                 |
| Nope    | `[u8; 2]`          | input, output | Can't use fixed length arrays                                         |
| Avoid   | `HashSet`, `HashMap` | input, output | No easy way to use                                                    |
| Yep!    | `Option<u8>`     | input, output | On JS side, use null or the value                                     |
| Yep!    | `Option<String>` | input, output | All owned types work. Will copy.                                      |
| Nope    | `Option<&[u8]>`  | input        | Reference types don't work                                            |
| Yep!    | `Result<u8, String>`| output    | On JS side, use try/catch. Works with any owned/copy type and an error message string.  |
| Yep!    | `JsValue`          | input, output | On Rust side, can construct and manipulate                            |
| Yep!    | any Rust struct  | input, output | On JS side, can construct and manipulate                              |

