## CyberAttacks Notes:


## 1. SSRF

Server-Side Request Forgery (SSRF) is a security vulnerability that tricks a web server into making unauthorized requests to internal or external systems on behalf of an attacker

### **How SSRF Works**

- **User Input**: An app takes a user-supplied URL or data field to fetch a remote resource (like an image or PDF).
- **Missing Validation**: The server fails to validate or restrict where it sends the request.
- **Trusted Position**: The server uses its trusted internal network position to access restricted data (such as localhost, private IPs, or cloud metadata)

### **How Attackers Perform SSRF**

Attackers trick the server by replacing a legitimate external web address with a malicious or internal address.

1. **Input Manipulation**
   
* **Normal Behavior**: A web application asks for a URL to import a profile picture: https://example.com.Attacker
* **Manipulation**: The attacker changes the URL to target an internal network address: https://example.com.
* **The Result**: The server executes the request, thinking it is safe, and accidentally returns private internal data to the attacker.

2. **Alternative Protocol URL Schemes**
   
Attackers often switch from standard http:// or https:// protocols to standard system protocols to access files directly from the server's hard drive:

* **File Access**: file:///etc/passwd (to read Linux user/configuration files).
* **Data Access**: gopher:// or dict:// (to send raw data bytes to internal databases like Redis or Memcached).

3. **Bypassing Defenses**
   
If a developer implements weak security filters (like blocking the word "localhost"), attackers use tricks to bypass them:

* **IP Encoding**: Converting 127.0.0.1 to alternative formats like Octal (0177.0.0.0.1), Hexadecimal (0x7f000001), or a decimal integer (2130706433).
* **DNS Rebinding**: Setting up a custom domain name that initially points to a safe public IP (to pass the application's check) but instantly changes to a private internal IP right when the server makes the actual request.

 ### **The Risks**
 
 What Damaging Actions Do They Take?Once an attacker successfully forces the server to make requests on their behalf, they focus on exploiting the server's trusted position inside the network.
 
 * **Stealing Cloud Credentials**: If the application is hosted in a cloud environment (like AWS, Google Cloud, or Azure), attackers target the internal Cloud Metadata Service. By requesting URLs like http://169.254.169, they can steal temporary administrative access keys and compromise the entire cloud infrastructure.
 * **Internal Network Mapping (Port Scanning)**: The attacker uses the server as a proxy to test every internal IP address and port (e.g., trying to connect to port 22 for SSH or port 3306 for MySQL). This lets them map out the company's hidden internal network topology.
 * **Accessing Internal APIs and Admin Panels**: Many internal tools (like administrative dashboards or database management consoles) lack strong authentication because developers assume they are safe behind the corporate firewall. An SSRF attack allows the attacker to interact with these consoles with full admin privileges.
 * **Remote Code Execution (RCE)**: In worst-case scenarios, attackers use SSRF to send malicious commands to unauthenticated internal services (like Redis or Webmin), allowing them to take total control over the internal network servers.

## 2. IDOR 

Insecure Direct Object Reference

IDOR is a security vulnerability that occurs when an application exposes a direct reference to an internal object (a file, record, or database key) and fails to verify that the requesting user is actually authorized to access that specific object.

How IDOR Works
User Input: An app uses an identifier (ID, filename, or reference) supplied in a URL, form field, or API parameter to fetch a specific resource.
Missing Validation: The server retrieves the resource based on the identifier alone, without checking whether the current user has permission to view or modify that particular object.
Trusted Position: The application assumes that if a user can reach the endpoint, they're allowed to access whatever object the identifier points to — the identifier's predictability becomes the flaw.

How Attackers Perform IDOR
Attackers manipulate a reference in a request to access data or perform actions belonging to another user.

Input Manipulation
Normal Behavior: A patient views their own lab report at:
https://sc1.example.com/pdffiles/12345677-2026-01-01.pdf

Manipulation: The attacker simply changes the ID/date in the URL:
https://sc1.example.com/pdffiles/26536378-2026-11-19.pdf

The Result: If the server doesn't verify ownership, it returns another patient's report — because the reference was predictable (sequential ID, sample ID + date combo) and access wasn't tied to the logged-in user.

Common IDOR Patterns
Sequential/Predictable IDs: Numeric IDs (invoice=1001, invoice=1002) that can simply be incremented to enumerate other users' records.
Parameter Tampering: Changing a hidden form field or API body parameter (e.g. "user_id": 45 → "user_id": 46) rather than the URL itself.
Horizontal Privilege Escalation: Accessing another user's data at the same privilege level (another customer's order, another patient's report).
Vertical Privilege Escalation: Manipulating a reference to reach admin-only or higher-privilege resources.
API-Based IDOR: REST/GraphQL endpoints (e.g. /api/users/123/profile) exposing object IDs directly, common in mobile app backends where the frontend doesn't hide requests.

Bypassing Defenses
If a developer adds partial checks (like verifying the user is logged in, but not that they own the object), attackers can still succeed:
Session Reuse: Using a valid, low-privilege session token while simply swapping the object ID — authentication passes, but authorization is never actually checked.
Method Switching: Trying the same endpoint with different HTTP methods (GET vs POST vs DELETE) in case the authorization check was only implemented for one of them.
Response Comparison: Sending requests for both an owned and a non-owned object ID, and comparing responses to confirm the missing access control.

The Risks
Once an attacker can freely swap references, they exploit the gap between "authenticated" and "authorized."

Data Theft/Privacy Breach: Viewing other users' personal, financial, or medical records — as in the lab report example.
Data Tampering: Editing, deleting, or overwriting another user's data (e.g. changing someone else's order or profile).
Account Takeover: Modifying account details (email, password reset tokens) belonging to another user.
Mass Data Exposure: Automating ID enumeration (scripting through a range of IDs) to scrape large volumes of records at once.

