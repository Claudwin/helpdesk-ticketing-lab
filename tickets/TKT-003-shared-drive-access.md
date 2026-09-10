
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

**Reviewed Thomas account to verify the properties.**

```powershell
Get-ADUser -Identity therrera -Properties MemberOf | Select -ExpandProperty MemberOf
```
Confirmed that tomas has access to the sales folder but not the reports folder.

Signed in as therrera and tried to get access to reports folder to recreate the issue. 

Confirmed Tomas didn't have access.  


## Resolution

**Enable inheritance on reports folder**


## Cause / Fix / Prevention

Permission inheritance was not enabled on reports folder

**Fix:** On Reports → Properties → Security → Advanced → Enable inheritance (this restores DL_FileShare_Sales_Modify's permissions from the parent). Screenshot the restored ACL as tkt-003-ntfs-reports-fixed.png.

**Prevention**

Enable enable permissions inheritance when creating new folders.

End-user guide to request shared drive access: KB-002 - Requesting shared drive access

## Screenshots

**Unable to access reports folder**

![Access denied](../screenshots/zohodesk/tkt-003/Screenshot%202026-09-02%20at%205.26.59%E2%80%AFAM.png)

**Review Tomas Access**

![Tomas Access](../screenshots/zohodesk/tkt-003/Screenshot%202026-09-01%20at%208.51.17%E2%80%AFPM.png)

**Enabled Inheritance on reports folder**

![Enable Inheritance](../screenshots/zohodesk/tkt-003/Inheritance%20enabled.png)

**Zoho Desk ticket thread — triage, first response, and resolution**

![Zoho ticket](../screenshots/zoho/tkt-001-ticket-thread.png)
