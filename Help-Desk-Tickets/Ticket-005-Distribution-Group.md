# INC-005 — Distribution List Membership and Mail Delivery

## Incident Information

| Field | Details |
|---|---|
| Incident | INC-005 |
| Requester | Brian Carter |
| Category | Microsoft 365 |
| Subcategory | Exchange Online |
| Service | Distribution List |
| Impact | Individual User |
| Urgency | Medium |
| Priority | P3 |
| Status | Resolved |

---

# Short Description

User is not receiving messages sent to the Finance Department distribution list because the account is not a member of the group.

---

# User-Reported Issue

Brian Carter reports that he is not receiving email messages sent to the Finance Department distribution list.

Other Finance Department members are receiving distribution list messages successfully.

---

# Environment

**Distribution List:** Finance Department  
**Primary SMTP Address:** Financesdl@Stefon.onmicrosoft.com  
**Affected User:** Brian Carter  
**User Email:** brian.carter@Stefon.onmicrosoft.com

Administrative tools used:

- Exchange Admin Center
- Exchange Online PowerShell
- Outlook on the web

---

# Initial Investigation

Exchange Admin Center was opened and the following location was reviewed:

`Recipients > Groups > Distribution list`

The Finance Department distribution list was located successfully.

The group information confirmed:

- Name: Finance Department
- Group Type: Distribution list
- Primary Email Address: Financesdl@Stefon.onmicrosoft.com
- Owner: Configured
- Existing Members: 2

This confirmed that the distribution list itself existed and was available.

---

# Membership Investigation

The Members section of the Finance Department distribution list was reviewed.

The existing members were:

- Emily Brown
- Robert Garcia

Brian Carter was not present.

A search for:

`Brian Carter`

returned no member result.

This established the initial failure state.

---

# PowerShell Group Verification

Exchange Online PowerShell was used to identify the distribution group.

The following command was executed:

`Get-DistributionGroup | Select-Object DisplayName,PrimarySmtpAddress | Format-Table -AutoSize`

The result confirmed:

`Finance Department`

with the primary SMTP address:

`Financesdl@Stefon.onmicrosoft.com`

---

# PowerShell Membership Verification

The group membership was reviewed using:

`Get-DistributionGroupMember -Identity "Financesdl@Stefon.onmicrosoft.com" | Select-Object DisplayName,PrimarySmtpAddress,RecipientType | Format-Table -AutoSize`

The existing members returned:

- Emily Brown
- Robert Garcia

A specific query was then used to search for Brian Carter:

`Get-DistributionGroupMember -Identity "Financesdl@Stefon.onmicrosoft.com" | Where-Object {$_.PrimarySmtpAddress -eq "brian.carter@Stefon.onmicrosoft.com"} | Format-Table DisplayName,PrimarySmtpAddress,RecipientType -AutoSize`

No result was returned.

This independently confirmed that Brian Carter was not a member of the Finance Department distribution list.

---

# Root Cause

Brian Carter was not included in the Finance Department distribution list membership.

Because distribution list mail is expanded only to configured recipients, Brian did not receive messages sent to the Finance Department address.

The issue was caused by group membership rather than:

- Mailbox provisioning
- Exchange Online licensing
- Outlook configuration
- Mailbox permissions
- Authentication
- Mailbox forwarding

---

# Resolution

Exchange Admin Center was used to update the Finance Department distribution list membership.

Navigation:

`Exchange Admin Center > Recipients > Groups > Distribution list > Finance Department > Members`

Brian Carter was added as a member.

The configuration was saved successfully.

The updated membership list contained:

- Emily Brown
- Robert Garcia
- Brian Carter

---

# PowerShell Resolution Verification

Exchange Online PowerShell was used to verify Brian Carter's new membership.

The following command was executed:

`Get-DistributionGroupMember -Identity "Financesdl@Stefon.onmicrosoft.com" | Where-Object {$_.PrimarySmtpAddress -eq "brian.carter@Stefon.onmicrosoft.com"} | Format-Table DisplayName,PrimarySmtpAddress,RecipientType -AutoSize`

The result returned:

`Brian Carter`

`brian.carter@Stefon.onmicrosoft.com`

