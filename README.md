# Ghost

Cross-platform process injection detection framework written in Rust.

## Overview

Ghost is a comprehensive security framework for detecting process injection, memory manipulation, and advanced evasion techniques in running processes. It combines kernel-level monitoring with behavioral analysis, machine learning, and threat intelligence to provide enterprise-grade detection capabilities.

## Features

- **Multi-layer detection**: Memory analysis, behavioral patterns, and ML-based anomaly detection
- **MITRE ATT&CK mapping**: Automatic technique classification using the ATT&CK framework
- **Threat intelligence**: Integration with threat feeds for IOC correlation and attribution
- **Cross-platform**: Windows (full support), Linux (with eBPF), macOS (planned)
- **Real-time monitoring**: Continuous scanning with configurable intervals
- **Low overhead**: Performance-optimized for production environments

## Architecture

```
ghost/
├── ghost-core/     # Core detection engine (21 modules)
├── ghost-cli/      # Command-line interface
├── ghost-tui/      # Interactive terminal UI
├── examples/       # Configuration examples
└── docs/           # Technical documentation
```

### Core Modules

- **Detection Engine**: Orchestrates all analysis components
- **Memory Analysis**: RWX region detection, shellcode patterns
- **Process Hollowing**: PE header validation, memory gap analysis
- **Thread Analysis**: Start address validation, behavioral patterns
- **Evasion Detection**: Anti-debugging, VM detection, obfuscation
- **MITRE ATT&CK Engine**: Technique mapping and threat actor profiling
- **Threat Intelligence**: IOC matching and campaign correlation

## Supported Detection Techniques

### Process Injection (T1055)
- RWX memory region detection
- Private executable memory analysis
- Remote thread creation monitoring
- SetWindowsHookEx injection (T1055.001)
- Thread hijacking (T1055.003)
- APC injection patterns (T1055.004)
- Process hollowing (T1055.012)
- Reflective DLL injection

### Evasion Techniques
- Anti-debugging detection
- Virtual machine detection attempts
- Code obfuscation analysis
- Timing-based analysis evasion
- Environment fingerprinting

### Behavioral Anomalies
- Thread count deviations
- Memory allocation patterns
- API call sequences
- Process relationship analysis

## Installation

### Requirements

- Rust 1.70+ (stable)
- Platform-specific dependencies:
  - **Windows**: MSVC Build Tools, Windows SDK
  - **Linux**: GCC/Clang, libelf-dev (for eBPF)
  - **macOS**: Xcode Command Line Tools

### Building

```bash
# Release build (recommended)
cargo build --release

# Development build
cargo build

# Run tests
cargo test

# Generate documentation
cargo doc --open
```

## Usage

### CLI

```bash
# Basic scan
cargo run --bin ghost-cli

# Target specific process
cargo run --bin ghost-cli -- --pid 1234

# JSON output
cargo run --bin ghost-cli -- --format json

# Load custom configuration
cargo run --bin ghost-cli -- --config examples/ghost.toml

# Show MITRE ATT&CK statistics
cargo run --bin ghost-cli -- --mitre-stats

# Verbose output with debug logging
cargo run --bin ghost-cli -- -v -d
```

### TUI (Terminal User Interface)

```bash
cargo run --bin ghost-tui
```

The TUI provides:
- Real-time process monitoring dashboard
- Detection history with threat levels
- System statistics and performance metrics
- Interactive process exploration
- Live system logs

**Keyboard Controls:**
- `Tab`: Switch between views
- `Up/Down`: Navigate lists
- `Enter`: Select item
- `R`: Force refresh
- `C`: Clear history
- `Q`: Quit

### Configuration

Create a configuration file (see `examples/ghost.toml`):

```toml
shellcode_detection = true
hollowing_detection = true
hook_detection = true
confidence_threshold = 0.3
skip_system_processes = true
max_memory_scan_size = 104857600  # 100MB
thread_analysis_enabled = true
evasion_detection = true
mitre_mapping = true
scan_interval_ms = 2000
```

## Exit Codes

- `0`: Clean scan, no suspicious activity
- `1`: Suspicious processes found
- `2`: Error occurred during scanning

## Performance

Ghost is designed for low-overhead monitoring:
- Memory enumeration: <100ms per process
- Thread analysis: <50ms per process
- Detection engine: <10ms per analysis
- Full system scan: <5s for 200 processes

## Documentation

- [Detection Methods](docs/DETECTION_METHODS.md)
- [MITRE ATT&CK Coverage](docs/MITRE_ATTACK_COVERAGE.md)
- [Performance Guide](docs/PERFORMANCE_GUIDE.md)
- [Research Framework](docs/RESEARCH_FRAMEWORK.md)
- [Build Instructions](BUILD.md)
- [Contributing Guidelines](CONTRIBUTING.md)
- [Security Policy](SECURITY.md)

## License

MIT License. See [LICENSE](LICENSE) for details.

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on:
- Code style (rustfmt, clippy)
- Performance requirements
- Testing standards
- Pull request process

## Security

Please review [SECURITY.md](SECURITY.md) for:
- Responsible disclosure policy
- Security considerations
- Threat model

## Status

Active development. Core detection engine stable. Windows support complete. Linux eBPF support in progress. macOS Endpoint Security framework planned.
