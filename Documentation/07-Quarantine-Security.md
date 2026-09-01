# 07 — Quarantine and Email Security Troubleshooting

## Overview

This section documents Exchange Online quarantine investigation and Microsoft 365 email security troubleshooting.

Messages that do not appear in a user's Inbox are not always lost or rejected.

Microsoft 365 can intentionally isolate messages in quarantine when they match:

- Exchange transport rules
- Anti-spam policies
- Anti-phishing policies
- Anti-malware policies
- Other organizational security controls

This scenario demonstrates how to investigate a quarantined message across multiple Microsoft 365 administrative tools and verify the complete remediation process.

Administrative tools used include:

- Exchange Admin Center
- Exchange Online Message Trace
- Exchange Online PowerShell
- Microsoft Defender
- Microsoft Defender Quarantine
- Outlook on the web

---

# Scenario 8 — Quarantined Message Investigation

## Scenario Summary

**Affected User:** Brian Carter  
**Recipient Address:** brian.carter@Stefon.onmicrosoft.com

**Sender:** Emily Brown  
**Sender Address:** emily.brown@Stefon.onmicrosoft.com

**Test Subject:**

`LAB QUARANTINE TEST - Review Required`

**Issue:**

An expected message did not arrive in Brian Carter's Inbox.

Investigation determined that Exchange Online successfully processed the message but an organizational transport rule redirected the message into Microsoft Defender quarantine.

The message was reviewed, released, and normal delivery was restored.

---

# Lab Objective

The objectives of this scenario were to demonstrate how an Exchange Online administrator can:

1. Create a controlled quarantine condition.
2. Verify the transport rule through Exchange Admin Center.
3. Verify the same rule through PowerShell.
4. Send a controlled test message.
5. Confirm the recipient does not receive the message normally.
6. Investigate the message using Message Trace.
7. Determine which policy caused the quarantine.
8. Locate the message in Microsoft Defender.
9. Review quarantine metadata.
10. Query the quarantined message using PowerShell.
11. Release the message.
12. Verify the release through PowerShell.
13. Disable the controlled transport rule.
14. Verify the disabled rule through PowerShell.
15. Confirm normal mail delivery is restored.

---

# Controlled Security Configuration

A dedicated Exchange Online transport rule was created for this scenario.

The rule was named:

`LAB - Quarantine Test`

The rule condition was:

`Subject includes LAB QUARANTINE TEST`

The rule action was:

`Deliver the message to the hosted quarantine`

The rule was configured with:

- State: Enabled
- Mode: Enforce
- Priority: 0
- Quarantine action: Enabled

This configuration created a predictable security event without requiring malware, suspicious links, or unsafe message content.

---

# Exchange Admin Center Rule Verification

The rule was reviewed through:

`Exchange Admin Center > Mail flow > Rules`

The rule details showed:

**Rule name**

`LAB - Quarantine Test`

**Status**

`Enabled`

**Mode**

`Enforce`

**Condition**

`Includes these words in the message subject: LAB QUARANTINE TEST`

**Action**

`Deliver the message to the hosted quarantine`

This confirmed that matching messages would be intentionally prevented from reaching the recipient's normal Inbox.

---

# PowerShell Transport Rule Verification

Exchange Online PowerShell was used to independently verify the rule configuration.

The following command was executed:

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

The output showed:

- Name: LAB - Quarantine Test
- State: Enabled
- Mode: Enforce
- SubjectContainsWords: LAB QUARANTINE TEST
- Quarantine: True

This confirmed that the transport rule configuration shown in the graphical portal matched the Exchange Online configuration returned through PowerShell.

---

# Controlled Test Message

Emily Brown sent a message directly to:

`brian.carter@Stefon.onmicrosoft.com`

The subject was:

`LAB QUARANTINE TEST - Review Required`

The body stated:

`This message is used to validate Exchange Online quarantine investigation and release procedures.`

