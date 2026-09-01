# Exchange Online Messaging Support Lab

Hands-on Microsoft 365 and Exchange Online support portfolio demonstrating mailbox administration, licensing troubleshooting, shared mailbox permissions, distribution groups, mail forwarding, message trace, Microsoft Defender quarantine, external mail flow, 
Outlook access troubleshooting, and Exchange Online PowerShell.

## Project Overview

This project simulates real-world Exchange Online incidents commonly handled by:

- Help Desk Technicians
- IT Support Specialists
- Microsoft 365 Support Engineers
- Junior Microsoft 365 Administrators
- Cloud Support Specialists

The lab focuses on a complete troubleshooting lifecycle:

1. Reproduce the user-reported issue
2. Establish a known-good baseline
3. Gather administrative evidence
4. Investigate through Microsoft 365 and Exchange tools
5. Validate findings with PowerShell
6. Identify the root cause
7. Apply targeted remediation
8. Perform end-to-end testing
9. Document the incident
10. Preserve screenshot evidence

## Portfolio Results

- **10 completed Exchange Online support incidents**
- **9 detailed technical documentation files**
- **96 troubleshooting screenshots**
- Microsoft 365 and Exchange Online administration
- Exchange Online PowerShell
- Microsoft Defender quarantine investigations
- Message Trace analysis
- Shared mailbox permission testing
- Internal and external mail-flow testing
- Controlled fault simulation and remediation
- ServiceNow-style incident documentation

---

## Technologies Used

### Microsoft Platforms

- Microsoft 365 Admin Center
- Microsoft Entra ID
- Exchange Online
- Exchange Admin Center
- Outlook on the web
- Microsoft Defender

### Administration

- PowerShell 7
- ExchangeOnlineManagement
- Exchange Online PowerShell
- Git
- GitHub

### Exchange Features

- User Mailboxes
- Shared Mailboxes
- Mailbox Delegation
- Full Access
- Send As
- Send on Behalf
- Distribution Groups
- Mailbox Forwarding
- Inbox Rules
- Message Trace
- Transport Rules
- Hosted Quarantine
- External Mail Flow
- Client Access Settings
- Microsoft 365 Licensing
- Exchange Online Service Plans

---

## Environment

The lab uses a Microsoft 365 cloud tenant containing test users, Exchange Online mailboxes, shared mailboxes, distribution groups, licensing, Microsoft Defender functionality, and controlled mail-flow configurations.

Administration was performed from macOS using browser-based Microsoft administrative portals and PowerShell.

### PowerShell Environment

```text
PowerShell 7.6.3
ExchangeOnlineManagement 3.10.1
```

Example connection:

```powershell
Connect-ExchangeOnline -ShowBanner:$false
```

---

# Support Scenarios

## 1. User Mailbox Not Provisioned

**Affected User:** Brian Carter

### Problem

Brian existed in Microsoft 365 and Microsoft Entra ID but did not have an Exchange Online mailbox.

### Investigation

- Verified the Microsoft 365 user account
- Verified the identity in Microsoft Entra ID
- Searched Exchange Admin Center
- Confirmed the mailbox was missing
- Reviewed Microsoft 365 licensing
- Confirmed Brian did not have an Exchange-capable license assigned

### Root Cause

No Microsoft 365 license containing Exchange Online services was assigned.

### Resolution

Microsoft 365 Business Basic was assigned and Exchange Online provisioned the mailbox.

### Verification

```text
DisplayName          : Brian Carter
PrimarySmtpAddress   : brian.carter@Stefon.onmicrosoft.com
RecipientTypeDetails : UserMailbox
```

[Technical Documentation](Documentation/02-Mailbox-Provisioning-Licensing.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-001-Mailbox-Not-Provisioned.md)

---

## 2. Missing Exchange Online Service Entitlement

**Affected User:** Emily Brown

### Problem

Emily still had Microsoft 365 Business Basic assigned, but her Exchange Online mailbox became unavailable.

### Investigation

The parent Microsoft 365 license remained assigned, but the Exchange Online service inside the license was disabled.

Exchange Admin Center no longer displayed Emily's active mailbox and Exchange Online PowerShell could not locate it.

### Root Cause

The Exchange Online service plan was disabled while the parent Microsoft 365 license remained assigned.

### Resolution

Re-enabled:

```text
Exchange Online (Plan 1)
```

### Verification

Emily's Exchange mailbox returned as:

```text
RecipientTypeDetails : UserMailbox
```

[Technical Documentation](Documentation/02-Mailbox-Provisioning-Licensing.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-002-Missing-Exchange-License.md)

