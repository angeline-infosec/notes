## CyberAttacks Notes:

Each note follows: what it is, how it works, how attackers do it, the risks, a real case, how to spot it (SOC view), how to fix it, and a MITRE ATT&CK reference.


## 1. SSRF (Server-Side Request Forgery)

SSRF is a vulnerability that tricks a web server into making requests to internal or external systems on behalf of an attacker.

### How SSRF works

- **User input:** The app takes a user-supplied URL to fetch something remote, like a profile picture or a PDF.
- **Missing validation:** The server doesn't check or restrict where that request goes.
- **Trusted position:** The server sits inside the network, so it can reach things the attacker can't (localhost, private IPs, cloud metadata).

### How attackers perform SSRF

1. **Input manipulation**
   - Normal: the app imports a picture from `https://example.com/photo.jpg`
   - Attack: the attacker swaps the URL for an internal target, like `http://localhost/admin` or `http://169.254.169.254/latest/meta-data/`
   - Result: the server makes the request and hands the private response back to the attacker.
2. **Other URL schemes**
   - `file:///etc/passwd` reads local files on the server.
   - `gopher://` or `dict://` can send raw bytes to internal services like Redis or Memcached.
3. **Bypassing weak filters** (for example, a filter that blocks the word "localhost")
   - IP encoding: `127.0.0.1` can be written as octal `0177.0.0.1`, hex `0x7f000001`, or decimal `2130706433`. IPv6 `::1` also works.
   - DNS rebinding: a domain that points to a safe public IP during the check, then flips to an internal IP when the server makes the real request.
   - Redirects: the attacker's URL passes the check, then redirects to an internal address.
4. **Blind SSRF:** The server makes the request but never shows the response. The attacker confirms it worked through timing or by watching their own server for an incoming call.

### The risks

- **Stealing cloud credentials:** On AWS, Google Cloud, or Azure, the metadata service at `169.254.169.254` can return temporary access keys. That can lead to a full cloud compromise.
- **Internal network mapping:** The server is used as a proxy to scan internal IPs and ports (22 for SSH, 3306 for MySQL).
- **Reaching internal admin tools:** Dashboards and consoles often have weak login because "they're behind the firewall."
- **Remote code execution:** Sending commands to unauthenticated internal services like Redis can give the attacker control of internal servers.

### Real case

Capital One, 2019. An attacker used SSRF against a misconfigured web application firewall to pull role credentials from the AWS metadata service, then used them to download data from S3 buckets.

### Detection (SOC view)

- Outbound requests from a web or app server to `169.254.169.254` or to private ranges (10.x, 172.16.x, 192.168.x, 127.x).
- URL parameters in web logs that contain IP addresses, `localhost`, `file://`, `gopher://`, or encoded IPs.
- A server suddenly making many connections to different internal hosts or ports (looks like a scan).
- Cloud logs showing role credentials used from an IP outside the cloud environment.

### Solution

1. Allowlist the domains the app is allowed to call. Don't rely on a blocklist.
2. Block requests to private, loopback, and link-local ranges after DNS resolution.
3. Turn off or limit redirects and unused URL schemes (file, gopher, dict).
4. Use IMDSv2 on AWS, which needs a session token and stops simple SSRF from reading metadata.
5. Put internal services behind authentication anyway, and use network segmentation.

**MITRE ATT&CK:** T1190 (Exploit Public-Facing Application), T1552.005 (Cloud Instance Metadata API)

---

## 2. IDOR (Insecure Direct Object Reference)

IDOR happens when an application exposes a direct reference to an object (a record, file, or database key) and doesn't check whether the requesting user is allowed to access that specific object. It falls under Broken Access Control, which is #1 in the OWASP Top 10. In the OWASP API Top 10 it appears as BOLA (Broken Object Level Authorization).

### How IDOR works

- **User input:** The app uses an identifier from the URL, a form field, or an API parameter to fetch a resource.
- **Missing validation:** The server returns the object based on the ID alone and never checks ownership.
- **Trusted position:** The app assumes that if you can reach the endpoint, you're allowed to see whatever the ID points to.

