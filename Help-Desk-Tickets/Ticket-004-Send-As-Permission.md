# INC-004 — Send As vs Send on Behalf Permissions

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-004 |
| Requesters | Brian Carter / Emily Brown |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Shared Mailbox Delegation |
| Impact | Individual Users |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

Users require different sending permissions for the Company Shared mailbox.

---

# Incident Summary

The Company Shared mailbox was configured to demonstrate and troubleshoot the difference between two Exchange Online sending permissions:

- Send As
- Send on Behalf

Brian Carter required permission to send messages directly as Company Shared.

Emily Brown required permission to send messages on behalf of Company Shared.

These permissions were configured separately and verified through Exchange Admin Center, Exchange Online PowerShell, and actual message delivery tests.

---

# Environment

**Shared Mailbox:** Company Shared  
**Email Address:** companyshared@Stefon.onmicrosoft.com

**Send As User:** Brian Carter  
**Email:** brian.carter@Stefon.onmicrosoft.com

**Send on Behalf User:** Emily Brown  
**Email:** emily.brown@Stefon.onmicrosoft.com

Administrative tools used:

- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web

---

# Permission Definitions

## Full Access

Full Access allows a delegate to open and manage mailbox contents.

It does not automatically provide permission to send messages using the mailbox identity.

---

## Send As

Send As allows a delegate to send a message that appears to originate directly from the shared mailbox.

The recipient sees:

`Company Shared`

The delegate's personal identity is not displayed as the sender.

---

## Send on Behalf

Send on Behalf allows a delegate to send using the shared mailbox while preserving the identity of the delegate.

The recipient sees:

`Emily Brown on behalf of Company Shared`

---

# Baseline Investigation — Send As

Exchange Admin Center was reviewed under:

`Recipients > Mailboxes > Company Shared > Mailbox delegation`

Brian Carter was not initially listed under Send As.

Exchange Online PowerShell was then used to verify the permission state.

The following command was executed:

`Get-RecipientPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.Trustee -like "*brian*"} | Format-Table Trustee,AccessRights,IsInherited -AutoSize`

No result was returned for Brian Carter.

This confirmed that Brian did not have Send As permission.

---

# Baseline Investigation — Send on Behalf

The Send on Behalf delegation list was reviewed.

Emily Brown was not initially listed.

PowerShell was also used to inspect the mailbox property:

`Get-Mailbox -Identity "companyshared@Stefon.onmicrosoft.com" | Format-List DisplayName,GrantSendOnBehalfTo`

Emily Brown was not present in the initial configuration.

---

# Root Cause

The users did not have the required Exchange Online sending permissions.

Brian Carter required:

`Send As`

Emily Brown required:

`Send on Behalf`

Full Access and sending permissions are configured independently in Exchange Online.

---

# Resolution — Brian Carter Send As

Exchange Admin Center was used to update:

`Company Shared > Mailbox delegation > Send As`

Brian Carter was added as a Send As delegate.

The change was saved successfully.

---

# PowerShell Verification — Brian Carter

The following command was executed:

`Get-RecipientPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.Trustee -like "*brian*"} | Format-Table Trustee,AccessRights,IsInherited -AutoSize`

The result returned:

`brian.carter@Stefon.onmicrosoft.com`

with:

`AccessRights : {SendAs}`

and:

`IsInherited : False`

This confirmed that Brian Carter had an explicit Send As permission assignment.

---

# Send As Message Test

Brian Carter signed into Outlook on the web.

A test message was sent using:

`companyshared@Stefon.onmicrosoft.com`

as the From address.

The test message was delivered successfully.

The recipient viewed the message and the sender appeared as:

`Company Shared`

Brian Carter's personal identity was not displayed as the sender.

This confirmed that Send As behaved as expected.

---

# Resolution — Emily Brown Send on Behalf

Exchange Admin Center was used to update:

`Company Shared > Mailbox delegation > Send on Behalf`

Emily Brown was added as a delegate.

The configuration was saved successfully.

---

# PowerShell Verification — Emily Brown

The mailbox's `GrantSendOnBehalfTo` property was reviewed.

The delegate value was initially represented internally by Exchange as an object identifier.

The following PowerShell commands were used to resolve the delegate to a readable recipient:

`$Delegates = (Get-Mailbox -Identity "companyshared@Stefon.onmicrosoft.com").GrantSendOnBehalfTo`

