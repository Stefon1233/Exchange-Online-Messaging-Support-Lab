# 06 — Message Trace and Missing Email Troubleshooting

## Overview

This section documents Exchange Online message trace troubleshooting for a reported missing email.

Message trace is used to determine how Exchange Online processed a message after it entered the Microsoft 365 mail system.

A message that appears to be missing from a user's Inbox may still have been successfully delivered by Exchange Online.

This scenario demonstrates how to distinguish:

- Transport failure
- Successful Exchange delivery
- Mailbox-level processing
- Inbox rule behavior

The investigation uses:

- Exchange Admin Center
- Exchange Online Message Trace
- Exchange Online PowerShell
- Outlook on the web
- Inbox Rules

---

# Scenario 7 — Missing Email Investigation

## Incident Summary

**Affected User:** Brian Carter  
**Recipient Address:** brian.carter@Stefon.onmicrosoft.com

**Sender:** Emily Brown  
**Sender Address:** emily.brown@Stefon.onmicrosoft.com

**Message Subject:** LAB TRACE TEST - Missing Email

**Issue:** Brian Carter reported that an expected email did not appear in his Inbox.

The purpose of the investigation was to determine whether the message failed during Exchange Online transport or was successfully delivered and then processed inside Brian's mailbox.

---

# Controlled Fault Configuration

A controlled Inbox rule was created in Brian Carter's Outlook mailbox.

The rule was named:

`LAB - Message Trace Test`

The condition was:

`Subject includes LAB TRACE TEST`

The action deleted matching messages.

The rule was enabled.

This simulated a common support situation where a user reports that mail is missing even though Exchange Online is operating correctly.

---

# Controlled Test Message

Emily Brown sent a test message directly to:

`brian.carter@Stefon.onmicrosoft.com`

The subject was:

`LAB TRACE TEST - Missing Email`

The message body stated:

`This message is used to investigate a reported missing email with Exchange Online message trace.`

Because the Inbox rule matched the subject, the message did not remain in Brian Carter's Inbox.

---

# Initial User Symptoms

Brian Carter reported that the email was missing from his Inbox.

At this point, several possible causes could have been investigated, including:

- Mail transport failure
- Recipient address problem
- Exchange Online service issue
- Spam filtering
- Inbox rules
- Deleted Items
- Mail forwarding
- Mailbox filtering

Rather than assuming a mailbox issue, Exchange Online message trace was used first to determine whether Microsoft 365 successfully processed the message.

---

# Exchange Admin Center Message Trace

Exchange Admin Center was opened.

Navigation:

`Mail flow > Message trace`

The trace was filtered using:

**Sender**

`emily.brown@Stefon.onmicrosoft.com`

**Recipient**

`brian.carter@Stefon.onmicrosoft.com`

The test message appeared in the trace results.

The message subject was:

`LAB TRACE TEST - Missing Email`

The status was:

`Delivered`

This was the first major troubleshooting finding.

Exchange Online transport had successfully processed the message.

The investigation therefore shifted away from transport failure and toward mailbox-level processing.

---

# Detailed Message Trace

The detailed message trace was opened.

The message progressed through Exchange processing stages including:

- Received
- Processed
- Delivered

The detailed Exchange Admin Center result stated that the message had been delivered to the recipient's mailbox.

It also showed that an Inbox rule affected the message after delivery.

The destination folder was:

`Deleted Items`

This directly explained why Brian Carter believed the message had disappeared.

Exchange Online had delivered it successfully, but the recipient's Inbox rule moved it away from the Inbox.

---

# Message Trace Interpretation

The important diagnostic distinction was:

`Exchange transport delivery`

versus:

`Mailbox visibility`

The message trace demonstrated that the Exchange Online mail transport system was functioning correctly.

Because the status was:

`Delivered`

the administrator did not need to troubleshoot:

- DNS
- Exchange service availability
- Sender connectivity
- Recipient provisioning
- Mail routing
- Connectors
- Transport rejection

Instead, the investigation continued inside the recipient mailbox.

---

# PowerShell Message Trace

Exchange Online PowerShell was used to independently locate the same message.

The following command was used:

`$Trace = Get-MessageTraceV2 -SenderAddress "emily.brown@Stefon.onmicrosoft.com" -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" -StartDate (Get-Date).AddHours(-2) -EndDate (Get-Date) | Where-Object {$_.Subject -like "*LAB TRACE TEST*"} | Sort-Object 
Received -Descending | Select-Object -First 1`

The result was displayed using:

