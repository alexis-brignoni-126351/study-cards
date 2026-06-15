---
type: concept-card
date: 2026-06-14
tags: [#type/concept, #session/1]
source: "Managing AI Projects, Ch. 2 - Model Training Fundamentals"
deck: ml-fundamentals
mastery: new
created: 2026-06-14
last-revised: 2026-06-14
version: 1.0
status: active
---

# Bias-Variance Tradeoff

**Q:** Your model performs great on training data (98% accuracy) but terrible on test data (65% accuracy). Walk through the bias-variance tradeoff: what's happening, how do you diagnose it, and what levers do you have to fix it (with their costs)?

**A:**
- **What's happening:** Your model is overfitting. High variance (too sensitive to training data noise), low bias (fits training data well). It memorized the training set instead of learning generalizable patterns.

- **Diagnosis from train-vs-test gap:**
  - **High variance (overfitting):** Train accuracy >> Test accuracy. Model is too complex for the data.
  - **High bias (underfitting):** Train accuracy ≈ Test accuracy, but both are low. Model is too simple to capture the pattern.
  - **Just right:** Train accuracy ≈ Test accuracy, both high. Model generalizes.

- **Levers to fix overfitting (high variance):**
  - **Regularization:** Add penalty for model complexity (L1/L2). Cost: may underfit if too aggressive.
  - **More training data:** Dilutes noise, helps model see real pattern. Cost: expensive to label.
  - **Simpler model:** Fewer parameters, less capacity to memorize. Cost: may lose important signal.
  - **Early stopping:** Stop training before model memorizes. Cost: need validation set to know when.
  - **Dropout/data augmentation:** Force model to be robust. Cost: longer training time.

- **Levers to fix underfitting (high bias):**
  - **More complex model:** More parameters, more capacity. Cost: risk overfitting, slower training.
  - **Better features:** Engineer features that capture the pattern. Cost: requires domain knowledge.
  - **Less regularization:** Remove constraints. Cost: risk overfitting.

- **So-what:** The train-test gap tells you which way to move. Big gap? Regularize or get more data. No gap but low accuracy? Add complexity or better features. You're always trading off: too simple misses the pattern, too complex memorizes noise.

## On-ramp

Think of it like studying for an exam. **Overfitting** = memorizing the practice test answers word-for-word. You ace the practice test but fail the real exam because the questions are slightly different. **Underfitting** = not studying enough. You fail both practice and real exam. **Just right** = understanding the concepts so you can handle new questions.

## Common pitfalls

- Assuming high training accuracy means your model is good (it might just be overfitting)
- Only looking at test accuracy without checking the train-test gap (you can't diagnose without both)
- Adding regularization when you have high bias (makes underfitting worse)
- Getting more data when you have high bias (more data won't help if your model is too simple)
- Not using a validation set for early stopping (can't tell when to stop without it)

## Connects to

[[Train-Validation-Test Split Strategy]] (need 3 splits to diagnose and tune) · [[Feature Engineering]] (better features can reduce bias without adding model complexity) · [[Model Explainability Tradeoffs]] (simpler models for interpretability may accept higher bias)

## Source

*Managing AI Projects*, Ch. 2 - [open](https://learning.oreilly.com/library/view/-/9798341641006/) · Section: "Bias-Variance Tradeoff in Model Training"
