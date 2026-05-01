# LAB-1-SECURITY: Mobexler + Android Emulator Lab Setup

## Overview

This lab establishes a complete **mobile security testing environment** using Mobexler (a pre-configured penetration testing VM) and Genymotion Android emulator. The setup enables dynamic analysis, network interception, and security testing of Android applications in an isolated, reproducible lab environment.

**Lab Goal:** Set up a fully functional Mobexler VM with proper network isolation and Android device connectivity for mobile security testing and analysis.

---

## Table of Contents

1. [Architecture](#architecture)
2. [Prerequisites](#prerequisites)
3. [Glossary](#glossary)
4. [Setup Instructions](#setup-instructions)
5. [Verification Steps](#verification-steps)
6. [Troubleshooting](#troubleshooting)
7. [Documentation & Screenshots](#documentation--screenshots)

---

## Architecture

### Network Topology

```
Host Computer (Windows 11)
│
├─ Internet (WAN)
│  └─> Mobexler VM (via NAT - Adapter 1)
│      └─> Can reach external resources, updates, tools
│
├─ Lab Network (Host-Only - Private, 192.168.X.X)
│  └─> Mobexler VM (Adapter 2)
│      └─> Genymotion Android Emulator (connected via ADB/TCP-IP)
│          └─> Isolated testing environment
│
└─ Hypervisor: VMware Workstation 17.x
   └─> VirtualBox (for Genymotion support)
```

### Key Components

| Component | Role | Purpose |
|-----------|------|---------|
| **Mobexler VM** | Security testing toolkit | Pre-loaded with Frida, Burp Suite, ADB, Apktool, etc. |
| **NAT Adapter (ens33)** | Internet access | Updates, tool downloads, external connectivity |
| **Host-Only Adapter (ens34)** | Lab network | Isolated private network (192.168.40.0/24) |
| **Genymotion Emulator** | Android target | Test device running on host, connected via ADB |
| **ADB (Android Debug Bridge)** | Communication | Connects Mobexler to Android device over TCP/IP |

---

## Prerequisites

### System Requirements

- **OS:** Windows 11 Professional / Enterprise
- **RAM:** 32 GB (minimum 8 GB, 4 GB for each VM)
- **Disk Space:** 157 GB free (after cleanup)
- **CPU:** Intel VT-x or AMD-V virtualization enabled
- **Network:** Host connectivity for downloads

### Software Required

- **VMware Workstation 17.x** (or later)
- **VirtualBox 7.1.x** (for Genymotion)
- **VirtualBox Extension Pack 7.1.x**
- **Genymotion For Fun (free version)**
- **Android SDK/Platform Tools** (for ADB)

### Verification Checklist

```bash
# Verify virtualization enabled (Windows PowerShell as Admin)
Get-ComputerInfo | Select-Object HyperVRequirementVirtualizationFirmwareEnabled

# Expected: True

# Verify disk space
Get-Volume | Select-Object DriveLetter, Size, SizeRemaining

# Expected: C: drive with 25+ GB free

# Verify RAM
[math]::Round((Get-ComputerInfo).TotalPhysicalMemory / 1GB, 2)

# Expected: 8+ GB available
```

---

## Glossary

### Key Terminology

| Term | Definition |
|------|-----------|
| **VM (Virtual Machine)** | Software-based computer isolated from host system |
| **OVA/OVF** | Virtual machine export format (ready-to-import image) |
| **Snapshot** | Point-in-time backup of VM state (revertible) |
| **NAT** | Network Address Translation; VM accesses internet via host gateway |
| **Host-Only** | Private network between host and VM(s), no internet access |
| **ADB** | Android Debug Bridge; tool for communicating with Android devices |
| **Genymotion** | Fast Android emulator optimized for security testing |
| **Burp Suite** | Proxy tool for intercepting and analyzing HTTP/HTTPS traffic |
| **Frida** | Dynamic instrumentation framework for hooking/injecting code into apps |

---

## Setup Instructions

### Phase 1: System Preparation

#### Step 1.1: Enable Virtualization

1. Restart computer
2. Enter BIOS (press Delete, F2, F10, or F12 during startup)
3. Find "Virtualization Technology" or "VT-x" setting
4. Enable it
5. Save and exit BIOS

**Verification:**
```powershell
Get-ComputerInfo | Select-Object HyperVRequirementVirtualizationFirmwareEnabled
# Output: True
```

#### Step 1.2: Clean Up Disk Space

1. Delete unused VirtualBox VMs
2. Clear Downloads folder
3. Run `cleanmgr` (Disk Cleanup utility)
4. Target: 25+ GB free on C: drive

**Verification:**
```powershell
(Get-Volume | Where-Object {$_.DriveLetter -eq 'C'}).SizeRemaining / 1GB
# Output: 25+ GB
```

### Phase 2: VM Import & Configuration

#### Step 2.1: Download Mobexler OVA

1. **Open:** https://drive.google.com/file/d/1rd8g3bmK_XMTtb6PlcfIwjyoJ-mEhAk5/view?usp=sharing
2. **Download:** Mobexler.ova file (~5-15 GB)
3. **Location:** `C:\Users\LENOVO\Downloads\Mobexler.ova`

#### Step 2.2: Import into VMware

1. **Open VMware Workstation**
2. Go to **File → Open** (or **File → Import**)
3. **Select** Mobexler.ova file
4. **Choose location** to save VM (default: `C:\Users\[user]\Documents\Virtual Machines\`)
5. Click **Import** → Wait 5-10 minutes

#### Step 2.3: Configure Network Adapters

1. **Right-click** Mobexler VM in VMware
2. Select **Settings** (or **Edit Virtual Machine Settings**)
3. **Configure Adapter 1:**
   - Select **Network Adapter**
   - Choose **NAT** from dropdown
4. **Configure Adapter 2:**
   - Click **Add → Network Adapter**
   - Choose **Host-Only** (usually VMnet1)
   - Click **OK**

**Expected Result:**
```
Network Adapter 1: NAT ✓
Network Adapter 2: Host-Only (VMnet1) ✓
```

### Phase 3: Mobexler Boot & Configuration

#### Step 3.1: Power On & Login

1. **Power on** Mobexler VM in VMware
2. **Wait** for Linux boot (30-60 seconds)
3. **Login credentials:**
   - Username: `root`
   - Password: `mobexler`

#### Step 3.2: Verify Network Configuration

1. **Open terminal** (right-click desktop → Open Terminal, or Ctrl+Alt+T)
2. **Check IP addresses:**

```bash
ip a
```

**Expected Output:**
```
1: lo: <LOOPBACK,UP,LOWER_UP>
   inet 127.0.0.1/8

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP>
   inet 192.168.153.144/24    ← NAT Adapter

3: ens34: <BROADCAST,MULTICAST,UP,LOWER_UP>
   inet 192.168.40.5/24       ← Host-Only Adapter
```

#### Step 3.3: Configure Host-Only Adapter (if needed)

If ens34 shows no IP:

```bash
# Bring up the interface
sudo ip link set ens34 up

# Assign IP address
sudo ip addr add 192.168.40.5/24 dev ens34

# Verify
ip a
```

#### Step 3.4: Test Internet Access

```bash
# Test NAT connectivity
ping 8.8.8.8

# Or open Firefox and visit a website
firefox &
```

**Expected:** Successful ping responses or Firefox loads websites

### Phase 4: Android Emulator Setup

#### Step 4.1: Install Genymotion (on host)

1. **Go to:** https://www.genymotion.com/download
2. **Download:** Genymotion For Fun (free version)
3. **Install** on your Windows host (outside any VM)
4. **Create account** (free)

#### Step 4.2: Install VirtualBox Extension Pack (critical!)

1. **Go to:** https://www.virtualbox.org/wiki/Downloads
2. **Download:** VirtualBox Extension Pack (matching your VirtualBox version)
3. **Double-click** the downloaded file
4. **Click Install** → Authorize installation

#### Step 4.3: Create Android Device in Genymotion

1. **Open Genymotion**
2. **Click "Create"** or **"+"** button
3. **Search:** "Pixel 6" (or any device)
4. **Select:** Android 7.0+ version (e.g., Android 7.1.0 Nougat)
5. **Click "Install"** → Wait 5-15 minutes for download
6. Once installed, click **"Start"** → Wait 2-3 minutes for boot

#### Step 4.4: Note Emulator IP Address

1. **Look at Genymotion window** (running emulator)
2. **Find IP address** (usually displayed at top-right or status bar)
3. **Example:** `192.168.56.101` or `10.0.2.15`
4. **Write it down!**

### Phase 5: Connect Mobexler to Android Emulator

#### Step 5.1: Connect via ADB

In Mobexler terminal:

```bash
# Connect to the emulator
adb connect 192.168.56.101:5555

# Replace 192.168.56.101 with your actual Genymotion IP!
```

**Expected Output:**
```
connected to 192.168.56.101:5555
```

#### Step 5.2: Verify Connection

```bash
# List connected devices
adb devices
```

**Expected Output:**
```
List of attached devices
192.168.56.101:5555         device
```

If you see **"device"** → ✅ **Connection successful!**

#### Step 5.3: Troubleshoot if Needed

```bash
# Restart ADB server
adb kill-server
adb start-server

# Try connecting again
adb connect 192.168.56.101:5555

# Verify
adb devices
```

### Phase 6: Create CLEAN Snapshot

#### Step 6.1: Shut Down Mobexler

```bash
# In Mobexler terminal
sudo shutdown -h now
```

Or via VMware:
- Click **VM → Power Off**
- Wait for shutdown (10-20 seconds)

#### Step 6.2: Create Snapshot

1. **In VMware**, right-click Mobexler VM
2. Select **Snapshot → Take Snapshot**
3. **Name:** `CLEAN`
4. **Description:** `Initial setup - NAT + Host-Only networks configured, ADB ready`
5. Click **Take Snapshot**

**Expected:** Snapshot appears in VMware Snapshots panel

---

## Verification Steps

### Checkpoint 1: VMware & Network

```bash
# In Mobexler terminal
ifconfig -a
# or
ip route
```

**Verify:**
- ✅ ens33 has IP (NAT)
- ✅ ens34 has IP (Host-Only)
- ✅ Default route via ens33 (internet)

### Checkpoint 2: Internet Access

```bash
# Test DNS
nslookup google.com

# Test ping
ping 8.8.8.8

# Test HTTP
curl -I https://www.google.com
```

**Verify:** All commands succeed

### Checkpoint 3: ADB Connection

```bash
# List devices
adb devices

# Get device info
adb shell getprop ro.build.version.release

# Get device model
adb shell getprop ro.product.model
```

**Verify:**
- ✅ Device listed as "device" (not "offline" or "unauthorized")
- ✅ Commands succeed

### Checkpoint 4: Snapshot

1. **VM → Snapshot → Manage Snapshots**
2. Verify **"CLEAN"** snapshot exists
3. Test revert (optional):
   - Create test file on desktop
   - **VM → Snapshot → Revert to CLEAN**
   - Verify file disappears after revert

---

## Troubleshooting

### Issue: Genymotion Won't Start

**Solution:**
1. Ensure **VirtualBox Extension Pack** is installed
2. Check VirtualBox opens without errors
3. Verify **RAM allocation** (4-6 GB recommended)
4. Restart Genymotion and try again

### Issue: ADB Connection Fails

**Symptoms:** `adb devices` shows nothing or "offline"

**Solutions:**
```bash
# Restart ADB
adb kill-server
adb start-server

# Try explicit connection
adb connect 192.168.56.101:5555 -vvv

# Check if emulator is still running
# (Look at Genymotion window - should see Android screen)

# If offline, try:
adb usb
adb tcpip 5555
```

### Issue: Mobexler Can't Reach Internet

**Symptoms:** `ping 8.8.8.8` fails

**Check:**
```bash
# Verify NAT adapter is up
ip a | grep ens33

# Check routes
ip route

# Should show: default via 192.168.X.X dev ens33
```

**Solution:**
- Restart Mobexler VM
- Check VMware NAT settings (Edit → Virtual Network Editor)
- Verify host has internet connectivity

### Issue: Host-Only Network Not Working

**Symptoms:** Ping to host IP fails from Mobexler

**Check:**
```bash
# Verify ens34 has IP
ip a | grep ens34

# Should show: inet 192.168.40.X/24
```

**Solution:**
```bash
# Manually bring up interface
sudo ip link set ens34 up

# Assign IP (if missing)
sudo ip addr add 192.168.40.5/24 dev ens34

# Verify
ping 192.168.40.1  # Should reach VMware host
```

### Issue: Snapshot Revert Takes Too Long

**Expected:** 30 seconds to 2 minutes

**If longer:**
- Close all applications before reverting
- Ensure sufficient free disk space (20+ GB)
- Check disk health (`chkdsk C: /F` in Windows)

---

## Documentation & Screenshots

### Screenshots to Add

Add screenshots to this README in the following locations:

#### 1. **Prerequisites Section**
- [ ] Windows Task Manager → Performance → CPU (virtualization enabled)
- [ ] Disk Management → Free space on C: drive

#### 2. **Setup Instructions Section**
- [ ] VMware Workstation with Mobexler imported
- [ ] VMware Network configuration (2 adapters)
- [ ] Mobexler login screen
- [ ] Mobexler terminal with `ip a` output showing both adapters

#### 3. **Android Emulator Section**
- [ ] Genymotion main window with device created
- [ ] Genymotion running Android emulator (show IP address)
- [ ] Mobexler terminal showing `adb devices` with device connected

#### 4. **Snapshot Section**
- [ ] VMware Snapshot Manager showing "CLEAN" snapshot
- [ ] Confirmation message after snapshot creation

### Image Format Guide

```markdown
![Description](./screenshots/01-vmware-import.png)
```

**Create a `screenshots/` folder** in this repository and add images with names:
- `01-vmware-network-config.png`
- `02-mobexler-login.png`
- `03-mobexler-terminal-ip-a.png`
- `04-genymotion-device-running.png`
- `05-adb-devices-connected.png`
- `06-snapshot-created.png`

---

## Lab Completion Checklist

- [ ] Virtualization enabled in BIOS
- [ ] 25+ GB disk space available
- [ ] VMware Workstation installed
- [ ] VirtualBox installed
- [ ] VirtualBox Extension Pack installed
- [ ] Mobexler OVA imported into VMware
- [ ] Network Adapter 1 configured as NAT
- [ ] Network Adapter 2 configured as Host-Only
- [ ] Mobexler boots and logs in successfully
- [ ] ens33 (NAT) has IP address
- [ ] ens34 (Host-Only) has IP address
- [ ] Internet access verified from Mobexler
- [ ] Genymotion installed on host
- [ ] Android device created in Genymotion
- [ ] Genymotion emulator running with visible IP
- [ ] ADB connects to emulator successfully
- [ ] `adb devices` shows device as "device"
- [ ] CLEAN snapshot created in VMware
- [ ] All screenshots captured
- [ ] README completed and documented

---

## Network Configuration Reference

### Mobexler IP Addresses

| Adapter | Network | Purpose | Example IP |
|---------|---------|---------|-----------|
| ens33 | NAT (external) | Internet access | 192.168.153.144 |
| ens34 | Host-Only (private) | Lab communication | 192.168.40.5 |

### Genymotion Emulator IPs

| Network | Typical IP Range | Access From |
|---------|-----------------|------------|
| Default (NAT via VirtualBox) | 10.0.2.15 | Host only |
| Host-Only (if configured) | 192.168.56.X | Mobexler + Host |

### ADB Connection

```bash
# Default ADB port
adb connect <IP>:5555

# Example with Genymotion at 192.168.56.101
adb connect 192.168.56.101:5555
```

---

## Resources & Links

- **Mobexler Official:** https://www.mobexler.com/
- **Mobexler Setup Guide:** https://www.mobexler.com/set-up
- **Genymotion:** https://www.genymotion.com/
- **VirtualBox:** https://www.virtualbox.org/
- **VMware Workstation:** https://www.vmware.com/products/workstation/
- **Android Debug Bridge (ADB):** https://developer.android.com/studio/command-line/adb

---

## Notes for Future Labs

- **Snapshot Management:** Keep the CLEAN snapshot for quick resets between labs
- **Network Security:** Host-Only network ensures test data doesn't leak to external network
- **ADB Best Practices:** Always verify device is "device" (not "offline") before testing
- **Disk Space:** Genymotion devices can be 1-3 GB each; manage multiple devices carefully
- **Performance:** Allocate at least 4-6 GB RAM to Genymotion emulator for smooth operation

---

## Author Notes

Created: May 2026  
Lab Name: LAB-1-SECURITY - Mobexler + Android Emulator Setup  
Purpose: Mobile Application Security Testing Foundation  
Status: ✅ Complete

---

**Next Lab:** [Link to LAB-2 when available]

---

## License

[Add your license here, e.g., MIT, Apache 2.0]

## Contributing

Contributions welcome! Please fork, create a branch, and submit a pull request.

---

**Last Updated:** May 1, 2026