---

## 3. Shared Mailbox Access

**Affected User:** Brian Carter  
**Shared Mailbox:** Company Shared

### Problem

Brian needed access to the Company Shared mailbox.

### Investigation

The shared mailbox existed, but Brian did not have Full Access delegation.

PowerShell also showed no Full Access permission for Brian.

### Root Cause

Missing mailbox delegation.

### Resolution

Assigned Brian:

```text
Full Access
```

### Verification

PowerShell confirmed:

```text
AccessRights : {FullAccess}
IsInherited  : False
Deny         : False
```

[Technical Documentation](Documentation/03-Shared-Mailboxes-Permissions.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-003-Shared-Mailbox-Access.md)

---

## 4. Send As vs Send on Behalf

**Shared Mailbox:** Company Shared

This scenario demonstrated the practical difference between Exchange Online mailbox delegation permissions.

### Brian Carter — Send As

Brian was assigned:

```text
Send As
```

PowerShell confirmed:

```text
AccessRights : {SendAs}
```

A live Outlook test showed the recipient saw the sender as:

```text
Company Shared
```

Brian's personal identity was not displayed.

### Emily Brown — Send on Behalf

Emily was assigned:

```text
Send on Behalf
```

A live Outlook test showed:

```text
Emily Brown on behalf of Company Shared
```

### Permission Comparison

| Permission | Function |
|---|---|
| Full Access | Open and manage mailbox contents |
| Send As | Send directly as the shared mailbox identity |
| Send on Behalf | Send while displaying the delegate identity |

[Technical Documentation](Documentation/03-Shared-Mailboxes-Permissions.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-004-Send-As-Permission.md)

---

## 5. Distribution List Membership

**Affected User:** Brian Carter  
**Distribution List:** Finance Department

### Problem

Brian was not receiving messages sent to the Finance Department distribution list.

### Investigation

Exchange Admin Center showed existing members including:

- Emily Brown
- Robert Garcia

Brian was not present.

PowerShell verification using:

```powershell
Get-DistributionGroupMember
```

also confirmed that Brian was not a member.

### Root Cause

Brian was missing from the Finance Department distribution list.

### Resolution

Added Brian Carter to the distribution list.

### Verification

PowerShell confirmed his membership and a live message sent to the Finance Department distribution list successfully arrived in Brian's Inbox.

[Technical Documentation](Documentation/04-Distribution-Groups.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-005-Distribution-Group.md)

---

## 6. Unintended Mailbox Forwarding

**Affected User:** Robert Garcia  
**Forwarding Destination:** Brian Carter

### Problem

Messages addressed to Robert were unexpectedly appearing in Brian's mailbox.

### Investigation

Exchange Online mailbox forwarding was configured.

PowerShell showed:

```text
DeliverToMailboxAndForward : False
```

The forwarding target was resolved as:

```text
Brian Carter
brian.carter@Stefon.onmicrosoft.com
```

### Controlled Test

A test message sent directly to Robert:

- Arrived in Brian's mailbox
- Was addressed to Robert
- Did not remain in Robert's mailbox

### Root Cause

Mailbox-level forwarding redirected incoming messages to Brian without retaining a copy in Robert's mailbox.

### Resolution

Disabled forwarding.

### Verification

PowerShell showed no forwarding destination and a new test message arrived normally in Robert's Inbox.

[Technical Documentation](Documentation/05-Mail-Forwarding-Mail-Flow.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-006-Mail-Forwarding.md)

---

## 7. Missing Email and Message Trace

**Affected User:** Brian Carter  
**Sender:** Emily Brown

### Problem

Brian reported that an expected email was missing from his Inbox.

### Investigation

Exchange Online Message Trace showed:

```text
Status : Delivered
```

Detailed trace information showed that the message had been delivered to:

```text
Deleted Items
```

PowerShell trace events included:

- Receive
- Submit
- Deliver

An Inbox rule named:

```text
LAB - Message Trace Test
```

was identified as the cause.

### Root Cause

An enabled Inbox rule moved the matching message away from Brian's Inbox.

### Resolution

Disabled the Inbox rule using Exchange Online PowerShell.

### Verification

PowerShell confirmed the rule was disabled and a new controlled message appeared directly in Brian's Inbox.

[Technical Documentation](Documentation/06-Message-Trace.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-007-Missing-Email-Message-Trace.md)

---

## 8. Quarantined Message Investigation

**Affected User:** Brian Carter  
**Sender:** Emily Brown

### Problem

An expected test message did not appear in Brian's Inbox.

### Controlled Rule

