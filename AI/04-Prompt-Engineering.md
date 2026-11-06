# [Prompt Engineering](https://tryhackme.com/room/promptengineeringaisec) - TryHackMe

**Path:** AI Fundamentals

**Focus:** How LLMs process text, how to control their output, and how to write effective prompts for security work

## Overview

This room covers the mechanics behind how an LLM reads and generates text, then moves on to structuring prompts and using specific techniques. It also sets up the security idea that the later prompt security material builds on: system and user instructions are not as separate as they look.

## LLM fundamentals

**Tokens** are the smallest units an LLM works with. Text is split into tokens (roughly 3 to 4 characters each, so most English words are one or two tokens), and each token is converted into a numeric ID. The model only ever sees those numbers and predicts which one comes next. Different models tokenise differently, for example GPT uses Byte-Pair Encoding and BERT uses WordPiece, so the same sentence can produce different token sequences.

**Nondeterminism** means the same prompt can give different answers each time, because the model samples from probabilities when picking the next token. Traditional software is deterministic, so the same input always gives the same output. The security consequence is that a defence might block a malicious prompt once and miss it the next time, which makes LLM behaviour much harder to test and defend than ordinary code.

Three parameters let you steer that behaviour:

| Parameter | What it controls | Notes |
|---|---|---|
| Temperature | How adventurous the next-token choice is | Low values (about 0.0 to 0.3) are closest to deterministic and suit code, extraction and factual answers. Higher values give more variety, and very high values become incoherent. |
| Max tokens | The length ceiling of the response | A cap, not a target, and also a cost control since many services charge per token |
| Top-p | Limits choices to the shortlist of tokens that together make up a set share of the probability (nucleus sampling) | The advice is to adjust temperature or top-p, not both |

The **context window** is the model's working memory, measured in tokens. If a conversation exceeds it, earlier content is dropped, so the model can effectively forget the start of a long conversation.

## The four pillars of a prompt

1. **Instruction:** the core task, stated with a clear verb such as write, analyse, summarise or compare.
2. **Context:** background, audience or a role, so the model has less to guess.
3. **Output format:** how the answer should be structured, such as bullets, a table, JSON or a word limit.
4. **Constraints:** rules and limits on the response, such as tone, topics to avoid or a maximum length.

The balance to aim for is specific without being verbose. A vague prompt leaves the model guessing, while a rambling one buries the task and can cause the model to ignore parts of it. A good prompt names the inputs, the checks and the expected result.

## System prompts and user prompts

A **system prompt** is set by the developer, persists across sessions and defines the assistant's role, rules and boundaries. A **user prompt** is written by the end user each time and carries the task or data. The intended order of priority is called the **instruction hierarchy**, with system rules winning over user requests.

The catch is that the model receives everything as one stream of text. The separation between roles comes from formatting conventions and training, not from a hard architectural barrier, so the model's respect for the hierarchy is probabilistic, not guaranteed. An attacker who writes user input that mimics or conflicts with system instructions, especially in persuasive or authoritative wording, may be able to override it. That is why the room calls the instruction hierarchy a security boundary and not just a design preference, and it connects directly to the prompt injection covered in AI Security Threats.

## Prompting techniques

- **Zero-shot:** an instruction and the input with no examples, relying on the model's training. It works for simple, familiar tasks.
- **One-shot:** a single input and output example, useful when the output format matters.
- **Few-shot:** two to five varied examples with an identical structure, which helps with complex or domain-specific patterns. This is called in-context learning.
- **Chain-of-thought (CoT):** asking the model to reason through intermediate steps before answering, introduced by Google researchers in 2022. It can be done with worked examples that show the reasoning, or as **zero-shot CoT** by adding "let's think step by step". The room notes that it works best on larger models, and that smaller ones can produce reasoning that looks coherent but reaches a wrong answer.
- **Prompt templates:** saved, reusable prompt structures for recurring tasks such as log analysis, IOC extraction or code review, which keep a team consistent and bake in what has been refined.

The room's point is that none of these is better than the others in general. Zero-shot suits simple tasks, one-shot suits format guidance, few-shot suits complex patterns, chain-of-thought suits multi-step reasoning that needs justification, and templates suit repeatable work.

## How this applies to a SOC

The most direct uses are log analysis, IOC extraction, alert summaries and incident write-ups, where a structured prompt with a defined output format gives more usable results than a loose question. Because outputs vary from run to run, an analyst should treat a single answer as a draft and verify it against the evidence, not assume the same prompt will always behave the same way. The instruction hierarchy point also matters for anyone defending an AI tool, since a system prompt alone is not a reliable security control.

## Key terms

Token, nondeterminism, temperature, max tokens, top-p, context window, system prompt, user prompt, instruction hierarchy, zero-shot, one-shot, few-shot, in-context learning, chain-of-thought, prompt template.

## Source notes

These notes follow the room's content. The specifics that date quickly or that I have not checked against primary sources are the context window sizes for named models, the temperature ranges (which differ between providers), and the claim that chain-of-thought works well only above roughly 100 billion parameters. Check the provider's documentation before quoting any of them.
