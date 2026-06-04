[![Docs](https://img.shields.io/badge/docs-latest-brightgreen.svg)](https://attuneops.io/docs/)
[![Discord](https://img.shields.io/discord/844971127703994369)](https://discord.gg/PUsEXApb)
[![Docs](https://img.shields.io/badge/videos-watch-brightgreen.svg)](https://www.youtube.com/@attuneops)
[![Generic badge](https://img.shields.io/badge/download-latest-brightgreen.svg)](https://attuneops.io)

# Essential Eight - Windows

An AttuneOps project that automates the configuration and verification of the Australian Signals Directorate (ASD) **Essential Eight** mitigation strategies, at **Maturity Level 1 (ML1)**, on Windows targets.

Reference: [ASD Essential Eight Explained](https://www.cyber.gov.au/business-government/asds-cyber-security-frameworks/essential-eight/essential-eight-explained)

## Disclaimer

Cybersecurity is highly context-specific. What is appropriate for one organisation's size, industry, data sensitivity, regulatory environment, threat profile, and existing controls can be irrelevant — or even harmful — in another. Frameworks like the Essential Eight (ASD guidance) are explicitly a **baseline recommendation, not a guarantee of security or compliance**, and must be tailored.

This project automates a defensible interpretation of ML1 controls for Windows targets. It is not a substitute for:

- A risk assessment of your environment.
- Architectural review by a qualified security practitioner.
- Change management, testing, and rollback planning before applying configuration to production.
- Ongoing monitoring, vulnerability management, and incident response capability.

Some steps make changes that can disrupt access (account policy changes, RDP/WinRM hardening, application control enforcement, browser policy lockdowns). **Always run in a representative test environment first, run application control in `Audit` mode and review event logs before switching to `Enforce`, and confirm you can recover the system before applying broadly.**

## Project Scope

The project covers all eight ASD Essential Eight mitigation strategies. Each strategy is implemented as a pair of blueprint step groups:

- A `Configure ... - ML1` group that applies the control.
- A `Verify ... - ML1` group that audits the control and emits a compliance report.

| # | Essential Eight Strategy | Configure Blueprint | Verify Blueprint |
|---|---|---|---|
| 1 | Application Control (WDAC) | `Configure WDAC Application Control - ML1` | `Verify WDAC Application Control - ML1` |
| 2 | Patch Applications | `Configure Patch Applications - ML1` | `Verify Patch Applications - ML1` |
| 3 | Configure Microsoft Office Macro Settings | `Configure Office Macro Settings - ML1` | `Verify Office Macro Settings - ML1` |
| 4 | User Application Hardening | `Configure User Application Hardening - ML1` | `Verify User Application Hardening - ML1` |
| 5 | Restrict Administrative Privileges | `Configure Restrict Administrative Privileges - ML1` | `Verify Restrict Administrative Privileges - ML1` |
| 6 | Patch Operating Systems | `Configure Patch Operating Systems - ML1` | `Verify Patch Operating Systems - ML1` |
| 7 | Multi-Factor Authentication (Prerequisites) | `Configure MFA Prerequisites - ML1` | `Verify MFA Prerequisites - ML1` |
| 8 | Regular Backups | `Configure Regular Backups Windows Server - ML1` / `Configure Regular Backups Windows Desktop - ML1` | `Verify Regular Backups Windows Server - ML1` / `Verify Regular Backups Windows Desktop - ML1` |

> **MFA note:** Native Windows cannot enforce true multi-factor authentication on a domain or local account without an IdP or third-party MFA provider. This project configures the ML1 **prerequisites** on the target — NLA for RDP, hardened WinRM authentication, account lockout policy, authentication audit logging — and detects whether a third-party MFA provider is installed. Issuing MFA credentials and binding them to accounts is out of scope for the target server itself.

## What Each Strategy Does

### Application Control (WDAC)
Deploys a Windows Defender Application Control policy implementing the ASD application control requirements. Uses the AllowAll base template with user-writable path deny rules. Supports `Audit` and `Enforce` modes via the `WDAC Enforcement Mode` parameter. Archives policy XML for change management and reads the WDAC event log to confirm activity.

### Patch Applications
Ensures the `PSWindowsUpdate` PowerShell module and `winget` are installed, installs application updates via Windows Update, upgrades third-party applications via winget, detects and removes end-of-life applications, then verifies outstanding security updates and update history against the ML1 timing threshold.

### Configure Microsoft Office Macro Settings
Detects installed Office versions and applications, disables VBA macros, blocks macros in files originating from the internet, blocks programmatic access to the VBA object model, and disables the Trust Bar for unsigned macros. Verification checks each policy setting and flags user-level overrides.

### User Application Hardening
Disables Internet Explorer 11, applies security policy settings to Microsoft Edge, Google Chrome and Mozilla Firefox, and verifies that Java browser plugins are not present. Compliance is reported per browser.

### Restrict Administrative Privileges
Enumerates and reports administrative accounts, disables the built-in Administrator, applies a stronger password policy to privileged accounts, restricts server logon rights, blocks network logon for local administrative accounts, and enables the privileged-account audit policy. Verification re-enumerates privileged accounts, checks policies, and reviews recent privileged logon events.

### Patch Operating Systems
Checks OS lifecycle status, configures Windows Update client settings and Windows Defender for vulnerability scanning, ensures the `PSWindowsUpdate` module is installed, installs OS security updates, reboots, and verifies pending updates against the ML1 timing threshold.

### MFA Prerequisites
Enables Network Level Authentication for RDP, hardens WinRM authentication methods, configures account lockout policy, enables authentication audit logging, and detects whether a third-party MFA provider is installed. Verification reports each prerequisite.

### Regular Backups
**Server:** Installs the Windows Server Backup feature, validates backup parameters, configures schedule and destination, verifies VSS and schedule, and runs an initial backup. Verification checks last backup status, destination isolation, and performs a test restore.

**Desktop:** Detects desktop OS, verifies the OneDrive sync client, checks Known Folder Move configuration and coverage, identifies local-only data, and verifies recovery path and sync status.

## Project Structure

Atomic steps follow a consistent naming convention:

- `configure*ml1` / `verify*ml1` — top-level blueprints for each strategy.
- `check*` — read-only audit steps used by `Verify` blueprints.
- `apply*` / `configure*` / `enable*` / `disable*` / `install*` — change-making steps used by `Configure` blueprints.
- `detect*` / `enumerate*` / `read*` — discovery and reporting steps.
- `output*compliancereport` — final compliance-report emitters used by `Verify` blueprints.

## Parameters

Set these on the plan before running:

| Parameter | Purpose |
|---|---|
| `Windows Server` | Target Windows Server 2022 to configure or verify. |
| `Windows Credential - svc-attune` | Administrator credential for the WinRM connection to the target. |
| `Windows Credential - svc-backup` | Credential used for backup operations on the target. |
| `Policy Export Path` | Local path on the target where AppLocker / WDAC policy XML will be archived. Example: `C:\E8\AppLocker`. |
| `WDAC Enforcement Mode` | `Audit` (log violations only) or `Enforce` (actively block). Always run `Audit` first and review Event Viewer before switching. |
| `Backup Destination Path` | Local or attached path where Windows Server Backup writes backups. |
| `Backup Retention Days` | Number of days of backup history to retain. |

## Maturity Level

This release targets **ML1**. Some steps (notably WDAC application control on servers) note where they implement controls ahead of ML2/ML3 requirements (e.g. ISM-1490, ISM-1656), but the project as a whole is scoped to ML1.
