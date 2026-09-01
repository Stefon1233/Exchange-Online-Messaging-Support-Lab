# INC-009 — External Email Delivery Failure

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-009 |
| Requester | Emily Brown |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | External Mail Flow |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User reports that an outbound message to an external Gmail recipient was rejected. Exchange Online Message Trace and PowerShell identified an organizational transport rule as the cause. The rule was disabled and external delivery was restored.

---

# User-Reported Issue

Emily Brown reported that she was unable to send a specific message from the Microsoft 365 tenant to an external Gmail recipient.

A non-delivery report stated that the message had been blocked by a mail flow rule.

The incident required investigation to determine whether the failure was caused by:

- External recipient availability
- Exchange Online routing
- Accepted domain configuration
- Remote domain configuration
- Outbound connectors
- Transport rules
- External mail provider rejection

---

# Environment

**Sender:** Emily Brown  
**Sender Address:** emily.brown@Stefon.onmicrosoft.com

**External Recipient:** Gmail mailbox

**Failure Test Subject:**

`LAB EXTERNAL BLOCK TEST - Delivery Failure`

Administrative tools used:

- Exchange Admin Center
- Exchange Online Message Trace
- Exchange Online PowerShell
- Outlook on the web
- Gmail

---

# External Mail Flow Baseline

Before creating the controlled failure, the Exchange Online environment was reviewed.

The following configuration was verified through PowerShell:

## Accepted Domain

The tenant contained:

`Stefon.onmicrosoft.com`

with:

- DomainType: Authoritative
- Default: True

This confirmed the Microsoft 365 tenant's authoritative Exchange Online domain.

---

# Remote Domain Baseline

The default remote domain was reviewed.

The configuration showed:

- DomainName: `*`
- AllowedOOFType: External
- AutoReplyEnabled: True
- AutoForwardEnabled: True

This established the tenant's default behavior for remote mail domains.

---

# Outbound Connector Baseline

The following command was executed:

`Get-OutboundConnector`

No custom outbound connectors were returned.

This was not considered an error.

Standard Exchange Online internet mail delivery can operate without a custom outbound connector.

---

# Baseline External Delivery Test

Emily Brown sent a controlled baseline message to the external Gmail mailbox.

Subject:

`LAB EXTERNAL MAIL - Baseline`

Body:

`This message validates normal outbound Exchange Online delivery to an external recipient.`

The message successfully arrived in Gmail.

This proved that:

- The external Gmail address was valid.
- Exchange Online could deliver internet mail.
- General outbound mail flow was working.
- DNS and standard Microsoft 365 outbound routing were functioning.

The later failure could therefore be compared against a known-good baseline.

---

# Controlled Fault Configuration

A dedicated Exchange Online transport rule was created.

Rule name:

`LAB - External Mail Block`

Condition:

`Subject or body contains LAB EXTERNAL BLOCK TEST`

Action:

Reject the message with the explanation:

`LAB external mail flow policy blocked this test message.`

Enhanced status code:

`5.7.1`

The rule was configured as:

- State: Enabled
- Mode: Enforce

This created a deterministic external mail delivery failure for troubleshooting practice.

---

# Exchange Admin Center Verification

Exchange Admin Center was opened.

Navigation:

`Mail flow > Rules`

The `LAB - External Mail Block` rule showed:

- Status: Enabled
- Mode: Enforce
- Subject/body condition: LAB EXTERNAL BLOCK TEST
- Reject action configured
- Status code: 5.7.1

This confirmed the controlled failure policy was active.

---

# PowerShell Transport Rule Verification

The rule was independently verified using Exchange Online PowerShell.

Command:

`Get-TransportRule -Identity "LAB - External Mail Block" | Format-List Name,State,Mode,SubjectOrBodyContainsWords,RejectMessageReasonText,RejectMessageEnhancedStatusCode`

The result showed:

- Name: LAB - External Mail Block
- State: Enabled
- Mode: Enforce
- SubjectOrBodyContainsWords: LAB EXTERNAL BLOCK TEST
- RejectMessageReasonText: LAB external mail flow policy blocked this test message.
- RejectMessageEnhancedStatusCode: 5.7.1

