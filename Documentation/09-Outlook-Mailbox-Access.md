# Exchange Online Messaging Support Lab

Hands-on Microsoft 365 and Exchange Online support lab demonstrating mailbox administration, licensing troubleshooting, shared mailbox permissions, distribution groups, mail forwarding, message trace, Microsoft Defender quarantine, external mail flow, Outlook 
access troubleshooting, and Exchange Online PowerShell.

---

## Project Overview

This project simulates real-world Exchange Online incidents commonly handled by:

- Help Desk Technicians
- IT Support Specialists
- Microsoft 365 Support Engineers
- Desktop Support Technicians
- Junior Microsoft 365 Administrators
- Cloud Support Specialists

Rather than documenting only administrative configuration, this lab focuses on a complete troubleshooting lifecycle:

1. Reproduce the user-reported issue
2. Establish a baseline
3. Gather evidence
4. Investigate through Microsoft administrative portals
5. Validate findings with PowerShell
6. Identify the root cause
7. Apply remediation
8. Perform end-to-end testing
9. Document the incident using ServiceNow-style tickets
10. Preserve screenshots and technical evidence

The lab contains **10 completed Exchange Online support scenarios** covering identity, licensing, mailbox permissions, mail flow, security, message tracing, external delivery, and Outlook access.

---

# Technologies and Platforms

## Microsoft 365

- Microsoft 365 Admin Center
- Microsoft Entra ID
- Exchange Online
- Exchange Admin Center
- Outlook on the web
- Microsoft Defender

## Administration and Automation

- PowerShell 7
- ExchangeOnlineManagement PowerShell Module
- Exchange Online PowerShell
- Git
- GitHub

## Troubleshooting Features

- Message Trace
- Mail Flow Rules
- Transport Rules
- Microsoft Defender Quarantine
- Mailbox Delegation
- Distribution Groups
- Mailbox Forwarding
- Client Access Settings
- Microsoft 365 Licensing
- Exchange Service Plans

---

# Environment

The lab was completed using a Microsoft 365 cloud tenant containing test identities, Exchange Online mailboxes, shared mailboxes, groups, licensing, and Microsoft Defender functionality.

Administrative access was performed from macOS using a web browser and PowerShell 7.

## PowerShell Environment

PowerShell version used:

```text
PowerShell 7.6.3
```

Exchange Online Management module:

```text
ExchangeOnlineManagement 3.10.1
```

Exchange Online connectivity was established using:

```powershell
Connect-ExchangeOnline -ShowBanner:$false
```

---

# Lab Architecture

```text
                         Microsoft 365 Tenant
                                  |
             +--------------------+--------------------+
             |                    |                    |
             |                    |                    |
       Microsoft Entra ID   Microsoft 365       Exchange Online
             |               Admin Center              |
             |                    |                    |
       +-----+------+          Licensing        +------+------+
       |            |                             |             |
     Users        Groups                       Mailboxes      Mail Flow
                                                  |             |
                                 +----------------+-------------+----------------+
                                 |                |             |                |
                             User Mailbox     Shared Mailbox  Message Trace   Transport Rules
                                 |                |                              |
                                 |          +-----+------+                       |
                                 |          |     |      |                       |
                                 |        Full  Send   Send on                   |
                                 |       Access   As    Behalf                    |
                                 |                                               |
                                 +----------------+------------------------------+
                                                  |
                                          Microsoft Defender
                                                  |
                                              Quarantine
```

---

# Support Scenarios

## Scenario 1 — User Mailbox Not Provisioned

**User:** Brian Carter

### Problem

Brian Carter existed in Microsoft 365 and Microsoft Entra ID but did not have an Exchange Online mailbox.

### Investigation

- Verified Brian existed as an active Microsoft 365 user
- Confirmed the identity existed in Microsoft Entra ID
- Searched Exchange Admin Center
- Exchange mailbox returned no result
- Reviewed Microsoft 365 licensing
- Brian had zero assigned licenses

