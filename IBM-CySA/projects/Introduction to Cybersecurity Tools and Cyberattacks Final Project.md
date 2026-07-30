# *Introduction to Cybersecurity Tools and Cyberattacks* Final Project
## Overview
In this project, you will explore a real-world-inspired scenario, identify potential security vulnerabilities, and provide recommendations to strengthen the organization's overall security posture.

This final project comprises three tasks:
1. Evaluate the organization's existing security infrastructure
2. Create a multifactor authentication (MFA) plan
3. Evaluate the organization's existing physical security measures
## Scenario
You are an IT Security Analyst at TechSolutions Inc. The company has recently experienced a data breach due to compromised credentials. TechSolutions Inc. currently employs the following security infrastructure.

The traditional access control methods include:
- Username and password sign-in for each system and application.
- SSH keys to access remote servers.
- Smart cards for secure physical and digital access across multiple company locations and the corporate VPN.

The resources that employees require access to:
- **Corporate email system:** Essential for day-to-day communication, both internally and with external stakeholders.
- **Internal databases:** Contains sensitive information such as client details, project data, and financial records.
- **Cloud storage services:** Used for storing and sharing documents and collaborative work.
- **HR systems:** Incorporates personal information of employees, benefits administration, and recruitment data.
- **Customer relationship management (CRM) software:** Critical for sales, marketing, and customer service operations.
- **Project management tools:** Used for tracking progress, assigning tasks, and optimizing workflow.
- **Development environments:** Required for software development, testing applications, and managing code repositories.
- **Company intranet and knowledge bases:** Provides access to company policies, training materials, and internal communication platforms.
---
### Questions:
**List three potential security concerns within the existing security framework, mainly focusing on areas that could have contributed to the compromise of credentials.**

1. Single-factor password authentication: Each system and application accepts a username and password. If an attacker obtains a password through phishing, reuse, malware, or another method, the description identifies no second factor that would stop the sign-in. This concern is especially serious for email, HR systems, internal databases, and development environments because those resources contain sensitive information or provide access to additional systems.
2. Decentralized credentials and possible password reuse: Separate sign-ins across many systems can create credential sprawl. Employees may reuse passwords, and administrators may apply password, lockout, recovery, and revocation rules inconsistently. An exposed password could therefore provide access to more than one resource, while separate account stores could delay complete containment.
3. Unspecified SSH key lifecycle controls: SSH keys are strong credentials when managed correctly, but the scenario does not describe inventory, expiration, passphrase protection, secure storage, or revocation. A copied private key could give an attacker remote server access, particularly if the key is long-lived or remains authorized after an employee changes roles or leaves the company.

**Provide a high-level solution for each of the three identified security concerns.**

1. Add phishing-resistant MFA: Require MFA through a central identity provider for all supported systems. Prioritize email, remote access, administrative accounts, HR systems, internal databases, and development environments. Use a company-issued smart card or security key as the possession factor, block legacy sign-in methods that bypass MFA, and test that access fails when the second factor is absent.
2. Centralize identity and access management: Use single sign-on where possible, apply a password standard, prohibit known compromised passwords, and provide an approved password manager. Connect account creation, role changes, and deactivation to a documented process. Review access regularly, remove unused accounts, and verify that disabling one identity blocks every connected service.
3. Establish an SSH credential lifecycle: Maintain an owner-approved inventory of SSH keys, protect private keys with passphrases and secure storage, and replace long-lived keys with short-lived SSH certificates where practical. Remove unused keys promptly, revoke access during role changes and departures, and regularly test that revoked credentials can no longer connect.

**Select the authentication factor you consider the most secure and practical for TechSolutions Inc.** 

The most secure and practical choice is a possession factor, specifically the company-issued smart card that the company already uses. Reusing the existing smart-card program can reduce deployment cost and user disruption. When the card performs certificate-based authentication, it proves possession without transmitting a reusable card secret to the application. The company should require a PIN to unlock the card, maintain spare-card and recovery procedures, and revoke lost or stolen cards immediately.

