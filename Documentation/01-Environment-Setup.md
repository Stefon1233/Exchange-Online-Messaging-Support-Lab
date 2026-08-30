# 01 — Environment Setup

## Project Overview

The Exchange Online Messaging Support Lab is a hands-on Microsoft 365 support project designed to simulate common messaging incidents handled by Help Desk technicians, Microsoft 365 administrators, and cloud support specialists.

The lab focuses on troubleshooting Exchange Online rather than only performing administrative configuration.

Support scenarios are investigated using multiple Microsoft administration tools, including:

- Microsoft 365 Admin Center
- Microsoft Entra Admin Center
- Exchange Admin Center
- Exchange Online PowerShell
- Microsoft Defender security features where available

The project also uses ServiceNow-style incident documentation to demonstrate structured troubleshooting, root cause analysis, remediation, verification, and ticket closure.

---

# Lab Objectives

The primary objectives of this lab are to demonstrate practical experience with:

- Exchange Online administration
- Microsoft 365 messaging support
- Mailbox provisioning
- Microsoft 365 licensing
- Shared mailboxes
- Mailbox permissions and delegation
- Send As permissions
- Send on Behalf permissions
- Distribution groups
- Mail forwarding
- Mail flow troubleshooting
- Message trace
- Spam and quarantine investigation
- External mail delivery troubleshooting
- Outlook and mailbox access troubleshooting
- Exchange Online PowerShell
- Root cause analysis
- Incident management
- ServiceNow-style ticket documentation

---

# Environment

## Host System

The administrative workstation used for the lab is a macOS computer.

PowerShell 7 is installed locally and is used to connect remotely to Exchange Online.

### PowerShell Version

```text
PowerShell 7.6.3
```

---

# Microsoft 365 Environment

The lab uses a Microsoft 365 cloud tenant containing test users, groups, licenses, and Exchange Online mailboxes.

The environment provides access to the primary administrative portals required for messaging support.

---

# Administrative Platforms

## Microsoft 365 Admin Center

Microsoft 365 Admin Center is used for general tenant and user administration.

Primary tasks performed through this portal include:

- Reviewing active users
- Creating and managing users
- Reviewing account information
- Assigning Microsoft 365 licenses
- Removing licenses
- Reviewing available license capacity
- Managing Microsoft 365 service entitlements
- Troubleshooting account-level service availability

The Microsoft 365 Admin Center is especially important when troubleshooting Exchange mailbox provisioning because Exchange Online access depends on appropriate licensing.

---

## Microsoft Entra Admin Center

Microsoft Entra ID provides the identity and access management layer for the Microsoft 365 environment.

The Entra Admin Center is used to review:

- User identities
- User principal names
- Account status
- Group membership
- Administrative roles
- Authentication configuration
- Sign-in information
- Identity-related troubleshooting data

A Microsoft Entra identity may exist even when the associated user does not yet have an Exchange Online mailbox.

This distinction is important when troubleshooting mailbox provisioning incidents.

---

## Exchange Admin Center

Exchange Admin Center is the primary graphical administration interface used throughout this project.

The following Exchange Online areas are used:

- Recipients
- User mailboxes
- Shared mailboxes
- Groups
- Mailbox delegation
- Mail forwarding
- Mail flow
- Message trace
- Recipient configuration
- Exchange settings

Exchange Admin Center is used to investigate both configuration and messaging-related incidents.

---

# Exchange Online PowerShell

Exchange Online PowerShell provides command-line administration and troubleshooting capabilities.

PowerShell is used to independently verify information observed in the graphical Microsoft 365 administration portals.

The ExchangeOnlineManagement module was installed with:

```powershell
Install-Module ExchangeOnlineManagement -Scope CurrentUser
```

The module was imported using:

```powershell
Import-Module ExchangeOnlineManagement
```

The installed ExchangeOnlineManagement module version used during the lab was:

```text
3.10.1
```

The Exchange Online connection command was verified using:

```powershell
Get-Command Connect-ExchangeOnline
```

An authenticated Exchange Online session can be established using:

```powershell
Connect-ExchangeOnline -ShowBanner:$false
```

Connection status can be verified using:

```powershell
Get-ConnectionInformation |
Format-Table UserPrincipalName,ConnectionId,State
```

A successful connection returns:

```text
State: Connected
```

---

# Example Exchange Online PowerShell Commands

The following commands are used throughout the lab.

## Mailbox Inventory

```powershell
Get-EXOMailbox -ResultSize Unlimited |
Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails |
Sort-Object DisplayName |
Format-Table -AutoSize
```

## Individual Mailbox Lookup

```powershell
Get-EXOMailbox -Identity "user@tenant.onmicrosoft.com"
```

## Recipient Lookup

```powershell
Get-Recipient
```

## Mailbox Permissions

```powershell
Get-MailboxPermission
```

## Distribution Groups

```powershell
Get-DistributionGroup
```

## Distribution Group Members

```powershell
Get-DistributionGroupMember
```

