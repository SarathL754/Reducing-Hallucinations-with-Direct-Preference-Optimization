# Reducing Hallucinations with Direct Preference Optimization (DPO)

## A Controlled Study on Answerability-Aware Preference Learning using SQuAD 2.0

---

## Project Overview

Large Language Models (LLMs) fine-tuned with **Supervised Fine-Tuning (SFT)** often perform well on answerable questions but tend to **hallucinate answers when the question is unanswerable**. This project investigates whether **Direct Preference Optimization (DPO)** can reduce hallucinations by explicitly teaching the model *when to refuse*.

Using **SQuAD 2.0**, which naturally contains both answerable and unanswerable questions, we design a **high-precision preference construction pipeline** and train multiple DPO variants on a **TinyLLaMA** base model.

The key result: **DPO v2 significantly reduces hallucination on impossible questions while maintaining reasonable answer behavior**, outperforming both SFT and later over-regularized DPO variants.

---

## Key Goals

* Study hallucination behavior on *impossible questions*
* Compare **SFT vs DPO** under controlled evaluation
* Design a **balanced preference judge** to avoid DPO collapse
* Quantify hallucination reduction and refusal accuracy

---

## Dataset

* **SQuAD 2.0**

  * ~50% answerable questions
  * ~50% impossible (no answer in context)

We flatten the dataset into a unified evaluation format while preserving the `is_impossible` signal for analysis.

---

## ⚙️ Training Pipeline

### 1️⃣ Supervised Fine-Tuning (SFT)

* Train TinyLLaMA on answer-only outputs
* No explicit signal for refusal vs hallucination

Result:

* Good EM on answerable questions
* ❌ High hallucination on impossible questions

---

### 2️⃣ Preference Construction

We generate response pairs:

* **cand_a**: base / weaker response
* **cand_b**: SFT response

A custom **judge** selects (chosen, rejected) pairs using:

#### Impossible Questions

* Prefer refusal over any answer
* If both refuse → shorter wins
* If both answer → skip

#### Answerable Questions

* Prefer gold-matching answer
* Penalize refusal
* Skip ambiguous or noisy pairs

This produces a **high-precision preference dataset** suitable for DPO.

---

### 3️⃣ Direct Preference Optimization (DPO)

We trained multiple variants:

| Model      | Description       | Outcome              |
| ---------- | ----------------- | -------------------- |
| DPO v1     | Naive preferences | Over-refusal         |
| **DPO v2** | Balanced judge    | ✅ Best tradeoff      |
| DPO v3     | Over-regularized  | Performance collapse |

---

## 📊 Evaluation Metrics

We evaluate SFT and DPO models on the SQuAD 2.0 dev set using:

* **Answer Accuracy (EM)** on answerable questions
* **Hallucination Rate** on impossible questions
* **Refusal Accuracy** on impossible questions
* **Over-refusal Rate** on answerable questions
* Pairwise win-rate (SFT vs DPO)

---

## Key Results

### Hallucination on Impossible Questions

* **SFT**: ~18.6%
* **DPO v2**: ~4.6%  ✅

> **~75% relative reduction in hallucination**

### Behavioral Comparison

* DPO v2 is more conservative
* Strong refusal behavior when context is insufficient
* Avoids the severe collapse observed in DPO v3

---

## Visualization

A simple visualization is provided to compare hallucination rates:

<img width="940" height="807" alt="image" src="https://github.com/user-attachments/assets/196e06d0-a665-41e4-9a22-d6f585c93bc7" />


---

## Key Takeaways

* DPO is **extremely sensitive** to preference quality
* High-precision filtering beats large noisy datasets
* Over-regularization leads to refusal collapse
* Balanced DPO can meaningfully reduce hallucinations

---

## Future Work

* Add calibrated refusal confidence
* Combine DPO with rejection sampling
* Explore reward-weighted DPO
* Extend to other QA benchmarks (Natural Questions, TriviaQA)

---

## Why This Project Matters

This project demonstrates a **practical, reproducible approach** to controlling hallucinations in LLMs using preference learning—without RLHF, reward models, or human annotation.

It is especially relevant for:

* Safety-critical QA systems
* RAG pipelines
* Agentic workflows requiring abstention



## Acknowledgements

* TinyLLaMA
* SQuAD 2.0 Dataset
* DPO (Rafailov et al., 2023)
