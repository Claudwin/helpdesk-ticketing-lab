
# TKT-007 — Wifi Disconnecting 

## Diagnostic Path for powermanagement suspending wifi

| Field | Value |
|-------|-------|
| **User** | Thomas Herrera |
| **Priority** | Medium |
| **Category** | Network |

---

## Symptom

User reported that their wifi keeps disconnecting but they have no issues when connecting to ethernet. 

### Wi-Fi Disconnection Diagnostic Workflow

### 1. Scope & Pattern Isolation

* **Device Scope:** Test a known-good device in the identical physical location. 

  * *Multi-device drop:* Escalate to infrastructure troubleshooting (router, switch, or ISP).
  * *Single-device drop:* Isolate troubleshooting to the specific client's hardware, drivers, or OS configuration.
* **Temporal Patterns:** Document precisely when the drops occur. Note if disconnects align with specific times of day, system resume events (waking from sleep), or physical movement between access points (roaming failures).

### 2. Live Adapter State Analysis

```netsh wlan show interfaces```

* **Radio Type & Band:** Verify if the client is on 2.4 GHz, 5 GHz, or 6 GHz.
* **Signal Quality:** Evaluate the signal percentage; consistent metrics below 70% indicate range or obstruction issues.
* **Link Rates:** Inspect the Transmit (Tx) and Receive (Rx) rates. Extreme fluctuations or severely asymmetric rates indicate high environmental interference.
* **Connection State:** Confirm if the adapter is maintaining a connected state or getting stuck in associating/authenticating loops.

### 3. Historical Telemetry Review

Generate a comprehensive wireless connectivity report to pinpoint exact historical failure points: 

```netsh wlan show wlanreport```

* **Log Location:** Open the generated HTML file located at C:\ProgramData\Microsoft\Windows\WlanReport\wlan-report-latest.html.
* **Analysis Focus:** Scan the graphical timeline and session summaries specifically for explicit disconnect events and their associated 802.11 reason codes.

### 4. Timestamp Correlation

* **Idle vs. Active Windows:** Map the exact drop timestamps against user activity.
* **The Diagnostic Tell:** If drops predominantly cluster during periods of system idleness rather than active network utilization, prioritize power management configurations over environmental interference.

### 5. Driver & Hardware Stability Logs

Review system event logs to determine if the physical adapter or its driver is failing: 

* **WLAN-AutoConfig Log:** Check for operational changes, connection drops, and explicit infrastructure-driven disconnections.
* **System Log:** Search for hardware-level events, specifically adapter resets or driver timeout warnings (e.g., Netwtw10/Netwtw12 for Intel cards).
* **Stability Rule:** An absolute absence of adapter resets or driver crashes effectively rules out underlying driver instability.

### 6. Signal & Roaming Validation

* **Fixed-Location Testing:** Monitor signal strength behavior at the exact spot where the drop occurs.
* **Range Rule:** If the netsh wlan telemetry confirms adequate, stable signal strength at the exact moment of a drop while the device is stationary, physical range and roaming boundaries are ruled out as root causes.

### 7. Power Management Verification

If timestamp correlation or logs point to idle-state drops, verify and disable aggressive OS power-saving features: 

* **Device Manager:** Open the properties for the wireless network adapter, navigate to the **Power Management** tab, and uncheck *“Allow the computer to turn off this device to save power.”*
* **Power Options:** Open the active Windows Power Plan, expand **Wireless Adapter Settings** -> **Power Saving Mode**, and change both 'On battery' and 'Plugged in' configurations to **Maximum Performance**.