These commands provide administrative evidence that can be compared against Exchange Admin Center configuration.

---

# Lab Architecture

```text
                       Microsoft 365 Tenant
                               |
             +-----------------+-----------------+
             |                 |                 |
             |                 |                 |
     Microsoft Entra ID   Microsoft 365    Exchange Online
             |             Admin Center           |
             |                 |                  |
       +-----+-----+           |          +-------+-------+
       |           |           |          |               |
     Users       Groups     Licensing   Mailboxes       Mail Flow
                                          |
                              +-----------+-----------+
                              |                       |
                       User Mailboxes          Shared Mailboxes
                              |                       |
                              +-----------+-----------+
                                          |
                                    Delegation
                                          |
                          +---------------+---------------+
                          |               |               |
                     Full Access       Send As      Send on Behalf
                                          |
                                 Exchange PowerShell
```

---

# Initial Environment Validation

Before beginning the Exchange support scenarios, the environment was validated.

The following services were confirmed accessible:

- Microsoft 365 Admin Center
- Microsoft Entra Admin Center
- Exchange Admin Center
- Exchange Online mailbox management
- Exchange Online group management
- Microsoft 365 licensing
- Exchange Online PowerShell

---

# Microsoft 365 User Validation

The Microsoft 365 Active Users page was reviewed to confirm that multiple test identities were available for troubleshooting scenarios.

The environment contains users representing different departments and support situations.

These test accounts allow messaging incidents to be created and resolved without affecting production users.

---

# License Validation

Microsoft 365 licensing was reviewed before beginning the troubleshooting scenarios.

Microsoft 365 Business Basic licenses were available within the tenant.

This license provides Microsoft 365 cloud services that can include Exchange Online capabilities.

Available license capacity allows the lab to demonstrate:

- License assignment
- Missing license troubleshooting
- Service provisioning
- License restoration
- Exchange mailbox provisioning

---

# Exchange Mailbox Validation

Exchange Admin Center was reviewed under:

```text
Recipients > Mailboxes
```

Existing Exchange Online user mailboxes were visible.

This confirmed that Exchange Online was active and operational in the tenant.

Mailbox information available through Exchange Admin Center includes:

- Display name
- Primary email address
- Recipient type
- Mailbox details
- Mailbox usage
- Delegation
- Forwarding
- Additional Exchange settings

---

# Exchange Group Validation

Exchange Admin Center was also reviewed under:

```text
Recipients > Groups
```

The groups management interface was accessible.

This area will be used later to administer and troubleshoot:

- Distribution groups
- Microsoft 365 groups
- Mail-enabled groups
- Group owners
- Group members
- Message delivery restrictions

---

# Microsoft Entra Validation

The Entra Admin Center was reviewed under:

```text
Identity > Users > All users
```

Test identities were visible and available for Exchange Online support scenarios.

Microsoft Entra ID will be used when troubleshooting incidents involving:

- User identity
- Account status
- User principal names
- Group membership
- Access configuration
- Authentication

---

# PowerShell Validation

PowerShell 7 was successfully launched from macOS.

The ExchangeOnlineManagement module was installed and imported successfully.

The following command confirmed that the Exchange Online connection function was available:

```powershell
Get-Command Connect-ExchangeOnline
```

The returned command showed:

```text
CommandType : Function
Name        : Connect-ExchangeOnline
Version     : 3.10.1
Source      : ExchangeOnlineManagement
```

An Exchange Online session was then established successfully.

Connection verification returned:

```text
State: Connected
```

This confirmed that the workstation could perform remote Exchange Online administration through PowerShell.

---

# Support Workflow

Each support scenario in this project follows a structured troubleshooting process.

## 1. Receive the Incident

A realistic user or administrator issue is simulated.

## 2. Document Symptoms

The reported behavior and observable symptoms are recorded.

## 3. Verify User Identity

Microsoft 365 and Microsoft Entra ID are reviewed to confirm the correct account.

## 4. Review Licensing

Microsoft 365 service entitlements are checked when relevant.

## 5. Investigate Exchange Configuration

Exchange Admin Center is used to inspect mailboxes, groups, permissions, forwarding, and mail flow.

## 6. Perform PowerShell Diagnostics

Exchange Online PowerShell is used when additional verification or administrative detail is needed.

## 7. Identify Root Cause

The underlying cause of the issue is documented.

## 8. Implement Remediation

The appropriate configuration or administrative correction is performed.

## 9. Verify Resolution

The service is tested or independently verified.

## 10. Document the Incident

The troubleshooting process is recorded as a ServiceNow-style help desk ticket.

---

# Planned Support Scenarios

The project contains approximately ten Exchange Online support scenarios.

---

## Scenario 1 — User Mailbox Not Provisioned

Investigate a Microsoft 365 user who exists in Microsoft Entra ID but does not have an Exchange Online mailbox.

Primary skills:

- User verification
- License investigation
- Mailbox provisioning
- PowerShell verification

---

## Scenario 2 — Missing Exchange Online License

