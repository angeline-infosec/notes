# CAPS Course Notes: Phishing Analysis

Notes from the Certified Anti-Phishing Specialist (CAPS) course from Cyber Ledger Academy, a free course of 23 lessons in six modules. The course teaches phishing analysis from static artifacts such as emails, headers, URLs, screenshots and code snippets, with no hands-on labs. Its stated aim is to build the habit of explaining why something is phishing, with evidence, instead of trusting a tool's verdict.

These notes are summarized in plain language from the course text. Statements that were checked against a source carry a bracketed number that points to the source list at the end, where each source is marked as primary or secondary. A few course statements turned out to be too strong, out of date, or inconsistent with the course's own diagrams, so section 8 lists what was checked and what changed. The course's lifecycle and kill chain diagrams in particular do not match its own text, so these notes follow the text and the standard references instead of the figures.

## 1. Why phishing works

Phishing is identity abuse. The attacker pretends to be someone the target already trusts, such as a colleague, a bank, a cloud provider or a delivery service, and the goal is believability rather than technical sophistication. A convincing phish matches how the organization communicates, when messages normally arrive, and what the target expects or fears, so nothing looks wrong at a glance. It is better understood as an initial access problem than an email problem, because one successful phish can lead to stolen credentials, a hijacked session, lateral movement and a much larger compromise.

The course groups the psychology into four levers. Authority works because people comply with titles, brands and official-sounding language, which is how CEO fraud and fake IT notices succeed. Urgency removes the time needed to verify anything. Fear threatens a loss, such as a suspended account or a legal notice, and keeps the wording vague so the victim imagines the worst. Curiosity is the quietest of the four: an incomplete message like "new voicemail" or "you were mentioned in a document" creates an information gap that a click closes. Real campaigns stack these levers (authority with urgency, fear with authority, curiosity with urgency) until clicking feels routine. The course's argument for why awareness training alone falls short is that these cues are processed emotionally and quickly, so knowing what to look for helps less once the pressure is on.

The lifecycle it describes has five stages: reconnaissance (learning roles, vendors and timing), delivery (an email, chat message or shared document that fits the workflow), interaction (a login, approval or file open), capture (credentials, session tokens or access grants) and abuse (lateral movement, data theft, fraud or persistence).

### Phishing mapped to the cyber kill chain

The course maps phishing onto the seven stages of the Lockheed Martin kill chain [6], and the useful part is how little malware appears in the middle of it.

| Stage | What it looks like in a phishing operation |
|---|---|
| Reconnaissance | Profiling roles, vendors and timing from LinkedIn, breach data, job postings, GitHub commits and even auto-replies. It happens entirely outside the target's network, so there is nothing to alert on. |
| Weaponization | The weapon is usually deception, not malware: cloned login pages, QR codes, HTML attachments, or an adversary-in-the-middle proxy that relays logins in real time. Kits are pre-built, cloud-hosted and updated to dodge detection. |
| Delivery | Email is still dominant, but SMS, Teams, Slack, WhatsApp, social DMs, calendar invites and shared documents all carry lures, often timed around payroll, audits or the end of the day. |
| Exploitation | The user clicks, scans, opens or signs in. No exploit code is needed. The outcome is captured credentials, a granted OAuth permission, a fatigued MFA approval or an intercepted session token. |
| Installation | Persistence without malware: mailbox forwarding rules, OAuth app consent, password resets on other services, stolen session tokens and reused passwords. |
| Command and control | The attacker uses webmail, cloud APIs and tokens, so activity looks like a normal user logging in and reading mail. This is why dwell times run long. |
| Actions on objectives | Business email compromise, invoice fraud, lateral phishing, data theft, or setting up the next attack. It is often discovered after the money or data is gone. |

In MITRE ATT&CK terms, phishing is the Initial Access technique T1566, with four sub-techniques (T1566.001 to T1566.004) [3]. The practical point of the mapping is that a phishing incident is a multi-stage failure. Email filtering touches only one stage and MFA by itself does not break the chain, so defenses need to cover several stages and watch identity behavior as well as malware.

