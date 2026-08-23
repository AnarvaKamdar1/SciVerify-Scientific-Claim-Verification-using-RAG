# SciVerify — Scientific Claim Verification using RAG

A retrieval-augmented pipeline for verifying scientific claims against evidence from the SciFact scientific literature corpus.

[Overview] • [Methodology] • [Experiments] • [Results] • [Reproducibility]

---

## Overview

Scientific literature contains a rapidly growing amount of information, making it
difficult to efficiently determine whether a scientific claim is supported or
contradicted by existing evidence.

**SciVerify** is a scientific claim verification system built on the **SciFact**
benchmark. Given a scientific claim, the system retrieves relevant scientific
documents, determines whether the retrieved evidence supports or contradicts the
claim, and generates a grounded explanation using retrieval-augmented generation.

The pipeline combines:

- **BM25** for scientific evidence retrieval
- **SciBERT** for claim-evidence verdict classification
- **Query expansion** experiments for improving retrieval
- **Transformer-based verdict models** for improving classification
- **RAG/LLM-based generation** for producing evidence-grounded explanations
- Retrieval and verification evaluation through quantitative metrics and ablation studies

---

## Pipeline

```text
                         Scientific Claim
                                │
                                ▼
                       ┌─────────────────┐
                       │ Query Processing │
                       └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │ BM25 Retrieval  │
                       │   SciFact      │
                       └────────┬────────┘
                                │
                          Top-k Documents
                                │
                                ▼
                       ┌─────────────────┐
                       │ Evidence /      │
                       │ Sentence        │
                       │ Selection       │
                       └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │    SciBERT      │
                       │ Verdict Model   │
                       └────────┬────────┘
                                │
                     SUPPORT / CONTRADICT
                                │
                                ▼
                       ┌─────────────────┐
                       │   RAG / LLM     │
                       │ Explanation     │
                       └────────┬────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │ Final Claim     │
                       │ Verification    │
                       └─────────────────┘