Investigate an Exchange Online service problem caused by a missing or incorrectly configured Microsoft 365 license.

Primary skills:

- License troubleshooting
- Service entitlement validation
- Exchange Online administration

---

## Scenario 3 — Shared Mailbox Access

Troubleshoot a user who requires access to a shared mailbox.

Primary skills:

- Shared mailboxes
- Full Access
- Delegation
- Permission troubleshooting

---

## Scenario 4 — Send As vs Send on Behalf

Configure and validate different Exchange mailbox delegation permissions.

Primary skills:

- Send As
- Send on Behalf
- Mailbox delegation
- Permission verification

---

## Scenario 5 — Distribution Group Membership

Investigate mail delivery issues involving distribution group configuration.

Primary skills:

- Distribution groups
- Membership
- Owners
- Recipient troubleshooting

---

## Scenario 6 — Mail Forwarding

Configure and investigate Exchange Online forwarding.

Primary skills:

- Mailbox forwarding
- Delivery configuration
- Mail flow troubleshooting

---

## Scenario 7 — Missing Email and Message Trace

Investigate a message reported as missing.

Primary skills:

- Message trace
- Sender verification
- Recipient verification
- Delivery status investigation
- Mail flow analysis

---

## Scenario 8 — Quarantined or Spam Message

Investigate a message affected by Microsoft 365 security controls.

Primary skills:

- Quarantine
- Spam investigation
- Message security
- Microsoft Defender

---

## Scenario 9 — External Mail Flow

Troubleshoot messaging between the Microsoft 365 tenant and external email systems.

Primary skills:

- External email
- Mail flow
- Delivery troubleshooting
- Message tracing

---

## Scenario 10 — Outlook or Mailbox Access Issue

Investigate a user who has an Exchange Online mailbox but cannot access email successfully.

Primary skills:

- Account verification
- Mailbox validation
- Outlook troubleshooting
- Exchange Online diagnostics

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
│   ├── 07-Spam-Quarantine.md
│   ├── 08-Exchange-PowerShell.md
│   └── 09-Troubleshooting.md
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
│   ├── Ticket-009-External-Mail-Flow.md
│   └── Ticket-010-Outlook-Mailbox-Access.md
│
├── Scripts/
│   ├── Get-Mailbox-Inventory.ps1
│   ├── Get-Mailbox-Permissions.ps1
│   ├── Get-Recipient-Inventory.ps1
│   └── Exchange-Health-Check.ps1
│
├── Screenshots/
│   ├── 01-Environment/
│   ├── 02-Mailboxes/
│   ├── 03-Shared-Mailboxes/
│   ├── 04-Groups/
│   ├── 05-Mail-Flow/
│   ├── 06-Message-Trace/
│   ├── 07-Security/
│   ├── 08-PowerShell/
│   └── 09-Troubleshooting/
│
└── Diagrams/
```

---

# Screenshot Strategy

Screenshots are used to provide evidence of important troubleshooting stages rather than capturing every administrative click.

The environment screenshot set includes:

- Microsoft 365 Active Users
- Microsoft 365 Licenses
- Exchange Online Mailboxes
- Exchange Online Groups
- Microsoft Entra Users

Scenario-specific screenshots are stored in the corresponding screenshot directory.

Sensitive information such as passwords, authentication tokens, MFA codes, and unnecessary administrative details should not be included in the public repository.

---

# Environment Screenshot Evidence

Initial environment screenshots are stored under:

```text
Screenshots/01-Environment/
```

Recommended filenames:

```text
01-Microsoft-365-Active-Users.png
02-Microsoft-365-Licenses.png
03-Exchange-Mailboxes.png
04-Exchange-Groups.png
05-Entra-Users.png
```

---

# Administrative Tools Used

- Microsoft 365 Admin Center
- Microsoft Entra Admin Center
- Exchange Admin Center
- Exchange Online
- PowerShell 7
- ExchangeOnlineManagement PowerShell Module
- Exchange Online PowerShell
- Microsoft Defender features where available
- Git
- GitHub

---

# Skills Demonstrated

The lab environment supports hands-on practice with:

- Exchange Online
- Microsoft 365 Administration
- Microsoft Entra ID
- Identity and Access Management
- Mailbox Administration
- Microsoft 365 Licensing
- Exchange Online Licensing
- Shared Mailboxes
- Mailbox Delegation
- Distribution Groups
- Mail Forwarding
- Mail Flow
- Message Trace
- Email Security
- Exchange Online PowerShell
- PowerShell
- Outlook Troubleshooting
- Incident Management
- Root Cause Analysis
- Technical Troubleshooting
- Technical Documentation
- ServiceNow-Style Ticket Documentation

---

# Environment Validation Result

**Status: Ready**

The Microsoft 365 tenant, Exchange Admin Center, Microsoft Entra Admin Center, licensing environment, Exchange Online mailboxes, Exchange groups, and Exchange Online PowerShell connectivity were successfully validated.

The environment is ready for Exchange Online messaging support scenarios.