### Root Cause

Brian Carter did not have a Microsoft 365 license containing Exchange Online services.

### Resolution

Assigned:

```text
Microsoft 365 Business Basic
```

Exchange Online subsequently provisioned the mailbox.

### Verification

PowerShell confirmed:

```text
DisplayName          : Brian Carter
PrimarySmtpAddress   : brian.carter@Stefon.onmicrosoft.com
RecipientTypeDetails : UserMailbox
```

### Documentation

[Mailbox Provisioning and Licensing](Documentation/02-Mailbox-Provisioning-Licensing.md)

### Ticket

[Ticket 001 — Mailbox Not Provisioned](Help-Desk-Tickets/Ticket-001-Mailbox-Not-Provisioned.md)

---

# Scenario 2 — Missing Exchange Online Service Entitlement

**User:** Emily Brown

### Problem

Emily Brown still had Microsoft 365 Business Basic assigned but her Exchange Online mailbox became unavailable.

### Investigation

The parent Microsoft 365 license remained assigned.

However:

```text
Exchange Online (Plan 1)
```

was disabled inside the license.

Exchange Admin Center no longer showed Emily's active mailbox.

PowerShell could not locate the active Exchange mailbox or recipient.

### Root Cause

The Exchange Online service plan was disabled while the parent Microsoft 365 product license remained assigned.

### Resolution

Re-enabled:

```text
Exchange Online (Plan 1)
```

### Verification

Exchange Admin Center restored Emily's mailbox and PowerShell returned:

```text
RecipientTypeDetails : UserMailbox
```

### Documentation

[Mailbox Provisioning and Licensing](Documentation/02-Mailbox-Provisioning-Licensing.md)

### Ticket

[Ticket 002 — Missing Exchange License](Help-Desk-Tickets/Ticket-002-Missing-Exchange-License.md)

---

# Scenario 3 — Shared Mailbox Access

**User:** Brian Carter  
**Shared Mailbox:** Company Shared

### Problem

Brian needed access to the Company Shared mailbox but could not open or manage it.

### Investigation

The Company Shared mailbox existed as:

```text
SharedMailbox
```

Mailbox delegation showed that Brian did not have Full Access.

PowerShell also returned no Full Access permission for Brian.

### Root Cause

Missing mailbox delegation.

### Resolution

Added Brian Carter under:

```text
Full Access
```

### Verification

PowerShell returned:

```text
AccessRights : {FullAccess}
IsInherited  : False
Deny         : False
```

### Documentation

[Shared Mailboxes and Permissions](Documentation/03-Shared-Mailboxes-Permissions.md)

### Ticket

[Ticket 003 — Shared Mailbox Access](Help-Desk-Tickets/Ticket-003-Shared-Mailbox-Access.md)

---

# Scenario 4 — Send As vs Send on Behalf

**Shared Mailbox:** Company Shared

### Objective

Demonstrate the difference between three Exchange mailbox permissions:

- Full Access
- Send As
- Send on Behalf

### Brian Carter — Send As

Brian was assigned:

```text
Send As
```

PowerShell confirmed:

```text
AccessRights : {SendAs}
```

A live Outlook test was performed.

The recipient saw the sender as:

```text
Company Shared
```

Brian's personal identity was not displayed.

### Emily Brown — Send on Behalf

Emily was assigned:

```text
Send on Behalf
```

PowerShell resolved Emily under:

```text
GrantSendOnBehalfTo
```

A live Outlook test was performed.

The recipient saw:

```text
Emily Brown on behalf of Company Shared
```

### Key Difference

| Permission | Result |
|---|---|
| Full Access | Open and manage mailbox contents |
| Send As | Message appears directly from shared mailbox |
| Send on Behalf | Delegate identity remains visible |

### Documentation

[Shared Mailboxes and Permissions](Documentation/03-Shared-Mailboxes-Permissions.md)

### Ticket

