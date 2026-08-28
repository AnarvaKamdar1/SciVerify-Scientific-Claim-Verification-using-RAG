# <p align="center"> SciVerify — Scientific Claim Verification using RAG </p>

A retrieval-augmented pipeline for verifying scientific claims against evidence from the SciFact scientific literature corpus.

[Overview] • [Architecture] • [Experiments] • [Results] • [References]

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

The project explores:

- **BM25** for scientific evidence retrieval
- **Query expansion** experiments for improving retrieval
- **Hybrid cross-encoder re-ranking** that combines normalized BM25 and cross-encoder relevance scores to refine the ranking of candidate documents while preserving lexical relevance
- **SciBERT** to classify each retrieved document as supporting or contradicting the claim
- **RAG/LLM-based generation** for producing evidence-grounded explanations with explicit citations to the retrieved documents
- Retrieval and verification evaluation through quantitative metrics and ablation studies

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Evidence Retrieval** | Uses BM25 algorithm to initially retrieve candidate documents from the SciFact corpus. |
| **Hybrid Re-ranking** | Refines candidate document ranking by combining cross-encoder semantic scores with BM25 lexical scores. |
| **Verdict Classification** | Utilizes domain-specific models like BioBERT to classify evidence as SUPPORT, CONTRADICT, or NEI. |
| **Grounded Generation** | Leverages RAG/LLM to produce grounded explanations with explicit citations to retrieved documents. |

---

## Architecture
SciVerify follows a three-stage pipeline consisting of evidence retrieval,
claim-evidence verification, and grounded response generation.

```mermaid
graph TD
    A[Scientific Claim] --> B(Stage 1 Evidence Retrieval)
    
    subgraph Stage 1 Retrieval
        B --> C[BM25 Initial Retrieval]
        C --> F[Hybrid Cross-Encoder Re-ranking]
    end
    
    F --> G[Top-K Candidate Documents]
    
    G --> H(Stage 2 Claim-Evidence Verification)
    
    subgraph Stage 2 Verification
        H --> I[Verdict Classifier using BioBERT]
        I --> L[Predict SUPPORT / CONTRADICT / NEI]
    end
    
    L --> M(Stage 3 Grounded Response Generation)
    G --> M
    
    subgraph Stage 3 Generation
        M --> O[LLM Synthesizes Evidence]
    end
    
    O --> P[Final Explanation with Citations]
```

## Ablation Experiments

### 1. Query Expansion

To investigate whether pseudo-relevance feedback could improve scientific
evidence retrieval, two query expansion strategies were evaluated:

- **PRF with KL divergence**
- **RM3 (Relevance Model 3)**

Both approaches were evaluated against the same BM25 retrieval baseline using
Recall, Mean Reciprocal Rank (MRR), and Normalized Discounted Cumulative Gain
(NDCG) at multiple retrieval depths.

#### 1. a) Recall

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

#### 1. b) MRR

| PRF with KL |  |  |  |  |  |  |
|---|---:|---:|---:|---:|---:|---:|
| Method | MRR@3 | MRR@5 | MRR@7 | MRR@10 | MRR@20 | MRR@50 |
| BM25 | 0.6111 | 0.6253 | 0.6308 | 0.6338 | 0.6364 | 0.6372 |
| BM25 + QE | 0.3944 | 0.4271 | 0.4344 | 0.4409 | 0.4448 | 0.4457 |
| BM25 + QE + Reranking | 0.5150 | 0.5280 | 0.5352 | 0.5380 | 0.5423 | 0.5432 |
| BM25 + Reranking | **0.6556** | **0.6646** | **0.6693** | **0.6741** | **0.6759** | **0.6763** |

