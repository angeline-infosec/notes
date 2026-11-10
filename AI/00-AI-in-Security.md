# AI in Cybersecurity: Quick Reference

A single recall sheet for interviews and SOC work, covering the TryHackMe AI Fundamentals rooms and the Securing AI Systems room content. It is split into AI basics, threats to AI, AI used by attackers, frameworks, AI used by defenders (SIEM, EDR, DFIR), legal and ethical points, and safe adoption.

## 1. AI basics

**Hierarchy:** AI > Machine Learning > Deep Learning > Generative AI > LLMs. Each is a subfield of the one before it.

**ML lifecycle:** define the problem, collect data, clean it, choose an algorithm, engineer features, train and evaluate, deploy, then monitor and retrain. It is a loop, because the world changes after deployment.

**Four ML categories**

| Type | Data | Security example |
|---|---|---|
| Supervised | Labelled | Classifying logs as malicious or benign, spam filters |
| Unsupervised | Unlabelled | Anomaly detection, UEBA, clustering user behaviour |
| Semi-supervised | Few labels, many unlabelled | Fraud detection with few confirmed cases |
| Reinforcement | Rewards and penalties | Autonomous pentesting agents |

**Neural networks and deep learning:** layers of weighted nodes (input, hidden, output). Training adjusts the weights. The room defines deep learning as more than three layers, and says it learns its own features from raw data. Many deep models are still trained on labelled data, so treat "needs no labels" as a simplification.

**LLMs:** transformer-based deep learning models that generate text by predicting the next token. Pre-training adjusts billions of parameters through backpropagation, and RLHF (human feedback) then shapes the model into a usable assistant. Attention lets the model weigh words by context.

**Overfitting:** a model that memorises training data and fails on new data. It also makes memorised sensitive data easier to reproduce.

## 2. LLM mechanics and prompting

- **Tokens:** the units a model reads (about 3 to 4 characters each), converted to numeric IDs. Tokenisation differs between models.
- **Nondeterminism:** the same prompt can give different answers. Code behaves identically every time, so a defence against a malicious prompt may work once and fail the next time.
- **Parameters:** low temperature (about 0.0 to 0.3) is closest to deterministic and suits extraction and code. Higher values give variety. Max tokens caps length and cost. Top-p limits choices to a probability shortlist, and the advice is to change temperature or top-p, not both.
- **Context window:** the model's working memory in tokens. Overflowing it drops earlier content.
- **Four pillars of a prompt:** instruction, context, output format, constraints. Specific beats vague, and concise beats rambling.
- **System vs user prompt:** the system prompt is developer-set and persistent, and the user prompt is per request. The model sees both as one stream of text, so the "instruction hierarchy" is a convention and not a hard boundary. That is why prompt injection works.
- **Techniques:** zero-shot (no examples), one-shot, few-shot (2 to 5 varied examples), chain-of-thought (step-by-step reasoning, or "let's think step by step"), and reusable prompt templates for repeated tasks.

## 3. Threats to AI systems

**AI assets to protect**

Training data is the raw material the model learns from (text, images, logs), not a database of facts that the model looks up. Model weights are the numbers that encode the patterns it learned from that data, and they are the model itself.

| Asset | What it is | Why it matters |
|---|---|---|
| Training data | The datasets used to teach the model its behaviour | Poisoning it corrupts the model at the source, and the damage is baked into the model |
| Model weights / parameters | The numerical values that define what the model has learned | Stealing them gives an attacker a working copy of the model |
| Embedding vectors | Numerical representations of text or data, used for similarity search, retrieval (RAG) and as inputs to other models | Poisoning or manipulating them changes what information the model sees at query time |
| System prompts | Instructions that define the model's behaviour, constraints and persona | Leaking them reveals security controls, business logic and guardrails, which gives attackers a roadmap |
| Feature stores | Preprocessed data repositories that feed real-time model inputs | Tampering changes what the model sees at inference time without touching the model |
| Model registry / artifacts | A versioned store of trained models ready for deployment | A compromised registry lets an attacker swap a legitimate model for a backdoored one (this is unrelated to the Windows Registry) |

**The five model vulnerabilities**

