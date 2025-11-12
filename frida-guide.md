# Frida Guide: Complete Installation and Usage Tutorial

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installing Frida](#installing-frida)
- [Checking Android Device Architecture](#checking-android-device-architecture)
- [Installing Frida Server on Android](#installing-frida-server-on-android)
- [Running Frida Scripts](#running-frida-scripts)
- [Common Use Cases](#common-use-cases)
- [Advanced Techniques](#advanced-techniques)
- [Troubleshooting](#troubleshooting)
- [Resources](#resources)

---

## Introduction

Frida is a dynamic instrumentation toolkit that allows you to inject JavaScript or Python scripts into running processes on various platforms including Android, iOS, Windows, macOS, and Linux. It's widely used for:

- Reverse engineering applications
- Security research and penetration testing
- Debugging and analysis
- Bypassing certificate pinning
- Hooking functions and modifying behavior
- API monitoring and tracing

---

## Prerequisites

Before installing Frida, ensure you have:

- Python 3.7 or higher installed
- pip (Python package manager)
- ADB (Android Debug Bridge) for Android devices
- USB debugging enabled on your Android device
- Root access on Android device (recommended but not always required)

---

## Installing Frida

### 1. Install Frida Tools on Your Computer

The easiest way to install Frida is via pip:

```bash
pip install frida-tools
```

Or for the latest development version:

```bash
pip install frida-tools --upgrade
```

Verify the installation:

```bash
frida --version
```

### 2. Install Frida Python Bindings

```bash
pip install frida
```

---

## Checking Android Device Architecture

Before downloading the Frida server for your Android device, you need to determine the correct architecture.

### Method 1: Using ADB

Connect your device via USB and run:

```bash
adb shell getprop ro.product.cpu.abi
```

**Common outputs:**
- `arm64-v8a` - 64-bit ARM (most modern devices)
- `armeabi-v7a` - 32-bit ARM
- `x86` - 32-bit x86 (emulators)
- `x86_64` - 64-bit x86 (some emulators)

### Method 2: Using Multiple Properties

For more detailed information:

```bash
adb shell getprop | grep abi
```

This will show:

```
[ro.product.cpu.abi]: [arm64-v8a]
[ro.product.cpu.abilist]: [arm64-v8a,armeabi-v7a,armeabi]
[ro.product.cpu.abilist32]: [armeabi-v7a,armeabi]
[ro.product.cpu.abilist64]: [arm64-v8a]
```

### Method 3: Using uname

```bash
adb shell uname -m
```

**Outputs:**
- `aarch64` - 64-bit ARM (use arm64)
- `armv7l` or `armv8l` - 32-bit ARM (use arm)
- `i686` - 32-bit x86
- `x86_64` - 64-bit x86

---

## Installing Frida Server on Android

### 1. Download the Correct Frida Server

Go to [Frida Releases](https://github.com/frida/frida/releases) and download the appropriate version.

**Architecture mapping:**
- `arm64-v8a` → `frida-server-<version>-android-arm64`
- `armeabi-v7a` → `frida-server-<version>-android-arm`
- `x86` → `frida-server-<version>-android-x86`
- `x86_64` → `frida-server-<version>-android-x86_64`

Or download via command line:

```bash
# Check your Frida version
FRIDA_VERSION=$(frida --version)

# For arm64 devices (most common)
wget https://github.com/frida/frida/releases/download/${FRIDA_VERSION}/frida-server-${FRIDA_VERSION}-android-arm64.xz

# Extract
unxz frida-server-${FRIDA_VERSION}-android-arm64.xz

# Rename for convenience
mv frida-server-${FRIDA_VERSION}-android-arm64 frida-server
```

### 2. Push Frida Server to Device

```bash
adb push frida-server /data/local/tmp/
```

### 3. Set Permissions and Run

```bash
# Make it executable
adb shell "chmod 755 /data/local/tmp/frida-server"

# Run as root (device must be rooted)
adb shell "su -c /data/local/tmp/frida-server &"
```

Or without root (limited functionality):

```bash
adb shell "/data/local/tmp/frida-server &"
```

### 4. Verify Connection

On your computer:

```bash
frida-ps -U
```

This should list all processes running on the connected device.

### Alternative: Using Frida Server as a Daemon

For persistent operation:

```bash
adb shell "su -c 'nohup /data/local/tmp/frida-server > /dev/null 2>&1 &'"
```

---

## Running Frida Scripts

### Basic Command Structure

```bash
frida [options] -U -f <package-name> -l <script.js>
```

**Common options:**
- `-U` - Connect to USB device
- `-f` - Spawn the app (start fresh)
- `-l` - Load script file
- `-n` - Attach to process by name
- `-p` - Attach to process by PID
- `--no-pause` - Automatically resume the app

### Method 1: Interactive REPL

Attach to a running process interactively:

```bash
frida -U -n com.example.app
```

Then enter JavaScript commands directly.

### Method 2: Running a Script File

Create a JavaScript file (e.g., `hook.js`):

```javascript
Java.perform(function() {
    console.log("[*] Script loaded successfully");

    // Hook a specific class method
    var MainActivity = Java.use("com.example.app.MainActivity");

    MainActivity.onCreate.implementation = function(savedInstanceState) {
        console.log("[*] MainActivity.onCreate() called");
        this.onCreate(savedInstanceState);
    };
});
```

Run the script:

```bash
# Spawn the app and inject script
frida -U -f com.example.app -l hook.js --no-pause

# Attach to already running app
frida -U com.example.app -l hook.js
```

### Method 3: Python Script

Create a Python file (e.g., `frida_script.py`):

```python
import frida
import sys

def on_message(message, data):
    if message['type'] == 'send':
        print("[*] {0}".format(message['payload']))
    else:
        print(message)

# Connect to device
device = frida.get_usb_device()

# Spawn or attach to process
pid = device.spawn(["com.example.app"])
session = device.attach(pid)

# Load JavaScript
script = session.create_script("""
    Java.perform(function() {
        console.log("[*] Frida injected");

        var String = Java.use("java.lang.String");
        console.log("[*] Hooked: " + String.toString());
    });
""")

script.on('message', on_message)
script.load()
device.resume(pid)

# Keep script running
sys.stdin.read()
```

Run it:

```bash
python frida_script.py
```

---

## Common Use Cases

### 1. Hooking a Method

```javascript
Java.perform(function() {
    var targetClass = Java.use("com.example.app.TargetClass");

    targetClass.methodName.implementation = function(arg1, arg2) {
        console.log("[*] Method called with args: " + arg1 + ", " + arg2);

        // Call original method
        var result = this.methodName(arg1, arg2);

        console.log("[*] Return value: " + result);
        return result;
    };
});
```

### 2. Bypassing SSL Pinning

```javascript
Java.perform(function() {
    // Hook OkHttp3
    var CertificatePinner = Java.use("okhttp3.CertificatePinner");
    CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function() {
        console.log("[*] SSL pinning bypassed for: " + arguments[0]);
    };

    // Hook TrustManager
    var X509TrustManager = Java.use("javax.net.ssl.X509TrustManager");
    var SSLContext = Java.use("javax.net.ssl.SSLContext");

    var TrustManager = Java.registerClass({
        name: 'com.example.TrustManager',
        implements: [X509TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
});
```

### 3. Tracing All Methods in a Class

```javascript
Java.perform(function() {
    var targetClass = Java.use("com.example.app.TargetClass");
    var methods = targetClass.class.getDeclaredMethods();

    methods.forEach(function(method) {
        var methodName = method.getName();
        console.log("[*] Hooking: " + methodName);

        targetClass[methodName].implementation = function() {
            console.log("[*] Called: " + methodName);
            return this[methodName].apply(this, arguments);
        };
    });
});
```

### 4. Dumping Method Arguments

```javascript
Java.perform(function() {
    var LoginActivity = Java.use("com.example.app.LoginActivity");

    LoginActivity.login.implementation = function(username, password) {
        console.log("[*] Login attempt:");
        console.log("    Username: " + username);
        console.log("    Password: " + password);

        return this.login(username, password);
    };
});
```

### 5. Changing Return Values

```javascript
Java.perform(function() {
    var LicenseChecker = Java.use("com.example.app.LicenseChecker");

    LicenseChecker.isPremium.implementation = function() {
        console.log("[*] isPremium() bypassed - returning true");
        return true;
    };
});
```

### 6. Intercepting Native Functions

```javascript
Interceptor.attach(Module.findExportByName("libnative.so", "native_function"), {
    onEnter: function(args) {
        console.log("[*] native_function called");
        console.log("    arg0: " + args[0]);
        console.log("    arg1: " + args[1]);
    },
    onLeave: function(retval) {
        console.log("[*] Return value: " + retval);
    }
});
```

---

## Advanced Techniques

### 1. Finding and Hooking Dynamically Loaded Classes

```javascript
Java.perform(function() {
    Java.enumerateLoadedClasses({
        onMatch: function(className) {
            if (className.includes("example")) {
                console.log("[*] Found class: " + className);
            }
        },
        onComplete: function() {
            console.log("[*] Class enumeration complete");
        }
    });
});
```

### 2. Memory Scanning

```javascript
var pattern = "48 8B 05 ?? ?? ?? ??";
var matches = Memory.scanSync(ptr("0x400000"), 0x1000, pattern);

matches.forEach(function(match) {
    console.log("[*] Pattern found at: " + match.address);
});
```

### 3. Calling Methods Programmatically

```javascript
Java.perform(function() {
    Java.choose("com.example.app.MainActivity", {
        onMatch: function(instance) {
            console.log("[*] Found instance: " + instance);

            // Call method on the instance
            instance.someMethod("test", 123);
        },
        onComplete: function() {}
    });
});
```

### 4. Monitoring File Operations

```javascript
var open = new NativeFunction(
    Module.findExportByName(null, "open"),
    'int', ['pointer', 'int']
);

Interceptor.replace(open, new NativeCallback(function(pathname, flags) {
    var path = Memory.readUtf8String(pathname);
    console.log("[*] Opening file: " + path);
    return open(pathname, flags);
}, 'int', ['pointer', 'int']));
```

---

## Troubleshooting

### Problem: "Failed to spawn: unable to find process with name"

**Solution:** Ensure the package name is correct:

```bash
adb shell pm list packages | grep <app-name>
```

### Problem: "Failed to attach: unable to access process"

**Solution:**
- Ensure Frida server is running: `adb shell ps | grep frida`
- Check if you need root access
- Restart Frida server

### Problem: Architecture Mismatch

**Solution:** Verify you downloaded the correct Frida server version for your device architecture.

### Problem: "Error: unable to find module"

**Solution:** The library might not be loaded yet. Use `Java.perform()` or wait for the appropriate timing.

### Problem: Version Mismatch

Ensure Frida tools and server versions match:

```bash
# Check tool version
frida --version

# Check server version on device
adb shell "/data/local/tmp/frida-server --version"
```

If they don't match, download the matching server version.

---

## Resources

### Official Documentation
- [Frida Official Website](https://frida.re/)
- [Frida JavaScript API](https://frida.re/docs/javascript-api/)
- [Frida GitHub Repository](https://github.com/frida/frida)

### Useful Tools
- **Objection** - Mobile security framework powered by Frida
  ```bash
  pip install objection
  objection -g com.example.app explore
  ```

- **Frida-DEXDump** - Dump DEX files from memory
- **Fridump** - Dump memory from running processes
- **r2frida** - Radare2 integration with Frida

### Learning Resources
- [Frida CodeShare](https://codeshare.frida.re/) - Community scripts
- [Frida Handbook](https://learnfrida.info/)
- Android reverse engineering tutorials using Frida

### Tips for Success
1. Always match Frida client and server versions
2. Start with simple hooks before complex modifications
3. Use `console.log()` extensively for debugging
4. Read the app's source code (if available) to understand structure
5. Join the Frida community on GitHub for support

---

## Quick Reference Commands

```bash
# List all processes on device
frida-ps -U

# List all apps
frida-ps -Uai

# Attach to process by name
frida -U -n com.example.app

# Spawn app with script
frida -U -f com.example.app -l script.js --no-pause

# Trace method calls
frida-trace -U -f com.example.app -j '*!*login*/isu'

# Kill Frida server
adb shell "su -c 'pkill frida-server'"

# Port forwarding (if needed)
adb forward tcp:27042 tcp:27042
```

---

**Happy Hooking!** 🎣