`UserMailbox`

This confirmed that Brian Carter was now an Exchange recipient member of the Finance Department distribution list.

---

# End-to-End Mail Delivery Test

A test message was sent to:

`Finance Department`

The message used the subject:

`LAB TEST - Distribution List Delivery`

The body stated:

`This message validates distribution list membership and mail delivery for Brian Carter.`

Brian Carter then signed into Outlook on the web.

The message successfully appeared in Brian's Inbox.

The received message showed:

- Sender: Emily Brown
- Recipient: Finance Department
- Subject: LAB TEST - Distribution List Delivery

This confirmed that Exchange Online successfully expanded the distribution list and delivered the message to Brian after the membership correction.

---

# Resolution Verification

The incident was considered resolved after confirming:

- Finance Department existed as a Distribution list.
- The distribution list had a valid SMTP address.
- Brian was absent from the initial member list.
- PowerShell confirmed Brian was not initially a member.
- Brian Carter was added through Exchange Admin Center.
- PowerShell confirmed Brian's new membership.
- A live email sent to the distribution list arrived in Brian's mailbox.

---

# Troubleshooting Workflow

1. Identify the affected distribution list.
2. Verify the group exists.
3. Confirm the group type is Distribution list.
4. Verify its SMTP address.
5. Review existing membership.
6. Search for the affected user.
7. Verify membership using Exchange Online PowerShell.
8. Identify missing membership as the root cause.
9. Add the user to the distribution list.
10. Save the configuration.
11. Verify the new membership through PowerShell.
12. Send a test message to the distribution list.
13. Confirm delivery in the affected user's mailbox.
14. Document the incident and resolution.

---

# Key Troubleshooting Lesson

When one user reports that they do not receive messages from a distribution list while other recipients do, group membership should be checked early.

Administrators should verify:

- Correct distribution group
- Primary SMTP address
- Group membership
- Recipient object
- Delivery restrictions
- Sender restrictions
- Message delivery

Membership can be independently confirmed using:

`Get-DistributionGroupMember`

A successful administrative change should also be validated with an actual message delivery test whenever possible.

---

# Commands Used

## List Distribution Groups

`Get-DistributionGroup | Select-Object DisplayName,PrimarySmtpAddress | Format-Table -AutoSize`

## List Finance Department Members

`Get-DistributionGroupMember -Identity "Financesdl@Stefon.onmicrosoft.com" | Select-Object DisplayName,PrimarySmtpAddress,RecipientType | Format-Table -AutoSize`

## Search Specifically for Brian Carter

`Get-DistributionGroupMember -Identity "Financesdl@Stefon.onmicrosoft.com" | Where-Object {$_.PrimarySmtpAddress -eq "brian.carter@Stefon.onmicrosoft.com"} | Format-Table DisplayName,PrimarySmtpAddress,RecipientType -AutoSize`

---

# Screenshot Evidence

Supporting evidence is stored under:

`Screenshots/04-Groups/`

Evidence includes:

- `01-Finance-Distribution-List-Baseline.png`
- `02-Finance-Distribution-List-Original-Members.png`
- `03-Brian-Not-Distribution-List-Member-EAC.png`
- `04-Distribution-Group-PowerShell-Inventory.png`
- `05-Brian-Not-Member-PowerShell.png`
- `06-Brian-Added-To-Distribution-List.png`
- `07-Brian-Distribution-List-PowerShell-Verification.png`
- `08-Brian-Distribution-List-Delivery-Verified.png`

---

# Skills Demonstrated

- Exchange Online Administration
- Distribution List Administration
- Recipient Management
- Group Membership Troubleshooting
- Exchange Admin Center
- Exchange Online PowerShell
- Get-DistributionGroup
- Get-DistributionGroupMember
- Outlook on the web
- Mail Delivery Testing
- Root Cause Analysis
- Incident Management
- ServiceNow-Style Documentation
- End-to-End Resolution Verification

---

# Final Status

**Resolved**

Brian Carter was not receiving Finance Department distribution list messages because he was not included in the group membership.

Brian was added to the Finance Department distribution list.

Exchange Online PowerShell confirmed the membership change, and an end-to-end Outlook test confirmed successful message delivery.
