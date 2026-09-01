# INC-008 — Quarantined Email Investigation

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-008 |
| Requester | Brian Carter |
| Category | Microsoft 365 |
| Subcategory | Exchange Online / Email Security |
| Service | Mail Flow / Quarantine |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User reports that an expected email did not arrive because an Exchange Online transport rule placed the message into Microsoft Defender quarantine.

---

# User-Reported Issue

Brian Carter reports that an expected email from Emily Brown did not appear in his Inbox.

The sender confirms that the message was sent successfully.

The incident requires investigation to determine whether the message was rejected, lost during transport, delivered to another folder, or quarantined by an Exchange Online security or mail flow control.

---

# Environment

**Affected User:** Brian Carter  
**Recipient Address:** brian.carter@Stefon.onmicrosoft.com

**Sender:** Emily Brown  
**Sender Address:** emily.brown@Stefon.onmicrosoft.com

**Test Subject:** LAB QUARANTINE TEST - Review Required

Administrative tools used:

- Exchange Admin Center
- Exchange Online PowerShell
- Exchange Online Message Trace
- Microsoft Defender
- Microsoft Defender Quarantine
- Outlook on the web

---

# Controlled Quarantine Configuration

A controlled Exchange Online transport rule was created for troubleshooting practice.

Rule name:

`LAB - Quarantine Test`

Condition:

`Subject includes LAB QUARANTINE TEST`

Action:

`Deliver the message to the hosted quarantine`

The rule was configured as:

- State: Enabled
- Mode: Enforce
- Quarantine: True

The rule only targeted messages containing the designated lab subject phrase.

---

# Exchange Admin Center Rule Verification

Exchange Admin Center was opened.

Navigation:

`Mail flow > Rules`

The `LAB - Quarantine Test` rule was reviewed.

The rule showed:

- Status: Enabled
- Mode: Enforce
- Subject condition: LAB QUARANTINE TEST
- Action: Deliver the message to the hosted quarantine

This confirmed that the controlled quarantine policy was active.

---

# PowerShell Transport Rule Verification

Exchange Online PowerShell was used to independently verify the transport rule.

The following command was executed:

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

The result showed:

- Name: LAB - Quarantine Test
- State: Enabled
- Mode: Enforce
- SubjectContainsWords: LAB QUARANTINE TEST
- Quarantine: True

This confirmed that Exchange Online was configured to quarantine matching messages.

---

# Controlled Test Message

Emily Brown sent a controlled test email directly to:

`brian.carter@Stefon.onmicrosoft.com`

Subject:

`LAB QUARANTINE TEST - Review Required`

Body:

`This message is used to validate Exchange Online quarantine investigation and release procedures.`

The message did not appear in Brian Carter's Inbox.

This established the user-facing symptom.

---

# Message Trace Investigation

Exchange Admin Center was opened.

Navigation:

`Mail flow > Message trace`

The trace was filtered for:

**Sender**

`emily.brown@Stefon.onmicrosoft.com`

**Recipient**

`brian.carter@Stefon.onmicrosoft.com`

The controlled test message appeared with the status:

`Quarantined`

This confirmed that Exchange Online had processed the message but intentionally prevented normal mailbox delivery.

---

# Detailed Message Trace

The detailed trace was opened for:

`LAB QUARANTINE TEST - Review Required`

Exchange Admin Center showed that the message was:

- Received
- Processed
- Not yet delivered

The trace explicitly stated that the message was quarantined because it matched an organizational mail flow rule.

The identified rule was:

`LAB - Quarantine Test`

This linked the missing-email symptom directly to the Exchange transport rule.

---

# Microsoft Defender Quarantine Investigation

Microsoft Defender was opened.

Navigation:

`Email & collaboration > Review > Quarantine`

The controlled test message appeared in the Email quarantine list.

The message displayed:

- Sender: Emily Brown
- Recipient: Brian Carter
- Subject: LAB QUARANTINE TEST - Review Required
- Quarantine reason: Transport Rule
- Release status: Needs review
- Policy type: Exchange transport rule

This confirmed that the message was retained in Microsoft Defender quarantine rather than being delivered to Brian's mailbox.

---

# Defender Message Details

The quarantined message details were reviewed.