A transport rule was created:

```text
LAB - Quarantine Test
```

The rule quarantined messages containing the controlled test subject.

### Investigation

Message Trace showed:

```text
Quarantined
```

Microsoft Defender identified:

```text
Quarantine reason : Transport rule
Policy name       : LAB - Quarantine Test
```

PowerShell initially showed:

```text
ReleaseStatus : NOTRELEASED
```

### Root Cause

A controlled Exchange Online transport rule sent the matching message to hosted quarantine.

### Resolution

The message was reviewed and released through Microsoft Defender.

The controlled transport rule was then disabled.

### Verification

PowerShell showed:

```text
ReleaseStatus : RELEASED
```

and:

```text
State : Disabled
```

A follow-up message successfully delivered to Brian's Inbox.

[Technical Documentation](Documentation/07-Quarantine-Security.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-008-Quarantined-Message.md)

---

## 9. External Mail-Flow Failure

**Sender:** Emily Brown  
**Destination:** External Gmail recipient

### Baseline

A normal external message from Exchange Online successfully arrived in Gmail.

This established a known-good baseline for outbound internet mail delivery.

### Controlled Failure

A transport rule was configured:

```text
LAB - External Mail Block
```

Condition:

```text
Subject or body contains LAB EXTERNAL BLOCK TEST
```

Action:

```text
Reject message
```

Enhanced status code:

```text
5.7.1
```

### User Symptom

Emily received a non-delivery report showing:

```text
Blocked by mail flow rule
```

and:

```text
550 5.7.1
```

### Investigation

Exchange Admin Center Message Trace showed:

```text
Status : Failed
```

Detailed trace information identified:

```text
LAB - External Mail Block
```

PowerShell showed:

```text
TRANSPORT.RULES.RejectMessage
```

and indicated:

```text
the message was rejected by organization policy
```

### Root Cause

An organizational Exchange transport rule rejected the message before normal external delivery completed.

### Resolution

Disabled the transport rule.

### Verification

PowerShell confirmed:

```text
State : Disabled
```

A new test message successfully reached Gmail.

PowerShell Message Trace returned:

```text
Status : Delivered
```

Detailed processing included:

```text
Send external
```

confirming successful handoff to Google's SMTP infrastructure.

[Technical Documentation](Documentation/08-External-Mail-Flow.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-009-External-Mail-Flow-Failure.md)

---

## 10. Outlook on the Web Access Issue

**Affected User:** Brian Carter

### Problem

Brian could not access Outlook on the web.

A fresh browser session returned an Outlook error containing:

```text
Error: 440
```

### Investigation

Brian's Exchange Online mailbox still existed as a valid:

```text
UserMailbox
```

Client Access settings showed:

```text
OWAEnabled        : False
MAPIEnabled       : True
PopEnabled        : True
ImapEnabled       : True
ActiveSyncEnabled : True
```

### Mailbox Health Test

Emily sent a controlled test message:

```text
LAB OWA ACCESS TEST - Mailbox Healthy
```

Message Trace reported:

```text
Status : Delivered
```

Detailed events showed:

- Receive
- Submit
- Deliver

This proved that Brian's mailbox remained healthy and continued receiving email while Outlook on the web access was disabled.

### Root Cause

Outlook on the web had been specifically disabled through Brian's Exchange Online Client Access configuration.

### Resolution

```powershell
Set-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" -OWAEnabled $true
```

### Verification

PowerShell returned:

```text
OWAEnabled : True
```

Brian successfully accessed Outlook again.

The message delivered while OWA was disabled was visible in his Inbox after access was restored.

[Technical Documentation](Documentation/09-Outlook-Mailbox-Access.md)

[Incident Ticket](Help-Desk-Tickets/Ticket-010-Outlook-Mailbox-Access-Issue.md)

---

# Technical Documentation

| File | Topic |
|---|---|
| [01-Environment-Setup.md](Documentation/01-Environment-Setup.md) | Microsoft 365 and Exchange environment |
| [02-Mailbox-Provisioning-Licensing.md](Documentation/02-Mailbox-Provisioning-Licensing.md) | Mailbox provisioning and licensing |
| [03-Shared-Mailboxes-Permissions.md](Documentation/03-Shared-Mailboxes-Permissions.md) | Shared mailbox delegation |
| [04-Distribution-Groups.md](Documentation/04-Distribution-Groups.md) | Distribution list administration |
| [05-Mail-Forwarding-Mail-Flow.md](Documentation/05-Mail-Forwarding-Mail-Flow.md) | Forwarding and mail flow |
| [06-Message-Trace.md](Documentation/06-Message-Trace.md) | Missing email investigation |
| [07-Quarantine-Security.md](Documentation/07-Quarantine-Security.md) | Defender and quarantine |
| [08-External-Mail-Flow.md](Documentation/08-External-Mail-Flow.md) | External delivery troubleshooting |
| [09-Outlook-Mailbox-Access.md](Documentation/09-Outlook-Mailbox-Access.md) | Outlook client-access troubleshooting |