`$Trace | Format-List Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId`

The output confirmed:

- SenderAddress: emily.brown@Stefon.onmicrosoft.com
- RecipientAddress: brian.carter@Stefon.onmicrosoft.com
- Subject: LAB TRACE TEST - Missing Email
- Status: Delivered

This independently confirmed the Exchange Admin Center message trace result.

---

# PowerShell Message Event Investigation

The detailed message events were retrieved using:

`$Trace | Get-MessageTraceDetailV2 | Format-Table Date,Event,Action,Detail -Wrap`

The results included:

- Receive
- Submit
- Deliver

The Deliver event showed that the message was delivered to the Deleted Items folder.

This was especially useful because it connected the mail transport investigation directly to the mailbox symptom.

---

# Inbox Rule Investigation

Because Exchange Online had successfully delivered the message, Brian Carter's mailbox rules were inspected.

The following command was executed:

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Format-Table Name,Enabled,Priority,Description -Wrap`

The output returned:

`LAB - Message Trace Test`

The rule showed:

`Enabled : True`

Its description showed that messages containing:

`LAB TRACE TEST`

in the subject were deleted and that additional rule processing was stopped.

This identified the rule responsible for the incident.

---

# Message Location Verification

Brian Carter's Outlook mailbox was opened.

The Deleted Items folder contained:

`LAB TRACE TEST - Missing Email`

The message showed:

- Sender: Emily Brown
- Recipient: Brian Carter
- Subject: LAB TRACE TEST - Missing Email

This confirmed that the message was never lost.

It had been:

1. Sent successfully.
2. Processed by Exchange Online.
3. Delivered to Brian Carter.
4. Processed by a mailbox rule.
5. Placed in Deleted Items.

---

# Root Cause

The root cause was an enabled Inbox rule in Brian Carter's mailbox.

The rule matched messages containing:

`LAB TRACE TEST`

and deleted them.

Exchange Online transport was functioning correctly.

The problem occurred after successful delivery.

---

# Remediation

The problematic Inbox rule was disabled using Exchange Online PowerShell.

The following command was executed:

`Disable-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" -Identity "LAB - Message Trace Test" -Confirm:$false`

The rule state was then verified using:

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Name -eq "LAB - Message Trace Test"} | Format-Table Name,Enabled,Priority -AutoSize`

The result showed:

`Enabled : False`

This confirmed that the rule was no longer processing incoming messages.

---

# End-to-End Recovery Test

A second controlled test message was sent from Emily Brown to Brian Carter.

The subject was:

`LAB TRACE TEST - Resolved`

The body stated:

`This message validates normal Inbox delivery after the problematic mailbox rule was disabled.`

Brian Carter opened Outlook on the web.

The new message appeared directly in his Inbox.

This confirmed that normal mailbox behavior had been restored.

---

# Resolution Verification

The incident was considered resolved after confirming:

- Message trace showed the original message as Delivered.
- Detailed EAC trace identified Deleted Items as the final folder.
- PowerShell confirmed Status was Delivered.
- Detailed PowerShell trace included the Deliver event.
- PowerShell identified the enabled Inbox rule.
- The original message was found in Deleted Items.
- The Inbox rule was disabled.
- PowerShell confirmed Enabled was False.
- A new test message arrived directly in Brian Carter's Inbox.

---

# Troubleshooting Workflow

The troubleshooting process used during this scenario was:

1. Identify the sender.
2. Identify the recipient.
3. Obtain the message subject.
4. Determine the approximate send time.
5. Search Exchange Admin Center message trace.
6. Confirm the message exists in the trace.
7. Review the message status.
8. Identify the message as Delivered.
9. Open the detailed message trace.
10. Review processing events.
11. Identify the Deleted Items destination.
12. Verify the result using Exchange Online PowerShell.
13. Review Receive, Submit, and Deliver events.
14. Inspect the recipient's Inbox rules.
15. Identify the matching rule.
16. Locate the message in Deleted Items.
17. Determine the Inbox rule is the root cause.
18. Disable the problematic rule.
19. Verify the rule is disabled.
20. Send another controlled test message.
21. Confirm normal Inbox delivery.
22. Document the incident and resolution.

---

# Message Trace Status Interpretation

## Delivered

A Delivered status means Exchange Online successfully delivered the message.

If the user still cannot locate it, the investigation should continue inside the mailbox.

Potential causes include:

- Inbox rules
- Deleted Items
- Junk Email
- Archive
- Mail forwarding
- User actions
- Client-side filtering
- Focused Inbox

---

## Failed

A Failed status indicates that Exchange Online could not successfully deliver the message.

The administrator should review the failure details and diagnostic information.

---

## Pending or Deferred

These states can indicate that delivery has not yet completed or that Exchange is temporarily delaying the message.

The administrator should review the associated message events and retry information.

---

# Transport vs Mailbox Troubleshooting

A useful troubleshooting decision point is:

`Did Exchange Online deliver the message?`

If the answer is no, investigate transport.

Potential areas include:

- Sender
- Recipient
- Connectors
- Mail-flow rules
- Accepted domains
- Security policies
- Transport errors

If the answer is yes, investigate the mailbox.

Potential areas include:

- Inbox rules
- Forwarding
- Deleted Items
- Junk Email
- Archive
- Outlook filtering
- Mailbox permissions
- User activity

---

# Exchange Admin Center vs PowerShell

## Exchange Admin Center

Exchange Admin Center message trace provides a visual representation of:

- Sender
- Recipient
- Subject
- Message time
- Delivery status
- Processing stages
- Detailed message events

It is useful for quickly investigating user-reported mail issues.

---

## Exchange Online PowerShell

PowerShell provides detailed and repeatable message-trace queries.

Commands used in this scenario included:

- `Get-MessageTraceV2`
- `Get-MessageTraceDetailV2`
- `Get-InboxRule`
- `Disable-InboxRule`

PowerShell is useful for:

- Filtering specific messages
- Automating investigations
- Retrieving detailed events
- Auditing mailbox rules
- Verifying remediation

---

# Commands Used

## Search for the Message

`$Trace = Get-MessageTraceV2 -SenderAddress "emily.brown@Stefon.onmicrosoft.com" -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" -StartDate (Get-Date).AddHours(-2) -EndDate (Get-Date) | Where-Object {$_.Subject -like "*LAB TRACE TEST*"} | Sort-Object 
Received -Descending | Select-Object -First 1`

## Display Message Trace Details

`$Trace | Format-List Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId`

## Review Detailed Events

`$Trace | Get-MessageTraceDetailV2 | Format-Table Date,Event,Action,Detail -Wrap`

## Review Brian Carter's Inbox Rules

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Format-Table Name,Enabled,Priority,Description -Wrap`

