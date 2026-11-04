# [The Building Blocks of AI](https://tryhackme.com/room/aimlsecuritythreats) - TryHackMe

**Path:** AI Fundamentals, Room 1

## Overview

An entry-level room that explains how AI, machine learning, neural networks and LLMs fit together. Some tasks use TryHackMe's AI Agent platform, where you chat with an agent instead of using a terminal or VM.

## AI and Machine Learning

**AI is the broad field of making machines do tasks that normally need human reasoning, comprehension, Problem-solving, or creativity**. **Machine learning is the subfield where a system learns from data without explicit instructions.** The ML lifecycle runs from problem definition through data collection and cleaning, training and evaluation, deployment, and ongoing monitoring and retraining.

**Overfitting** is when a model memorises its training data and fails to generalise to new data.

## ML algorithm categories

Every ML algorithm has a decision process, an error function and an optimisation process that repeat in a loop until performance is good enough. The four categories differ in the data they use.

| Category | Data | Example |
|---|---|---|
| Supervised | Labelled | Classifying logs as malicious or benign |
| Unsupervised | Unlabelled | Clustering user behaviour, anomaly detection |
| Semi-supervised | Small labelled set, large unlabelled set | Fraud detection with few confirmed cases |
| Reinforcement | Rewards and penalties from an environment | Autonomous pentesting agent |

In the mission agent, my first justification for the reinforcement learning scenario was too vague and was rejected. It only accepted the answer once I explained that the agent learns by acting in an environment and receiving rewards or penalties.

## Neural networks and deep learning

**Deep learning is a specialized subset of machine learning and artificial intelligence (AI) that utilizes multi-layered artificial neural networks to automatically discover complex patterns in data**

A neural network has an input layer for raw data, hidden layers that extract increasingly complex features, and an output layer that gives the prediction. Connections between nodes carry weights, and training adjusts them. The room defines deep learning as a network with more than three layers.

The room also says deep learning does not need labelled data. I think that is a simplification, since many deep learning models are trained on labelled data and the real difference is that they learn their own features. I have not checked this against a primary source.

In the NEURON-1 task you play each layer by hand: give an input and its raw features, describe the pattern the hidden layer would detect, then give the final classification. I was confused at first and my initial reply did not advance it, but it clicked once I treated each step as one layer.

## Large Language Models

An LLM is a deep learning model that generates text by predicting the next word. It is pre-trained on huge amounts of text, and its parameters are adjusted through **backpropagation** based on how wrong each prediction was. **Transformers** (from the 2017 paper *Attention Is All You Need*) made this scale possible, and their key mechanism is **attention**, which weighs words by context. After pre-training, **RLHF**(Reinforcement Learning from Human Feedback (RLHF)) uses human feedback to shape the model into a usable assistant.

Hierarchy: AI > ML > Deep Learning > Generative AI > LLMs.

## SOC relevance

Supervised models power classifiers for logs and email, unsupervised models underpin anomaly detection, and monitoring and retraining explain why detection models drift. AI-generated alerts are probabilistic, so they need the same verification as any other alert.

## Key terms

Overfitting, supervised, unsupervised, semi-supervised and reinforcement learning, weights, backpropagation, transformer, attention, RLHF.