Explain how the two authentication factors will work together to create an MFA plan for TechSolutions Inc.

The MFA plan would combine something the employee knows, the account password, with something the employee has, the assigned smart card. The employee first enters a username and password through the central identity provider. The identity provider then asks the employee to insert or tap the registered smart card and unlock it with a PIN. The card completes a cryptographic challenge, and access is granted only if both the password and card verification succeed.

This design limits the value of a stolen password because the attacker would also need the assigned card. A stolen card alone would also be insufficient because the attacker would still need the employee's password and the card PIN. The company should enforce MFA for remote and sensitive access, record authentication events, alert on repeated failures, and provide a controlled recovery process that verifies identity before issuing a replacement card.

---
## Scenario
TechSolutions Inc. currently employs the following physical security measures:
- **Reception and lobby:** The entrance features a reception desk that is always manned. The receptionist takes the names of each visitor and issues temporary visitor badges.
- **Employee workspaces:** The open-plan workspaces for employees are accessible from two directions. One entrance, located near the receptionist's desk, leads to the front of the workspace area, while another door at the back opens into the employee parking lot. Both entrances are clearly marked with signs reading "Authorized Personnel Only." The receptionist locks the front door, and the last person leaving for the day locks the back door.
- **Meeting rooms and conference halls:** Larger meeting spaces are adjacent to the shared workspace. Each of these spaces has doors that can be locked. Reservations for these spaces can be made through a central calendar system.
- **Data centers and server rooms:** These critical areas are equipped with lockable doors, with access restricted to the IT and janitorial staff. Additionally, these rooms are fitted with a thermostat and humidity sensor for monitoring purposes.
- **File storage:** File cabinets with locking mechanisms are strategically located throughout the office space to ensure sensitive documents are secure and accessible only by authorized personnel.
- **Parking lot:** The employee parking lot is enclosed by a fence with only one entry point. Parking stickers are required to access this parking lot.
- **Common areas and facilities:** Breakrooms and restrooms are adjacent to the common work area. The doors leading to these areas do not have locks.
---
### Questions:
**Identify security vulnerabilities in the physical infrastructure or policies of TechSolutions Inc. (Select three).** 
**Provide one recommendation for each of the three identified physical security vulnerabilities. These recommendations must be practical, address the concern effectively, and suggest a clear path for remediation or improvement.**

1. Weak control of the rear workspace entrance: The rear door opens directly from the employee parking lot into the workspace, and the last employee leaving is responsible for locking it. A sign does not prevent entry, and a manual closing process can leave the door unlocked or allow tailgating.

Recommendation: Install a badge reader with an automatic closer and access logging on the rear door. Configure an alert for a door held open, and review the access log after an alarm or reported incident. A functional test should confirm that the door locks automatically and rejects inactive badges.

2. Incomplete visitor management:  The receptionist records visitor names and issues temporary badges, but the policy does not mention identity verification, host approval, badge expiration, escort requirements, or badge return. A visitor could provide a false name, enter without confirmed business need, or retain a badge for later use.

Recommendation: Implement a documented visitor-management process that requires identity verification, host confirmation, time-limited badges, visible escort status, and badge return at departure. The receptionist should reconcile issued and returned badges at the end of each day.

3. Excessive server-room access: Both IT and janitorial staff can enter the data centers and server rooms. Routine janitorial access gives nontechnical personnel unnecessary proximity to servers, network equipment, and storage media. This broad access increases the risk of unauthorized viewing, accidental disruption, device theft, or installation of unauthorized equipment.

Recommendation: Restrict server-room badge access to approved IT personnel and require an authorized escort for cleaning or maintenance staff. Review access logs regularly, and verify the control by testing that a janitorial badge cannot open the door without an approved temporary exception.