[Ticket 004 — Send As Permission](Help-Desk-Tickets/Ticket-004-Send-As-Permission.md)

---

# Scenario 5 — Distribution List Membership

**Distribution List:** Finance Department  
**Affected User:** Brian Carter

### Problem

Brian was not receiving messages sent to the Finance Department distribution list.

### Investigation

Exchange Admin Center showed existing Finance Department members:

- Emily Brown
- Robert Garcia

Brian was not included.

PowerShell verification with:

```powershell
Get-DistributionGroupMember
```

also returned no Brian Carter membership.

### Root Cause

Brian was not a member of the Finance Department distribution list.

### Resolution

Added Brian Carter to the distribution list.

### Verification

PowerShell returned:

```text
Brian Carter
brian.carter@Stefon.onmicrosoft.com
UserMailbox
```

A live message sent to the Finance Department distribution list successfully arrived in Brian's Inbox.

### Documentation

[Distribution Groups](Documentation/04-Distribution-Groups.md)

### Ticket

[Ticket 005 — Distribution Group](Help-Desk-Tickets/Ticket-005-Distribution-Group.md)

---

# Scenario 6 — Unintended Mailbox Forwarding

**Affected User:** Robert Garcia  
**Unexpected Destination:** Brian Carter

### Problem

Robert reported that expected messages were not appearing in his mailbox.

### Investigation

Exchange mailbox forwarding was configured to Brian Carter.

PowerShell showed:

```text
DeliverToMailboxAndForward : False
```

The forwarding target was resolved through PowerShell as:

```text
Brian Carter
brian.carter@Stefon.onmicrosoft.com
```

### Controlled Test

Emily sent a message directly to Robert.

The message:

- Arrived in Brian's mailbox
- Was addressed to Robert
- Did not remain in Robert's mailbox

### Root Cause

Mailbox-level forwarding redirected incoming mail to Brian without retaining a copy in Robert's mailbox.

### Resolution

Disabled forwarding.

### Verification

PowerShell showed:

```text
ForwardingAddress     :
ForwardingSmtpAddress :
```

A new test message successfully arrived in Robert's Inbox.

### Documentation

[Mail Forwarding and Mail Flow](Documentation/05-Mail-Forwarding-Mail-Flow.md)

### Ticket

[Ticket 006 — Mail Forwarding](Help-Desk-Tickets/Ticket-006-Mail-Forwarding.md)

---

# Scenario 7 — Missing Email / Message Trace

**Affected User:** Brian Carter  
**Sender:** Emily Brown

### Problem

Brian reported that a specific email never appeared in his Inbox.

### Investigation

Exchange Online Message Trace showed:

```text
Status : Delivered
```

Detailed trace information showed that the message had been delivered to:

```text
Deleted Items
```

PowerShell trace events confirmed:

- Receive
- Submit
- Deliver

Mailbox rule investigation identified:

```text
LAB - Message Trace Test
```

The rule matched the message subject and deleted the message.

### Root Cause

An enabled Inbox rule moved matching messages away from Brian's Inbox.

### Resolution

Disabled the problematic Inbox rule using:

```powershell
Disable-InboxRule
```

### Verification

PowerShell showed:

```text
Enabled : False
```

A new controlled message appeared directly in Brian's Inbox.

### Documentation

[Message Trace](Documentation/06-Message-Trace.md)

### Ticket

[Ticket 007 — Missing Email / Message Trace](Help-Desk-Tickets/Ticket-007-Missing-Email-Message-Trace.md)

---

# Scenario 8 — Quarantined Message Investigation

**Affected User:** Brian Carter  
**Sender:** Emily Brown

### Problem

An expected message did not appear in Brian's Inbox.

### Controlled Security Rule

A transport rule was configured:

```text
LAB - Quarantine Test
```

Condition:

```text
Subject includes LAB QUARANTINE TEST
```

Action:

```text
Deliver the message to the hosted quarantine
```

