# TKT-010 — Slow Computer Performance 

## Slow Computer Performance Diagnostic Workflow

| Field | Value |
|-------|-------|
| **User** | Emily David |
| **Priority** | Medium |
| **Category** | Hardware |

---

## Symptom

User reported that they are unable to open their outlook application. States they tried a restart but still unable to launh outlook application

### Slow Computer Performance Diagnostic Workflow

### 1. Scope & Resource Baseline (The Initial Tell)

Isolate whether the slowdown is constant, tied to specific apps, or completely saturating the hardware. 

* **Methodology:** Open **Task Manager** ```(Ctrl + Shift + Esc)``` and review the **Performance** tab.
* **Analysis Focus:** Look at CPU, Memory, and Disk utilization percentages.
* **The Diagnostic Tell:** 

  * *100% Disk Usage:* Points to a failing hard drive (HDD), a corrupted Windows Search/SysMain service, or a pending Windows Update background installation.
  * *90%+ Memory Usage:* Indicates insufficient physical RAM for the user's workload or a software application with a severe memory leak.
  * *High CPU Usage:* Usually points to an aggressive background process, anti-malware scan, or browser tab lockup.

### 2. Network vs. Local Isolation

Determine if the computer itself is slow or if the user is experiencing network latency. 

* **Methodology:** Have the user perform local tasks (e.g., opening a blank Notepad file or navigating local folders) vs. network tasks (e.g., loading web pages or opening files from a shared network drive).
* **The Diagnostic Tell:** If local actions are snappy but web browsers and network shares lag, abandon local hardware troubleshooting and pivot immediately to network bandwidth, DNS, or Wi-Fi diagnostics.

### 3. Startup & Background Process Triage

Identify resource-heavy applications launched automatically during boot. 

* **Methodology:** In Task Manager, navigate to the **Startup apps** tab (or run msconfig). Sort the list by **Startup impact**.
* **Analysis Focus:** Look for unneeded third-party apps (e.g., communication apps, cloud sync tools, launcher utilities) set to "High impact."
* **Resolution:** Disable non-essential startup items and reboot. If performance dramatically increases, the issue is isolated to software bloat.

### 4. Storage Health & Capacity Check

Verify if the storage medium has the physical space and health to function properly. 

* **Capacity Rule:** Windows requires a absolute minimum of 10% to 15% of the primary drive (C:) to be free for virtual memory paging and temporary file allocation. If the drive is in the "red bar" zone in File Explorer, performance will tank.
* **Hardware Health Check:** For mechanical drives, check Event Viewer for disk errors or warnings. Run a quick SMART health check via PowerShell: 

```powershell
 Get-PhysicalDisk | Select-Object DeviceId, FriendlyName, OperationalStatus, HealthStatus 
```

### 5. Thermal Throttling Verification (The Physical Pivot)

Determine if the computer's processor is intentionally slowing itself down to prevent overheating. 

* **Methodology:** Observe the CPU clock speed in Task Manager under the Performance tab while the computer is under load.
* **The Diagnostic Tell:** If the CPU speed is locked at an extremely low frequency (e.g., 0.79 GHz or 1.1 GHz) despite high utilization, the system is thermal throttling.
* **Physical Inspection:** Check for clogged dust vents, a failed internal fan, or a laptop sitting on a soft surface (like a blanket) that blocks airflow.

### 6. OS Integrity & Update Status

Rule out background maintenance tasks or corrupted OS files. 

* **Pending Updates:** Check **Settings** -> **Update & Security**. A Windows Update stuck in a failed install loop or actively compressing files in the background is a primary cause of temporary slowdowns.
* **File Corruption Check:** Run a deployment image and system file check via an elevated Command Prompt to rule out corrupted OS binaries: 


```DISM.exe /Online /Cleanup-image /Restorehealth```
```sfc /scannow```

### 7. Profile & Malware Isolation 

Isolate whether the slowdown is system-wide or restricted to a corrupted user profile or malware infection. 

* **Test Profile:** Log out the user and log into a local admin "test" profile.
* **The Pivot:** If the test profile runs perfectly, the root cause is corrupted registry keys, an overloaded browser cache, or a bloated app data folder specific to the user's profile. If both profiles are slow, run a thorough malware scan using Windows Defender or your enterprise endpoint tool.
