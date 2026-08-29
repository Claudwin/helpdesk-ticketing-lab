# TKT-001 — Locked out of my account

| Field | Value |
|-------|-------|
| **User** | Michael Chen (`mchen`) — IT |
| **Priority** | High |
| **Category** | Password / Login |
| **Opened** | 7:15 am |
| **Resolved** | 7:745am |
| **Time to resolve** | 30 minutes |
| **SLA met** | Yes (target: 1h first response / 4h resolution) |

---

## Symptom

User reported that his password was no longer working to sign-in. He had attempted
to log in several times without success and needed access to a departmental file
share before a scheduled meeting. No error code was provided by the user only that
the credentials "stopped working."

"Locked out" can mean four different things, and each one needs a different fix:

Locked — too many wrong passwords tripped a limit. Temporary, and it clears on its own after a while.
Disabled — an admin switched the account off. Waiting won't help.
Password expired — the password aged out and needs to be changed.
Account expired — the account itself has an end date that has passed.

The user sees roughly the same thing in all four cases. So the first job was figuring out which one this was.

## Diagnosis

**1. Confirmed the domain lockout policy was in effect.**

```powershell
Get-ADDefaultDomainPasswordPolicy
```

Returned `LockoutThreshold: 5`, `LockoutDuration: 00:30:00`,
`LockoutObservationWindow: 00:30:00`. The lockout threshold is set to 5 five wrong passwords locks the account. If it had been 0, lockouts would be switched off in this domain and I'd have to look somewhere else. It's 5, so a lockout is possible.

The other two settings: the account stays locked for 30 minutes, and the counter of wrong passwords resets after 30 minutes without a failure.

**2. Queried the domain for locked accounts.**

```powershell
Search-ADAccount -LockedOut
```

Returned a single object: `CN=Michael Chen,OU=Users,OU=IT,OU=Departments,DC=enterprise,DC=lab`, with `LockedOut: True`. I used  `Search-ADAccount` because it's built for exactly this asking about account states like locked, disabled, or expired.

**3. Pulled full account state to rule out the other cases.**

```powershell
Get-ADUser mchen -Properties LockedOut,BadLogonCount,LastBadPasswordAttempt |
    Format-List Name,LockedOut,BadLogonCount,LastBadPasswordAttempt
```

| Property | Value | Hypothesis |
|----------|-------|------------|
| `Enabled` | True | **Ruled out** — account not disabled |
| `AccountExpirationDate` | (empty) | **Ruled out** — account has no expiry date |
| `LockedOut` | True | **Confirmed** — policy lockout is active |
| `BadLogonCount` | 5 | Matches the policy exactly |
| `LastBadPasswordAttempt` | 8/29/2026 7:04:33 AM | Matches when users said they had issues |
| `PasswordExpired` | True | **Not ruled out** — see below |

`BadLogonCount` landing exactly on the threshold value of 5 is the strongest single
indicator: the account did not fail once or twice, it accumulated failures until the
policy tripped.

**4. Identified the source of the failed authentications.**

```powershell
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4740} -MaxEvents 5 |
    Format-List TimeCreated,Message
```

Event ID 4740 ("A user account was locked out") returned a single entry at
`8/29/2026 7:04:33 AM`, matching `LastBadPasswordAttempt`, for account `mchen`
(SID `S-1-5-21-628932227-2289402440-423683375-1136`), with **Caller Computer Name: DC01**.


## Resolution

```powershell
Unlock-ADAccount -Identity mchen
```

`Unlock-ADAccount` clears the `lockoutTime` attribute. It does **not** modify the
password. This distinction was deliberate: the user's credential was not the problem,
and resetting it would have forced an unnecessary credential change to solve a problem
that did not exist. Password resets are a separate remediation for a separate symptom.

Verified post-remediation:

```powershell
Get-ADUser mchen -Properties LockedOut,BadLogonCount,LastBadPasswordAttempt |
    Format-List Name,LockedOut,BadLogonCount,LastBadPasswordAttempt
```

`LockedOut: False`, `BadLogonCount: 0`. A follow-up `Search-ADAccount -LockedOut`
returned no results, confirming no other accounts in the domain were affected.

The user was advised that a password change would be required at next sign-in, per the
secondary finding above.

## Cause / Fix / Prevention

Cause — Five wrong passwords in a row hit the domain's lockout limit of 5, which locked the account for 30 minutes. Event `4740` confirmed one lockout event, with all the failures coming from a single source.

**Fix:** Unlocked the account with `Unlock-ADAccount`. Confirmed `LockedOut` went back to `False` and `BadLogonCount` reset to 0. No password change, since the password itself was working.

**Prevention — Two things:**

If an account keeps locking, check the Caller Computer Name in event 4740 before unlocking, and fix whatever is on that machine. Unlocking alone just means the same ticket comes back.
This account has two different names on it: the sign-in name is mchen, but the email-style name is michael.chen@enterprise.lab. Different systems expect different ones, and a user who types the wrong one gets a generic password error with no hint about what's actually wrong. Try it a few times and you get this exact lockout. Picking one naming convention across the directory would remove that whole category of ticket.

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