| Vulnerability | Meaning | Impact |
|---|---|---|
| Prompt injection | Crafted input makes the model ignore its instructions or restrictions | Unauthorised output or actions |
| Data poisoning | Malicious data inserted into training data | Wrong, biased or manipulated predictions |
| Model theft | Copying a model, often by querying its API and training a clone (model extraction) | Loss of intellectual property |
| Privacy leakage | The model exposes sensitive data from training or interactions | Confidentiality breach |
| Model drift | Accuracy falls as real-world data changes | Missed threats, so monitoring is a security task |

**OWASP Top 10 for LLM Applications (2025)**

| ID | Risk | In plain terms |
|---|---|---|
| LLM01 | Prompt Injection | Crafted input changes model behaviour |
| LLM02 | Sensitive Information Disclosure | Leaks of confidential data, PII or internals |
| LLM03 | Supply Chain | Compromised models, datasets or dependencies |
| LLM04 | Data and Model Poisoning | Corrupted training data or weights |
| LLM05 | Improper Output Handling | Model output passed unchecked into other systems |
| LLM06 | Excessive Agency | Too many tools, permissions or autonomy |
| LLM07 | System Prompt Leakage | Hidden instructions exposed |
| LLM08 | Vector and Embedding Weaknesses | Attacks on retrieval (RAG) and embeddings |
| LLM09 | Misinformation | False content generated |
| LLM10 | Unbounded Consumption | Resource exhaustion or runaway cost |

**LLM attack surface by category (LLM Security room cheat sheet)**

| Category | Threat | What it targets | Result |
|---|---|---|---|
| Data-based | Training data extraction | Training dataset | Memorised text, PII or secrets reproduced |
| Data-based | Membership inference | Whether a sample was in the training data | A yes/no (or probability) answer |
| Data-based | System prompt leakage (LLM07) | Developer instructions | Hidden prompt disclosed |
| Model-based | Weight extraction (model stealing) | Model parameters | A surrogate model built from many API queries |
| Model-based | Model inversion | Internal representations | Training data or attributes reconstructed |
| System-based | Context window poisoning (prompt injection) | Context window and instruction hierarchy | Altered behaviour, policy bypass, unintended actions |
| System-based | Context overflow (LLM10) | Context size and resources | Truncated safeguards, degraded service or denial of service |
| System-based | Memory poisoning | Persistent conversation memory | Corrupted or misleading future responses |
| User-based | LLM-powered social engineering | Human decision-making | Phishing, fraud and coerced actions |
| User-based | Trust exploitation (LLM09) | User trust | False or unsafe information accepted |

The pattern to remember is that LLMs process every input as one context, so the attack surface covers the data they learned from, the model itself, the system around it and the people using it.

**The AI data and model supply chain**

- **Provenance:** being able to say where data came from, when it was collected and whether it was modified. Most AI supply chains cannot. The AI equivalent of an SBOM is the **ML-BOM**.
- **Common Crawl** underpins almost every major model. Truffle Security found 11,908 live secrets in a 400 TB sample of it, so credentials can end up in model weights and cannot be patched out afterwards.
- **Quantisation and pruning** compress models after training and can quietly weaken safety behaviour or backdoor defences, usually done by a third party with little documentation.
- **Federated learning** keeps raw data local, but a participant can submit poisoned updates that are hard to detect.
- **Fine-tuning inheritance:** fine-tuning does not change the base weights, so you inherit everything in them, including weakened safety alignment and unknown backdoors. Fine-tuned models can also be more prompt-injectable, and the exact base version matters.
- **Black box and model cards:** weights cannot be audited directly, and testing only samples behaviour. A model card (training data, intended use, evaluation, limitations, bias, licence) is the audit trail, and a missing or thin one is a warning sign.

## 4. AI used by attackers

| Threat | Meaning | What to check |
|---|---|---|
| AI-generated malware | Generative AI creates or modifies malicious code faster and with less skill | Behaviour and detections, not authorship |
| Deepfakes | AI-generated or manipulated audio, images or video imitating real people | Urgency, a new payee, "don't call me back", verify through a known channel |
| AI-enhanced phishing | Fluent, personalised, targeted messages at scale | Lookalike domains, links, urgency and requests for credentials or IDs, since bad grammar is no longer a signal |