### Myths that lower people's guard

The myths lesson reads as a list of reasons people feel safe when they should not. Smart or experienced users are not immune, because phishing targets workload, habit and time pressure rather than intelligence. Checking links is not enough when attacks use compromised legitimate domains, shorteners, look-alike characters, QR codes, attachments or OAuth consent screens, and some of those never involve a suspicious-looking URL at all. Having MFA does not settle the matter, and section 8 covers the nuance there. Phishing emails are no longer easy to spot, because they are clean, on-brand and well timed. Attachments that look like invoices, resumes or shared files are expected rather than random, and HTML files that open a fake portal locally are a common format. Under pressure, emotion tends to beat intuition. Treating phishing as a personal failing makes victims delay reporting, and one-off training does not replace continuous simulation, detection and a safe way to report.

## 2. Email structure, headers and triage

### How an email travels

A message starts in a mail client, is submitted to an authenticated SMTP server (ports 587 or 465), and then hops between mail transfer agents that are found through DNS MX records. Each server adds a Received header. The final server runs the SPF, DKIM and DMARC checks, assigns a spam or phishing score, and decides between the inbox, the spam folder, quarantine or a block. Because headers are added along the way rather than removed, each stage leaves traces that an analyst can read.

Headers are there to answer four questions: who really sent this, where did it actually come from, was it authenticated, and was it spoofed or tampered with. Most of the rest is noise, and the skill is finding the few lines that matter.

### Fields that matter

Received shows the path the message took. Each server adds its line above the previous one, so it is read from the bottom up and the lowest line is the earliest hop. Unexpected countries, residential addresses, cheap cloud VPS ranges, and malformed or missing hops deserve a closer look.

From is what the reader sees. Return-Path records the envelope sender used for bounces, so a trusted brand in From paired with an unrelated domain in Return-Path is the classic identity split. Authentication-Results summarizes the SPF, DKIM and DMARC outcomes written by the receiving server. Message-ID is generated by the sending system, and its domain often hints at the real sender. Reply-To shows where replies go, which matters for credential harvesting and for invoice-fraud follow-ups.

Two cautions apply when using these fields (details in section 8). Only the Received lines added by servers you control or trust can be relied on, since lines further down can be written by the sender. Private addresses in a Received line and a Return-Path that differs from From both appear in legitimate mail, for example from internal relays and bulk-mail services. They are clues to weigh against the authentication results, not verdicts.

### SPF, DKIM and DMARC

| Mechanism | What it checks | What it does not tell you |
|---|---|---|
| SPF | Whether the sending IP is authorized by the domain in the envelope sender (MAIL FROM). The course lists the results pass, fail, softfail, neutral and none. | Nothing about the visible From address, the content or the sender's intent. |
| DKIM | That the domain named in the signature's `d=` tag signed the message, and that the signed parts were not altered. | A valid signature can come from any domain, including one the attacker owns [1]. |
| DMARC | That the visible From domain lines up (aligns) with a domain that passed SPF or DKIM, plus the domain owner's requested handling: none, quarantine or reject [1]. | It is not a safety verdict, and it does not address look-alike domains or display names [1]. |

The idea that unlocks most header analysis is alignment. An attacker can send mail with a MAIL FROM on their own domain, pass SPF for it, and still put the CEO's address in the visible From line. SPF passes and the identity is still fake. DMARC closes that gap by requiring the authenticated domain to match the From domain. Under relaxed alignment the two only need the same organizational domain, so mail.example.com aligns with example.com, while strict alignment requires an exact match [1]. A message passes DMARC when SPF or DKIM passes and aligns [1]. Alignment is a DMARC concept, not part of SPF itself, even though the course teaches it under SPF [1].

