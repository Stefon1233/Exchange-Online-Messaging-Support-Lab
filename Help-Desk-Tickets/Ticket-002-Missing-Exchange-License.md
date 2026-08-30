# INC-002 — Missing Exchange Online Service Entitlement

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-002 |
| Requester | Emily Brown |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Email / Mailbox |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User has Microsoft 365 Business Basic assigned but cannot access Exchange Online because the Exchange Online service entitlement is disabled.

---

# User-Reported Issue

Emily Brown reports that her organizational mailbox is unavailable.

The Microsoft 365 Admin Center shows that Microsoft 365 Business Basic is still assigned to the account, but the Exchange Online mailbox is no longer available in Exchange Admin Center.

---

# Baseline Verification

Before reproducing the incident, Emily Brown had a functioning Exchange Online mailbox.

Exchange Admin Center showed:

- Display Name: Emily Brown
- Email Address: emily.brown@Stefon.onmicrosoft.com
- Recipient Type: UserMailbox

Microsoft 365 Admin Center showed:

- Microsoft 365 Business Basic: Assigned
- Exchange Online (Plan 1): Enabled

This established a known-good baseline before the service configuration was changed.

---

# Symptoms

After Exchange Online (Plan 1) was disabled:

- Microsoft 365 Business Basic remained assigned.
- Exchange Online (Plan 1) was not enabled.
- Emily Brown no longer appeared under Exchange Admin Center mailboxes.
- Searching Exchange Admin Center for Emily Brown returned `No results found`.
- Exchange Online PowerShell could not locate the mailbox.
- Exchange recipient lookup could not locate the recipient.

---

# Troubleshooting Performed

## Step 1 — Verify Microsoft 365 License

Emily Brown was reviewed through:

`Microsoft 365 Admin Center > Users > Active users > Emily Brown > Licenses and apps`

Microsoft 365 Business Basic remained assigned.

This demonstrated that the user was not completely unlicensed.

---

## Step 2 — Review Individual Service Entitlements

The Apps section of the Microsoft 365 Business Basic license was reviewed.

The Exchange-specific service was identified as:

`Exchange Online (Plan 1)`

Exchange Online (Plan 1) was disabled while the parent Microsoft 365 Business Basic license remained assigned.

This created a condition where the user appeared licensed at the product level but did not have an active Exchange Online service entitlement.

---

## Step 3 — Check Exchange Admin Center

Exchange Admin Center was opened.

Navigation:

`Recipients > Mailboxes`

A search was performed for:

`Emily Brown`

The result returned:

`No results found`

This confirmed that Emily was no longer available as an active Exchange Online user mailbox.

---

## Step 4 — Connect to Exchange Online PowerShell

PowerShell 7 and the ExchangeOnlineManagement module were used.

The session was established with:

`Connect-ExchangeOnline -ShowBanner:$false`

---

## Step 5 — Query the Mailbox

The following command was executed:

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

The command returned an object-not-found error.

This confirmed that the active Exchange Online mailbox could not be located.

---

## Step 6 — Query the Exchange Recipient

The following command was executed:

`Get-Recipient -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientType,RecipientTypeDetails`

The recipient lookup also returned an object-not-found error.

This provided additional evidence that the account no longer had an active Exchange Online recipient object available through the normal recipient lookup.

---

# Root Cause

The root cause was not removal of the complete Microsoft 365 license.

Microsoft 365 Business Basic remained assigned to Emily Brown.

However, the individual:

`Exchange Online (Plan 1)`

service entitlement had been disabled.

Because the Exchange Online component of the license was unavailable, the user's active Exchange mailbox became unavailable.

---

# Resolution

The Microsoft 365 Business Basic license was retained.

Under the license's Apps configuration, the following service was re-enabled:

`Exchange Online (Plan 1)`

The changes were saved successfully.

No additional Microsoft 365 product license needed to be assigned because Business Basic was already present.

---

# Exchange Admin Center Recovery Verification

After Exchange Online (Plan 1) was restored, Exchange Admin Center was refreshed.

Emily Brown again appeared under:

`Recipients > Mailboxes`

The restored mailbox displayed:

- Display Name: Emily Brown
- Email Address: emily.brown@Stefon.onmicrosoft.com
- Recipient Type: UserMailbox

This confirmed successful Exchange Online service restoration.

---

# PowerShell Recovery Verification

The mailbox was queried again using:

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

The result returned:

`DisplayName          : Emily Brown`

`PrimarySmtpAddress   : emily.brown@Stefon.onmicrosoft.com`

`RecipientTypeDetails : UserMailbox`

This independently verified that Emily Brown's Exchange Online mailbox had been restored.

---

# Resolution Verification

The incident was considered resolved after confirming:

- Microsoft 365 Business Basic remained assigned.
- Exchange Online (Plan 1) was re-enabled.
- Emily Brown returned to Exchange Admin Center.
- Recipient type was `UserMailbox`.
- `Get-EXOMailbox` successfully located the mailbox.
- The correct primary SMTP address was returned.

---

# Key Troubleshooting Lesson

A user showing a Microsoft 365 license does not necessarily mean every service inside that license is enabled.

When troubleshooting Microsoft 365 application access, administrators should review both:

1. The assigned Microsoft 365 product license.
2. The individual service plans contained within the license.

For Exchange incidents, verify specifically that:

`Exchange Online (Plan 1)`

or the applicable Exchange Online service plan is enabled.

This can prevent misdiagnosing the issue as an Outlook, authentication, or mailbox corruption problem.

---

# Closure Notes

User's Exchange Online mailbox was unavailable even though Microsoft 365 Business Basic remained assigned.

Investigation identified that Exchange Online (Plan 1) had been disabled within the user's Microsoft 365 license.

Exchange Online (Plan 1) was restored.

Exchange Admin Center and Exchange Online PowerShell both confirmed that the mailbox returned successfully as a valid `UserMailbox`.

No further action required.

---

# Administrative Tools Used

- Microsoft 365 Admin Center
- Exchange Admin Center
- Microsoft Entra ID
- PowerShell 7
- ExchangeOnlineManagement Module
- Exchange Online PowerShell

---

# Commands Used

## Connect to Exchange Online

`Connect-ExchangeOnline -ShowBanner:$false`

## Query Mailbox During Failure

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

## Query Exchange Recipient

`Get-Recipient -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientType,RecipientTypeDetails`

## Verify Mailbox After Recovery

`Get-EXOMailbox -Identity "emily.brown@Stefon.onmicrosoft.com" | Format-List DisplayName,PrimarySmtpAddress,RecipientTypeDetails`

---

# Screenshot Evidence

Supporting screenshots are stored under:

`Screenshots/02-Mailboxes/`

Evidence includes:

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

- Exchange Online Troubleshooting
- Microsoft 365 Licensing
- Microsoft 365 Service Plans
- Exchange Online Service Entitlements
- Exchange Admin Center
- Exchange Online PowerShell
- Mailbox Recovery
- Recipient Troubleshooting
- Root Cause Analysis
- Service Restoration
- Incident Management
- ServiceNow-Style Documentation
- Resolution Verification

---

# Final Status

**Resolved**

Emily Brown retained Microsoft 365 Business Basic but lost Exchange Online access because Exchange Online (Plan 1) was disabled within the license.

Re-enabling Exchange Online (Plan 1) restored the mailbox.

Exchange Admin Center and Exchange Online PowerShell confirmed that the mailbox returned successfully as a valid `UserMailbox`.
