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

eg. Access to patients reports/pdf via url means. 
https://sc1.sukraa.in/LeoLab/pdffiles/_xxxxx.pdf_

Here, the last part (italicized), poses the vulnerability. 
This specific URL uses the combination of sample ID + Date. This makes it easier for anyone to access other patient records using random IDs and dates. 
for eg. 12345677-2026-01-01.pdf or 26536378-2026-11-19.pdf (each would be a pdf record of a patient)

Solution: 1. Password protected pdf (unique identifier can be used to open the pdf file that's only accessible to the patient/family)
          2. UUID (random unique ID) It's when a developer opens the pdf -> open developer tools -> typer in crypto.randomUUID () -> enter (gives you the ID) -> Save this ID to the database -> Rename the pdf with this random UUID. 