| RM3 |  |  |  |  |  |  |
|---|---:|---:|---:|---:|---:|---:|
| Method | MRR@3 | MRR@5 | MRR@7 | MRR@10 | MRR@20 | MRR@50 |
| BM25 | 0.6111 | 0.6253 | 0.6308 | 0.6338 | 0.6364 | 0.6372 |
| BM25 + QE | 0.5956 | 0.6119 | 0.6172 | 0.6226 | 0.6245 | 0.6256 |
| BM25 + QE + Reranking | 0.6061 | 0.6194 | 0.6251 | 0.6290 | 0.6326 | 0.6334 |
| BM25 + Reranking | **0.6556** | **0.6646** | **0.6693** | **0.6741** | **0.6759** | **0.6763** |

#### 1. c) NDCG

| PRF with KL |  |  |  |  |  |  |
|---|---:|---:|---:|---:|---:|---:|
| Method | NDCG@3 | NDCG@5 | NDCG@7 | NDCG@10 | NDCG@20 | NDCG@50 |
| BM25 | 0.6245 | 0.6503 | 0.6630 | 0.6736 | 0.6841 | 0.6913 |
| BM25 + Query Expansion | 0.4268 | 0.4852 | 0.5031 | 0.5221 | 0.5387 | 0.5456 |
| BM25 + Query Expansion + Reranking | 0.5418 | 0.5683 | 0.5881 | 0.5975 | 0.6133 | 0.6189 |
| **BM25 + Reranking** | **0.6690** | **0.6865** | **0.6979** | **0.7111** | **0.7194** | **0.7228** |

| RM3 |  |  |  |  |  |  |
|---|---:|---:|---:|---:|---:|---:|
| Method | NDCG@3 | NDCG@5 | NDCG@7 | NDCG@10 | NDCG@20 | NDCG@50 |
| BM25 | 0.6245 | 0.6503 | 0.6630 | 0.6736 | 0.6841 | 0.6913 |
| BM25 + Query Expansion | 0.6057 | 0.6376 | 0.6511 | 0.6670 | 0.6746 | 0.6822 |
| BM25 + Query Expansion + Reranking | 0.6182 | 0.6470 | 0.6607 | 0.6715 | 0.6854 | 0.6897 |
| **BM25 + Reranking** | **0.6690** | **0.6865** | **0.6979** | **0.7111** | **0.7194** | **0.7228** |

### 2. Verification Classifiers

#### 2. a) SciBERT:

SciBERT Model is a pre-defined BERT model specialized for the SciFact Dataset.
The classification metrics recorded over a held-out set are - 

| Class | Precision | Recall | F1-Score | Support |
|:------|----------:|-------:|---------:|--------:|
| CONTRADICT | 0.5000 | 0.5897 | 0.5412 | 39 |
| SUPPORT | 0.6667 | 0.6757 | 0.6711 | 74 |
| NEI | 0.8448 | 0.7424 | 0.7903 | 66 |
| **Accuracy** | | | **0.6816** | **179** |
| **Macro Avg** | **0.6705** | **0.6693** | **0.6675** | **179** |
| **Weighted Avg** | **0.6960** | **0.6816** | **0.6868** | **179** |

#### 2. b) BioBERT:

BioBERT Model is another variant which has strong generalization capability.
The classification metrics recorded over a held-out set are -

| Class       | Precision | Recall | F1-Score | Support |
|-------------|-----------|--------|----------|---------|
| CONTRADICT  | 0.5870    | 0.6923 | 0.6353   | 39      |
| SUPPORT     | 0.7857    | 0.7432 | 0.7639   | 74      |
| NEI         | 0.8889    | 0.8485 | 0.8682   | 66      |
| **Accuracy**|           |        | **0.7709** | **179** |
| **Macro Avg** | 0.7539  | 0.7613 | 0.7558   | 179     |
| **Weighted Avg** | 0.7805 | 0.7709 | 0.7743 | 179 |

## Results

The final pipeline was evaluated end-to-end on scientific claims from the
SciFact benchmark. For each claim, SciVerify retrieves relevant scientific
documents, re-ranks the candidates, predicts a claim-document verdict, and
uses the retrieved evidence and verdicts to generate a grounded explanation.

Representative outputs from the final pipeline are available in
[`final_outputs.txt`](final_outputs.txt).

