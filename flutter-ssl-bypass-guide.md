# Flutter SSL Bypass: How to Intercept HTTPS Traffic When All Other Frida Scripts Fail

A comprehensive guide to bypassing SSL pinning in Flutter applications when traditional Frida scripts don't work.

**Author:** Adham A. Makroum
**Original Publication:** May 17, 2025

---

## Table of Contents

- [Introduction](#introduction)
- [The Problem](#the-problem)
- [Understanding the Challenge](#understanding-the-challenge)
- [Solution Overview](#solution-overview)
- [Prerequisites](#prerequisites)
- [Step-by-Step Guide](#step-by-step-guide)
  - [Step 1: Extract libflutter.so](#step-1-extract-libflutterso)
  - [Step 2: Analyze with Ghidra](#step-2-analyze-with-ghidra)
  - [Step 3: Find the Target Function](#step-3-find-the-target-function)
  - [Step 4: Calculate the Offset](#step-4-calculate-the-offset)
  - [Step 5: Write the Frida Script](#step-5-write-the-frida-script)
  - [Step 6: Execute the Script](#step-6-execute-the-script)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

---

## Introduction

This guide walks you through intercepting HTTPS traffic from a Flutter-based APK during a penetration testing engagement. After 2 days of research and trying several Frida scripts that didn't work, this method successfully bypassed Flutter's SSL pinning.

---

## The Problem

### Why Traditional Methods Fail

Flutter apps are written in Dart, and Dart does **not use the system CA store**. This means traditional certificate pinning bypass techniques often don't work.

When attempting to capture HTTPS traffic using Burp Suite initially, no requests came through - the app's SSL verification was blocking the proxy.

### Common Scripts That May Fail

During research, several popular scripts were tested but failed:

- [NVISO Flutter TLS bypass](https://github.com/NVISOsecurity/disable-flutter-tls-verification)
- [Frida FlutterProxy](https://github.com/httptoolkit/frida-android-unpinning)

These scripts worked for some users but not in all cases, suggesting the issue might be related to:
- Specific APK implementation
- Device architecture differences
- Flutter version variations

---

## Understanding the Challenge

### The Architecture Issue

The key problem identified was **architecture mismatch**:

- **Friend's setup:** Running AVD on macOS with **ARM-based emulator**
- **Your setup:** Running AVD on PC with **x86_64 architecture**

This architectural difference led to different memory layouts and offsets in the binary. As a result, scripts that worked for ARM couldn't locate the correct patterns in x86_64.

> **Key Insight:** The scripts successfully matched memory patterns on ARM, but failed to do so on x86_64 because the bytecode and structure were different.

---

## Solution Overview

The solution involves manually:

1. Extracting the `libflutter.so` library from the APK
2. Reverse engineering it with Ghidra
3. Finding the SSL certificate verification function
4. Calculating the memory offset
5. Writing a custom Frida script to hook and bypass the function

---

## Prerequisites

### Tools Required

- **APKTool** - For decompiling APK files
- **Ghidra** - Free reverse engineering tool
- **Frida** - Dynamic instrumentation toolkit
- **Burp Suite** - Web proxy for intercepting traffic
- **Android Device/Emulator** - For testing

### Environment Setup

1. **Configure Burp Suite:**
   - Go to `Tools → Options → Connections`
   - Check "Allow remote computers to connect"
   - Restart Burp Suite (Important!)

2. **Configure Burp Proxy Listener:**
   - Bind to port: `8083`
   - Bind to address: `All interfaces`
   - Enable "Support invisible proxying"

3. **Set up iptables to redirect traffic to Burp:**
   ```bash
   iptables -t nat -A OUTPUT -p tcp -j DNAT --to-destination Burp_IP:Burp_Port
   ```

---

## Step-by-Step Guide

### Step 1: Extract libflutter.so

First, extract the Flutter library from your APK:

```bash
# Decompile the APK using apktool
apktool d app.apk

# Navigate to the lib directory
cd app/lib

# List available architectures
ls
# Output: arm64-v8a  armeabi-v7a  x86  x86_64

# Navigate to your architecture (x86_64 in this example)
cd x86_64

# List the libraries
ls
# Output: libflutter.so  libimage_processing_util_jni.so  libsentry-android.so  libsentry.so
```

**Important:** Choose the architecture that matches your emulator. In this case, **x86_64** was selected because the emulator was x86_64 architecture.

### Step 2: Analyze with Ghidra

1. **Import the library:**
   - Open Ghidra
   - File → Import → Select `libflutter.so`

2. **Analyze the binary:**
   - Double-click to analyze
   - Ghidra will automatically identify functions and structures

3. **Understand the SSL implementation:**

   Flutter uses **BoringSSL** to handle everything related to SSL.

### Step 3: Find the Target Function

#### Search for ssl_client

1. Go to **Search → For Strings**

2. Configure search parameters:
   - ☑ **Require Null Termination**
   - Minimum Length: `5`
   - Alignment: `1`
   - Word Model: `StringModel.sng`
   - Memory Block Types: **☑ Loaded Blocks**
   - Selection Scope: **☑ Search All**

3. Search for: `ssl_client`

4. **Results:** Look for `ssl_client` in the results, then double-click to explore its XREFs (cross-references)

#### Identify the Correct Function

You'll find 2 XREFs. Check each referenced function by double-clicking.

The correct function signature:
- **Takes 3 arguments**
- **Returns a boolean** (`true` = success, `false` = failed)

This function is: **`ssl_crypto_x509_session_verify_cert_chain`**

In Ghidra, search for the string `"ssl_client"` which appears around line 230 in the source file `ssl_x509.cc`.

### Step 4: Calculate the Offset

#### Get the Function Offset

Once you locate the function in Ghidra:

1. **Double-click on the function name** to view its decompiled code
2. **Note the offset** displayed in Ghidra's address column

**Example:** `Offset: 02184644`

#### Calculate the Relative Offset

Subtract the base load address (usually `100000`) to get the relative offset used in Frida:

```
02184644 - 100000 = 2084644
```

This is the address we'll use in the Frida script.

**Screenshot reference:** The offset shows as `02184644` in Ghidra's listing view.

---

## Step 5: Write the Frida Script

Here's a simple script to hook and patch the return value of the `ssl_crypto_x509_session_verify_cert_chain` function:

```javascript
function hookVerifyCert() {
    var base = Process.findModuleByName("libflutter.so").base;
    var offset = 0x2084644; // Change this value with your offset
    var addr = base.add(offset);

    console.log("[*] Hooking function at: " + addr);

    Interceptor.attach(addr, {
        onLeave: function (retval) {
            console.log("[*] Original retval: " + retval);
            retval.replace(0x1); // Force 'true'
            console.log("[*] SSL certificate validation bypassed");
        }
    });
}

setTimeout(hookVerifyCert, 1000);
```

**Important Notes:**

- Replace `0x2084644` with **your calculated offset**
- The script waits 1000ms before hooking to ensure the module is loaded
- `retval.replace(0x1)` forces the function to return `true`, bypassing validation

### Architecture-Specific Considerations

> ⚠️ **This script has been tested on an AVD with Android 11 based on x86_64, so if you encounter any errors, just ask chatgpt to edit the script to suit your environment.**

If running on different architectures:
- **ARM/ARM64:** You may need to adjust the offset
- **x86:** Similar approach but verify offsets match

---

## Step 6: Execute the Script

### Run Frida

```bash
cd ~/flutterSSLBypass/app
frida -U -f com.example.app -l script.js
```

**Expected output:**

```
Frida 16.6.6 - A world-class dynamic instrumentation toolkit
Commands:
    help      -> Displays the help system
    object?   -> Display information about 'object'
    exit/quit -> Exit

More info at https://frida.re/docs/home/

Connected to Pixel 5 (id=127.0.0.1:6555)
Spawned com.example.app - Resuming main thread!

[Pixel 5::com.caraleva.caraleva ]-> [*] Hooking function at: 0x7aeedc612644
[*] Original retval: 0x0
[*] SSL certificate validation bypassed
[*] Original retval: 0x0
[*] SSL certificate validation bypassed
[*] Original retval: 0x0
[*] SSL certificate validation bypassed
[Pixel 5::com.carateya.caraleya ]->
```

### Verify in Burp Suite

After running the script, **Burp Suite should finally be able to intercept all HTTPS traffic** from the app.

The SSL pinning was completely bypassed!

---

## Troubleshooting

### Issue: libflutter.so Not Showing in Ghidra

**Problem:** After decompiling and importing the libflutter.so file to Ghidra, it's not showing any symbols or is empty.

**Solution:** You likely didn't let Ghidra analyze the file. When you import a file:
1. Double-click the imported file
2. Allow Ghidra to analyze it (this may take a few minutes)
3. Once analysis completes, all functions and strings will be visible

### Issue: Offset Doesn't Match

**Problem:** The offset calculated doesn't work when running the Frida script.

**Potential causes:**
1. **Architecture mismatch:** Ensure you extracted `libflutter.so` from the correct architecture folder (arm64-v8a, x86_64, etc.)
2. **Base address incorrect:** The base load address might not be `100000`. Check in Ghidra under "Program Tree"
3. **Different Flutter version:** Different versions may have different offsets

**Solution:** Double-check the architecture matches your emulator/device.

### Issue: Script Runs But Traffic Still Blocked

**Possible causes:**
1. **Routing issue:** Verify iptables rules are correctly redirecting traffic to Burp
2. **Multiple instances:** The app might check certificates in multiple places
3. **Certificate store validation:** Some apps do additional validation beyond BoringSSL

**Solution:** Monitor Frida output to confirm the hook is being triggered. If not triggered, the function might be at a different offset.

### Issue: App Crashes When Script Runs

**Problem:** The app crashes immediately when Frida attaches.

**Causes:**
- Incorrect offset overwrites critical memory
- Anti-tampering detection in the app

**Solution:**
- Verify your offset calculation
- Try using `-f` flag to spawn the app fresh with Frida attached
- Check if the app has root/debugging detection

---

## Additional Resources

### Related Frida Scripts

- [flutter_SSL_Bypass](https://github.com/PiRogueToolSuite/deb-frida) - GitHub repository with automated scripts
- [NVISO Flutter TLS bypass](https://github.com/NVISOsecurity/disable-flutter-tls-verification) - Alternative approach
- [Frida FlutterProxy](https://github.com/httptoolkit/frida-android-unpinning) - HTTP Toolkit's Flutter unpinning

### Tools & Documentation

- [Ghidra](https://ghidra-sre.org/) - NSA's reverse engineering tool
- [Frida](https://frida.re/) - Dynamic instrumentation toolkit
- [APKTool](https://ibotpeaches.github.io/Apktool/) - Tool for reverse engineering APKs
- [Burp Suite](https://portswigger.net/burp) - Web security testing tool

### Further Reading

- **BoringSSL Documentation:** [https://boringssl.googlesource.com/boringssl/](https://boringssl.googlesource.com/boringssl/)
- **Flutter Security:** Understanding Flutter's approach to SSL/TLS
- **Android SSL Pinning:** General concepts applicable to Flutter