Receivers decide what to do with a failure locally, and the current standard says they must not reject mail solely because a domain publishes `p=reject` [1]. A domain at `p=none` has asked receivers for no special handling, so spoofed mail claiming that domain is more likely to reach an inbox. The DMARC specification is now RFC 9989, published in May 2026, and it replaces RFC 7489 [1].

### Header manipulation tricks

The tricks the course lists mostly work by splitting identity across fields that people read separately. The visible From can be spoofed while Return-Path points elsewhere. The display name can carry a trusted brand while the address behind it is unrelated, which works because people read names faster than domains. Reply-To can quietly send replies to attacker infrastructure. A message can pass SPF but fail DMARC alignment, or come from a domain whose DMARC policy is `none`, and gateways often score that as only mildly suspicious. DKIM can be valid but signed by a domain the attacker controls. Received lines can be padded with fake hops, and X-headers such as X-Mailer or X-Originating-IP are untrusted metadata that anyone can add. Forwarders and compromised mailing lists can make authentication pass for a message that is still malicious, and send times or language can be matched to the target's business hours to blend in.

The defender's habit that follows is to stop trusting any single signal and correlate From, Return-Path, SPF, DKIM, DMARC and Received together, and to treat headers as forensic evidence rather than decoration.

## 3. Domains, URLs and infrastructure abuse

### Look-alike domains and homographs

Look-alike domains use characters that resemble the real brand: digits for letters (`micr0soft.com`, `goog1e.com`), a capital I for a lowercase l (`paypaI.com`), keyboard-adjacent typos (`amazom.com`, `linkdin.com`), brand-plus-keyword mashups (`office365-alerts.com`) and misleading prefixes. Subdomain tricks work by hiding the real domain at the end of a long name. In `paypal.com.login.secure-auth[.]site` the registrable domain is the rightmost part, `secure-auth.site`, and everything before it is just labels the attacker chose.

Homograph attacks go a step further and use Unicode characters from other alphabets that look identical to Latin letters, such as a Cyrillic "а" in place of a Latin "a". Internationalized domain names are stored in an ASCII form called punycode, which starts with `xn--`. A worked check confirms the course's main example: a label made of a Cyrillic "а" followed by "pple" encodes as `xn--pple-43d` [7]. The course's German example is also right, since `müller-büromöbel` encodes as `xn--mller-brombel-rmb4fg` [7].

Two details in the course need care. The course says a browser may show such a domain as plain `apple.com`. Chrome's published policy shows punycode instead when a label mixes Latin and Cyrillic, and also when a label is made entirely of Cyrillic look-alikes under a non-Cyrillic TLD [4]. The risk is real in other places, though, such as email clients, chat apps and mobile interfaces, and ordinary ASCII look-alikes like `paypaI.com` never trigger script-mixing checks at all. Chrome's separate lookalike warning only covers sites that resemble top or recently visited domains [4]. The second detail is the header-trick example `xn--microsft-hrb.com`, which decodes to a label beginning with the letter ŋ followed by "microsft" [7], so it does not impersonate Microsoft and should be treated as illustrative only.

Analysts detect these by pasting the domain into a plain text editor, comparing it with the known-good domain, looking for mixed scripts, decoding punycode, checking domain age and registrar, correlating passive DNS, and scoring edit distance against brand names.

### Typosquatting and brand impersonation

Typosquatting relies on muscle memory and skimming. Brand impersonation copies the logo, layout, email tone and support workflow so that the victim feels helped rather than attacked. The build is cheap: register the deceptive domain, clone the real login page, add a credential logger, get a free TLS certificate and launch the campaign. WHOIS data gives red flags such as a recent creation date, a privacy-protected registrant, a CDN in front of the name servers, and a brand name inside a domain the brand does not own. The padlock only means the connection to that domain is encrypted. It says nothing about whether the domain deserves trust.

### Shorteners, redirects and tracking

