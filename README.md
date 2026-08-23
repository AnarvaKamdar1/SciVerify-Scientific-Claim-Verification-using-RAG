# SciVerify — Scientific Claim Verification using RAG

A retrieval-augmented pipeline for verifying scientific claims against evidence from the SciFact scientific literature corpus.

[Overview] • [Architecture] • [Experiments] • [Results]

---

## Overview

Scientific literature contains a rapidly growing amount of information, making it
difficult to efficiently determine whether a scientific claim is supported or
contradicted by existing evidence.

**SciVerify** is a scientific claim verification system built on the **SciFact**
benchmark. Given a scientific claim, the system retrieves relevant scientific
documents,re-ranks them for better lexical relevance and determines whether the
retrieved evidence supports or contradicts the claim, and generates a 
grounded explanation using retrieval-augmented generation with explicit citation.

The pipeline combines:

- **BM25** for scientific evidence retrieval
- **Query expansion** experiments for improving retrieval
- **Hybrid cross-encoder re-ranking** that combines normalized BM25 and cross-encoder relevance scores to refine the ranking of candidate documents while preserving lexical relevance
- **SciBERT** to classify each retrieved document as supporting or contradicting the claim
- **RAG/LLM-based generation** for producing evidence-grounded explanations with explicit citations to the retrieved documents
- Retrieval and verification evaluation through quantitative metrics and ablation studies

---

## Architecture
SciVerify follows a three-stage pipeline consisting of evidence retrieval,
claim-evidence verification, and grounded response generation.

![SciVerify Architecture](architecture.png)

## Experiments

### Query Expansion

To investigate whether pseudo-relevance feedback could improve scientific
evidence retrieval, two query expansion strategies were evaluated:

- **PRF with KL divergence**
- **RM3 (Relevance Model 3)**

Both approaches were evaluated against the same BM25 retrieval baseline using
Recall, Mean Reciprocal Rank (MRR), and Normalized Discounted Cumulative Gain
(NDCG) at multiple retrieval depths.

### Recall

| PRF with KL |  |  |  |  |  |  |
|---|---:|---:|---:|---:|---:|---:|
| Method | R@3 | R@5 | R@7 | R@10 | R@20 | R@50 |
| BM25 | 0.6861 | 0.7469 | 0.7812 | 0.8128 | 0.8517 | 0.8841 |
| BM25 + QE | 0.5372 | 0.6758 | 0.7239 | 0.7842 | 0.8459 | 0.8799 |
| BM25 + QE + Reranking | 0.6490 | 0.7124 | 0.7648 | 0.7929 | 0.8521 | 0.8799 |
| BM25 + Reranking | **0.7278** | **0.7697** | **0.7988** | **0.8387** | **0.8691** | **0.8841** |

| RM3 |  |  |  |  |  |  |
|---|---:|---:|---:|---:|---:|---:|
| Method | R@3 | R@5 | R@7 | R@10 | R@20 | R@50 |
| BM25 | 0.6861 | 0.7469 | 0.7812 | 0.8128 | 0.8517 | 0.8841 |
| BM25 + QE | 0.6503 | 0.7278 | 0.7636 | 0.8136 | 0.8432 | 0.8767 |
| BM25 + QE + Reranking | 0.6689 | 0.7359 | 0.7717 | 0.8049 | 0.8560 | 0.8767 |
| BM25 + Reranking | **0.7278** | **0.7697** | **0.7988** | **0.8387** | **0.8691** | **0.8841** |

## Results

