# Ghost

Ghost is a process injection detection tool written in Rust. It watches running processes and tries to catch suspicious stuff like code injection, memory manipulation, and other tricks that malware uses to hide.

## What it does

The main idea is simple: scan processes and look for weird memory patterns, hooked functions, shellcode, and other signs that something's been tampered with. It works on Windows, Linux, and macOS (though Windows support is the most complete right now).

Some of the things it can detect:

- **Memory regions with read-write-execute permissions** - Usually a red flag
- **Shellcode patterns** - Common instruction sequences found in injected code
- **Process hollowing** - When a legit process gets gutted and replaced with malicious code
- **API hooks** - Functions that have been redirected by inline patches or IAT modifications
- **Thread hijacking** - Threads that are redirected to execute shellcode
- **APC injection** - Malicious code queued via Asynchronous Procedure Calls
- **YARA signatures** - Matches against known malware patterns and payloads

It also maps detected behaviors to the MITRE ATT&CK framework, which is helpful if you're documenting threats or writing reports.

## Screenshots

### Detection Dashboard
![Ghost TUI - Detection Overview](assets/screenshot1.png)

### Active Threats
![Detected Malware](assets/screenshot2.png)

### Live Monitoring
![Real-time Process Analysis Logs](assets/screenshot3.png)

## Building it

You'll need Rust installed (1.70 or newer). Then:

```bash
cargo build --release
```

On Windows, you'll also need the MSVC build tools. Linux needs basic dev tools (gcc, etc.). macOS needs Xcode command line tools.

## Running it

There are two interfaces: a command-line tool and an interactive terminal UI.

**CLI:**

```bash
# Scan all processes
cargo run --bin ghost-cli --release

# Target one process
cargo run --bin ghost-cli --release -- --pid 1234

# Output results as JSON
cargo run --bin ghost-cli --release -- --format json

# Use a config file
cargo run --bin ghost-cli --release -- --config ghost.toml
```

**TUI:**

```bash
cargo run --bin ghost-tui --release
```

The TUI gives you a dashboard with live stats, detection history, and you can navigate around with keyboard shortcuts (Tab to switch views, Q to quit).

## Configuration

You can tweak behavior with a TOML config file. Check `examples/ghost.toml` for a starting point. You can enable/disable specific detection methods, set confidence thresholds, skip system processes, and control how often it scans.

Example config snippet:

```toml
shellcode_detection = true
hollowing_detection = true
hook_detection = true
confidence_threshold = 0.3
skip_system_processes = true
scan_interval_ms = 2000
```

## What the results mean

When Ghost finds something suspicious, it assigns a threat level: Clean, Low, Medium, High, or Critical. This is based on how many indicators it found and how serious they are.

High confidence doesn't always mean malware - some legit software does weird stuff with memory too. Use your judgment and investigate further if needed.

## Platform differences

**Windows:** Pretty much everything works. Process enumeration, memory reading, hook detection, PE validation, etc.

**Linux:** Works but relies on procfs (`/proc`). Can detect LD_PRELOAD shenanigans and ptrace-based injection. eBPF support is stubbed out for now.

**macOS:** Basic process enumeration works. Memory reading is implemented but not as feature-complete as Windows. Some detection methods don't apply or aren't implemented yet.

## Performance

It's designed to be fast enough for continuous monitoring. A full system scan (200 processes) usually takes under 5 seconds. Memory enumeration per process is around 50-100ms. The detection engine itself adds about 5-10ms per analysis.

## YARA rules

The tool includes YARA rule integration. Rules are stored in the `rules/` directory and cover common malware families like Metasploit, Cobalt Strike, generic shellcode patterns, and evasion techniques. You can add your own rules - just drop `.yar` files in that folder.

## Exit codes

- 0 = Everything looks clean
- 1 = Found suspicious processes
- 2 = Something went wrong (error during scan)

## Limitations

This is a userspace tool. It can't see kernel-level manipulation without help (like a driver on Windows or eBPF on Linux - which isn't fully implemented yet).

Some legit programs will trigger false positives. For example, game anti-cheat software, debuggers, sandboxes, and even browsers with JIT compilers can show up as suspicious because they do memory tricks.

macOS support is limited compared to Windows. Some advanced features just aren't there yet because the APIs are different and less documented.

## Documentation

There's more detail in the `docs/` folder:

- `DETECTION_METHODS.md` - Explains how each detection technique works
- `MITRE_ATTACK_COVERAGE.md` - Lists which ATT&CK techniques are covered
- `PERFORMANCE_GUIDE.md` - Tips for tuning performance

Also check out `CONTRIBUTING.md` if you want to contribute, and `SECURITY.md` for the security policy.

## License

MIT. See the LICENSE file.

## A note on usage

This tool is for security research, testing your own systems, and catching actual threats. Don't use it on systems you don't own or don't have permission to test. Be responsible.

Also, if you're investigating a real incident, remember that malware can detect when it's being analyzed and might behave differently or shut down. Ghost tries to be stealthy but there's no guarantee advanced malware won't notice.