The message matched the subject condition configured in the transport rule.

---

# Recipient-Side Symptom

Brian Carter signed into Outlook on the web.

The message did not appear in Brian's normal Inbox.

This represented the user-facing incident:

> An expected message was sent but was never received.

At this point, several possible causes could normally be considered:

- Transport failure
- Invalid recipient
- Inbox rules
- Forwarding
- Junk mail
- Deleted Items
- Quarantine
- Anti-spam processing
- Mail flow rules
- Security policy

Message Trace was used to determine what Exchange Online actually did with the message.

---

# Message Trace Investigation

Exchange Admin Center was opened.

Navigation:

`Mail flow > Message trace`

The trace was filtered using:

**Sender**

`emily.brown@Stefon.onmicrosoft.com`

**Recipient**

`brian.carter@Stefon.onmicrosoft.com`

The message:

`LAB QUARANTINE TEST - Review Required`

appeared in the trace results.

Its status was:

`Quarantined`

This provided the first direct indication that Exchange Online had intentionally isolated the message rather than losing or rejecting it unexpectedly.

---

# Detailed Message Trace

The detailed message trace was opened.

The message processing information showed that the message had been:

- Received
- Processed
- Not yet delivered normally

The trace specifically stated that the message was quarantined because it matched an organizational mail flow rule.

The responsible rule was identified as:

`LAB - Quarantine Test`

The message event list also contained a transport rule event.

This connected the user-facing missing-email symptom directly to a known Exchange Online configuration.

---

# Why Message Trace Was Important

Without Message Trace, the issue could have been mistaken for:

- Mailbox failure
- Outlook synchronization
- Recipient configuration
- Sender failure
- Microsoft 365 service interruption

Message Trace provided evidence that:

1. Exchange Online received the message.
2. The sender and recipient were valid.
3. The transport system processed the message.
4. An organizational policy affected delivery.
5. The message was placed in quarantine.

This allowed the troubleshooting process to shift from general mail-flow investigation to security and quarantine analysis.

---

# Microsoft Defender Investigation

Microsoft Defender was opened.

Navigation:

`Email & collaboration > Review > Quarantine`

The **Email** quarantine view contained the test message.

The quarantine list showed:

- Time received
- Subject
- Sender
- Quarantine reason
- Release status
- Policy type
- Recipient

The controlled message appeared with:

**Subject**

`LAB QUARANTINE TEST - Review Required`

**Sender**

Emily Brown

**Recipient**

Brian Carter

**Quarantine reason**

`Transport Rule`

**Release status**

`Needs review`

**Policy type**

`Exchange transport rule`

This independently confirmed the Message Trace finding.

---

# Quarantine Message Details

The quarantined message was opened for additional investigation.

Microsoft Defender displayed detailed information including:

**Subject**

`LAB QUARANTINE TEST - Review Required`

**Quarantine reason**

`Transport Rule`

**Policy type**

`Exchange transport rule`

**Policy name**

`LAB - Quarantine Test`

**Recipient**

`brian.carter@Stefon.onmicrosoft.com`

**Release state**

Not yet released

**Original location**

`Quarantine`

**Latest delivery location**

`Quarantine`

**Delivery action**

`Blocked`

The message was therefore being held according to organizational Exchange transport policy.

---

# Security Interpretation

The message was not automatically classified as malicious.

Instead, the message had intentionally matched a controlled organizational transport rule.

This distinction is important.

Possible reasons that messages can appear in quarantine include:

- Spam classification
- High-confidence spam
- Phishing
- High-confidence phishing
- Malware
- Transport rules
- Administrative policies

In this lab, the quarantine reason was explicitly:

`Transport Rule`

This allowed the administrator to investigate the precise control responsible for the delivery behavior.

---

# PowerShell Quarantine Cmdlet Verification

Before querying the quarantined message, PowerShell was used to verify that the required command was available.

The following command was executed:

