# 02 — Mailbox Provisioning and Licensing

## Overview

This section documents Exchange Online mailbox provisioning and Microsoft 365 licensing troubleshooting.

A Microsoft 365 user can exist successfully in Microsoft Entra ID without having an Exchange Online mailbox. Mailbox provisioning depends on the user receiving an appropriate Microsoft 365 license that includes Exchange Online services.

This scenario demonstrates how to investigate a missing mailbox, identify a licensing-related root cause, remediate the issue, and verify successful provisioning through both the Exchange Admin Center and Exchange Online PowerShell.

---

# Scenario 1 — User Mailbox Not Provisioned

## Incident Summary

**User:** Brian Carter  
**Service:** Exchange Online  
**Issue:** User existed in Microsoft 365 and Microsoft Entra ID but did not have an Exchange Online mailbox.

Brian Carter was available as an active Microsoft 365 user and Microsoft Entra ID identity. However, Exchange Admin Center did not contain a mailbox for the account.

The issue was investigated using:

- Microsoft 365 Admin Center
- Microsoft Entra Admin Center
- Exchange Admin Center
- Exchange Online PowerShell

---

## Initial Symptoms

The following symptoms were observed:

- Brian Carter existed under Microsoft 365 Active Users.
- Brian Carter existed in Microsoft Entra ID.
- Microsoft 365 displayed the account as **Unlicensed**.
- Exchange Admin Center did not contain a mailbox for Brian Carter.
- Searching `Recipients > Mailboxes` for Brian Carter returned `No results found`.

These symptoms demonstrated that the user identity existed successfully, but Exchange Online mailbox provisioning had not occurred.

---

## Initial Exchange Admin Center Investigation

Exchange Admin Center was opened and the following location was reviewed:

`Recipients > Mailboxes`

A search was performed for:

`Brian Carter`

The Exchange Admin Center returned:

`No results found`

The mailbox list showed zero matching recipients.

This confirmed that Brian Carter did not currently have an Exchange Online user mailbox.

---

## Microsoft 365 License Investigation

Brian Carter was reviewed through:

`Microsoft 365 Admin Center > Users > Active users > Brian Carter > Licenses and apps`

The account showed:

`Licenses (0)`

The following Microsoft 365 license was available within the tenant but was not assigned:

`Microsoft 365 Business Basic`

The tenant had available Microsoft 365 Business Basic licenses, confirming that lack of license capacity was not preventing assignment.

Because Microsoft 365 Business Basic includes Exchange Online services, the missing license was identified as the likely reason that a mailbox had not been provisioned.

---

## Root Cause

The root cause was a missing Microsoft 365 license containing Exchange Online services.

Brian Carter had been created successfully as a Microsoft 365 and Microsoft Entra ID user, but Microsoft 365 Business Basic had not been assigned.

Without an Exchange Online service entitlement, Exchange Online had not provisioned a user mailbox.

---

# Remediation

## License Assignment

The following administrative path was used:

`Microsoft 365 Admin Center > Users > Active users > Brian Carter > Licenses and apps`

The following license was selected:

`Microsoft 365 Business Basic`

The configuration was saved successfully.

Microsoft 365 displayed:

`Your changes have been saved.`

Brian Carter then displayed Microsoft 365 Business Basic as the assigned license.

---

## Exchange Online Provisioning

After the license was assigned, Exchange Online automatically provisioned a mailbox for the account.

The provisioning result was verified through both the Exchange Admin Center and Exchange Online PowerShell.

---

# Exchange Admin Center Verification

Exchange Admin Center was reopened and the following location was reviewed:

`Recipients > Mailboxes`

Brian Carter now appeared in the mailbox list.

The mailbox displayed:

- **Display Name:** Brian Carter
- **Email Address:** brian.carter@Stefon.onmicrosoft.com
- **Recipient Type:** UserMailbox

This provided graphical verification that Exchange Online provisioning had completed successfully.

---

# Exchange Online PowerShell Verification

## PowerShell Environment

Exchange Online was also verified using PowerShell from macOS.