`$Delegates | ForEach-Object { Get-Recipient -Identity $_ } | Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize`

The result returned:

- DisplayName: Emily Brown
- PrimarySmtpAddress: emily.brown@Stefon.onmicrosoft.com
- RecipientTypeDetails: UserMailbox

This confirmed that Emily Brown was the configured Send on Behalf delegate.

---

# Send on Behalf Message Test

Emily Brown signed into Outlook on the web.

A test message was sent using Company Shared as the From mailbox.

The message was successfully delivered.

The recipient saw the sender displayed as:

`Emily Brown on behalf of Company Shared`

This confirmed that Send on Behalf behaved differently from Send As.

---

# Permission Comparison

| Permission | User | Result |
|---|---|---|
| Full Access | Brian Carter | Can open and manage Company Shared |
| Send As | Brian Carter | Recipient sees Company Shared as sender |
| Send on Behalf | Emily Brown | Recipient sees Emily Brown on behalf of Company Shared |

---

# Resolution Verification

The incident was considered resolved after confirming:

- Brian Carter was configured with Send As.
- PowerShell returned `{SendAs}` for Brian.
- Brian successfully sent an email as Company Shared.
- The recipient saw Company Shared as the sender.
- Emily Brown was configured with Send on Behalf.
- PowerShell resolved Emily under `GrantSendOnBehalfTo`.
- Emily successfully sent a message using Company Shared.
- The recipient saw `Emily Brown on behalf of Company Shared`.

---

# Key Troubleshooting Lesson

Full Access, Send As, and Send on Behalf are separate Exchange permissions.

They should not be treated as interchangeable.

## Full Access

Controls mailbox content access.

## Send As

Controls whether a user can impersonate the mailbox identity when sending.

## Send on Behalf

Allows the user to send using the mailbox while visibly identifying the delegate.

Exchange administrators should assign only the permission required for the business need.

---

# Closure Notes

Brian Carter and Emily Brown required different sending capabilities for the Company Shared mailbox.

Brian Carter was granted Send As permission.

Emily Brown was granted Send on Behalf permission.

Both configurations were verified through Exchange Online PowerShell.

Actual Outlook message tests confirmed the behavioral difference between the two permission types.

No further action required.

---

# Commands Used

## Check Brian Send As

`Get-RecipientPermission -Identity "companyshared@Stefon.onmicrosoft.com" | Where-Object {$_.Trustee -like "*brian*"} | Format-Table Trustee,AccessRights,IsInherited -AutoSize`

## Review Send on Behalf

`Get-Mailbox -Identity "companyshared@Stefon.onmicrosoft.com" | Format-List DisplayName,GrantSendOnBehalfTo`

## Resolve Send on Behalf Delegate

`$Delegates = (Get-Mailbox -Identity "companyshared@Stefon.onmicrosoft.com").GrantSendOnBehalfTo`

`$Delegates | ForEach-Object { Get-Recipient -Identity $_ } | Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/03-Shared-Mailboxes/`

Evidence includes:

- `06-Company-Shared-Send-As-Baseline.png`
- `07-Brian-No-Send-As-EAC.png`
- `08-Brian-No-Send-As-PowerShell.png`
- `09-Emily-No-Send-On-Behalf-PowerShell.png`
- `10-Brian-Send-As-Assigned.png`
- `11-Emily-No-Send-On-Behalf-EAC.png`
- `12-Emily-Send-On-Behalf-Assigned.png`
- `13-Brian-Send-As-PowerShell-Verification.png`
- `14-Emily-Send-On-Behalf-PowerShell-Verification.png`
- `15-Brian-Send-As-Message-Verification.png`
- `16-Emily-Send-On-Behalf-Message-Verification.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Shared Mailbox Administration
- Mailbox Delegation
- Full Access
- Send As
- Send on Behalf
- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web
- Permission Troubleshooting
- End-to-End Service Testing
- Root Cause Analysis
- Incident Management
- ServiceNow-Style Documentation
- Resolution Verification

---

# Final Status

**Resolved**

Brian Carter was successfully configured with Send As permission for Company Shared.

Emily Brown was successfully configured with Send on Behalf permission.

Exchange Online PowerShell verified both configurations, and live Outlook message testing demonstrated the expected difference in sender identity.