This confirmed that Exchange Online was configured to reject matching messages.

---

# Failure Reproduction

Emily Brown sent another message to the same external Gmail recipient.

Subject:

`LAB EXTERNAL BLOCK TEST - Delivery Failure`

Body:

`This controlled message validates external mail-flow troubleshooting in Exchange Online.`

Because the message matched the active transport rule, Exchange Online rejected it.

---

# Non-Delivery Report

Emily Brown received a non-delivery report in Outlook.

The report displayed:

`Blocked by mail flow rule`

The NDR also included:

`550 5.7.1`

The message explained that an email administrator had created a custom mail flow rule that blocked the message.

This provided direct user-facing evidence of the Exchange Online policy failure.

---

# Message Trace Investigation

Exchange Admin Center was opened.

Navigation:

`Mail flow > Message trace`

The trace was filtered using Emily Brown as the sender and the Gmail address as the recipient.

The controlled failure message appeared with:

`Status: Failed`

The trace also displayed an earlier successful external delivery.

This comparison demonstrated that external mail flow itself was functioning and that the later message had failed because of a message-specific condition.

---

# Detailed Message Trace

The failed message was opened in Message Trace.

Exchange Admin Center showed:

- Received
- Processed
- Not delivered

The trace stated that Office 365 received the message but could not deliver it because an email administrator configured a mail flow rule that rejected the message.

The responsible rule was identified as:

`LAB - External Mail Block`

Message events included:

- Receive
- Submit
- Fail
- Transport rule

This directly identified the root cause.

---

# PowerShell Message Trace

Exchange Online PowerShell was used to retrieve the same failed message.

The trace showed:

- SenderAddress: emily.brown@Stefon.onmicrosoft.com
- External Gmail recipient
- Subject: LAB EXTERNAL BLOCK TEST - Delivery Failure
- Status: Failed

The message trace object contained a unique MessageTraceId for further analysis.

---

# Detailed PowerShell Trace

Detailed message events were retrieved using:

`Get-MessageTraceDetailV2`

The results included:

- Receive
- Submit
- Fail
- Fail
- Transport rule

The failure details included:

`550 5.7.1 TRANSPORT.RULES.RejectMessage`

and indicated that:

`the message was rejected by organization policy`

The transport rule event identified:

`LAB - External Mail Block`

This independently confirmed the Exchange Admin Center diagnosis.

---

# Root Cause

The root cause was the enabled Exchange Online transport rule:

`LAB - External Mail Block`

The rule matched:

`LAB EXTERNAL BLOCK TEST`

in the subject or body and rejected the message using enhanced status code:

`5.7.1`

The external Gmail provider was not responsible for the failure.

The message was rejected inside the Microsoft 365 organization before normal internet delivery could complete.

---

# Root Cause Classification

The incident was caused by:

**Exchange Online organizational mail flow policy**

It was not caused by:

- Invalid external Gmail address
- Exchange Online outage
- Missing mailbox
- Missing Exchange license
- Accepted domain failure
- Remote domain failure
- Custom outbound connector failure
- Gmail rejecting the message
- DNS failure
- Internet connectivity failure

---

# Remediation

Exchange Admin Center was opened.

Navigation:

`Mail flow > Rules > LAB - External Mail Block`

The rule was changed from:

`Enabled`

to:

`Disabled`

This stopped the controlled policy from rejecting matching messages.

---

# PowerShell Remediation Verification

The rule was queried again:

`Get-TransportRule -Identity "LAB - External Mail Block" | Format-List Name,State,Mode,SubjectOrBodyContainsWords,RejectMessageReasonText,RejectMessageEnhancedStatusCode`

The output showed:

`State : Disabled`

The rule definition still contained its original condition and rejection action, but the disabled state prevented the rule from processing new messages.

---

# End-to-End Resolution Test

Emily Brown sent another message to the same Gmail recipient.

Subject:

`LAB EXTERNAL BLOCK TEST - Resolved`