### Example

**Claim:**  
> A deficiency of vitamin B12 increases blood levels of homocysteine.

**Retrieved evidence and verdicts:**

| Document Title | Verdict | Support Probability | Contradict Probability |
|---|---|---:|---:|
| British Journal of Nutrition...* | CONTRADICT | 0.4329 | 0.4688 |
| Homocysteine and the risk... | CONTRADICT | 0.2174 | 0.7024 |
| Effect of homocysteine... | CONTRADICT | 0.0816 | 0.7221 |
| Gene--nutrition interactions... | SUPPORT | 0.5497 | 0.3846 |
| The effect of folate... | CONTRADICT | 0.0633 | 0.7505 |

The final LLM response synthesizes the retrieved evidence and explicitly references the source documents used to support its explanation.

**Example generated response:**  
> The evidence is mixed with respect to the claim that a deficiency of vitamin B12 increases blood levels of homocysteine.

> Some studies suggest that a high intake of folate can decrease plasma homocysteine levels (Document 1), while others indicate that elevated homocysteine levels are associated with increased risk of ischemic stroke, with a threshold of 15 micromol/L (Document 2). However, these findings do not directly address the relationship between vitamin B12 deficiency and homocysteine levels.

> Document 4 suggests that a specific genotype of the MTHFR gene (C677T) may interact with folate levels to influence homocysteine levels, but this does not directly relate to vitamin B12.

> Document 5 found that folic acid supplementation may have a modest benefit on stroke prevention in regions without folate fortification, but this does not address the relationship between vitamin B12 deficiency and homocysteine levels.

> There is no clear evidence from the provided documents that directly links a deficiency of vitamin B12 to increased blood levels of homocysteine.

### End-to-End Output

The complete outputs generated by the final pipeline, including retrieved
documents, verdict probabilities, and grounded responses, can be found in
[`final_outputs.txt`](final_outputs.txt).

## Design Decisions

1. Query Expansion

Query expansion was initially expected to improve retrieval, but experiments showed that it underperformed the original query. Expansion often introduced less relevant documents, adding noise to re-ranking. This effect is especially noticeable in biomedical literature, where small terminology changes can substantially alter retrieval results. Therefore, the final system uses the original claim as the primary retrieval query.

2. Hybrid Re-ranking

Pure cross-encoder re-ranking was sometimes too aggressive and could push highly relevant BM25 results down the ranking. SciVerify therefore combines the BM25 score with the cross-encoder score, retaining the lexical retrieval signal while benefiting from semantic similarity.

3. Document-Level Verdict

SciVerify formulates verification as a claim-document classification task, since a single claim can be associated with multiple documents.

Although SciFact provides sentence-level evidence labels, these are intentionally extended to the document level:

A document is SUPPORT when all annotated evidence sentences are SUPPORT.
A document is CONTRADICT when all annotated evidence sentences are CONTRADICT.
Documents containing mixed labels are excluded because a single document-level verdict would be ambiguous.
Claims without annotated evidence are used to construct NEI examples.

The verifier receives the full abstract, rather than only the annotated evidence sentences. Thus, the model learns to determine whether the document as a whole supports, contradicts, or does not provide sufficient evidence for the claim.

This is an intentional document-level formulation; passage-level evidence attribution is left as a potential future extension.

4. LLM Inference

The LLM acts as a grounded interpretation and report-generation layer, rather than the primary verifier. It receives the classifier output together with retrieved documents and supporting context.

The LLM is not forced to blindly reproduce the classifier's verdict. If its interpretation differs from the classifier, this can expose cases where the document-level prediction and the underlying evidence are not perfectly aligned. Such disagreements are treated as potential failure or interpretability signals rather than assuming either component is always correct.

## References

The model weights after training will be available [here](https://huggingface.co/Anarva/SciVerify_Model_Files).

The dataset was downloaded from [here](https://github.com/allenai/scifact).

The methods and techniques are discussed in [methodology](methodology.md)





