# SciVerify Methodology

This document details the various techniques and models used in the SciVerify pipeline to perform scientific claim verification. The system is designed to be highly accurate, utilizing a multi-stage process from initial retrieval to final verdict generation.

## 1. Evidence Retrieval: BM25

The first step in the pipeline involves retrieving a set of candidate documents from the SciFact corpus that might contain evidence related to a given scientific claim. For this initial retrieval phase, SciVerify uses the BM25 algorithm.

BM25 (Best Matching 25) is a robust and widely used probabilistic information retrieval model. It ranks documents based on the occurrence and frequency of query terms in each document, while also factoring in document length. 
- **Term Frequency:** It rewards documents where the claim terms appear more frequently.
- **Inverse Document Frequency:** It assigns higher weight to terms that are rare across the entire corpus, filtering out common but uninformative words.
- **Length Normalization:** It penalizes extremely long documents to ensure fair scoring across documents of varying sizes.

## 2. Query Expansion Models

To improve the recall of the initial BM25 retrieval, query expansion techniques are used to augment the original claim with additional relevant terms. This is particularly useful in scientific literature where different authors might use distinct terminology to describe the same concepts. SciVerify experimented with two pseudo-relevance feedback (PRF) methods:

### PRF with KL Divergence
This method assumes that the top retrieved documents from the initial BM25 pass are relevant (pseudo-relevance). It builds a language model of these top documents and compares it against the language model of the entire corpus using Kullback-Leibler (KL) divergence. Terms that have a high probability in the top documents but a low probability in the general corpus are selected to expand the query. 

### Relevance Model 3 (RM3)
RM3 is another pseudo-relevance feedback technique that combines the original query model with a relevance model built from the top retrieved documents. 
- It estimates the probability of each term given the top documents.
- It interpolates these probabilities with the original query term probabilities.
- The highest-scoring terms are then appended to the original claim for a second pass of retrieval, often bridging the lexical gap between the claim and relevant evidence.

## 3. Hybrid Cross-Encoder Re-ranking

While BM25 is efficient for retrieving an initial candidate pool, it only relies on exact lexical matches and lacks a deep semantic understanding of the text. To refine the ranking of the candidate documents, SciVerify employs a hybrid re-ranking approach.

- **Cross-Encoder Model:** A pre-trained language model (acting as a cross-encoder) takes the claim and a candidate document simultaneously as input and computes a relevance score based on their semantic relationship.
- **Hybrid Scoring:** The cross-encoder score is combined with the normalized BM25 score. This ensures that the final ranking benefits from both the deep semantic alignment provided by the neural model and the exact keyword matching preserved by BM25.

## 4. Verdict Classifiers

Once the most relevant documents are identified, the system must determine the stance of each document with respect to the claim. The verdict classifier assigns one of three labels: SUPPORT, CONTRADICT, or NEI (Not Enough Information). SciVerify utilizes specialized BERT architectures for this task:

### SciBERT
SciBERT is a variant of the BERT architecture that has been pre-trained on a large corpus of scientific publications. Because its vocabulary and internal representations are tuned to scientific text, it is highly effective at understanding the complex jargon and nuanced argumentation found in the SciFact dataset.

### BioBERT
BioBERT is another domain-specific language model, pre-trained primarily on biomedical research articles. Given that many claims in the SciFact benchmark pertain to biology and medicine, BioBERT provides a strong alternative with excellent generalization capabilities for biomedical text verification.

Both models evaluate the claim-evidence pair and output a probability distribution over the three stance classes. These predictions are then used in the final generation stage to formulate a grounded response.