Solution:
1. Server-Side Authorization Checks — always verify the requesting user owns or has explicit permission for the specific object, on every request, not just that they're logged in.
2. Unpredictable Identifiers (UUIDs) — replace sequential or guessable IDs with random UUIDs (e.g. via crypto.randomUUID()) so references can't be enumerated.
3. Indirect Reference Maps — map exposed IDs to internal database keys per-session, so the ID shown to the user never directly maps to the real record.
4. Access Control Testing — explicitly test each endpoint by trying to access another user's object ID during development/QA, not just the happy path.
5. Signed/Password-Protected Resources — for files like PDFs, require a unique token or password tied to the intended recipient rather than relying on obscurity.

eg. Access to patients reports/pdf via url means. 
https://sc1.sukraa.in/LeoLab/pdffiles/_xxxxx.pdf_

Here, the last part (italicized), poses the vulnerability. 
This specific URL uses the combination of sample ID + Date. This makes it easier for anyone to access other patient records using random IDs and dates. 
for eg. 12345677-2026-01-01.pdf or 26536378-2026-11-19.pdf (each would be a pdf record of a patient)

Solution: 1. Password protected pdf (unique identifier can be used to open the pdf file that's only accessible to the patient/family)
          2. UUID (random unique ID) It's when a developer opens the pdf -> open developer tools -> typer in crypto.randomUUID () -> enter (gives you the ID) -> Save this ID to the database -> Rename the pdf with this random UUID. 

## 3. SQL Injection (SQLi)

SQL Injection is a security vulnerability that lets an attacker interfere with the queries an application makes to its database by injecting malicious SQL code through user input fields.

How SQL Injection Works
User Input: An app takes user-supplied data (login form, search box, URL parameter) and inserts it directly into a SQL query string.
Missing Validation: The server fails to sanitize or parameterize this input before running the query.
Trusted Position: The database executes whatever query it receives, without knowing part of it came from an untrusted user.

How Attackers Perform SQL Injection
Attackers craft input that changes the structure or meaning of the intended SQL query.

Input Manipulation
Normal Behavior: A login form builds a query like:
SELECT * FROM users WHERE username = 'angie' AND password = 'pass123';

Manipulation: The attacker enters ' OR '1'='1 as the username, turning the query into:
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = '';

The Result: Since '1'='1' is always true, the query returns a valid user row, letting the attacker log in without a real password.

Common SQLi Techniques
Union-Based: Uses the UNION SQL operator to combine the results of the injected query with the original one, pulling data from other tables (e.g. ' UNION SELECT username, password FROM admin--).
Error-Based: Deliberately triggers database error messages that leak information about the database structure (table names, column names).
Blind SQLi (Boolean-based): No error or data is shown directly; the attacker infers information by asking true/false questions and observing subtle differences in the app's response (e.g. ' AND 1=1-- vs ' AND 1=2--).
Blind SQLi (Time-based): Similar to boolean-based, but the attacker uses functions like SLEEP(5) to measure response delays instead of visible differences, confirming injection when the response is delayed.
Out-of-Band: Relies on the database making external network requests (DNS/HTTP) to exfiltrate data, used when direct responses aren't visible to the attacker.

Bypassing Defenses
If a developer implements weak filters (like blocking the word "OR" or a single quote), attackers use tricks to bypass them:
Comment Syntax: Using -- , #, or /* */ to comment out the rest of a query and neutralize trailing conditions.
Encoding: URL-encoding or using hex/char encoding to sneak characters like ' past naive filters.
Case Variation / Whitespace Tricks: Mixing case (SeLeCT) or inserting comments between keywords to evade regex-based filters.

The Risks
What Damaging Actions Do They Take? Once an attacker can inject arbitrary SQL, they exploit the database's trust in the application layer.

Data Theft: Extracting sensitive data such as usernames, password hashes, credit card numbers, or personal records directly from the database.
Authentication Bypass: Logging into accounts (including admin accounts) without valid credentials, as shown in the example above.
Data Tampering: Modifying or deleting records — changing prices, granting themselves privileges, or wiping audit logs.
Privilege Escalation: In some database configurations, injected queries can be used to read/write files on the server's filesystem or interact with the OS, similar in impact to RCE.
Full Database Compromise: Combined techniques (e.g. UNION-based enumeration) can let an attacker dump entire databases, including schemas from other applications sharing the same DB server.

Solution:
1. Parameterized Queries / Prepared Statements — separate SQL code from data so user input is never interpreted as part of the query structure (the most reliable fix).
2. Input Validation & Allowlisting — restrict input to expected formats (e.g. numeric IDs should only contain digits).
3. Least Privilege DB Accounts — the application's DB user should only have the permissions it actually needs (no DROP, no access to unrelated tables).
4. ORM Usage — using an ORM (Object-Relational Mapper) correctly avoids raw string-concatenated SQL by default.
5. WAF (Web Application Firewall) — an additional layer to catch known injection patterns, though not a substitute for fixing the code itself.

One quick note since your preferences ask for direct feedback: this task (formatting existing knowledge into structured notes) is well within Haiku's range — you don't need Sonnet's reasoning for it, so it'd work fine and save cost.





# CyberAttacks Notes

## 2. Cloud Computing Threats

Cloud computing is an on-demand delivery of IT capabilities where infrastructure and applications are provided to subscribers as a metered service over a network. Clients often store sensitive information in the cloud. A flaw in one client's cloud application could let an attacker reach another client's data.

### How Cloud Threats Work

* **Shared Infrastructure:** Many clients run on the same underlying hardware and platform (multi-tenancy).
* **Shared Responsibility:** The provider secures the platform, but the client is responsible for configuration, access control, and data. Gaps between the two are where attacks happen.
* **Internet Exposure:** Cloud resources are reachable over the network by design, so a mistake is often visible to the whole internet.

### How Attackers Perform Cloud Attacks

1. **Misconfiguration:** Public storage buckets, open databases, and overly permissive security groups.
2. **Weak Access Control:** Stolen or leaked credentials, access keys committed to public GitHub repos, accounts without MFA.
3. **Insecure APIs:** Cloud management APIs with weak authentication or no rate limiting.
4. **Cross-Tenant Attacks:** Exploiting a flaw in a shared component to move from one client's environment into another's.

### The Risks

* Data breaches and exposure of sensitive customer data
* Account hijacking and abuse of the victim's cloud resources (for example, crypto mining)
* Loss of data or service availability
* Compliance violations

---

## 3. Advanced Persistent Threat (APT)

An Advanced Persistent Threat is a long-term, targeted attack that focuses on stealing information from the victim's machines without the user noticing. APTs mostly target large companies and government networks.

### How APTs Work

* **Slow and Quiet:** APT activity is slow by design, so the effect on computer performance and internet connections is almost unnoticeable.
* **Vulnerability Driven:** They exploit weaknesses in applications, operating systems, and embedded systems.
* **Goal:** Stay inside the network for as long as possible and keep collecting data.

### How Attackers Perform APTs

1. **Initial Access:** Spear phishing, exploiting a public-facing application, or compromising a supplier.
2. **Foothold:** Install a backdoor or remote access tool so they can return at any time.
3. **Privilege Escalation and Lateral Movement:** Move from machine to machine toward the most valuable systems.
4. **Data Collection and Exfiltration:** Gather data and send it out in small amounts to avoid detection.
5. **Persistence:** Hide, clear logs, and keep multiple ways back in.

### The Risks

* Long-term espionage and theft of intellectual property
* Theft of government or military information
* Damage that is found months or years after the initial breach
* Very expensive investigation and recovery

---

## 4. Viruses and Worms

Viruses and worms are the most common networking threats and can infect a network within seconds.

* **Virus:** A self-replicating program that makes a copy of itself by attaching to another program, a computer boot sector, or a document.
* **Worm:** A malicious program that replicates, executes, and spreads across network connections on its own.

| | Virus | Worm |
|---|---|---|
| Needs a host file | Yes | No |
| Needs user action | Usually (opening the infected file) | No, spreads by itself |
| Spreads through | Shared files, removable media | Network connections |

### How They Enter a System

* **Viruses:** The attacker shares a malicious file with the victim over the internet or through removable media (USB drives).
* **Worms:** The victim downloads a malicious file, opens a spam email, or browses a malicious website.

### The Risks

* Files corrupted or deleted
* Network slowed down or taken offline by the amount of traffic worms create
* A foothold for installing other malware
* Fast spread across many machines in seconds

---

## 5. Ransomware

Ransomware is a type of malware that restricts access to a computer system's files and folders and demands an online ransom payment to the malware creator to remove the restrictions.

### How Ransomware Spreads

* Malicious email attachments
* Infected software applications
* Infected disks
* Compromised websites

### How Attackers Perform Ransomware Attacks

1. **Delivery:** The victim opens an attachment or runs an infected program.
2. **Execution:** The malware runs and starts looking for files and network shares.
3. **Encryption or Lockout:** Files are encrypted (crypto ransomware) or the whole system is locked (locker ransomware).
4. **Ransom Note:** A message demands payment, usually in cryptocurrency, in exchange for the decryption key.
5. **Double Extortion (common now):** Data is stolen before encryption, and the attacker threatens to leak it if the ransom is not paid.

### The Risks

* Loss of access to critical files and systems
* Business downtime
* Financial loss, with no guarantee that paying will restore the files
* Public leak of stolen data

---

## 6. Mobile Threats

Attackers are increasingly targeting mobile devices because smartphones are widely used for both business and personal tasks and usually have fewer security controls than computers.

### How Mobile Threats Work

* **Malicious Apps:** Users download malware (APKs) onto their phones. These can damage other apps and data and send sensitive information to attackers.
* **Remote Access:** Attackers can access the phone's camera and recording apps to watch user activity and track voice communication, which can help them plan a further attack.

### How Attackers Perform Mobile Attacks

1. **Sideloading APKs:** Users install apps from outside the official store, often fake or modified versions of popular apps.
2. **Smishing:** Malicious links sent by SMS or messaging apps.
3. **Excessive Permissions:** An app asks for access to the camera, microphone, contacts, or SMS that it does not need.
4. **Unsafe Networks:** Public Wi-Fi used to intercept traffic.
5. **Outdated Software:** Old OS versions with known vulnerabilities.

### The Risks

* Theft of personal and corporate data
* Spying through the camera and microphone
* Stolen banking and login details
* A phone used as an entry point into the company network (BYOD)

---

## 7. Botnet

A botnet is a large network of compromised systems used by attackers to perform denial-of-service attacks.

### How Botnets Work

* **Bots:** Infected devices that follow the attacker's commands, often without the owner knowing.
* **C2 Server:** A command and control server sends instructions to every bot at once.
* **Detection Gap:** Antivirus programs may fail to find, or even scan for, spyware and botnets, so tools designed specifically to find and remove them are needed.

### What Bots Are Used For

* **DDoS Attacks:** Thousands of devices flood a target with traffic until it goes offline.
* **Uploading Viruses:** Spreading more malware to other systems.
* **Sending Emails:** Spam and phishing emails, sometimes with botnet malware attached.
* **Stealing Data:** Collecting credentials and other information from infected machines.

### The Risks

* Website and service outages
* The victim's own device used in attacks on others
* Stolen data
* Blacklisted IP addresses and email domains

---

## 8. Insider Threat

An insider threat is an attack launched by someone inside the organization who has authorized access to the network and knows the network architecture.

### Types of Insiders

* **Malicious Insider:** Deliberately steals data or causes damage (for example, a disgruntled employee).
* **Negligent Insider:** Causes harm by mistake, such as clicking a phishing link or sending data to the wrong person.
* **Compromised Insider:** An attacker has taken over a legitimate user's account.

### How Insiders Perform Attacks

1. **Abusing Legitimate Access:** They already have credentials, so they do not need to break in.
2. **Using Network Knowledge:** They know where valuable data is stored and which controls are weak.
3. **Data Theft:** Copying files to USB drives, personal email, or cloud storage.
4. **Sabotage:** Deleting data or changing system settings.

### The Risks

* Data theft and leaks
* Hard to detect because the activity looks like normal work
* Sabotage of systems
* Fraud

---

## 9. Phishing

Phishing is the practice of sending an illegitimate email that falsely claims to be from a legitimate site, in order to get a user's personal or account information.

### How Phishing Works

* **Lure:** The email is designed to look like it comes from a trusted source, or it contains a link to a website that looks like the real one.
* **Delivery Channel:** Malicious links are distributed by email or other communication channels.
* **Goal:** Collect private information such as account numbers, credit card numbers, and mobile numbers.

### Types of Phishing

1. **Spear Phishing:** Targeted at a specific person or group, using personal details.
2. **Whaling:** Targeted at senior executives.
3. **Vishing:** Done over phone calls.
4. **Smishing:** Done over SMS.
5. **Clone Phishing:** A real email is copied and resent with a malicious link or attachment.

### Common Warning Signs

* Urgent or threatening language
* Sender address that does not match the real domain
* Links that point to a different site than the displayed text
* Unexpected attachments
* Requests for passwords or payment details

### The Risks

* Stolen credentials and account takeover
* Financial fraud
* Malware infection through attachments or links
* Often the first step of a larger attack (ransomware, APT)

---

## 10. Web Application Threats

Web application attacks like SQL injection and cross-site scripting make web applications a favorite target for attackers who want to steal credentials, set up a phishing site, or get private information. Many of these attacks come from flawed coding and improper sanitization of input and output data.

### How Web Application Threats Work

* **User Input:** Forms, URL parameters, cookies, and headers all accept data from the user.
* **Missing Validation:** The application does not check or clean that input properly.
* **Result:** The application treats attacker input as part of a command or page.

### How Attackers Perform Web Application Attacks

1. **SQL Injection (SQLi):** SQL code is added to an input field so the database returns or changes data it should not (for example, `' OR 1=1 --`).
2. **Cross-Site Scripting (XSS):** A malicious script is injected into a page and runs in other users' browsers, often to steal session cookies.
3. **Broken Authentication:** Weak passwords, no lockout, or poor session handling.
4. **Cross-Site Request Forgery (CSRF):** A logged-in user is tricked into sending an unwanted request.

### The Risks

* Stolen credentials and private data
* Phishing pages hosted on a compromised site
* Website defacement or downtime
* Damage to the performance and security of the website

---

## 11. IoT Threats

IoT devices connected to the internet often have little or no security, which makes them vulnerable to many types of attacks.

### How IoT Threats Work

* **Remote Access Software:** IoT devices run many software applications that are used to access the device remotely.
* **Hardware Limits:** Because of limited memory and battery, these applications usually lack strong security mechanisms.
* **Result:** Attackers can reach the device remotely and use it to carry out further attacks.

### How Attackers Perform IoT Attacks

1. **Default Credentials:** Logging in with the factory username and password (for example, admin/admin).
2. **Unpatched Firmware:** Exploiting known vulnerabilities that were never fixed.
3. **Open Services:** Telnet, SSH, or web interfaces exposed to the internet.
4. **Botnet Recruitment:** Infecting devices and adding them to a botnet (the Mirai botnet is a well-known example).

### The Risks

* Devices such as cameras and routers taken over and used for spying
* Devices added to botnets for DDoS attacks
* A way into the rest of the network
* Privacy loss
