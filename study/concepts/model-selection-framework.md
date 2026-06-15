---
type: concept-card
date: 2026-06-14
tags: [#type/concept, #session/1]
source: "Managing AI Projects, Ch. 2 - Choosing the Right Technique"
deck: model-selection
mastery: new
created: 2026-06-14
last-revised: 2026-06-14
version: 1.0
status: active
---

# Classic ML vs Deep Learning vs GenAI - Decision Framework

**Q:** Your team is evaluating three approaches for a new AI feature: classic ML (random forest), deep learning (custom neural net), and GenAI (LLM with RAG). What's your decision framework? Walk through when to pick each based on data, explainability, cost, and risk.

**A:**
- **Three techniques, different tradeoffs:**

  **Classic ML (random forest, XGBoost, logistic regression):**
  - **When to use:** Structured/tabular data, need explainability, small-to-medium datasets (100s to 100Ks of rows).
  - **Pros:** Fast to train, interpretable (feature importance), works with less data, predictable performance.
  - **Cons:** Won't learn complex patterns (images, text, sequences), manual feature engineering required.
  - **Example:** Fraud detection (need to explain why transaction flagged), customer churn prediction.

  **Deep Learning (CNNs, RNNs, transformers you train yourself):**
  - **When to use:** Unstructured data (images, audio, text), large datasets (millions of examples), pattern too complex for classic ML.
  - **Pros:** Learns features automatically, handles complex patterns (image recognition, speech).
  - **Cons:** Needs massive data, expensive to train (GPUs), black box (hard to explain), fragile (distribution shift breaks it).
  - **Example:** Medical image classification, speech recognition.

  **GenAI (LLMs via API, RAG, prompting):**
  - **When to use:** Text generation, reasoning, few-shot learning, rapid prototyping without training data.
  - **Pros:** Zero training (use pre-trained model), handles novel tasks, generates human-like text, fast to deploy.
  - **Cons:** Expensive at scale (pay per token), hallucinations (makes stuff up), no control over model internals, API dependency.
  - **Example:** Document summarization, conversational interfaces, content generation.

- **Decision framework:**
  1. **Data type:** Structured → classic ML. Unstructured (images/audio) → deep learning. Text generation/reasoning → GenAI.
  2. **Data volume:** <10K rows → classic ML or GenAI (few-shot). 10K-1M → classic ML or deep learning. 1M+ → deep learning.
  3. **Explainability:** Need to explain every decision (regulated) → classic ML. Black box OK → deep learning or GenAI.
  4. **Cost:** Budget-constrained → classic ML (cheap to train/run). Budget flexible → deep learning (upfront GPU cost) or GenAI (ongoing API cost).
  5. **Time to deploy:** Need it fast → GenAI (no training). Have time to build → classic ML or deep learning.
  6. **Risk tolerance:** Low (production-critical) → classic ML (predictable). Medium → deep learning (test thoroughly). High (prototype) → GenAI (iterate fast).

- **So-what:** Start with the simplest technique that could work. Classic ML beats deep learning on tabular data 90% of the time. Deep learning beats classic ML on images/audio. GenAI beats both for rapid prototyping but costs more at scale. Don't pick deep learning because it's trendy; pick it because your data/task actually needs it.

## On-ramp

**Classic ML** = hand-crafted recipe (you tell it which ingredients matter: "age + income + purchase history"). **Deep Learning** = learns the recipe from millions of examples (you just show it inputs and outputs). **GenAI** = pre-trained chef (you describe what you want, it generates it without training).

## Common pitfalls

- Jumping to deep learning for tabular data (random forest often wins and trains in seconds)
- Using GenAI for high-volume, repetitive tasks (API costs explode; train a model instead)
- Picking technique based on hype instead of data type and explainability needs
- Not starting with simplest approach first (classic ML baseline before trying deep learning)
- Ignoring operational costs (GenAI API bills, deep learning GPU hosting, classic ML is cheap)

## Connects to

[[Bias-Variance Tradeoff]] (applies to all three techniques) · [[Model Explainability Tradeoffs]] (classic ML wins on explainability) · [[Feature Engineering]] (critical for classic ML, automatic in deep learning)

## Source

*Managing AI Projects*, Ch. 2 - [open](https://learning.oreilly.com/library/view/-/9798341641006/) · Section: "Choosing the Right AI Technique"
