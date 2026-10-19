# Software Setup

This document tells you which tools to install and how to do a check of each tool. All tools work natively on Windows, macOS, and Linux. On Windows, you do not need WSL2 for this course (one exception: fuzzing in phase 8 is optional, and works best on Linux, macOS, or WSL2).

## 1. Install the tools

### Rust

| Operating system | Steps |
|---|---|
| Windows | Download and run `rustup-init.exe` from rustup.rs. Rust on Windows needs the Microsoft C++ build tools (the linker). If they are not installed, `rustup-init` offers to install them with the Visual Studio installer. Select the workload "Desktop development with C++". Keep the default toolchain (`stable-x86_64-pc-windows-msvc`, or `aarch64` on Arm computers). |
| macOS | Install the Xcode command-line tools: `xcode-select --install`. Then run the command from rustup.rs: `curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs \| sh`. |
| Linux | Install a C compiler and linker (for example `sudo apt install build-essential` on Ubuntu or Debian). Then run the command from rustup.rs. Do not use the Rust package of your distribution: it is often old. |

After the installation, open a new terminal, so that the `PATH` contains `~/.cargo/bin` (on Windows: `%USERPROFILE%\.cargo\bin`).

Add the components:

```sh
rustup component add rustfmt clippy rust-analyzer rust-src
```

### Other tools

| Tool | Source | Phase | Purpose |
|---|---|---|---|
| Git | git-scm.com | 0 | Version control |
| VS Code | code.visualstudio.com | 0 | Editor |
| RustRover (alternative to VS Code) | jetbrains.com/rust | 0 | IDE. Free for non-commercial use. Check the current license terms. |
| `bacon` (optional) | `cargo install --locked bacon` | 0 | Runs `cargo check`, `clippy`, or tests again on each file change |
| Rustlings | `cargo install rustlings` | 1 | Small exercises |
| `sqlx-cli` | `cargo install --locked sqlx-cli --no-default-features --features sqlite` | 7 | Database migrations and offline query data |
| An HTTP client: `curl` (included in Windows 10 and newer, macOS, and most Linux systems), or a graphical client (for example Bruno) | | 7 | Send requests to your API |
| SQLite command-line shell (`sqlite3`, optional) | sqlite.org | 7 | Look at the database |
| GitHub CLI `gh` (optional) | cli.github.com | 8 | Pull requests from the terminal |
| Docker Desktop or Docker Engine | docker.com | 8 | Container images |
| `cargo-nextest`, `cargo-llvm-cov`, `cargo-deny`, `cargo-audit`, `cargo-mutants`, `cargo-semver-checks`, `cargo-insta` | `cargo install --locked <name>` | 8 | Testing and analysis tools |

`cargo install` compiles a tool from source, which can take some minutes. `--locked` uses the dependency versions that the authors tested. The optional tool `cargo-binstall` downloads ready binaries for many tools, which is faster.

### VS Code extensions

- **rust-analyzer** (rust-lang). Do not install the old extension with the name "Rust".
- **CodeLLDB** (vadimcn), for the debugger. On Windows with the MSVC toolchain, the **C/C++** extension (Microsoft) can also debug Rust programs.
- **Even Better TOML** (tamasfe), for `Cargo.toml`. Optional.
- **Error Lens** (usernamehw), shows errors next to the code. Optional.

Recommended settings (`settings.json`):

```json
{
  "editor.formatOnSave": true,
  "rust-analyzer.check.command": "clippy"
}
```

## 2. Do a check of the tools

Run these commands in a new terminal. Each command must print a version.

```sh
rustup --version
rustc --version
cargo --version
cargo fmt --version
cargo clippy --version
git --version
```

The version of `rustc` must be 1.85 or newer for edition 2024. Run `rustup update` to get the newest stable version.

## 3. Do a check of the build

```sh
cargo new hello-check
cd hello-check
cargo run
cargo test
cargo clippy
```

`cargo run` must print `Hello, world!`. Delete the `hello-check` directory after the check.

**If the build fails on Windows** with `linker 'link.exe' not found`: the C++ build tools are missing. Run the Visual Studio installer, and add the workload "Desktop development with C++".

**If the build fails on macOS** with `xcrun: error`: run `xcode-select --install`.

**If the build fails on Linux** with `linker 'cc' not found`: install `build-essential` (Debian, Ubuntu) or the C compiler package of your distribution.

## 4. Do a check of the debugger

1. Open the `hello-check` project in VS Code.
2. Set a breakpoint on the `println!` line.
3. Click "Debug" above `fn main` (rust-analyzer shows it).
4. The program must stop on the breakpoint.

If "Debug" does not appear, check that the rust-analyzer and CodeLLDB extensions are installed and enabled.

## 5. Keep the tools up to date

At the start of each phase:

```sh
rustup update
```

Update the Cargo tools with `cargo install --locked <name>` again, or with `cargo install-update -a` (from the crate `cargo-update`).
