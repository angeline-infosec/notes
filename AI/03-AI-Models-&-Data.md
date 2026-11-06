# [AI Models & Data](https://tryhackme.com/room/aimodelsdata) - TryHackMe

**Path:** AI Fundamentals

**Focus:** Where training data comes from, how model-building decisions create security risk, and why trained models are hard to audit

## Overview

The central idea of this room is that a model is a product of its training data and of every decision made while building it, so the security risks start long before deployment. Most of those decisions are poorly documented and almost never audited, which makes the AI data supply chain as exploitable as a software supply chain.

## Training data

Training data usually comes from four places, and each has a different level of trust.

| Source | What it is | Trust |
|---|---|---|
| Web scraping | Automated crawls of public content | Low: no curator, no version control, content changes after collection |
| Licensed datasets | Data bought or agreed with platforms | Medium: terms often unclear, original users rarely consented to AI training |
| Synthetic data | AI-generated content used to train other AI | Variable and growing |
| Internal corpora | Company knowledge bases, support transcripts | Higher: the organisation controls it, but also carries the liability |

The most widely used public corpus is **Common Crawl**, a free archive of web crawl data that underpins essentially every major model family.

**Data provenance** means being able to say where a piece of data came from, when it was collected, and whether it has been modified. In most AI supply chains none of these can be answered fully, because models are trained on composites of hundreds of upstream datasets where attribution has been lost. The software equivalent of the fix is the SBOM, and the AI version is the **ML-BOM**, an inventory of dataset sources, licences, PII categories and filtering decisions. Adoption is still early.

### PII and credentials in the weights

Large-scale scraping pulls in anything that was publicly accessible at crawl time, including personal data and secrets, and once it is baked into the weights it is very hard to remove. Truffle Security scanned the December 2024 Common Crawl archive (400 TB from 2.67 billion web pages) with TruffleHog and found 11,908 live secrets, meaning credentials that authenticated successfully, spread across 2.76 million pages. About 63% were repeated across multiple pages, and one WalkScore API key appeared 57,029 times across 1,871 subdomains. A model trained on this data can sometimes be prompted into reproducing such content, and this is not an attack on the model but a consequence of what went into it, so no patch fixes it after deployment.

## Building the model

- **Epoch:** one complete pass of the training algorithm through the whole dataset. More epochs do not always help, because training too long leads to **overfitting**, where the model memorises its training data. For security this matters because memorised data can include sensitive details that the model may reproduce when prompted.
- **Validation set:** a portion of the data held back from training and used to test the model on unseen data. If training accuracy keeps rising while validation accuracy stalls or drops, the model is overfitting. Validation is the quality gate, and skipping it means the model's real behaviour is unknown.
- **Pruning:** removes parameters that contribute little, shrinking the model. It changes behaviour after training and is rarely documented in detail.
- **Quantisation:** lowers the numerical precision of the weights (for example 32-bit to 8-bit) to cut memory and compute. The room says it can quietly degrade safety behaviour, and that backdoor defences tested on a full-precision model may fail to detect threats in the quantised version. Both steps are often done by a third party packaging the model, so a downloaded quantised model carries unknown changes.
- **Federated learning:** training is spread across many devices or organisations, and only weight updates, not raw data, go back to a central server. It reduces privacy risk at the data level, but it makes training integrity harder to verify, because a participant can submit poisoned updates that are difficult to spot during aggregation. It solves one trust problem and creates another.

## The inheritance problem

Most organisations do not train from scratch. They take a **pre-trained model** (trained on a large general dataset) and apply **fine-tuning**, which continues training it on a smaller task-specific dataset. Fine-tuning changes task behaviour, tone and domain knowledge, but it does not change the base weights, so everything in the base model is inherited, including data nobody downstream audited. The room gives three concrete consequences:

1. **Safety alignment erodes.** The room cites Stanford and Princeton research showing alignment can be compromised by fine-tuning on as few as 10 adversarial examples, and that even benign fine-tuning degrades safety as a side effect.
2. **Specialisation increases attack surface.** It cites Cisco as finding fine-tuned models more susceptible to prompt injection than their base models.
3. **Version matters.** Fine-tuning targets a specific checkpoint, so if that checkpoint later turns out to contain a backdoor or bad data, every derivative inherits it, and without knowing the exact version there is no way to assess exposure.

## The black box problem

A trained model is billions of floating-point numbers, the **weights**, with no human-readable record of how they were shaped. Testing and red teaming sample behaviour but do not audit it: they show how the model acted on inputs someone tried, not what it will do on inputs nobody thought of. Trusting a model therefore means trusting the process that produced it.

The **model card** is the main transparency tool, a structured document describing training data, intended use, evaluation results, known limitations, bias assessment and licence. It is voluntary for most uses, so cards are often incomplete, vague or missing, and the room treats a sparse or absent card as a warning sign and not an inconvenience.

## Practical: model card audit

The practical was a simulated Hugging Face-style repository for `trustedai-lab/enterprise-classifier-v2`. The task was to act as a security analyst reviewing the model card, files and sidebar details before sign-off, flagging suspicious text through hotspots and rating each finding's severity, with feedback on each one at the end.

*(Add the specific red flags you found and the severity you gave each one.)*

The room's content gives a reusable checklist for judging any third-party model:

- Does the model card exist, and does it cover training data, intended use, evaluation, limitations, bias and licence?
- Can the dataset sources and filtering be traced, and is there anything like an ML-BOM?
- Is the licence clear, and is it consistent with what the data allows?
- Which base model and exact checkpoint was it fine-tuned from?
- Was it pruned or quantised, and is that documented along with any safety or backdoor testing on the compressed version?
- Are validation results given, and do they show signs of overfitting?

## How this applies to a SOC

An analyst may never train a model, but the organisation they defend will probably adopt third-party models and AI tools. The useful habits from this room are asking where a model came from and what it contains before it is trusted, treating a missing model card as a risk indicator, and remembering that credentials exposed publicly can end up in model weights, which is one more reason leaked secrets need to be rotated and not just removed.

## Key terms

Data provenance, Common Crawl, ML-BOM, epoch, overfitting, validation set, pruning, quantisation, federated learning, pre-trained model, fine-tuning, weights, model card.

## Source notes

- The Truffle Security figures match the post the room links to, which is the original research and so a primary source for those numbers.
- The other statistics in the room (the synthetic data share, the Data Provenance Initiative licence audit, the Stanford and Princeton result, the Cisco finding) are given here as the room states them, and I have not checked them against the original papers.


