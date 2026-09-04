
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

**Domain account lockout policy — threshold, duration, and observation window**

![Access denied error](../screenshots/ad/tkt-003-access-denied.png)

**Failed authentication sequence — five 1326 errors followed by 1909 once the threshold tripped**

![Failed logons](../screenshots/ad/tkt-001-failed-logons.png)

**Locked account located and full account state pulled to rule out disabled and expired conditions**

![Search-ADAccount](../screenshots/ad/tkt-001-search-lockedout.png)

**Event 4740 showing the lockout event and Caller Computer Name**

![Event 4740](../screenshots/ad/tkt-001-event-4740.png)

**Unlock applied and verified — LockedOut False, BadLogonCount reset to 0**

![Unlock verified](../screenshots/ad/tkt-001-unlock-verified.png)

**Zoho Desk ticket thread — triage, first response, and resolution**

![Zoho ticket](../screenshots/zoho/tkt-001-ticket-thread.png)