Body:

`This message validates normal external Exchange Online delivery after the blocking mail flow rule was disabled.`

The message successfully arrived in Gmail.

This proved that outbound external mail delivery had been restored.

---

# PowerShell Resolution Trace

The final message was queried through Exchange Online PowerShell.

The result showed:

`Status : Delivered`

The detailed events included:

- Receive
- Submit
- TRANSFER
- Send external

The final Send external event showed Exchange Online handing the message off to Google's SMTP infrastructure.

There was no:

`TRANSPORT.RULES.RejectMessage`

event in the successful processing path.

This confirmed that the organizational block had been removed and normal internet delivery was restored.

---

# Resolution Verification

The incident was considered resolved after confirming:

- A baseline message successfully reached Gmail.
- The controlled transport rule was enabled.
- PowerShell confirmed the rejection configuration.
- The failure test produced an NDR.
- The NDR showed `550 5.7.1`.
- The NDR stated the message was blocked by a mail flow rule.
- Message Trace returned `Failed`.
- Detailed Message Trace identified `LAB - External Mail Block`.
- PowerShell returned `Status : Failed`.
- Detailed PowerShell trace showed `TRANSPORT.RULES.RejectMessage`.
- The transport rule was disabled.
- PowerShell confirmed `State : Disabled`.
- A new controlled message successfully arrived in Gmail.
- PowerShell returned `Status : Delivered`.
- The successful trace contained `Send external`.

---

# Troubleshooting Workflow

1. Confirm sender address.
2. Confirm external recipient address.
3. Establish normal external delivery baseline.
4. Review accepted domains.
5. Review remote domain configuration.
6. Review outbound connectors.
7. Reproduce the affected message condition.
8. Review the non-delivery report.
9. Record the SMTP enhanced status code.
10. Run Exchange Online Message Trace.
11. Identify the message as Failed.
12. Open detailed trace results.
13. Identify the responsible mail flow rule.
14. Query the message through PowerShell.
15. Review detailed PowerShell events.
16. Identify `TRANSPORT.RULES.RejectMessage`.
17. Confirm organization policy is the root cause.
18. Disable the offending transport rule.
19. Verify the disabled state through PowerShell.
20. Send another controlled external message.
21. Confirm external Gmail receipt.
22. Run a final Message Trace.
23. Verify `Status : Delivered`.
24. Confirm the `Send external` event.
25. Document the root cause and remediation.

---

# Important Troubleshooting Lesson

External mail failure does not automatically mean the external provider rejected the message.

Administrators should determine where the failure occurred.

Potential locations include:

- Sender mailbox
- Exchange transport
- Organizational transport rules
- Connectors
- DNS
- Internet routing
- External mail gateway
- Recipient mail server
- Recipient mailbox

In this scenario, both the NDR and Message Trace showed that Microsoft 365 rejected the message because of an organizational mail flow rule.

---

# SMTP Enhanced Status Code

The failure used:

`5.7.1`

A 5.x.x response represents a permanent delivery failure.

The specific NDR and trace information provided the important context that the failure was caused by organizational policy rather than a generic external mail issue.

Administrators should review both:

- The SMTP status code
- The associated diagnostic text

rather than troubleshooting from the status code alone.

---

# Accepted Domains

Accepted domains define the SMTP domains for which an Exchange organization accepts messages.

The tenant's accepted domain was:

`Stefon.onmicrosoft.com`

and was configured as:

`Authoritative`

This was appropriate for the Microsoft 365 tenant.

The external Gmail domain does not need to be configured as an accepted domain for normal outbound mail.

---

# Remote Domains

Remote domains control certain Exchange messaging behavior with external domains.

The default remote domain:

`*`

was present.

Remote domain configuration was reviewed as part of the external mail troubleshooting baseline but was not responsible for the failure.

---

# Outbound Connectors

No custom outbound connector was present.

This was not considered an issue.

Exchange Online can provide normal internet mail delivery using Microsoft's standard outbound routing without an organization-specific outbound connector.

A custom connector would normally be investigated if the organization used:

- Third-party mail gateways
- Smart hosts
- Partner organizations
- Hybrid Exchange
- Specialized mail routing

---

# Transport Rules

Transport rules can inspect organizational email and perform actions such as:

- Reject messages
- Quarantine messages
- Redirect messages
- Add recipients
- Modify headers
- Add disclaimers
- Modify subjects
- Apply routing controls

An incorrectly configured transport rule can therefore appear to users as an external email outage.

Message Trace is especially useful for identifying these policy-driven failures.

---

# Commands Used

## Accepted Domains

`Get-AcceptedDomain | Select-Object Name,DomainName,DomainType,Default | Format-Table -AutoSize`

## Remote Domains

`Get-RemoteDomain | Select-Object Name,DomainName,AllowedOOFType,AutoReplyEnabled,AutoForwardEnabled | Format-Table -AutoSize`

## Outbound Connectors

`Get-OutboundConnector | Select-Object Name,Enabled,ConnectorType,RecipientDomains | Format-Table -AutoSize`

## Review Transport Rule

`Get-TransportRule -Identity "LAB - External Mail Block" | Format-List Name,State,Mode,SubjectOrBodyContainsWords,RejectMessageReasonText,RejectMessageEnhancedStatusCode`

## Failed Message Trace

`Get-MessageTraceV2`

## Detailed Failed Message Trace

`Get-MessageTraceDetailV2`

## Verify Disabled Rule

`Get-TransportRule -Identity "LAB - External Mail Block" | Format-List Name,State,Mode,SubjectOrBodyContainsWords,RejectMessageReasonText,RejectMessageEnhancedStatusCode`

## Final Resolution Trace

`Get-MessageTraceV2`

## Final Detailed Trace

`Get-MessageTraceDetailV2`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/08-External-Mail-Flow/`

Evidence includes:

- `01-External-Mail-Flow-Environment-Baseline.png`
- `02-External-Mail-Baseline-Delivered.png`
- `03-LAB-External-Mail-Block-Rule.png`
- `04-External-Mail-Block-PowerShell-Verification.png`
- `05-External-Mail-Non-Delivery-Report.png`
- `06-External-Mail-Failed-Message-Trace.png`
- `07-External-Mail-Trace-Rule-Diagnosis.png`
- `08-External-Mail-PowerShell-Trace-Diagnosis.png`
- `09-LAB-External-Mail-Block-Rule-Disabled.png`
- `10-External-Mail-Block-Disabled-PowerShell.png`
- `11-External-Mail-Delivery-Restored.png`
- `12-External-Mail-Resolution-PowerShell-Trace.png`

---

# Skills Demonstrated

- Exchange Online Administration
- External Mail Flow
- Exchange Transport Rules
- Mail Flow Rules
- Non-Delivery Reports
- SMTP Enhanced Status Codes
- Message Trace
- Get-MessageTraceV2
- Get-MessageTraceDetailV2
- Accepted Domains
- Remote Domains
- Outbound Connectors
- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web
- Gmail External Delivery Testing
- Root Cause Analysis
- Policy Troubleshooting
- Service Restoration
- End-to-End Verification
- Incident Documentation

---

# Final Status

**Resolved**

Emily Brown experienced an external email delivery failure to a Gmail recipient.

A normal baseline message had previously demonstrated successful external Exchange Online delivery.

The affected test message generated a non-delivery report containing:

`550 5.7.1`

and indicated that the message had been blocked by a mail flow rule.

Exchange Admin Center Message Trace reported:

`Failed`

and explicitly identified:

`LAB - External Mail Block`

Exchange Online PowerShell independently returned:

`Status : Failed`

and detailed events showed:

`TRANSPORT.RULES.RejectMessage`

The transport rule was disabled.

PowerShell confirmed:

`State : Disabled`

A new controlled external message successfully arrived in Gmail.

The final PowerShell trace reported:

`Status : Delivered`

and included:

`Send external`

confirming successful handoff to Google's mail infrastructure.

Normal external Exchange Online mail flow was restored.