`Get-Command Get-QuarantineMessage -ErrorAction SilentlyContinue`

PowerShell returned the:

`Get-QuarantineMessage`

function.

This confirmed that quarantine information could also be investigated through the Exchange Online PowerShell session.

---

# PowerShell Quarantine Investigation

The following command was used to query Brian Carter's quarantined messages:

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

The test message was returned.

The output confirmed:

- SenderAddress: emily.brown@Stefon.onmicrosoft.com
- RecipientAddress: brian.carter@Stefon.onmicrosoft.com
- Subject: LAB QUARANTINE TEST - Review Required
- Type: Transport rule
- ReleaseStatus: NOTRELEASED

This provided PowerShell-based confirmation that:

1. The message existed in quarantine.
2. The quarantine was caused by a transport rule.
3. The message had not yet been released.

---

# Root Cause

The root cause was the Exchange Online transport rule:

`LAB - Quarantine Test`

The rule matched any message containing:

`LAB QUARANTINE TEST`

in the subject.

The configured action was:

`Deliver the message to the hosted quarantine`

The rule was:

`Enabled`

and operating in:

`Enforce`

mode.

Therefore, Exchange Online intentionally routed the message into hosted quarantine rather than Brian Carter's Inbox.

---

# Root Cause Classification

The incident was caused by:

**Exchange Online mail flow policy**

rather than:

- Missing Exchange Online license
- Mailbox provisioning failure
- Distribution list membership
- Shared mailbox delegation
- Send As permissions
- Send on Behalf permissions
- Mailbox forwarding
- Inbox rules
- Outlook client failure
- Invalid recipient
- Exchange Online outage

---

# Quarantine Review

Before releasing the message, the quarantined item was reviewed.

The investigation verified:

- Known sender
- Known recipient
- Known test subject
- Controlled transport rule
- Expected quarantine reason
- Expected policy name

Because the message was intentionally generated as part of the controlled lab scenario, release was considered appropriate.

---

# Message Release

Microsoft Defender was used to release the quarantined message.

The administrator selected:

`Release email`

Microsoft Defender confirmed:

`Email has been released to recipient inboxes`

This indicated that the quarantined message had been approved for re-delivery.

---

# PowerShell Release Verification

After the message was released, the same PowerShell query was executed again:

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

The result now showed:

`ReleaseStatus : RELEASED`

This independently confirmed that the Microsoft Defender release operation had succeeded.

---

# Transport Rule Remediation

Because the transport rule existed only for controlled testing, it was disabled after the quarantine investigation.

Exchange Admin Center navigation:

`Mail flow > Rules > LAB - Quarantine Test`

The rule state was changed from:

`Enabled`

to:

`Disabled`

This prevented the controlled rule from affecting future messages.

---

# PowerShell Rule-State Verification

The rule was queried again with:

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

The output showed:

- Name: LAB - Quarantine Test
- State: Disabled
- Mode: Enforce
- SubjectContainsWords: LAB QUARANTINE TEST
- Quarantine: True

The quarantine action remained part of the rule definition.

However, because:

`State : Disabled`

the transport rule no longer processed new messages.

---

# End-to-End Recovery Test

A fresh message was sent from Emily Brown to Brian Carter.

The subject was:

`LAB QUARANTINE TEST - Resolved`

The message body stated:

`This message validates normal mail delivery after the quarantine transport rule was disabled.`

Brian Carter signed into Outlook on the web.

The message appeared successfully in Brian's Inbox.

This demonstrated that normal mail delivery was restored after disabling the transport rule.

---

# Resolution Verification

Scenario 8 was considered successfully resolved after confirming all of the following:

- The transport rule was enabled at the beginning of the scenario.
- PowerShell confirmed `Quarantine : True`.
- The controlled test message did not appear normally in Brian's Inbox.
- Message Trace reported `Quarantined`.
- Detailed Message Trace identified `LAB - Quarantine Test`.
- Microsoft Defender contained the message.
- Defender reported `Transport Rule` as the quarantine reason.
- Defender identified the policy as `LAB - Quarantine Test`.
- PowerShell returned `ReleaseStatus : NOTRELEASED`.
- The message was released through Microsoft Defender.
- Defender confirmed successful release.
- PowerShell returned `ReleaseStatus : RELEASED`.
- The transport rule was disabled.
- PowerShell confirmed `State : Disabled`.
- A new test message arrived normally in Brian Carter's Inbox.

---

# Troubleshooting Workflow

The troubleshooting workflow demonstrated in this scenario was:

1. Confirm the sender.
2. Confirm the recipient.
3. Obtain the message subject.
4. Determine the approximate send time.
5. Confirm the message is absent from the recipient's Inbox.
6. Run Exchange Online Message Trace.
7. Identify the message.
8. Review the trace status.
9. Determine that the message was quarantined.
10. Open detailed trace information.
11. Identify the responsible mail flow rule.
12. Open Microsoft Defender.
13. Navigate to Quarantine.
14. Locate the affected message.
15. Review the quarantine reason.
16. Review the policy type.
17. Review the policy name.
18. Verify the recipient.
19. Verify the sender.
20. Query the quarantine item through PowerShell.
21. Confirm it has not been released.
22. Review whether release is appropriate.
23. Release the message.
24. Verify the release operation.
25. Query PowerShell again.
26. Confirm `ReleaseStatus : RELEASED`.
27. Disable the controlled transport rule.
28. Verify the rule state through Exchange Admin Center.
29. Verify the rule state through PowerShell.
30. Send another controlled message.
31. Confirm normal delivery.
32. Document the incident.

---

# Quarantine vs Message Rejection

Understanding the difference between quarantine and rejection is important in Exchange troubleshooting.

## Quarantine

A quarantined message is retained by Microsoft 365 for review.

Depending on policy and administrator permissions, the message may be:

- Reviewed
- Previewed
- Released
- Deleted
- Reported

The original sender may believe the message was sent successfully even though the recipient has not received it.

---

## Rejection

A rejected message is refused during mail processing.

Depending on the condition, the sender may receive a non-delivery report.

A rejected message typically requires investigation into:

- Recipient validity
- Transport rules
- Authentication
- Accepted domains
- Connectors
- Policy
- Message restrictions

---

# Message Trace vs Quarantine

Message Trace and Microsoft Defender Quarantine serve different troubleshooting purposes.

## Message Trace

Message Trace answers:

> What happened to the message while Exchange Online processed it?

It can provide information about:

- Sender
- Recipient
- Subject
- Time
- Delivery status
- Transport events
- Quarantine
- Mail flow rules

---

## Microsoft Defender Quarantine

Microsoft Defender answers:

> Why is the message being held and what security action can be taken?

It provides information such as:

- Quarantine reason
- Policy type
- Policy name
- Sender
- Recipient
- Release status
- Threat information
- Delivery action
- Release controls

Both tools were required to fully investigate this scenario.

---

# Exchange Admin Center vs Microsoft Defender

## Exchange Admin Center

Exchange Admin Center was used for:

- Creating the transport rule
- Reviewing the transport rule
- Running Message Trace
- Identifying the quarantine result
- Determining which transport rule affected the message
- Disabling the transport rule

---

## Microsoft Defender

Microsoft Defender was used for:

- Locating the quarantined message
- Reviewing quarantine metadata
- Confirming the quarantine reason
- Reviewing the controlling policy
- Reviewing delivery status
- Releasing the message

---

# PowerShell Role in the Investigation

PowerShell provided independent verification at multiple points.

PowerShell was used to:

- Verify the transport rule
- Confirm the rule was enabled
- Confirm the quarantine action
- Query the quarantined message
- Confirm the message had not been released
- Verify successful release
- Verify the transport rule had been disabled

This provided administrative evidence separate from the graphical portals.

