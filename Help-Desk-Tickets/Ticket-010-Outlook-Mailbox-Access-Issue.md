# Ticket 010 — Outlook on the Web Access Issue

## Ticket Information

**Ticket ID:** INC-010  
**Category:** Exchange Online / Outlook  
**Subcategory:** Mailbox Access  
**Priority:** Medium  
**Status:** Resolved  
**Affected User:** Brian Carter  
**Affected Service:** Outlook on the web  
**Environment:** Microsoft 365 / Exchange Online

---

## User Report

Brian Carter reported that he was unable to access his Exchange Online mailbox through Outlook on the web.

When attempting to open Outlook in a new browser session, the application returned an error instead of loading the mailbox.

---

## Initial Symptoms

The user encountered an Outlook on the web error page displaying:

`Something went wrong.`

Additional diagnostic information showed:

`Error: 440`

The failure occurred when Outlook attempted to initialize the mailbox session.

---

## Initial Investigation

The Exchange Online mailbox was verified using PowerShell.

Command:

`Get-EXOMailbox -Identity "brian.carter@Stefon.onmicrosoft.com"`

The mailbox returned successfully as:

- Display Name: Brian Carter
- Recipient Type: UserMailbox
- Primary SMTP Address: brian.carter@Stefon.onmicrosoft.com

This confirmed that the mailbox existed and was provisioned correctly.

---

## Client Access Investigation

Client Access settings were reviewed using:

`Get-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com"`

The investigation showed:

- OWAEnabled: False
- MAPIEnabled: True
- PopEnabled: True
- ImapEnabled: True
- ActiveSyncEnabled: True

The key finding was:

`OWAEnabled : False`

This indicated that Outlook on the web had been specifically disabled for the user while the mailbox itself remained active.

---

## Mailbox Health Validation

To determine whether the issue affected mail delivery or only client access, a test message was sent from Emily Brown to Brian Carter.

Subject:

`LAB OWA ACCESS TEST - Mailbox Healthy`

The Exchange Admin Center Message Trace showed:

`Status: Delivered`

Detailed message events included:

- Receive
- Submit
- Deliver

This confirmed that Exchange Online continued to process and deliver mail to Brian's mailbox even while Outlook on the web access was disabled.

---

## PowerShell Message Trace Validation

Exchange Online PowerShell was used to independently verify delivery.

The trace returned:

- Sender: emily.brown@Stefon.onmicrosoft.com
- Recipient: brian.carter@Stefon.onmicrosoft.com
- Subject: LAB OWA ACCESS TEST - Mailbox Healthy
- Status: Delivered

Detailed events confirmed:

- Receive
- Submit
- Deliver

This demonstrated that the incident was not a mailbox provisioning or mail-flow failure.

---

## Root Cause

The root cause was an Exchange Online Client Access configuration setting.

Brian Carter's mailbox had:

`OWAEnabled : False`

This prevented Outlook on the web from opening the mailbox even though:

- The mailbox existed
- The Exchange Online license remained valid
- Mail flow continued normally
- Messages continued to be delivered
- Other client access protocols remained enabled

The issue was therefore isolated specifically to Outlook on the web access.

---

## Resolution

Outlook on the web access was restored using Exchange Online PowerShell.

Command:

`Set-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" -OWAEnabled $true`

The setting was then verified using:

`Get-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com"`

The result showed:

`OWAEnabled : True`

Other client access settings remained enabled.

---

## Validation

After remediation:

1. A new Outlook browser session was opened.
2. Brian Carter successfully accessed Outlook on the web.
3. Brian's Inbox loaded normally.
4. The previously delivered test message was visible.
5. The message body confirmed that Exchange Online had continued receiving mail while OWA access was disabled.

The test message stated:

`This message validates that Brian Carter's Exchange Online mailbox continues receiving mail while Outlook on the web access is disabled.`

---

## Resolution Summary

**Problem:** User unable to access Outlook on the web.

**Finding:** Exchange mailbox was healthy, but OWA was disabled.

**Root Cause:** `OWAEnabled` was set to `False`.

**Resolution:** Enabled Outlook on the web using `Set-CASMailbox`.

**Verification:** User successfully accessed Outlook and confirmed the previously delivered test message was present.

**Final Status:** Resolved

---

## Commands Used

### Verify Mailbox

`Get-EXOMailbox -Identity "brian.carter@Stefon.onmicrosoft.com"`

### Review Client Access Settings

`Get-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" | Format-List OWAEnabled,MAPIEnabled,PopEnabled,ImapEnabled,ActiveSyncEnabled`

### Restore Outlook on the Web

`Set-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" -OWAEnabled $true`

### Verify Remediation

`Get-CASMailbox -Identity "brian.carter@Stefon.onmicrosoft.com" | Format-List Identity,OWAEnabled,MAPIEnabled,PopEnabled,ImapEnabled,ActiveSyncEnabled`

### Verify Mail Delivery

`Get-MessageTraceV2`

### Review Message Events

`Get-MessageTraceDetailV2`

---

## Skills Demonstrated

- Exchange Online Administration
- Outlook on the Web Troubleshooting
- Exchange Online PowerShell
- Get-CASMailbox
- Set-CASMailbox
- Get-EXOMailbox
- Client Access Configuration
- Message Trace
- Get-MessageTraceV2
- Get-MessageTraceDetailV2
- Mailbox Health Validation
- Root Cause Analysis
- Service Restoration
- End-to-End Verification
- Help Desk Incident Documentation
