# Contributing to NullTrace

Thank you for your interest in contributing to NullTrace! We welcome contributions from the security research community and are committed to maintaining high-quality, production-ready code.

## 🎯 Contributing Overview

NullTrace is a production-grade Windows kernel driver for anti-forensics research. All contributions must meet enterprise-level quality standards and undergo thorough security review.

### Types of Contributions

- 🐛 **Bug Reports**: Detailed issue reports with reproduction steps
- 🔧 **Bug Fixes**: Patches for identified issues
- ⚡ **Performance Improvements**: Optimizations and efficiency enhancements  
- 📚 **Documentation**: Improvements to guides, comments, and examples
- 🧪 **Testing**: Enhanced test coverage and validation tools
- 💡 **Feature Requests**: Proposals for new functionality (pre-approved only)

## 📋 Before You Start

### Prerequisites

1. **Sign Contributor License Agreement (CLA)**
   - Required for all contributions
   - Available at: https://lackadaisical-security.com/cla
   - Email signed CLA to: legal@lackadaisical-security.com

2. **Development Environment**
   - Windows 10/11 x64 development machine
   - Visual Studio 2019/2022 with C++ workload
   - Windows Driver Kit (WDK) 10.0.22621 or later
   - Windows SDK 10.0.22621 or later
   - Git for Windows

3. **Security Clearance** (for sensitive contributions)
   - Background check may be required for core driver changes
   - Contact: security@lackadaisical-security.com

## 🛠️ Development Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub
# Clone your fork
git clone https://github.com/yourusername/nulltrace.git
cd nulltrace

# Add upstream remote
git remote add upstream https://github.com/lackadaisical-security/nulltrace.git
```

### 2. Environment Configuration

```cmd
# Enable test signing (required for development)
bcdedit /set testsigning on

# Set up Windows SDK paths
set WINDOWS_SDK_PATH=C:\Program Files (x86)\Windows Kits\10
set WDK_PATH=C:\Program Files (x86)\Windows Kits\10

# Configure Visual Studio
devenv /resetuserdata
```

### 3. Build Verification

```cmd
cd scripts
make.bat
```

Verify successful build with no errors or warnings.

## 📝 Coding Standards

### General Principles

- **Security First**: All code must be secure by design
- **Performance Critical**: Minimize performance impact
- **Stability Required**: No crashes or memory leaks
- **Maintainable**: Clear, well-documented code

### Code Style

#### File Headers
```c
//
// FileName.c - Brief Description
// © Lackadaisical Security, 2025
//
```

#### Function Documentation
```c
//
// Brief function description
//
NTSTATUS
FunctionName(
    _In_ PARAMETER_TYPE Parameter1,
    _Out_ PARAMETER_TYPE Parameter2
    )
{
    // Implementation
}
```

#### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| **Functions** | PascalCase with prefix | `NullTraceProcessIrp` |
| **Variables** | camelCase | `deviceExtension` |
| **Constants** | UPPER_SNAKE_CASE | `NULLTRACE_POOL_TAG` |
| **Types** | PascalCase with prefix | `NULLTRACE_DEVICE_EXTENSION` |
| **Macros** | UPPER_SNAKE_CASE | `NULLTRACE_MAX_BUFFER_SIZE` |

#### Code Formatting

- **Indentation**: 4 spaces (no tabs)
- **Line Length**: Maximum 120 characters
- **Braces**: Allman style (opening brace on new line)
- **Comments**: `//` for single line, `/* */` for blocks

#### Example Code Block
```c
NTSTATUS
NullTraceProcessRequest(
    _In_ PDEVICE_OBJECT DeviceObject,
    _In_ PIRP Irp
    )
{
    NTSTATUS status = STATUS_SUCCESS;
    PIO_STACK_LOCATION irpStack;
    PNULLTRACE_DEVICE_EXTENSION deviceExtension;

    //
    // Validate parameters
    //
    if (DeviceObject == NULL || Irp == NULL) {
        return STATUS_INVALID_PARAMETER;
    }

    //
    // Get current IRP stack location
    //
    irpStack = IoGetCurrentIrpStackLocation(Irp);
    deviceExtension = NullTraceGetDeviceExtension(DeviceObject);

    //
    // Process the request
    //
    switch (irpStack->MajorFunction) {
    case IRP_MJ_CREATE:
        status = NullTraceHandleCreate(DeviceObject, Irp);
        break;

    default:
        status = STATUS_NOT_IMPLEMENTED;
        break;
    }

    return status;
}
```

## 🔍 Security Requirements

### Code Security

- **Input Validation**: Validate all parameters and user input
- **Buffer Safety**: Use safe string functions (`RtlStringCch*`)
- **Memory Management**: Proper allocation/deallocation with pool tags
- **Error Handling**: Check all return values and handle errors gracefully
- **Privilege Checks**: Verify caller privileges for sensitive operations

### Security Review Process

1. **Automated Scanning**: Code passes static analysis tools
2. **Peer Review**: At least two security-focused reviewers
3. **Testing**: Comprehensive security testing including fuzzing
4. **Approval**: Security team sign-off required

### Prohibited Patterns

