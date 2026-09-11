
# TKT-003 — Shared Drive Access

| Field | Value |
|-------|-------|
| **User** | Tomas Herrera |
| **Priority** | Medium |
| **Category** | Networ |
| **Opened** | 6:52 pm |
| **Resolved** | 5:08 am |
| **Time to resolve** | 10 hours |
| **SLA met** | Yes (target: 4h first response / 1 day resolution) |

---

## Symptom

User reported that they were unable to reports folder and that they are getting I don't have permission message. User states they still have access to the sales folder and only unable to access reports folder.


## Diagnosis
Reviewed the access chain from identity through to the folder.

Get-ADUser -Identity therrera -Properties MemberOf
Get-ADGroupMember -Identity DL_FileShare_Sales_Modify


| Layer | Value | Hypothesis |
|---|---|---|
| Group membership | `therrera` in `GG_Sales_Users` | Ruled out — identity chain intact |
| Group nesting | `GG_Sales_Users` in `DL_FileShare_Sales_Modify` (Modify) | Ruled out — AGDLP chain correct |
| Share permissions on `Sales` | `DL_FileShare_Sales_Modify` — Modify | Ruled out |
| NTFS on `Sales` (parent) | `DL_FileShare_Sales_Modify` — Modify | Ruled out |
| NTFS on `Reports` (subfolder) | Inheritance disabled; group absent from ACL | Confirmed |

Reproduced as the user via `runas /netonly` against the UNC path
(`\\DC01\Sales\Reports`) — plain `runas` first returned error 1385, since
domain controllers restrict interactive logon for standard accounts by
design. `/netonly` applies the credential to network authentication only,
which is the correct way to test share/NTFS access without an interactive
session. Result matched the reported symptom.

## Resolution
1. Re-enabled inheritance on `Reports`, restoring `DL_FileShare_Sales_Modify`
   from the parent
2. Verified the group present in the resulting ACL
3. Re-ran the `/netonly` access test — succeeded

## Cause / Fix / Prevention

**Cause:** Inheritance had been disabled on the `Reports` subfolder,
disconnecting it from the parent's permissions. The share, the parent
folder, and the group chain were all correctly configured throughout.

**Fix:** Re-enabled inheritance on `Reports`, verified restored access.

**Prevention:** Audit subfolder permissions after any share reorganization —
disabling inheritance on a child folder produces no warning and silently
diverges it from the parent.

## Screenshots

**Unable to access reports folder**

![Access denied](../screenshots/zohodesk/tkt-003/Screenshot%202026-09-02%20at%205.26.59%E2%80%AFAM.png)

**Review Tomas Access**

![Tomas Access](../screenshots/zohodesk/tkt-003/Screenshot%202026-09-01%20at%208.51.17%E2%80%AFPM.png)

**Enabled Inheritance on reports folder**

![Enable Inheritance](../screenshots/zohodesk/tkt-003/Inheritance%20enabled.png)

**Zoho Desk ticket thread — triage, first response, and resolution**

![Zoho ticket](../screenshots/zohodesk/tkt-003/tkt-003%20email%20response.png)
