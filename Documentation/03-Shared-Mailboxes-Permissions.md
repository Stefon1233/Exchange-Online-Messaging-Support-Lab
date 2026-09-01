# 03 — Shared Mailboxes and Permissions

## Overview

This section documents Exchange Online shared mailbox administration and mailbox delegation troubleshooting.

Shared mailboxes are commonly used for departmental or team-based communication where multiple users need access to a common mailbox.

This scenario focuses on troubleshooting Full Access permission for a shared mailbox and verifying the final permission state through both Exchange Admin Center and Exchange Online PowerShell.

---

# Scenario 3 — Shared Mailbox Access

## Incident Summary

**Shared Mailbox:** Company Shared  
**Email Address:** companyshared@Stefon.onmicrosoft.com  
**Affected User:** Brian Carter  
**Issue:** User required access to the shared mailbox but did not have Full Access permission.

The incident was investigated using:

- Exchange Admin Center
- Exchange Online PowerShell

---

# Shared Mailbox Baseline

Exchange Admin Center was opened and the following location was reviewed:

`Recipients > Mailboxes`

The Company Shared mailbox was located successfully.

The mailbox displayed:

- Display Name: Company Shared
- Email Address: companyshared@Stefon.onmicrosoft.com
- Recipient Type: SharedMailbox

This confirmed that the shared mailbox itself existed and was functioning as an Exchange Online shared mailbox.

---

# Initial Delegation Investigation

Mailbox delegation settings were reviewed for Company Shared.

The Full Access permission list contained an existing administrator account.

Brian Carter was not listed.

This established the initial failure state.

The issue was therefore not caused by:

- Missing shared mailbox
- Incorrect mailbox type
- Missing Exchange environment
- Mailbox provisioning failure

The likely cause was missing mailbox delegation.

---

# Understanding Full Access

Full Access permission allows a delegate to open a mailbox and manage mailbox contents.

A user with Full Access can typically:

- Open the shared mailbox
- Read messages
- Manage folders
- Move messages
- Delete messages
- Work with mailbox content

Full Access does not automatically grant permission to send messages using the shared mailbox identity.

Sending permissions are controlled separately through:

- Send As
- Send on Behalf

These permissions are investigated in a separate scenario.

---

# PowerShell Baseline Verification

Exchange Online PowerShell was used to review the existing mailbox permissions.

The following command displayed explicit mailbox permissions while excluding built-in NT AUTHORITY entries:

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -notlike "NT AUTHORITY*"} | Select-Object User,AccessRights,IsInherited,Deny | Format-Table -AutoSize`

The output showed an existing administrator with:

`{FullAccess}`

Brian Carter did not appear.

A more specific query was then performed:

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

No result was returned.

This confirmed that Brian Carter did not have explicit Full Access permission.

---

# Root Cause

Brian Carter had not been assigned Full Access permission to the Company Shared mailbox.

The mailbox existed and was functioning normally.

The incident was caused by missing delegation rather than a mailbox or licensing failure.

---

# Remediation

Exchange Admin Center was used to update the shared mailbox permissions.

Navigation:

`Exchange Admin Center > Recipients > Mailboxes > Company Shared > Mailbox delegation`

Under:

`Full Access`

Brian Carter was added as a delegate.

The change was saved successfully.

---

# Exchange Admin Center Verification

After the permission change was applied, the Full Access delegation list displayed:

- Brian Carter
- Existing administrator

This provided graphical confirmation that Brian Carter had been added successfully.

---

# PowerShell Verification

Exchange Online PowerShell was used to independently verify Brian Carter's permission.

The following command was executed:

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

The output returned:

`brian.carter@Stefon.onmicrosoft.com`

with:

`AccessRights : {FullAccess}`

The permission also showed:

`IsInherited : False`

`Deny : False`

This confirmed that Brian Carter had an explicit Full Access assignment and that the permission was not blocked.

---

# Permission Interpretation

## AccessRights

`{FullAccess}`

This confirms that Brian Carter can open and manage the shared mailbox.

## IsInherited

`False`

This confirms that the permission was explicitly assigned rather than inherited from another permission source.

## Deny

`False`

This confirms that the access permission is not explicitly denied.

---

# Troubleshooting Workflow

The troubleshooting process used for this shared mailbox incident was:

1. Identify the affected shared mailbox.
2. Verify the mailbox exists.
3. Confirm the mailbox recipient type is `SharedMailbox`.
4. Review mailbox delegation.
5. Verify whether the affected user has Full Access.
6. Connect to Exchange Online PowerShell.
7. Query explicit mailbox permissions.
8. Search specifically for the affected user.
9. Confirm no Full Access permission exists.
10. Identify missing delegation as the root cause.
11. Add the user under Full Access.
12. Save the Exchange configuration.
13. Verify the user appears in Exchange Admin Center.
14. Query the permission again with PowerShell.
15. Confirm `{FullAccess}`.
16. Verify `IsInherited` is False.
17. Verify `Deny` is False.
18. Document the incident and resolution.

---

# Key Troubleshooting Lesson

Shared mailbox access should be investigated separately from mailbox sending permissions.

A user may be able to open a shared mailbox but still be unable to send from it.

Likewise, a user may have a sending permission but still lack Full Access to manage the mailbox contents.

Exchange administrators should identify which permission is required before making changes.

The primary delegation permissions are:

## Full Access

Allows the delegate to open and manage the mailbox.

## Send As

Allows the delegate to send a message that appears to come directly from the shared mailbox.

## Send on Behalf

Allows the delegate to send a message where the sender is displayed as the user acting on behalf of the shared mailbox.

These permissions should be configured according to the user's business requirement rather than granting all permissions automatically.

---

# Administrative Tools Used

- Exchange Admin Center
- Exchange Online
- Exchange Online PowerShell
- PowerShell 7
- ExchangeOnlineManagement PowerShell Module

---

# Commands Used

## Review Explicit Shared Mailbox Permissions

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -notlike "NT AUTHORITY*"} | Select-Object User,AccessRights,IsInherited,Deny | Format-Table -AutoSize`

## Check Brian Carter Before Permission Assignment

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

## Verify Brian Carter After Permission Assignment

`Get-MailboxPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.User -like "*brian*"} | Format-Table User,AccessRights,IsInherited,Deny -AutoSize`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/03-Shared-Mailboxes/`

Scenario 3 evidence includes:

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
- Resolution Verification
- Incident Management
- Technical Documentation

---

# Scenario Outcome

**Status: Resolved**

Brian Carter could not access the Company Shared mailbox because Full Access permission had not been assigned.

Brian Carter was added to the Full Access delegation list.

Exchange Admin Center confirmed the assignment, and Exchange Online PowerShell independently verified:

- `AccessRights : {FullAccess}`
- `IsInherited : False`
- `Deny : False`

The shared mailbox access issue was successfully resolved.

---

# Next Scenario

The next support scenario will investigate sending permissions for the same shared mailbox.

The lab will demonstrate the difference between:

- Send As
- Send on Behalf

Brian Carter's existing Full Access permission will remain in place while sending permissions are tested separately.