Modern link phishing is layered. A short link hides the destination, an open redirect on a trusted site launders it through a reputable domain, and a tracking service adds a per-victim ID before a geo or bot check decides what the visitor sees. The layers exist to delay detection, break signature-based blocking and defeat scanners that stop at the first hop. Warning signs include shortened links in login or payment emails, parameters such as `redirect=`, `url=`, `next=` and `continue=`, chains of three or more hops, long encoded query strings, and different users receiving different final pages. The analyst's answer is to expand every short link, follow the chain manually in a safe environment, inspect the HTML of the final landing page instead of judging by appearance, and correlate tracking IDs across incidents.

### Free hosting abuse

Free hosting gives attackers zero cost, no payment trail, instant HTTPS and a trusted parent domain. The course names static site hosts, app hosting platforms, free PHP hosts, blog platforms and cloud object storage as common choices. Patterns include brand login pages under paths like `/auth/` or `/verify/`, malicious intent buried in deep paths on a clean-looking domain, free hosts used as a middle hop in a redirect chain, and pages that need no backend at all because JavaScript sends the captured data to a chat bot, form service or database. Indicators to look for are forms that post to unrelated endpoints, Base64-encoded or obfuscated scripts, hard-coded bot tokens, no custom domain, a single-page structure and a recently created project. A free-hosted login page for a major brand should be treated as high risk by default.

### Domain age and reputation

Domain age is how long ago a domain was registered, and reputation is a behavioral score, so the two are different things. Attackers register domains early, park them with harmless content and activate them later, which the course calls seasoning. They also buy expired or auctioned domains with a clean history, and compromise legitimate sites to inherit their reputation. Email reputation is separate again, because it starts at the first message sent rather than at registration. The signals that matter are recent changes: new DNS records, a freshly issued certificate, a new MX record on a long-silent domain, a changed hosting provider, a login page appearing where there was none, and a redirect that ends somewhere unrelated. An old domain with a recently added MX record, a new certificate, a new login page and a brand impersonation is camouflage, not trust.

### Worked example: one fake login, two delivery routes

Two screenshots in the course show the same fake Microsoft "Enter password" page arriving in two different ways. One is hosted on cloud object storage under a long, voicemail-themed path, which fits the free-hosting pattern above. The other is opened from a local file path (a `file://` address), which means it arrived as an HTML attachment. In the first case the hosting domain is the evidence to inspect. In the second there is no domain to check, so the analysis moves to the file's contents and to where its form submits the data. The small wording differences between the two pages ("Forgot password?" versus "Forgot my password") suggest different kits or variants.

## 4. Phishing kits and attacker workflows

### Kit anatomy

A phishing kit is a ready-to-deploy bundle that lets a low-skill attacker run credential harvesting at scale. The course describes a typical kit as a front end (pixel-accurate login pages built from HTML, CSS and JavaScript, often with inline logos and locally copied fonts so there is nothing to hotlink-check), a back-end handler that receives the submitted fields, a configuration file holding the attacker's destination, folders of anti-bot and geo-filtering rules, server rules that control redirects, and sometimes an admin panel for viewing and exporting captured data. The flow is always the same: victim, landing page, handler, then storage or exfiltration, with the sophistication going into evasion rather than invention. Common evasions are blocking crawlers by user agent or IP range, limiting access to target countries, showing a fake "incorrect password" message so the victim types the password again, and redirecting to the real site afterwards so the login seems to have worked. Captured data typically goes to a text file, an email, a chat bot API or an external drop server. Campaigns are sent with mass-mailing tools, abused open relays or compromised mailboxes, and the course calls the admin-panel model "phishing as a service".

### Four ways credentials get harvested

The course reduces credential harvesting to four models, and every kit is a variation on one of them.