The message showed:

- Subject: LAB QUARANTINE TEST - Review Required
- Quarantine reason: Transport Rule
- Policy type: Exchange transport rule
- Policy name: LAB - Quarantine Test
- Recipient: brian.carter@Stefon.onmicrosoft.com
- Original location: Quarantine
- Latest delivery location: Quarantine
- Delivery action: Blocked
- Not yet released to: brian.carter@Stefon.onmicrosoft.com

This provided direct evidence that organizational policy was responsible for the delivery interruption.

---

# PowerShell Quarantine Investigation

The availability of the quarantine command was verified using:

`Get-Command Get-QuarantineMessage -ErrorAction SilentlyContinue`

The command was available in the Exchange Online session.

The quarantined message was queried using:

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

The result showed:

- SenderAddress: emily.brown@Stefon.onmicrosoft.com
- RecipientAddress: brian.carter@Stefon.onmicrosoft.com
- Subject: LAB QUARANTINE TEST - Review Required
- Type: Transport rule
- ReleaseStatus: NOTRELEASED

This independently confirmed that the message remained quarantined and had not yet been released.

---

# Root Cause

The root cause was the Exchange Online transport rule:

`LAB - Quarantine Test`

The rule matched the subject:

`LAB QUARANTINE TEST`

and applied the action:

`Deliver the message to the hosted quarantine`

As a result, the test message was processed successfully by Exchange Online but was intentionally isolated in Microsoft Defender quarantine.

The issue was not caused by:

- Exchange Online outage
- Mailbox provisioning
- Missing license
- Distribution group membership
- Shared mailbox permissions
- Inbox rules
- Mailbox forwarding
- Recipient address failure

---

# Resolution — Message Release

The quarantined message was reviewed in Microsoft Defender.

The administrator selected:

`Release email`

Microsoft Defender confirmed:

`Email has been released to recipient inboxes`

The quarantined message was therefore released for delivery to Brian Carter.

---

# PowerShell Release Verification

The quarantine object was queried again after the release.

The following command was executed:

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

The result now showed:

`ReleaseStatus : RELEASED`

This independently confirmed that the administrator successfully released the message from quarantine.

---

# Transport Rule Remediation

After the quarantine behavior was validated, the controlled transport rule was disabled.

Exchange Admin Center was opened.

Navigation:

`Mail flow > Rules > LAB - Quarantine Test`

The rule state was changed to:

`Disabled`

This prevented subsequent test messages containing the lab subject phrase from being quarantined.

---

# PowerShell Rule-State Verification

The transport rule was queried again using:

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

The result showed:

- Name: LAB - Quarantine Test
- State: Disabled
- Mode: Enforce
- SubjectContainsWords: LAB QUARANTINE TEST
- Quarantine: True

The quarantine action remained configured in the rule, but the disabled state prevented the rule from processing new messages.

---

# End-to-End Resolution Test

A fresh test message was sent from Emily Brown to Brian Carter.

Subject:

`LAB QUARANTINE TEST - Resolved`

Body:

`This message validates normal mail delivery after the quarantine transport rule was disabled.`

Brian Carter signed into Outlook on the web.

The message successfully appeared directly in Brian's Inbox.

This confirmed that normal Exchange Online mail delivery had been restored after the transport rule was disabled.

---

# Resolution Verification

The incident was considered resolved after confirming:

- The original message was absent from Brian's Inbox.
- Message trace showed `Quarantined`.
- Detailed message trace identified `LAB - Quarantine Test`.
- Microsoft Defender contained the quarantined message.
- Defender identified the quarantine reason as `Transport Rule`.
- PowerShell returned `ReleaseStatus : NOTRELEASED`.
- The message was released through Microsoft Defender.
- PowerShell then returned `ReleaseStatus : RELEASED`.
- The quarantine transport rule was disabled.
- PowerShell confirmed `State : Disabled`.
- A new controlled test message arrived normally in Brian's Inbox.

---

# Troubleshooting Workflow

