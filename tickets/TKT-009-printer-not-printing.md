
# TKT-009 — Printer Not printing

## Diagnostic Path for Printer

| Field | Value |
|-------|-------|
| **User** | Lisa Wong |
| **Priority** | Medium |
| **Category** | Software |

---

## Symptom

User reported that they are unable to open their outlook application. States they tried a restart but still unable to launh outlook application

## Diagnosis Workflow

1. **Physical & Basic Checks** — Confirm the printer has power, paper loaded properly, and no paper jams.Check ink or toner levels to ensure cartridges are not empty or poorly seated.Turn off the printer, unplug it for one minute, and turn it back on.Verify that USB cables are secure or network cables are plugged in
2. **Check the Print Queue & Spooler** — Open Printers & scanners in Windows settings and select your device.Open the print queue to clear any stuck or paused print jobs.Open Services from the Start menu, locate Print Spooler, and make sure it is running and set to automatic. Restart the service if jobs remain stuck.
3. **Network vs. USB Troubleshooting** — For USB connections, try a different USB port on the computer.For network printers, ping the printer's IP address to check network reachability.Use manufacturer tools like the HP Diagnose & Fix utility if managing an HP device.
4. **Drivers and Device Manager** — Open Device Manager, expand print queues, and look for error symbols.Update or reinstall the printer driver using official packages from the manufacturer.Run the built-in Windows printer troubleshooter under system settings