---

# Help Desk Incident Portfolio

| Incident | Scenario |
|---|---|
| [INC-001](Help-Desk-Tickets/Ticket-001-Mailbox-Not-Provisioned.md) | Mailbox not provisioned |
| [INC-002](Help-Desk-Tickets/Ticket-002-Missing-Exchange-License.md) | Exchange service entitlement |
| [INC-003](Help-Desk-Tickets/Ticket-003-Shared-Mailbox-Access.md) | Shared mailbox access |
| [INC-004](Help-Desk-Tickets/Ticket-004-Send-As-Permission.md) | Send As / Send on Behalf |
| [INC-005](Help-Desk-Tickets/Ticket-005-Distribution-Group.md) | Distribution group membership |
| [INC-006](Help-Desk-Tickets/Ticket-006-Mail-Forwarding.md) | Mail forwarding |
| [INC-007](Help-Desk-Tickets/Ticket-007-Missing-Email-Message-Trace.md) | Missing email investigation |
| [INC-008](Help-Desk-Tickets/Ticket-008-Quarantined-Message.md) | Quarantined message |
| [INC-009](Help-Desk-Tickets/Ticket-009-External-Mail-Flow-Failure.md) | External mail-flow failure |
| [INC-010](Help-Desk-Tickets/Ticket-010-Outlook-Mailbox-Access-Issue.md) | Outlook on the web access |

Each ticket documents:

- User-reported symptoms
- User impact
- Initial investigation
- Troubleshooting steps
- Commands used
- Root cause
- Resolution
- Validation
- Closure notes
- Skills demonstrated

---

# Screenshot Evidence

The repository contains **96 organized screenshots**.

```text
Screenshots/
├── 01-Environment/
├── 02-Mailboxes/
├── 03-Shared-Mailboxes/
├── 04-Groups/
├── 05-Mail-Flow/
├── 06-Message-Trace/
├── 07-Security/
├── 08-External-Mail-Flow/
└── 09-Outlook-Mailbox-Access/
```

Evidence includes:

- Microsoft 365 Admin Center
- Microsoft Entra ID
- Exchange Admin Center
- Microsoft Defender
- Exchange Online PowerShell
- Message Trace
- Mailbox delegation
- Distribution groups
- Mail forwarding
- Inbox rule troubleshooting
- Hosted quarantine
- Non-delivery reports
- External email delivery
- Outlook access failures
- Post-remediation validation

---

# Exchange Online PowerShell

Commands demonstrated throughout the lab include:

```powershell
Connect-ExchangeOnline
Get-ConnectionInformation

Get-EXOMailbox
Get-Mailbox

Get-CASMailbox
Set-CASMailbox

Get-MailboxPermission
Get-RecipientPermission
Get-Recipient

Get-DistributionGroup
Get-DistributionGroupMember

Get-InboxRule
Disable-InboxRule

Get-MessageTraceV2
Get-MessageTraceDetailV2

Get-TransportRule
Get-QuarantineMessage

Get-AcceptedDomain
Get-RemoteDomain
Get-OutboundConnector
```

---

# Troubleshooting Methodology

The scenarios follow a repeatable enterprise-support workflow:

```text
User Report
    ↓
Establish Baseline
    ↓
Reproduce Problem
    ↓
Verify Identity and Licensing
    ↓
Inspect Exchange Configuration
    ↓
Run Message Trace / PowerShell
    ↓
Identify Root Cause
    ↓
Apply Minimal Remediation
    ↓
Test End-to-End
    ↓
Document Resolution
```

The objective was not simply to restore service, but to prove the cause through administrative evidence before making changes.

---

# Skills Demonstrated

## Exchange Online

- Mailbox Administration
- Mailbox Provisioning
- Recipient Administration
- Shared Mailboxes
- Mailbox Delegation
- Full Access
- Send As
- Send on Behalf
- Distribution Groups
- Mailbox Forwarding
- Inbox Rules
- Message Trace
- Transport Rules
- External Mail Flow
- Client Access Settings

## Microsoft 365

- Microsoft 365 Admin Center
- Microsoft Entra ID
- Microsoft 365 Licensing
- Exchange Online Service Plans
- Identity Troubleshooting

