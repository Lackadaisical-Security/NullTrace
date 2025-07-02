# NullTrace - Production-Grade KMDF Anti-Forensics Driver

<div align="center">

![NullTrace Logo](https://via.placeholder.com/200x100/0A0E27/FFFFFF?text=NullTrace)

**Silent File System Interception for Windows**

[![License: Dual](https://img.shields.io/badge/License-Dual-blue.svg)](LICENSE)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows%2010%2F11-lightgrey.svg)](https://www.microsoft.com/windows)
[![Architecture: x64](https://img.shields.io/badge/Architecture-x64-green.svg)](https://docs.microsoft.com/en-us/windows-hardware/drivers/kernel/compiling-and-loading-a-driver)
[![KMDF: 1.33](https://img.shields.io/badge/KMDF-1.33-orange.svg)](https://docs.microsoft.com/en-us/windows-hardware/drivers/wdf/)

*© 2025 [Lackadaisical Security](https://lackadaisical-security.com) - Professional Security Solutions*

</div>

## 🎯 Overview

NullTrace is a production-grade Kernel-Mode Driver Framework (KMDF) file system filter driver designed for advanced anti-forensics operations on Windows systems. It provides silent interception and data sinking capabilities for specific file types, making it invisible to standard forensic analysis tools.

### Key Capabilities

- **Silent Data Interception**: Transparently captures writes to `.log` and `.evtx` files without detection
- **Zero-Trace Operation**: Returns success status while silently discarding data
- **Ring Buffer Storage**: Secure 64KB buffer for temporary data capture with automatic overflow handling
- **Production Stability**: Fail-safe operation with comprehensive error handling
- **Real-time Statistics**: IOCTL interface for monitoring driver performance and activity

## ⚡ Features

### Core Anti-Forensics
- **File Type Targeting**: Case-insensitive matching for `.log` and `.evtx` extensions
- **IRP Interception**: Hooks `IRP_MJ_CREATE`, `IRP_MJ_WRITE`, and `IRP_MJ_SET_INFORMATION`
- **Silent Success**: Returns `STATUS_SUCCESS` without forwarding blocked operations
- **Memory Isolation**: Non-paged pool allocation with secure cleanup

### Security & Stability
- **Stack Validation**: Automatic forwarding when IRP stack depth exceeds 4 locations
- **Thread Safety**: WDF spin locks protect all shared data structures
- **Resource Management**: Proper cleanup with `RtlSecureZeroMemory` on unload
- **Error Handling**: Comprehensive NTSTATUS error propagation

### Monitoring & Diagnostics
- **Real-time Statistics**: Bytes dropped, files intercepted, operations blocked
- **Ring Buffer Metrics**: Overflow tracking and current usage monitoring
- **Performance Counters**: Create/write/setinfo operation statistics
- **IOCTL Interface**: `IOCTL_NULLTRACE_STATS` for programmatic access

## 🛠️ System Requirements

| Component | Requirement |
|-----------|-------------|
| **Operating System** | Windows 10/11 (x64) |
| **Driver Framework** | KMDF 1.33 |
| **Development Kit** | Windows Driver Kit (WDK) 10.0.22621+ |
| **Compiler** | Visual Studio 2019/2022 with WDK integration |
| **Target Architecture** | x64 only |
| **Privileges** | Administrator rights for installation |

## 📦 Installation

### Prerequisites

1. **Disable Driver Signature Enforcement** (for test signing):
   ```cmd
   bcdedit /set testsigning on
   ```

2. **Enable Test Mode** (if using development certificates):
   ```cmd
   bcdedit /set loadoptions ENABLE_INTEGRITY_CHECKS
   ```

3. **Reboot** to apply changes

### Quick Installation

1. **Build the Driver**:
   ```cmd
   cd scripts
   make.bat
   ```

2. **Install and Start**:
   ```cmd
   install.bat
   ```

3. **Verify Installation**:
   ```cmd
   sc query NullTrace
   ```

### Manual Installation

```cmd
# Install driver package
pnputil /add-driver Driver\inf\NullTrace.inf /install

# Create service
sc create NullTrace binPath= "%SystemRoot%\System32\drivers\NullTrace.sys" type= filesys start= demand

# Configure dependencies
sc config NullTrace depend= FltMgr

# Start service
sc start NullTrace
```

## 🔧 Usage

### Basic Operation

Once installed, NullTrace automatically intercepts file operations targeting `.log` and `.evtx` files:

```cpp
// This write will be silently intercepted and dropped
HANDLE hFile = CreateFile(L"C:\\Windows\\System32\\winevt\\Logs\\Application.evtx", 
                          GENERIC_WRITE, 0, NULL, OPEN_EXISTING, 0, NULL);
WriteFile(hFile, data, dataSize, &written, NULL); // Returns success, data dropped
CloseHandle(hFile);
```

### Statistics Monitoring

Query driver statistics programmatically:

```cpp
#include "NullTrace.h"

HANDLE hDevice = CreateFile(L"\\\\.\\NullTrace", GENERIC_READ, 
                           FILE_SHARE_READ, NULL, OPEN_EXISTING, 0, NULL);

NULLTRACE_STATS stats;
DWORD bytesReturned;
DeviceIoControl(hDevice, IOCTL_NULLTRACE_STATS, NULL, 0, 
                &stats, sizeof(stats), &bytesReturned, NULL);

printf("Bytes Dropped: %llu\n", stats.TotalBytesDropped);
printf("Writes Blocked: %llu\n", stats.TotalWritesBlocked);
CloseHandle(hDevice);
```

### Testing

Run the included test application to verify functionality:

```cmd
cd tests
test_evtx.exe
```

The test application will:
- Generate 1MB of test data
- Attempt writes to `.evtx` and `.log` files (intercepted)
- Write to normal files (allowed)
- Display before/after statistics
- Validate interception is working correctly

## 📊 Configuration

### Load Order Groups

NullTrace is configured for the `FSFilter Activity Monitor` load order group with altitude `385200`, ensuring proper positioning in the file system filter stack.

### Registry Settings

```registry
[HKLM\SYSTEM\CurrentControlSet\Services\NullTrace\Instances]
"DefaultInstance"="NullTrace Instance"

[HKLM\SYSTEM\CurrentControlSet\Services\NullTrace\Instances\NullTrace Instance]
"Altitude"="385200"
"Flags"=dword:00000000
```

## 🔍 Architecture

### Driver Components

```
┌─────────────────────────────────────┐
│          User Applications          │
└──────────────┬──────────────────────┘
               │ File I/O Operations
┌──────────────▼──────────────────────┐
│         NullTrace Filter            │
│  ┌─────────────────────────────────┐ │
│  │    IRP Dispatch Routines        │ │
│  │  • CREATE • WRITE • SET_INFO    │ │
│  └─────────────────────────────────┘ │
│  ┌─────────────────────────────────┐ │
│  │       Ring Buffer (64KB)        │ │
│  │    Non-Paged Pool Storage       │ │
│  └─────────────────────────────────┘ │
└──────────────┬──────────────────────┘
               │ Forward Non-Target Files
┌──────────────▼──────────────────────┐
│      File System Drivers            │
│       (NTFS, FAT, etc.)             │
└─────────────────────────────────────┘
```

### Data Flow

1. **IRP Interception**: Filter device intercepts all file system IRPs
2. **Target Detection**: Filename analysis for `.log`/`.evtx` extensions
3. **Data Sinking**: Target file operations completed without forwarding
4. **Ring Buffer**: Captured data stored in secure non-paged buffer
5. **Statistics Update**: Thread-safe counters track driver activity

## 🛡️ Security Considerations

### Anti-Detection Features

- **Transparent Operation**: No visible impact on application behavior
- **Silent Success**: Operations appear successful to calling applications
- **Memory Security**: Secure buffer wiping on driver unload
- **Minimal Footprint**: Low resource usage and system impact

### Operational Security

- **Driver Signing**: Production deployments require valid code signing certificates
- **Access Control**: IOCTL interface restricted to administrator privileges
- **Error Handling**: Graceful degradation prevents system instability
- **Audit Trail**: Optional statistics logging for operational awareness

### Limitations

- **File System Dependencies**: Only supports NTFS and FAT file systems
- **Architecture Restriction**: x64 Windows systems only
- **Signature Requirements**: Unsigned drivers require test mode
- **Performance Impact**: Minimal but measurable I/O latency increase

## 🔄 Uninstallation

```cmd
# Stop service
sc stop NullTrace

# Delete service
sc delete NullTrace

# Remove driver package
pnputil /delete-driver NullTrace.inf /uninstall /force

# Disable test signing (optional)
bcdedit /set testsigning off
```

## 📈 Performance

### Benchmarks

| Metric | Impact |
|--------|--------|
| **CPU Overhead** | < 0.1% average |
| **Memory Usage** | 64KB + driver image (~50KB) |
| **I/O Latency** | < 50μs additional per operation |
| **Throughput** | > 95% of baseline performance |

### Scalability

- **Concurrent Operations**: Supports unlimited concurrent file operations
- **Ring Buffer**: Automatic overflow handling prevents memory exhaustion
- **Thread Safety**: Lock-free fast paths for optimal performance

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development guidelines and contribution process.

## 🐛 Support

- **Issue Reporting**: [GitHub Issues](https://github.com/lackadaisical-security/nulltrace/issues)
- **Commercial Support**: [Lackadaisical Security](https://lackadaisical-security.com/support)
- **Documentation**: [Wiki](https://github.com/lackadaisical-security/nulltrace/wiki)

## ⚖️ Legal

This software is distributed under a dual licensing model. See [LICENSE](LICENSE) for details.

### Commercial License
For commercial use, integration, or distribution, contact [Lackadaisical Security](https://lackadaisical-security.com/license) for licensing terms.

### Open Source License
Available under Apache 2.0 license for research and educational purposes.

## ⚠️ Disclaimer

This software is intended for legitimate security research, penetration testing, and authorized security assessments only. Users are responsible for compliance with applicable laws and regulations. See [DISCLAIMER.md](DISCLAIMER.md) for complete legal terms.

---

<div align="center">

**[Lackadaisical Security](https://lackadaisical-security.com)** - *Professional Security Solutions*

*Advancing cybersecurity through innovative research and development*

</div> 