### Investigation

Message Trace returned:

```text
Quarantined
```

Detailed trace identified:

```text
LAB - Quarantine Test
```

Microsoft Defender showed:

- Quarantine reason: Transport Rule
- Policy type: Exchange transport rule
- Policy name: LAB - Quarantine Test
- Delivery action: Blocked

PowerShell showed:

```text
ReleaseStatus : NOTRELEASED
```

### Resolution

The message was reviewed and released through Microsoft Defender.

PowerShell then showed:

```text
ReleaseStatus : RELEASED
```

The controlled transport rule was disabled.

PowerShell confirmed:

```text
State : Disabled
```

### Verification

A new test message successfully arrived directly in Brian's Inbox.

### Documentation

[Quarantine and Email Security](Documentation/07-Quarantine-Security.md)

### Ticket

[Ticket 008 — Quarantined Message](Help-Desk-Tickets/Ticket-008-Quarantined-Message.md)

---

# Scenario 9 — External Mail-Flow Failure

**Sender:** Emily Brown  
**External Recipient:** Gmail

### Baseline

A normal external email from Exchange Online successfully arrived in Gmail.

This established that:

- External mail routing worked
- Recipient address was valid
- Microsoft 365 outbound delivery worked

### Controlled Failure

A transport rule was created:

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

Emily received an NDR stating:

```text
Blocked by mail flow rule
```

### Message Trace

Exchange Admin Center returned:

```text
Status : Failed
```

Detailed trace identified:

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

PowerShell confirmed:

```text
State : Disabled
```

### Verification

A new message successfully reached Gmail.

PowerShell reported:

```text
Status : Delivered
```

Detailed events included:

```text
Send external
```

### Documentation

[External Mail Flow](Documentation/08-External-Mail-Flow.md)

### Ticket

[Ticket 009 — External Mail Flow Failure](Help-Desk-Tickets/Ticket-009-External-Mail-Flow-Failure.md)

---

# Scenario 10 — Outlook on the Web Access Issue

**Affected User:** Brian Carter

### Problem

Brian could not open Outlook on the web.

A new browser session returned an Outlook error including:

```text
Error: 440
```

### Investigation

Brian's Exchange Online mailbox still existed as:

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

Emily sent Brian:

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

This proved Brian's mailbox remained healthy even though Outlook on the web access was unavailable.

### Root Cause

Outlook on the web had been specifically disabled through the Exchange Client Access configuration.

### Resolution

Executed:

```powershell
Set-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" -OWAEnabled $true
```

### Verification

PowerShell returned:

```text
OWAEnabled : True
```

Brian successfully opened Outlook on the web again.

The message delivered while OWA was disabled was present in his Inbox.

### Documentation

[Outlook and Mailbox Access](Documentation/09-Outlook-Mailbox-Access.md)

### Ticket

[Ticket 010 — Outlook Mailbox Access](Help-Desk-Tickets/Ticket-010-Outlook-Mailbox-Access-Issue.md)

---

# Documentation

The repository includes detailed technical documentation for each major Exchange Online support area.

| Document | Topic |
|---|---|
| [01-Environment-Setup.md](Documentation/01-Environment-Setup.md) | Tenant, administrative tools, PowerShell, architecture |
| [02-Mailbox-Provisioning-Licensing.md](Documentation/02-Mailbox-Provisioning-Licensing.md) | Mailbox provisioning and service-plan troubleshooting |
| [03-Shared-Mailboxes-Permissions.md](Documentation/03-Shared-Mailboxes-Permissions.md) | Shared mailbox access and delegation |
| [04-Distribution-Groups.md](Documentation/04-Distribution-Groups.md) | Distribution list administration |
| [05-Mail-Forwarding-Mail-Flow.md](Documentation/05-Mail-Forwarding-Mail-Flow.md) | Forwarding and mail flow |
| [06-Message-Trace.md](Documentation/06-Message-Trace.md) | Missing email and message tracing |
| [07-Quarantine-Security.md](Documentation/07-Quarantine-Security.md) | Microsoft Defender and quarantine |
| [08-External-Mail-Flow.md](Documentation/08-External-Mail-Flow.md) | External mail troubleshooting |
| [09-Outlook-Mailbox-Access.md](Documentation/09-Outlook-Mailbox-Access.md) | Outlook and Client Access troubleshooting |

