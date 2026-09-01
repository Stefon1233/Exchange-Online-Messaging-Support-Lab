# INC-006 — Unintended Mailbox Forwarding

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-006 |
| Requester | Robert Garcia |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Mail Flow / Forwarding |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User reports that new email messages are not appearing in the mailbox because Exchange Online forwarding is redirecting messages to another user.

---

# User-Reported Issue

Robert Garcia reports that expected email messages are not appearing in his Exchange Online mailbox.

The account has a valid Exchange Online mailbox and can access Outlook normally.

Investigation is required to determine why newly delivered mail is not remaining in Robert's Inbox.

---

# Environment

**Affected User:** Robert Garcia  
**Email Address:** robert.garcia@Stefon.onmicrosoft.com

**Unexpected Forwarding Destination:** Brian Carter  
**Forwarding Address:** brian.carter@Stefon.onmicrosoft.com

Administrative tools used:

- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web

---

# Initial Forwarding Baseline

Robert Garcia's mailbox was reviewed through Exchange Admin Center.

Navigation:

`Exchange Admin Center > Recipients > Mailboxes > Robert Garcia > Email forwarding`

The initial configuration showed:

`Forward all emails sent to this mailbox: Off`

This established a known-good forwarding baseline.

---

# PowerShell Baseline Verification

Exchange Online PowerShell was used to verify Robert Garcia's forwarding configuration.

The following command was executed:

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

The result showed:

- DisplayName: Robert Garcia
- PrimarySmtpAddress: robert.garcia@Stefon.onmicrosoft.com
- ForwardingAddress: Blank
- ForwardingSmtpAddress: Blank
- DeliverToMailboxAndForward: False

This confirmed that mailbox-level forwarding was not configured at baseline.

---

# Fault Simulation

Mailbox forwarding was enabled for Robert Garcia.

The forwarding destination was configured as:

`Brian Carter`

The following option was intentionally left disabled:

`Deliver message to both forwarding address and mailbox`

This created a configuration where messages sent to Robert were redirected to Brian without retaining a copy in Robert's mailbox.

---

# Exchange Admin Center Diagnosis

Exchange Admin Center showed:

`Forward all emails sent to this mailbox: On`

The configured internal forwarding destination was:

`Brian Carter`

The option:

`Deliver message to both forwarding address and mailbox`

was not selected.

This indicated that incoming messages would be redirected rather than delivered to both mailboxes.

---

# PowerShell Forwarding Diagnosis

The mailbox configuration was queried again using:

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

The result showed:

- DisplayName: Robert Garcia
- ForwardingAddress: Configured
- DeliverToMailboxAndForward: False

The forwarding address was represented internally by Exchange Online as an object identifier.

---

# Forwarding Destination Resolution

PowerShell was used to resolve the internal Exchange object to a readable recipient.

The following commands were executed:

`$ForwardTarget = (Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com").ForwardingAddress`

`Get-Recipient -Identity $ForwardTarget | Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize`

The result returned:

- DisplayName: Brian Carter
- PrimarySmtpAddress: brian.carter@Stefon.onmicrosoft.com
- RecipientTypeDetails: UserMailbox

This confirmed that Robert Garcia's mailbox was forwarding incoming messages to Brian Carter.

---

# Mail-Flow Test During Failure

A test message was sent from Emily Brown directly to:

`robert.garcia@Stefon.onmicrosoft.com`

The message used the subject:

`LAB TEST - Mail Forwarding Investigation`

The message body stated:

`This message validates Exchange Online mailbox forwarding behavior for the messaging support lab.`

---

# Forwarded Message Verification

Brian Carter signed into Outlook on the web.

The test message appeared successfully in Brian Carter's Inbox.

The message showed:

- Sender: Emily Brown
- Original Recipient: Robert Garcia
- Subject: LAB TEST - Mail Forwarding Investigation

This confirmed that Exchange Online redirected the message from Robert's mailbox to Brian.

---

# Robert Mailbox Verification

Robert Garcia signed into Outlook on the web.

A search was performed for the exact subject:

`LAB TEST - Mail Forwarding Investigation`

Outlook returned:

`We didn't find anything.`

This confirmed that Robert did not retain a copy of the message.

The result matched the Exchange configuration:

`DeliverToMailboxAndForward : False`

---

# Root Cause

Robert Garcia's Exchange Online mailbox had unintended forwarding configured to Brian Carter.

Because:

`DeliverToMailboxAndForward`

was set to:

`False`

messages were forwarded to Brian without also being retained in Robert's mailbox.

The issue was caused by mailbox-level forwarding rather than:

- Missing Exchange Online license
- Mailbox provisioning
- Distribution list membership
- Shared mailbox permissions
- Outlook client configuration
- Authentication failure

---

# Resolution

Exchange Admin Center was used to remove the unintended forwarding configuration.

Navigation:

`Exchange Admin Center > Recipients > Mailboxes > Robert Garcia > Email forwarding`

The setting:

`Forward all emails sent to this mailbox`

was changed to:

`Off`

The configuration was saved.

---

# PowerShell Resolution Verification

The mailbox was queried again using:

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

The resulting configuration showed:

- DisplayName: Robert Garcia
- ForwardingAddress: Blank
- ForwardingSmtpAddress: Blank
- DeliverToMailboxAndForward: False

The blank forwarding fields confirmed that mailbox-level forwarding had been removed.

---

# End-to-End Resolution Test

A new message was sent from Emily Brown directly to Robert Garcia.

The message used the subject:

`LAB TEST - Mail Forwarding Resolved`

The message body stated:

`This message validates normal mailbox delivery after unintended forwarding was removed.`

Robert Garcia signed into Outlook on the web.

The test message successfully appeared in Robert's Inbox.

This confirmed that normal mailbox delivery had been restored.

---

# Resolution Verification

The incident was considered resolved after confirming:

- Forwarding was disabled in Exchange Admin Center.
- `ForwardingAddress` was blank in PowerShell.
- `ForwardingSmtpAddress` was blank in PowerShell.
- A new email addressed to Robert remained in Robert's mailbox.
- Normal Exchange Online delivery was restored.

---

# Troubleshooting Workflow

1. Verify the affected user's mailbox exists.
2. Review mailbox forwarding in Exchange Admin Center.
3. Establish the original forwarding configuration.
4. Verify forwarding properties using PowerShell.
5. Identify an unexpected forwarding destination.
6. Review `DeliverToMailboxAndForward`.
7. Resolve the forwarding target to a readable recipient.
8. Send a controlled test message.
9. Confirm the message arrives at the forwarding destination.
10. Search the original mailbox for the test message.
11. Confirm the original mailbox did not retain a copy.
12. Identify unintended forwarding as the root cause.
13. Disable mailbox forwarding.
14. Verify forwarding fields are blank through PowerShell.
15. Send another test message.
16. Confirm normal delivery to the affected mailbox.
17. Document the incident and resolution.

---

# Key Troubleshooting Lesson

Mailbox forwarding should be checked when users report that messages appear to be disappearing from their mailbox.

Administrators should review:

- ForwardingAddress
- ForwardingSmtpAddress
- DeliverToMailboxAndForward
- Inbox rules
- Mail flow rules
- Message trace
- Mailbox permissions

The value of:

`DeliverToMailboxAndForward`

is particularly important.

When forwarding is configured and this value is `False`, messages may be redirected without leaving a copy in the original mailbox.

When the value is `True`, Exchange can deliver the message to both the forwarding destination and the original mailbox.

---

# Commands Used

## Review Mailbox Forwarding

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

## Resolve Forwarding Target

`$ForwardTarget = (Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com").ForwardingAddress`

`Get-Recipient -Identity $ForwardTarget | Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize`

## Verify Forwarding Removal

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/05-Mail-Flow/`

Evidence includes:

- `01-Robert-Garcia-Forwarding-Baseline.png`
- `02-Robert-No-Forwarding-PowerShell.png`
- `03-Robert-Forwarding-To-Brian-Enabled.png`
- `04-Robert-Forwarding-PowerShell-Diagnosis.png`
- `05-Forwarded-Message-Received-By-Brian.png`
- `06-Robert-Forwarded-Message-Not-Received.png`
- `07-Robert-Forwarding-Target-PowerShell.png`
- `08-Robert-Forwarding-Disabled.png`
- `09-Robert-Forwarding-Removed-PowerShell.png`
- `10-Robert-Normal-Mail-Delivery-Restored.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Mailbox Forwarding
- Mail Flow Troubleshooting
- Exchange Admin Center
- Exchange Online PowerShell
- Get-Mailbox
- Get-Recipient
- Recipient Resolution
- Outlook on the web
- End-to-End Mail Testing
- Root Cause Analysis
- Service Restoration
- Incident Management
- ServiceNow-Style Documentation
- Resolution Verification

---

# Final Status

**Resolved**

Robert Garcia's missing email issue was caused by unintended Exchange Online mailbox forwarding to Brian Carter.

Because `DeliverToMailboxAndForward` was set to `False`, forwarded messages were not retained in Robert's mailbox.

The forwarding configuration was removed.

PowerShell confirmed that the forwarding fields were blank, and a live Outlook test verified that normal mail delivery to Robert Garcia was restored.
