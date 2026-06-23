# Windows-Sysadmin-Helpdesk-Lab
Hands-on Windows Sysadmin and Helpdesk lab covering new employee onboarding, six helpdesk ticket simulations, PowerShell automation scripting, scheduled tasks, and security log investigation in an Active Directory environment.

## Windows Sysadmin & Helpdesk Lab

## Overview
This lab documents hands-on Windows administration and helpdesk tasks performed in a VMware home lab environment. All tasks were completed using PowerShell on Windows Server 2022 (DC) and Windows 10 (Client01) joined to the mydomain.com Active Directory domain. Topics covered include new employee onboarding, helpdesk ticket simulations, PowerShell automation scripting, scheduled task management, and security log analysis.

---

## Technologies Used
- Windows Server 2022 (DC)
- Windows 10 Enterprise (Client01)
- Active Directory Domain Services
- Group Policy Management Console (GPMC)
- PowerShell
- Windows Task Scheduler
- Windows Event Viewer
- Print and Document Services
- VMware Workstation

---

## Lab Environment

| Machine | Role | Domain |
|---|---|---|
| DC | Domain Controller / Print Server | mydomain.com |
| Client01 | Windows 10 Workstation | mydomain.com |

---

## What I Built

**New Employee Onboarding** — Provisioned a complete domain account for a new hire entirely in PowerShell including AD user creation with correct OU placement, security group creation and membership, SMB share creation with permissions, GPO drive mapping, remote GPO push, and roaming profile configuration.

**Ticket #001 — Account Lockout** — Investigated a domain account lockout, confirmed the lockout policy, identified the locked account using Search-ADAccount, unlocked it with Unlock-ADAccount, and reset the password with a forced change at next login.

**Ticket #002 — Software Deployment via GPO** — Created a software distribution share on DC, downloaded the 7-Zip MSI using Invoke-WebRequest, created and linked a GPO to the IT OU, pushed the policy remotely with Invoke-GPUpdate, and verified the installation with Get-Package.

**Ticket #003 — Printer Setup & Deployment** — Installed the Print and Document Services role, created a printer port and printer object on DC, shared the printer via Print Management, and deployed it automatically to Marketing users via a GPO preference.

**Ticket #004 — User Profile Issues** — Diagnosed a profile loss caused by a local profile being wiped during a reimage. Created a Profiles share on DC and configured a roaming profile path on the user's AD account so settings persist across machines and survive reimaging.

**Ticket #005 — Machine Won't Boot** — Covered the full boot failure troubleshooting workflow including BIOS boot order verification, Windows Recovery Environment access, boot file repair using bootrec, system file repair using sfc and DISM, BSOD investigation using Event Viewer and minidump files, and deleted file recovery using Shadow Copies.

**Ticket #006 — Permission Denied on Shared Folder** — Investigated a shared folder access denial by verifying group membership, NTFS permissions, share permissions, and network connectivity. Identified the root cause as a stale security token and resolved it by having the user log off and back on.

**PowerShell Onboarding Script** — Built a reusable CSV-driven onboarding script that automates the full new hire provisioning workflow — AD account creation, group membership, roaming profile path, and forced password change — with Try/Catch error handling to gracefully skip duplicate accounts.

**Scheduled Tasks** — Automated the onboarding script using Windows Task Scheduler to run every Monday at 8AM with elevated privileges so new hire provisioning runs automatically without IT intervention.

**Event Viewer Security Investigation** — Queried Windows Security logs using Get-WinEvent to investigate failed login attempts (Event ID 4625), account lockouts (Event ID 4740), and account creation events (Event ID 4720). Exported findings to CSV for audit documentation.

---

## Table of Contents

| # | Section | Description |
|---|---|---|
| 01 | [New Employee Onboarding](01-New-Employee-Onboarding/README.md) | Full PowerShell onboarding workflow for a new hire in the Marketing department |
| 02 | [Ticket #001 — Account Lockout](02-Ticket-001-Account-Lockout/README.md) | Investigated and resolved a domain account lockout using PowerShell |
| 03 | [Ticket #002 — Software Deployment via GPO](03-Ticket-002-Software-Deployment-Gpo/README.md) | Deployed 7-Zip to all IT OU machines silently via Group Policy |
| 04 | [Ticket #003 — Printer Setup & Deployment](04-Ticket-003-Printer-Setup-Deployment/README.md) | Configured and deployed a network printer to Marketing users via GPO |
| 05 | [Ticket #004 — User Profile Issues](05-Ticket-004-User-Profile-Issues/README.md) | Configured roaming profiles to prevent profile loss after machine reimage |
| 06 | [Ticket #005 — Machine Won't Boot](06-Ticket-005-Machine-Wont-Boot/README.md) | Full boot failure troubleshooting including bootrec, sfc, DISM, and Shadow Copies |
| 07 | [Ticket #006 — Permission Denied on Shared Folder](07-Ticket-006-Permission-Denied/README.md) | Diagnosed shared folder access denial and identified stale security token as root cause |
| 08 | [PowerShell Onboarding Script](08-Powershell-Onboarding-Script/README.md) | Built a CSV-driven automation script for bulk new hire provisioning |
| 09 | [Scheduled Tasks](09-Scheduled-Tasks/README.md) | Automated the onboarding script with Task Scheduler to run every Monday at 8AM |
| 10 | [Event Viewer Security Investigation](10-Event-Viewer-Security-Investigation/README.md) | Queried Security logs with Get-WinEvent to investigate failed logins, lockouts, and account creation |

---

## Key Takeaways
- Completed a full end-to-end new hire onboarding workflow entirely in PowerShell without touching the GUI
- Simulated and resolved six real-world helpdesk tickets covering the most common Windows support scenarios
- Built a reusable CSV-driven automation script with error handling for bulk user provisioning
- Automated the onboarding script with Task Scheduler to eliminate manual intervention
- Queried and analyzed Windows Security logs programmatically for IAM auditing and security investigations
- Demonstrated the ability to troubleshoot across multiple layers — AD, permissions, profiles, boot, and security logs
