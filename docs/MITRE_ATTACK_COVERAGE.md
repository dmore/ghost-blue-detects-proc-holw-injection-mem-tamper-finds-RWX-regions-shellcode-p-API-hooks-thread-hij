# MITRE ATT&CK Detection Coverage

Ghost detection engine coverage mapped to MITRE ATT&CK framework techniques.

## Process Injection (T1055)

### T1055.001 - Dynamic-link Library Injection

- **Detection**: Hook-based injection detection (`hooks.rs`)
- **Indicators**: 
  - SetWindowsHookEx API monitoring
  - Suspicious DLL loading patterns
  - Global hook chain analysis
- **Confidence**: High (0.8-0.9)

### T1055.002 - Portable Executable Injection  

- **Detection**: Shellcode pattern detection (`shellcode.rs`)
- **Indicators**:
  - PE headers in private memory regions
  - Meterpreter payload signatures
  - High entropy executable regions
- **Confidence**: High (0.7-0.9)

### T1055.003 - Thread Execution Hijacking

- **Detection**: Thread analysis (`thread.rs`, `detection.rs`)
- **Indicators**:
  - Threads with unusual start addresses
  - High ratio of recently created threads
  - Thread count anomalies
- **Confidence**: Medium (0.5-0.7)

### T1055.004 - Asynchronous Procedure Call

- **Detection**: Memory pattern analysis
- **Indicators**:
  - Suspicious memory layout changes
  - RWX region proliferation
  - Thread creation spikes
- **Confidence**: Medium (0.4-0.6)

### T1055.012 - Process Hollowing

- **Detection**: Comprehensive hollowing detection (`hollowing.rs`)
- **Indicators**:
  - Unmapped main executable image
  - Suspicious memory gaps (>16MB)
  - PE header validation (DOS/NT signatures)
  - Image base mismatches
  - Corrupted PE structures
  - Unusual entry point locations
  - Memory layout anomalies
- **Confidence**: Very High (0.8-1.0)

## Defense Evasion (TA0005)

### T1027 - Obfuscated Files or Information

- **Detection**: Entropy analysis in shellcode detector
- **Indicators**:
  - High entropy regions (>7.0 Shannon entropy)
  - Encrypted/packed code patterns
- **Confidence**: Medium (0.6-0.8)

### T1055 - Process Injection (General)

- **Detection**: Multi-layered approach across all modules
- **Indicators**: Combination of all injection-specific indicators
- **Confidence**: Varies by technique

### T1036 - Masquerading

- **Detection**: Process metadata analysis
- **Indicators**:
  - Process name/path mismatches
  - Suspicious parent-child relationships
- **Confidence**: Low-Medium (0.3-0.6)

## Execution (TA0002)

### T1106 - Native API

- **Detection**: Memory pattern analysis, syscall indicators
- **Indicators**:
  - Direct syscall usage patterns
  - Unusual API call sequences
- **Confidence**: Medium (0.5-0.7)

### T1055 - Process Injection

- **Detection**: Primary focus of Ghost detection engine
- **Coverage**: Comprehensive across all sub-techniques

## Detection Methodology

### Heuristic Analysis

1. **Memory Layout Analysis**
   - RWX region detection
   - Memory gap analysis
   - Region size anomalies

2. **Behavioral Patterns**
   - Thread creation patterns
   - Hook installation monitoring
   - Process lifecycle anomalies

3. **Signature Matching**
   - Known shellcode patterns
   - Malware family signatures
   - API usage fingerprints

### Confidence Scoring

- **0.9-1.0**: Very High - Multiple strong indicators
- **0.7-0.8**: High - Clear malicious patterns
- **0.5-0.6**: Medium - Suspicious but may be legitimate
- **0.3-0.4**: Low - Anomalous but likely false positive
- **0.0-0.2**: Very Low - Minimal suspicious activity

## Coverage Matrix

