# Python for CyberSecurity


**1. Python fundamentals (2–4 weeks)**
- *Automate the Boring Stuff with Python* (free online, automatetheboringstuff.com) — practical, not academic. Best starting point.
- freeCodeCamp's Python course on YouTube if you prefer video.
- Skip anything CS-theory-heavy for now (data structures/algorithms courses) — you don't need that for defense scripting.

**2. Security-specific Python (once basics feel comfortable)**
- *Black Hat Python* by Justin Seitz — technically offense-leaning but the networking/socket chapters translate directly to understanding what you're detecting.
- *Violent Python* (older, free PDF floats around, some code is dated but concepts hold) — has detection-relevant sections (log parsing, forensics).
- TryHackMe's Python for Pentesters / Python Playground rooms — since you're already on TryHackMe for your Hacker's Holiday work, this slots right in and reuses a platform you know.

**3. Applied practice — build small tools instead of just following tutorials**
- Write a script that parses a sample Windows Event Log (EVTX) export and flags suspicious login patterns — use `python-evtx` (free, pip-installable).
- Pull IOCs from a text file and check them against VirusTotal's free-tier API.
- Parse a CSV of fake/sample log data and summarize top talkers, failed logins, etc. with `pandas`.

These map directly onto SOC Tier 1 work and give you something concrete for your portfolio repo — you're already doing writeups on GitHub, so a small "log-parsing-scripts" repo alongside your CSA and CTF work would round things out nicely.

**4. Libraries worth knowing specifically for blue team work**
- `requests` (API calls to threat intel sources)
- `pandas` (log/CSV analysis)
- `python-evtx`, `Registry` (Windows forensics)
- `volatility3` (memory forensics, more advanced)

One honest note: don't over-invest here before you've nailed core SOC skills (SIEM querying, ticketing, alert triage logic). Python is a strong differentiator, not a substitute — pace it as a side track alongside your CSA and lab work rather than the main focus.