## Disable the Problematic Rule

`Disable-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" -Identity "LAB - Message Trace Test" -Confirm:$false`

## Verify the Rule Is Disabled

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Name -eq "LAB - Message Trace Test"} | Format-Table Name,Enabled,Priority -AutoSize`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/06-Message-Trace/`

Scenario 7 evidence includes:

- `01-Brian-Message-Trace-Test-Rule.png`
- `02-Missing-Email-Search-Shows-Deleted-Items.png`
- `03-Message-Trace-Delivered-To-Deleted-Items.png`
- `04-Message-Trace-Delivered-Status.png`
- `05-PowerShell-Message-Trace-Delivered.png`
- `06-Brian-Inbox-Rule-PowerShell-Diagnosis.png`
- `07-Missing-Email-Found-In-Deleted-Items.png`
- `08-Brian-Inbox-Rule-Disabled.png`
- `09-Brian-Normal-Inbox-Delivery-Restored.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Message Trace
- Mail Flow Troubleshooting
- Exchange Admin Center
- Exchange Online PowerShell
- Get-MessageTraceV2
- Get-MessageTraceDetailV2
- Inbox Rule Troubleshooting
- Get-InboxRule
- Disable-InboxRule
- Outlook on the web
- Missing Email Investigation
- Transport vs Mailbox Troubleshooting
- Root Cause Analysis
- Service Restoration
- End-to-End Verification
- Technical Documentation

---

# Scenario Outcome

**Status: Resolved**

Brian Carter reported that an email sent by Emily Brown was missing.

Exchange Admin Center message trace showed that Exchange Online successfully delivered the message.

Detailed trace information showed that the message was delivered to Deleted Items because of an Inbox rule.

Exchange Online PowerShell independently confirmed the Delivered status and the detailed delivery event.

The problematic Inbox rule was identified and disabled.

The original message was found in Deleted Items.

A second test message successfully arrived directly in Brian Carter's Inbox.

The incident was fully resolved.

---

# Next Scenario

The next Exchange Online support scenario will focus on email security.

The lab will investigate a message affected by spam or quarantine controls and will demonstrate how an administrator determines why a message was blocked or isolated before restoring appropriate delivery.
