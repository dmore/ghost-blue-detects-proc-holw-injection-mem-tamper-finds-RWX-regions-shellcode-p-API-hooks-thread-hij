# Build Instructions

## Prerequisites

### Windows
- Rust toolchain (MSVC target)
- Visual Studio Build Tools with C++ workload
- Windows SDK (for windows crate bindings)

Install via:
```powershell
rustup default stable-msvc
```

### Linux
- Rust toolchain
- GCC/Clang
- libc development headers

```bash
# Debian/Ubuntu
sudo apt install build-essential

# RHEL/Fedora
sudo dnf groupinstall "Development Tools"
```

### macOS
- Rust toolchain
- Xcode Command Line Tools (for libc bindings)

```bash
xcode-select --install
```

## Building

```bash
# Release build (recommended for performance)
cargo build --release

# Debug build
cargo build

# Check for compilation errors without full build
cargo check
```

## Running

```bash
# CLI interface
cargo run --bin ghost-cli

# Terminal UI
cargo run --bin ghost-tui

# With arguments
cargo run --bin ghost-cli -- --pid 1234
cargo run --bin ghost-cli -- --config examples/ghost.toml
```

## Testing

```bash
# Run all tests
cargo test

# Run specific test module
cargo test --package ghost-core detection_tests

# Run with output
cargo test -- --nocapture
```

## Documentation

```bash
# Generate and open documentation
cargo doc --open

# Generate without dependencies
cargo doc --no-deps --open
```

## Platform Notes

- **Windows**: Requires elevated privileges (Administrator) for full process memory access
- **Linux**: Requires appropriate permissions to read /proc/[pid]/mem (root or ptrace capability)
- **macOS**: Some features require System Integrity Protection (SIP) to be adjusted for full functionality