1. **A fake login endpoint.** The victim types into a look-alike page and the form submits to an attacker-controlled address. The artifacts are a form action pointing at an unknown domain, a POST to something that is not the real identity provider, and a redirect afterwards.
2. **A reverse proxy in the middle.** The victim talks to the fake domain, which relays everything to the real service, so the attacker captures the password and the session cookie after MFA completes. Cookie reuse from a new network or location is the telltale sign.
3. **Client-side capture.** Injected JavaScript reads the fields before or during submission. Obfuscated inline scripts, scripts from unknown hosts and network calls that fire before the form is submitted give it away.
4. **Token or session theft.** The attacker skips the password, either through a malicious OAuth app that the victim authorizes or by stealing tokens from a browser with an infostealer. New OAuth consents, excessive scopes and API calls with no matching login are the signals.

The defensive controls the course lists are phishing-resistant authentication such as FIDO2 or passkeys, strict OAuth approval workflows and DMARC enforcement for prevention, session replay analytics, OAuth app monitoring and form-destination analysis for detection, and, after an incident, revoking tokens and sessions rather than only resetting the password.

### Data exfiltration paths

This lesson covers what happens after access is gained, so it is broader than phishing. Attackers move data out through command-and-control channels (HTTPS most often, DNS tunneling when the network is poorly monitored), through trusted cloud services using stolen tokens, through email and silent auto-forward rules, through plain web POSTs with encoded payloads, through FTP or SSH transfers, through removable media, and through covert channels such as steganography or chat-platform bots. Before moving anything they usually compress, encrypt, chunk and throttle it, which is why watching volume alone misses them. The detection ideas are egress filtering with deny by default, DNS logging with entropy analysis, cloud audit logs, context-aware data loss prevention, and a baseline of normal outbound behavior.

### Session hijacking

A session is a temporary trust agreement between a client and a server, represented by a token such as a cookie, a JWT or an API key. Whoever holds the token is treated as the user, so stealing it bypasses the password and MFA entirely, because the authentication has already happened.

| Technique | Idea | Main defense |
|---|---|---|
| Cookie theft | Reusing a stolen server-issued session ID | `HttpOnly`, `Secure` and `SameSite` flags, TLS, rotating session IDs, short timeouts |
| Session fixation | The attacker supplies a known session ID, the victim logs in on it, and the attacker reuses it | Regenerate the session ID after login and never accept IDs from URLs |
| XSS token theft | Injected script reads the session material | HttpOnly cookies instead of `localStorage`, a strong content security policy |
| JWT replay | A stolen stateless token is reused until it expires | Short lifetimes, refresh token rotation, validating issuer, audience and expiry |
| API or OAuth token abuse | A leaked bearer token or refresh token acts as a trusted app | Scoped permissions, redirect URI allowlists, PKCE, rotation |
| Browser or endpoint theft | Malware takes the post-login state | EDR, extension allowlists, token invalidation on device risk |
| CSRF | The attacker's request rides on the victim's existing session | CSRF tokens, `SameSite` cookies, origin checks |

CSRF is listed in the course under session hijacking, but it works differently from the rest, since it uses the victim's browser instead of stolen token material. The session fixation diagram in the course shows three steps: the attacker sends a link carrying a fixed session ID, the user logs in through it, and the attacker then uses the same ID. Detection signals across all of these are the same token appearing from different networks or regions, token use with no login event, API tokens still active after a password reset, privileged actions from a session that was never freshly authenticated, and impossible travel without re-authentication.

## 5. Phishing simulations, awareness and campaign design

The course frames a simulation as a controlled attack on the human layer. Its purpose is to measure where the risk really is, turn awareness into habit, and build a reporting culture in which people act as sensors rather than victims. The measures are click rate, credential submission rate, report rate, time to report and repeat offenders, and the lesson's most useful idea is that the win is a fast report, not zero clicks.

