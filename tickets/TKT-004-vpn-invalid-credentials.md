
# TKT-004 — VPN Invalid Credentials

| Field | Value |
|-------|-------|
| **User** | Lisa Wong |
| **Priority** | High |
| **Category** | Network |
| **Opened** | 6:10 am |
| **Resolved** | 5:30 am |
| **Time to resolve** | 20 MINS |
| **SLA met** | Yes (target: 1h first response / 4h resolution) |

---

## Symptom

User reported that they were unable to access their account due to VPN connectivity issue. Mentioned that they reset their password last week so they are confident they are entering the correct password. Said is had no issues connecting when in the ofice

## Diagnosis

**Reviewed Lisa account to verify the properties.**

```powershell
Get-ADUser -Identity <her SamAccountName> -Properties Enabled,LockedOut,PasswordExpired,AccountExpirationDate | Format-List
```
Confirmed that Lisa's account expired AccountExpirationDate: 9/1/2026 12:00:00 AM 

Confirmed Lisa didn't have access.  


## Resolution

**Reset Expiration Date on Account**


## Cause / Fix / Prevention

Account Expired blocking the account from being accessed entirely

**Fix:** 
```powershell 
Set-ADUser -Identity lwong -AccountExpirationDate $null
```

Cleared the expiration date property on the account so it never expires.

**Prevention**
Flag accounts nearing their expiration date in advance (a scheduled Get-ADUser -Filter report is a common approach) so IT can confirm with the department whether an extension is needed before it lapses and generates a ticket.


## Screenshots

**Domain account lockout policy — threshold, duration, and observation window**

![Access denied error](../screenshots/ad/tkt-003-access-denied.png)

**Failed authentication sequence — five 1326 errors followed by 1909 once the threshold tripped**

![Failed logons](../screenshots/ad/tkt-001-failed-logons.png)

