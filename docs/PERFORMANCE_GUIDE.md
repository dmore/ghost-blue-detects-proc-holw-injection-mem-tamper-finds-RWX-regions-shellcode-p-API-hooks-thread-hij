# Performance Optimization Guide

## Overview

Ghost is designed for process injection detection with configurable performance characteristics. This guide covers actual optimization strategies and expected performance.

## Performance Characteristics

### Expected Detection Engine Performance

- **Process Enumeration**: 10-50ms for all system processes
- **Memory Region Analysis**: 1-5ms per process (platform-dependent)
- **Thread Enumeration**: 1-10ms per process
- **Detection Heuristics**: <1ms per process
- **Memory Usage**: ~10-20MB for core engine

**Note**: Actual performance varies significantly by:
- Number of processes (100-1000+ typical)
- Memory region count per process
- Thread count per process
- Platform (Windows APIs vs Linux procfs)

### Configuration Options

#### 1. Selective Detection

```rust
use ghost_core::config::DetectionConfig;

// Disable expensive detections for performance
let mut config = DetectionConfig::default();
config.rwx_detection = true;      // Fast: O(n) memory regions
config.shellcode_detection = false; // Skip pattern matching
config.hook_detection = false;    // Skip module enumeration
config.thread_detection = true;   // Moderate: thread enum
config.hollowing_detection = false; // Skip heuristics
```

#### 2. Preset Modes

```rust
// Fast scanning mode
let config = DetectionConfig::performance_mode();

// Thorough scanning mode
let config = DetectionConfig::thorough_mode();
```

#### 3. Process Filtering

```rust
// Skip system processes
config.skip_system_processes = true;

// Limit memory scan size
config.max_memory_scan_size = 10 * 1024 * 1024; // 10MB per process
```

## Performance Considerations

### Platform-Specific Performance

**Windows**:
- CreateToolhelp32Snapshot: Single syscall, fast
- VirtualQueryEx: Iterative, slower for processes with many regions
- ReadProcessMemory: Cross-process, requires proper handles
- NtQueryInformationThread: Undocumented API call per thread

**Linux**:
- /proc enumeration: Directory reads, fast
- /proc/[pid]/maps parsing: File I/O, moderate
- /proc/[pid]/mem reading: Requires ptrace or same user
- /proc/[pid]/task parsing: Per-thread file I/O

**macOS**:
- sysctl KERN_PROC_ALL: Single syscall, fast
- Memory/thread analysis: Not yet implemented

### Running Tests

```bash
# Run all tests including performance assertions
cargo test

# Run tests with timing output
cargo test -- --nocapture
```

## Tuning Guidelines

### For Continuous Monitoring

1. **Adjust scan interval**: Configure `scan_interval_ms` in DetectionConfig
2. **Skip system processes**: Set `skip_system_processes = true`
3. **Limit memory scans**: Reduce `max_memory_scan_size`
4. **Disable heavy detections**: Turn off hook_detection and shellcode_detection

### For One-Time Analysis

1. **Enable all detections**: Use `DetectionConfig::thorough_mode()`
2. **Full memory scanning**: Increase `max_memory_scan_size`
3. **Include system processes**: Set `skip_system_processes = false`

## Platform-Specific Optimizations

### Windows

- Run as Administrator for full process access
- Use `PROCESS_QUERY_LIMITED_INFORMATION` when `PROCESS_QUERY_INFORMATION` fails
- Handle access denied errors gracefully (system processes)

### Linux

- Run with appropriate privileges (root or CAP_SYS_PTRACE)
- Handle permission denied for /proc/[pid]/mem gracefully
- Consider using process groups for batch access

### macOS

- Limited functionality (process enumeration only)
- Most detection features require kernel extensions or Endpoint Security framework

## Troubleshooting Performance Issues

### High CPU Usage

1. Reduce scan frequency (`scan_interval_ms`)
2. Disable thread analysis for each scan
3. Skip memory region enumeration
4. Filter out known-good processes

### High Memory Usage

1. Reduce baseline cache size (limited processes tracked)
2. Clear detection history periodically
3. Limit memory reading buffer sizes

### Slow Detection Response

1. Disable hook detection (expensive module enumeration)
2. Skip shellcode pattern matching
3. Use performance preset mode

## Current Implementation Limits

**What's NOT implemented**:
- No performance metrics collection system
- No Prometheus/monitoring integration
- No SIMD-accelerated pattern matching
- No parallel/async process scanning (single-threaded)
- No LRU caching of results
- No batch processing APIs

**Current architecture**:
- Sequential process scanning
- Simple HashMap for baseline tracking
- Basic confidence scoring
- Manual timer-based intervals (TUI)

## Testing Performance

```rust
#[test]
fn test_detection_performance() {
    use std::time::Instant;

    let mut engine = DetectionEngine::new().unwrap();
    let process = ProcessInfo::new(1234, 4, "test.exe".to_string());
    let regions = vec![/* test regions */];

    let start = Instant::now();
    for _ in 0..100 {
        engine.analyze_process(&process, &regions, None);
    }
    let duration = start.elapsed();

    // Should complete 100 analyses in under 100ms
    assert!(duration.as_millis() < 100);
}
```

## Best Practices

1. **Start with defaults**: Use `DetectionConfig::default()` initially
2. **Profile specific modules**: Identify which detection is slow
3. **Adjust based on needs**: Disable features you don't need
4. **Handle errors gracefully**: Processes may exit during scan
5. **Test on target hardware**: Performance varies by system

## Future Performance Improvements

Potential enhancements (not yet implemented):
- Parallel process analysis using rayon
- Async I/O for file system operations (Linux)
- Result caching with TTL
- Incremental scanning (only changed processes)
- Memory-mapped file parsing
- SIMD pattern matching for shellcode