Campaign design follows a sequence. Start with measurable objectives, because a campaign that cannot be measured is cosmetic. Segment users by risk exposure (finance for invoice fraud, HR for resume lures, IT for credential harvesting and OAuth, executives for whaling, and general staff for brand impersonation). Model the threat on real incidents, industry patterns and seasonal timing, such as payroll cycles. Scale difficulty through four levels, from obvious spoofed domains up to internal sender spoofing and kit-built, business-contextual lures. Build each lure from five parts: a trusted sender, a hook (urgency, fear or reward), a believable context, a single clear action and low friction. Send during business hours at a sensible frequency (monthly for mature programs, quarterly for early ones), because too often causes fatigue and too rarely gives false confidence. When someone fails, show a short, non-shaming explanation immediately, and when someone reports, acknowledge it. Include the SOC in the test so alerting, quarantine speed and triage are checked too, report results to leadership in risk terms, and repeat. The module title mentions ethics, but the lessons themselves cover purpose and design far more than ethical constraints.

## 6. Sample emails from the course

The course illustrates the trust levers with real-looking samples. Describing the tells in words is more useful than reusing the screenshots, which come from third-party sources.

| Sample | Tells | Levers |
|---|---|---|
| Credit union "account restricted" notice | Generic "Dear Account Holder" greeting, the typo "restriction statues", awkward wording about log-on details, a "Sign-On" link whose destination is not visible, and a 24-hour expiry | Authority, fear, urgency |
| "Please transfer funds to client" | Empty sender address (`<>`), a payment instruction with a tight deadline, then a request to open an Excel file | Authority, urgency. A business email compromise pattern that adds a payload |
| Microsoft password expiry notice | Generic "Mail System Notification" display name with a personal-looking address at an unrelated domain, a button labelled "Keep Same Password", and a Friday-evening send time | Authority, urgency |
| Mailbox suspension notice | Display name shows the recipient's own domain while the real address is `admin@serveradminhelpdesk.com`, a different Reply-To, the odd greeting "Dear Esteemed User's Of", and a "validate here" link with a permanent-suspension threat | Authority, fear, urgency. Display name deception plus Reply-To |
| State attorney complaint | A government office writing from a free webmail address, a "Dear Bar Member" greeting about a complaint against "your Business", a link styled as a PDF filename, a 10-day deadline, and long confidentiality boilerplate that exists only to look official | Authority, fear |

The two dated samples are from 2016 and 2023, even though the lesson is titled "Phishing in 2026". That is a reminder that these tells are old and still work.

## 7. A working order for triage

This is a synthesis of the modules rather than a course checklist.

1. Read the message cold: who claims to have sent it, what action it wants, and what pressure it applies.
2. Compare From, Return-Path and Reply-To, and check whether the display name matches the address behind it.
3. Read Authentication-Results, and check DMARC alignment against the visible From domain, not just whether SPF passed.
4. Walk the Received lines from the bottom up, trusting only the hops added by infrastructure you control or trust.
5. Take every URL apart: expand shorteners, decode punycode, read the domain from the right, and follow redirects in an isolated environment.
6. Check domain registration, recent DNS and certificate changes, hosting and any impersonated brand.
7. For attachments and landing pages, read the HTML and find where the credentials would go.
8. Write the verdict with a confidence level, the evidence behind it and the containment action, such as blocking the sender and domain, purging copies, and revoking sessions and tokens for anyone who interacted.

## 8. What was checked and what changed

The course text is uneven, so key statements were checked against primary sources where possible. Source numbers refer to the list below.

