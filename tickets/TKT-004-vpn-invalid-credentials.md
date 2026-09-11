
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

Get-ADUser -Identity lwong -Properties Enabled,LockedOut,PasswordExpired,AccountExpirationDate


| Property | Value | Hypothesis |
|---|---|---|
| Enabled | True | Ruled out — account not disabled |
| LockedOut | False | Ruled out — not a lockout |
| PasswordExpired | True | Ruled out — attributable to `ChangePasswordAtLogon` set at initial provisioning on all lab accounts, consistent with the user's report that her reset worked fine afterward |
| AccountExpirationDate | 9/1/2026 (past) | Confirmed |

Verified via interactive logon as `lisa.wong@enterprise.lab`, which returned
an account-expired message distinct from a standard bad-credential error —
confirming the cause before any change was made.

## Resolution
Extended the expiration date and confirmed restored access.

Set-ADUser -Identity lwong -AccountExpirationDate "null"


Verified with `Get-ADUser -Properties AccountExpirationDate`, then repeated
the interactive logon test — succeeded normally.

## Cause / Fix / Prevention
**Cause:** `AccountExpirationDate` had passed. This blocks all
authentication independent of password state, which is why the symptom
presented as a credentials issue when the password was never actually
invalid.

**Fix:** Extended the expiration date, verified successful logon before
closing the ticket.

**Prevention:** Run a scheduled report on accounts nearing their expiration
date so the relevant department can confirm ahead of time whether an
extension is needed, rather than discovering it only after the account has
already lapsed and a ticket is filed.


## Screenshots

**Get-ADUser -Identity lwong -Properties**

![Get-ADUser -Identity lwong -Properties](../screenshots/zohodesk/tkt-004/Identity%20lwong%20Propertie.png)

**Extended the expiration date and confirmed restored access.**

![Extended Expiration](../screenshots/zohodesk/tkt-004-Extended%20the%20expiration.png)

**Zoho Desk ticket thread — triage, first response, and resolution**

![Zoho ticket](../screenshots/zohodesk/tkt-004/tkt-004%20resolution%20email.png)

