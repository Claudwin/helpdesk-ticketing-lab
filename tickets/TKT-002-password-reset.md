
# TKT-002 — Password Reset

| Field | Value |
|-------|-------|
| **User** | Emily David edavis |
| **Priority** | High |
| **Category** | Password / Login |
| **Opened** | 5:29 am |
| **Resolved** | 6:07 am |
| **Time to resolve** | 37 minutes |
| **SLA met** | Yes (target: 1h first response / 4h resolution) |

---

## Symptom

User reported that her password was no longer working to sign-in. She had attempted
to log in several times without success and needed access for an upcoming presentation.
No error code was provided by the user only that the credentials "keep rejecting."

"Locked out" can mean four different things, and each one needs a different fix:

Locked — too many wrong passwords tripped a limit. Temporary, and it clears on its own after a while.
Disabled — an admin switched the account off. Waiting won't help.
Password expired — the password aged out and needs to be changed.
Account expired — the account itself has an end date that has passed.

The user sees roughly the same thing in all four cases. So the first job was figuring out which one this was.

## Diagnosis

**Reviewed Emily's account to verify the properties.**

```powershell
Get-ADUser -Identity edavis -Properties PasswordExpired,PasswordLastSet | Format-List
```

Returned `PasswordExpired: True`. The password exceeded the domain maximum age and expired. The account was neither locked or disabled


| Property | Value | Hypothesis |
|----------|-------|------------|
| `Enabled` | True | **Ruled out** — account not disabled |
| `AccountExpirationDate` | (empty) | **Ruled out** — account has no expiry date |
| `LockedOut` | True | **Confirmed** — policy lockout is active |
| `BadLogonCount` | 5 | **Ruled out** |
| `LastBadPasswordAttempt` | **Ruled out** |
| `PasswordExpired` | True | **Confirmed**  |


## Resolution

**Reset and force change at next logon**

```powershell
Set-ADAccountPassword -Identity sjones -Reset -NewPassword (Read-Host -AsSecureString "New password")
Set-ADUser -Identity sjones -ChangePasswordAtLogon $true
```

## Cause / Fix / Prevention

Cause — Password exceeded the domain maximum age and expired; the account was neither locked nor disabled.

**Fix:** Administrative password reset with forced change at next logon, after verifying requester identity.

**Prevention**

Enable expiry notifications so users are warned before the deadline, and publish a self-service reset article (that's your KB-001).

End-user guide for password resets: KB-001

## Screenshots

**Domain account lockout policy — threshold, duration, and observation window**

![Lockout policy](../screenshots/ad/tkt-001-lockout-policy.png)

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