## Security

- Microsoft Defender
- Quarantine Investigation
- Message Release
- Transport Rule Analysis
- Policy Troubleshooting

## PowerShell

- ExchangeOnlineManagement
- Exchange Online PowerShell
- Recipient Queries
- Permission Queries
- Message Trace Queries
- Client Access Administration
- Transport Rule Investigation

## IT Support

- Help Desk Troubleshooting
- Incident Management
- Root Cause Analysis
- Service Restoration
- User Impact Analysis
- End-to-End Validation
- Technical Documentation
- ServiceNow-Style Incident Documentation

---

# Key Troubleshooting Lessons

## Licensing Has Multiple Layers

A Microsoft 365 product license can remain assigned while the Exchange Online service plan inside that license is disabled.

Both the product license and individual service entitlement should be checked during troubleshooting.

## Mailbox Permissions Are Separate

Full Access, Send As, and Send on Behalf provide different capabilities and should be investigated independently.

## Delivered Does Not Mean Inbox

Message Trace can show a message as delivered while an Inbox rule moves it into Deleted Items or another folder.

## Transport and Mailbox Problems Are Different Layers

Successful Exchange transport does not guarantee that the user's mailbox experience is functioning normally.

## External Delivery Failures May Originate Internally

An NDR to an external recipient does not automatically mean the external provider rejected the message.

Exchange Online transport rules and organizational policies can stop mail before it leaves Microsoft 365.

## Client Access Is Separate From Mailbox Health

A mailbox can remain provisioned and continue receiving messages while Outlook on the web is disabled.

---

# Repository Structure

```text
Exchange-Online-Messaging-Support-Lab/
│
├── README.md
├── .gitignore
│
├── Documentation/
│   ├── 01-Environment-Setup.md
│   ├── 02-Mailbox-Provisioning-Licensing.md
│   ├── 03-Shared-Mailboxes-Permissions.md
│   ├── 04-Distribution-Groups.md
│   ├── 05-Mail-Forwarding-Mail-Flow.md
│   ├── 06-Message-Trace.md
│   ├── 07-Quarantine-Security.md
│   ├── 08-External-Mail-Flow.md
│   └── 09-Outlook-Mailbox-Access.md
│
├── Help-Desk-Tickets/
│   ├── Ticket-001-Mailbox-Not-Provisioned.md
│   ├── Ticket-002-Missing-Exchange-License.md
│   ├── Ticket-003-Shared-Mailbox-Access.md
│   ├── Ticket-004-Send-As-Permission.md
│   ├── Ticket-005-Distribution-Group.md
│   ├── Ticket-006-Mail-Forwarding.md
│   ├── Ticket-007-Missing-Email-Message-Trace.md
│   ├── Ticket-008-Quarantined-Message.md
│   ├── Ticket-009-External-Mail-Flow-Failure.md
│   └── Ticket-010-Outlook-Mailbox-Access-Issue.md
│
├── Screenshots/
│   ├── 01-Environment/
│   ├── 02-Mailboxes/
│   ├── 03-Shared-Mailboxes/
│   ├── 04-Groups/
│   ├── 05-Mail-Flow/
│   ├── 06-Message-Trace/
│   ├── 07-Security/
│   ├── 08-External-Mail-Flow/
│   └── 09-Outlook-Mailbox-Access/
│
├── Scripts/
└── Diagrams/
```

---

# Portfolio Highlights

This repository provides evidence of:

- **10 completed support incidents**
- **9 detailed technical documentation files**
- **96 organized screenshots**
- Live Exchange Online administration
- Real PowerShell command output
- Microsoft Defender quarantine investigation
- Exchange Online Message Trace
- Internal and external mail tests
- Root cause analysis
- Controlled fault simulation
- Targeted remediation
- End-to-end validation
- Enterprise-style incident documentation

---

# Project Outcome

This project demonstrates practical support experience across the Microsoft 365 messaging stack:

```text
Identity
   ↓
Licensing
   ↓
Mailbox Provisioning
   ↓
Recipient Configuration
   ↓
Permissions
   ↓
Mail Flow
   ↓
Message Trace
   ↓
Security / Quarantine
   ↓
External Routing
   ↓
Client Access
```

The support approach demonstrated throughout the project is:

> Verify the symptom, isolate the affected layer, gather evidence, identify the root cause, apply the smallest appropriate remediation, test the result, and document the resolution.

---

## Project Status

**10 / 10 Exchange Online support scenarios completed**

**Technical lab work complete**

**Portfolio documentation complete**