## 5. Frameworks

| Framework | Question it answers |
|---|---|
| MITRE ATT&CK | How do adversaries attack conventional IT? The current Enterprise matrix lists 15 tactics and now includes a few AI-related techniques such as prompt injection |
| MITRE ATLAS | How do adversaries attack AI systems? Same tactic and technique structure, with case studies |
| OWASP LLM Top 10 | Which weaknesses matter most in LLM applications? |
| NIST AI RMF | How does an organisation manage AI risk? Govern, Map, Measure, Manage |
| NIST AI 100-4 | How are AI-generated content provenance, watermarking and detection approached? |
| ISO/IEC 27090 | Guidance on security threats specific to AI systems (named in the room) |

**ATT&CK vs ATLAS**

| | MITRE ATT&CK | MITRE ATLAS |
|---|---|---|
| Covers | Attacks on conventional IT (endpoints, networks, cloud) | Attacks on AI and ML systems (training data, models, inference APIs, LLM applications) |
| Structure | Tactics (why), techniques (how), sub-techniques, mitigations | The same structure, plus case studies of real AI attacks |
| Size | Enterprise matrix v20: 15 tactics | 16 tactics, 155 techniques, 35 mitigations, 52 case studies (room, early 2026, and the counts keep growing) |
| AI-specific tactics | None | For example AI Model Access and AI Attack Staging |
| Example techniques | Phishing, Valid Accounts, Command and Scripting Interpreter | LLM Prompt Injection, Training Data Poisoning, Extract AI Model, Evade AI Model |
| Overlap | | Reuses some ATT&CK techniques (such as Phishing and Valid Accounts) and adds AI-specific ones |
| Use it for | Tagging alerts, planning hunts and checking detection coverage for conventional intrusions | Threat modelling AI systems and describing incidents that involve them |

They are complementary, with the same structure applied to a different domain. An incident involving an AI system often maps to both.

In a SOC, ATT&CK is mostly a shared language: tag alerts with technique IDs, check detection coverage, plan hunts and describe incidents consistently. An incident involving an AI system can be described in both ATLAS and ATT&CK.

## 6. Threat modelling AI systems (STRIDE, ATLAS, OWASP)

The three work as zoom levels. STRIDE-AI gives the wide view of what could go wrong, ATLAS gives the technical detail of how an attacker would do it, and the OWASP LLM Top 10 shows where each risk lives in the architecture and how critical it is. In practice: break the system into components, walk each one through the six STRIDE categories, look up the matching ATLAS technique, map the OWASP risk to the component, then apply mitigations. See the general STRIDE note for the basics.

| STRIDE | Primary AI manifestation | Other AI threats | Example |
|---|---|---|---|
| Spoofing | Data source impersonation (RAG injection) | Model impersonation, adversarial identity attacks | Fake policy documents injected into a chatbot's knowledge base |
| Tampering | Data poisoning | Model manipulation, prompt injection, feature tampering | Crafted transactions shift a fraud model's decision boundary |
| Repudiation | No audit trail for model decisions | Context volatility, model version ambiguity | Cannot explain why the fraud model approved a suspicious transaction |
| Information disclosure | Model extraction (stealing) | Training data extraction, prompt leakage, embedding inversion | A competitor rebuilds a recommendation engine through API queries |
| Denial of service | Inference cost exploitation (denial of wallet) | GPU exhaustion, sponge examples, pipeline disruption | Expensive prompts push the cloud bill up 12 times while the service stays up |
| Elevation of privilege | Jailbreaking and guardrail bypass | Excessive agency, tool exploitation, cross-plugin escalation | A jailbroken chatbot uses its database tools to pull customer PII |

Prompt injection's STRIDE category depends on context: it is Tampering when the attacker alters the model's input, and Elevation of Privilege when the goal is bypassing guardrails. STRIDE also misses adversarial examples, model bias and emergent behaviour, which is why ATLAS is added.

**Key ATLAS techniques and their STRIDE category**

