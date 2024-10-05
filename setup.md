# Setup

We'll check if you are already setup. If not, we'll guide you through the installations.

> If you have questions, before the conference, feel free to
> send me an email at <carlk@msn.com> or find me on the conference
> Discord.

## *Are you already setup?*

> Note: If you cut-and-paste, remember to add an *`ENTER`* at the end of each paste.

```bash
# Hour 1 - WASM WASI
# Rust install and up-to-date w/ stable (default) and nightly
rustc --version
rustup update
rustup toolchain install nightly
rustup default stable
# Git installed
git --version
# Some editor installed, for example VS Code
code --version
# a target and tool
rustup target add wasm32-wasip1
cargo install wasmtime-cli


# Hour 2 - WASM in the Browser
# A target and tool
rustup target add wasm32-unknown-unknown
cargo install wasm-pack
cargo install wasm-bindgen-cli
# Web server installed (can instead use VS Code's Live Preview)
simple-http-server --version
## Skipping
## Chrome and Chromedriver installed and on path
# chromedriver --version
# which google-chrome # Linux
# where chrome # Windows


# Hour 3 - no_std
# QEMU installed
qemu-system-arm --version
# a target
rustup target add thumbv7m-none-eabi
```

### Table of Contents for Installations

Hour 1: WASM WASI

* [Rust](#rust)
* [Git](#git)
* [VS Code](#vs-code-for-example) (or another editor)
* [Targets and Tools](#install-all-targets-and-tools)

Hour 2: WASM in the Browser

* [Targets and Tools](#install-all-targets-and-tools)
* [Web Server for Local Testing](#web-server-for-local-testing) (or use VS Code's Live Preview)
* *SKIPPING*: [Chrome for Testing and Chromedriver](#chrome-for-testing-and-chromedriver)

Hour 3: no_std

* [`QEMU` Emulator for Embedded](#qemu-emulator-for-embedded)
* [Targets and Tools](#install-all-targets-and-tools)

## *Installations*

## Rust

### Install on Linux/MacOs/WSL

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

### Install on Windows

Download and run the installer: <https://static.rust-lang.org/rustup/dist/x86_64-pc-windows-msvc/rustup-init.exe>

### Check Version and Update (get 'nightly', too, for later)

```bash
rustc --version
rustup update
rustup default stable
rustup toolchain install nightly
```

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

### Some Possible Extensions

Here are the extensions I use:

* Live Preview (ms-vscode.live-server)
* Rust-Analyzer (matklad.rust-analyzer)
* CodeLLDB (vadimcn.vscode-lldb)
* GitLens — Git supercharged
* Rust Extension Pack (Swellaby)
* Dependi (Seray Uzgur)

## Test Set Up

```bash
# cd to top of projects directory
cargo new hello_rust
cd hello_rust
code . # just to look at project
cargo run
```

Should see

```text
Hello, world!
```

## Install All Targets and Tools

```bash
rustup target add wasm32-wasip1 wasm32-unknown-unknown thumbv7m-none-eabi
cargo install wasmtime-cli wasm-pack
cargo install wasm-bindgen-cli
```

## *SKIP*: Chrome for Testing and Chromedriver

As detailed below, we download binaries listed on <https://googlechromelabs.github.io/chrome-for-testing/> and add them to our path.

### Linux and WSL

```bash
cd ~
mkdir -p ~/.chrome-for-testing
cd .chrome-for-testing/
wget https://storage.googleapis.com/chrome-for-testing-public/126.0.6478.61/linux64/chrome-linux64.zip
wget https://storage.googleapis.com/chrome-for-testing-public/126.0.6478.61/linux64/chromedriver-linux64.zip
unzip chrome-linux64.zip
unzip chromedriver-linux64.zip

# Permanently update path
# echo 'export PATH=$PATH:~/.chrome-for-testing/chrome-linux64' >> ~/.bashrc
# echo 'export PATH=$PATH:~/.chrome-for-testing/chromedriver-linux64' >> ~/.bashrc

# Temporarily add the directories to the PATH for the current session
export PATH=$PATH:~/.chrome-for-testing/chrome-linux64:~/.chrome-for-testing/chromedriver-linux64

```

To test, type `chome` and see Chrome for Testing start.

### Windows

```cmd
powershell
# Make folder
New-Item -Path $HOME -Name ".chrome-for-testing" -ItemType "Directory"
Set-Location -Path $HOME\.chrome-for-testing

# Download zip files
bitsadmin /transfer "ChromeDownload" https://storage.googleapis.com/chrome-for-testing-public/126.0.6478.61/win64/chrome-win64.zip $HOME\.chrome-for-testing\chrome-win64.zip

bitsadmin /transfer "ChromeDriverDownload" https://storage.googleapis.com/chrome-for-testing-public/126.0.6478.61/win64/chromedriver-win64.zip $HOME\.chrome-for-testing\chromedriver-win64.zip

# Expand zip files
Expand-Archive -Path "$HOME\.chrome-for-testing\chrome-win64.zip" -DestinationPath "$HOME\.chrome-for-testing"

Expand-Archive -Path "$HOME\.chrome-for-testing\chromedriver-win64.zip" -DestinationPath "$HOME\.chrome-for-testing"

# # Permanently update path
# [System.Environment]::SetEnvironmentVariable("Path", $env:PATH + ";$HOME\.chrome-for-testing\chrome-win64;$HOME\.chrome-for-testing\chromedriver-win64", [System.EnvironmentVariableTarget]::User)

# Temporary path update
$env:PATH += ";$HOME\.chrome-for-testing\chrome-win64;$HOME\.chrome-for-testing\chromedriver-win64"

```

### Mac

Look at <https://googlechromelabs.github.io/chrome-for-testing/> and adapt the Linux method.

### Testing

Test with

```bash
chromedriver --version
```

## Web Server for Local Testing

I like VS Codes's "Live Preview" extension. As a simple alternative:

```bash
cargo install simple-http-server
# Later, to run, `simple-http-server` in the directory you want to serve
```

Then, in `hello_world`, create file `index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hello, World!</title>
</head>
<body>
    <h1>Hello, World!</h1>
</body>
</html>
```

```bash
simple-http-server --ip 127.0.0.2 --port 3000 --index
```

Then open a browser to <http://127.0.0.2:3000>.

## `QEMU` (Quick Emulator, "cue-em-you") for Embedded

See the [download page](https://www.qemu.org/download/) for full info.

Here are two handy links:

* Linux
  * Ubuntu: `sudo apt-get install qemu-system`
  * Arch: `sudo pacman -S qemu-system-arm`
  * Fedora: `sudo dnf install qemu-system-arm`
* Windows:
  * Method 1:
    * <https://qemu.weilnetz.de/w64/qemu-w64-setup-20240815.exe>
    * Tell windows it is okay to run the installer.
    * Then run the installer.
    * Add `"C:\Program Files\qemu\"` to your path.
  * Method 2:
    * Install msys2 from <https://github.com/msys2/msys2-installer/releases/download/2024-07-27/msys2-x86_64-20240727.exe>
    * Open msys2 ucrt64  terminal
    * `pacman -S mingw-w64-x86_64-qemu`
    * Put `C:\msys64\mingw64\bin\` in your path
* Mac: `brew install qemu` or `sudo port install qemu`

Test with

```bash
qemu-system-arm --version
```
