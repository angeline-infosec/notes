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
