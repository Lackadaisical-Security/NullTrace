# NullTrace Quick Start Guide

<div align="center">

**Get NullTrace Running in Under 10 Minutes**

*© 2025 [Lackadaisical Security](https://lackadaisical-security.com)*

</div>

## ⚡ 5-Minute Setup

### Prerequisites Check

Ensure you have:
- [ ] Windows 10/11 x64
- [ ] Administrator privileges  
- [ ] Windows Driver Kit (WDK) installed
- [ ] Visual Studio 2019/2022

### Quick Installation

```cmd
# 1. Enable test signing (REQUIRED)
bcdedit /set testsigning on

# 2. Reboot your system
shutdown /r /t 0

# 3. Clone and build
git clone https://github.com/lackadaisical-security/nulltrace.git
cd nulltrace\scripts
make.bat

# 4. Install driver
install.bat

# 5. Verify installation
sc query NullTrace
```

**That's it!** NullTrace is now intercepting `.log` and `.evtx` file writes.

## 🧪 Quick Test

Run the test application to verify everything works:

```cmd
cd tests
test_evtx.exe
```

**Expected Output:**
```
[SUCCESS] All tests PASSED! NullTrace is working correctly.
```

## 🎯 What NullTrace Does

Once installed, NullTrace automatically:

- ✅ **Intercepts writes** to files ending in `.log` or `.evtx`
- ✅ **Returns success** to applications (they think the write succeeded)
- ✅ **Silently drops** the actual data (anti-forensics)
- ✅ **Captures statistics** about blocked operations
- ✅ **Maintains performance** for non-target files

## 📊 Monitor Activity

Check driver statistics:

```cmd
# View current stats via test app
test_evtx.exe

# Or query service status
sc query NullTrace
```

## 🔧 Common Issues

### "Driver failed to start"
```cmd
# Check test signing is enabled
bcdedit /enum | findstr testsigning

# Should show: testsigning Yes
```

### "Access denied" 
```cmd
# Run as Administrator
# Right-click Command Prompt -> "Run as administrator"
```

### "File not found"
```cmd
# Ensure WDK is installed
# Verify Visual Studio has C++ workload
```

## 🛑 Uninstall

To remove NullTrace:

```cmd
# Stop and remove service
sc stop NullTrace
sc delete NullTrace

# Remove driver package  
pnputil /delete-driver NullTrace.inf /uninstall /force

# Disable test signing (optional)
bcdedit /set testsigning off
```

## 🔍 Troubleshooting

### Build Errors
- **Solution**: Install Windows Driver Kit (WDK) 10.0.22621+
- **Download**: https://docs.microsoft.com/en-us/windows-hardware/drivers/download-the-wdk

### Driver Won't Load
- **Check**: Test signing enabled (`bcdedit /enum`)
- **Check**: Running as Administrator
- **Check**: Windows 10/11 x64 system

### Service Fails to Start
- **Check**: Windows Event Viewer (System log)
- **Common**: Secure Boot conflicts (disable in BIOS)
- **Solution**: Use production-signed driver for Secure Boot

## 📚 Next Steps

### Learn More
- **Full Documentation**: [README.md](README.md)
- **Advanced Usage**: [Documentation Wiki](https://github.com/lackadaisical-security/nulltrace/wiki)
- **Contributing**: [CONTRIBUTING.md](CONTRIBUTING.md)

### Commercial Use
- **Licensing**: Contact licensing@lackadaisical-security.com
- **Support**: https://lackadaisical-security.com/support

### Security Research
- **Best Practices**: Use only in authorized environments
- **Legal**: Read [DISCLAIMER.md](DISCLAIMER.md) carefully
- **Ethics**: Follow responsible disclosure practices

## 🎓 Understanding the Code

### Key Files
- **`Driver/NullTrace.c`**: Main driver implementation
- **`Driver/NullTrace.h`**: Headers and data structures  
- **`tests/test_evtx.cpp`**: Validation test suite
- **`scripts/make.bat`**: Build automation

### Architecture Overview
```
User Application
       ↓ (write to .evtx/.log)
NullTrace Filter Driver
       ↓ (intercepts & drops)
Status: SUCCESS ✅
Actual Data: 🗑️ (discarded)
```

## 🔒 Security Notes

**FOR AUTHORIZED TESTING ONLY**

- ⚠️ Use only in test environments
- ⚠️ Obtain written authorization
- ⚠️ Follow local laws and regulations
- ⚠️ Never use on production systems

## 🤝 Support

### Community
- **GitHub Issues**: Bug reports and questions
- **Discussions**: Feature requests and ideas

### Commercial
- **Email**: support@lackadaisical-security.com
- **Website**: https://lackadaisical-security.com

---

<div align="center">

**Need Help?** 

Create an issue on GitHub or contact our support team.

**[Lackadaisical Security](https://lackadaisical-security.com)** - *Professional Security Solutions*

</div> 