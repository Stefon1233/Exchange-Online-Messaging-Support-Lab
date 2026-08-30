# INC-001 — User Mailbox Not Provisioned

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-001 |
| Requester | Brian Carter |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Email / Mailbox |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User exists in Microsoft 365 and Microsoft Entra ID but does not have an Exchange Online mailbox.

---

# User-Reported Issue

Brian Carter requires access to organizational email, but no Exchange Online mailbox is available for his account.

The user account exists successfully in Microsoft 365 and Microsoft Entra ID.

---

# Symptoms

The following symptoms were observed:

- Brian Carter appeared under Microsoft 365 Active Users.
- Brian Carter existed in Microsoft Entra ID.
- The Microsoft 365 account showed as unlicensed.
- Exchange Admin Center did not contain a mailbox for Brian Carter.
- Searching `Recipients > Mailboxes` returned no matching result.
- The user therefore had no provisioned Exchange Online mailbox.

---

# Initial Investigation

The user's identity was first verified through the Microsoft 365 Admin Center.

Brian Carter appeared under:

`Users > Active users`

Exchange Admin Center was then opened and the following location was reviewed:

`Recipients > Mailboxes`

A search was performed for:

`Brian Carter`

The Exchange Admin Center returned:

`No results found`

This confirmed that the Microsoft 365 identity existed but an Exchange Online mailbox had not been provisioned.

---

# Troubleshooting Performed

## Step 1 — Verify Microsoft 365 Account

The Microsoft 365 Admin Center was reviewed.

Brian Carter appeared as an active user.

This confirmed that the issue was not caused by a missing Microsoft 365 identity.

---

## Step 2 — Verify Microsoft Entra ID Identity

Microsoft Entra Admin Center was reviewed under:

`Identity > Users > All users`

Brian Carter was present.

This confirmed that the user identity existed in Microsoft Entra ID.

---

## Step 3 — Check Exchange Admin Center

Exchange Admin Center was opened.

Navigation:

`Recipients > Mailboxes`

A search for Brian Carter returned:

`No results found`

This confirmed that the account did not currently have an Exchange Online user mailbox.

---

## Step 4 — Review Microsoft 365 Licensing

Brian Carter's account was opened through:

`Microsoft 365 Admin Center > Users > Active users > Brian Carter > Licenses and apps`

The account displayed:

`Licenses (0)`

Microsoft 365 Business Basic licenses were available within the tenant.

However, Microsoft 365 Business Basic had not been assigned to Brian Carter.

---

## Step 5 — Identify Root Cause

The account had no Microsoft 365 license containing Exchange Online services.

Although Brian Carter existed as a valid Microsoft Entra ID identity, the account did not have the Exchange Online service entitlement required for mailbox provisioning.

This explained why Exchange Admin Center did not contain a mailbox for the user.

---

# Root Cause

**Missing Microsoft 365 license containing Exchange Online services.**

Brian Carter had been created as a Microsoft 365 and Microsoft Entra ID user, but Microsoft 365 Business Basic had not been assigned.

Without the Exchange Online entitlement, a user mailbox had not been provisioned.

---

# Resolution

Microsoft 365 Business Basic was assigned to Brian Carter through:

`Microsoft 365 Admin Center > Users > Active users > Brian Carter > Licenses and apps`

The following license was selected:

`Microsoft 365 Business Basic`

The configuration was saved successfully.

Microsoft 365 displayed:

`Your changes have been saved.`

Exchange Online subsequently provisioned a mailbox for Brian Carter.

---

# Exchange Admin Center Verification

After provisioning completed, Exchange Admin Center was reviewed again.

Navigation:

`Recipients > Mailboxes`

Brian Carter now appeared successfully.

The mailbox displayed:

- Display Name: Brian Carter
- Primary Email Address: brian.carter@Stefon.onmicrosoft.com
- Recipient Type: UserMailbox

This confirmed that mailbox provisioning had completed.

---

# PowerShell Verification

Exchange Online PowerShell was used to independently verify the resolution.

## PowerShell Environment

PowerShell version:

`PowerShell 7.6.3`

ExchangeOnlineManagement module version:

`3.10.1`

---

## Connect to Exchange Online

The Exchange Online session was established using:

`Connect-ExchangeOnline -ShowBanner:$false`

The connection was verified using:

`Get-ConnectionInformation | Format-Table UserPrincipalName,ConnectionId,State`

The resulting connection state was:

`Connected`

---

## Verify Brian Carter Mailbox

The following command was executed:

`Get-EXOMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

The result returned:

`DisplayName          : Brian Carter`

`PrimarySmtpAddress   : brian.carter@Stefon.onmicrosoft.com`

`RecipientTypeDetails : UserMailbox`

This independently confirmed that the Exchange Online mailbox existed and had been provisioned correctly.

---

## Verify Mailbox Inventory

A tenant-wide mailbox inventory was generated using:

`Get-EXOMailbox -ResultSize Unlimited | Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails | Sort-Object DisplayName | Format-Table -AutoSize`

Brian Carter appeared successfully in the mailbox inventory as:

- Brian Carter
- brian.carter@Stefon.onmicrosoft.com
- UserMailbox

---

# Resolution Verification

The incident was considered resolved after confirming all of the following:

- Brian Carter remained an active Microsoft 365 user.
- Brian Carter existed in Microsoft Entra ID.
- Microsoft 365 Business Basic was assigned.
- Exchange Admin Center displayed the mailbox.
- The recipient type was `UserMailbox`.
- Exchange Online PowerShell successfully located the mailbox.
- The primary SMTP address was configured correctly.
- Brian Carter appeared in the Exchange Online mailbox inventory.

---

# Closure Notes

User reported that an Exchange Online mailbox was unavailable.

Investigation confirmed that the Microsoft 365 identity existed but the account had no assigned Exchange-capable Microsoft 365 license.

Microsoft 365 Business Basic was assigned.

Exchange Online subsequently provisioned the mailbox.

Exchange Admin Center and Exchange Online PowerShell both confirmed successful provisioning.

The account now has a valid `UserMailbox`.

No further action required.

---

# Administrative Tools Used

- Microsoft 365 Admin Center
- Microsoft Entra Admin Center
- Exchange Admin Center
- Exchange Online
- PowerShell 7
- ExchangeOnlineManagement PowerShell Module
- Exchange Online PowerShell

---

# Commands Used

## Verify Exchange Online Module

`Get-Module ExchangeOnlineManagement -ListAvailable`

## Connect to Exchange Online

`Connect-ExchangeOnline -ShowBanner:$false`

## Verify Exchange Connection

`Get-ConnectionInformation | Format-Table UserPrincipalName,ConnectionId,State`

## Query Mailbox

`Get-EXOMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

## Generate Mailbox Inventory

`Get-EXOMailbox -ResultSize Unlimited | Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails | Sort-Object DisplayName | Format-Table -AutoSize`

---

# Screenshot Evidence

Supporting screenshots are stored under:

`Screenshots/02-Mailboxes/`

Evidence includes:

- `01-Brian-Carter-Mailbox-Missing.png`
- `02-Brian-Carter-Unlicensed.png`
- `03-Brian-Carter-License-Assigned.png`
- `04-Brian-Carter-Mailbox-Provisioned.png`
- `05-Brian-Carter-PowerShell-Verification.png`

---

# Skills Demonstrated

- Exchange Online Troubleshooting
- Microsoft 365 Administration
- Microsoft Entra ID
- Microsoft 365 Licensing
- Mailbox Provisioning
- Exchange Admin Center
- Exchange Online PowerShell
- Recipient Management
- Root Cause Analysis
- Incident Management
- ServiceNow-Style Ticket Documentation
- Resolution Verification
- Technical Documentation

---

# Final Status

**Resolved**

Brian Carter's missing mailbox was caused by a missing Microsoft 365 Business Basic license.

After the license was assigned, Exchange Online successfully provisioned the mailbox.

Exchange Admin Center and Exchange Online PowerShell independently confirmed that Brian Carter now has a valid `UserMailbox`.