The installed PowerShell version was:

`PowerShell 7.6.3`

The Exchange Online Management module was installed using:

`Install-Module ExchangeOnlineManagement -Scope CurrentUser`

The module was imported using:

`Import-Module ExchangeOnlineManagement`

The Exchange Online connection command was verified using:

`Get-Command Connect-ExchangeOnline`

The installed ExchangeOnlineManagement module version was:

`3.10.1`

---

## Exchange Online Connection

An authenticated Exchange Online PowerShell session was established using:

`Connect-ExchangeOnline -ShowBanner:$false`

The connection was verified with:

`Get-ConnectionInformation | Format-Table UserPrincipalName,ConnectionId,State`

The connection returned:

`State: Connected`

This confirmed that the administrative PowerShell session was successfully connected to Exchange Online.

---

## Direct Mailbox Verification

Brian Carter's mailbox was queried directly using:

`Get-EXOMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

The command returned:

`DisplayName          : Brian Carter`

`PrimarySmtpAddress   : brian.carter@Stefon.onmicrosoft.com`

`RecipientTypeDetails : UserMailbox`

This confirmed that:

- The mailbox existed.
- The primary SMTP address was correctly configured.
- The recipient was recognized by Exchange Online as a `UserMailbox`.

---

## Mailbox Inventory Verification

A broader Exchange Online mailbox inventory was generated using:

`Get-EXOMailbox -ResultSize Unlimited | Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails | Sort-Object DisplayName | Format-Table -AutoSize`

Brian Carter appeared successfully within the Exchange Online mailbox inventory.

The resulting entry confirmed:

- Brian Carter
- brian.carter@Stefon.onmicrosoft.com
- UserMailbox

This provided additional verification that the mailbox had been successfully provisioned within the tenant.

---

# Resolution

The incident was resolved by assigning Microsoft 365 Business Basic to Brian Carter.

After the license assignment:

1. Exchange Online provisioned a user mailbox.
2. Brian Carter appeared in Exchange Admin Center.
3. The mailbox showed the recipient type `UserMailbox`.
4. Exchange Online PowerShell successfully returned the mailbox.
5. The mailbox appeared in the tenant-wide mailbox inventory.

No additional mailbox provisioning action was required.

---

# Troubleshooting Workflow

The troubleshooting process used during this scenario was:

1. Verify that the user exists in Microsoft 365.
2. Verify that the identity exists in Microsoft Entra ID.
3. Search Exchange Admin Center for the user's mailbox.
4. Confirm that no mailbox currently exists.
5. Review the user's Microsoft 365 licensing.
6. Identify that the account has no assigned license.
7. Confirm that an Exchange-capable license is available.
8. Assign Microsoft 365 Business Basic.
9. Save the licensing configuration.
10. Allow Exchange Online mailbox provisioning to complete.
11. Verify the mailbox in Exchange Admin Center.
12. Connect to Exchange Online PowerShell.
13. Query the mailbox using `Get-EXOMailbox`.
14. Verify the primary SMTP address.
15. Verify that `RecipientTypeDetails` equals `UserMailbox`.
16. Generate a mailbox inventory and confirm that the user appears.
17. Document the root cause, remediation, and final resolution.

---

# Key Troubleshooting Lesson

A Microsoft 365 user identity and an Exchange Online mailbox should be treated as separate components during troubleshooting.

A user may exist successfully in Microsoft Entra ID while still lacking an Exchange Online mailbox.

When investigating a missing mailbox, administrators should verify:

- User identity
- Account status
- Usage location
- License assignment
- Available tenant licenses
- Exchange Online service entitlement
- Mailbox provisioning status
- Recipient type
- Primary SMTP address

Licensing should be checked early because a missing Exchange Online entitlement can prevent mailbox provisioning entirely.

This prevents unnecessary troubleshooting of Outlook, browser access, authentication, or local workstation settings when the underlying problem is actually Microsoft 365 service provisioning.

---

# Administrative Tools Used

- Microsoft 365 Admin Center
- Microsoft Entra Admin Center
- Exchange Admin Center
- PowerShell 7
- ExchangeOnlineManagement PowerShell Module
- Exchange Online PowerShell

---

# Commands Used

## Check Exchange Online Module

`Get-Module ExchangeOnlineManagement -ListAvailable`

## Install Exchange Online Module

`Install-Module ExchangeOnlineManagement -Scope CurrentUser`

## Import Exchange Online Module

`Import-Module ExchangeOnlineManagement`

## Verify Connection Command

`Get-Command Connect-ExchangeOnline`

## Connect to Exchange Online

`Connect-ExchangeOnline -ShowBanner:$false`

## Verify Connection

`Get-ConnectionInformation | Format-Table UserPrincipalName,ConnectionId,State`

## Query Brian Carter's Mailbox

`Get-EXOMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

