# 05 — Mail Forwarding and Mail Flow Troubleshooting

## Overview

This section documents Exchange Online mailbox forwarding and mail flow troubleshooting.

Mailbox forwarding can be useful for legitimate business workflows, but an incorrect or unexpected forwarding configuration can cause users to believe messages are missing from their mailbox.

This scenario demonstrates how to:

- Review mailbox forwarding in Exchange Admin Center
- Verify forwarding properties with Exchange Online PowerShell
- Resolve an internal forwarding target
- Reproduce a mail-delivery problem
- Confirm the effect of `DeliverToMailboxAndForward`
- Remove unintended forwarding
- Validate restored message delivery

---

# Scenario 6 — Unintended Mailbox Forwarding

## Incident Summary

**Affected User:** Robert Garcia  
**Email Address:** robert.garcia@Stefon.onmicrosoft.com  
**Unexpected Forwarding Destination:** Brian Carter  
**Forwarding Address:** brian.carter@Stefon.onmicrosoft.com

**Issue:** Messages addressed to Robert Garcia were being redirected to another mailbox and were not remaining in Robert's Inbox.

The incident was investigated using:

- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web

---

# Known-Good Baseline

Robert Garcia's mailbox was reviewed through Exchange Admin Center.

Navigation:

`Recipients > Mailboxes > Robert Garcia > Email forwarding`

The forwarding configuration initially showed:

`Forward all emails sent to this mailbox: Off`

This established a known-good baseline before the forwarding issue was introduced.

---

# PowerShell Baseline Verification

Exchange Online PowerShell was used to inspect the mailbox forwarding properties.

The following command was executed:

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

The initial result showed:

- DisplayName: Robert Garcia
- PrimarySmtpAddress: robert.garcia@Stefon.onmicrosoft.com
- ForwardingAddress: Blank
- ForwardingSmtpAddress: Blank
- DeliverToMailboxAndForward: False

Because both forwarding address fields were blank, no mailbox-level forwarding was active.

---

# Forwarding Fault Simulation

Forwarding was enabled through Exchange Admin Center.

The following configuration was applied:

- Forward all emails sent to this mailbox: On
- Forwarding destination: Brian Carter
- Deliver message to both forwarding address and mailbox: Disabled

This created the following delivery behavior:

`Message sent to Robert Garcia`

↓

`Exchange Online forwards message to Brian Carter`

↓

`Robert Garcia does not retain a mailbox copy`

---

# Exchange Admin Center Investigation

Robert Garcia's Email forwarding configuration showed that forwarding was active.

The internal destination was:

`Brian Carter`

The setting:

`Deliver message to both forwarding address and mailbox`

was not selected.

This indicated that incoming messages would be redirected to Brian instead of being delivered to both users.

---

# PowerShell Diagnosis

The forwarding configuration was verified through PowerShell.

The following command was executed:

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

The output showed:

- DisplayName: Robert Garcia
- ForwardingAddress: Configured
- ForwardingSmtpAddress: Blank
- DeliverToMailboxAndForward: False

The internal forwarding destination appeared as an Exchange object identifier rather than a readable display name.

---

# Resolving the Forwarding Destination

The forwarding object was resolved using Exchange Online PowerShell.

The following command stored the forwarding destination:

`$ForwardTarget = (Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com").ForwardingAddress`

The target was then resolved with:

`Get-Recipient -Identity $ForwardTarget | Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize`

The result returned:

- DisplayName: Brian Carter
- PrimarySmtpAddress: brian.carter@Stefon.onmicrosoft.com
- RecipientTypeDetails: UserMailbox

This confirmed that Robert Garcia's mailbox was forwarding messages to Brian Carter.

---

# Understanding ForwardingAddress and ForwardingSmtpAddress

Exchange Online can expose forwarding configuration through two different properties.

## ForwardingAddress

`ForwardingAddress` references another recipient object within the Exchange organization.

In this scenario, Robert's internal forwarding destination was Brian Carter.

## ForwardingSmtpAddress

`ForwardingSmtpAddress` can contain an SMTP forwarding address.

In this scenario, this field remained blank because the forwarding destination was configured as an internal Exchange recipient.

---

# Understanding DeliverToMailboxAndForward

The property:

`DeliverToMailboxAndForward`

determines whether Exchange retains a message in the original mailbox while also forwarding it.

In this scenario the value was:

`False`

This meant that when forwarding was active:

- Brian Carter received the forwarded message.
- Robert Garcia did not retain a copy.

This property was central to explaining the user's symptoms.

---

# Controlled Mail-Flow Test

A controlled test message was sent from Emily Brown directly to:

`robert.garcia@Stefon.onmicrosoft.com`

The subject was:

`LAB TEST - Mail Forwarding Investigation`

The body stated:

`This message validates Exchange Online mailbox forwarding behavior for the messaging support lab.`

---

# Forwarding Destination Verification

Brian Carter signed into Outlook on the web.

The test message appeared successfully in Brian's Inbox.

The message showed:

- Sender: Emily Brown
- Recipient: Robert Garcia
- Subject: LAB TEST - Mail Forwarding Investigation

This demonstrated that Exchange accepted a message addressed to Robert and redirected it to Brian.

---

# Original Mailbox Verification

Robert Garcia signed into Outlook on the web.

The exact subject was searched:

`LAB TEST - Mail Forwarding Investigation`

Outlook returned:

`We didn't find anything.`

This confirmed that the message was not retained in Robert's mailbox.

The observed result matched:

`DeliverToMailboxAndForward : False`

---

# Root Cause

