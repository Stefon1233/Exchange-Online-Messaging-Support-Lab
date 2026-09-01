# INC-003 — Shared Mailbox Access

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-003 |
| Requester | Brian Carter |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Shared Mailbox |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User requires access to the Company Shared mailbox but does not have Full Access permission.

---

# User-Reported Issue

Brian Carter requires access to the Company Shared mailbox.

The shared mailbox exists and is operational, but Brian is unable to access the mailbox because the required mailbox delegation permission has not been assigned.

---

# Environment

**Shared Mailbox:** Company Shared  
**Email Address:** companyshared@Stefon.onmicrosoft.com  
**Affected User:** Brian Carter  
**User Email:** brian.carter@Stefon.onmicrosoft.com

Administrative tools used:

- Exchange Admin Center
- Exchange Online PowerShell
- ExchangeOnlineManagement PowerShell Module

---

# Initial Investigation

Exchange Admin Center was opened and the following location was reviewed:

`Recipients > Mailboxes`

The Company Shared mailbox was located successfully.

The mailbox displayed:

- Display Name: Company Shared
- Email Address: companyshared@Stefon.onmicrosoft.com
- Recipient Type: SharedMailbox

This confirmed that the shared mailbox itself existed and was operational.

---

# Mailbox Delegation Investigation

Mailbox delegation settings were reviewed for Company Shared.

The Full Access permission list contained an existing administrator account.

Brian Carter was not listed.

This indicated that the issue was not caused by a missing shared mailbox.

The user simply had not been granted the permission required to open and manage the mailbox.

---

# PowerShell Baseline Verification

Exchange Online PowerShell was used to review the current mailbox permissions.

The following command was executed:

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -notlike "NT AUTHORITY*"} | Select-Object User,AccessRights,IsInherited,Deny | Format-Table -AutoSize`

The existing explicit Full Access permissions were displayed.

A second command was used to specifically search for Brian Carter:

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

No result was returned for Brian Carter.

This confirmed through PowerShell that Brian did not have Full Access permission.

---

# Root Cause

Brian Carter had not been assigned Full Access permission to the Company Shared mailbox.

The shared mailbox was functioning normally.

The incident was caused by missing mailbox delegation rather than:

- Licensing
- Mailbox provisioning
- Authentication
- Outlook configuration
- Mail flow

---

# Resolution

Exchange Admin Center was used to update mailbox delegation.

Navigation:

`Exchange Admin Center > Recipients > Mailboxes > Company Shared > Mailbox delegation`

Under:

`Full Access`

Brian Carter was added as a delegate.

The configuration was saved successfully.

---

# Exchange Admin Center Verification

After the change was applied, the Full Access delegation list displayed:

- Brian Carter
- Existing administrator

This provided graphical confirmation that Brian Carter had been granted access.

---

# PowerShell Verification

The permission was independently verified with Exchange Online PowerShell.

The following command was executed:

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

The command returned:

`brian.carter@Stefon.onmicrosoft.com`

with:

`AccessRights : {FullAccess}`

The permission also showed:

`IsInherited : False`

`Deny : False`

This confirmed that Brian Carter had an explicit Full Access permission assignment to the shared mailbox.

---

# Resolution Verification

The incident was considered resolved after confirming:

- Company Shared remained a valid SharedMailbox.
- Brian Carter was added under Full Access delegation.
- Exchange Admin Center displayed Brian as a delegate.
- Exchange Online PowerShell returned Brian's permission.
- AccessRights showed `{FullAccess}`.
- The permission was explicitly assigned rather than inherited.
- Deny was set to False.

---

# Important Permission Distinction

Full Access allows a delegate to open and manage the contents of a mailbox.

Full Access does not automatically provide the ability to send email as the shared mailbox.

Sending permissions are controlled separately through:

- Send As
- Send on Behalf

Those permissions are tested separately in the next messaging support scenario.

---

# Closure Notes

Brian Carter required access to the Company Shared mailbox.

Investigation confirmed that the shared mailbox existed and was operational, but Brian did not have Full Access permission.

Brian Carter was added to the mailbox Full Access delegation list.

Exchange Admin Center and Exchange Online PowerShell independently confirmed that the new permission was applied successfully.

No further action required for mailbox access.

---

# Commands Used

## Review Explicit Mailbox Permissions

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -notlike "NT AUTHORITY*"} | Select-Object User,AccessRights,IsInherited,Deny | Format-Table -AutoSize`

## Check Brian Carter Permission

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/03-Shared-Mailboxes/`

Evidence includes:

- `01-Company-Shared-Mailbox-Baseline.png`
- `02-Company-Shared-Brian-No-Full-Access.png`
- `03-Brian-No-Full-Access-PowerShell.png`
- `04-Brian-Full-Access-Assigned.png`
- `05-Brian-Full-Access-PowerShell-Verification.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Shared Mailbox Administration
- Mailbox Delegation
- Full Access Permissions
- Exchange Admin Center
- Exchange Online PowerShell
- Permission Troubleshooting
- Root Cause Analysis
- Incident Management
- Resolution Verification
- ServiceNow-Style Documentation

---

# Final Status

**Resolved**

Brian Carter could not access the Company Shared mailbox because Full Access permission had not been assigned.

Brian was added as a Full Access delegate.

Exchange Admin Center and Exchange Online PowerShell confirmed that the explicit `{FullAccess}` permission was successfully applied.