| Technique | Detection Module | Implementation Status | Test Coverage |
|-----------|------------------|----------------------|---------------|
| T1055.001 | hooks.rs | ✅ Inline hooks + Linux LD_PRELOAD | ❌ Basic |
| T1055.002 | shellcode.rs | ⚠️ Heuristic only | ✅ Basic |
| T1055.003 | thread.rs | ✅ Thread enumeration | ✅ Unit tests |
| T1055.004 | detection.rs | ⚠️ Heuristic only | ✅ Basic |
| T1055.012 | hollowing.rs | ✅ PE header validation | ❌ Pending |
| T1027 | shellcode.rs | ⚠️ Basic patterns | ❌ Pending |
| T1036 | process.rs | ❌ Not implemented | ❌ Pending |
| T1106 | detection.rs | ❌ Not implemented | ❌ Pending |

**Implementation Status Legend**:
- ✅ Complete: Full implementation with actual API calls
- ⚠️ Partial: Heuristic-based or incomplete implementation
- ❌ Not implemented: Placeholder or missing

## Current Implementation Details

### What's Actually Implemented

1. **Memory Analysis** (memory.rs)
   - Windows: VirtualQueryEx, ReadProcessMemory
   - Linux: /proc/[pid]/maps parsing, /proc/[pid]/mem reading
   - macOS: mach_vm_region, mach_vm_read_overwrite

2. **Thread Analysis** (thread.rs)
   - Windows: Thread32First/Next, NtQueryInformationThread, GetThreadTimes
   - Linux: /proc/[pid]/task enumeration, stat parsing
   - macOS: Not implemented (task_threads needed)

3. **Hook Detection** (hooks.rs)
   - Windows: Inline hook detection via JMP pattern scanning
   - Linux: LD_PRELOAD detection, LD_LIBRARY_PATH monitoring, ptrace detection
   - Detects suspicious library loading from /tmp/, /dev/shm/, etc.
   - Does NOT enumerate SetWindowsHookEx chains on Windows
   - No IAT/EAT hook scanning (pattern detection only)

4. **Process Hollowing Detection** (hollowing.rs)
   - Windows: Full PE header validation (DOS/NT signatures, image base)
   - Detects corrupted PE structures
   - Detects image base mismatches
   - Memory layout anomaly detection
   - Memory gap analysis

5. **Process Enumeration** (process.rs)
   - Windows: CreateToolhelp32Snapshot
   - Linux: /proc filesystem
   - macOS: sysctl KERN_PROC_ALL

### What's NOT Implemented

- SetWindowsHookEx chain parsing (Windows)
- APC injection detection (Windows)
- IAT/EAT hook scanning (only inline hooks detected)
- process_vm_writev monitoring (Linux)
- Thread enumeration on macOS
- Automatic MITRE ATT&CK technique attribution (framework exists but no inference logic)
- ML-based behavioral analysis (requires trained models)
- Live threat intelligence feeds (framework exists but no connections)
- eBPF kernel monitoring (stub implementation behind feature flag)

## Future Enhancements

### High Priority

- **T1055.008** - Ptrace System Calls (Linux) - ✅ Basic detection implemented
- **T1055.013** - Process Doppelgänging
- **T1055.014** - VDSO Hijacking (Linux)
- Shellcode signature database

### Medium Priority

- **T1134** - Access Token Manipulation
- SetWindowsHookEx chain enumeration
- IAT/EAT hook scanning
- LD_PRELOAD detection (Linux) - ✅ Implemented

### Research Areas

- Behavioral analysis over time
- Process relationship analysis
- Integration with threat intelligence feeds

## References

- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Process Injection Techniques](https://attack.mitre.org/techniques/T1055/)
- [Windows Process Injection Research](https://www.endgame.com/blog/technical-blog/ten-process-injection-techniques-technical-survey-common-and-trending-process)
- [Linux Process Injection](https://blog.sektor7.net/#!res/2018/pure-in-memory-linux.md)

---

*Coverage updated: November 2024*  
*Next review: December 2024*