| Technique | ID | STRIDE |
|---|---|---|
| Training Data Poisoning | AML.T0020 | Tampering |
| Extract AI Model (model stealing) | AML.T0024.002 | Information disclosure |
| Infer Training Data Membership | AML.T0024.000 | Information disclosure |
| Evade AI Model (adversarial inputs) | AML.T0015 | Tampering, Spoofing or Elevation of Privilege |
| LLM Prompt Injection (direct .000, indirect .001) | AML.T0051 | Tampering |
| Backdoor ML Model (hidden trigger planted in training) | AML.T0018 | Tampering |

Two case studies to know: ShadowRay (AML.CS0023), where attackers exploited the Ray framework used for distributed AI workloads, showing that AI supply chain attacks happen in production, and the Morris II worm (AML.CS0024), a self-replicating prompt injection that spread between AI agents through RAG-based email systems.

**Where the OWASP risks live (read it in both directions)**

- The LLM inference endpoint carries the most risk, appearing in seven of the ten entries (LLM01, 02, 05, 06, 07, 09 and 10), so it needs the most hardening.
- The vector database and RAG pipeline carry indirect prompt injection (LLM01), embedding weaknesses (LLM08) and misinformation from stale sources (LLM09). Harden them with input validation on indexed content, access control and freshness monitoring.
- The training pipeline carries sensitive data entering training (LLM02), supply chain (LLM03) and poisoning (LLM04).
- Going from component to risk is the useful direction: when a new component is added, look up which OWASP entries it inherits.

## 7. AI used by defenders

**Where it shows up (SIEM, EDR, email security, DFIR)**

| Use | How AI helps | Tools the rooms name |
|---|---|---|
| Anomaly detection and UEBA | Unsupervised models (such as Isolation Forests and Autoencoders) learn a baseline and flag deviations without fixed rules | Splunk UEBA, Elastic ML, Exabeam |
| Phishing and communication analysis | Transformer models (BERT, RoBERTa) classify messages by tone, structure and known patterns | Microsoft Defender for O365, Splunk NLP |
| Malware and file classification | Classifies files from static and dynamic features | Microsoft Defender (STAMINA), Cylance, VirusTotal ML |
| Alert triage and prioritisation | Ranks alerts from past data and analyst feedback to cut noise | Cortex XSOAR/XSIAM, IBM QRadar Advisor, CrowdStrike Falcon with Charlotte AI |
| Timeline and correlation | Clusters and links logs across sources to rebuild attack chains | Timesketch, Velociraptor |
| LLM assistants | Summarise incidents, explain raw logs, suggest queries, and propose hunting ideas | Varies by product |

**Four defensive capabilities:** analysis (patterns at scale), prediction (detect and block before impact), summarisation (condense logs, reports and intel) and investigation (explain logs, suggest queries, help triage).

**Impact figures (IBM, as the room cites):** organisations using AI saved about $2.2 million per breach, contained breaches 108 days faster, and only 24% of generative AI initiatives were secured. These come from one edition of IBM's report, so check the current edition before quoting them.

**Judging an AI detection:** read the metrics together. Accuracy is misleading on imbalanced data (calling all 100 files benign scores 99% when one is malicious). Precision is how many flags were real, and recall is how many real threats were found. High precision alone can miss attacks, and high recall alone floods analysts. Also remember garbage in, garbage out, and that models drift as attacker behaviour changes.

**Lessons from the Digital Trail lab:** the ML scripts surfaced three log lines and nine files quickly and caught a disguised reverse shell, but they flagged legitimate source code as suspicious, never looked in the home directory where the phishing document lived, and explained none of their flags. A human followed the leads, built the timeline and separated true from false positives.

## 8. Legal and ethical points (DFIR)

- **Explainability:** black-box output can fail admissibility standards such as Daubert if nobody can explain why something was flagged.
- **Bias:** models reproduce skew in their training data, and facial recognition has produced wrongful arrests, so tools need validation.
- **Chain of custody:** every AI step on evidence must be logged and secured, or the findings can be challenged on procedure.
- **Privacy:** sending evidence to public AI services can breach privacy law or court orders, so prefer controlled or offline environments.