The root cause was an unintended mailbox-level forwarding configuration.

Robert Garcia's incoming email was being forwarded internally to Brian Carter.

Because:

`DeliverToMailboxAndForward`

was:

`False`

Exchange Online redirected messages without preserving a copy in Robert's mailbox.

---

# Remediation

Exchange Admin Center was used to remove the forwarding configuration.

Navigation:

`Exchange Admin Center > Recipients > Mailboxes > Robert Garcia > Email forwarding`

The setting:

`Forward all emails sent to this mailbox`

was changed to:

`Off`

The configuration was saved.

---

# PowerShell Resolution Verification

After forwarding was disabled, the mailbox properties were queried again:

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

The output showed:

- DisplayName: Robert Garcia
- ForwardingAddress: Blank
- ForwardingSmtpAddress: Blank
- DeliverToMailboxAndForward: False

The blank forwarding fields confirmed that mailbox-level forwarding had been removed.

---

# End-to-End Recovery Test

A second controlled message was sent from Emily Brown directly to Robert Garcia.

The subject was:

`LAB TEST - Mail Forwarding Resolved`

The body stated:

`This message validates normal mailbox delivery after unintended forwarding was removed.`

Robert Garcia signed into Outlook on the web.

The message appeared successfully in Robert's Inbox.

This confirmed that normal Exchange Online delivery was restored.

---

# Troubleshooting Workflow

The troubleshooting process used during this scenario was:

1. Verify that the affected mailbox exists.
2. Review forwarding through Exchange Admin Center.
3. Establish the known-good baseline.
4. Query forwarding properties through PowerShell.
5. Enable a controlled forwarding condition.
6. Configure an internal forwarding recipient.
7. Leave local mailbox delivery disabled.
8. Query the new forwarding state.
9. Review `DeliverToMailboxAndForward`.
10. Resolve the internal forwarding target using `Get-Recipient`.
11. Send a controlled test message.
12. Confirm the forwarding destination receives the message.
13. Search the original mailbox.
14. Confirm the original mailbox does not contain the message.
15. Identify mailbox forwarding as the root cause.
16. Disable forwarding.
17. Verify forwarding properties are blank.
18. Send a second controlled message.
19. Confirm normal delivery to the original mailbox.
20. Document the incident and resolution.

---

# Key Troubleshooting Lesson

Mailbox forwarding should be reviewed when a user reports that messages are unexpectedly missing or appearing in another mailbox.

Administrators should inspect:

- ForwardingAddress
- ForwardingSmtpAddress
- DeliverToMailboxAndForward
- Inbox rules
- Mail flow rules
- Message trace
- Recipient configuration

A forwarding destination should also be resolved to a readable user or address when PowerShell displays an internal object identifier.

---

# Security Consideration

Unexpected forwarding can also represent a security concern.

If an administrator discovers forwarding that was not intentionally configured, additional investigation may be appropriate.

Potential areas to review include:

- User account activity
- Authentication history
- Inbox rules
- Administrative changes
- Mail flow rules
- Audit logs
- Compromised account indicators

In this lab, the forwarding condition was intentionally created for troubleshooting practice.

---

# Exchange Admin Center vs PowerShell

## Exchange Admin Center

Exchange Admin Center provides a straightforward graphical interface for reviewing and changing mailbox forwarding.

It is useful for:

- Identifying whether forwarding is enabled
- Selecting internal forwarding recipients
- Configuring external forwarding
- Controlling delivery to both addresses

## Exchange Online PowerShell

PowerShell provides detailed properties that make the forwarding behavior easier to diagnose.

Important properties include:

- ForwardingAddress
- ForwardingSmtpAddress
- DeliverToMailboxAndForward

PowerShell also allows an internal object identifier to be resolved using:

`Get-Recipient`

---

# Commands Used

## Review Forwarding Configuration

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

## Store Forwarding Target

`$ForwardTarget = (Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com").ForwardingAddress`

## Resolve Forwarding Target

`Get-Recipient -Identity $ForwardTarget | Format-Table DisplayName,PrimarySmtpAddress,RecipientTypeDetails -AutoSize`

## Verify Forwarding Removal

`Get-Mailbox -Identity "robert.garcia@Stefon.onmicrosoft.com" | Format-List DisplayName,ForwardingAddress,ForwardingSmtpAddress,DeliverToMailboxAndForward`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/05-Mail-Flow/`

Scenario 6 evidence includes:

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
- Exchange Mail Flow
- Mailbox Forwarding
- ForwardingAddress
- ForwardingSmtpAddress
- DeliverToMailboxAndForward
- Exchange Admin Center
- Exchange Online PowerShell
- Get-Mailbox
- Get-Recipient
- Outlook on the web
- Controlled Mail Testing
- Root Cause Analysis
- Service Restoration
- Security Awareness
- Technical Documentation

---

# Scenario Outcome

**Status: Resolved**

Robert Garcia's missing email issue was caused by unintended mailbox forwarding to Brian Carter.

PowerShell confirmed that forwarding was configured and that `DeliverToMailboxAndForward` was set to `False`.

A controlled message test demonstrated that messages addressed to Robert were redirected to Brian and were not retained in Robert's mailbox.

The forwarding configuration was removed.

PowerShell verified the forwarding fields were blank, and a second controlled message successfully arrived in Robert's Inbox.

Normal Exchange Online mail delivery was restored.

---

# Next Scenario

The next scenario will focus on missing email investigation using Exchange Online message trace.

The lab will use message trace to determine what happened to a message after Exchange Online processed it and will demonstrate how administrators distinguish delivery problems from mailbox visibility problems.
