# Ghost

Cross-platform process injection detection framework written in Rust.

## Overview

Ghost is a security framework for detecting process injection, memory manipulation, and suspicious process behavior. It provides memory analysis, behavioral monitoring, and MITRE ATT&CK technique mapping for security research and defensive purposes.

## Features

- **Memory Analysis**: RWX region detection, shellcode pattern scanning, memory protection analysis
- **MITRE ATT&CK Mapping**: Technique identification using the ATT&CK framework
- **Cross-platform Support**:
  - **Windows**: Process enumeration, memory reading (ReadProcessMemory), thread analysis (NtQueryInformationThread), inline hook detection, PE header validation
  - **Linux**: Process enumeration via procfs, memory region analysis (/proc/[pid]/maps), thread state monitoring, LD_PRELOAD detection, ptrace detection
  - **macOS**: Process enumeration via sysctl/KERN_PROC_ALL
- **Real-time Monitoring**: Continuous scanning with configurable intervals
- **Threat Intelligence**: IOC storage and correlation framework

## Architecture

```
ghost/
├── ghost-core/     # Core detection engine and platform abstractions
├── ghost-cli/      # Command-line interface
├── ghost-tui/      # Interactive terminal UI (Ratatui-based)
├── examples/       # Configuration examples
└── docs/           # Technical documentation
```

### Core Modules

- **Detection Engine** ([detection.rs](ghost-core/src/detection.rs)): Orchestrates analysis and threat scoring
- **Memory Analysis** ([memory.rs](ghost-core/src/memory.rs)): Platform-specific memory enumeration and reading
- **Process Enumeration** ([process.rs](ghost-core/src/process.rs)): Cross-platform process listing
- **Thread Analysis** ([thread.rs](ghost-core/src/thread.rs)): Thread enumeration with start address and creation time
- **Hook Detection** ([hooks.rs](ghost-core/src/hooks.rs)): Inline hook detection via JMP pattern analysis
- **MITRE ATT&CK** ([mitre.rs](ghost-core/src/mitre.rs)): Technique mapping and categorization
- **Configuration** ([config.rs](ghost-core/src/config.rs)): TOML-based configuration with validation

## Supported Detection Techniques

### Process Injection (T1055)
- RWX memory region detection
- Private executable memory analysis
- Thread count anomaly detection
- Inline hook detection (JMP patches on ntdll.dll, kernel32.dll, user32.dll)
- LD_PRELOAD and LD_LIBRARY_PATH detection (Linux)
- Ptrace injection detection (Linux)
- SetWindowsHookEx hook enumeration
- Thread hijacking indicators (T1055.003)
- Process hollowing detection with PE header validation (T1055.012)

### Memory Analysis
- Memory protection flags (R/W/X combinations)
- Region type classification (IMAGE, PRIVATE, MAPPED, HEAP, STACK)
- Small executable region detection (shellcode indicators)
- Memory region size anomalies

### Behavioral Monitoring
- Thread count changes from baseline
- New thread creation detection
- Process parent-child relationships
- System process identification

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

## Platform Support Matrix

| Feature | Windows | Linux | macOS |
|---------|---------|-------|-------|
| Process Enumeration | CreateToolhelp32Snapshot | /proc filesystem | sysctl KERN_PROC_ALL |
| Memory Enumeration | VirtualQueryEx | /proc/[pid]/maps | Not implemented |
| Memory Reading | ReadProcessMemory | /proc/[pid]/mem | Not implemented |
| Thread Enumeration | Thread32First/Next | /proc/[pid]/task | Not implemented |
| Thread Start Address | NtQueryInformationThread | /proc/[pid]/task/[tid]/syscall | Not implemented |
| Thread Creation Time | GetThreadTimes | /proc/[pid]/task/[tid]/stat | Not implemented |
| Hook Detection | Inline JMP pattern scanning | LD_PRELOAD/ptrace detection | Not applicable |
| PE Header Validation | Full PE validation | Not applicable | Not applicable |
| Process Path | GetProcessImageFileNameW | /proc/[pid]/exe | proc_pidpath |

## Status

Active development. Core detection engine functional with cross-platform abstractions. Windows support most complete. Linux support via procfs. macOS has process enumeration but limited memory/thread analysis.

### Known Limitations

- macOS memory enumeration and reading not yet implemented (requires vm_read and mach_vm_region)
- Windows SetWindowsHookEx chain enumeration requires parsing undocumented USER32.dll structures
- Shellcode pattern matching currently uses heuristics (no actual signature database)
- No kernel-level monitoring (all userspace APIs)
