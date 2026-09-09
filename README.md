# Help Desk Ticketing Lab

A working help desk built on Freshdesk, with tickets resolved against live Active Directory and Microsoft Azure lab environments rather than simulated on paper. Ten tickets were submitted, triaged, and worked to resolution under defined SLA targets, with six requiring hands-on remediation in a real directory or cloud environment.

**Environments:** Windows Server 2022 / Active Directory (`enterprise.lab`) · Microsoft Azure · Freshdesk (Free)

---

## Ticket Index

| ID | Issue | Category | Priority | Resolved In | SLA |
|----|-------|----------|----------|-------------|-----|
| [TKT-001](tickets/TKT-001-account-lockout.md) | Locked out of my account | Password / Login | High | Active Directory | _TBD_ |
| [TKT-002](tickets/TKT-002-password-reset.md) | Need a password reset ASAP | Password / Login | High | Active Directory | _TBD_ |
| [TKT-003](tickets/TKT-003-shared-drive-access.md) | Can't access shared drive | Network | Medium | Active Directory | _TBD_ |
| [TKT-004](tickets/TKT-004-vpn-invalid-credentials.md) | VPN says invalid credentials | Network | High | Active Directory | _TBD_ |
| [TKT-005](tickets/TKT-005-azure-files-mount-failure.md) | Can't reach the cloud file share | Network | High | Azure | _TBD_ |
| [TKT-006](tickets/TKT-006-blob-sas-expired.md) | Report link says access denied | Software | Medium | Azure | _TBD_ |
| [TKT-007](tickets/TKT-007-wifi-disconnecting.md) | Wi-Fi keeps disconnecting | Network / Wi-Fi | Medium | Documented | _TBD_ |
| [TKT-008](tickets/TKT-008-outlook-wont-open.md) | Outlook won't open | Email / Outlook | Medium | Documented | _TBD_ |
| [TKT-009](tickets/TKT-009-printer-not-printing.md) | Printer won't print | Printer | Low | Documented | _TBD_ |
| [TKT-010](tickets/TKT-010-laptop-slow.md) | Laptop is running slow | Hardware / Laptop | Low | Documented | _TBD_ |

---

## Problem

<!-- 3-5 sentences. What gap does a ticketing system fill? Frame it around: unstructured support requests arriving by email/hallway with no prioritization, no audit trail, no way to identify recurring issues, and no accountability for response time. Mention that the goal was not just to run tickets but to work them against real infrastructure so the diagnosis steps are genuine. -->

## Architecture

<!-- Describe the three connected pieces and how they relate. -->

**Freshdesk (Free tier)** — Ticket intake, categorization, priority assignment, agent workflow, and knowledge base. Configured with six ticket categories and five test contacts, two flagged VIP.

**Active Directory lab** — Windows Server 2022 running in Oracle VirtualBox, domain `enterprise.lab`. OU structure: `OU=Users,OU=<Department>,OU=Departments,DC=enterprise,DC=lab` with `GG_*_Users` global groups. Used to reproduce and resolve account, permission, and authentication issues.

**Azure subscription** — Pay-as-you-go, resources in `storage-lab-rg`. Used to reproduce and resolve cloud storage access issues involving Azure Files and Blob Storage.

<!-- Optional: add a simple diagram or ASCII sketch showing user -> Freshdesk -> agent -> AD/Azure. -->

## Implementation

### Priority definitions

| Priority | Criteria |
|----------|----------|
| High | Work stopped, hard deadline affected, or VIP user |
| Medium | Work impacted but a workaround exists |
| Low | Minor inconvenience, no work impact |

### SLA targets

| Priority | First Response | Resolution |
|----------|----------------|------------|
| High | 1 hour | 4 hours |
| Medium | 4 hours | 1 business day |
| Low | 1 business day | 3 business days |

### Ticket handling standard

Every ticket was worked to the same standard:

