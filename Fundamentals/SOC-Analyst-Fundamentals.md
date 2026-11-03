# SOC Analyst Fundamentals: Study Notes


Personal revision notes for the Tier 1 SOC Analyst skill areas, written while preparing for the Certified SOC Analyst assessment from The Cyber Ledger Academy. Organized by topic. See the references at the bottom for the primary sources each section should be checked against.

---

## 1. SOC Fundamentals

A Security Operations Center exists to monitor, detect, investigate and respond to security threats. It is usually described as three things working together: people, process and technology.

A Tier 1 analyst sits at the front of that pipeline. The job is watching the alert queue, triaging what comes in, documenting what was found, and escalating anything that needs deeper investigation.

| Term | Meaning |
|---|---|
| True positive | A genuine threat that was correctly detected |
| False positive | Harmless activity that was flagged as malicious |
| False negative | A genuine threat that went undetected |
| True negative | Harmless activity correctly left alone |
| Alert fatigue | So many alerts that analysts start overlooking real threats |

## 2. SIEM and Security Tooling

A SIEM collects logs from many sources, aggregates them, correlates related events and helps analysts search and investigate. Two ideas come up constantly:

- **Event correlation** links separate events into a larger attack pattern, for example repeated failed logins followed by a success.
- **Log normalization** converts logs from different sources into one consistent format so they can be searched and compared.

| Tool | Main function |
|---|---|
| Microsoft Sentinel, IBM QRadar, Splunk Enterprise Security | SIEM platforms |
| Wireshark | Network packet analysis |
| Sysmon | Detailed Windows activity logging |
| EDR | Endpoint detection and response |
| DLP | Data loss prevention |
| IDS | Detects suspicious activity and alerts |
| IPS | Detects and blocks malicious traffic |
| WAF | Protects web applications |
| Firewall | Filters network traffic |

A quick way to remember the IDS and IPS difference: IDS detects, IPS blocks.

## 3. Windows and Linux Logs

### Windows Security events

| Event ID | Meaning |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4688 | Process creation |
| 4720 | User account created |
| 1102 | Audit log cleared |

### Sysmon events

| Event ID | Meaning |
|---|---|
| 1 | Process creation |
| 3 | Network connection |
| 5 | Process terminated |
| 10 | Process access |
| 11 | File creation |
| 22 | DNS query |

### Linux authentication logs

- Debian and Ubuntu: `/var/log/auth.log`
- Red Hat family: `/var/log/secure`

A cleared audit log (1102) deserves attention on its own, since clearing logs is a common way to cover tracks.

## 4. Common Attacks and Malware

| Attack | What it is |
|---|---|
| Brute force | Trying many password combinations |
| Dictionary attack | Trying likely passwords from a wordlist |
| Rainbow table | Looking up precomputed password hashes |
| Credential stuffing | Reusing credentials stolen from another breach |
| Phishing | Fraudulent messages meant to steal information |
| Spear phishing | Phishing aimed at a specific person or group |
| Whaling | Phishing aimed at senior executives |
| Smishing | Phishing over SMS |
| Vishing | Phishing over voice calls |
| SQL injection | Injecting SQL statements through user input |
| XSS | Injecting malicious scripts into web pages |
| DDoS | Flooding a service until it is unavailable |
| Ransomware | Encrypts files and demands payment |
| Trojan | Malware disguised as legitimate software |
| Worm | Self-replicating malware |
| Keylogger | Records keystrokes |

## 5. Threat Intelligence Frameworks

**MITRE ATT&CK** is a knowledge base of adversary tactics and techniques built from real-world observations. Tactics are the attacker's goal at a stage, and techniques are how they achieve it. A few tactics worth knowing well:

| Tactic | Goal |
|---|---|
| Initial Access | Getting into the target |
| Execution | Running malicious code |
| Persistence | Keeping access over time |
| Discovery | Learning about the environment |
| Exfiltration | Stealing data |
| Impact | Disrupting or damaging systems |

This is a subset. The full matrix has more tactics, so check the current list on the ATT&CK site.

**Cyber Kill Chain** was developed by Lockheed Martin and describes seven stages of an intrusion: reconnaissance, weaponization, delivery, exploitation, installation, command and control, and actions on objectives.

## 6. Incident Response

A widely taught six-phase sequence:

**Preparation, Identification, Containment, Eradication, Recovery, Lessons Learned**

| Phase | Purpose |
|---|---|
| Preparation | Get tools, plans and training in place |
| Identification | Detect and confirm an incident |
| Containment | Limit the damage and stop the spread |
| Eradication | Remove malware and any persistence |
| Recovery | Restore systems to a clean state |
| Lessons Learned | Review what happened and improve |

**A note on NIST.** The older four-phase NIST model (preparation, detection and analysis, containment/eradication/recovery, post-incident activity) comes from SP 800-61 Rev. 2. NIST published Rev. 3 in April 2025, which supersedes Rev. 2 and is built around the CSF 2.0 functions instead. Rev. 2 is still useful for understanding the classic lifecycle, but it is no longer the current guidance.

Evidence handling matters throughout. Chain of custody documents who handled digital evidence and when, so it stays trustworthy.

## 7. Network Security Basics

| Protocol | Port | Note |
|---|---:|---|
| SSH | 22 | Encrypted remote access |
| Telnet | 23 | Unencrypted remote access |
| DNS | 53 | Resolves names to IP addresses |
| HTTP | 80 | Not encrypted by default |
| HTTPS | 443 | HTTP over TLS |

A port scan means attempting connections to many ports to find which services are listening.

## 8. Email Security and Endpoint Hygiene

| Control | What it does |
|---|---|
| SPF | Lists which servers may send mail for a domain |
| DKIM | Adds a cryptographic signature to verify a message |
| DMARC | Sets the policy for what to do when SPF or DKIM checks fail |
| MFA | Requires more than one authentication factor |
| Least privilege | Gives only the permissions needed |
| Patch management | Fixes known vulnerabilities |
| System hardening | Reduces the attack surface |
| 3-2-1 backups | Three copies, two media types, one offsite |

## 9. CIA Triad

| Principle | Protects against | Example of a failure |
|---|---|---|
| Confidentiality | Unauthorized access | Customer records leaked |
| Integrity | Unauthorized modification | Financial records altered |
| Availability | Loss of access | Site knocked offline by a DDoS |

## 10. Triage Habits (in my own words)

Rather than memorizing scenario answers, these are the habits behind them:

- A burst of failed logins is worth investigating as brute force, and failures followed by a success is a possible account compromise.
- Malware on an endpoint means isolating that host first, then investigating.
- Suspicious email attachments are never opened, they are submitted for analysis.
- If someone entered credentials on a fake site, reset the password and revoke active sessions.
- A log source that suddenly goes quiet is itself a finding, so check the forwarder and the network path.
- A critical asset such as a domain controller raises the priority of any alert on it.
- Unexpected account creation gets checked for who created it and what privileges it has.

---

## References to check these notes against

- Windows event IDs: Microsoft Learn security auditing documentation
- Sysmon events: Microsoft Sysinternals Sysmon documentation
- MITRE ATT&CK: attack.mitre.org
- Cyber Kill Chain: Lockheed Martin
- Incident response: NIST SP 800-61 Rev. 3 (csrc.nist.gov)
- Ports: IANA service name and port number registry