| Course statement | Finding | Source |
|---|---|---|
| Phishing is the most successful attack vector, and most breaches start with phishing | Not supported by the latest data. Reports on the 2026 Verizon DBIR say vulnerability exploitation is now the top initial access vector at 31%, with phishing at about 16%. The DBIR also notes that phishing, pretexting and credential abuse overlap. Phishing is a leading vector, not the leading one. One outlet reports 36% for exploitation, while most report 31%, so check the DBIR's own figure before quoting it. | [5] secondary (DBIR itself not read) |
| MFA protects credentials but not sessions | True for SMS, app-based and push MFA. CISA names FIDO/WebAuthn as the only widely available phishing-resistant option as of its October 2022 fact sheet and also counts PKI-based MFA. Number matching helps against push bombing but is an interim step. A blanket statement that MFA does not protect sessions overlooks phishing-resistant MFA. | [2] primary (read through extracts, not the full PDF) |
| Alignment is part of SPF, and "DMARC fail with aligned domain" is strong evidence | Alignment is defined by DMARC, and the phrase is internally contradictory. A DMARC pass needs SPF or DKIM to pass and align. SPF and DKIM passes can come from an attacker's own domain. DMARC does not cover look-alike domains or display-name abuse, and a pass is not a safety verdict. | [1] primary |
| DMARC is described by RFC 7489 | RFC 7489 was obsoleted by RFC 9989 in May 2026. | [1] primary |
| A Cyrillic look-alike shows as `apple.com` in the browser | Chrome shows punycode for labels that mix Latin and Cyrillic and for whole-script look-alikes under a non-matching TLD. The risk remains in non-browser contexts and for same-script look-alikes. | [4] primary |
| `аpple.com` is `xn--pple-43d.com`, and `xn--microsft-hrb.com` is a Microsoft look-alike | The first is correct. The second decodes to a label starting with ŋ and is not a Microsoft look-alike. | [7] computed locally |
| Seven kill chain stages, and a different seven-stage diagram | The text uses the standard stage names. The diagram uses its own labels (Preparation, Access Persistence, Remote Activity, Impact), which are not the standard ones. | [6] secondary (original paper not read) |
| Phishing is an Initial Access technique | Confirmed as T1566, with sub-techniques T1566.001 to T1566.004. | [3] primary |
| Read Received headers bottom up, and the first hop is the truth. Private IPs and a Return-Path mismatch are red flags | Reading bottom up is right, but only hops added by servers you trust are reliable. Private addresses and different Return-Path values appear in legitimate mail. This is analyst judgment, not a cited standard, so test it on real headers. | Unsourced |
| Highly skilled users are more vulnerable, and curiosity fuels zero-day campaigns | The course gives no evidence for either. Not checked, and best presented as the course's view. | Not checked |
| OneNote and ISO attachments bypass basic filtering | Not checked. This was a prominent trend around 2022 to 2023, so confirm current status before repeating it. | Not checked |

## 9. Sources

1. **RFC 9989, DMARC** (IETF, May 2026). Primary. Read directly: sections 1, 2.2, 3.2, 4.4, 4.7, 5.3, 5.4 and 8. https://www.rfc-editor.org/rfc/rfc9989.html
2. **CISA, Implementing Phishing-Resistant MFA** (October 2022). Primary. Read through search extracts only. https://cisa.gov/sites/default/files/publications/fact-sheet-implementing-phishing-resistant-mfa-508c.pdf
3. **MITRE ATT&CK, T1566 Phishing.** Primary. Read through a search extract. https://attack.mitre.org/techniques/T1566/index.html
4. **Chromium project, Internationalized Domain Names (IDN) in Google Chrome.** Primary vendor documentation. https://chromium.googlesource.com/chromium/src/+/main/docs/idn.md
5. **Verizon 2026 Data Breach Investigations Report.** Secondary only: figures were taken from SC World and Push Security coverage, and the report itself was not read. https://www.scworld.com/news/verizon-dbir-2026-vulnerability-exploits-top-initial-access-as-patching-coverage-falls and https://pushsecurity.com/blog/verizon-dbir-2026-review
6. **Lockheed Martin Cyber Kill Chain** (Hutchins, Cloppert and Amin, 2011). Stage names confirmed through secondary summaries, including EC-Council's explainer. The original paper was not read. https://www.eccouncil.org/threat-intelligence/the-cyber-kill-chain-the-seven-steps-of-a-cyberattack/
7. **Local computation.** Punycode encodings and decodings were run with Python's built-in codec and the `idna` library. This is a check, not a source.
