# Hybrid Identity Troubleshooting & Remediation Runbook

## Overview

This repository documents the troubleshooting, investigation, remediation, and validation activities performed during a Microsoft Entra Connect Hybrid Identity deployment.

Unlike a standard implementation guide, this project focuses on real-world synchronization issues encountered while integrating an on-premises Active Directory environment with Microsoft Entra ID and Microsoft 365.

The objective was to identify and resolve issues preventing successful soft matching of existing Microsoft 365 cloud identities with synchronized Active Directory identities.

---

## Environment

### On-Premises

- Windows Server 2022
- Active Directory Domain Services (AD DS)
- DNS
- Organizational Units (OUs)
- Microsoft Entra Connect

### Cloud

- Microsoft Entra ID
- Microsoft 365
- Exchange Online
- Microsoft Teams
- SharePoint Online
- OneDrive for Business

---

## Technologies Used

- Microsoft Entra Connect
- Microsoft Entra ID
- Active Directory
- Microsoft 365
- PowerShell
- Microsoft Graph Administration
- Microsoft Entra Administrative Roles

---

## Business Scenario

The Microsoft 365 tenant was initially built using cloud-only identities.

Later, an on-premises Active Directory infrastructure was deployed and connected using Microsoft Entra Connect.

The requirement was to:

- Synchronize existing Active Directory users
- Convert cloud-only identities into hybrid identities
- Preserve existing Microsoft 365 data
- Avoid duplicate user creation
- Establish Active Directory as the Source of Authority

---

# Initial Architecture

```mermaid
flowchart LR

    AD["Active Directory"]

    Connect["Microsoft Entra Connect"]

    Entra["Microsoft Entra ID"]

    M365["Microsoft 365"]

    AD --> Connect

    Connect --> Entra

    Entra --> M365
```

---

# Issues Encountered

## Duplicate User Creation

Following initial synchronization several duplicate accounts were created.

Examples:

```text
david.carter5076@
miracle.botchway0932@
herty.yankson0604@
```

Symptoms:

- Existing cloud users remained.
- Additional synchronized users were created.
- Soft matching failed.
- On-premises synchronization status remained No for original accounts.

---

## Soft Match Failure

Some users synchronized successfully while others generated duplicate objects despite appearing identical.

### Successful Departments

✅ Finance

✅ Sales

✅ Customer Service

### Failed Departments

⚠ IT

⚠ HR (Alpha Botchway)

---

# Investigation Timeline

## Phase 1 - UPN Validation

Discovered that Active Directory users still contained:

```text
user@solotech.local
```

while Microsoft 365 users contained:

```text
user@solotechinnovations.onmicrosoft.com
```

### Remediation

Added a Microsoft 365 UPN suffix and updated users to:

```text
user@solotechinnovations.onmicrosoft.com
```

---

## Phase 2 - Bulk UPN Remediation

Performed mass updates across:

- Finance
- HR
- IT
- Sales
- Customer Service

Result:

✅ Newly synchronized users began matching correctly.

---

## Phase 3 - Pilot Sync Testing

Created controlled synchronization pilots.

### Sales Pilot

Test User:

```text
Abraham Botchway
```

Result:

```text
On-premises sync enabled = Yes
```

No duplicate account created.

This confirmed Microsoft Entra Connect was functioning correctly.

---

# Root Cause Investigation

Despite UPN remediation:

- Finance matched
- Sales matched
- Customer Service matched

However:

- IT users duplicated
- Alpha Botchway duplicated

Further investigation reviewed:

- Display names
- Email addresses
- Mail nicknames
- Proxy addresses
- Department
- Office location
- Job title
- Employee ID

No significant differences were identified.

---

# Critical Discovery

Investigation revealed affected users held Microsoft Entra administrative privileges.

Examples included:

### Miracle Botchway

- Helpdesk Administrator
- Domain Name Administrator
- Agent ID Administrator
- Entra Backup Administrator

### Alpha Botchway

- Organizational Message Writer

Additionally, some affected users acted as:

- Microsoft 365 Group Owners
- Administrative Group Owners

---

# Resolution

Removed:

- Direct Entra role assignments
- Group ownership assignments
- Administrative ownership assignments

Ran:

```powershell
Start-ADSyncSyncCycle -PolicyType Initial
```

---

# Result

Previously duplicated accounts successfully soft matched.

Examples:

### Before

```text
On-premises sync enabled = No
```

### After

```text
On-premises sync enabled = Yes
```

Successfully validated for:

- Alpha Botchway
- Miracle Botchway
- Abraham Botchway

---

# Troubleshooting Workflow

```mermaid
flowchart TB

    Sync["Initial Sync"]

    Duplicate["Duplicate User Created"]

    UPN["Validate UPN"]

    Mail