---

# Commands Used

## Verify Quarantine Transport Rule

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

---

## Verify Quarantine Cmdlet

`Get-Command Get-QuarantineMessage -ErrorAction SilentlyContinue`

---

## Query Quarantined Message

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

---

## Verify Message Release

`Get-QuarantineMessage -RecipientAddress "brian.carter@Stefon.onmicrosoft.com" | Where-Object {$_.Subject -like "*LAB QUARANTINE TEST*"} | Format-Table ReceivedTime,SenderAddress,RecipientAddress,Subject,Type,ReleaseStatus -AutoSize`

---

## Verify Disabled Transport Rule

`Get-TransportRule -Identity "LAB - Quarantine Test" | Format-List Name,State,Mode,SubjectContainsWords,Quarantine`

---

# Security Best Practices

Administrators should not automatically release every quarantined message requested by a user.

Before release, administrators should consider:

- Whether the sender is expected
- Whether the recipient expected the message
- Why the message was quarantined
- Which policy caused the action
- Whether the message contains suspicious content
- Whether links or attachments are present
- Whether the organization permits release
- Whether security escalation is required

Potential malicious messages should remain quarantined until properly reviewed.

---

# Transport Rule Security Consideration

Transport rules can significantly change how organizational email is processed.

Incorrect mail flow rules may:

- Redirect messages
- Reject messages
- Quarantine messages
- Add recipients
- Remove headers
- Modify subject lines
- Apply disclaimers
- Prevent expected mail delivery

Administrators should therefore:

- Use descriptive rule names
- Document business purpose
- Verify conditions
- Review exceptions
- Check rule priority
- Test controlled messages
- Monitor message trace
- Disable unused testing rules

---

# Controlled Testing Approach

This lab intentionally used a harmless subject-based mail flow rule.

No malware, phishing links, unsafe attachments, or malicious content were required.

This approach allowed quarantine troubleshooting to be practiced safely while still demonstrating:

- Transport rule administration
- Message Trace
- Microsoft Defender
- Quarantine review
- PowerShell
- Message release
- Security policy remediation

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/07-Security/`

Scenario 8 evidence includes:

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
- Microsoft 365 Email Security
- Exchange Transport Rules
- Mail Flow Rules
- Message Trace
- Quarantine Investigation
- Quarantine Message Release
- Exchange Admin Center
- Microsoft Defender Quarantine
- Exchange Online PowerShell
- Get-TransportRule
- Get-QuarantineMessage
- Security Policy Troubleshooting
- Mail Flow Troubleshooting
- Outlook on the web
- Root Cause Analysis
- Incident Response
- Service Restoration
- End-to-End Verification
- Technical Documentation

---

# Scenario Outcome

**Status: Resolved**

Brian Carter reported that an expected email from Emily Brown did not appear in his Inbox.

Exchange Online Message Trace showed that the message was quarantined.

Detailed trace information identified the organizational mail flow rule:

`LAB - Quarantine Test`

Microsoft Defender confirmed:

- Quarantine reason: Transport Rule
- Policy type: Exchange transport rule
- Policy name: LAB - Quarantine Test
- Delivery action: Blocked

Exchange Online PowerShell independently confirmed that the message was initially:

`NOTRELEASED`

The quarantined message was reviewed and released through Microsoft Defender.

PowerShell then confirmed:

`ReleaseStatus : RELEASED`

The controlled transport rule was disabled.

PowerShell independently confirmed:

`State : Disabled`

A fresh controlled test message successfully arrived directly in Brian Carter's Inbox.

Normal Exchange Online mail delivery was restored.

---

# Next Scenario

The next support scenario will focus on external mail-flow troubleshooting.

The scenario will demonstrate how Exchange administrators investigate whether messages can be delivered between Microsoft 365 and external recipients and how accepted domains, remote domains, connectors, mail flow rules, and Message Trace contribute to 
troubleshooting external email problems.
