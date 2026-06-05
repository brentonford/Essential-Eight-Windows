Automate the configuration and verification of the Australian Signals Directorate (ASD) **Essential Eight** mitigation strategies, at **Maturity Level 1 (ML1)**, on Windows targets.

Reference: [ASD Essential Eight Explained](https://www.cyber.gov.au/business-government/asds-cyber-security-frameworks/essential-eight/essential-eight-explained)

## Disclaimer
 
Cybersecurity is highly context-specific. What is appropriate for one organisation's size, industry, data sensitivity, regulatory environment, threat profile, and existing controls can be irrelevant — or even harmful — in another. Frameworks like the Essential Eight (ASD guidance) are explicitly a **baseline recommendation, not a guarantee of security or compliance**, and must be tailored.
 
This project automates a defensible interpretation of ML1 controls for Windows targets. It is not a substitute for:
 
- A risk assessment of your environment.
- Architectural review by a qualified security practitioner.
- Change management, testing, and rollback planning before applying configuration to production.
- Ongoing monitoring, vulnerability management, and incident response capability.
Some steps make changes that can disrupt access (account policy changes, RDP/WinRM hardening, application control enforcement, browser policy lockdowns). **Always run in a representative test environment first, run application control in `Audit` mode and review event logs before switching to `Enforce`, and confirm you can recover the system before applying broadly.**