## Generate Mailbox Inventory

`Get-EXOMailbox -ResultSize Unlimited | Select-Object DisplayName,PrimarySmtpAddress,RecipientTypeDetails | Sort-Object DisplayName | Format-Table -AutoSize`

---

# Screenshot Evidence

Supporting evidence for this scenario is stored under:

`Screenshots/02-Mailboxes/`

The evidence set includes:

- `01-Brian-Carter-Mailbox-Missing.png`
- `02-Brian-Carter-Unlicensed.png`
- `03-Brian-Carter-License-Assigned.png`
- `04-Brian-Carter-Mailbox-Provisioned.png`
- `05-Brian-Carter-PowerShell-Verification.png`

These screenshots demonstrate the progression from the initial failure through successful remediation and verification.

---

# Skills Demonstrated

- Exchange Online Administration
- Microsoft 365 Administration
- Microsoft Entra ID
- Exchange Admin Center
- Microsoft 365 Licensing
- Exchange Online Licensing
- Mailbox Provisioning
- Recipient Management
- Exchange Online PowerShell
- PowerShell 7
- Microsoft 365 Troubleshooting
- Root Cause Analysis
- Service Restoration
- Resolution Verification
- Incident Documentation
- Technical Documentation

---

# Scenario Outcome

**Status:** Resolved

Brian Carter's missing Exchange Online mailbox was traced to a missing Microsoft 365 Business Basic license.

After the license was assigned, Exchange Online successfully provisioned the mailbox.

The final mailbox was independently verified through both the Exchange Admin Center and Exchange Online PowerShell and was confirmed to be a valid `UserMailbox`.
---

# Scenario 2 — Missing Exchange Online Service Entitlement

## Overview

This scenario demonstrates a Microsoft 365 licensing issue in which a user retains an assigned Microsoft 365 product license but loses access to Exchange Online because the Exchange-specific service plan inside that license is disabled.

Unlike Scenario 1, where the user had no Microsoft 365 license at all, this scenario demonstrates why administrators must inspect individual service entitlements when troubleshooting application access.

---

## Incident Summary

**User:** Emily Brown  
**Service:** Exchange Online  
**Issue:** Microsoft 365 Business Basic remained assigned, but Exchange Online (Plan 1) was disabled.

Emily Brown initially had a working Exchange Online mailbox.

After the Exchange Online service entitlement was disabled within Microsoft 365 Business Basic, the mailbox became unavailable through Exchange Admin Center and Exchange Online PowerShell.

The incident was investigated and resolved using:

- Microsoft 365 Admin Center
- Exchange Admin Center
- Exchange Online PowerShell

---

## Known-Good Baseline

Before reproducing the incident, Emily Brown had a functioning Exchange Online mailbox.

Exchange Admin Center displayed:

- Display Name: Emily Brown
- Primary SMTP Address: emily.brown@Stefon.onmicrosoft.com
- Recipient Type: UserMailbox

Microsoft 365 Admin Center showed:

- Microsoft 365 Business Basic: Assigned
- Exchange Online (Plan 1): Enabled

This established a known-good baseline.

---

## Fault Simulation

Microsoft 365 Business Basic remained assigned to Emily Brown.