---

# Help Desk Incident Portfolio

Ten ServiceNow-style support incidents are included.

| Incident | Scenario |
|---|---|
| [INC-001](Help-Desk-Tickets/Ticket-001-Mailbox-Not-Provisioned.md) | User mailbox not provisioned |
| [INC-002](Help-Desk-Tickets/Ticket-002-Missing-Exchange-License.md) | Missing Exchange service entitlement |
| [INC-003](Help-Desk-Tickets/Ticket-003-Shared-Mailbox-Access.md) | Shared mailbox access |
| [INC-004](Help-Desk-Tickets/Ticket-004-Send-As-Permission.md) | Send As vs Send on Behalf |
| [INC-005](Help-Desk-Tickets/Ticket-005-Distribution-Group.md) | Distribution group membership |
| [INC-006](Help-Desk-Tickets/Ticket-006-Mail-Forwarding.md) | Mailbox forwarding |
| [INC-007](Help-Desk-Tickets/Ticket-007-Missing-Email-Message-Trace.md) | Missing email / Message Trace |
| [INC-008](Help-Desk-Tickets/Ticket-008-Quarantined-Message.md) | Quarantine investigation |
| [INC-009](Help-Desk-Tickets/Ticket-009-External-Mail-Flow-Failure.md) | External mail failure |
| [INC-010](Help-Desk-Tickets/Ticket-010-Outlook-Mailbox-Access-Issue.md) | Outlook on the web access |

Each ticket documents:

- User-reported symptoms
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

The repository contains **96 organized screenshots** providing evidence of the troubleshooting process.

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

Screenshots document:

- Baseline configurations
- User symptoms
- Exchange Admin Center settings
- Microsoft 365 licensing
- Exchange Online PowerShell
- Message Trace
- Microsoft Defender
- Quarantine
- Non-delivery reports
- Outlook testing
- Final resolution verification

---

# Exchange Online PowerShell Commands Demonstrated

## Exchange Connection

```powershell
Connect-ExchangeOnline
Get-ConnectionInformation
```

## Mailboxes

```powershell
Get-EXOMailbox
Get-Mailbox
```

## Client Access

```powershell
Get-CASMailbox
Set-CASMailbox
```

## Permissions

```powershell
Get-MailboxPermission
Get-RecipientPermission
```

## Recipients

```powershell
Get-Recipient
```

## Distribution Groups

```powershell
Get-DistributionGroup
Get-DistributionGroupMember
```

## Message Trace

```powershell
Get-MessageTraceV2
Get-MessageTraceDetailV2
```

## Inbox Rules

```powershell
Get-InboxRule
Disable-InboxRule
```

## Mail Flow Rules

```powershell
Get-TransportRule
```

## Microsoft Defender Quarantine

```powershell
Get-QuarantineMessage
```

## Exchange Domains and Routing

```powershell
Get-AcceptedDomain
Get-RemoteDomain
Get-OutboundConnector
```

---

# Troubleshooting Methodology

The project consistently follows a structured troubleshooting approach.

## 1. Establish Scope

Determine:

- Who is affected?
- Which service is affected?
- Is the issue isolated or widespread?

## 2. Verify Identity

Check:

- Microsoft 365 user
- Microsoft Entra ID identity
- User principal name
- Account status

## 3. Verify Licensing

Check:

- Product license
- Exchange Online service plan
- Available licenses

## 4. Verify Exchange Objects

Check:

- Mailbox existence
- Recipient type
- SMTP address
- Group membership
- Shared mailbox configuration

