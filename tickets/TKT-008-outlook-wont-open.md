
# TKT-008 — Outlook Won't Open 

## Diagnostic Path for Outlook 

| Field | Value |
|-------|-------|
| **User** | Lisa Wong |
| **Priority** | Medium |
| **Category** | Software |

---

## Symptom

User reported that they are unable to open their outlook application. States they tried a restart but still unable to launh outlook application

### Outlook Startup Failure Diagnostic Workflow

### 1. Webmail Validation (Server vs. Client Isolation)

Verify mailbox health by having the user log into Outlook on the web (OWA). 

* **Successful Login:** Confirms the mailbox is active, credentials are correct, and the underlying server is healthy. The fault is strictly isolated to the local client.
* **The Diagnostic Tell:** This single initial check completely eliminates the entire server infrastructure and account status from the troubleshooting scope.

### 2. Safe Mode Isolation

Execute the launch command to bypass local customizations and third-party integrations: 

* **Command:** Press Win + R, type outlook.exe /safe, and press Enter.
* **Successful Launch:** Indicates that a faulty application add-in, custom toolbar, or corrupted viewing preference is blocking standard startup.
* **Failure to Launch:** Indicated a deeper core issue, shifting the diagnostic focus toward local profile corruption or corrupted application binaries.

### 3. COM Add-In Triage

If Safe Mode successfully bypasses the launch failure, isolate the specific root cause: 

* **Path:** Navigate to **File** -> **Options** -> **Add-ins**. Set the Manage dropdown to **COM Add-ins** and click **Go**.
* **Methodology:** Uncheck all enabled add-ins. Re-enable them one at a time, restarting Outlook standardly after each addition.
* **Resolution:** This systematic test rules individual add-ins in or out definitively.

### 4. Offline Data Storage (.OST) Inspection

Investigate the health and size of the local cached mailbox database: 

* **Path:** Navigate to %localappdata%\Microsoft\Outlook via File Explorer.
* **Analysis Focus:** Inspect the file size of the primary .ost file.
* **The Diagnostic Tell:** A bloated file approaching maximum limits (typically 50GB) or a file locked by a hung background process will routinely block standard application startup.

### 5. Mail Profile Reconstruction (The Pivot Step)

Determine if local user profile registry keys or configuration files are corrupt: 

* **Path:** Open **Control Panel** -> **Mail** -> **Show Profiles**.
* **Methodology:** Click **Add** to create a clean test profile. Configure the account and set the option to *“Prompt for a profile to be used”* or set the new profile as default.
* **The Pivot:** If Outlook successfully opens using the new profile, the original local profile is definitively corrupted. Abandon it and migrate the user to the new profile.

### 6. Application Binary Repair

Execute an application-level repair if a clean mail profile fails to resolve the launch crash: 

* **Path:** Open **Settings** -> **Apps** -> **Installed Apps**, locate Microsoft 365/Office, click the three dots, and select **Modify**.
* **Quick Repair:** Run this first to rapidly verify and replace missing or corrupted registry entries and file pointers locally without an internet connection.
* **Online Repair:** Fall back to this comprehensive option if Quick Repair fails. It performs a complete, fresh reinstall of the application binaries, ruling out the core installation suite as the point of failure.