```c
// ❌ Bad: Unchecked buffer operations
strcpy(dest, source);  
gets(buffer);

// ✅ Good: Safe buffer operations
RtlStringCchCopyW(dest, destSize, source);
RtlStringCchGetsW(buffer, bufferSize);

// ❌ Bad: Unchecked memory allocation
ptr = ExAllocatePool(PagedPool, size);
*ptr = value;  // Potential crash

// ✅ Good: Checked allocation
ptr = ExAllocatePoolWithTag(PagedPool, size, POOL_TAG);
if (ptr != NULL) {
    *ptr = value;
}
```

## 🧪 Testing Requirements

### Unit Tests

- **Coverage**: Minimum 80% code coverage
- **Scenarios**: Test normal, edge, and error cases
- **Performance**: Validate performance requirements
- **Security**: Test security boundaries and attack vectors

### Integration Tests

- **Driver Loading**: Verify proper installation and startup
- **File Operations**: Test all intercepted file operations
- **Statistics**: Validate IOCTL interface functionality
- **Resource Management**: Test under memory pressure

### Test Environment

```cmd
# Run test suite
cd tests
test_evtx.exe

# Performance testing
perftest.exe --duration=300 --threads=16

# Security testing
fuzz_ioctl.exe --iterations=10000
```

## 📤 Submission Process

### 1. Branch Strategy

```bash
# Create feature branch
git checkout -b feature/description-of-change

# Or bug fix branch
git checkout -b bugfix/issue-number-description
```

### 2. Commit Guidelines

#### Commit Message Format
```
type(scope): brief description

Longer explanation of what changed and why.

Fixes: #issue-number
Signed-off-by: Your Name <your.email@example.com>
```

#### Commit Types
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `perf`: Performance improvements
- `test`: Testing changes
- `refactor`: Code refactoring
- `security`: Security improvements

#### Example Commit
```
fix(driver): prevent buffer overflow in ring buffer allocation

Added proper bounds checking when allocating ring buffer entries to
prevent potential buffer overflow when processing large file writes.
The allocation size is now validated against maximum buffer size
before proceeding with memory allocation.

Fixes: #123
Signed-off-by: John Doe <john.doe@example.com>
```

### 3. Pull Request Process

#### Pre-Submission Checklist

- [ ] Code builds without warnings
- [ ] All tests pass
- [ ] Security scan passes
- [ ] Documentation updated
- [ ] CLA signed and submitted
- [ ] Commit messages follow format
- [ ] No sensitive information in code/comments

#### Pull Request Template

```markdown
## Description
Brief description of changes

## Type of Change
- [ ] Bug fix
- [ ] New feature
- [ ] Performance improvement
- [ ] Documentation update
- [ ] Security enhancement

## Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Performance tests pass
- [ ] Security tests pass

## Security Impact
Describe any security implications

## Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Documentation updated
- [ ] CLA signed
```

### 4. Review Process

1. **Automated Checks**: CI/CD pipeline validates build and tests
2. **Code Review**: Maintainers review code quality and design
3. **Security Review**: Security team reviews for vulnerabilities
4. **Testing**: QA team validates functionality
5. **Approval**: Project maintainers approve merge
6. **Integration**: Code merged to main branch

## 🚫 Contribution Guidelines

### What We DON'T Accept

- Code without proper testing
- Changes that break existing functionality
- Security vulnerabilities or weaknesses
- Code that doesn't follow style guidelines
- Contributions without signed CLA
- Malicious or harmful code
- Copyright violations

### Review Criteria

**Code Quality (Required)**
- Follows coding standards
- Properly documented
- Handles errors gracefully
- Includes comprehensive tests

**Security (Critical)**
- No security vulnerabilities
- Proper input validation
- Secure memory management
- Appropriate privilege checks

**Performance (Important)**
- Maintains or improves performance
- Efficient algorithms and data structures
- Minimal resource usage
- Proper optimization

## 🆘 Getting Help

### Communication Channels

- **GitHub Issues**: Bug reports and feature requests
- **Security Issues**: security@lackadaisical-security.com
- **Development Questions**: dev@lackadaisical-security.com
- **Documentation**: https://github.com/lackadaisical-security/nulltrace/wiki

### Development Support

- **Code Reviews**: Request feedback on complex changes
- **Architecture Discussions**: Discuss major design decisions
- **Security Guidance**: Get security review early in development
- **Testing Assistance**: Help with test strategy and implementation

## 📜 Legal

### Intellectual Property

- All contributions become property of Lackadaisical Security
- Contributors retain attribution rights
- Commercial licensing terms apply to derivatives
- Open source license covers research and educational use

### Compliance

- Export control regulations compliance required
- No contributions from sanctioned entities
- Follow responsible disclosure for security issues
- Respect third-party intellectual property

## 🏆 Recognition

### Contributors

Outstanding contributors may be recognized through:
- GitHub contributor badge
- Credit in release notes
- Invitation to security conferences
- Potential employment opportunities

### Hall of Fame

Top contributors are featured on our website and in project documentation.

---

**Thank you for contributing to NullTrace!**

*By contributing, you agree to the terms outlined in this document and our Contributor License Agreement.*

---

© 2025 Lackadaisical Security - Professional Security Solutions 