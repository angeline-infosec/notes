# AI in Security: Quick Reference

A one-page recall sheet for interviews and SOC work, built from the AI Fundamentals rooms and the Securing AI Systems room content.

## The frameworks and what each one is for

| Framework | The question it answers | Notes |
|---|---|---|
| OWASP Top 10 for LLM Applications (2025) | What are the most critical vulnerabilities in LLM applications? | A ranked list of ten risk categories |
| MITRE ATLAS | How do adversaries attack AI systems? | Tactics, techniques and case studies, built as a counterpart to ATT&CK |
| MITRE ATT&CK | How do adversaries attack conventional IT? | The current Enterprise matrix now includes a few AI-related techniques, such as prompt injection |
| NIST AI Risk Management Framework | How does an organisation manage AI risk? | Four functions: Govern, Map, Measure, Manage |
| NIST AI 100-4 | How do we identify and prove AI-generated content? | Covers provenance, watermarking and detection |

A simple way to remember it: OWASP names the weakness, ATLAS shows how it is exploited, and the NIST framework asks whether the organisation has a repeatable process for dealing with it.

## OWASP Top 10 for LLM Applications (2025)

| ID | Risk | In plain terms | Example |
|---|---|---|---|
| LLM01 | Prompt Injection | Crafted input changes what the model does | Claiming to be the person updating MENTOR's instructions made it reveal its system prompt |
| LLM02 | Sensitive Information Disclosure | The system leaks confidential data, PII or internal details | Engineers pasting proprietary code into a public chatbot, as the room describes |
| LLM03 | Supply Chain | Compromised pre-trained models, datasets or dependencies | A downloaded model with no model card or documented origin |
| LLM04 | Data and Model Poisoning | Training data or model weights are corrupted | Mislabelled emails teaching a spam filter to trust malicious mail |
| LLM05 | Improper Output Handling | Model output is passed unchecked into another system | LLM text containing SQL reaching a database query |
| LLM06 | Excessive Agency | The AI has more tools, permissions or autonomy than it needs | A code review assistant with write access to a production database |
| LLM07 | System Prompt Leakage | Hidden instructions are exposed | Internal endpoints stored in a system prompt |
| LLM08 | Vector and Embedding Weaknesses | Attacks on retrieval (RAG) and embeddings | Poisoned documents in the vector store |
| LLM09 | Misinformation | The model produces false or misleading content | A chatbot inventing a refund policy |
| LLM10 | Unbounded Consumption | Resource exhaustion or runaway cost | Thousands of maximum-length requests |

## What changes when AI is added to a system

- Input becomes free-form text, so most existing input validation no longer fits.
- Outputs are probabilistic, so the same input can give different results, which makes testing and defence harder.
- The model receives system and user instructions as one stream of text, so the separation between them is a convention and not a hard boundary.
- New components appear (prompt construction, tool layer, vector store, logging of every conversation), each with its own trust boundary.

## Defensive habits that keep coming up

- Give every AI component the least privilege it needs, with read-only access by default.
- Require human approval before any write, delete or deployment action.
- Never trust model output as input to another system, and parameterise database queries.
- Keep secrets, credentials and internal URLs out of system prompts, and write prompts assuming they will be read.
- Rate limit, cap input length and set cost ceilings.
- Log and monitor tool calls, token use and attempts to extract prompts, and strip sensitive data from stored conversations.

## What a Tier 1 analyst should remember

- AI output is a lead and not a verdict. Verify it against the evidence, as the AEGIS and Digital Trail labs showed, where the models over-claimed or flagged legitimate files.
- A model only looks where it was pointed, and it can say something is unusual without saying why.
- Read accuracy, precision and recall together, since any one of them can be misleading on its own.
- Be careful what is pasted into AI tools, because credentials and source code can leave the organisation without any attack.
- Phishing, voice and video impersonation are harder to spot now that fluent, personalised content is cheap, so rely on domain checks, out-of-band verification and behaviour.

## Sources and verification notes

- OWASP Top 10 for LLM Applications (genai.owasp.org): **primary** site. The ten category names above follow the 2025 list as given in the TryHackMe room and a secondary summary, so check the official PDF before quoting wording. The official site still presents the 2025 list as the current LLM Top 10, but a search result mentioned a 2026 resource, so check whether a newer edition exists.
- MITRE ATT&CK (attack.mitre.org): **primary**, checked directly for the AI-related techniques.
- NIST AI 100-4 (nist.gov): **primary**. The details of NIST AI 100-2 and the AI RMF functions are as the TryHackMe room states them.
- The examples marked "as the room describes" come from TryHackMe and have not been checked against the original reporting.
