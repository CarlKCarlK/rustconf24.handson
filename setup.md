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
* Live Preview (ms-vscode.live-server)

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

## Chrome for Testing and Chromedriver

We download binaries list on <https://googlechromelabs.github.io/chrome-for-testing/> and add them to our path.

### Linux and WSL

You may need `sudo apt-get install unzip`.

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

### Windows

```powershell
# Make folder
New-Item -Path $HOME -Name ".chrome-for-testing" -ItemType "Directory"
Set-Location -Path $HOME\.chrome-for-testing

# Download zip files
bitsadmin /transfer "ChromeDriverDownload" https://storage.googleapis.com/chrome-for-testing-public/126.0.6478.61/win64/chrome-win64.zip $HOME\.chrome-for-testing\chrome-win64.zip

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
chrome --version
chromedriver --version
```


## Web Server for Local Testing

I like VS Codes's "Live Preview" extension. As a simple alternative:

```bash
cargo install simple-http-server
```

Then, in a folder that contains an `index.html`, run

```bash
simple-http-server --ip 127.0.0.2 --port 3000 --index
```




