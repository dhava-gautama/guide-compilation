# MuMu Player Developer Essentials

A comprehensive guide to commonly used ADB commands for developers working with MuMu Player.

## Table of Contents

- [ADB Setup](#adb-setup)
- [Device Connection](#device-connection)
  - [Finding MuMu Player Ports](#finding-mumu-player-ports)
  - [Multiple Devices](#multiple-devices)
- [Application Management](#application-management)
- [Device Information](#device-information)
- [Input Simulation](#input-simulation)
- [File Operations](#file-operations)
- [Screen Capture](#screen-capture)
- [Package Capture Setup](#package-capture-setup)

---

## ADB Setup

### Built-in ADB Location

MuMu Player includes a built-in ADB in its installation directory:

```
C:\Program Files\Netease\MuMuPlayerGlobal-12.0\nx_main\adb.exe
```

### Download ADB

If you prefer to use a standalone ADB installation:

1. **Recommended**: [ADBShell Downloads](https://adbshell.com/downloads)
2. **Official**: [Android Platform Tools](https://developer.android.com/studio/releases/platform-tools)

---

## Device Connection

### Connect to MuMu Player

1. Open Command Prompt (if using built-in ADB, navigate to the installation directory):
   ```cmd
   cd C:\Program Files\Netease\MuMuPlayerGlobal-12.0\nx_main
   ```

2. Kill any existing ADB server:
   ```bash
   adb kill-server
   # Built-in ADB: adb_server.exe kill-server
   ```

3. Connect to the emulator (default port 7555):
   ```bash
   adb connect 127.0.0.1:7555
   # Built-in ADB: adb_server.exe connect 127.0.0.1:7555
   ```

4. Verify connection:
   ```bash
   adb devices
   # Built-in ADB: adb_server.exe devices
   ```

   Expected output: `127.0.0.1:7555 device`

**Note**: If the device doesn't appear, repeat the kill-server and connect steps. The emulator may be using a different port.

### Finding MuMu Player Ports

If you're running multiple MuMu Player instances or need to find the correct port, use these commands:

**PowerShell:**
```powershell
Get-NetTCPConnection -State Listen | Where-Object {$_.OwningProcess -in (Get-Process -Name "*MuMu*").Id} | Select-Object LocalAddress, LocalPort, @{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}} | Format-Table -AutoSize
```

**PowerShell (simplified - MuMu ADB ports only):**
```powershell
Get-NetTCPConnection -State Listen | Where-Object {(Get-Process -Id $_.OwningProcess -ErrorAction SilentlyContinue).ProcessName -like "*MuMuVMM*" -and $_.LocalPort -match "^(555\d|755\d)$"} | Select-Object LocalAddress, LocalPort | Format-Table -AutoSize
```

**CMD (using netstat):**
```cmd
netstat -ano | findstr "LISTENING" | findstr "555"
```

**Common MuMu Player ports:**
- First instance: `127.0.0.1:7555` or `127.0.0.1:5555`
- Second instance: `127.0.0.1:5557`
- Third instance: `127.0.0.1:5559`
- Fourth instance: `127.0.0.1:5561`

The pattern typically increments by 2 for each additional instance (5555, 5557, 5559, 5561...).

### Multiple Devices

To target a specific device when multiple are connected:

```bash
adb -s 127.0.0.1:7555 <command>

# Example:
adb -s 127.0.0.1:7555 shell pm list packages -3
```

---

## Application Management

### Install and Uninstall APK

**Install an APK:**
```bash
adb install C:\path\to\app.apk
```

**Uninstall an APK:**
```bash
adb uninstall com.package.name
```

### List Package Names

**All packages:**
```bash
adb shell pm list packages
```

**Third-party packages only:**
```bash
adb shell pm list packages -3
```

**System packages only:**
```bash
adb shell pm list packages -s
```

**Currently focused application:**
```bash
adb shell dumpsys window | findstr mCurrentFocus
```

### Multi-Instance Applications

**For emulator versions earlier than 2.2.2 (x86/x64):**
- Multi-instance package names follow the format: `original.package.name + suffix`
- Example: Honkai Impact 3rd instances will have different package names

**For emulator versions 2.2.2 (x86/x64) and newer:**
- Multi-instance applications share the same package name
- Use `UserId` to control specific instances
- Don't forget to connect first: `adb connect 127.0.0.1:7555`

### Get Activity Class Name

Start monitoring activity launches:
```bash
adb logcat ActivityManager:I *:s | findstr "cmp"
```

Then launch the target application. Example output for "Identity V":
```
cmp=com.netease.dwrg/.Launcher
```

This means:
- Package name: `com.netease.dwrg`
- Activity class name: `.Launcher`
- Full activity name: `com.netease.dwrg.Launcher`

### Start Application

**Basic start:**
```bash
adb shell am start -n <package>/<activity>

# Example (Identity V):
adb shell am start -n com.netease.dwrg/.Launcher
```

**Start with timing information:**
```bash
adb shell am start -W <package>/<activity>
```

This displays startup time metrics.

### Stop Application

Force stop an application:
```bash
adb shell am force-stop <package>

# Example (Identity V):
adb shell am force-stop com.netease.dwrg
```

### View Application Version

```bash
adb shell dumpsys package <package> | findstr version
```

### Clear Application Data

```bash
adb shell pm clear <package>
```

---

## Device Information

### Model Information

**Device model:**
```bash
adb shell getprop ro.product.model
```

**Brand:**
```bash
adb shell getprop ro.product.brand
```

**Processor model:**
```bash
adb shell getprop ro.product.board
```

**Android version:**
```bash
adb shell getprop ro.build.version.release
```

### Rendering Information

**Engine rendering mode:**
```bash
adb shell dumpsys SurfaceFlinger | findstr "GLES"
```

**Note**: This command is not available in MuMu Player version 2.0.30 and above. Use older versions if this information is needed.

---

## Input Simulation

### Key Input

Simulate key presses:
```bash
adb shell input keyevent <keycode>

# Example - Press HOME key:
adb shell input keyevent 3
```

Search online for key code values of other keys.

### Text Input

Input text strings:
```bash
adb shell input text <string>

# Example:
adb shell input text test
```

**Note**: Chinese characters are not supported.

### Touch Input

**Tap at coordinates:**
```bash
adb shell input tap <X> <Y>
```

Where X and Y are the screen coordinates.

**Swipe gesture:**
```bash
adb shell input swipe <X1> <Y1> <X2> <Y2>
```

Where (X1, Y1) is the start point and (X2, Y2) is the end point.

---

## File Operations

### Upload Files to Emulator

```bash
adb push <source_path> <destination_path>

# Example:
adb push C:\test.apk /data
```

### Download Files from Emulator

```bash
adb pull <source_path> <destination_path>

# Example:
adb pull /data/test.apk C:\
```

---

## Screen Capture

### Screenshot

**Capture screenshot:**
```bash
adb shell screencap /data/screen.png
```

**Save to computer:**
```bash
adb pull /data/screen.png C:\
```

### Screen Recording

**Start recording:**
```bash
adb shell screenrecord /data/test.mp4
```

**Stop recording:**
Press `CTRL+C`

**Export video:**
```bash
adb pull /data/test.mp4 C:\
```

---

## Package Capture Setup

Follow these steps to capture network traffic from MuMu Player using Fiddler:

### Prerequisites

- Latest version of Fiddler
- Latest version of MuMu Player

### Configuration Steps

1. **Configure Fiddler:**
   - Open Fiddler
   - Go to `Tools → Options → Connections`
   - Check "Allow remote computers to connect"
   - Restart Fiddler (Important!)

2. **Check Host IP Address:**
   - Open Command Prompt
   - Run `ipconfig /all`
   - Note your computer's IP address (if you have virtual network adapters, ensure you use the correct physical adapter IP)

3. **Configure MuMu Player Proxy:**
   - Start MuMu Player
   - Long press the WiFi name
   - Click "Modify network"
   - Configure proxy settings with your computer's IP and Fiddler's port (default: 8888)

4. **Save and Test:**
   - Save the proxy configuration
   - Launch applications in MuMu Player
   - View captured traffic in Fiddler

---

## Additional Resources

For more ADB commands and documentation, visit:
- [ADBShell Commands Reference](https://adbshell.com/commands)
- [Android Debug Bridge (ADB) Documentation](https://developer.android.com/studio/command-line/adb)

---

**Note**: This guide is specifically tailored for MuMu Player. Command syntax and functionality may vary with different Android emulators or devices.