## 5. Review Exchange Configuration

Check:

- Delegation
- Forwarding
- Distribution groups
- Transport rules
- Inbox rules
- Client Access settings

## 6. Use Message Trace

Determine whether a message was:

- Delivered
- Failed
- Quarantined
- Rejected
- Redirected

## 7. Use PowerShell

Independently verify graphical administration findings.

## 8. Apply Remediation

Correct only the configuration responsible for the incident.

## 9. Test End-to-End

Perform controlled user-facing validation.

## 10. Document Resolution

Record:

- Root cause
- Administrative action
- Commands
- Evidence
- Validation
- Closure notes

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
- Mail Flow
- Message Trace
- Transport Rules
- External Mail Routing
- Client Access Settings

## Microsoft 365

- Microsoft 365 Admin Center
- Microsoft Entra ID
- Microsoft 365 Licensing
- Service Plan Administration
- Identity Troubleshooting

## Security

- Microsoft Defender
- Quarantine Investigation
- Transport Rule Security
- Message Release
- Email Policy Troubleshooting

## PowerShell

- ExchangeOnlineManagement
- Exchange Online PowerShell
- Mailbox Queries
- Permission Queries
- Recipient Queries
- Message Trace Queries
- Transport Rule Queries
- Client Access Administration

## Support Operations

- Help Desk Troubleshooting
- Incident Management
- ServiceNow-Style Documentation
- Root Cause Analysis
- User Impact Analysis
- Technical Documentation
- End-to-End Testing
- Resolution Verification

---

# Key Lessons Learned

## Licensing Has Multiple Layers

A user may appear to have a Microsoft 365 license while an individual service such as Exchange Online remains disabled.

Troubleshooting should verify both:

- Product license
- Service plan

---

## Mailbox Access and Sending Are Different Permissions

Full Access does not automatically provide Send As or Send on Behalf.

Each permission should be granted based on the user's business requirement.

---

## Message Trace Is Essential

A user reporting that an email is missing does not prove mail transport failed.

Message Trace can quickly determine whether Exchange Online:

- Delivered
- Failed
- Rejected
- Quarantined

the message.

---

## Delivered Does Not Mean Visible in Inbox

A delivered message can still be affected by:

- Inbox rules
- Deleted Items
- Junk Email
- Forwarding
- Client filtering

Transport troubleshooting and mailbox troubleshooting are separate layers.

---

## External Failures May Originate Internally

An external message failure does not necessarily mean Gmail or another remote provider rejected it.

Exchange transport rules and organizational policies can stop messages before they leave Microsoft 365.

---

## Outlook Access Is Separate From Mailbox Health

A valid Exchange Online mailbox can continue receiving mail while Outlook on the web is unavailable.

Client access configuration should be investigated separately from mailbox provisioning and transport.

---

# Repository Structure

```text
Exchange-Online-Messaging-Support-Lab/
│
├── README.md
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
├── Scripts/
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
├── Diagrams/
└── .gitignore
```

---

# Portfolio Highlights

This repository demonstrates more than configuration screenshots.

It provides evidence of:

- **10 completed support incidents**
- **9 detailed technical documentation files**
- **96 organized screenshots**
- Live Exchange Online administration
- Real PowerShell command output
- Microsoft Defender quarantine investigation
- Real Message Trace results
- Live internal and external email tests
- Root cause analysis
- Controlled fault simulation
- Remediation
- End-to-end validation
- Enterprise-style incident documentation

---

# Project Outcome

The Exchange Online Messaging Support Lab demonstrates practical experience supporting Microsoft 365 messaging services across multiple layers:

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

The project emphasizes a support mindset:

> Verify the symptom, isolate the affected layer, gather evidence, identify the root cause, apply the smallest appropriate remediation, test the result, and document the resolution.

---

## Status

**10 / 10 support scenarios completed**

**Project technical work: Complete**

Final repository validation and GitHub publishing remain.
