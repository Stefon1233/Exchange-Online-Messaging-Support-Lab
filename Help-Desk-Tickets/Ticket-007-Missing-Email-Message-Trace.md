# INC-007 — Missing Email Investigation Using Message Trace

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-007 |
| Requester | Brian Carter |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Mail Flow / Message Trace |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User reports a missing email. Exchange Online message trace confirms successful delivery, and further investigation identifies an Inbox rule that moved the message to Deleted Items.

---

# User-Reported Issue

Brian Carter reports that an expected email from Emily Brown did not appear in his Inbox.

The sender reports that the message was sent successfully.

The incident requires investigation to determine whether the message failed during Exchange Online transport or was processed after reaching Brian's mailbox.

---

# Environment

**Affected User:** Brian Carter  
**Recipient Address:** brian.carter@Stefon.onmicrosoft.com

**Sender:** Emily Brown  
**Sender Address:** emily.brown@Stefon.onmicrosoft.com

**Test Subject:** LAB TRACE TEST - Missing Email

Administrative tools used:

- Exchange Admin Center
- Exchange Online Message Trace
- Exchange Online PowerShell
- Outlook on the web
- Inbox Rules

---

# Initial Mailbox Rule Condition

A controlled Inbox rule was configured in Brian Carter's mailbox for troubleshooting practice.

Rule name:

`LAB - Message Trace Test`

Condition:

`Subject includes LAB TRACE TEST`

Action:

`Delete the message`

The rule was enabled.

In Outlook, this processing caused the matching message to be placed in Deleted Items instead of remaining in Brian's Inbox.

---

# Controlled Test Message

Emily Brown sent a test message directly to:

`brian.carter@Stefon.onmicrosoft.com`

Subject:

`LAB TRACE TEST - Missing Email`

Body:

`This message is used to investigate a reported missing email with Exchange Online message trace.`

Brian Carter did not see the message in his normal Inbox.

---

# Exchange Admin Center Message Trace

Exchange Admin Center was opened.

Navigation:

`Mail flow > Message trace`

The trace was filtered for:

**Sender:** emily.brown@Stefon.onmicrosoft.com

**Recipient:** brian.carter@Stefon.onmicrosoft.com

The test message appeared in the message trace results.

Its status was:

`Delivered`

This immediately indicated that Exchange Online transport had successfully processed and delivered the message.

The incident therefore required mailbox-level investigation rather than transport-level remediation.

---

# Detailed Message Trace Investigation

The detailed message trace was opened for:

`LAB TRACE TEST - Missing Email`

Exchange Admin Center showed the message progressing through:

- Received
- Processed
- Delivered

The detailed status stated that the message was delivered to the recipient's mailbox.

It also identified that an Inbox rule configured by the recipient affected the message.

The resulting folder was:

`Deleted Items`

This provided direct Exchange Online evidence explaining why the user reported the message as missing.

---

# PowerShell Message Trace

Exchange Online PowerShell was used to independently investigate the message.

The following command was used to retrieve the most recent matching message:

`$Trace = Get-MessageTraceV2 -SenderAddress "emily.brown@Stefon.onmicrosoft.com" -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" -StartDate (Get-Date).AddHours(-2) -EndDate (Get-Date) | Where-Object {$_.Subject -like "*LAB TRACE TEST*"} | Sort-Object 
Received -Descending | Select-Object -First 1`

The trace object was displayed using:

`$Trace | Format-List Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId`

The output confirmed:

- SenderAddress: emily.brown@Stefon.onmicrosoft.com
- RecipientAddress: brian.carter@Stefon.onmicrosoft.com
- Subject: LAB TRACE TEST - Missing Email
- Status: Delivered

This independently confirmed successful Exchange Online transport.

---

# Message Trace Event Details

Detailed message events were retrieved using:

`$Trace | Get-MessageTraceDetailV2 | Format-Table Date,Event,Action,Detail -Wrap`

The output showed events including:

- Receive
- Submit
- Deliver

The Deliver event stated that the message was delivered to the Deleted Items folder.

This confirmed that Exchange Online successfully delivered the message and that mailbox processing determined its final location.

---

# Inbox Rule Investigation

Because transport delivery was successful, the investigation shifted to Brian Carter's mailbox.

Inbox rules were queried using:

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Format-Table Name,Enabled,Priority,Description -Wrap`

The output returned:

`LAB - Message Trace Test`

with:

`Enabled : True`

The rule description showed that messages containing:

`LAB TRACE TEST`

in the subject were processed by the rule and deleted.

This identified the mailbox rule responsible for the user's missing-email symptom.

---

# Message Location Verification

Brian Carter's Deleted Items folder was reviewed through Outlook on the web.

The missing message was located successfully.

The message showed:

- Sender: Emily Brown
- Recipient: Brian Carter
- Subject: LAB TRACE TEST - Missing Email

This confirmed that the message was not lost during mail transport.

It had reached Brian's mailbox and was subsequently processed by the Inbox rule.

---

# Root Cause

The root cause was an enabled Inbox rule in Brian Carter's mailbox.

The rule matched messages containing:

`LAB TRACE TEST`

in the subject and deleted them.

Exchange Online message transport was operating correctly.

The issue was therefore caused by mailbox-level rule processing rather than:

- Exchange Online outage
- Sender failure
- Recipient failure
- Mailbox provisioning
- Licensing
- Distribution group membership
- Mail forwarding
- Transport rejection

---

# Resolution

The problematic Inbox rule was disabled through Exchange Online PowerShell.

The following command was executed:

`Disable-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" -Identity "LAB - Message Trace Test" -Confirm:$false`

