# STRIDE

A threat modelling method. You break a system into components, then ask of each component whether each of six kinds of threat could apply. Each letter is a threat category that violates one security property.

| Letter | Threat | Property violated | Meaning | Simple example |
|---|---|---|---|---|
| S | Spoofing | Authenticity | Pretending to be someone or something you are not | Logging in with a stolen password |
| T | Tampering | Integrity | Changing data or code without authorisation | Editing a database record or a config file |
| R | Repudiation | Non-repudiability | Denying that you performed an action | A user claims they never sent the email |
| I | Information disclosure | Confidentiality | Exposing information to unauthorised parties | A verbose error page leaking internal details |
| D | Denial of service | Availability | Making a system or resource unavailable | Flooding a server with traffic |
| E | Elevation of privilege | Authorisation | Gaining access beyond what is permitted | A normal user becoming an administrator |

## How to use it

1. Draw the system and its trust boundaries.
2. For each component, walk through the six categories and write down what could go wrong.
3. Record a mitigation for each threat found.

## Repudiation vs non-repudiation

**Repudiation** is denying an action you performed. **Non-repudiation** means you cannot credibly deny it, because there is evidence tying the action to you. Your understanding is right.

One refinement: a message by itself is not enough proof, since someone else could have used your phone or account. What creates non-repudiation is evidence that can be tied to one person and checked, such as a digital signature on the message, authenticated and tamper-evident logs, and trusted timestamps. For example, a digitally signed email cannot easily be denied, while a plain unsigned one can. In a SOC the same idea shows up as audit logs: a Windows 4624 login event and the Logon ID that follows it connect actions to a specific authenticated session.

Typical fixes for repudiation threats are audit logging, digital signatures and protected, time-stamped logs.

## STRIDE for AI systems

The same six categories apply to AI components, but they look different (for example data poisoning as Tampering and denial of wallet as Denial of service). See the STRIDE-AI table in AI-Security-Quick-Reference.md.
