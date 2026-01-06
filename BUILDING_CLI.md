# Building goose CLI from Source

This guide covers building only the goose CLI from source. If you want to build the full Desktop application, see [BUILDING_LINUX.md](BUILDING_LINUX.md) instead.

## What You'll Build

The goose CLI is the command-line interface for goose, which includes:
- The `goose` binary - CLI tool for running sessions, managing configuration, and executing recipes
- Core goose functionality (agent logic, provider integrations)
- MCP (Model Context Protocol) server support
- Benchmarking capabilities

## Prerequisites

### System Dependencies

**Debian/Ubuntu:**
```bash
sudo apt update
sudo apt install -y build-essential protobuf-compiler pkg-config libssl-dev libxcb1-dev
```

**Arch/Manjaro:**
```bash
sudo pacman -S --needed base-devel protobuf
```

**Fedora/RHEL/CentOS:**
```bash
sudo dnf install gcc gcc-c++ make protobuf-compiler openssl-devel
```

**openSUSE:**
```bash
sudo zypper install gcc gcc-c++ make protobuf openssl-devel
```

**macOS:**
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install protobuf (using Homebrew)
brew install protobuf
```

**Windows:**

For Windows, we recommend using one of these environments:
- **Git Bash** (recommended): Comes with [Git for Windows](https://git-scm.com/download/win)
- **MSYS2**: Available from [msys2.org](https://www.msys2.org/)
- **WSL2**: Windows Subsystem for Linux (follow Ubuntu instructions above)

### Development Tools

- **Rust**: Install via [rustup](https://rustup.rs/)
  ```bash
  curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
  ```
  This will install:
  - `rustc` - Rust compiler
  - `cargo` - Rust package manager and build tool
  - Standard library and documentation

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/block/goose.git
cd goose
```

### 2. Build the CLI

To build the goose CLI in release mode (optimized):

```bash
cargo build --release -p goose-cli
```

This will:
- Download and compile all necessary dependencies
- Build the goose core library (`crates/goose`)
- Build goose-mcp for MCP server support (`crates/goose-mcp`)
- Build goose-bench for benchmarking (`crates/goose-bench`)
- Build the goose-cli binary (`crates/goose-cli`)
- Place the final binary at `./target/release/goose`

The build process typically takes 5-15 minutes depending on your system.

### 3. Verify the Build

```bash
./target/release/goose --version
```

You should see output like:
```
goose 1.19.0
```

### 4. Run the CLI

```bash
# Show help
./target/release/goose --help

# Configure goose (first-time setup)
./target/release/goose configure

# Start a session
./target/release/goose session
```

## Installation Options

### Option 1: Use from Build Directory

You can run goose directly from the build directory:

```bash
./target/release/goose session
```

### Option 2: Install to System Path

**Linux/macOS:**
```bash
# Copy to a directory in your PATH
sudo cp target/release/goose /usr/local/bin/

# Or create a symlink
sudo ln -s $(pwd)/target/release/goose /usr/local/bin/goose

# Verify installation
goose --version
```

**Windows (Git Bash/MSYS2):**
```bash
# Copy to user bin directory
mkdir -p ~/.local/bin
cp target/release/goose.exe ~/.local/bin/

# Add to PATH (add to ~/.bashrc for persistence)
export PATH="$HOME/.local/bin:$PATH"
```

### Option 3: Install with Cargo

You can also install directly using cargo:

```bash
cargo install --path crates/goose-cli
```

This will compile and install the binary to `~/.cargo/bin/goose` (which should be in your PATH if you installed Rust via rustup).

## Development Build

For development and debugging, you can build without optimizations:

```bash
cargo build -p goose-cli
```

The binary will be at `./target/debug/goose` and will include debug symbols.

To run directly without building separately:
```bash
cargo run -p goose-cli -- --help
cargo run -p goose-cli -- session
```

## Understanding the Components

The goose CLI depends on these workspace crates:

1. **goose** (`crates/goose`) - Core library containing:
   - Agent logic and execution
   - LLM provider integrations (OpenAI, Anthropic, Google, etc.)
   - Extension system
   - Recipe management
   - Session handling

2. **goose-cli** (`crates/goose-cli`) - Command-line interface:
   - CLI commands and argument parsing
   - Interactive terminal interface
   - Configuration management
   - Web interface server

3. **goose-mcp** (`crates/goose-mcp`) - MCP server support:
   - Model Context Protocol implementation
   - MCP server integrations

4. **goose-bench** (`crates/goose-bench`) - Benchmarking utilities:
   - Performance testing
   - Benchmark scenarios

When you build `goose-cli`, Cargo automatically builds all dependencies.

## Testing Your Build

### Run Unit Tests

```bash
# Test all goose components
cargo test -p goose -p goose-cli -p goose-mcp -p goose-bench

# Test just the CLI
cargo test -p goose-cli
```

### Run Integration Tests