**Identifying AI-generated media:** no single test works. NIST groups approaches into provenance (such as C2PA Content Credentials, a signed record that shows whether a file changed and who signed it, but not whether the content is true), watermarking, and detection (classifiers such as ELA plus CNN). Detectors are probabilistic and chase new generators, so conclusions should rest on several methods together.

## 9. Securing and adopting AI safely

- Least privilege for every AI component, read-only by default, with RBAC and MFA on who can interact with it.
- Human approval before any write, delete or deployment action.
- Treat model output as untrusted input to other systems, and parameterise database queries.
- Keep secrets and internal URLs out of system prompts and assume prompts can be read.
- Rate limit, cap input length and set cost ceilings.
- Log and monitor tool calls, token use and prompt-extraction attempts, and strip sensitive data from stored conversations.
- Govern training data by auditing, minimising and encrypting it, and follow standards such as ISO/IEC 27090.
- Monitor deployed models for drift and odd outputs, and use explainability tools such as SHAP and LIME.
- Vet third-party models: model card, provenance, licence, base version, and documentation of any compression.

## 10. What a Tier 1 analyst should remember

- AI output is a lead, not a verdict. Verify it against the evidence.
- A model only looks where it was pointed, and it can say something is unusual without saying why.
- Be careful what is pasted into AI tools, since credentials and source code can leave the organisation without any attack.
- Fluent phishing, voice and video impersonation are cheap now, so rely on domain checks, out-of-band verification and behaviour.
- Document what is proven versus inferred, and note any AI-assisted step.

## 11. Likely interview questions

- **How is AI changing the threat landscape?** Attackers get fluent phishing, deepfakes and faster malware, and organisations add attack surface through AI systems themselves.
- **What is prompt injection?** Input that makes a model ignore its instructions. It works because the model sees system and user text as one stream.
- **How would you triage an AI-generated alert?** Treat it as a lead, check the underlying logs, look at precision and false positives, and document the reasoning.
- **What is model drift and why is it a security issue?** Accuracy falls as the data changes, so detections quietly stop catching new attacks.
- **ATT&CK vs ATLAS?** ATT&CK covers conventional intrusions and ATLAS covers attacks on AI systems. They share the same tactic and technique structure and are used together.
- **How would you threat model an AI system?** Break it into components, walk each through STRIDE, enrich the findings with ATLAS techniques, map OWASP LLM risks to the components, then apply mitigations.
- **Why might AI evidence be rejected in court?** It may be unexplainable, biased, undocumented in the chain of custody, or privacy-violating.
- **Precision vs recall?** Precision is how many flags were right, and recall is how many real threats were found. You need both.
- **What is the inheritance problem?** Fine-tuned models carry the base model's data, biases and weaknesses.

## Sources and verification notes

- MITRE ATT&CK (attack.mitre.org) and NIST AI 100-4 (nist.gov) are **primary** sources I checked. The C2PA specification (spec.c2pa.org) is also primary.
- OWASP Top 10 for LLM Applications (genai.owasp.org) is the primary site. The category names follow the 2025 list as given in the TryHackMe room and a secondary summary, so check the official PDF before quoting wording. The official site still presents the 2025 list as current, but a search result mentioned a 2026 resource, so check for a newer edition.
- The 16-tactic count for ATLAS comes from secondary sources, and the NIST AI RMF functions, ISO/IEC 27090, the IBM figures and the Truffle Security figures are as the TryHackMe rooms present them. Truffle's own post matches the numbers above.
- The LLM attack surface table is as the TryHackMe LLM Security room presents it, and I have not checked it against primary sources.
- ATLAS IDs: I confirmed Extract AI Model as AML.T0024.002 on MITRE's D3FEND site (primary). The other IDs (T0024.000, T0015, T0020, T0051) match secondary sources, and T0018 is as the room states it, so check atlas.mitre.org. The room labels model extraction as AML.T0024 and membership inference as AML.T0025, but current ATLAS lists both as sub-techniques of AML.T0024.
- The STRIDE-AI table, the ATLAS counts and the case studies are as the TryHackMe AI Threat Modelling room presents them.
- Tool names are as the rooms list them, and vendor features change.