Under the Apps section of the license, only the following service was disabled:

`Exchange Online (Plan 1)`

The overall Microsoft 365 Business Basic product license was not removed.

This created the following configuration:

- Microsoft 365 Business Basic: Assigned
- Exchange Online (Plan 1): Disabled

---

## Symptoms After Service Removal

After Exchange Online (Plan 1) was disabled:

- Emily Brown still appeared as licensed with Microsoft 365 Business Basic.
- Exchange Online was no longer enabled for the account.
- Emily Brown disappeared from Exchange Admin Center mailboxes.
- Searching Exchange Admin Center returned `No results found`.
- `Get-EXOMailbox` could not locate the mailbox.
- `Get-Recipient` could not locate the Exchange recipient.

This demonstrated that an assigned Microsoft 365 product license does not guarantee that all services within the product are enabled.

---

# Exchange Admin Center Investigation

Exchange Admin Center was opened and the following location was reviewed:

`Recipients > Mailboxes`

A search was performed for:

`Emily Brown`

The result returned:

`No results found`

This confirmed that Emily Brown was no longer available as an active Exchange Online user mailbox after the Exchange service entitlement was disabled.

---

# PowerShell Diagnosis

## Exchange Online Connection

An Exchange Online PowerShell session was established using:

`Connect-ExchangeOnline -ShowBanner:$false`

---

## Active Mailbox Lookup

The following command was executed:

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

The command returned an object-not-found error.

This demonstrated that Emily Brown could no longer be located as an active Exchange Online mailbox.

---

## Exchange Recipient Lookup

The following command was executed:

`Get-Recipient -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientType,RecipientTypeDetails`

The command also returned an object-not-found result.

This provided additional evidence that the user no longer had an active Exchange recipient available through the standard recipient lookup.

---

## Additional Mailbox State Investigation

An additional mailbox-state check was attempted during troubleshooting.

The investigation reinforced that mailbox and recipient visibility can change after Exchange Online licensing or service entitlement changes.

The primary diagnostic evidence used for this scenario was:

- Microsoft 365 service-plan configuration
- Exchange Admin Center mailbox visibility
- `Get-EXOMailbox`
- `Get-Recipient`

---

# Root Cause

The root cause was a disabled Exchange Online service entitlement.

Emily Brown still had:

`Microsoft 365 Business Basic`

assigned.

However, the individual service:

`Exchange Online (Plan 1)`

was disabled within the license.

As a result, Exchange Online access and the active mailbox became unavailable even though the user still appeared to have Microsoft 365 Business Basic assigned.

---

# Remediation

The Microsoft 365 Business Basic license remained assigned.

The following path was used:

`Microsoft 365 Admin Center > Users > Active users > Emily Brown > Licenses and apps`

Under the Microsoft 365 Business Basic Apps configuration, the following service was re-enabled:

`Exchange Online (Plan 1)`

The updated license configuration was saved.

---

# Exchange Admin Center Recovery Verification

After Exchange Online (Plan 1) was restored, Exchange Admin Center was refreshed.

Navigation:

`Recipients > Mailboxes`

Emily Brown returned successfully.

The mailbox displayed:

- Display Name: Emily Brown
- Email Address: emily.brown@Stefon.onmicrosoft.com
- Recipient Type: UserMailbox

This confirmed that Exchange Online service restoration had completed.

---

# PowerShell Recovery Verification

Emily Brown's mailbox was queried again using:

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

The command returned:

`DisplayName          : Emily Brown`

`PrimarySmtpAddress   : emily.brown@Stefon.onmicrosoft.com`

`RecipientTypeDetails : UserMailbox`

This independently confirmed successful recovery.

---

# Resolution

The incident was resolved by re-enabling:

`Exchange Online (Plan 1)`

within Emily Brown's existing Microsoft 365 Business Basic license.

No new Microsoft 365 product license was required.

After the service entitlement was restored:

1. Emily Brown returned to Exchange Admin Center.
2. The recipient type returned as `UserMailbox`.
3. Exchange Online PowerShell successfully located the mailbox.
4. The primary SMTP address was verified.
5. Normal Exchange Online mailbox availability was restored.

---

# Troubleshooting Workflow

The troubleshooting process for this incident was:

1. Establish a working Exchange Online mailbox baseline.
2. Confirm Microsoft 365 Business Basic was assigned.
3. Verify Exchange Online (Plan 1) was enabled.
4. Disable only Exchange Online (Plan 1).
5. Leave Microsoft 365 Business Basic assigned.
6. Verify the product license still appeared assigned.
7. Search for the mailbox in Exchange Admin Center.
8. Confirm that the mailbox was unavailable.
9. Connect to Exchange Online PowerShell.
10. Query the mailbox with `Get-EXOMailbox`.
11. Query the recipient with `Get-Recipient`.
12. Identify the missing Exchange service entitlement.
13. Re-enable Exchange Online (Plan 1).
14. Refresh Exchange Admin Center.
15. Verify the mailbox returned.
16. Query the mailbox again through PowerShell.
17. Confirm `RecipientTypeDetails` returned `UserMailbox`.
18. Document the incident and resolution.

---

# Comparison With Scenario 1

## Scenario 1

Brian Carter had:

- No Microsoft 365 Business Basic license
- No Exchange Online entitlement
- No Exchange Online mailbox

Resolution:

`Assign Microsoft 365 Business Basic`

## Scenario 2

Emily Brown had:

- Microsoft 365 Business Basic assigned
- Exchange Online (Plan 1) disabled
- Exchange mailbox unavailable

Resolution:

`Re-enable Exchange Online (Plan 1) within the existing license`

---

# Key Troubleshooting Lesson

Microsoft 365 licensing should be investigated at two levels:

## Product License

Example:

`Microsoft 365 Business Basic`

## Individual Service Plans

Example:

`Exchange Online (Plan 1)`

A user can appear licensed at the Microsoft 365 product level while still being unable to access a particular Microsoft 365 service.

For Exchange Online incidents, administrators should verify both:

- The appropriate Microsoft 365 product license is assigned.
- The Exchange Online service plan within that license is enabled.

This helps distinguish licensing and cloud-service issues from:

- Outlook client problems
- Password issues
- Authentication failures
- Local workstation problems
- Mail profile corruption

---

# Commands Used

## Connect to Exchange Online

`Connect-ExchangeOnline -ShowBanner:$false`

## Check Mailbox During Failure

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

## Check Recipient During Failure

`Get-Recipient -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientType,RecipientTypeDetails`

## Verify Mailbox After Recovery

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/02-Mailboxes/`

Scenario 2 evidence includes:

- `06-Emily-Brown-Mailbox-Baseline.png`
- `07-Emily-Brown-Exchange-Enabled.png`
- `08-Emily-Brown-Exchange-Disabled.png`
- `09-Emily-Brown-Mailbox-Missing-After-Exchange-Disabled.png`
- `10-Emily-Brown-PowerShell-Missing-Mailbox-Diagnosis.png`
- `11-Emily-Brown-Exchange-Reenabled.png`
- `12-Emily-Brown-Mailbox-Restored.png`
- `13-Emily-Brown-PowerShell-Recovery-Verification.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Microsoft 365 Administration
- Microsoft 365 Licensing
- Microsoft 365 Service Plans
- Exchange Online Service Entitlements
- Exchange Admin Center
- Exchange Online PowerShell
- Recipient Troubleshooting
- Mailbox Recovery
- Root Cause Analysis
- Service Restoration
- Resolution Verification
- Incident Management
- Technical Documentation

---

# Scenario Outcome

**Status: Resolved**

Emily Brown retained Microsoft 365 Business Basic but lost Exchange Online functionality because Exchange Online (Plan 1) was disabled within the assigned license.

The Exchange Online service entitlement was re-enabled.

Exchange Admin Center and Exchange Online PowerShell independently verified that Emily Brown's mailbox returned successfully as a valid `UserMailbox`.
