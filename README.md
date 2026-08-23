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

## Results