```bash
# Configure with a test provider
./target/release/goose configure

# Try a simple command
./target/release/goose run -t "echo hello"

# Start an interactive session
./target/release/goose session
```

## Building for Different Targets

### Cross-Compilation

To build for a different architecture:

```bash
# List available targets
rustup target list

# Add a target (example: ARM64 Linux)
rustup target add aarch64-unknown-linux-gnu

# Build for that target
cargo build --release -p goose-cli --target aarch64-unknown-linux-gnu
```

### Windows Cross-Compilation (from Linux)

```bash
# Install MinGW toolchain
sudo apt install mingw-w64

# Add Windows target
rustup target add x86_64-pc-windows-gnu

# Build
cargo build --release -p goose-cli --target x86_64-pc-windows-gnu
```

The binary will be at `./target/x86_64-pc-windows-gnu/release/goose.exe`

## Optimizing Build Size

For a smaller binary:

```bash
# Enable size optimization in profile
export CARGO_PROFILE_RELEASE_OPT_LEVEL=z
export CARGO_PROFILE_RELEASE_LTO=true
export CARGO_PROFILE_RELEASE_STRIP=true

cargo build --release -p goose-cli
```

Or strip an existing binary:
```bash
strip target/release/goose
```

## Troubleshooting

### Missing protobuf compiler

**Error:** `Could not find protoc`

**Solution:** Install the protobuf compiler:
```bash
# Debian/Ubuntu
sudo apt install protobuf-compiler

# macOS
brew install protobuf

# Or download from: https://github.com/protocolbuffers/protobuf/releases
```

### OpenSSL issues

**Error:** `Could not find OpenSSL`

**Solution (Linux):**
```bash
# Debian/Ubuntu
sudo apt install libssl-dev pkg-config

# Fedora/RHEL
sudo dnf install openssl-devel
```

**Solution (macOS):**
```bash
brew install openssl
export OPENSSL_DIR=$(brew --prefix openssl)
```

### Linker errors on Windows

**Error:** `linker 'link.exe' not found`

**Solution:** Install Visual Studio Build Tools or use the MinGW toolchain:
1. Download [Visual Studio Build Tools](https://visualstudio.microsoft.com/downloads/)
2. Install "Desktop development with C++"
3. Restart your terminal

Or use Git Bash/MSYS2 which includes MinGW.

### Out of memory during compilation

**Solution:** Reduce parallel jobs:
```bash
cargo build --release -p goose-cli -j 2
```

### Slow compilation

**Solution:** Use a faster linker:

**Linux:**
```bash
# Install mold linker
sudo apt install mold  # or build from source

# Use it
cargo build --release -p goose-cli
```

**macOS:**
```bash
# Install lld
brew install llvm

# Configure in ~/.cargo/config.toml
```

## Development Workflow

When developing the goose CLI:

1. **Make code changes** in `crates/goose-cli/` or `crates/goose/`

2. **Check compilation:**
   ```bash
   cargo check -p goose-cli
   ```

3. **Run tests:**
   ```bash
   cargo test -p goose-cli
   ```

4. **Format code:**
   ```bash
   cargo fmt --all
   ```

5. **Run linter:**
   ```bash
   ./scripts/clippy-lint.sh
   ```

6. **Test your changes:**
   ```bash
   cargo run -p goose-cli -- session
   ```

7. **Build release version:**
   ```bash
   cargo build --release -p goose-cli
   ```

## Build Cache

Cargo caches compiled dependencies in `~/.cargo/registry` and `./target`. To clean:

```bash
# Remove target directory (frees space, next build will be slow)
cargo clean

# Remove and rebuild
cargo clean && cargo build --release -p goose-cli
```

## Next Steps

After building the CLI:

1. **Configure your LLM provider:**
   ```bash
   goose configure
   ```

2. **Read the documentation:**
   - [Installation Guide](https://block.github.io/goose/docs/getting-started/installation)
   - [Provider Configuration](https://block.github.io/goose/docs/getting-started/providers)
   - [Using Extensions](https://block.github.io/goose/docs/getting-started/using-extensions)

3. **Try goose:**
   ```bash
   goose session
   ```

4. **Explore recipes:**
   ```bash
   goose recipe list
   ```

## Contributing

If you're building from source to contribute to goose:

1. Read [CONTRIBUTING.md](CONTRIBUTING.md)
2. Fork the repository
3. Create a branch for your changes
4. Make your changes and test thoroughly
5. Submit a pull request

## Related Documentation

- [BUILDING_LINUX.md](BUILDING_LINUX.md) - Building the Desktop application
- [BUILDING_DOCKER.md](BUILDING_DOCKER.md) - Building with Docker
- [CONTRIBUTING.md](CONTRIBUTING.md) - Contributing guide
- [README.md](README.md) - Project overview

## Getting Help

- [Discord Community](https://discord.gg/goose-oss)
- [GitHub Issues](https://github.com/block/goose/issues)
- [Documentation](https://block.github.io/goose/docs)