1. Confirm sender and recipient addresses.
2. Obtain the message subject and approximate send time.
3. Verify the message is absent from the user's Inbox.
4. Run Exchange Online message trace.
5. Identify the message status as Quarantined.
6. Open detailed trace information.
7. Identify the transport rule responsible.
8. Open Microsoft Defender quarantine.
9. Locate the quarantined message.
10. Review the quarantine reason.
11. Review the policy type and policy name.
12. Verify quarantine state using PowerShell.
13. Confirm the message has not been released.
14. Review the message and determine that release is appropriate.
15. Release the message.
16. Verify release status with PowerShell.
17. Disable the controlled transport rule.
18. Verify the rule state with PowerShell.
19. Send a new controlled test message.
20. Confirm normal Inbox delivery.
21. Document the root cause and resolution.

---

# Key Troubleshooting Lesson

A missing email may have been intentionally quarantined by Microsoft 365 security or mail flow controls.

Administrators should not assume that a missing message failed during transport.

Useful investigation areas include:

- Message trace
- Transport rules
- Microsoft Defender quarantine
- Spam policies
- Anti-phishing policies
- Anti-malware policies
- Safe Attachments
- Safe Links
- Tenant Allow/Block configuration

Message trace can help determine whether the message was quarantined and identify the mail flow rule responsible.

Microsoft Defender then provides security-focused information about the quarantined item and allows authorized administrators to review and release appropriate messages.

---

# Quarantine vs Rejection

## Quarantine

The message is retained by Microsoft 365 for review.

An administrator may be able to inspect and release it according to organizational policy.

## Rejection

The message is refused during mail processing.

The sender may receive a non-delivery report depending on the failure condition.

Understanding this distinction helps determine whether an administrator should investigate transport errors or Microsoft Defender quarantine.

---

# Security Consideration

Quarantined messages should not automatically be released simply because a user requests them.

Administrators should review:

- Sender
- Recipient
- Subject
- Quarantine reason
- Policy
- Message contents when permitted
- Threat information
- Organizational security policy

In this lab, the quarantine was intentionally caused by a controlled transport rule and used a harmless test message.

---

# Commands Used

## Verify Transport Rule

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

## Verify Quarantine Command

`Get-Command Get-QuarantineMessage -ErrorAction SilentlyContinue`

## Query Quarantined Message

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

## Verify Released Message

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

## Verify Disabled Transport Rule

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/07-Security/`

Evidence includes:

- `01-LAB-Quarantine-Mail-Flow-Rule.png`
- `02-Quarantine-Rule-PowerShell-Verification.png`
- `03-Brian-Quarantined-Message-Not-In-Inbox.png`
- `04-Quarantined-Message-Trace.png`
- `05-Quarantined-Message-Trace-Details.png`
- `06-Message-Found-In-Microsoft-Defender-Quarantine.png`
- `07-Quarantine-Message-Details.png`
- `08-Quarantined-Message-PowerShell-Verification.png`
- `09-Quarantined-Message-Released.png`
- `10-Quarantine-Release-PowerShell-Verification.png`
- `11-LAB-Quarantine-Rule-Disabled.png`
- `12-Quarantine-Rule-Disabled-PowerShell.png`
- `13-Brian-Normal-Delivery-After-Quarantine-Resolution.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Microsoft Defender
- Email Security
- Quarantine Investigation
- Message Release
- Exchange Transport Rules
- Mail Flow Rules
- Message Trace
- Exchange Online PowerShell
- Get-TransportRule
- Get-QuarantineMessage
- Outlook on the web
- Root Cause Analysis
- Security Policy Troubleshooting
- Service Restoration
- Incident Management
- End-to-End Resolution Verification
- ServiceNow-Style Documentation

---

# Final Status

**Resolved**

Brian Carter reported that an expected message from Emily Brown did not arrive.

Exchange Online message trace showed that the message was quarantined because it matched the `LAB - Quarantine Test` transport rule.

Microsoft Defender confirmed that the quarantine reason was `Transport Rule` and identified the controlling Exchange transport rule.

PowerShell independently confirmed that the message was initially `NOTRELEASED`.

The message was reviewed and released through Microsoft Defender.

PowerShell then confirmed `ReleaseStatus : RELEASED`.

The controlled transport rule was disabled, and PowerShell confirmed `State : Disabled`.

A new controlled message successfully arrived directly in Brian Carter's Inbox.

The quarantine incident was fully resolved.
