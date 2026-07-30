# *Penetration Testing, Threat Hunting, and Cryptography* Final Project
## Part 1: Perform Vulnerability Analysis and Penetration Testing
You've recently been hired as an ethical hacker by SecureBank, a mid-sized financial institution known for its rapidly growing digital services. In the wake of recent incidents in the banking sector, the management has become increasingly anxious about potential vulnerabilities in SecureBank's online banking platform. Preliminary investigations have highlighted weak security measures. To address these risks, the bank has tasked you with identifying vulnerabilities and investigating potential threats.
### Task 1: Identify a vulnerability
Your first task will be to identify any vulnerabilities that might expose the bank's systems to cyber threats.

[CVE-2024-47575](https://nvd.nist.gov/vuln/detail/CVE-2024-47575) is a critical authentication flaw in Fortinet FortiManager and FortiManager Cloud. The vulnerability, rated 9.8 under CVSS v3.1, allows an unauthenticated remote attacker to send crafted requests to the fgfmd service and execute arbitrary code or commands. It affects several releases from versions 6.2 through 7.6. Attackers exploited the flaw as early as June 2024 to exfiltrate FortiGate configurations, device information, usernames, and hashed passwords. CISA lists it as a known exploited vulnerability. Organizations should immediately install Fortinet’s fixed releases, restrict access to trusted devices and IP addresses, check for compromise indicators, and investigate any potentially unauthorized device registrations.

### Task 2: Investigate the selected vulnerability using Google Dorking commands
Your second task will be to further investigate the identified vulnerability using Google Dorking commands. You will use specific search queries to uncover any exposed or publicly available sensitive information that could potentially compromise the bank's systems.

- Google Cloud Documentation: https://cloud.google.com/blog/topics/threat-intelligence/fortimanager-zero-day-exploitation-cve-2024-47575
- Rapid7 Documentation: https://www.rapid7.com/blog/post/ra-cve-2024-47575-analysis/
- Fortiguard Documentation: https://www.fortiguard.com/psirt/FG-IR-24-423
- watchTowr Labs PoC: https://github.com/watchtowrlabs/Fortijump-Exploit-CVE-2024-47575

### Task 3: Create a penetration testing plan
SecureBank's management wants you to conduct a penetration test on their network. Once you have identified the vulnerabilities, you will need to create a penetration testing plan. This plan will outline your strategy for simulating cyberattacks to test the network's defenses and detect weak points. In this task, you will make decisions that will affect the outcome of the penetration test.
#### Scenario
You are in a meeting with SecureBank’s IT team to discuss the penetration test. The team includes the IT manager, the network administrator, and a security officer. They provide you with an overview of their infrastructure, which includes external web applications, internal databases, and a mix of on-premises and cloud-based services.

**What should you include in the scope?**

The scope should identify every authorised target and boundary. Record asset owners, business criticality, and data sensitivity. Define permitted testing methods, access levels, supplied credentials, testing windows, and evidence-handling requirements. State exclusions, third-party restrictions, prohibited techniques, rate limits, and acceptable disruption. Include escalation contacts, stop conditions, incident procedures, reporting expectations, and retesting arrangements. SecureBank and the testing team should approve the final scope in writing before any active work begins.

#### Scenario
You are required to define the primary objectives of the penetration test. The IT team has expressed concerns about recent phishing attacks and regulatory compliance requirements.

**Which objective should be prioritized for the penetration test based on the IT team’s concerns?**

The primary objective should be to assess SecureBank's resilience to phishing-led account compromise and determine whether a compromised identity could expose regulated systems or data. With explicit written authorisation, conduct a phishing simulation that tests email filtering, user reporting, authentication controls, conditional access, and incident response. Use harmless payloads, avoid collecting real passwords, and apply agreed stop conditions. Then evaluate whether a simulated identity compromise could enable privilege escalation, lateral movement, or access to critical assets. Map each finding to applicable regulatory controls and evidence requirements. This objective addresses the immediate threat while producing compliance evidence and remediation priorities.

#### Scenario
You have to establish the rules of engagement for the penetration test. However, the IT team is concerned about potential disruptions to business operations.

**What approach will you take to minimize disruptions while conducting the penetration test?**

To minimise disruption, establish a risk-based, staged testing plan with SecureBank before testing begins. Start with passive discovery and low-impact validation, then progress to controlled exploitation only after explicit approval. Schedule intrusive activities during agreed maintenance windows, use test accounts and non-production replicas where practical, and rate-limit scanners. Exclude denial-of-service, destructive payloads, persistence, and uncontrolled data extraction unless separately authorised. Coordinate through a nominated SecureBank contact, provide live progress updates, monitor critical services throughout testing, and pause whenever instability, unexpected access, or sensitive data exposure occurs.

#### Scenario
You begin the Discovery phase to gather information about the target systems. However, the IT team has provided you with limited information about their network topology.

**Which approach will be most effective for gathering information?**

Use an iterative discovery approach that combines passive evidence collection with controlled, authorised network enumeration. Begin by interviewing administrators and reviewing asset inventories, configurations, and logs. Build a provisional topology showing systems, trust boundaries, and ingress paths. Validate it through host discovery, port scanning, service identification, and authenticated configuration checks within the approved scope. Cross-reference sources, record unreachable or unmanaged assets, and confirm discrepancies. This approach improves coverage without assuming that documentation or scan results are complete.

#### Scenario
You have identified several vulnerabilities and are ready to exploit them. The vulnerabilities include an outdated web server, weak passwords, and an unpatched database.

**Which approach will you take to exploit the identified vulnerabilities?**

Use a risk-based, minimally invasive validation strategy. First, confirm each asset's version, configuration, exposure path, and compensating controls. Obtain explicit approval for each exploit technique and define success criteria, rate limits, stop conditions, and rollback steps. Test the outdated web server and unpatched database with safe proof-of-concept checks. Assess weak passwords using approved test accounts, limited password candidates, lockout safeguards, and monitored attempts. Stop after obtaining sufficient evidence of impact. Avoid persistence, destructive payloads, and unnecessary data access. Preserve evidence, remove test artefacts, and report unexpected sensitive access immediately.

#### Scenario
You now have to compile your findings and provide recommendations. Your audience includes technical staff and the executive leadership.

**What is the most effective way to present your findings?**

Present the findings in a layered report supported by separate executive and technical briefings. The executive summary should explain overall risk, affected business services, regulatory exposure, and residual risk, likely impact, and remediation priorities in plain language. The technical section should document scope, methods, evidence, affected assets, validation status, exploit paths, reproduction details, existing controls, recommended fixes, and retest criteria. Rank findings using exploitability, exposure, asset criticality, business impact, and control strength.

## Part 2: Secure Information Using Symmetric Encryption
### Scenario
Imagine you have recently been hired as an ethical hacker by SecureBank, a mid-sized financial institution known for its rapidly growing digital services. In the wake of recent incidents in the banking sector, the management has become increasingly anxious about potential vulnerabilities in SecureBank's online banking platform. Preliminary investigations have highlighted weak security measures, including inadequate cryptographic protections for sensitive financial data. To address these risks, the bank has tasked you with strengthening its security posture through advanced cryptographic techniques.

During the penetration testing exercise, you discovered a confidential file named userdetails on a web server. The file contains important credentials stored in plain text format, which poses a significant security risk, as unauthorized individuals could easily access them. Your task is to secure this information by implementing proper encryption techniques and demonstrating how to manage sensitive data securely.
#### Preparation
```bash
mkdir -p "$HOME/secure-encryption-lab"
chmod 700 "$HOME/secure-encryption-lab"
cd "$HOME/secure-encryption-lab"
umask 077
openssl version
```
#### Step 1: Create userdetails
```bash
cat > userdetails <<'EOF'
Username: admin
Password: P@ssw0rd123
Email: admin@example.com
Phone: 123-456-7890
EOF
```
#### Step 2: Generate the AES passkey
Generate 32 cryptographically random bytes represented as hexadecimal:
```bash
openssl rand -hex -out passkey 32
echo "Key generation exit status: $?"
chmod 600 passkey
ls -l passkey
```
An exit status of `0` indicates success. The passkey contains a 256-bit random secret.
#### Step 3: Encrypt userdetails
```bash
openssl enc -aes-256-cbc -e -salt -pbkdf2 -iter 200000 -md sha256 -in userdetails -out userdetails_encrypt -pass file:./passkey
echo "Encryption exit status: $?"
chmod 600 userdetails_encrypt
ls -l userdetails_encrypt
```
The options provide:
* AES-256-CBC encryption
* A random salt
* PBKDF2 key derivation
* 200,000 derivation iterations
* SHA-256 for key derivation
#### Step 4: Display the encrypted content
Use `cat -v` so binary control characters cannot disrupt the terminal:
```bash
cat -v userdetails_encrypt
```
The output should be unreadable and will normally begin with `Salted__`. It will differ each time because OpenSSL generates a new salt.
#### Step 5: Decrypt the file
The decryption parameters must match those used during encryption:
```bash
openssl enc -aes-256-cbc -d -pbkdf2 -iter 200000 -md sha256 -in userdetails_encrypt -out userdetails_decrypt -pass file:./passkey
echo "Decryption exit status: $?"
chmod 600 userdetails_decrypt
ls -l userdetails_decrypt
```
#### Step 6: Display and verify the decrypted content
```bash
cat userdetails_decrypt
sha256sum userdetails userdetails_decrypt
```
The credentials should match the original file. The two SHA-256 hashes should also be identical.