The gap here is between authenticated (the server knows who you are) and authorized (the server checks what you're allowed to touch). IDOR is a failure of the second one.

### How attackers perform IDOR

1. **Input manipulation**
   - Normal: a logged-in customer opens `https://shop.example.com/invoice?id=1001`
   - Attack: the attacker changes it to `id=1002`
   - Result: if ownership isn't checked, the server returns someone else's invoice.
2. **Common patterns**
   - Sequential or predictable IDs that can be incremented.
   - Parameter tampering in a hidden form field or API body (`"user_id": 45` to `46`).
   - Horizontal escalation: reaching another user's data at the same privilege level.
   - Vertical escalation: reaching admin-only objects or functions.
   - API-based IDOR: endpoints like `/api/users/123/profile`, common in mobile app backends.
3. **Working around partial checks**
   - Session reuse: a valid low-privilege session with a swapped object ID. The login check passes, the ownership check never happens.
   - Method switching: the check exists for GET but not for POST, PUT, or DELETE.
   - Response comparison: request an owned object and a non-owned one and compare the responses.

A related issue is a guessable file name on a static path (for example, a PDF named by sample ID plus date). That is closer to forced browsing than classic IDOR, since there may be no login check at all, but the fix is the same idea: access must be tied to the user, not to a guessable name.

### The risks

- **Data theft:** Personal, financial, or medical records of other users.
- **Data tampering:** Editing or deleting another user's data.
- **Account takeover:** Changing another user's email or reset details.
- **Mass exposure:** Scripting through a range of IDs to scrape thousands of records.

### Real case

First American Financial, 2019. Document links used sequential numbers, and changing the number showed other customers' documents. Hundreds of millions of records were exposed.

### How to test for it

- Use two accounts (A and B). Capture A's request, swap in B's object ID, and see if A can read or change it.
- Tools: Burp Suite Repeater and Intruder, plus the Autorize extension to replay requests automatically with a different session.
- Test every method (GET, POST, PUT, DELETE), not only the one the app uses normally.

### Detection (SOC view)

- One IP or session requesting many different or sequential IDs in a short time.
- A burst of 200 responses mixed with 403 or 404 responses for the same endpoint.
- A user requesting object IDs far outside their normal range.
- Log sources: web server logs, API gateway logs, WAF logs.
- Tier 1 triage: identify the source IP and account, count how many unique IDs were requested, check whether the 200s returned data, and escalate if data was exposed.

### Solution

1. **Server-side authorization on every request.** Check that the logged-in user owns or has permission for that specific object. This is the real fix.
2. Use random IDs (UUIDs) generated on the server and stored in the database. This makes guessing harder, but it is not access control. If a UUID leaks through a shared link, browser history, or logs, whoever has it gets in.
3. Indirect reference maps, so the ID shown to the user isn't the real database key.
4. Test access control in development and QA by trying other users' IDs, not only the normal path.
5. For sensitive files, use signed, expiring links or password protection tied to the recipient.

**MITRE ATT&CK:** Loosely T1190 (Exploit Public-Facing Application). IDOR is mostly tracked through OWASP, not ATT&CK.


**IDOR ALT NOTE**: 
IDOR is a security vulnerability that occurs when an application exposes a direct reference to an internal object (a file, record, or database key) and fails to verify that the requesting user is actually authorized to access that specific object.

How IDOR Works User Input: An app uses an identifier (ID, filename, or reference) supplied in a URL, form field, or API parameter to fetch a specific resource. Missing Validation: The server retrieves the resource based on the identifier alone, without checking whether the current user has permission to view or modify that particular object. Trusted Position: The application assumes that if a user can reach the endpoint, they're allowed to access whatever object the identifier points to — the identifier's predictability becomes the flaw.

How Attackers Perform IDOR Attackers manipulate a reference in a request to access data or perform actions belonging to another user.

Input Manipulation Normal Behavior: A patient views their own lab report at: https://sc1.example.com/pdffiles/12345677-2026-01-01.pdf

Manipulation: The attacker simply changes the ID/date in the URL: https://sc1.example.com/pdffiles/26536378-2026-11-19.pdf

The Result: If the server doesn't verify ownership, it returns another patient's report — because the reference was predictable (sequential ID, sample ID + date combo) and access wasn't tied to the logged-in user.

Common IDOR Patterns Sequential/Predictable IDs: Numeric IDs (invoice=1001, invoice=1002) that can simply be incremented to enumerate other users' records. Parameter Tampering: Changing a hidden form field or API body parameter (e.g. "user_id": 45 → "user_id": 46) rather than the URL itself. Horizontal Privilege Escalation: Accessing another user's data at the same privilege level (another customer's order, another patient's report). Vertical Privilege Escalation: Manipulating a reference to reach admin-only or higher-privilege resources. API-Based IDOR: REST/GraphQL endpoints (e.g. /api/users/123/profile) exposing object IDs directly, common in mobile app backends where the frontend doesn't hide requests.

Bypassing Defenses If a developer adds partial checks (like verifying the user is logged in, but not that they own the object), attackers can still succeed: Session Reuse: Using a valid, low-privilege session token while simply swapping the object ID — authentication passes, but authorization is never actually checked. Method Switching: Trying the same endpoint with different HTTP methods (GET vs POST vs DELETE) in case the authorization check was only implemented for one of them. Response Comparison: Sending requests for both an owned and a non-owned object ID, and comparing responses to confirm the missing access control.

The Risks Once an attacker can freely swap references, they exploit the gap between "authenticated" and "authorized."

Data Theft/Privacy Breach: Viewing other users' personal, financial, or medical records — as in the lab report example. Data Tampering: Editing, deleting, or overwriting another user's data (e.g. changing someone else's order or profile). Account Takeover: Modifying account details (email, password reset tokens) belonging to another user. Mass Data Exposure: Automating ID enumeration (scripting through a range of IDs) to scrape large volumes of records at once.

Solution:

Server-Side Authorization Checks — always verify the requesting user owns or has explicit permission for the specific object, on every request, not just that they're logged in.
Unpredictable Identifiers (UUIDs) — replace sequential or guessable IDs with random UUIDs (e.g. via crypto.randomUUID()) so references can't be enumerated.
Indirect Reference Maps — map exposed IDs to internal database keys per-session, so the ID shown to the user never directly maps to the real record.
Access Control Testing — explicitly test each endpoint by trying to access another user's object ID during development/QA, not just the happy path.
Signed/Password-Protected Resources — for files like PDFs, require a unique token or password tied to the intended recipient rather than relying on obscurity.
eg. Access to patients reports/pdf via url means. https://sc1.sukraa.in/LeoLab/pdffiles/_xxxxx.pdf_

Here, the last part (italicized), poses the vulnerability. This specific URL uses the combination of sample ID + Date. This makes it easier for anyone to access other patient records using random IDs and dates. for eg. 12345677-2026-01-01.pdf or 26536378-2026-11-19.pdf (each would be a pdf record of a patient)

Solution: 1. Password protected pdf (unique identifier can be used to open the pdf file that's only accessible to the patient/family) 2. UUID (random unique ID) It's when a developer opens the pdf -> open developer tools -> typer in crypto.randomUUID () -> enter (gives you the ID) -> Save this ID to the database -> Rename the pdf with this random UUID.


## 3. SQL Injection (SQLi)

SQL injection lets an attacker interfere with the database queries an application makes by injecting SQL through user input.

### How SQLi works

- **User input:** The app puts user data (login form, search box, URL parameter) straight into a SQL query string.
- **Missing validation:** The input isn't sanitized or parameterized.
- **Trusted position:** The database runs whatever query it receives and can't tell which part came from an untrusted user.

### How attackers perform SQLi

1. **Input manipulation**
   - Normal query: `SELECT * FROM users WHERE username = 'angie' AND password = 'pass123';`
   - Attack: the attacker enters `' OR '1'='1' -- ` as the username (note the space after the dashes).
   - Resulting query: `SELECT * FROM users WHERE username = '' OR '1'='1' -- ' AND password = '';`
   - The `--` comments out the rest, including the password check. Since `'1'='1'` is always true, the query returns a user row and the attacker logs in.
2. **Common techniques**
   - **Union-based:** uses `UNION` to attach results from another table (`' UNION SELECT username, password FROM admin--`).
   - **Error-based:** triggers database errors that leak table and column names.
   - **Boolean-based blind:** no data shown, so the attacker asks true/false questions and watches for small differences in the page (`' AND 1=1--` vs `' AND 1=2--`).
   - **Time-based blind:** uses `SLEEP(5)` and measures the delay.
   - **Out-of-band:** makes the database send DNS or HTTP requests to the attacker's server to leak data.
3. **Bypassing weak filters**
   - Comment syntax (`--`, `#`, `/* */`) to cut off the rest of the query.
   - URL, hex, or char encoding to sneak past naive filters.
   - Mixed case (`SeLeCT`) or comments between keywords to beat regex filters.

### The risks

- **Data theft:** usernames, password hashes, card numbers, personal records.
- **Authentication bypass:** logging in as any user, including admin.
- **Data tampering:** changing or deleting records, or wiping logs.
- **File access and command execution:** on some database setups, injected queries can read or write server files or run OS commands.
- **Full database compromise:** dumping whole databases, including other apps on the same server.

### Real cases

- Heartland Payment Systems, 2008: SQL injection was the entry point for one of the largest card data breaches at the time.
- TalkTalk, 2015: a simple SQLi attack exposed customer data and led to a large fine.

### Detection (SOC view)

- Web or WAF logs containing `'`, `%27`, `UNION`, `SELECT`, `SLEEP`, `--`, or `OR 1=1` in parameters.
- The sqlmap user agent, or many rapid requests to one endpoint with small parameter changes.
- A spike in HTTP 500 errors (database errors leaking to users).
- Unusually slow responses on specific requests (time-based injection).
- Tier 1 triage: confirm whether the payload got a successful response, check the source IP, and escalate if data came back or the app returned database errors.

### Solution

1. **Parameterized queries / prepared statements.** This keeps code and data separate and is the most reliable fix.
2. Input validation with allowlists (a numeric ID should only be digits).
3. Least privilege database accounts (no DROP, no access to unrelated tables).
4. Use an ORM properly, so you don't build raw SQL strings.
5. A WAF as an extra layer. It helps, but it doesn't replace fixing the code.
6. Don't show raw database errors to users.

**MITRE ATT&CK:** T1190 (Exploit Public-Facing Application)

---

## 4. Cloud Computing Threats

Cloud computing delivers IT resources on demand as a metered service over a network. Clients often store sensitive data there, so a flaw in one client's cloud application can sometimes expose another client's data.

### How cloud threats work

- **Shared infrastructure (multi-tenancy):** many clients run on the same hardware and platform.
- **Shared responsibility:** the provider secures the platform, the client secures configuration, access, and data. Attacks happen in the gaps.
- **Internet exposure:** cloud resources are reachable over the network by design, so one mistake can be visible to everyone.

### How attackers perform cloud attacks

1. **Misconfiguration:** public storage buckets, open databases, security groups open to the whole internet.
2. **Weak access control:** stolen credentials, access keys committed to public GitHub repos (bots scan for these within minutes), accounts with no MFA.
3. **Insecure APIs:** management APIs with weak authentication or no rate limiting.
4. **Cross-tenant attacks:** abusing a flaw in a shared component to move from one client's environment to another.
5. **SSRF to the metadata service** (see note 1).

### The risks

- Data breaches of customer data.
- Account hijacking and abuse of resources (for example, crypto mining on the victim's bill).
- Loss of data or service availability.
- Compliance violations.

### Real cases

- Many breaches have come from public S3 buckets, including a 2017 exposure of Verizon customer data held by a partner company.
- Leaked cloud keys in public repos are a repeat cause of crypto mining abuse.

### Detection (SOC view)

- **Log sources:** AWS CloudTrail, Azure Activity Log, Google Cloud Audit Logs, plus alerts from GuardDuty or similar.
- Console login from a new country or an impossible travel pattern.
- An access key used from a new IP, or used right after being posted publicly.
- Security group or bucket policy changed to allow `0.0.0.0/0` or public access.
- Sudden creation of many compute instances (mining) or large data downloads.
- Root account usage, or MFA being disabled.

### Solution

1. Enforce MFA, especially on admin and root accounts.
2. Least privilege IAM roles. No long-lived keys where roles will do.
3. Block public access on storage by default, and scan for it with a CSPM tool.
4. Never commit keys to repos. Use secret scanning and rotate any key that leaks.
5. Turn on logging and alerts (CloudTrail, GuardDuty) in every region.
6. Encrypt data at rest and in transit.

**MITRE ATT&CK:** T1078.004 (Valid Accounts: Cloud Accounts), T1530 (Data from Cloud Storage)

---

## 5. Advanced Persistent Threat (APT)

An APT is a long-term, targeted attack that steals information from the victim's systems without being noticed. APTs mostly target large companies and government networks, and are often run by well-funded groups.

### How APTs work

- **Slow and quiet:** activity is kept low so the effect on performance and network traffic is hard to notice.
- **Vulnerability driven:** they exploit weaknesses in applications, operating systems, and embedded systems.
- **Goal:** stay inside as long as possible and keep collecting data.

### How attackers perform APTs

1. **Initial access:** spear phishing, exploiting a public-facing app, or compromising a supplier.
2. **Foothold:** install a backdoor or remote access tool so they can come back any time.
3. **Privilege escalation and lateral movement:** move from machine to machine toward the most valuable systems.
4. **Data collection and exfiltration:** gather data and send it out in small amounts to avoid alerts.
5. **Persistence:** hide, clear logs, and set up several ways back in.

### The risks

- Long-term espionage and theft of intellectual property.
- Theft of government or military information.
- Damage found months or years after the first breach.
- Very expensive investigation and recovery.

### Real case

SolarWinds, 2020. Attackers slipped malicious code into a software update for the Orion product, which then reached thousands of customers. It is a textbook supply chain attack.

### Detection (SOC view)

- Beaconing: a host connecting to the same external IP or domain at regular intervals.
- Long-lived outbound connections, or small but steady data transfers to unusual destinations.
- DNS tunneling signs: very long or random-looking subdomains, high DNS volume from one host.
- Off-hours logins and logins from unusual locations.
- Lateral movement: RDP, SMB, or PsExec between workstations that don't normally talk.
- New scheduled tasks, services, or accounts created without a change ticket.
- Tier 1 role: spot and document the odd behavior, then escalate. APT investigation is usually Tier 2 and 3 work.

### Solution

1. Network segmentation, so one foothold doesn't reach everything.
2. MFA and least privilege, with tight control over admin accounts.
3. EDR on endpoints and centralized logging in a SIEM.
4. Threat hunting, using threat intelligence and ATT&CK.
5. Patch management and supplier security reviews.
6. User training against spear phishing.

**MITRE ATT&CK:** T1566 (Phishing), T1071 (Application Layer Protocol), T1041 (Exfiltration Over C2 Channel)

---

## 6. Viruses and Worms

Viruses and worms are among the most common network threats and can infect a network within seconds.

- **Virus:** a self-replicating program that copies itself by attaching to another program, a boot sector, or a document.
- **Worm:** a malicious program that replicates and spreads across network connections on its own.

| | Virus | Worm |
|---|---|---|
| Needs a host file | Yes | No |
| Needs user action | Usually (opening the infected file) | No, spreads by itself |
| Spreads through | Shared files, removable media | Network connections |

### How they enter a system

- **Viruses:** the victim opens a malicious file shared over the internet or on removable media like a USB drive.
- **Worms:** the main route is exploiting a vulnerability in a network service, like SMB. Some also start from a spam email or a malicious download, but the fast spreading comes from the exploit.

### The risks

- Files corrupted or deleted.
- Network slowed or taken down by the traffic worms create.
- A foothold for more malware.
- Spread to many machines in seconds.

### Real cases

- ILOVEYOU, 2000: spread by email and overwrote files.
- Conficker, 2008: a worm that infected millions of Windows machines.
- WannaCry, 2017: a worm using the EternalBlue SMB exploit, carrying ransomware.

### Detection (SOC view)

- One host scanning many others, especially on port 445 (SMB).
- The same unknown file hash showing up on many machines.
- A sudden spike in internal network traffic.
- Antivirus or EDR alerts repeating across several hosts.
- Tier 1 response: isolate the infected host, identify the file or exploit, check for other hosts with the same indicator, escalate.

### Solution

1. Patch quickly, especially network-facing services (WannaCry hit unpatched systems).
2. Disable SMBv1 and block port 445 at the network edge.
3. Keep antivirus and EDR up to date.
4. Segment the network so a worm can't cross freely.
5. Email filtering and USB controls.

**MITRE ATT&CK:** T1210 (Exploitation of Remote Services), T1091 (Replication Through Removable Media)

---

## 7. Ransomware

Ransomware is malware that restricts access to a computer's files and folders and demands an online ransom payment to the malware creator to remove the restrictions.

### How ransomware spreads

- Malicious email attachments
- Infected software applications
- Infected disks
- Compromised websites
- Exposed remote access (like RDP) and unpatched systems

### How attackers perform ransomware attacks

1. **Delivery:** the victim opens an attachment or runs an infected program.
2. **Execution:** the malware runs and starts looking for files and network shares.
3. **Encryption or lockout:** files are encrypted (crypto ransomware) or the whole system is locked (locker ransomware). Backups and shadow copies are often deleted first.
4. **Ransom note:** a message demands payment, usually in cryptocurrency, for the decryption key.
5. **Double extortion:** data is stolen before encryption, and the attacker threatens to leak it if the ransom isn't paid.

Many attacks now run as ransomware-as-a-service. Developers rent the malware to affiliates who carry out the attack and share the profit.

### The risks

- Loss of access to critical files and systems.
- Business downtime.
- Financial loss, with no guarantee that paying restores anything.
- Public leak of stolen data.

### Real cases

- WannaCry, 2017: hit hundreds of thousands of machines, including hospitals.
- Colonial Pipeline, 2021: a DarkSide attack that shut down a major US fuel pipeline for days.

### Detection (SOC view)

- Many files renamed or given a new extension in a short time.
- Ransom note files (README or similar) appearing in many folders.
- `vssadmin delete shadows` or other commands that delete backups and shadow copies.
- Heavy disk activity from one process, or EDR alerts for encryption behavior.
- Before encryption: unusual logins, lateral movement, and large outbound data transfers.
- Tier 1 response: isolate the host immediately and escalate. Don't wait.

### Solution

1. Offline or immutable backups, and test restores.
2. Patch systems and close exposed RDP.
3. Email filtering and user training.
4. EDR with ransomware behavior detection.
5. Segmentation and least privilege, to limit how far it spreads.
6. A tested incident response plan.

**MITRE ATT&CK:** T1486 (Data Encrypted for Impact), T1490 (Inhibit System Recovery)

---

## 8. Mobile Threats

Attackers increasingly target mobile devices because phones are used for both work and personal tasks and often have fewer security controls than computers.

### How mobile threats work

- **Malicious apps:** malware (APKs) installed on the phone can damage other apps and data and send sensitive information to attackers.
- **Spyware and remote access:** malware can access the camera, microphone, and messages to watch the user. Advanced spyware like Pegasus can infect phones through zero-click exploits, with no user action needed.

### How attackers perform mobile attacks

1. **Sideloading APKs:** apps installed from outside the official store, often fake or modified versions of popular apps. Malicious apps have also slipped into the official stores.
2. **Smishing:** malicious links sent by SMS or messaging apps.
3. **Excessive permissions:** an app asks for camera, microphone, contacts, or SMS access it doesn't need.
4. **Unsafe networks:** public Wi-Fi used to intercept traffic.
5. **Outdated software:** old OS versions with known vulnerabilities.
6. **SIM swapping:** the attacker convinces the carrier to move the victim's number to a new SIM, then takes over accounts that use SMS codes.
7. **Rooted or jailbroken devices:** security protections are removed, so malware has more access.

### The risks

- Theft of personal and corporate data.
- Spying through the camera and microphone.
- Stolen banking and login details.
- The phone used as a way into the company network (BYOD).

### Real cases

- Pegasus (NSO Group): spyware used against journalists and activists.
- Joker malware: apps on the official Google Play store that signed users up for paid services.

### Detection (SOC view)

- MDM alerts for jailbroken or rooted devices, or non-compliant OS versions.
- A device suddenly installing apps from unknown sources.
- Impossible travel logins on accounts accessed from a phone.
- Unexpected SMS-based MFA codes or a sudden loss of mobile signal (possible SIM swap).
- Phones connecting to known malicious domains in DNS or proxy logs.

### Solution

1. Use MDM to enforce updates, encryption, and app rules.
2. Install apps only from official stores and review permissions.
3. Avoid public Wi-Fi or use a VPN.
4. Use app-based or hardware MFA instead of SMS where possible.
5. Add a carrier PIN to prevent SIM swaps.
6. Train users on smishing.

**MITRE ATT&CK:** See ATT&CK for Mobile (for example, malicious apps and phishing techniques for Android and iOS).

---

## 9. Botnet

A botnet is a large network of compromised systems controlled by an attacker. It is used for things like denial-of-service attacks, spam, and data theft.

### How botnets work

- **Bots:** infected devices that follow the attacker's commands, often without the owner knowing.
- **C2 server:** a command and control server sends instructions to all the bots at once.
- **Detection gap:** antivirus alone often misses bots, so EDR and network monitoring are needed to catch them.

### What bots are used for

- **DDoS attacks:** thousands of devices flood a target with traffic until it goes offline. Main types are volumetric (raw bandwidth), protocol (like SYN floods), and application layer (like HTTP floods).
- **Spreading malware:** pushing more malware to other systems.
- **Sending email:** spam and phishing, sometimes with botnet malware attached.
- **Stealing data:** collecting credentials and other info from infected machines.

### The risks

- Website and service outages.
- The victim's own device used to attack others.
- Stolen data.
- Blacklisted IP addresses and email domains.

### Real cases

- Mirai, 2016: infected IoT devices with default passwords and took down large parts of the internet through the Dyn DNS attack.
- Emotet: a botnet that delivered other malware, including ransomware.

### Detection (SOC view)

- Beaconing to the same IP or domain at regular intervals.
- DNS lookups for random-looking domains (domain generation algorithms).
- A workstation sending SMTP traffic when it shouldn't.
- Spikes in outbound traffic, or traffic to IPs flagged in threat intel feeds as C2.
- Tier 1 response: isolate the host, check the destination against threat intel, look for other hosts talking to it, escalate.

### Solution

1. EDR and network monitoring, with threat intel feeds on C2 IPs and domains.
2. DNS filtering and egress filtering.
3. Patch systems and change default credentials on devices.
4. Rate limiting and DDoS protection services for your own servers.
5. Email filtering against the malware that recruits bots.

**MITRE ATT&CK:** T1071 (Application Layer Protocol), T1498 (Network Denial of Service)

---

## 10. Insider Threat

An insider threat is an attack by someone inside the organization who has authorized access to the network and knows how it's set up.

### Types of insiders

- **Malicious insider:** deliberately steals data or causes damage (a disgruntled employee, for example).
- **Negligent insider:** causes harm by mistake, like clicking a phishing link or emailing data to the wrong person.
- **Compromised insider:** an outside attacker has taken over a real user's account. This is really account takeover, but it looks like an insider in the logs.

### How insiders perform attacks

1. **Abusing legitimate access:** they already have credentials, so they don't need to break in.
2. **Using network knowledge:** they know where valuable data lives and which controls are weak.
3. **Data theft:** copying files to USB drives, personal email, or cloud storage.
4. **Sabotage:** deleting data or changing system settings.

### The risks

- Data theft and leaks.
- Hard to detect, because the activity looks like normal work.
- Sabotage of systems.
- Fraud.

### Real case

Edward Snowden, 2013. A contractor with legitimate access copied and leaked a large amount of classified data.

### Detection (SOC view)

- DLP alerts for sensitive files going to personal email, USB, or cloud storage.
- Large or unusual downloads, especially right before a resignation.
- Access to data outside the person's job role.
- Off-hours access, or access from unusual locations.
- Use of removable media on machines where it's normally blocked.
- Activity after notice of termination or after access should have been removed.
- Tier 1 role: document and escalate. Insider cases often involve HR and legal, so don't confront the user.

### Solution

1. Least privilege and separation of duties.
2. DLP and activity monitoring on sensitive data.
3. Proper offboarding, with access removed on the last day.
4. Regular access reviews.
5. Security awareness training and a culture where mistakes get reported.

**MITRE ATT&CK:** T1567 (Exfiltration Over Web Service), T1052.001 (Exfiltration over USB)

---

## 11. Phishing

Phishing is sending a fake email that claims to be from a legitimate source, in order to get someone's personal or account information.

### How phishing works

- **Lure:** the email looks like it's from a trusted source, or contains a link to a site that looks like the real one.
- **Delivery channel:** links are sent by email or other channels.
- **Goal:** collect private information like account numbers, card numbers, and passwords, or get the user to run malware.

### Types of phishing

1. **Spear phishing:** aimed at a specific person or group, using personal details.
2. **Whaling:** aimed at senior executives.
3. **Vishing:** done over phone calls.
4. **Smishing:** done over SMS.
5. **Clone phishing:** a real email is copied and resent with a malicious link or attachment.
6. **Business email compromise (BEC):** the attacker poses as an executive or vendor to get a payment or data, often with no link or attachment at all.
7. **QR phishing ("quishing"):** a QR code in an email or poster that leads to a fake site.
8. **MFA-bypass kits:** fake login pages that sit in the middle and steal the session cookie after the user enters the MFA code.

### Common warning signs

- Urgent or threatening language.
- Sender address that doesn't match the real domain.
- Links that go somewhere different from the displayed text.
- Unexpected attachments.
- Requests for passwords or payment details.

### The risks

- Stolen credentials and account takeover.
- Financial fraud.
- Malware infection through attachments or links.
- Often the first step of a bigger attack (ransomware, APT).

### Real case

Twitter, 2020. Attackers phoned employees pretending to be IT support, got credentials, and took over high-profile accounts.

### How to analyze a suspicious email (SOC view)

1. **Check the headers.** Look at the From, Return-Path, and Reply-To addresses and the originating IP. Check the results for SPF, DKIM, and DMARC. A fail on these is a red flag, though a pass doesn't prove the email is safe.
2. **Check links safely.** Don't click. Look the URL up in VirusTotal or URLScan, or open it in a sandbox. Watch for look-alike domains (`paypa1.com`).
3. **Check attachments safely.** Get the file hash and look it up on VirusTotal, or detonate it in a sandbox.
4. **Check who else got it.** Search the mail gateway for the same sender, subject, or attachment hash.
5. **Respond.** Delete or quarantine it from every mailbox, block the sender and domain, block malicious URLs and IPs at the proxy or firewall.
6. **If someone clicked or opened it:** isolate the machine, reset their password, revoke their sessions, and check the sign-in logs for suspicious activity. Escalate if credentials were entered or a file ran.
7. **Document** everything in the ticket (indicators, actions, who was affected).

### Solution

1. Email filtering, plus SPF, DKIM, and DMARC set up correctly on your own domain.
2. MFA, ideally phishing-resistant (security keys or passkeys).
3. User training and a simple "report phishing" button.
4. Regular phishing simulations.
5. Verify payment and data requests through a second channel (a phone call to a known number).

**MITRE ATT&CK:** T1566 (Phishing), including T1566.001 (Spearphishing Attachment) and T1566.002 (Spearphishing Link)

---

## 12. Web Application Threats

Web apps are a favorite target because attackers can use them to steal credentials, set up phishing sites, or get private information. Many of these attacks come from flawed code and poor handling of input and output data. The OWASP Top 10 is the standard list of the most common ones. SQL injection and IDOR have their own notes (2 and 3), so this note covers the rest.

### How web application threats work

- **User input:** forms, URL parameters, cookies, and headers all take data from the user.
- **Missing validation:** the app doesn't check or clean that input properly.
- **Result:** the app treats attacker input as part of a command or page.

### How attackers perform web application attacks

1. **Cross-Site Scripting (XSS):** a malicious script is injected into a page and runs in other users' browsers, often to steal session cookies. Three types:
   - **Reflected:** the script is in a link, and runs when the victim clicks it.
   - **Stored:** the script is saved on the server (in a comment, for example) and runs for everyone who views it.
   - **DOM-based:** the flaw is in the page's own JavaScript, which handles input unsafely in the browser.
2. **Cross-Site Request Forgery (CSRF):** a logged-in user is tricked into sending an unwanted request (like changing their email) because the browser attaches their session cookie automatically.
3. **Broken authentication:** weak passwords, no lockout, credential stuffing (trying leaked passwords on other sites), or poor session handling.
4. **SQL injection** and **IDOR:** see notes 2 and 3.

### The risks

- Stolen credentials and private data.
- Session hijacking and account takeover.
- Phishing pages hosted on a compromised site.
- Defacement or downtime.

### Real case

The Samy worm, 2005. A stored XSS flaw on MySpace let a script spread to over a million profiles in about a day.

### Detection (SOC view)

- `<script>`, `onerror=`, `javascript:` or encoded versions in request parameters (XSS attempts).
- Many failed logins from one IP, or many IPs trying the same account (brute force and credential stuffing).
- Successful logins right after a long run of failures.
- State-changing requests with a missing or odd Referer or Origin header (CSRF).
- Log sources: web server, WAF, and authentication logs.

### Solution

1. **XSS:** encode output, use a Content Security Policy (CSP), and set the `HttpOnly` flag on session cookies.
2. **CSRF:** anti-CSRF tokens and `SameSite` cookies.
3. **Authentication:** MFA, account lockout or rate limiting, and strong session handling.
4. Validate input on the server side, not just in the browser.
5. Keep frameworks and libraries updated, and test with the OWASP Top 10 as a checklist.

**MITRE ATT&CK:** T1190 (Exploit Public-Facing Application), T1110 (Brute Force)

---

## 13. IoT Threats

IoT devices connected to the internet often have little or no security, so they are open to many types of attack.

### How IoT threats work

- **Remote access software:** IoT devices run software that lets people manage them remotely.
- **Hardware limits:** because of limited memory and battery, that software usually lacks strong security.
- **Result:** attackers can reach the device remotely and use it for further attacks.

### How attackers perform IoT attacks

1. **Default credentials:** logging in with factory logins like `admin/admin`.
2. **Unpatched firmware:** using known vulnerabilities that were never fixed.
3. **Open services:** Telnet, SSH, or web interfaces exposed to the internet. Attackers find them with search engines like Shodan.
4. **Botnet recruitment:** infecting devices and adding them to a botnet (see note 9).

### The risks

- Cameras and routers taken over and used for spying.
- Devices added to botnets for DDoS attacks.
- A way into the rest of the network.
- Privacy loss.

### Real case

Mirai, 2016. It scanned the internet for IoT devices (cameras, routers) still using default passwords, infected them, and used them for record-size DDoS attacks.

### Detection (SOC view)

- A camera, printer, or router suddenly talking to new external IPs.
- Scanning or login attempts on ports 23 and 2323 (Telnet), the ones Mirai-style malware favors.
- Many failed Telnet or SSH logins from or to an IoT device.
- A device sending far more traffic than normal.
- Devices showing up on the network that aren't in the asset inventory.

### Solution

1. Change default credentials on every device.
2. Keep firmware updated, and replace devices that no longer get updates.
3. Put IoT devices on a separate VLAN so they can't reach important systems.
4. Disable services you don't use (Telnet, UPnP) and don't expose devices to the internet.
5. Keep an asset inventory and monitor IoT traffic.

**MITRE ATT&CK:** T1110 (Brute Force), T1498 (Network Denial of Service). See also ATT&CK for ICS if you work with industrial devices.