1. **Triage** — assign category and priority against the definitions above
2. **First response** — acknowledgment, clarifying questions where needed, statement of next steps
3. **Diagnosis** — documented in order, including steps that ruled out incorrect hypotheses
4. **Resolution** — commands run or configuration changed, with evidence captured
5. **Documentation** — Cause / Fix / Prevention recorded in the ticket before closure
6. **Status progression** — Open → In Progress → Pending (if awaiting user) → Resolved

## Screenshots

<!-- Pull 4-6 of your strongest images up here. The rest stay in the individual ticket files. Suggested picks below - replace filenames with your actuals. -->

**Domain account lockout policy — threshold, duration, and observation window**

![Lockout policy](../screenshots/zohodesk/tkt-001-lockout-policy.png)

**Failed authentication sequence — five 1326 errors followed by 1909 once the threshold tripped**

![Failed logons](../screenshots/zohodesk/tkt-001-failed-logons.png)

**Locked account located and full account state pulled to rule out disabled and expired conditions**

![Search-ADAccount](../screenshots/zohodesk/tkt-001-search-lockedout.png)

**Event 4740 showing the lockout event and Caller Computer Name**

![Event 4740](../screenshots/zohodesk/tkt-001-event-4740.png)

**Unlock applied and verified — LockedOut False, BadLogonCount reset to 0**

![Unlock verified](../screenshots/zohodesk/tkt-001-unlock-verified.png)

**Zoho Desk ticket thread — triage, first response, and resolution**

![Zoho ticket](../screenshots/zoho/tkt-001-ticket-thread.png)

## Problems Encountered

<!-- Be specific and honest. Genuine dead ends read better than a clean narrative. Candidates:
- TKT-003: assumed share permissions were the cause; they were correct. Actual cause was NTFS inheritance broken at the folder level.
- TKT-005: initial assumption was a credential problem on the SMB mount. Storage account key was valid; the failure was network-layer.
- Freshdesk free tier limits on automations/workflows and how you worked within them.
- Anything that went sideways in the VM or portal.
-->

## Solution

<!-- What the finished system does and what the data showed. Reference the metrics rather than restating them. -->

Full breakdown: [docs/metrics-summary.md](docs/metrics-summary.md)

| Metric | Result |
|--------|--------|
| Tickets handled | 10 |
| Resolved against live infrastructure | 6 |
| Average first response time | _TBD_ |
| Average time to resolution | _TBD_ |
| SLA attainment | _TBD_ |
| Highest-volume category | _TBD_ |
| Top deflection candidate | _TBD_ |

## Skills Demonstrated

<!-- Trim to what you can actually defend in an interview. Delete anything you'd stumble on. -->

**Active Directory administration** — Account lockout diagnosis and remediation, password resets with forced change at next logon, account status and expiry troubleshooting, NTFS and share permission analysis using effective access.

**Azure administration** — Azure Files share creation and SMB mounting, storage account network rule configuration and troubleshooting, Blob Storage container management, SAS token generation and expiry handling.

**PowerShell** — Directory queries and remediation using `Search-ADAccount`, `Unlock-ADAccount`, `Set-ADAccountPassword`, and `Get-ADUser` with property filtering.

**Service desk operations** — Ticket triage and prioritization against defined criteria, SLA policy design and measurement, tier 1 to tier 2 escalation criteria, knowledge base authoring for ticket deflection, queue metrics analysis.

**Technical documentation** — Structured incident write-ups with reproducible diagnosis paths, root cause analysis in Cause / Fix / Prevention format, end-user facing knowledge base articles.

---

## Repository Structure

```
helpdesk-ticketing-lab/
├── README.md
├── docs/
│   ├── sla-policy.md
│   ├── escalation-criteria.md
│   └── metrics-summary.md
├── tickets/          # One file per ticket, TKT-001 through TKT-010
├── kb-articles/      # Four end-user knowledge base articles
└── screenshots/
    ├── freshdesk/
    ├── ad/
    └── azure/
```
