# OWASP Top 10 (2025)

The OWASP Top 10 is the standard list of the most common types of web application security threats.

It is a ranking of categories, not a checklist of individual bugs. Each category groups many related weaknesses (CWEs), so the useful skill is recognizing an attack pattern and knowing which category it belongs to.


## The list

| # | Category | Meaning |
|---|----------|---------|
| A01 | Broken Access Control | Users can act or read beyond their permissions |
| A02 | Security Misconfiguration | Insecure settings in apps, servers, cloud or containers |
| A03 | Software Supply Chain Failures | Trust in dependencies, build pipelines or packages is broken |
| A04 | Cryptographic Failures | Sensitive data is poorly protected or encrypted |
| A05 | Injection | Untrusted input is run as a command or query |
| A06 | Insecure Design | The flaw is in the design, not just the code |
| A07 | Authentication Failures | The system does not verify identity properly |
| A08 | Software or Data Integrity Failures | Software, updates or data are trusted without verification |
| A09 | Security Logging and Alerting Failures | Attacks go unseen because logging or alerting is weak |
| A10 | Mishandling of Exceptional Conditions | The app handles errors and unusual states badly (new in 2025) |

### Quick Note: 

**Access control** is the security mechanism that determines who or what can view, use, or interact with specific data, systems, or network resources, while **least privilege** is a security principle that restricts users, apps, and processes to only the minimum permissions required to perform a specific task. Rather than competing concepts, least privilege is a guiding rule applied within access control systems.

**How Access Control Works** is that it needs **Identification** (needs to identify the person), **Authentication** (verifying the identity), **Authorization** (Deciding what actions the verified user/system is allowed to do), and **Auditing** ( Logging and reviewing access events to maintain accountability and enforce least privilege)

## Examples per category

- **A01 Broken Access Control:** changing an ID in a request to see another user's data, reaching admin pages without being an admin.
- **A02 Security Misconfiguration:** default credentials, debug mode on in production, missing security headers, unneeded services running.
- **A03 Software Supply Chain Failures:** malicious packages, a compromised build pipeline, a tampered container image.
- **A04 Cryptographic Failures:** weak algorithms, missing encryption, poor key management, passwords stored without proper hashing.
- **A05 Injection:** SQL injection, command injection, LDAP injection, NoSQL injection.
- **A06 Insecure Design:** no rate limiting, no abuse protection, a workflow that can be skipped.
- **A07 Authentication Failures:** weak password rules, no MFA, insecure session handling, tokens not validated.
- **A08 Software or Data Integrity Failures:** unsigned updates, insecure deserialization, a compromised deployment pipeline.
- **A09 Security Logging and Alerting Failures:** no audit logs, logs nobody monitors, no alerts on suspicious behavior. This is the gap a SOC exists to close.
- **A10 Mishandling of Exceptional Conditions:** poor error handling, failing open instead of failing securely. OWASP's example is an upload handler that catches an exception but never releases its resources, so repeated uploads cause a denial of service.

## What it looks like in logs

These four are the ones a Tier 1 analyst is most likely to meet. Each point is tagged with its source. Exact field names and log formats depend on the log source, so these are patterns to look for and not ready-made queries.

### A05 Injection

- OWASP's own SQL injection example is a request where the `id` parameter carries `' OR '1'='1`, which makes the query return every record. [OWASP A05](https://top10.owasp.org/2025/A05_2025-Injection/)
- OWASP's command injection example is a `domain` parameter carrying `example.com; cat /etc/passwd`, where a shell command follows the legitimate value. [OWASP A05](https://top10.owasp.org/2025/A05_2025-Injection/)
- MITRE's detection strategy for exploiting public-facing applications lists exploit-like strings to search for in web logs, such as `../`, `union select`, `cmd=` and `${jndi:`. [MITRE DET0080](https://attack.mitre.org/detectionstrategies/DET0080)
- MITRE describes the usual chain: suspicious requests in the access log, then a spike in 5xx errors or WAF blocks, then the web server process spawning a shell or a tool like curl or wget, then an outbound callback. The last two steps are what separate a probe from a successful exploit. [MITRE DET0080](https://attack.mitre.org/detectionstrategies/DET0080)
- Inference: in raw access logs these strings often appear URL-encoded, so a search for the plain text can miss them.

### A01 Broken Access Control

- OWASP's examples are a parameter changed to another account's value (`acct=notmyacct`) and a user requesting an admin URL directly. If a non-admin can open the admin page, that is the flaw. [OWASP A01](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/)
- OWASP also lists missing access controls on API POST, PUT and DELETE calls, and tampering with JWTs or cookies to raise privileges. [OWASP A01](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/)
- OWASP says access control failures should be logged and that repeated failures should alert an administrator. [OWASP A01](https://top10.owasp.org/2025/A01_2025-Broken_Access_Control/)
- Inference: a standard user account getting successful responses on admin paths, or one account requesting many different record IDs in a short time, is worth escalating.

### A07 Authentication Failures

- MITRE separates three login attack patterns, and each one looks different in the logs:
  - Password guessing: repeated failures against an account, sometimes followed by a success from the same source. [T1110.001](https://attack.mitre.org/techniques/T1110/001/)
  - Password spraying: one password, or a small list, tried against many accounts. MITRE notes attackers may throttle attempts to stay under alert thresholds. [T1110.003](https://attack.mitre.org/techniques/T1110/003/)
  - Credential stuffing: many different username and password pairs from one source, taken from breaches of unrelated sites. [T1110.004](https://attack.mitre.org/techniques/T1110/004/)
- The pattern that matters most across all three is a burst of failed logins followed by a success from the same user, host or time window. [MITRE T1110](https://attack.mitre.org/techniques/T1110)
- OWASP adds that attackers now vary leaked passwords, for example changing `Winter2025` to `Winter2026`. [OWASP A07](https://top10.owasp.org/2025/A07_2025-Authentication_Failures/)
- MITRE's password guessing page mentions Windows event ID 4625 as the logon failure event. [T1110.001](https://attack.mitre.org/techniques/T1110/001/)

### A09 Security Logging and Alerting Failures

This one is about gaps in your own visibility, so the signs are often things that are missing. All of these come from the OWASP description. [OWASP A09](https://top10.owasp.org/2025/A09_2025-Security_Logging_and_Alerting_Failures/)

- Only successful logins are logged, with failed attempts missing.
- Logs are stored only on the local machine and not backed up or centralized.
- Security scans and penetration tests trigger no alerts.
- Alerts exist but nobody reviews them in a reasonable time, or too many false positives bury the real ones.
- The playbook for an alert is missing or out of date.

## References

- [OWASP Top 10:2025](https://owasp.org/Top10/2025/)
- [A01, A05, A07, A09 and A10 category pages](https://top10.owasp.org/2025/)
- [MITRE ATT&CK T1190 Exploit Public-Facing Application](https://attack.mitre.org/techniques/T1190/)
- [MITRE ATT&CK T1110 Brute Force](https://attack.mitre.org/techniques/T1110)
- [MITRE DET0080 detection strategy](https://attack.mitre.org/detectionstrategies/DET0080)