The rule state was then verified using:

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Name -eq "LAB - Message Trace Test"} | Format-Table Name,Enabled,Priority -AutoSize`

The result showed:

`Enabled : False`

This confirmed that the problematic mailbox rule was no longer active.

---

# End-to-End Resolution Test

Emily Brown sent a second controlled message to Brian Carter.

Subject:

`LAB TRACE TEST - Resolved`

Body:

`This message validates normal Inbox delivery after the problematic mailbox rule was disabled.`

Brian Carter signed into Outlook on the web.

The new message appeared directly in his Inbox.

This confirmed that normal mailbox delivery and Inbox visibility had been restored.

---

# Resolution Verification

The incident was considered resolved after confirming:

- Exchange Admin Center message trace showed `Delivered`.
- Detailed trace identified the destination as Deleted Items.
- PowerShell returned `Status : Delivered`.
- Message trace events included `Deliver`.
- PowerShell identified the enabled Inbox rule.
- The original message was found in Deleted Items.
- The problematic Inbox rule was disabled.
- PowerShell confirmed `Enabled : False`.
- A second test message arrived normally in Brian's Inbox.

---

# Troubleshooting Workflow

1. Confirm the sender and recipient addresses.
2. Obtain the approximate message time.
3. Obtain the message subject.
4. Search Exchange Online message trace.
5. Determine whether the message was delivered, failed, deferred, or rejected.
6. Review detailed message events.
7. Confirm Exchange Online delivered the message.
8. Shift investigation from transport to mailbox processing.
9. Review Inbox rules.
10. Identify the rule affecting the message.
11. Locate the message in Deleted Items.
12. Disable the problematic Inbox rule.
13. Verify the rule is disabled.
14. Send a new controlled test message.
15. Confirm normal Inbox delivery.
16. Document the root cause and resolution.

---

# Key Troubleshooting Lesson

A user saying that an email is "missing" does not necessarily mean Exchange Online failed to deliver it.

Message trace should be used to determine what happened during transport.

If message trace reports:

`Delivered`

the investigation should continue inside the recipient mailbox.

Potential mailbox-level causes include:

- Inbox rules
- Junk Email
- Deleted Items
- Archive
- Focused Inbox
- Mailbox forwarding
- User actions
- Outlook filtering
- Retention or compliance processing

Message trace helps administrators distinguish transport problems from mailbox visibility and processing problems.

---

# Message Trace Status Interpretation

## Delivered

Exchange Online successfully delivered the message.

Further mailbox-level investigation may be necessary if the user still cannot locate it.

## Failed

The message could not be delivered.

The failure details should be investigated.

## Pending or Deferred

Exchange is still attempting delivery or temporarily delayed processing.

## Filtered or Security-Affected

Security or policy controls may have altered the message's processing.

---

# Commands Used

## Search Message Trace

`$Trace = Get-MessageTraceV2 -SenderAddress "emily.brown@Stefon.onmicrosoft.com" -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" -StartDate (Get-Date).AddHours(-2) -EndDate (Get-Date) | Where-Object {$_.Subject -like "*LAB TRACE TEST*"} | Sort-Object 
Received -Descending | Select-Object -First 1`

## Display Trace Result

`$Trace | Format-List Received,SenderAddress,RecipientAddress,Subject,Status,MessageTraceId`

## Display Trace Events

`$Trace | Get-MessageTraceDetailV2 | Format-Table Date,Event,Action,Detail -Wrap`

## Review Inbox Rules

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Format-Table Name,Enabled,Priority,Description -Wrap`

## Disable Problematic Rule

`Disable-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" -Identity "LAB - Message Trace Test" -Confirm:$false`

## Verify Rule State

`Get-InboxRule -Mailbox "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Name -eq "LAB - Message Trace Test"} | Format-Table Name,Enabled,Priority -AutoSize`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/06-Message-Trace/`

Evidence includes:

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
- Exchange Mail Flow
- Get-MessageTraceV2
- Get-MessageTraceDetailV2
- Inbox Rule Troubleshooting
- Get-InboxRule
- Disable-InboxRule
- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web
- Missing Email Investigation
- Root Cause Analysis
- Incident Management
- End-to-End Resolution Verification
- ServiceNow-Style Documentation

---

# Final Status

**Resolved**

Brian Carter reported that an email from Emily Brown was missing.

Exchange Admin Center and Exchange Online PowerShell both confirmed that Exchange Online successfully delivered the message.

Detailed trace information showed that an Inbox rule caused the message to be placed in Deleted Items.

The enabled Inbox rule was identified through PowerShell and disabled.

The original message was located in Deleted Items, and a second controlled test message successfully arrived in Brian Carter's Inbox.

The missing-email incident was fully resolved.
