# Setup

## Rust

### Check Version and Update

```bash
rustc --version
rustup update
```

### Install Linux/MacOs/WSL

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Install Windows

Download and run the installer: <https://static.rust-lang.org/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe>

## Git

### Check Version

```bash
git --version
```

### Install

Download and install Git from <https://git-scm.com/downloads>

For Ubuntu

```bash
sudo apt-get update
sudo apt-get install git
```

## VS Code (for example)

### Check Version (VS Code)

```bash
code --version
````

### Install VS Code

Download and install VS Code from <https://code.visualstudio.com/Download>

### Extensions

Search and install the following extensions:

* Rust-Analyzer (matklad.rust-analyzer)
* CodeLLDB (vadimcn.vscode-lldb)

## Test Set Up

```bash
cargo new hello_rust
cd hello_rust
code . # just to look at project
cargo run
```

Should see

```text
Hello, world!
```
